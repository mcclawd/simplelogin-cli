# SimpleLogin CLI - Prioritized Backlog (ChatGPT User Perspective)

**Generated:** 2026-04-08  
**Priority Framework:** P0 (Critical) → P1 (High) → P2 (Medium) → P3 (Low)  
**Effort Scale:** S (Small, <1 day) → M (Medium, 1-3 days) → L (Large, >3 days)

---

## P0 - Critical Blockers (Fix Immediately)

### P0-01: Unify Documentation Between README.md and SKILL.md

**Priority:** P0  
**Effort:** S  
**Impact:** High  
**User Type:** All users

**Problem:**  
README.md omits contact-create/contact-list commands that are documented in SKILL.md. Users reading different files get different information.

**Tasks:**
- [ ] Add contact-create and contact-list to README.md usage section
- [ ] Ensure README.md troubleshooting matches SKILL.md
- [ ] Add all command examples from SKILL.md to README.md
- [ ] Create documentation sync checklist for future updates

**Acceptance Criteria:**
- README.md contains all features documented in SKILL.md
- No contradictory information between files
- Clear indication which file is primary reference

---

### P0-02: Clarify Installation Instructions

**Priority:** P0  
**Effort:** S  
**Impact:** High  
**User Type:** New users

**Problem:**  
Manual installation instructions use placeholder `yourusername`, don't explain PATH setup, and don't clarify which script to use (root vs scripts/).

**Tasks:**
- [ ] Replace placeholder with actual repo URL (or remove if not published)
- [ ] Add step-by-step PATH instructions for bash/zsh
- [ ] Clarify which `simplelogin` script is canonical
- [ ] Add `chmod +x` instruction
- [ ] Add verification step (`simplelogin --help`)

**Acceptance Criteria:**
- User can follow instructions and have working installation in <10 minutes
- No ambiguity about which files to use
- Clear success verification step

**Sample Updated Instructions:**
```bash
# Manual Installation

# 1. Clone the repository
git clone https://github.com/mcclawd/simplelogin-cli.git
cd simplelogin-cli

# 2. Make the script executable
chmod +x simplelogin

# 3. Add to your PATH (choose one location)
# Option A: Move to /usr/local/bin (system-wide)
sudo mv simplelogin /usr/local/bin/

# Option B: Add current directory to PATH (per-session)
export PATH="$PWD:$PATH"

# Option C: Add to ~/.bashrc or ~/.zshrc (persistent)
echo 'export PATH="$HOME/path/to/simplelogin-cli:$PATH"' >> ~/.bashrc

# 4. Verify installation
simplelogin --help
```

---

### P0-03: Add API Key Setup Walkthrough

**Priority:** P0  
**Effort:** S  
**Impact:** High  
**User Type:** New users

**Problem:**  
"Generate in SimpleLogin dashboard → API Keys" is too vague for non-technical users.

**Tasks:**
- [ ] Add numbered steps with screenshots or detailed descriptions
- [ ] Include exact navigation path in SimpleLogin UI
- [ ] Explain what permissions the API key needs
- [ ] Add troubleshooting for invalid/expired keys

**Acceptance Criteria:**
- User can find and generate API key without external help
- Clear what the key should look like
- Know what to do if key doesn't work

---

## P1 - High Priority (Next Release)

### P1-01: Add Debug/Verbose Mode

**Priority:** P1  
**Effort:** S  
**Impact:** High  
**User Type:** Technical users, troubleshooters

**Problem:**  
No way to see what's happening under the hood when commands fail.

**Tasks:**
- [ ] Add `SIMPLELOGIN_DEBUG=true` environment variable
- [ ] Output curl commands, headers, response codes, full responses
- [ ] Document in README/SKILL
- [ ] Add `--verbose` flag as alternative

**Acceptance Criteria:**
```bash
SIMPLELOGIN_DEBUG=true simplelogin create test
# Shows: curl command, API response, parsed output
```

---

### P1-02: Expand Troubleshooting Section

**Priority:** P1  
**Effort:** M  
**Impact:** High  
**User Type:** All users

**Problem:**  
Only 3 error scenarios documented; many common issues unaddressed.

**Tasks:**
- [ ] Add rate limiting errors
- [ ] Add network/curl failures
- [ ] Add invalid API key errors
- [ ] Add duplicate alias errors
- [ ] Add permission errors
- [ ] Add "command not found" troubleshooting
- [ ] Add diagnostic commands (how to check API key, connectivity, etc.)

**Acceptance Criteria:**
- Troubleshooting section covers 10+ common errors
- Each error has clear resolution steps
- Include links to SimpleLogin support/docs

---

### P1-03: Add Search Functionality

**Priority:** P1  
**Effort:** M  
**Impact:** High  
**User Type:** Power users, business users

**Problem:**  
Users accumulate many aliases; no way to find specific ones.

**Tasks:**
- [ ] Implement `simplelogin search <query>` command
- [ ] Search in email addresses and notes
- [ ] Support partial matching
- [ ] Add `--limit` flag
- [ ] Document with examples

