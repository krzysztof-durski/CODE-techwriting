# README Cheat Sheet

**The 5-minute rule:** a stranger should learn *what this is*, *how to run it*, and *where to get help* within five minutes.

## Section order (essentials first)

| # | Section | What to include |
|---|---|---|
| 1 | **Title + one-liner** | What it is, in one plain sentence |
| 2 | **Overview** | Problem, audience, approach (2-4 sentences) |
| 3 | **Quick start** | Prerequisites with versions, copy-paste commands, expected output |
| 4 | **Usage** | 2-3 realistic examples, simple to advanced |
| 5 | **Configuration** | Env vars, flags, defaults (a table works well) |
| 6 | **Development** | Run tests, lint, debug |
| 7 | **Contributing** | Branching, PR process, or "not accepting contributions" |
| 8 | **Support** | Where to file issues or ask questions |
| 9 | **License** | Link to `LICENSE` |

## Minimal template

````markdown
# Project Name
One sentence: what it does and who it's for.

## Quick Start
Requires Tool X 1.2+.
```bash
git clone <repo> && cd <repo>
make install && make run
```
Expected: `Server listening on http://localhost:8080`

## Usage
```bash
your-tool --input in.csv --output out.parquet
```

## Contributing / Support / License
Links or short notes.
````

## Do

- Write for a smart newcomer
- Show real commands and output
- Pin versions ("Python 3.11+")
- Use headings, tables, short paragraphs
- Link to `docs/` for depth
- Use relative links inside the repo
- Add a screenshot or GIF for visual tools
- Test your Quick Start on a fresh clone

## Don't

- Use vague marketing language
- Assume "the usual setup"
- Cram everything into one file
- Leave out a license
- Keep stale badges or dead links

## Keep it fresh

- Update the README in the same PR as the code
- Run a link checker in CI
- Have each new teammate follow it and fix what confuses them

## Pre-publish checklist

- [ ] One-liner explains the project
- [ ] Prerequisites and versions listed
- [ ] Quick start works from a clean clone
- [ ] At least one realistic usage example
- [ ] Config documented
- [ ] Contributing and support paths clear
- [ ] License included
- [ ] Someone else has read it
