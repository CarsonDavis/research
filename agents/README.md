# Agents

Claude Code can be customized in two ways:

1. **Markdown instructions** - Put guidelines in `CLAUDE.md` files and Claude reads them automatically
2. **Agents** - Specialized subagents with their own tools, models, and permissions

This folder contains the research agent I use on my machine. You can use it directly or adapt it for your own workflow.

## Using My Agent

Copy to your Claude agents directory:

```bash
cp deep-research.md ~/.claude/agents/
```

Now when you ask Claude to "research" or "investigate" something, it will automatically use this agent.

## Making Your Own

Create a markdown file in `~/.claude/agents/` with YAML frontmatter:

```markdown
---
name: my-agent
description: When to trigger this agent. Be specific about keywords.
model: opus
tools: Read, Grep, WebSearch, WebFetch, Write
---

Instructions for the agent go here. Tell it what to do,
how to structure output, what standards to follow.
```

Key fields:
- **name** - Identifier for the agent
- **description** - Trigger conditions (Claude uses this to decide when to invoke it)
- **model** - `opus` (complex), `sonnet` (balanced), or `haiku` (fast)
- **tools** - What the agent can use

## Agents vs CLAUDE.md

Use **CLAUDE.md** for project-specific rules that apply to everything (coding standards, commit style, etc.).

Use **agents** for specialized tasks that need isolation - separate context window, limited tools, different model.
