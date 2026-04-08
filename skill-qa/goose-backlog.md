# SimpleLogin CLI - Prioritized Backlog

**Generated:** 2026-04-08  
**Perspective:** Goose/Open Source Community Review  
**Format:** MoSCoW (Must, Should, Could, Won't) + Effort/Impact matrix

---

## Priority Legend

- 🔴 **P0 - Critical** (Must have - blocks adoption)
- 🟠 **P1 - High** (Should have - significantly impacts usability)
- 🟡 **P2 - Medium** (Could have - nice to have)
- 🟢 **P3 - Low** (Nice to have - low priority)

**Effort:** S/M/L/XL (Story points approximation)  
**Impact:** Low/Medium/High/Very High

---

## P0 - Critical (Must Have)

### 1. Consolidate Duplicate Scripts
**ID:** BACKLOG-001  
**Priority:** 🔴 P0  
**Effort:** S  
**Impact:** Very High  

**Problem:** Two `simplelogin` scripts exist (root and scripts/), causing confusion about which is canonical.

**Tasks:**
- [ ] Audit differences between root `simplelogin` and `scripts/simplelogin`
- [ ] Choose canonical version (recommend scripts/simplelogin - more complete)
- [ ] Remove or deprecate non-canonical version
- [ ] Update all references in documentation
- [ ] Update ClawHub configuration if needed

**Rationale:** This critically impacts installation experience and maintenance. Blocking P0.

---

### 2. Fix Error Handling & API Response Parsing
**ID:** BACKLOG-002  
**Priority:** 🔴 P0  
**Effort:** M  
**Impact:** Very High  

**Problem:** API errors are dumped as raw JSON, silent failures with `> /dev/null`, no validation.

**Tasks:**
- [ ] Parse API error responses (401, 429, 500, etc.)
- [ ] Show user-friendly error messages
- [ ] Remove `> /dev/null` or add error checking after
- [ ] Validate API responses (check for expected fields)
- [ ] Add API key validation on first use
- [ ] Implement consistent exit codes (1-5)
- [ ] Add curl timeout (`--max-time 30`)

**Acceptance Criteria:**
- User sees "Invalid API key" not `{"error": "...", "status": 401}`
- All failed API calls result in clear error messages
- Exit codes are documented and consistent

**Rationale:** Poor error handling makes debugging impossible for users.

---

### 3. Document Missing Commands
**ID:** BACKLOG-003  
**Priority:** 🔴 P0  
**Effort:** XS  
**Impact:** High  

**Problem:** `delete` command exists in scripts/simplelogin but not documented in SKILL.md or README.md.

**Tasks:**
- [ ] Add `delete` command to SKILL.md usage section
- [ ] Add `delete` command to README.md examples
- [ ] Add `--help` text for `delete` command
- [ ] Verify all commands in scripts/simplelogin are documented

**Rationale:** Undocumented features are broken features from user perspective.

---

## P1 - High (Should Have)

### 4. Create CONTRIBUTING.md
**ID:** BACKLOG-004  
**Priority:** 🟠 P1  
**Effort:** S  
**Impact:** High  

**Problem:** No contribution guidelines, issue templates, or PR template.

**Tasks:**
- [ ] Create CONTRIBUTING.md with:
  - How to set up dev environment
  - Code style guidelines
  - Testing requirements
  - Commit message format
  - PR process
- [ ] Create GitHub issue templates (bug report, feature request)
- [ ] Create pull request template
- [ ] Add CODE_OF_CONDUCT.md (use Contributor Covenant)

**Rationale:** Essential for community contribution.

---

### 5. Add Installation Verification & Tests
**ID:** BACKLOG-005  
**Priority:** 🟠 P1  
**Effort:** M  
**Impact:** High  

**Problem:** No way to verify installation works, no test suite.

**Tasks:**
- [ ] Add "Verify Installation" section to README
- [ ] Create smoke test script (`tests/smoke-test.sh`)
- [ ] Add basic test fixtures/mocks for API
- [ ] Add CI configuration (GitHub Actions)
- [ ] Add test documentation

**Acceptance Criteria:**
- User can run `simplelogin --test` or similar
- CI runs tests on every PR
- Test coverage > 50% for critical paths

**Rationale:** Builds trust and prevents regressions.

---

### 6. Complete package.json
**ID:** BACKLOG-006  
**Priority:** 🟠 P1  
**Effort:** XS  
**Impact:** Medium  

**Problem:** package.json incomplete - missing bin field, repository, author, etc.

**Tasks:**
- [ ] Add `bin` field for npm global installation
- [ ] Add `repository` URL
- [ ] Add `author` field
- [ ] Add `bugs` URL
- [ ] Add `homepage` URL
- [ ] Add `scripts` for dev/test
- [ ] Update version in package.json to match ClawHub

**Rationale:** Proper package.json enables npm installation and better metadata.

---

### 7. Add Configuration File Support
**ID:** BACKLOG-007  
**Priority:** 🟠 P1  
**Effort:** M  
**Impact:** Medium  

**Problem:** Everything is environment variables - no persistent configuration, no defaults.

**Tasks:**
- [ ] Support `~/.simplelogin/config` or `~/.simpleloginrc`
- [ ] Support config in XDG config directory (`$XDG_CONFIG_HOME/simplelogin/config`)
- [ ] Allow setting defaults (default mailbox, default domain, etc.)
- [ ] Config file overrides environment variables
- [ ] Add `simplelogin config` command to view/edit config

**Rationale:** Improves usability for power users and reduces env var clutter.

---

### 8. Add Search/Filter Capabilities
**ID:** BACKLOG-008  
**Priority:** 🟠 P1  
**Effort:** M  
**Impact:** Medium  

**Problem:** Can only list recent aliases, no search or advanced filtering.

**Tasks:**
- [ ] Add `--search` option to list command
- [ ] Add `--status` filter (enabled/disabled/all)
- [ ] Add `--note` filter (search in notes)
- [ ] Add `--created-after` / `--created-before` date filters
- [ ] Add pagination support for large accounts

**Rationale:** Essential for users with many aliases.

---

### 9. Create CHANGELOG.md
**ID:** BACKLOG-009  
**Priority:** 🟠 P1  
**Effort:** XS  
**Impact:** Medium  

**Problem:** No version history or changelog.

**Tasks:**
- [ ] Create CHANGELOG.md in Keep a Changelog format
- [ ] Document all versions from 1.0.0
- [ ] Add changelog automation (optional)
- [ ] Link changelog in README

**Rationale:** Standard open source practice, helps users understand changes.

---

### 10. Resolve Contact Command Discrepancy
**ID:** BACKLOG-010  
**Priority:** 🟠 P1  
**Effort:** S  
**Impact:** Medium  

**Problem:** Contact commands documented in SKILL.md but may not exist in scripts/simplelogin main dispatcher.

**Tasks:**
- [ ] Verify contact-create/contact-list commands exist in canonical script
- [ ] If missing, implement them
- [ ] If present, add to --help output
- [ ] Verify documentation matches implementation
- [ ] Add examples to README

**Rationale:** Feature documentation/code mismatch damages trust.

---

## P2 - Medium (Could Have)

### 11. Add Output Format Options
**ID:** BACKLOG-011  
**Priority:** 🟡 P2  
**Effort:** M  
**Impact:** Medium  

**Problem:** Only plain text and JSON output available.

**Tasks:**
- [ ] Add `--format` flag (table, json, csv, tsv)
- [ ] Implement table formatting for list command
- [ ] Add CSV export for scripting
- [ ] Add custom template support (optional, advanced)

**Acceptance Criteria:**
- `simplelogin list --format=table` shows pretty table
- `simplelogin list --format=csv` for import to spreadsheets

---

### 12. Add Shell Completion
**ID:** BACKLOG-012  
**Priority:** 🟡 P2  
**Effort:** S  
**Impact:** Low  

**Problem:** No bash/zsh completion scripts.

**Tasks:**
- [ ] Create bash completion script
- [ ] Create zsh completion script
- [ ] Document installation for each shell
- [ ] Add to package.json bin field

---

### 13. Add Man Page
**ID:** BACKLOG-013  
**Priority:** 🟡 P2  
**Effort:** S  
**Impact:** Low  

**Problem:** No `man simplelogin` support.

**Tasks:**
- [ ] Create man page in troff format or markdown (pandoc)
- [ ] Document installation to `/usr/share/man/man1/`
- [ ] Include in package

---

### 14. Refactor to Modules
**ID:** BACKLOG-014  
**Priority:** 🟡 P2  
**Effort:** L  
**Impact:** Medium  

**Problem:** scripts/simplelogin is 478 lines monolithic file.

**Tasks:**
- [ ] Split into modules:
  - `lib/api.sh` - API client
  - `lib/auth.sh` - Authentication
  - `lib/output.sh` - Output formatting
  - `cmds/create.sh` - Create command
  - `cmds/list.sh` - List command
  - etc.
- [ ] Create main entry point that sources modules
- [ ] Update installation instructions

**Rationale:** Improves maintainability and testability.

---

### 15. Add API Provider Abstraction
**ID:** BACKLOG-015  
**Priority:** 🟡 P2  
**Effort:** L  
**Impact:** Medium  

**Problem:** Tightly coupled to SimpleLogin API - can't support AnonAddy, etc.

**Tasks:**
- [ ] Create API provider abstraction layer
- [ ] Implement SimpleLogin provider
- [ ] Document how to add new providers
- [ ] Add `--provider` flag to select API provider
- [ ] Create example AnonAddy provider (optional)

**Rationale:** Enables multi-provider support, appeals to FOSS purists who prefer self-hosted.

---

### 16. Add Plugin/Hook System
**ID:** BACKLOG-016  
**Priority:** 🟡 P2  
**Effort:** L  
**Impact:** Medium  

**Problem:** Can't extend behavior without modifying source.

**Tasks:**
- [ ] Define hook points (pre-command, post-command, on-error, etc.)
- [ ] Create plugin directory structure (`~/.simplelogin/plugins/`)
- [ ] Document plugin API
- [ ] Create example plugins
- [ ] Add `simplelogin plugin` command to manage plugins

---

### 17. Add Batch Operations
**ID:** BACKLOG-017  
**Priority:** 🟡 P2  
**Effort:** M  
**Impact:** Medium  

**Problem:** Can only manage one alias at a time.

**Tasks:**
- [ ] Add batch enable/disable/delete
- [ ] Support file input (`--file aliases.txt`)
- [ ] Support pattern matching (`--pattern ".*@domain.com"`)
- [ ] Add confirmation prompts for batch operations

---

### 18. Add Import/Export Functionality
**ID:** BACKLOG-018  
**Priority:** 🟡 P2  
**Effort:** M  
**Impact:** Medium  

**Problem:** No backup or migration capability.

**Tasks:**
- [ ] Add `simplelogin export` command (JSON format)
- [ ] Add `simplelogin import` command
- [ ] Support encryption for exports (optional)
- [ ] Document backup best practices

---

## P3 - Low (Nice to Have)

### 19. Add Statistics & Reporting
**ID:** BACKLOG-019  
**Priority:** 🟢 P3  
**Effort:** M  
**Impact:** Low  

**Features:**
- Email forwarding statistics per alias
- Activity timeline
- Usage reports
- Export to CSV/PDF

---

### 20. Add Mailbox Management
**ID:** BACKLOG-020  
**Priority:** 🟢 P3  
**Effort:** S  
**Impact:** Low  

**Features:**
- List all mailboxes
- Set default mailbox
- Create aliases on specific mailboxes
- Mailbox statistics

---

### 21. Add Domain Management
**ID:** BACKLOG-021  
**Priority:** 🟢 P3  
**Effort:** S  
**Impact:** Low  

**Features:**
- List custom domains
- Create aliases on specific domains
- Domain statistics

---

### 22. Add Activity Logs
**ID:** BACKLOG-022  
**Priority:** 🟢 P3  
**Effort:** M  
**Impact:** Low  

**Features:**
- Show recent email activity
- Filter by alias
- Show sender, date, subject

---

### 23. Add Interactive TUI Mode
**ID:** BACKLOG-023  
**Priority:** 🟢 P3  
**Effort:** XL  
**Impact:** Low  

**Features:**
- Terminal UI for browsing aliases
- Keyboard navigation
- Quick actions (toggle, delete)
- Search/filter in TUI

---

### 24. Add Temporary Aliases
**ID:** BACKLOG-024  
**Priority:** 🟢 P3  
**Effort:** M  
**Impact:** Low  

**Features:**
- Create alias with auto-delete time
- `--expires-in 1h` or `--expires-at 2026-04-08T20:00:00Z`
- Scheduled cleanup

---

### 25. Add Email Client Integrations
**ID:** BACKLOG-025  
**Priority:** 🟢 P3  
**Effort:** L  
**Impact:** Low  

**Features:**
- Mutt/NeoMutt integration
- Thunderbird plugin
- Apple Mail plugin
- Automatic reverse alias creation

---

## Documentation Debt

### 26. Add Quick Start Guide
**ID:** DOC-001  
**Priority:** 🟠 P1  
**Effort:** XS  

**Tasks:**
- [ ] Create "5-Minute Quick Start" section in README
- [ ] Include copy-paste commands
- [ ] Add verification step

---

### 27. Add API Key Setup Walkthrough
**ID:** DOC-002  
**Priority:** 🟠 P1  
**Effort:** XS  

**Tasks:**
- [ ] Add direct link to SimpleLogin API key page
- [ ] Add step-by-step screenshots (or ASCII art)
- [ ] Explain API key permissions

---

### 28. Add Real-World Examples
**ID:** DOC-003  
**Priority:** 🟡 P2  
**Effort:** S  

**Tasks:**
- [ ] Common workflows section
- [ ] Integration examples (scripts, cron jobs)
- [ ] Email client integration examples

---

### 29. Add Security Best Practices
**ID:** DOC-004  
**Priority:** 🟡 P2  
**Effort:** XS  

**Tasks:**
- [ ] Threat model documentation
- [ ] API key security recommendations
- [ ] File permission guidance

---

### 30. Add Badges to README
**ID:** DOC-005  
**Priority:** 🟢 P3  
**Effort:** XS  

**Tasks:**
- [ ] License badge
- [ ] Version badge
- [ ] CI status badge (after adding CI)
- [ ] ClawHub badge

---

## Effort/Impact Matrix

```
Impact
  ↑
  │  BACKLOG-001    BACKLOG-004
  │  BACKLOG-002    BACKLOG-005
  │  BACKLOG-003    BACKLOG-007
  │─────────────────────────────────
  │               BACKLOG-014
  │               BACKLOG-015
  │               BACKLOG-016
  │
  └──────────────────────────────────→ Effort
     S    M    L    XL
```

**Quick Wins (Low Effort, High Impact):**
- BACKLOG-001: Consolidate scripts
- BACKLOG-003: Document missing commands
- BACKLOG-006: Complete package.json
- BACKLOG-009: Create CHANGELOG.md
- DOC-001: Add quick start guide
- DOC-002: Add API key walkthrough

**Strategic Investments (High Effort, High Impact):**
- BACKLOG-002: Fix error handling (M effort, Very High impact)
- BACKLOG-005: Add tests & CI (M effort, High impact)
- BACKLOG-007: Config file support (M effort, Medium impact)

**Nice to Haves (Low Priority):**
- TUI mode, email integrations, statistics - defer until core is solid

---

## Recommended Sprint 1 (First 2 Weeks)

**Goal:** Fix critical blockers for community adoption

1. BACKLOG-001: Consolidate duplicate scripts
2. BACKLOG-002: Fix error handling & API response parsing
3. BACKLOG-003: Document missing commands
4. DOC-001: Add quick start guide
5. DOC-002: Add API key setup walkthrough
6. BACKLOG-006: Complete package.json
7. BACKLOG-009: Create CHANGELOG.md

**Estimated Effort:** 3-4 days total

---

## Recommended Sprint 2 (Weeks 3-4)

**Goal:** Enable community contribution

1. BACKLOG-004: Create CONTRIBUTING.md and issue templates
2. BACKLOG-005: Add installation verification & basic tests
3. BACKLOG-010: Resolve contact command discrepancy
4. BACKLOG-008: Add search/filter capabilities
5. DOC-003: Add real-world examples

**Estimated Effort:** 4-5 days total

---

*Backlog generated from Goose/open source community review. Prioritize based on community feedback and adoption goals.*
