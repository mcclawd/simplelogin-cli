# SimpleLogin CLI - LangChain Backlog

**Prioritized feature backlog for LangChain users and agent integrations**

---

## 🔴 P0 - Critical (Fix Immediately)

### 1. Implement Missing `delete` Command
- **Issue:** Documented but not implemented in script
- **Impact:** Users can't clean up aliases
- **Effort:** 30 minutes (copy pattern from enable/disable)
- **LangChain relevance:** High - agents need full CRUD operations

```bash
# Implementation needed:
simplelogin delete shopping@example.com
```

### 2. Standardize JSON Error Output
- **Issue:** Errors break programmatic parsing in JSON mode
- **Impact:** LangChain can't handle errors gracefully
- **Effort:** 1 hour (refactor error handling)
- **LangChain relevance:** Critical - structured errors required for agent workflows

```bash
# Current (broken for agents):
Error: API key not found

# Desired:
{"error": "MISSING_API_KEY", "message": "SIMPLELOGIN_API_KEY env var not set", "help": "Export variable or use Warden"}
```

### 3. Add Curl Timeouts and Retry Logic
- **Issue:** Script can hang indefinitely on network issues
- **Impact:** Agent workflows timeout/break
- **Effort:** 30 minutes (add `--max-time 30 --retry 3` to curl)
- **LangChain relevance:** High - reliability for automated workflows

---

## 🟠 P1 - High Priority (Next Sprint)

### 4. Implement Pagination Support
- **Issue:** Hardcoded to `page_id=0`, limited to first page
- **Impact:** Users with >20 aliases can't see all
- **Effort:** 2 hours (parse `--page` and `--limit` args)
- **LangChain relevance:** High - agents may manage many aliases

```bash
simplelogin list --page 2 --limit 50
```

### 5. Add `--version` and `--health` Commands
- **Issue:** No way to verify installation or diagnose issues
- **Impact:** Debugging is harder
- **Effort:** 1 hour
- **LangChain relevance:** Medium - useful for setup validation

```bash
simplelogin --version
# → simplelogin-cli v1.0.0

simplelogin status
# → API: ✓ | Key: ✓ | Mailbox: ✓ | Rate limit: 450/500
```

### 6. Create LangChain Python Wrapper
- **Issue:** Need to manually invoke CLI from Python
- **Impact:** Friction for LangChain developers
- **Effort:** 3 hours (create `@tool` decorated functions)
- **LangChain relevance:** Critical - first-class integration

```python
from langchain.tools import tool
from simplelogin_cli import create_alias, list_aliases, disable_alias

@tool
def create_simplelogin_alias(prefix: str, note: str = "") -> dict:
    """Create a SimpleLogin email alias.
    
    Args:
        prefix: The alias prefix (e.g., 'shopping' for shopping@example.com)
        note: Optional description of what the alias is for
    
    Returns:
        dict with 'email', 'id', 'status' keys
    """
    return create_alias(prefix, note)
```

### 7. Document Rate Limits and API Behavior
- **Issue:** No info on API limits, throttling, quotas
- **Impact:** Agents may hit undocumented limits
- **Effort:** 2 hours (research SimpleLogin API docs)
- **LangChain relevance:** High - agents need to handle 429s

---

## 🟡 P2 - Medium Priority (Future Enhancement)

### 8. Add Search/Filter Options
- **Issue:** Can only filter by domain
- **Impact:** Hard to find specific aliases
- **Effort:** 2 hours (add `--search`, `--status`, `--tag` flags)
- **LangChain relevance:** Medium - agents managing many aliases

```bash
simplelogin list --search "amazon" --status enabled --created-after 2025-01-01
```

### 9. Implement Bulk Operations
- **Issue:** One alias at a time
- **Impact:** Tedious for batch changes
- **Effort:** 2 hours
- **LangChain relevance:** Medium - agents may batch-process

```bash
simplelogin disable --all --domain "temp.com"
simplelogin enable --list shopping@example.com,network@example.com
```

