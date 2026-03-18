---
name: simplelogin-cli
description: Create and manage SimpleLogin email aliases from the command line. Protect your real email with secure, private aliases.
homepage: https://simplelogin.io
metadata:
  openclaw:
    emoji: "📧"
    os: ["linux", "darwin", "win32"]
    requires:
      bins: ["curl", "jq"]
    install: []
  clawhub:
    category: "privacy"
    tags: ["email", "alias", "privacy", "simplelogin"]
    license: "MIT"
---

# SimpleLogin CLI

Create and manage privacy-focused email aliases with SimpleLogin. Protect your real email address when signing up for services, newsletters, or online shopping.

## What is SimpleLogin?

SimpleLogin is an open-source email aliasing service that lets you:
- Create unlimited email aliases (e.g., `shopping@alias.com` → your real email)
- Receive emails anonymously without exposing your real address
- Reply to emails while keeping your identity private
- Track which services sell your data when spam arrives

## Features

- ✅ Create custom aliases (you choose the prefix)
- ✅ Create random aliases (generated automatically)
- ✅ List all your aliases with status
- ✅ Enable/disable aliases on demand
- ✅ Smart hostname detection (auto-suggests alias based on website)
- ✅ Multiple mailbox support
- ✅ Secure API key management

## Prerequisites

1. **SimpleLogin account** - Sign up at https://simplelogin.io
2. **API key** - Generate in SimpleLogin dashboard → API Keys
3. **Store API key securely** - Use environment variable or password manager

## Installation

```bash
# Install via ClawHub (when published)
clawhub install simplelogin-cli

# Or clone manually
git clone https://github.com/yourusername/simplelogin-cli.git
```

## Configuration

### Option 1: Environment Variable (Quick)

```bash
export SIMPLELOGIN_API_KEY="your-api-key-here"
```

### Option 2: Bitwarden/Password Manager (Recommended)

Store your API key in your password manager:
- **Name:** `SimpleLogin API Key`
- **Custom Field:** `api_key` = your API key

The skill will automatically retrieve this if Warden or similar credential management is available.

## Usage

### Create a Custom Alias

```bash
# Create alias with your chosen prefix
simplelogin create shopping
# → shopping@yourdomain.com

# With note
simplelogin create amazon --note "Amazon purchases"
# → amazon@yourdomain.com

# For specific website
simplelogin create --for github.com
# → github-xyz@simplelogin.com
```

### Create a Random Alias

```bash
# Generate random alias
simplelogin random
# → random_word123@simplelogin.com

# With note
simplelogin random --note "Newsletter signup"
```

### List Aliases

```bash
# Show recent aliases
simplelogin list

# Show all
simplelogin list --all

# Filter by domain
simplelogin list --domain simplelogin.com
```

### Manage Aliases

```bash
# Disable alias
simplelogin disable shopping@yourdomain.com

# Enable alias
simplelogin enable shopping@yourdomain.com

# Delete alias
simplelogin delete shopping@yourdomain.com
```

## API Reference

### simplelogin create [prefix]

Create a custom alias.

**Options:**
- `prefix` - The alias prefix (before @)
- `--note, -n` - Add a note/description
- `--for, -f` - Suggest alias based on hostname (e.g., `amazon.com`)
- `--mailbox, -m` - Specify mailbox ID (default: first available)

**Examples:**
```bash
simplelogin create shopping
simplelogin create amazon --note "Prime member"
simplelogin create --for netflix.com --note "Streaming"
```

### simplelogin random

Create a random alias.

**Options:**
- `--note, -n` - Add a note/description
- `--word, -w` - Use word-based random (default: uuid-style)
- `--mailbox, -m` - Specify mailbox ID

**Examples:**
```bash
simplelogin random
simplelogin random --note "Forum signup"
simplelogin random --word
```

### simplelogin list

List your aliases.

**Options:**
- `--all, -a` - Show all aliases (not just recent)
- `--domain, -d` - Filter by domain
- `--enabled` - Show only enabled
- `--disabled` - Show only disabled

### simplelogin disable|enable|delete <alias>

Manage existing aliases.

**Examples:**
```bash
simplelogin disable shopping@yourdomain.com
simplelogin enable shopping@yourdomain.com
simplelogin delete temp@simplelogin.com
```

## Agent/JSON Mode

For programmatic use (agents, scripts), set `SIMPLELOGIN_JSON=true`:

```bash
export SIMPLELOGIN_JSON=true
simplelogin create shopping --note "Test"
# → {"email":"shopping@yourdomain.com","id":12345,"status":"created"}
```

## Security Notes

- 🔐 **API keys are never stored in the skill** - Always use environment variables or password managers
- 🔐 **Aliases are private** - SimpleLogin doesn't log or sell your data
- 🔐 **Open source** - SimpleLogin code is auditable at https://github.com/simple-login

## Troubleshooting

### "API key not found"
Make sure you've set the `SIMPLELOGIN_API_KEY` environment variable or stored the key in your password manager.

### "No suffixes available"
Check your SimpleLogin account status. Free accounts have limited suffixes.

### Emails going to spam
This is normal for test emails. Gmail may flag programmatic emails. Check your spam folder.

## Contributing

Contributions welcome! Please follow the existing code style and add tests for new features.

## License

MIT License - See LICENSE file
