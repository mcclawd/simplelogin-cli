# SimpleLogin CLI - Prioritized Backlog

**Generated from:** Claude/Anthropic User Perspective Review  
**Date:** 2026-04-08  
**Skill Version:** 1.0.0  

---

## Backlog Organization

Items are organized by priority:
- **P0 (Critical)**: Security issues, blocking bugs, missing core functionality
- **P1 (High)**: Important improvements for usability and reliability
- **P2 (Medium)**: Nice-to-have features, documentation enhancements
- **P3 (Low)**: Polish, quality of life improvements

---

## P0 - Critical (Do These First)

### 1. Add API Key Validation Command
**Priority:** P0  
**Effort:** 1-2 hours  
**Impact:** High  
**Type:** Feature

**Description:**  
Add a command to validate the API key before performing operations. This prevents confusing errors when users have an invalid or expired key.

**Acceptance Criteria:**
- [ ] New command: `simplelogin auth-test` or `simplelogin whoami`
- [ ] Returns account information (email, plan type, premium status)
- [ ] Returns clear error if API key is invalid (HTTP 401)
- [ ] Works in both human-readable and JSON modes
- [ ] Exit code 0 on success, 1 on failure

**Example:**
```bash
$ simplelogin whoami
✓ Authenticated as: user@example.com
  Plan: Premium
  Suffixes available: 15

$ simplelogin whoami --json
{"email": "user@example.com", "plan": "premium", "suffixes_available": 15, "status": "authenticated"}
```

**Implementation Notes:**
- Use `/user/info` or `/user` endpoint
- Check for HTTP 200 response
- Parse and display relevant account fields

---

### 2. Comprehensive HTTP Error Handling
**Priority:** P0  
**Effort:** 2-3 hours  
**Impact:** High  
**Type:** Bug Fix / Improvement

**Description:**  
Currently, HTTP errors (401, 403, 404, 429, 5xx) are not handled explicitly, leading to confusing error messages or silent failures.

**Acceptance Criteria:**
- [ ] All API calls check HTTP status codes
- [ ] Specific error messages for each status code:
  - 401: "Invalid or expired API key"
  - 403: "Permission denied. Check your SimpleLogin plan."
  - 404: "Resource not found"
  - 429: "Rate limited. Please wait before retrying."
  - 5xx: "SimpleLogin server error. Try again later."
- [ ] Non-zero exit codes for all error conditions
- [ ] Error messages suggest corrective actions

**Implementation Notes:**
- Modify `api_call()` function to capture HTTP status
- Use `curl -w "%{http_code}"` to get status code
- Add case statement for error handling
- Don't suppress errors with `2>/dev/null` without logging

---

### 3. Add Curl Timeout Configuration
**Priority:** P0  
**Effort:** 30 minutes  
**Impact:** High  
**Type:** Bug Fix

**Description:**  
API calls have no timeout, which can cause the script to hang indefinitely if SimpleLogin API is unresponsive.

**Acceptance Criteria:**
- [ ] All curl calls include `--max-time 30` (or configurable)
- [ ] Timeout errors are handled gracefully
- [ ] Error message indicates timeout occurred
- [ ] Environment variable to customize timeout: `SIMPLELOGIN_TIMEOUT`

**Implementation:**
```bash
curl -s --max-time "${SIMPLELOGIN_TIMEOUT:-30}" \
    -H "Authentication: $API_KEY" \
    ...
```

---

### 4. Document Secure File Permissions
**Priority:** P0  
**Effort:** 15 minutes  
**Impact:** High  
**Type:** Documentation

**Description:**  
The secrets file (`$HOME/.openclaw/secrets/simplelogin.json`) should have restricted permissions, but this is not documented.

**Acceptance Criteria:**
- [ ] SKILL.md includes command to set permissions: `chmod 600`
- [ ] README.md mentions security best practices
- [ ] Automated check in script warns if permissions are too open

**Implementation:**
```bash
# Add to script initialization
if [[ -f "$HOME/.openclaw/secrets/simplelogin.json" ]]; then
    perms=$(stat -c %a "$HOME/.openclaw/secrets/simplelogin.json" 2>/dev/null)
    if [[ "$perms" != "600" && "$perms" != "400" ]]; then
        log_warn "Secrets file has insecure permissions ($perms). Consider: chmod 600"
    fi
fi
```