**Acceptance Criteria:**
```bash
simplelogin search amazon
# Returns: amazon@alias.com, amazon-prime@alias.com, etc.

simplelogin search --limit 5 newsletter
```

---

### P1-04: Add Bulk Operations

**Priority:** P1  
**Effort:** M  
**Impact:** Medium  
**User Type:** Power users, business users

**Problem:**  
Managing many aliases requires repetitive manual commands.

**Tasks:**
- [ ] Implement `--all` flag for disable/enable/delete
- [ ] Implement domain-based bulk operations
- [ ] Add confirmation prompts for bulk actions
- [ ] Add dry-run mode
- [ ] Implement export to CSV/JSON

**Acceptance Criteria:**
```bash
simplelogin disable --domain example.com --all --dry-run
simplelogin export --format csv > aliases.csv
simplelogin export --format json > aliases.json
```

---

### P1-05: Add Package Manager Support

**Priority:** P1  
**Effort:** L  
**Impact:** High  
**User Type:** All users

**Problem:**  
Only ClawHub installation documented; mainstream users expect npm/Homebrew/apt.

**Tasks:**
- [ ] Publish to npm (as global package)
- [ ] Create Homebrew formula for macOS
- [ ] Consider apt repository for Debian/Ubuntu
- [ ] Update README with all installation methods
- [ ] Add version management

**Acceptance Criteria:**
```bash
# npm
npm install -g simplelogin-cli

# Homebrew
brew install simplelogin-cli

# Verify
simplelogin --version
```

---

## P2 - Medium Priority (Future Releases)

### P2-01: Add Statistics and Reporting

**Priority:** P2  
**Effort:** M  
**Impact:** Medium  
**User Type:** Business users, analytics-minded users

**Problem:**  
No insight into alias usage or account status.

**Tasks:**
- [ ] Implement `simplelogin stats` command
- [ ] Show total aliases, enabled/disabled counts
- [ ] Show emails forwarded (if API supports)
- [ ] Show storage/usage limits
- [ ] Add JSON output for automation

**Acceptance Criteria:**
```bash
simplelogin stats
# Total aliases: 42
# Enabled: 38, Disabled: 4
# Emails forwarded (30 days): 156
# Premium account: Yes
```

---

### P2-02: Interactive Mode

**Priority:** P2  
**Effort:** M  
**Impact:** Medium  
**User Type:** Non-technical users

**Problem:**  
CLI requires memorizing commands; some users prefer menus.

**Tasks:**
- [ ] Implement `simplelogin interactive` command
- [ ] Text-based menu interface
- [ ] Navigate options with arrow keys or numbers
- [ ] Show context/help for each action
- [ ] Allow quick alias creation from menu

**Acceptance Criteria:**
```bash
simplelogin interactive
# Shows menu:
# 1. Create custom alias
# 2. Create random alias
# 3. List aliases
# 4. Manage alias (enable/disable/delete)
# 5. Search
# 6. Stats
# 7. Exit
```

---

### P2-03: Manage Alias Notes Post-Creation

**Priority:** P2  
**Effort:** S  
**Impact:** Medium  
**User Type:** All users

**Problem:**  
Notes can only be set during creation, not updated later.

**Tasks:**
- [ ] Implement `simplelogin note set <alias> <note>`
- [ ] Implement `simplelogin note get <alias>`
- [ ] Implement `simplelogin note clear <alias>`
- [ ] Document with examples

**Acceptance Criteria:**
```bash
simplelogin note set shopping@alias.com "Amazon Prime - annual renewal in June"
simplelogin note get shopping@alias.com
# Output: Amazon Prime - annual renewal in June
```

---

### P2-04: Add Integration Examples

**Priority:** P2  
**Effort:** S  
**Impact:** Medium  
**User Type:** Automation users

**Problem:**  
No real-world examples of how to use this in workflows.

**Tasks:**
- [ ] Add shell script examples to README
- [ ] Create bash_aliases file with shortcuts
- [ ] Add GitHub Actions example
- [ ] Add cron job example for cleanup
- [ ] Add Zapier/webhook conceptual guide

**Acceptance Criteria:**
```bash
# Example: ~/.bash_aliases
alias sl-create='simplelogin create --for'
alias sl-list='simplelogin list --all'

# Example: Create alias for website you're visiting
sl-site() {
  local domain=$(echo "$1" | sed 's|https\?://||; s|/.*||')
  simplelogin create --for "$domain" --note "Auto: $(date +%Y-%m-%d)"
}
```

---

### P2-05: Document Security Best Practices

**Priority:** P2  
**Effort:** S  
**Impact:** Medium  
**User Type:** Security-conscious users

**Problem:**  
Security section is brief; needs more depth.

**Tasks:**
- [ ] Document API key permission requirements
- [ ] Recommend dedicated API keys per application
- [ ] Explain key rotation process
- [ ] Warn about committing secrets
- [ ] Link to SimpleLogin security documentation
- [ ] Add audit logging recommendations

