# CONTRIBUTING_LLM.md

> The quality bar doesn't drop. It gets automated.

## The Problem

Last week I got a PR merged into a Rust project without knowing Rust. Some reactions were predictable: *"AI-generated code is garbage."*

They're right — if there's no process.

The problem isn't that AI generates bad code. The problem is that open-source repositories aren't prepared for this new type of contribution.

Today, anyone can generate a fix with Claude Code, Copilot, or Cursor. But if a repo doesn't define what it demands from code — regardless of where it came from — the maintainer becomes the only quality filter. **And that doesn't scale.**

Vibe coding has shifted the weight of contributing from the developer to whoever has to approve the PRs. This is my attempt to fix that.

## The Solution

`CONTRIBUTING_LLM.md` is a document any open-source project can copy and use tomorrow.

It defines three things:

- **Mandatory disclosure** — `Co-Authored-By` trailer in every AI-assisted commit
- **Mandatory tests** — no new functionality without test coverage
- **License compliance** — contributors are responsible for verifying AI output doesn't reproduce incompatible licensed code

It also includes ready-to-use **GitHub Actions workflows** that enforce these rules automatically and block merges if they aren't met.

## Usage

1. Copy `CONTRIBUTING_LLM.md` to your repository root.
2. Reference it from your existing `CONTRIBUTING.md`.
3. Add the GitHub Actions workflow to `.github/workflows/ai-contribution-check.yml`.
4. Add the PR template to `.github/PULL_REQUEST_TEMPLATE.md`.
5. Communicate the new requirements to existing contributors.

## What the CI Checks Enforce

| Check | Blocks merge? |
|-------|:---:|
| AI co-author trailer present in commits | Yes |
| Test files included when source files change | Yes |
| Linter pass | Yes |
| PR description contains AI disclosure section | Yes |
| License scan on changed files | Yes |

## License

[CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) — copy it, fork it, adapt it. No attribution required.