---

## P1 - High Priority (Do Next)

### 5. Implement Pagination for List Command
**Priority:** P1  
**Effort:** 2-3 hours  
**Impact:** Medium  
**Type:** Feature

**Description:**  
`simplelogin list` only fetches the first page (20 aliases). Users with many aliases cannot see all of them.

**Acceptance Criteria:**
- [ ] `list` command fetches all pages by default
- [ ] Option to limit results: `--limit 20`
- [ ] Progress indicator if fetching multiple pages
- [ ] Performance optimization: cache recently fetched pages

**Implementation Notes:**
- Loop through `page_id=0,1,2,...` until no more results
- Check for `.has_more` or empty result set
- Consider adding `--page` for manual pagination

---

### 6. Add Verbose/Debug Mode
**Priority:** P1  
**Effort:** 1-2 hours  
**Impact:** Medium  
**Type:** Feature

**Description:**  
Add a verbose mode to help users troubleshoot issues by showing API requests and responses.

**Acceptance Criteria:**
- [ ] New flag: `--verbose` or `-v`
- [ ] Environment variable: `SIMPLELOGIN_VERBOSE=true`
- [ ] Verbose output includes:
  - Full API request (URL, headers, body)
  - Full API response (status code, body)
  - Timing information
- [ ] Verbose output suppressed in JSON mode
- [ ] Redact API key in verbose output

**Example:**
```bash
$ simplelogin create test --verbose
ℹ API Request: POST https://app.simplelogin.io/api/alias/custom/new
ℹ Headers: Authentication: [REDACTED], Content-Type: application/json
ℹ Body: {"name": "test"}
ℹ Response: 201 Created (45ms)
✓ Created alias: test@example.com
```

---

### 7. Add Search/Filter Capability
**Priority:** P1  
**Effort:** 3-4 hours  
**Impact:** Medium  
**Type:** Feature

**Description:**  
Allow users to search for aliases by prefix, domain, note, or creation date.

**Acceptance Criteria:**
- [ ] New command: `simplelogin search <query>`
- [ ] Search across: alias prefix, domain, note
- [ ] Filter options: `--domain`, `--enabled`, `--disabled`, `--created-after`
- [ ] Output shows matching aliases with context
- [ ] Case-insensitive search

**Example:**
```bash
$ simplelogin search amazon
shopping@sl.co (note: Amazon purchases, enabled)
amazon@sl.co (note: Prime member, enabled)

$ simplelogin list --domain sl.co --enabled
```

---

### 8. Document API Limitations and Rate Limits
**Priority:** P1  
**Effort:** 30 minutes  
**Impact:** Medium  
**Type:** Documentation

**Description:**  
Users need to know about API rate limits, free vs. premium differences, and suffix availability.

**Acceptance Criteria:**
- [ ] SKILL.md includes "API Limits" section
- [ ] Documents rate limits (if known, or links to SimpleLogin docs)
- [ ] Explains free vs. premium feature differences
- [ ] Lists available suffixes/domains
- [ ] Links to SimpleLogin API documentation

**Research Required:**
- Check SimpleLogin API docs for rate limits
- Document differences between free and premium plans
- List public suffixes available for free accounts

---

### 9. Add Alias Statistics Command
**Priority:** P1  
**Effort:** 2-3 hours  
**Impact:** Medium  
**Type:** Feature

**Description:**  
Show usage statistics to help users understand their SimpleLogin activity.

**Acceptance Criteria:**
- [ ] New command: `simplelogin stats`
- [ ] Shows:
  - Total aliases created
  - Enabled vs. disabled
  - Emails forwarded (last 30 days)
  - Emails blocked (last 30 days)
  - Contact count
- [ ] JSON mode for programmatic access
- [ ] Works with free and premium accounts

**Example:**
```bash
$ simplelogin stats
Account: user@example.com (Premium)

Aliases: 47 total (42 enabled, 5 disabled)
Emails forwarded: 1,234 (last 30 days)
Emails blocked: 89 (last 30 days)
Contacts: 23
Top domains: sl.co (35), simplelogin.com (12)
```

---

### 10. Standardize Output Formats
**Priority:** P1  
**Effort:** 2 hours  
**Impact:** Medium  
**Type:** Improvement

