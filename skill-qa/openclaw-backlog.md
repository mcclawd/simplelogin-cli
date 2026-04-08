# SimpleLogin CLI - Prioritized Backlog

**Generated:** 2026-04-08  
**From:** OpenClaw User Feedback Review  
**Skill:** simplelogin-cli

---

## Priority Legend

- **P0** - Critical blocker, must fix before publishing
- **P1** - High priority, fix in next release
- **P2** - Medium priority, backlog for v1.1
- **P3** - Low priority, nice-to-have
- **P4** - Future consideration, optional enhancement

---

## P0 - Critical Blockers

### 1. Consolidate Duplicate Scripts
**Issue:** Two different `simplelogin` scripts exist with different feature sets.

- Root script: `/root/.openclaw/workspace/skills/simplelogin-cli/simplelogin` (~270 lines, has contact commands)
- Scripts version: `/root/.openclaw/workspace/skills/simplelogin-cli/scripts/simplelogin` (~640 lines, missing contact commands)

**Action Items:**
- [ ] Audit both scripts, identify best features from each
- [ ] Merge into single canonical version (prefer `scripts/simplelogin` structure)
- [ ] Ensure contact-create and contact-list are present
- [ ] Remove duplicate root-level script
- [ ] Update `.clawhub/skill.json` to reference correct path
- [ ] Test all commands work in merged version

**Impact:** Users won't know which script to use; missing contact management breaks email workflows.

**Estimated Effort:** 2-3 hours

---

### 2. Fix Installation Documentation
**Issue:** README has placeholder GitHub URL and vague installation steps.

**Action Items:**
- [ ] Replace `https://github.com/yourusername/simplelogin-cli.git` with real URL
- [ ] Add explicit installation steps:
  ```bash
  git clone <real-repo-url>
  cd simplelogin-cli
  chmod +x scripts/simplelogin
  sudo ln -s $(pwd)/scripts/simplelogin /usr/local/bin/simplelogin
  ```
- [ ] Add ClawHub installation instructions with expected behavior
- [ ] Document how ClawHub installs the skill (symlink vs copy)

**Impact:** Users can't install the skill without guessing.

**Estimated Effort:** 30 minutes

---

### 3. Enhance .clawhub/skill.json
**Issue:** Minimal metadata prevents proper ClawHub integration.

**Current:**
```json
{
  "name": "simplelogin-cli",
  "version": "1.0.0"
}
```

**Action Items:**
- [ ] Add `bin` field pointing to main executable
- [ ] Add `scripts` array for files to install
- [ ] Add `description` for search/discovery
- [ ] Add `postinstall` script or instructions
- [ ] Consider adding `env` field for required environment variables

**Example:**
```json
{
  "name": "simplelogin-cli",
  "version": "1.0.0",
  "description": "Create and manage SimpleLogin email aliases",
  "bin": "scripts/simplelogin",
  "scripts": ["scripts/simplelogin"],
  "postinstall": "chmod +x scripts/simplelogin && echo 'Set SIMPLELOGIN_API_KEY to configure'"
}
```

**Impact:** ClawHub can't properly install or document the skill.

**Estimated Effort:** 30 minutes

---

### 4. Add API Key Validation Command
**Issue:** No way to test if API key is configured correctly.

**Action Items:**
- [ ] Add `simplelogin auth-test` or `simplelogin status` command
- [ ] Command should:
  - Fetch mailbox list from API
  - Validate API key works
  - Report number of mailboxes found
  - Return non-zero exit code on failure
- [ ] Add to documentation as first step in setup

**Example Output:**
```bash
$ simplelogin auth-test
✓ API key valid
→ Found 2 mailboxes
→ Default: personal@example.com
```

**Impact:** Users can't debug credential issues.

**Estimated Effort:** 1 hour

---

## P1 - High Priority

### 5. Port Contact Commands to scripts/ Version
**Issue:** Better-structured scripts/ version missing contact-create and contact-list.

**Action Items:**
- [ ] Copy `cmd_contact_create()` from root script
- [ ] Copy `cmd_contact_list()` from root script
- [ ] Integrate into `main()` dispatcher in scripts/ version
- [ ] Add to help text
- [ ] Test with real SimpleLogin account

**Impact:** Can't reply to forwarded emails without contacts.

**Estimated Effort:** 1 hour

---

### 6. Add Comprehensive Error Handling
**Issue:** Missing handling for rate limits, network failures, API errors.

**Action Items:**
- [ ] Add HTTP status code checking (401, 403, 429, 500)
- [ ] Add retry logic for transient failures
- [ ] Add JSON error output in JSON mode
- [ ] Handle malformed API responses
- [ ] Add timeout for curl requests

**Example:**
```bash
api_call() {
    local response=$(curl -s -w "%{http_code}" --max-time 10 ...)
    local http_code="${response: -3}"
    local body="${response:0:-3}"
    
    case "$http_code" in
        401) log_error "Invalid API key"; return 1 ;;
        429) log_error "Rate limit exceeded"; sleep 60; retry ;;
        5*)  log_error "API error"; return 1 ;;
    esac
}
```

