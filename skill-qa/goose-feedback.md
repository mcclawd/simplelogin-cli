# SimpleLogin CLI - Goose/Open Source Community Review

**Reviewed:** 2026-04-08  
**Reviewer Perspective:** Goose/open source community values (extensibility, modularity, transparency, community contribution)

---

## Executive Summary

This is a **solid, privacy-focused CLI tool** with a clear value proposition. The codebase demonstrates strong open source values with an MIT license, clean bash implementation, and straightforward API integration. However, there are opportunities to improve documentation completeness, error handling, and community contribution pathways.

**Overall Impression:** Good foundation, production-ready for personal use, needs polish for broader community adoption.

---

## 1. Documentation Clarity and Completeness

### ✅ Strengths

- **SKILL.md is comprehensive** - Clear feature list, good usage examples, covers main commands well
- **README.md provides good overview** - Explains what SimpleLogin is, good for newcomers
- **Agent/JSON mode documented** - Important for automation and integration use cases
- **Security notes included** - Explicitly mentions API key handling best practices
- **Troubleshooting section** - Addresses common issues (API key, suffixes, spam)

### ❌ Gaps & Issues

#### Critical

1. **No installation verification** - No "verify your installation" section with a simple test command
2. **Missing API key setup walkthrough** - Says "generate in dashboard" but doesn't link to the exact URL or show screenshots
3. **No quickstart for impatient users** - No "5-minute setup" guide with copy-paste commands

#### Important

4. **Duplicate content between SKILL.md and README.md** - They're nearly identical. Consider:
   - README.md: Installation, quickstart, basic usage
   - SKILL.md: Technical details, API reference, contributing guidelines

5. **Missing command reference completeness**:
   - `delete` command documented in scripts/simplelogin but NOT in SKILL.md or README.md usage sections
   - Contact commands (`contact-create`, `contact-list`) appear in SKILL.md but the scripts/simplelogin doesn't show these commands in main dispatcher (version mismatch?)

6. **No examples of real-world workflows** - Missing:
   - "I want to sign up for newsletters without my real email" walkthrough
   - "I need to reply to an email anonymously" workflow
   - Integration examples with email clients (mutt, neomutt, etc.)

#### Minor

7. **No badges** - Missing common open source badges:
   - License badge
   - Version badge
   - Maybe build status if CI is added

8. **No changelog** - No CHANGELOG.md or version history

### 📝 Recommendations

- [ ] Add "Quick Start (5 min)" section to README.md
- [ ] Add direct link to SimpleLogin API key generation page
- [ ] Add installation verification step (`simplelogin --help` or `simplelogin list`)
- [ ] Document the `delete` command in SKILL.md
- [ ] Resolve version mismatch between root `simplelogin` and `scripts/simplelogin`
- [ ] Add CHANGELOG.md with semantic versioning
- [ ] Add shields.io badges to README.md

---

## 2. Ease of Installation and Configuration

### ✅ Strengths

- **ClawHub integration** - Ready for `clawhub install simplelogin-cli`
- **Multiple configuration options** - Environment variable or credential file
- **Minimal dependencies** - Only requires `curl` and `jq` (common tools)
- **Credential manager fallback** - Falls back to `~/.openclaw/secrets/simplelogin.json`

### ❌ Issues

#### Critical

1. **No manual installation instructions** - "Add scripts/ to your PATH" is too vague:
   - Which script file? (`scripts/simplelogin` or root `simplelogin`?)
   - How to add to PATH? (export command, .bashrc edit?)
   - What about making it executable? (`chmod +x`)

2. **File structure confusion** - There are TWO `simplelogin` scripts:
   - `/skills/simplelogin-cli/simplelogin` (6461 bytes)
   - `/skills/simplelogin-cli/scripts/simplelogin` (13285 bytes, more complete)
   - **Which one is canonical?** This is confusing for users and maintainers

#### Important

3. **No package.json completeness** - `package.json` only has name, version, description. Missing:
   - `bin` field for npm-style installation
   - `scripts` for development
   - `repository` URL
   - `author` field
   - `bugs` URL

4. **No installation testing** - No CI/CD badge, no installation test script

5. **Credential fallback is hardcoded** - Only checks `~/.openclaw/secrets/simplelogin.json`. What about:
   - Standard password manager CLIs (pass, bw, op)?
   - Other common locations?

### 📝 Recommendations

- [ ] **CRITICAL: Consolidate to single canonical script** - Either keep root `simplelogin` or `scripts/simplelogin`, not both
- [ ] Add detailed manual installation:
  ```bash
  git clone https://github.com/mcclawd/simplelogin-cli.git
  cd simplelogin-cli
  chmod +x simplelogin
  ln -s $(pwd)/simplelogin /usr/local/bin/simplelogin
  ```
- [ ] Update `package.json` with proper npm fields for global installation
- [ ] Add installation verification script/test
- [ ] Support multiple credential manager CLIs (optional enhancement)

---

## 3. Security Considerations

