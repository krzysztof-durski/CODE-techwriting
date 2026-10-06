# README 101: A Starter Guide for Engineers

A README is the front door to your project. It's the first thing a teammate, contributor, or future-you sees, and it decides whether they keep reading or close the tab.

This guide teaches you how to write one that works. It's also a README itself, so treat its structure as a worked example.

## Table of Contents

- [Why READMEs matter](#why-readmes-matter)
- [The 5-minute rule](#the-5-minute-rule)
- [Anatomy of a good README](#anatomy-of-a-good-readme)
- [Copy-paste template](#copy-paste-template)
- [Writing tips](#writing-tips)
- [Common mistakes](#common-mistakes)
- [Keeping it fresh](#keeping-it-fresh)
- [Checklist](#checklist)
- [Further reading](#further-reading)

## Why READMEs matter

A good README:

- **Saves time.** Answers the questions people would otherwise ask you in chat.
- **Speeds up onboarding.** New teammates can run the project on day one.
- **Signals quality.** Well-documented projects are trusted and adopted more.
- **Forces clarity.** If you can't explain what your project does in a paragraph, that's useful feedback about the project.

## The 5-minute rule

A stranger should be able to answer these three questions within five minutes of landing on your README:

1. **What is this?**
2. **How do I run it?**
3. **Where do I go for help or to contribute?**

Everything else is optional. If your README nails those three, it's already better than most.

## Anatomy of a good README

Order sections by how urgently a reader needs them. Put the essentials first.

| Section | Purpose | Required? |
|---|---|---|
| **Title & one-liner** | What the project is, in one sentence | Yes |
| **Badges** | Build status, version, license at a glance | Optional |
| **Overview** | The problem it solves and who it's for | Yes |
| **Quick start** | The shortest path to a running project | Yes |
| **Usage** | Realistic examples of common tasks | Yes |
| **Configuration** | Env vars, flags, config files | If applicable |
| **Project structure** | Map of key directories | For larger repos |
| **Development** | Setup, tests, linting, debugging | For contributors |
| **Contributing** | How to propose changes | Open source / team repos |
| **Support** | Where to ask questions or report bugs | Yes |
| **License** | Terms of use | Open source |

### Title and one-liner

Lead with the name and a single sentence a non-expert can follow.

> **Weak:** "A robust, scalable, next-generation data solution."
> **Strong:** "A command-line tool that converts CSV files to Parquet."

### Quick start

This is the most important section after the intro. Rules:

- Use **copy-pasteable commands** in code blocks.
- List **prerequisites** with versions (`Node 20+`, `Python 3.11+`).
- Show **expected output** so readers know it worked.
- Keep it to a handful of steps. Link to deeper docs for the rest.

### Usage

Show, don't tell. Give two or three realistic examples, from simple to advanced, rather than exhaustively listing every option.

### Contributing and support

Tell people what you want from them: how to run tests, your branching model, where to file issues. If you don't accept contributions, say that too.

## Copy-paste template

Start here and delete what doesn't apply.

````markdown
# Project Name

One sentence describing what this project does and who it's for.

![build](badge-url) ![license](badge-url)

## Overview

Two to four sentences: the problem, your approach, and what makes it useful.

## Quick Start

### Prerequisites

- Tool A (version X or higher)
- Tool B

### Install and run

```bash
git clone https://github.com/your-org/your-project.git
cd your-project
make install
make run
```

You should see:

```
Server listening on http://localhost:8080
```

## Usage

```bash
your-tool --input data.csv --output data.parquet
```

Describe what the example does and when you'd use it.

## Configuration

| Variable | Description | Default |
|---|---|---|
| `PORT` | Port to listen on | `8080` |
| `LOG_LEVEL` | `debug`, `info`, `warn`, `error` | `info` |

## Development

```bash
make test    # run the test suite
make lint    # run linters
```

## Contributing

1. Fork the repo and create a branch.
2. Make your change, with tests.
3. Open a pull request describing what and why.

## Support

Open an issue or contact the maintainers at team@example.com.

## License

Released under the [MIT License](LICENSE).
````

## Writing tips

- **Write for your reader, not yourself.** Assume they're smart but new to this project.
- **Be concrete.** Prefer examples and real commands over abstract descriptions.
- **Keep it scannable.** Use headings, short paragraphs, tables, and lists.
- **Test your instructions.** Follow your own Quick Start on a clean machine or fresh clone.
- **Link out for depth.** The README is a map, not the whole territory. Send readers to `docs/` for architecture notes, API references, and tutorials.
- **Pin versions.** "Works with Python" is vague. "Requires Python 3.11+" is actionable.
- **Use relative links** for files inside the repo so they work on forks and branches.
- **Add a screenshot or GIF** for anything visual. It's worth a thousand words of description.

## Common mistakes

| Mistake | Fix |
|---|---|
| No description of what the project does | Add a clear one-liner at the top |
| Setup steps that no longer work | Test them regularly; automate if you can |
| Assumed knowledge ("just configure the usual way") | Spell out every step for a newcomer |
| Wall of text | Break it up with headings, lists, and code blocks |
| Marketing language | Replace adjectives with facts and examples |
| Documenting everything in one file | Move deep dives to `docs/` and link to them |
| No license | Add one, or readers legally can't use your code |
| Stale badges and broken links | Prune what you can't maintain |

## Keeping it fresh

A stale README is worse than a short one, because it misleads people.

- **Update docs in the same pull request as the code change.** Add a README check to your PR template.
- **Run a link checker** in CI to catch dead links.
- **Have a new teammate follow it** as part of onboarding, and fix whatever confuses them.
- **Review it periodically.** A quarterly skim is enough to catch drift.

## Checklist

Before you publish, confirm:

- [ ] The title and one-liner explain what the project is
- [ ] Prerequisites and versions are listed
- [ ] Quick Start works from a clean clone
- [ ] Usage includes at least one realistic example
- [ ] Configuration options are documented
- [ ] Contributing and support paths are clear
- [ ] A license is included
- [ ] Links and commands have been tested
- [ ] Someone else has read it

## Further reading

- [Make a README](https://www.makeareadme.com/)
- [GitHub Docs: About READMEs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)
- [Standard Readme specification](https://github.com/RichardLitt/standard-readme)
- [Keep a Changelog](https://keepachangelog.com/) (a natural companion to your README)

## License

This guide is released under the [MIT License](LICENSE). Adapt it freely for your team.
