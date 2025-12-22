# Varlock Skill for Claude Code

> Secure-by-default environment variable management. Ensures secrets are **never exposed** in Claude sessions.

## Why This Skill?

When working with Claude Code, secrets can accidentally leak into:
- Terminal output
- Claude's input/output context
- Log files or traces
- Git commits or diffs

This skill wraps [Varlock](https://varlock.dev) to enforce secure patterns and prevent accidental exposure.

## Installation

### Option A: Claude Plugin (Recommended)

```bash
claude plugin add github:wrsmith108/varlock-claude-skill
```

### Option B: Manual

```bash
git clone https://github.com/wrsmith108/varlock-claude-skill ~/.claude/skills/varlock
```

## Prerequisites

Install the Varlock CLI:

```bash
curl -sSfL https://varlock.dev/install.sh | sh -s -- --force-no-brew
export PATH="$HOME/.varlock/bin:$PATH"
```

## Core Principle

**Secrets must NEVER appear in Claude's context.**

| Never Do | Safe Alternative |
|----------|------------------|
| `cat .env` | `cat .env.schema` |
| `echo $SECRET` | `varlock load` |
| `printenv \| grep API` | `varlock load \| grep API` |

## Quick Reference

```bash
# Validate all secrets (shows masked values)
varlock load

# Quiet validation (no output on success)
varlock load --quiet

# Run command with secrets injected
varlock run -- npm start

# View schema (safe - no values)
cat .env.schema
```

## Schema File

Create `.env.schema` to define variable types and sensitivity:

```bash
# Global defaults
# @defaultSensitive=true @defaultRequired=infer

# Public config
# @type=enum(development,staging,production) @sensitive=false
NODE_ENV=development

# Sensitive secrets
# @type=string(startsWith=sk_) @required @sensitive
STRIPE_SECRET_KEY=

# @type=url @required @sensitive
DATABASE_URL=
```

### Annotations

| Annotation | Effect |
|------------|--------|
| `@sensitive` | Value masked in all output |
| `@sensitive=false` | Value shown (for public keys) |
| `@required` | Must be present |
| `@type=string(startsWith=X)` | Prefix validation |

## Integration with Other Skills

### Docker Skill

```dockerfile
# Use Varlock as entrypoint
CMD ["varlock", "run", "--", "npm", "start"]
```

### Clerk Skill

```bash
# Test passwords are sensitive
# @type=string @sensitive
TEST_ADMIN_PASSWORD=

# Test emails are NOT sensitive (contain +clerk_test)
# @type=string(contains=+clerk_test) @sensitive=false
TEST_ADMIN_EMAIL=
```

## Handling Secret Requests

When users ask Claude to:

- **"Check if API key is set"** → `varlock load | grep API_KEY`
- **"Debug authentication"** → `varlock load` (validates all)
- **"Update a secret"** → Decline; ask user to update manually
- **"Show me .env"** → `cat .env.schema` instead

## Credits

This skill wraps [Varlock](https://github.com/dmno-dev/varlock) by [DMNO](https://dmno.dev).

## Related Skills

- [docker-claude-skill](https://github.com/wrsmith108/docker-claude-skill) — Container-based development
- [clerk-claude-skill](https://github.com/wrsmith108/clerk-claude-skill) — Authentication patterns
- [linear-claude-skill](https://github.com/wrsmith108/linear-claude-skill) — Issue management

## License

MIT
