# ai-skills

Claude Code skills.

| Skill | What it does |
|---|---|
| [delegate](delegate/) | Splits a large task into independent subtasks, runs each in its own sub-agent, verifies the result, and compiles one recap. |
| [summarize](summarize/) | Layered summary of large context (files, folders, PDFs, URLs, PRs, chat) with source locators and ASCII flow diagrams (`--mermaid` for Mermaid). |
| [reverse-engineer](reverse-engineer/) | Reverse-engineers a codebase into business-analyst and technical documentation (flows, schema, API surface, CLAUDE.md). |

## Install

Copy or symlink a skill folder into `~/.claude/skills/`:

```bash
git clone https://github.com/fiqridwi/ai-skills.git
ln -s "$PWD/ai-skills/summarize" ~/.claude/skills/summarize
```