**Impact:** Better user experience, easier debugging.

**Estimated Effort:** 2 hours

---

### 7. Add Debug/Verbose Mode
**Issue:** No way to see raw API requests for troubleshooting.

**Action Items:**
- [ ] Add `--verbose` or `-v` flag to all commands
- [ ] Print curl commands being executed
- [ ] Print raw API responses
- [ ] Allow API endpoint override via `SIMPLELOGIN_API_BASE` env var

**Example:**
```bash
$ simplelogin create test --verbose
[DEBUG] API_BASE: https://app.simplelogin.io/api
[DEBUG] curl -X POST -H "Authentication: ***" -d '{"alias_prefix":"test"...}'
[DEBUG] Response: {"email":"test@...", ...}
✓ Created alias: test@simplelogin.com
```

**Impact:** Easier debugging for users and maintainers.

**Estimated Effort:** 1.5 hours

---

### 8. Expand Troubleshooting Documentation
**Issue:** Only 3 error scenarios documented.

**Action Items:**
- [ ] Add sections for:
  - "API returns 401 Unauthorized"
  - "Command not found" (PATH issue)
  - "jq not installed"
  - "curl timeout"
  - "JSON parsing failed"
  - "No suffixes available" (quota limit)
  - "Rate limit exceeded"
- [ ] Add diagnostic commands section
- [ ] Add links to SimpleLogin support

**Estimated Effort:** 1 hour

---

### 9. Document Environment Variables in Metadata
**Issue:** SKILL.md mentions env vars but not in machine-readable format.

**Action Items:**
- [ ] Add `env` section to SKILL.md metadata
- [ ] Document required vs optional vars
- [ ] Include in README configuration section

```yaml
metadata:
  openclaw:
    env:
      SIMPLELOGIN_API_KEY:
        required: true
        description: "Your SimpleLogin API key"
      SIMPLELOGIN_JSON:
        required: false
        description: "Set to 'true' for JSON output"
      SIMPLELOGIN_MAILBOX_ID:
        required: false
        description: "Default mailbox ID (auto-selects first if omitted)"
      SIMPLELOGIN_API_BASE:
        required: false
        default: "https://app.simplelogin.io/api"
        description: "Override API endpoint"
```

**Estimated Effort:** 30 minutes

---

## P2 - Medium Priority

### 10. Add Agent Workflow Examples
**Issue:** Users don't see realistic OpenClaw integration patterns.

**Action Items:**
- [ ] Add examples section to SKILL.md:
  - Create alias when signing up for service
  - Rotate compromised alias
  - Bulk disable for cleanup
  - JSON parsing in scripts
- [ ] Create `examples/` directory with sample scripts
- [ ] Document best practices for agent use

**Estimated Effort:** 2 hours

---

### 11. Add Installation Script
**Issue:** Manual installation is tedious.

**Action Items:**
- [ ] Create `install.sh`:
  ```bash
  #!/bin/bash
  set -e
  SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
  BIN_DIR="${BIN_DIR:-/usr/local/bin}"
  
  cp "$SCRIPT_DIR/scripts/simplelogin" "$BIN_DIR/simplelogin"
  chmod +x "$BIN_DIR/simplelogin"
  
  echo "✓ Installed simplelogin to $BIN_DIR"
  echo "Next steps:"
  echo "  1. Set SIMPLELOGIN_API_KEY environment variable"
  echo "  2. Run 'simplelogin auth-test' to verify"
  ```
- [ ] Make executable
- [ ] Add to installation docs
- [ ] Reference in .clawhub/skill.json postinstall

**Estimated Effort:** 1 hour

---

### 12. Add Search/Filter Commands
**Issue:** Can't quickly find specific aliases.

**Action Items:**
- [ ] Add `simplelogin search <pattern>` - filter by name/note
- [ ] Add `simplelogin status` - show counts (enabled/disabled)
- [ ] Add `simplelogin list --domain <domain>` (already exists, test it)
- [ ] Improve list output formatting

**Estimated Effort:** 1.5 hours

---

### 13. Add File Permission Documentation
**Issue:** Secrets file security not mentioned.

**Action Items:**
- [ ] Document `$HOME/.openclaw/secrets/simplelogin.json` approach in SKILL.md
- [ ] Add note about file permissions:
  ```bash
  chmod 600 ~/.openclaw/secrets/simplelogin.json
  ```
- [ ] Mention alternative: use Warden/Bitwarden instead

**Estimated Effort:** 30 minutes

---

### 14. Create Integration Tests
**Issue:** No automated testing for skill functionality.

**Action Items:**
- [ ] Create `tests/` directory
- [ ] Add test script that:
  - Validates script is executable
  - Tests help commands
  - Tests API key detection
  - Requires test SimpleLogin account or mocks
- [ ] Add to CI/CD pipeline

**Estimated Effort:** 3 hours

---

## P3 - Low Priority