**Description:**  
Output formats are inconsistent between commands. Standardize for predictability.

**Acceptance Criteria:**
- [ ] All commands follow consistent format in human mode
- [ ] All commands return valid JSON in JSON mode
- [ ] Success messages use consistent structure
- [ ] Error messages use consistent structure
- [ ] Document output format for each command

**Proposed Standard:**
```
# Success (human mode)
✓ Action completed: details

# Success (JSON mode)
{"status": "success", "action": "create", "alias": "email@domain.com"}

# Error (human mode)
✗ Error: description (suggestion for fix)

# Error (JSON mode)
{"status": "error", "error": "description", "code": "ERROR_CODE"}
```

---

## P2 - Medium Priority (Backlog)

### 11. Add Export/Import Functionality
**Priority:** P2  
**Effort:** 4-6 hours  
**Type:** Feature

**Description:**  
Backup all aliases to JSON file and restore from backup.

**Commands:**
- `simplelogin export [filename]` - Export all aliases
- `simplelogin import <filename>` - Import aliases from backup

**Acceptance Criteria:**
- [ ] Export includes: email, note, creation date, enabled status, mailbox
- [ ] Import skips existing aliases (or has `--force` flag)
- [ ] Progress indicator during import/export
- [ ] Validate backup file format before import

---

### 12. Differentiate README.md and SKILL.md
**Priority:** P2  
**Effort:** 2-3 hours  
**Type:** Documentation

**Description:**  
README.md currently duplicates SKILL.md. Differentiate them for clarity.

**Acceptance Criteria:**
- [ ] README.md is quick-start focused (5 min to first alias)
- [ ] SKILL.md is comprehensive reference
- [ ] README.md includes badges (version, license, CI status)
- [ ] SKILL.md includes full API reference
- [ ] Cross-link between documents where appropriate

**README.md Structure:**
- What is SimpleLogin? (1 paragraph)
- Quick Start (install, configure, create first alias)
- Basic Commands (create, list, disable)
- Links to full documentation

**SKILL.md Structure:**
- Full feature list
- Complete command reference
- Configuration options (detailed)
- Troubleshooting guide
- API reference
- Security best practices

---

### 13. Add Example Scripts Directory
**Priority:** P2  
**Effort:** 3-4 hours  
**Type:** Documentation / Tooling

**Description:**  
Provide example scripts showing common automation patterns.

**Acceptance Criteria:**
- [ ] Create `examples/` directory
- [ ] Include 3-5 example scripts:
  1. Auto-create alias for new domain
  2. Bulk disable old aliases
  3. Integration with password manager
  4. Monitoring/reporting script
  5. Email client integration
- [ ] Each example has comments explaining purpose
- [ ] Examples are tested and working

---

### 14. Add Shell Completion
**Priority:** P2  
**Effort:** 2-3 hours  
**Type:** Feature

**Description:**  
Add bash/zsh tab completion for commands and options.

**Acceptance Criteria:**
- [ ] `simplelogin completion bash` generates completion script
- [ ] `simplelogin completion zsh` generates completion script
- [ ] Completions include:
  - Command names
  - Common options
  - Existing aliases (for enable/disable)
- [ ] Installation instructions in README

**Example:**
```bash
# Enable bash completion
source <(simplelogin completion bash)

# Or install system-wide
simplelogin completion bash | sudo tee /etc/bash_completion.d/simplelogin
```

---

### 15. Add Bulk Operations
**Priority:** P2  
**Effort:** 3-4 hours  
**Type:** Feature

**Description:**  
Enable/disable/delete multiple aliases in one command.

**Acceptance Criteria:**
- [ ] Support multiple aliases: `simplelogin disable alias1.com alias2.com`
- [ ] Pattern matching: `simplelogin disable --pattern "*-temp*"`
- [ ] Confirmation prompt for bulk operations (unless `--force`)
- [ ] Progress indicator
- [ ] Summary of actions taken

**Example:**
```bash
$ simplelogin disable --domain sl.co --created-before 2025-01-01
Found 15 aliases matching criteria. Disable them? [y/N] y
ℹ Disabling 15 aliases...
✓ Disabled 15 aliases
```

---

### 16. Add Health Check Command
**Priority:** P2  
**Effort:** 1 hour  
**Type:** Feature

**Description:**  
Verify skill is working correctly for monitoring/automation.

