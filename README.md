# Unified audit framework

255 expert-persona frameworks across 13 quality domains for Claude Code.

Each framework is a deep specialist document (~190 lines) loaded into its own agent context — not a checklist. The agent reading a framework thinks like a 20-year specialist in that discipline.

## Domains

| Domain | Command | Frameworks |
|--------|---------|------------|
| UX | `/qq-audit-ux` | 20 |
| Product | `/qq-audit-product` | 20 |
| Visual UI | `/qq-audit-visual` | 22 |
| Content/Copy | `/qq-audit-copy` | 22 |
| Frontend | `/qq-audit-frontend` | 22 |
| Performance | `/qq-audit-performance` | 22 |
| Security | `/qq-audit-security` | 23 |
| Testing | `/qq-audit-testing` | 22 |
| Backend/API | `/qq-audit-backend` | 22 |
| SEO | `/qq-audit-seo` | 15 |
| DevOps | `/qq-audit-devops` | 15 |
| Data Quality | `/qq-audit-data` | 15 |
| Compliance | `/qq-audit-compliance` | 15 |
| **Total** | | **255** |

## Installation

Copy the skill directories into your Claude Code skills folder:

```bash
cp -r skills/qq-audit-* ~/.agents/skills/
cp commands/qq-audit*.md ~/.claude/commands/
```

## Usage

- `/qq-audit` — smart routing picks the right domains for your project
- `/qq-audit-ux` — run all 20 UX frameworks serially
- `/qq-audit-ux Fitts's Law` — run just one framework
- `/qq-audit-ux list` — see available frameworks
- `/qq-audit pre-launch` — run the pre-launch preset (8 domains)

## License

MIT
