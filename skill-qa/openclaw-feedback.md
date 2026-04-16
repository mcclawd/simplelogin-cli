# SimpleLogin CLI - OpenClaw User Feedback

**Review Date:** 2026-04-08  
**Reviewer:** OpenClaw Subagent (OpenClaw/ClawHub user perspective)  
**Skill Location:** `/root/.openclaw/workspace/skills/simplelogin-cli/`

---

## Executive Summary

A **well-structured, practical skill** that addresses a genuine privacy need. Documentation is clear and comprehensive, with good examples for both human and agent use. However, there are several gaps that affect production usability: **missing contact management commands in main script**, **no installation guidance for the binary**, **dual-script confusion**, and **limited error recovery**.

**Overall Assessment:** Promising foundation (⭐⭐⭐⭐☆) but needs polishing for seamless OpenClaw integration.

---

## 1. Documentation Clarity & Completeness

### ✅ Strengths

- **SKILL.md is comprehensive** - Well-organized feature list with clear emoji indicators
- **Agent/JSON mode documented** - Explicitly calls out programmatic use with examples
- **Good usage examples** - Covers common workflows (create, random, list, disable, enable)
- **Contact/reverse alias explanation** - Excellent section explaining the use case for replying to forwarded emails
- **Security section present** - Acknowledges API key handling best practices
- **Troubleshooting section** - Covers common issues (API key, suffixes, spam)

### ⚠️ Issues

#### 1.1 Dual-Script Confusion
**Critical:** There are **two different versions** of the `simplelogin` script:
- `/root/.openclaw/workspace/skills/simplelogin-cli/simplelogin` (main, ~270 lines)
- `/root/.openclaw/workspace/skills/simplelogin-cli/scripts/simplelogin` (duplicate, ~640 lines)

The scripts have **different command sets**:
- **Main script:** Has `contact-create` and `contact-list` commands
- **Scripts/ version:** Missing `contact-create` and `contact-list` entirely

**Impact:** Users installing via ClawHub won't know which version they're getting. The scripts/ version appears newer (has better JSON mode handling, colors, better error messages) but lacks contact management.

**Recommendation:**
- Consolidate into ONE canonical script
- Move to `scripts/simplelogin` and update package.json/bin or .clawhub config
- Remove duplicate at root level

#### 1.2 Installation Instructions Incomplete
README.md says:
```bash
# Or clone manually
git clone https://github.com/yourusername/simplelogin-cli.git
cd simplelogin-cli
# Add scripts/ to your PATH
```

**Issues:**
- Generic GitHub URL (`yourusername`) - not a real repo
- "Add scripts/ to your PATH" is vague - which file? How?
- No mention of making the script executable (`chmod +x`)
- For ClawHub: no explanation of how ClawHub installs it (does it symlink? copy to /usr/local/bin?)

**Recommendation:**
```bash
# Manual installation
git clone https://github.com/mcclawd/simplelogin-cli.git
cd simplelogin-cli
chmod +x scripts/simplelogin
sudo ln -s $(pwd)/scripts/simplelogin /usr/local/bin/simplelogin
```

#### 1.3 Missing .clawhub/skill.json Details
The `.clawhub/skill.json` only has:
```json
{
  "name": "simplelogin-cli",
  "version": "1.0.0"
}
```

**Missing fields:**
- `main` or `bin` - which executable to install?
- `scripts` or `files` - what gets installed?
- `postinstall` - setup instructions?
- `description` - for search/discovery

#### 1.4 README vs SKILL.md Inconsistencies
- README mentions "MIT License - See [LICENSE](LICENSE)" but uses markdown link syntax that may not render in all contexts
- SKILL.md has more detailed contact/reverse alias docs than README
- README features list is shorter (missing contact management)

---

## 2. Ease of Installation & Configuration

### ✅ Strengths
- Clear environment variable setup (`SIMPLELOGIN_API_KEY`)
- Bitwarden integration mentioned (smart for OpenClaw users with Warden)
- Minimal dependencies: just `curl` and `jq`

### ⚠️ Issues

#### 2.1 No Installation Script
There's no `install.sh` or setup script. Users must:
1. Clone repo
2. Figure out which script to use
3. Make it executable
4. Add to PATH manually

**ClawHub Context:** ClawHub skills typically auto-install, but the `.clawhub/skill.json` doesn't specify how.

**Recommendation:** Add an `install.sh`:
```bash
#!/bin/bash
# install.sh - Install simplelogin CLI
set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
BIN_DIR="${BIN_DIR:-/usr/local/bin}"

cp "$SCRIPT_DIR/scripts/simplelogin" "$BIN_DIR/simplelogin"
chmod +x "$BIN_DIR/simplelogin"

echo "✓ Installed simplelogin to $BIN_DIR/simplelogin"
echo "Next: Set SIMPLELOGIN_API_KEY or configure credential manager"
```

#### 2.2 Credential Setup Not Automated
The skill mentions Bitwarden/Warden but doesn't provide:
- A setup wizard
- Validation that the API key works
- A test command