### ✅ Strengths

- **API key not hardcoded** - Never stored in the script itself ✓
- **Environment variable support** - Standard practice ✓
- **MIT license** - Permissive, community-friendly ✓
- **Open source** - Code is auditable ✓
- **Uses HTTPS** - API calls go to `https://app.simplelogin.io/api` ✓

### ⚠️ Concerns

#### Important

1. **API key exposure in process list** - When passed via environment, it's visible in `/proc/[pid]/environ`. Consider:
   - Reading from stdin
   - Using a file with restricted permissions (600)

2. **No API key validation** - Doesn't test if the API key is valid before use. Should:
   - Make a test API call on first use
   - Provide clear error message if key is invalid/expired

3. **Credential file permissions not checked** - `~/.openclaw/secrets/simplelogin.json` - what if it's world-readable?

4. **curl without certificate verification option** - While good that it doesn't disable cert checking, there's no option for users in corporate environments with MITM proxies

5. **No rate limiting awareness** - Doesn't mention SimpleLogin API rate limits or handle 429 responses gracefully

#### Minor

6. **Error messages leak API structure** - Some error messages expose API endpoints and structure. Not critical but worth noting for security-conscious users

7. **No audit logging** - No log of what actions were performed when. For a privacy tool, some users may want audit trails

### 📝 Recommendations

- [ ] Add API key validation on first use with helpful error message
- [ ] Add file permission check for credential files (warn if >600)
- [ ] Add rate limit handling (catch 429, show retry-after)
- [ ] Document security considerations for users (threat model)
- [ ] Consider adding `--insecure` flag for corporate proxy users (with warning)
- [ ] Add optional audit logging (`SIMPLELOGIN_LOG=/path/to/log`)

---

## 4. Error Handling and Troubleshooting

### ✅ Strengths

- **Uses `set -euo pipefail`** in scripts/simplelogin - Good bash practices ✓
- **JSON mode for programmatic error handling** - Can parse errors in automation ✓
- **Helper functions for logging** - `log_error`, `log_success`, etc. ✓
- **Some validation** - Checks for required arguments in contact-create ✓

### ❌ Issues

#### Critical

1. **Inconsistent error handling between scripts** - Root `simplelogin` uses `set -e`, but error handling varies:
   - Some commands exit with generic error codes
   - No consistent error message format

2. **No API error response parsing** - When API returns error, just dumps raw JSON:
   ```bash
   # Current behavior
   {"error": "Invalid API key", "status": 401}
   
   # Should be
   ✗ Error: Invalid API key
   ```

3. **Silent failures** - Several commands use `> /dev/null` which hides errors:
   ```bash
   curl -s -X POST ... > /dev/null
   ```
   What if the POST failed?

#### Important

4. **No validation of API responses** - Doesn't check if API calls succeeded:
   - What if SimpleLogin API is down?
   - What if the response is malformed?
   - No JSON validation after curl

5. **Missing validation for user input**:
   - No email format validation
   - No hostname validation for `--for` option
   - No sanitization of notes (could inject JSON if not careful)

6. **No timeout handling** - curl has no timeout, could hang indefinitely

7. **Missing error recovery** - If alias creation fails mid-way, no cleanup or rollback

### 📝 Recommendations

- [ ] Parse API error responses and show user-friendly messages
- [ ] Remove `> /dev/null` or check exit codes after
- [ ] Add API response validation (check for expected fields)
- [ ] Add input validation (email format, hostname format)
- [ ] Add curl timeout (`--max-time 30`)
- [ ] Add network connectivity check before API calls
- [ ] Implement consistent error exit codes:
  - `1` - Invalid arguments
  - `2` - Configuration error (API key)
  - `3` - Network error
  - `4` - API error
  - `5` - Internal error

---

## 5. Extensibility and Modularity

### ✅ Strengths

- **Modular command structure** - Each command is a function ✓
- **Helper functions** - `api_call`, `get_api_key`, logging functions ✓
- **JSON mode for integration** - Easy to integrate with other tools ✓
- **Bash-based** - Easy to modify, no compilation needed ✓
- **Environment variable configuration** - Easy to customize behavior ✓

### ❌ Limitations

#### Important

1. **No plugin/hook system** - Can't extend behavior without modifying source:
   - No pre/post command hooks
   - No custom command registration
   - No event system for alias creation, etc.

2. **Tightly coupled to SimpleLogin API** - Hard to adapt for other alias services:
   - API URLs hardcoded
   - Response parsing specific to SimpleLogin
   - No abstraction layer for API provider

3. **No configuration file support** - Everything is environment variables:
   - No `~/.simpleloginrc` or similar
   - Can't set defaults (e.g., default domain, default note format)
   - Environment variables can be cumbersome for many options

4. **Limited output formatting options** - Plain text or JSON only:
   - No table formatting for list command
   - No CSV export
   - No custom templates

5. **No testing framework** - Hard to verify changes don't break existing functionality:
   - No unit tests
   - No integration tests
   - No test fixtures/mocks

