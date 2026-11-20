# engineering-rules

Agent skills that encode a strict engineering discipline for AI coding agents: a correction pass before every task, verification before any claim, mutation test-driven development, and a fixed prose style.

| Skill | Loads when |
| --- | --- |
| [`engineering-rules`](skills/engineering-rules/SKILL.md) | Any task that writes, changes, tests, reviews, commits or ships code, configures CI, or authors agent instructions |
| [`code-review-standards`](skills/code-review-standards/SKILL.md) | Reviewing a pull request, branch, diff or commit, self-review, or responding to review comments |

`code-review-standards` is part of `engineering-rules` and refers to its section numbers, so install both.

## Install

Copy both skill directories into your agent's skills directory. For Claude Code:

```sh
git clone https://github.com/Sharma-IT/engineering-rules.git
cp -r engineering-rules/skills/* ~/.claude/skills/
```

Use `.claude/skills/` inside a repository instead to scope the rules to one project.

## Skill or instruction file

An instruction file (`CLAUDE.md`, `AGENTS.md`) is loaded on every turn. A skill is loaded only when the agent judges its description relevant, so the rules apply from that point on rather than from the first message. Where you want them binding on every turn, paste the body of `engineering-rules` into your instruction file instead.

Do not do both. The rules require one home per instruction, and two loaded copies drift apart.