**Recommendation:** Add `simplelogin auth-test`:
```bash
simplelogin auth-test
# → Validates API key by fetching mailbox list
# → "✓ API key valid, found 2 mailboxes"
# → or "✗ Invalid API key"
```

#### 2.3 PATH Configuration Unclear
For OpenClaw agents: Will the skill be in PATH automatically? How do agents invoke it?
- As `simplelogin` (if in PATH)?
- As full path `/root/.openclaw/workspace/skills/simplelogin-cli/scripts/simplelogin`?
- Via skill loader mechanism?

**Recommendation:** Document in SKILL.md:
```markdown
## For OpenClaw Agents

The skill installs to your PATH. Use in agent scripts:

```bash
# Direct invocation
simplelogin create newsletter --note "Agent test"

# In JSON mode for parsing
export SIMPLELOGIN_JSON=true
result=$(simplelogin create test)
email=$(echo "$result" | jq -r '.email')
```
```

---

## 3. Security Considerations

### ✅ Strengths
- **API key never hardcoded** - Good practice
- **Environment variable support** - Standard secure pattern
- **Credential manager integration** - Mentions Bitwarden/Warden
- **Open source** - Links to SimpleLogin's audited code

### ⚠️ Issues

#### 3.1 Alternative Credential Storage
The root `simplelogin` script has hardcoded fallback:
```bash
if [ -z "$API_KEY" ] && [ -f "$HOME/.openclaw/secrets/simplelogin.json" ]; then
    API_KEY=$(jq -r '.password // .api_key // empty' "$HOME/.openclaw/secrets/simplelogin.json" 2>/dev/null)
fi
```

**Issues:**
- Only checks `$HOME/.openclaw/secrets/` - what if workspace is elsewhere?
- No encryption mentioned for the secrets file
- No guidance on file permissions (should be `chmod 600`)

**Recommendation:** 
- Document this option explicitly in SKILL.md
- Add note about file permissions:
  ```bash
  chmod 600 ~/.openclaw/secrets/simplelogin.json
  ```

#### 3.2 No API Key Rotation Guidance
Users should know:
- How to rotate API keys in SimpleLogin dashboard
- That old keys should be invalidated
- Where to update the key in their setup

#### 3.3 No Audit Logging
For automation use, it would be valuable to log:
- When aliases are created
- By which agent/script
- For what purpose

**Optional Enhancement:** Add `SIMPLELOGIN_LOG_FILE` env var for audit trail.

---

## 4. Error Handling & Troubleshooting

### ✅ Strengths
- Checks for API key presence
- Validates alias exists before disable/enable/delete
- JSON mode for programmatic error parsing

### ⚠️ Issues

#### 4.1 Inconsistent Error Handling Between Scripts
The `scripts/simplelogin` has better error handling:
- Colored output
- Structured JSON errors when in JSON mode
- More detailed messages

The root `simplelogin` has simpler, less informative errors.

#### 4.2 Missing Error Cases
Neither script handles:
- **Rate limiting** (HTTP 429) - SimpleLogin API may throttle
- **Network failures** - No retry logic
- **API endpoint changes** - Hardcoded `API_BASE` with no override
- **Invalid JSON responses** - What if SimpleLogin returns HTML error page?
- **Quota limits** - Free tier has limited aliases

**Recommendation:**
```bash
# Add to api_call function
handle_response() {
    local response="$1"
    local http_code=$(echo "$response" | jq -r '.status_code // 200')
    
    if [[ "$http_code" == "429" ]]; then
        log_error "Rate limit exceeded. Retry after 60 seconds."
        return 1
    elif [[ "$http_code" == "401" ]]; then
        log_error "Invalid API key. Check your credentials."
        return 1
    elif [[ "$http_code" == "403" ]]; then
        log_error "Permission denied. Check your SimpleLogin plan."
        return 1
    fi
}
```

#### 4.3 No Debug Mode
For troubleshooting API issues, users need:
- Verbose output (show raw API requests/responses)
- Ability to override API endpoint (for testing)
- Verbose flag: `simplelogin list --verbose`

#### 4.4 Troubleshooting Section Too Brief
Current docs only cover 3 issues. Expand to include:
- "API returns 401 Unauthorized"
- "Commands not found" (PATH issue)
- "jq not installed"
- "curl timeout"
- JSON parsing failures

---

## 5. Integration Capabilities with OpenClaw

### ✅ Strengths
- **JSON mode** - Perfect for agent scripting
- **Warden integration** - Works with OpenClaw's credential system
- **Non-interactive** - Good for automation
- **Bash-based** - Easy to call from other skills/scripts

### ⚠️ Issues

#### 5.1 No OpenClaw Skill Hooks
Skills in OpenClaw can define:
- Pre-install scripts
- Post-install configuration
- Environment setup
- Health checks

The `.clawhub/skill.json` doesn't leverage these capabilities.

#### 5.2 Missing `env` Field in Metadata
SKILL.md mentions env vars but `.clawhub/skill.json` doesn't document:
- `SIMPLELOGIN_API_KEY` (required)
- `SIMPLELOGIN_JSON` (optional)
- `SIMPLELOGIN_MAILBOX_ID` (optional)