**Acceptance Criteria:**
- [ ] New command: `simplelogin health`
- [ ] Checks:
  - API key validity
  - Network connectivity
  - API response time
  - Alias count (sanity check)
- [ ] Returns JSON for monitoring systems
- [ ] Exit code 0 if healthy, 1 if unhealthy

**Example:**
```bash
$ simplelogin health --json
{"status": "ok", "api_connected": true, "latency_ms": 234, "aliases_count": 47}

$ echo $?
0
```

---

### 17. Add Contribution Guidelines
**Priority:** P2  
**Effort:** 1-2 hours  
**Type:** Documentation

**Description:**  
Formalize contribution process for community involvement.

**Acceptance Criteria:**
- [ ] CONTRIBUTING.md file
- [ ] Code style requirements
- [ ] How to run tests (once tests exist)
- [ ] Pull request template
- [ ] Issue templates (bug report, feature request)

---

### 18. Fix Placeholder GitHub URLs
**Priority:** P2  
**Effort:** 15 minutes  
**Type:** Documentation Bug

**Description:**  
README.md has placeholder URLs (`yourusername`) that need to be updated.

**Acceptance Criteria:**
- [ ] Update clone URL to correct repository
- [ ] Update issue tracker link
- [ ] Update contribution link
- [ ] Verify all links work

---

### 19. Add API Key Rotation Guidance
**Priority:** P2  
**Effort:** 30 minutes  
**Type:** Documentation

**Description:**  
Document how to revoke and rotate API keys for security.

**Acceptance Criteria:**
- [ ] Security section mentions key rotation
- [ ] Steps to generate new API key
- [ ] How to revoke old key
- [ ] Recommended rotation schedule (e.g., every 90 days)

---

### 20. Add Version/Changelog
**Priority:** P2  
**Effort:** 1 hour  
**Type:** Documentation

**Description:**  
Track versions and changes for users.

**Acceptance Criteria:**
- [ ] CHANGELOG.md with version history
- [ ] Semantic versioning (major.minor.patch)
- [ ] Update package.json with proper version
- [ ] Version command: `simplelogin --version`

---

## P3 - Low Priority (Nice to Have)

### 21. Add Linting (ShellCheck)
**Priority:** P3  
**Effort:** 1-2 hours  
**Type:** Code Quality

**Description:**  
Integrate ShellCheck for bash syntax validation.

**Acceptance Criteria:**
- [ ] Add ShellCheck configuration (.shellcheckrc)
- [ ] Fix all current ShellCheck warnings
- [ ] Add linting to CI/CD workflow
- [ ] Document how to run linter locally

---

### 22. Add Test Suite
**Priority:** P3  
**Effort:** 8-12 hours  
**Type:** Code Quality

**Description:**  
Add automated tests using bats or shunit2.

**Acceptance Criteria:**
- [ ] Test framework selected (bats preferred)
- [ ] Tests for each command
- [ ] Mock API responses for isolated testing
- [ ] CI/CD integration
- [ ] Code coverage reporting

---

### 23. Add CI/CD Pipeline
**Priority:** P3  
**Effort:** 2-3 hours  
**Type:** Infrastructure

**Description:**  
Automated testing and deployment.

**Acceptance Criteria:**
- [ ] GitHub Actions workflow
- [ ] Run linter on PR
- [ ] Run tests on PR
- [ ] Automated releases on tag
- [ ] Publish to ClawHub on release

---

### 24. Add Interactive Mode (TUI)
**Priority:** P3  
**Effort:** 8-12 hours  
**Type:** Feature

**Description:**  
Text-based UI for browsing and managing aliases.

**Acceptance Criteria:**
- [ ] New command: `simplelogin ui` or `simplelogin interactive`
- [ ] Navigate alias list with arrow keys
- [ ] Quick actions (enable, disable, delete)
- [ ] Search/filter interface
- [ ] Use dialog, fzf, or similar TUI library

---

### 25. Add Configuration File Support
**Priority:** P3  
**Effort:** 2-3 hours  
**Type:** Feature

**Description:**  
Support `~/.config/simplelogin/config.json` for default settings.

**Acceptance Criteria:**
- [ ] Config file supports:
  - Default domain/suffix
  - Default mailbox
  - API key (optional, env var preferred)
  - Output format preferences
  - Timeout settings