### 10. Add Alias Statistics
- **Issue:** Can't see email counts per alias
- **Impact:** Hard to audit unused aliases
- **Effort:** 3 hours (requires SimpleLogin API endpoint research)
- **LangChain relevance:** Medium - agents may clean up based on usage

```bash
simplelogin stats shopping@example.com
# → 2026-04: 15 emails received, 3 forwarded, 2 blocked
```

### 11. Implement Export/Import
- **Issue:** No backup/restore functionality
- **Impact:** Can't migrate or backup aliases
- **Effort:** 3 hours
- **LangChain relevance:** Low-Medium - agents may manage backups

```bash
simplelogin export > aliases-backup.json
simplelogin import aliases-backup.json
```

### 12. Add Integration Tests
- **Issue:** No automated tests
- **Impact:** Breaking changes undetected
- **Effort:** 4 hours (use SimpleLogin test mode or sandbox)
- **LangChain relevance:** Medium - ensures reliability for agent workflows

---

## 🟢 P3 - Nice to Have (Optional Enhancements)

### 13. Add Alias Expiration
- **Feature:** Auto-disable after date
- **Effort:** 2 hours (cron job or SimpleLogin API feature)
- **LangChain relevance:** Low - specialized use case

```bash
simplelogin create temp --expires 2026-12-31
```

### 14. Implement Tags/Labels
- **Feature:** Organize aliases with tags
- **Effort:** 4 hours (requires tracking local metadata)
- **LangChain relevance:** Low - organizational preference

```bash
simplelogin tag shopping@example.com shopping subscriptions finance
```

### 15. Add Forwarding Rules
- **Feature:** Conditional forwarding based on sender/subject
- **Effort:** 6 hours (complex, may not be supported by SimpleLogin API)
- **LangChain relevance:** Low - advanced feature

```bash
simplelogin rule-add shopping@example.com --from "amazon.com" --forward purchases@inbox.com
```

### 16. Webhook Support
- **Feature:** Notify agent when new email arrives
- **Effort:** 4 hours (requires SimpleLogin webhook API)
- **LangChain relevance:** Medium - enables reactive agent workflows

```bash
simplelogin webhook-add --url "http://my-server/hook" --event "new-email"
```

### 17. Integrations with Other Tools
- **Feature:** Notion, Slack, Calendar integrations
- **Effort:** 4-8 hours each
- **LangChain relevance:** Variable - depends on agent ecosystem

```bash
simplelogin integrate --slack --channel "#security-alerts"
```

### 18. Mailbox Management Commands
- **Feature:** Full mailbox CRUD operations
- **Effort:** 3 hours
- **LangChain relevance:** Low - most users use single mailbox

```bash
simplelogin mailbox-list
simplelogin mailbox-set default 12345
```

### 19. Improve Security Documentation
- **Feature:** Key rotation guide, secure file permissions
- **Effort:** 1 hour (docs only)
- **LangChain relevance:** Medium - security best practices

### 20. Better Error Codes and Help Text
- **Feature:** Structured error codes with help URLs
- **Effort:** 2 hours
- **LangChain relevance:** Medium - faster debugging

```bash
# Error: [SL-001] API key missing
# Help: https://simplelogin-cli.example.com/docs/errors/SL-001
```

---

## 📊 Effort Summary

| Priority | Items | Total Effort |
|----------|-------|--------------|
| P0 - Critical | 3 | ~2 hours |
| P1 - High | 4 | ~10 hours |
| P2 - Medium | 5 | ~14 hours |
| P3 - Nice to Have | 8 | ~30 hours |
| **Total** | **20** | **~56 hours** |

---

## 🎯 Recommended MVP for LangChain Users

If you want to make this skill **LangChain-ready quickly**, focus on:

1. ✅ Fix `delete` command (P0-1)
2. ✅ Standardize JSON errors (P0-2)
3. ✅ Add timeouts/retry (P0-3)
4. ✅ Create LangChain Python wrapper (P1-6)
5. ✅ Document rate limits (P1-7)

**Total:** ~6 hours of work, massive improvement for agent users.

---

*Backlog prioritized from perspective of LangChain developer building agent workflows. Adjust based on your use case.*