**Acceptance Criteria:**
- Dedicated "Security Best Practices" section in README
- Clear do's and don'ts
- Links to external security resources

---

### P2-06: Improve Error Messages

**Priority:** P2  
**Effort:** M  
**Impact:** Medium  
**User Type:** All users

**Problem:**  
Some error messages are cryptic or unhelpful.

**Tasks:**
- [ ] Audit all error messages in scripts
- [ ] Make errors specific and actionable
- [ ] Add suggestions for resolution
- [ ] Include relevant context (what was the request, what failed)
- [ ] Add error codes for reference

**Acceptance Criteria:**
```bash
# Before:
Error: API call failed

# After:
Error: Failed to create alias
Reason: Duplicate alias "shopping@example.com" already exists
Suggestion: Use a different prefix or --random flag
Details: HTTP 409 from POST /v3/alias/custom/new
```

---

## P3 - Low Priority (Nice-to-Have)

### P3-01: Multiple Account Support

**Priority:** P3  
**Effort:** M  
**Impact:** Low  
**User Type:** Power users

**Problem:**  
Only one account configured at a time.

**Tasks:**
- [ ] Implement account profiles
- [ ] Switch between personal/work accounts
- [ ] Store multiple API keys securely
- [ ] Document use cases

---

### P3-02: Scheduled Alias Expiration

**Priority:** P3  
**Effort:** M  
**Impact:** Low  
**User Type:** Privacy-focused users

**Problem:**  
No automatic cleanup of temporary aliases.

**Tasks:**
- [ ] Add `--expires` flag to create commands
- [ ] Create cron helper for auto-disabling expired aliases
- [ ] Document limitations

---

### P3-03: Browser Integration

**Priority:** P3  
**Effort:** L  
**Impact:** Low  
**User Type:** Convenience-focused users

**Problem:**  
No browser integration for quick alias creation.

**Tasks:**
- [ ] Create bookmarklet
- [ ] Consider browser extension
- [ ] Integration with password managers

---

### P3-04: Alias Templates

**Priority:** P3  
**Effort:** S  
**Impact:** Low  
**User Type:** Business users

**Problem:**  
Common alias patterns must be typed repeatedly.

**Tasks:**
- [ ] Define template system
- [ ] Support variables (domain, date, random)
- [ ] Pre-configured templates for common services

---

### P3-05: Email Testing

**Priority:** P3  
**Effort:** M  
**Impact:** Low  
**User Type:** Testing/QA users

**Problem:**  
No way to verify alias is working without external email.

**Tasks:**
- [ ] Send test email to alias
- [ ] Check delivery status (if API supports)
- [ ] Report results

---

## Technical Debt

### TD-01: Script Duplication

**Priority:** P1  
**Effort:** M

**Problem:**  
Two `simplelogin` scripts exist (root and scripts/) with different implementations.

**Tasks:**
- [ ] Determine which is canonical
- [ ] Merge features from both
- [ ] Remove duplicate or clearly mark as deprecated
- [ ] Update documentation

---

### TD-02: Missing Tests

**Priority:** P2  
**Effort:** L

**Problem:**  
No automated test suite.

**Tasks:**
- [ ] Add bash unit tests (bats)
- [ ] Mock API responses for testing
- [ ] CI/CD integration
- [ ] Test coverage reporting

---

### TD-03: Hardcoded OpenClaw Paths

**Priority:** P2  
**Effort:** S

**Problem:**  
Script references `$HOME/.openclaw/secrets/` which is ecosystem-specific.

**Tasks:**
- [ ] Make secret paths configurable
- [ ] Support multiple secret backends
- [ ] Document OpenClaw-specific features separately

---

## Summary by Effort

| Priority | Small (S) | Medium (M) | Large (L) | Total |
|----------|-----------|------------|-----------|-------|
| P0 | 3 | 0 | 0 | 3 |
| P1 | 2 | 3 | 1 | 6 |
| P2 | 3 | 3 | 0 | 6 |
| P3 | 2 | 2 | 1 | 5 |
| Tech Debt | 1 | 1 | 1 | 3 |
| **Total** | **11** | **9** | **3** | **23** |

---

## Recommended Release Roadmap

### v1.1.0 (Immediate - 1-2 weeks)
- P0-01, P0-02, P0-03 (Documentation fixes)
- TD-01 (Script cleanup)

### v1.2.0 (Short-term - 1 month)
- P1-01, P1-02 (Debug mode, troubleshooting)
- P1-03 (Search)
- P2-06 (Better errors)
- P2-05 (Security docs)

### v1.3.0 (Medium-term - 2-3 months)
- P1-04 (Bulk operations)
- P1-05 (Package managers)
- P2-01 (Stats)
- P2-03 (Notes management)
- P2-04 (Integration examples)

### v2.0.0 (Long-term - 6+ months)
- P2-02 (Interactive mode)
- P3 features
- Major refactor if needed