### 15. Add Bulk Operations
**Features:**
- [ ] `simplelogin disable-all --pattern "<glob>"` - Bulk disable
- [ ] `simplelogin export` - Export all aliases to JSON
- [ ] `simplelogin import` - Import aliases from JSON (create in batch)

**Estimated Effort:** 3 hours

---

### 16. Add Configuration File Support
**Feature:** Alternative to environment variables.

**Spec:**
```json
// ~/.config/simplelogin/config.json
{
  "api_key_env": "SIMPLELOGIN_API_KEY",
  "default_mailbox": 12345,
  "default_domain": "simplelogin.com",
  "json_mode": false,
  "verbose": false
}
```

**Action Items:**
- [ ] Add config file loader
- [ ] Config overrides env vars (or vice versa?)
- [ ] Add `simplelogin config` command to show current config

**Estimated Effort:** 2-3 hours

---

### 17. Add Statistics Command
**Feature:** Usage overview for power users.

```bash
$ simplelogin stats
Total aliases: 45
  Enabled: 42
  Disabled: 3
By domain:
  simplelogin.com: 30
  alias.com: 15
Created this month: 5
```

**Estimated Effort:** 1.5 hours

---

### 18. Add Interactive Mode
**Feature:** Menu-driven interface for casual users.

```bash
$ simplelogin interactive
SimpleLogin CLI - Interactive Mode

1. Create custom alias
2. Create random alias
3. List aliases
4. Disable alias
5. Enable alias
6. Delete alias
7. Create contact
8. Exit

Choose command: 1
Enter prefix: amazon
Creating alias...
✓ Created: amazon@simplelogin.com
```

**Estimated Effort:** 2-3 hours

---

### 19. Add Note Management Commands
**Features:**
- [ ] `simplelogin note-set <alias> <note>` - Set/update note
- [ ] `simplelogin note-get <alias>` - Show note
- [ ] `simplelogin note-clear <alias>` - Remove note

**Estimated Effort:** 1.5 hours

---

### 20. Add API Key Rotation Documentation
**Feature:** Guide users on key rotation.

**Action Items:**
- [ ] Add section to SKILL.md:
  - How to rotate keys in SimpleLogin dashboard
  - How to update environment variable or secrets file
  - Security best practices (rotate every 90 days?)

**Estimated Effort:** 30 minutes

---

## P4 - Future Consideration

### 21. Add Audit Logging
**Feature:** Log actions for compliance/security.

**Spec:**
```bash
# With SIMPLELOGIN_LOG_FILE=/var/log/simplelogin.log
simplelogin create shopping
# → Logs: 2026-04-08T10:30:00Z CREATE shopping@simplelogin.com (user: root, pid: 1234)
```

**Estimated Effort:** 2 hours

---

### 22. Add Webhook Support
**Feature:** Notify external services on alias events.

**Spec:**
```bash
# SIMPLELOGIN_WEBHOOK_URL=https://my-service.com/hook
# POST on create/delete/disable
{
  "event": "alias.created",
  "alias": "shopping@simplelogin.com",
  "timestamp": "2026-04-08T10:30:00Z"
}
```

**Estimated Effort:** 2-3 hours

---

### 23. Add Multi-Account Support
**Feature:** Manage multiple SimpleLogin accounts.

**Commands:**
- `simplelogin account-add <name>`
- `simplelogin account-select <name>`
- `simplelogin account-list`
- `simplelogin account-remove <name>`

**Estimated Effort:** 4 hours

---

### 24. Add Alias Import/Export
**Feature:** Backup and migration support.

**Commands:**
- `simplelogin export > backup.json`
- `simplelogin import < backup.json`

**Estimated Effort:** 2 hours

---

### 25. Add Shell Completion
**Feature:** Tab completion for bash/zsh.

**Action Items:**
- [ ] Create completion script
- [ ] Document installation
- [ ] Test with bash and zsh

**Estimated Effort:** 2 hours

---

## Summary by Priority

| Priority | Count | Total Effort |
|----------|-------|--------------|
| P0 | 4 items | ~4 hours |
| P1 | 5 items | ~6 hours |
| P2 | 5 items | ~8 hours |
| P3 | 6 items | ~10 hours |
| P4 | 5 items | ~12 hours |

**Total Backlog:** 25 items, ~40 hours estimated

---

## Recommended Next Sprint (P0 + P1)

**Total Effort:** ~10 hours

1. Consolidate scripts (3h)
2. Fix installation docs (0.5h)
3. Enhance .clawhub/skill.json (0.5h)
4. Add auth-test command (1h)
5. Port contact commands (1h)
6. Add error handling (2h)
7. Add debug mode (1.5h)
8. Expand troubleshooting docs (1h)
9. Document env vars (0.5h)

**Deliverable:** Production-ready v1.0 for ClawHub publishing

---

**Generated by:** OpenClaw Subagent  
**Session:** simplelogin-cli QA: OpenClaw perspective  
**Date:** 2026-04-08