**Recommendation:** Add to SKILL.md metadata:
```yaml
metadata:
  openclaw:
    env:
      SIMPLELOGIN_API_KEY: "Required - Your SimpleLogin API key"
      SIMPLELOGIN_JSON: "Optional - Set to 'true' for JSON output"
      SIMPLELOGIN_MAILBOX_ID: "Optional - Default mailbox ID"
```

#### 5.3 No Agent Example Workflows
For OpenClaw users, show realistic agent use cases:

```bash
# Example: Create alias when signing up for a service
signup_for_service() {
    local service="$1"
    local alias_email=$(simplelogin create "$service" --note "Auto-created by agent" | jq -r '.email')
    
    # Use alias_email for signup...
    echo "Created alias for $service: $alias_email"
}

# Example: Rotate compromised alias
rotate_alias() {
    local old_alias="$1"
    local new_alias=$(simplelogin create "${old_alias%@*}-new" | jq -r '.email')
    simplelogin disable "$old_alias"
    echo "Rotated $old_alias → $new_alias"
}
```

#### 5.4 Missing Integration Tests
For CI/CD or skill validation:
- Test API key validity
- Test alias creation/deletion (with test account)
- Test JSON mode output parsing

---

## 6. Missing Features & Improvements

### High Priority

#### 6.1 Contact Management Commands (scripts/ version missing these)
The root script has `contact-create` and `contact-list`, but scripts/ doesn't. This is **critical** for email workflows.

**Action:** Port contact commands from root → scripts/ version

#### 6.2 Unified Script
Eliminate the duplicate. Keep the better-structured version (scripts/ has better architecture) and add missing features.

#### 6.3 Alias Search/Filter
```bash
simplelogin search amazon   # Find aliases containing "amazon"
simplelogin status          # Show enabled/disabled count
```

#### 6.4 Bulk Operations
```bash
simplelogin disable-all --pattern "*temp*"   # Disable matching aliases
simplelogin export                            # Export all aliases to JSON
```

### Medium Priority

#### 6.5 Configuration File
Instead of only env vars:
```bash
# ~/.config/simplelogin/config.json
{
  "api_key_env": "SIMPLELOGIN_API_KEY",
  "default_mailbox": 12345,
  "default_domain": "simplelogin.com",
  "json_mode": false
}
```

#### 6.6 Interactive Mode
For human users:
```bash
simplelogin interactive
# → Menu-driven interface for common operations
```

#### 6.7 Alias Notes Management
```bash
simplelogin note-set shopping@... "Amazon Prime"
simplelogin note-get shopping@...
simplelogin note-clear shopping@...
```

#### 6.8 Statistics
```bash
simplelogin stats
# → Total aliases: 45
# → Enabled: 42, Disabled: 3
# → By domain: simplelogin.com (30), alias.com (15)
```

### Low Priority (Nice-to-Have)

#### 6.9 Webhook Support
Notify external services when alias created/disabled (for automation workflows)

#### 6.10 Multi-Account Support
```bash
simplelogin account-add personal
simplelogin account-add work
simplelogin account-select personal
```

#### 6.11 Alias Import/Export
For backup or migration

---

## Recommendations Summary

### Immediate (Before Publishing)
1. **Consolidate scripts** - Pick one canonical version, remove duplicate
2. **Add contact commands to scripts/ version** - Essential for email workflows
3. **Fix installation docs** - Real GitHub URL, clear PATH setup
4. **Enhance .clawhub/skill.json** - Add bin, scripts, description fields
5. **Add auth-test command** - Validate API key on setup

### Short-Term (Post-Publish V1.1)
6. **Add error handling improvements** - Rate limits, retries, debug mode
7. **Expand troubleshooting docs** - Common error patterns
8. **Add agent workflow examples** - Show realistic OpenClaw integrations
9. **Add environment metadata** - Document required/optional env vars in SKILL.md

### Long-Term (V2.0)
10. **Add bulk operations** - disable-all, export
11. **Add configuration file support** - Alternative to env vars
12. **Add statistics command** - Useful for power users
13. **Add audit logging** - For compliance/security tracking

---

## Final Verdict

**Overall Score: 4.0/5.0**

| Category | Score | Notes |
|----------|-------|-------|
| Documentation | 4.5/5 | Clear, comprehensive, but has inconsistencies |
| Installation | 3.5/5 | Works but rough edges, no install script |
| Security | 4.0/5 | Good practices, could add audit logging |
| Error Handling | 3.5/5 | Basic present, needs robustness |
| Integration | 4.5/5 | JSON mode + Warden integration excellent |
| Completeness | 4.0/5 | Core features present, some gaps |

**Recommendation:** Ready for OpenClaw use with minor fixes (script consolidation, installation docs). The dual-script issue is the most critical blocker.

**For OpenClaw Users:** This skill is valuable for privacy-focused automation. Recommended for agents that sign up for services, manage subscriptions, or handle email workflows. Just be aware of the script duplication and pick the right one.

---

**Reviewed by:** OpenClaw Subagent  
**Session:** simplelogin-cli QA: OpenClaw perspective  
**Date:** 2026-04-08