- [ ] Environment variables override config file
- [ ] Command-line flags override both

---

### 26. Support Self-Hosted SimpleLogin
**Priority:** P3  
**Effort:** 2 hours  
**Type:** Feature

**Description:**  
Allow users to point the CLI at self-hosted SimpleLogin instances.

**Acceptance Criteria:**
- [ ] Environment variable: `SIMPLELOGIN_API_URL`
- [ ] Document self-hosting setup
- [ ] Handle self-signed certificates (or document how to configure)
- [ ] Test with self-hosted instance

**Example:**
```bash
export SIMPLELOGIN_API_URL="https://simplelogin.mydomain.com/api"
simplelogin list
```

---

### 27. Add Color Customization
**Priority:** P3  
**Effort:** 1 hour  
**Type:** Feature

**Description:**  
Allow users to customize or disable colors.

**Acceptance Criteria:**
- [ ] Environment variable: `SIMPLELOGIN_COLORS=false`
- [ ] Support NO_COLOR standard
- [ ] Optional: custom color configuration
- [ ] Documented in help text

---

### 28. Add Mailbox Management Commands
**Priority:** P3  
**Effort:** 3-4 hours  
**Type:** Feature

**Description:**  
List and manage mailboxes in addition to aliases.

**Acceptance Criteria:**
- [ ] `simplelogin mailbox list` - List all mailboxes
- [ ] `simplelogin mailbox create` - Add new mailbox
- [ ] `simplelogin mailbox set-default` - Change default mailbox
- [ ] Integration with alias creation (specify mailbox)

---

### 29. Add Webhook Support
**Priority:** P3  
**Effort:** 6-8 hours  
**Type:** Feature

**Description:**  
Trigger actions when alias receives email (requires SimpleLogin webhook support).

**Acceptance Criteria:**
- [ ] Documentation on webhook setup (if SimpleLogin supports)
- [ ] Example webhook handler script
- [ ] Common webhook actions documented
- [ ] Security considerations for webhooks

---

### 30. Create Packaging (deb, rpm, brew)
**Priority:** P3  
**Effort:** 4-6 hours  
**Type:** Distribution

**Description:**  
Official packages for easy installation.

**Acceptance Criteria:**
- [ ] .deb package for Debian/Ubuntu
- [ ] .rpm package for RHEL/Fedora
- [ ] Homebrew formula for macOS
- [ ] Arch Linux AUR package
- [ ] Installation instructions for each

---

## Summary

**Total Items:** 30  
**P0 (Critical):** 4 items  
**P1 (High):** 10 items  
**P2 (Medium):** 10 items  
**P3 (Low):** 6 items  

**Estimated Total Effort:** 80-100 hours

### Quick Wins (< 2 hours each)
1. Document secure file permissions (P0)
2. Add curl timeout (P0)
3. Document API limitations (P1)
4. Add health check command (P2)
5. Fix placeholder URLs (P2)
6. Add version/changelog (P2)

### High Impact, Medium Effort
1. Add API key validation (P0, 1-2h)
2. Comprehensive error handling (P0, 2-3h)
3. Verbose/debug mode (P1, 1-2h)
4. Search/filter capability (P1, 3-4h)
5. Standardize output formats (P1, 2h)

### Large Projects (> 6 hours)
1. Test suite (P3, 8-12h)
2. Interactive mode (P3, 8-12h)
3. Export/import functionality (P2, 4-6h)
4. Packaging for distribution (P3, 4-6h)

---

## Appendix: Implementation Order Recommendation

**Phase 1: Stability (Week 1)**
1. API key validation (P0)
2. HTTP error handling (P0)
3. Curl timeout (P0)
4. File permissions documentation (P0)

**Phase 2: Usability (Week 2-3)**
5. Pagination support (P1)
6. Verbose mode (P1)
7. Search/filter (P1)
8. Standardize output (P1)
9. Statistics command (P1)

**Phase 3: Features (Week 4-6)**
10. Export/import (P2)
11. Documentation restructure (P2)
12. Example scripts (P2)
13. Shell completion (P2)
14. Bulk operations (P2)
15. Health check (P2)

**Phase 4: Quality (Week 7-10)**
16. Linting (P3)
17. Test suite (P3)
18. CI/CD (P3)
19. Remaining P3 features as time permits
