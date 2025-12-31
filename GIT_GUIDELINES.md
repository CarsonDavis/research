# Git Guidelines

## Workflow

Two-step process:

1. **Add first** - Stage the files so the user can examine what will be committed
2. **Commit only after review** - Wait for the user to approve before committing

Don't combine these into one step. The user needs a chance to review staged changes.

## What to Add

- Add files that were recently worked on
- Or add what the user specifically requests
- Don't stage unrelated changes

## Commit Messages

Keep messages short and descriptive. Focus on what changed and why.

```bash
# Good
git commit -m "Add inflation adjustment to stock analysis"
git commit -m "Fix CPI interpolation for daily alignment"

# Bad - too vague
git commit -m "Updates"

# Bad - unnecessary metadata
git commit -m "Add feature

Generated with Claude Code
Co-Authored-By: ..."
```

No boilerplate, no tool attribution, no co-author lines. Just the message.