#### Minor

6. **No shell completion** - No bash/zsh completion scripts

7. **No man page** - No `man simplelogin` support

8. **Single file monolith** - `scripts/simplelogin` is 478 lines. Could be split into:
   - Core functions
   - Command implementations
   - API client

### 📝 Recommendations

- [ ] Add configuration file support (`~/.simplelogin/config` or similar)
- [ ] Create abstraction layer for API provider (enable AnonAddy, etc.)
- [ ] Add pre/post command hooks for extensibility
- [ ] Add output format options (`--format=table|json|csv`)
- [ ] Implement basic test suite (even if just smoke tests)
- [ ] Add bash/zsh completion scripts
- [ ] Consider splitting into modules for maintainability
- [ ] Create example plugins/hooks directory

---

## 6. Missing Features & Improvements

### High Priority

1. **Delete command** - Present in scripts/simplelogin but not documented or in root script
2. **Search/filter aliases** - Can only list, can't search by note, domain, status
3. **Alias statistics** - How many emails received, last activity, etc.
4. **Batch operations** - Enable/disable/delete multiple aliases at once
5. **Import/export** - Backup aliases, migrate between accounts

### Medium Priority

6. **Mailbox management** - List mailboxes, switch default
7. **Domain management** - List custom domains, create aliases on specific domains
8. **Activity logs** - Show recent email activity for aliases
9. **Premium features** - Support for premium SimpleLogin features if available
10. **Webhook support** - Notify on new alias creation, email received, etc.

### Nice to Have

11. **Interactive mode** - TUI for browsing/managing aliases
12. **Email preview** - Show subject lines of recent forwarded emails
13. **Temporary aliases** - Auto-delete after time period
14. **Alias sharing** - Share aliases with family/team members
15. **Integration with email clients** - Plugins for mutt, neomutt, thunderbird

### Community Features

16. **Shareable configurations** - Import configs from community
17. **Plugin marketplace** - Community-contributed extensions
18. **Multi-account support** - Manage multiple SimpleLogin accounts

---

## Open Source Values Assessment

### ✅ Strong Open Source Values

- **MIT License** - Permissive, encourages adoption and modification ✓
- **Transparent codebase** - Clear, readable bash scripts ✓
- **No telemetry** - No tracking, analytics, or phone-home ✓
- **Privacy-focused** - Aligns with FOSS privacy values ✓
- **No vendor lock-in** - Works with SimpleLogin but could be adapted ✓

### ⚠️ Areas for Improvement

- **Contribution guidelines** - No CONTRIBUTING.md
- **Issue templates** - No GitHub issue templates
- **Pull request template** - No PR template
- **Code of Conduct** - No CODE_OF_CONDUCT.md
- **Roadmap** - No public roadmap or feature planning
- **Release process** - No clear versioning/release process

---

## Community Adoption Readiness

**Current State:** 🔶 **Personal/Small Team Ready**

The tool works well for individual users or small teams familiar with CLI tools. However, it needs polish for broader community adoption.

**To Reach Community Ready:**

1. Fix the duplicate script confusion (CRITICAL)
2. Add CONTRIBUTING.md with clear guidelines
3. Implement consistent error handling
4. Add test suite (even minimal)
5. Create issue/PR templates
6. Establish release/versioning process
7. Add Code of Conduct

---

## Comparison to Similar Projects

### AnonAddy CLI

- **AnonAddy:** More mature, has official CLI, active community
- **SimpleLogin CLI:** More privacy-focused brand, better documentation (currently)

**Competitive Advantage:** Simpler, more focused on core use cases

### Email Self-Defense Tools

- Most are GUI-based or email client plugins
- This CLI fills a gap for terminal users and automation

**Niche:** CLI-first users, automation/agent integration, bash scripting

---

## Final Verdict

### Scores (detailed in goose-scores.json)

- **Documentation:** 7.5/10 - Good but incomplete
- **Installation:** 6/10 - Confusing file structure hurts this
- **Security:** 7/10 - Good practices, needs validation
- **Error Handling:** 5/10 - Needs significant work
- **Extensibility:** 6.5/10 - Modular but no plugin system
- **Open Source Values:** 8/10 - Strong foundation, missing community docs

### Overall: 6.7/10 - Promising, Needs Polish

**Would I recommend this to the Goose community?** 
- **For personal use:** Yes, with caveats about the duplicate scripts
- **For production/team use:** Not yet, needs better error handling and testing
- **As a contribution target:** Yes! Great candidate for community improvements

---

## Next Steps for Maintainers

1. **CRITICAL:** Consolidate to single canonical script
2. **HIGH:** Fix error handling and API response parsing
3. **HIGH:** Add installation verification and testing
4. **MEDIUM:** Add CONTRIBUTING.md and community docs
5. **MEDIUM:** Implement configuration file support
6. **LOW:** Add plugins/hooks system

---

*Review conducted from Goose/open source community perspective, focusing on extensibility, community contribution potential, and open source values.*
