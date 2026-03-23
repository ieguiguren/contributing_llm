# CONTRIBUTING_LLM.md

> The quality bar doesn't drop. It gets automated.

## The Problem

Anyone can open a PR today. That's always been true — but the barrier just dropped to zero.

With Claude Code, Copilot, or Cursor, generating a plausible-looking fix takes minutes. No deep understanding required. The diff compiles. The description sounds right. And now it's sitting in your review queue.

The usual reaction from maintainers is: *"AI-generated code is garbage."*

That's not the real problem.

**The real problem is that most repositories have no standard for what they expect from code — regardless of where it came from.** No automated enforcement. No explicit quality contract. Just a maintainer reading diffs, one by one, making judgment calls that could have been policies.

Vibe coding didn't create new bad contributors. It created a volume problem. It shifted the weight of quality from the person submitting the PR to the person approving it. And that doesn't scale.

The bottleneck isn't AI. It's the absence of a defined, automated quality bar — and maintainers are paying the price.

## The Solution

`CONTRIBUTING_LLM.md` is a document any open-source project can copy and use tomorrow.

Instead of relying on maintainer intuition, it codifies the quality contract into three enforceable rules:

- **Mandatory disclosure** — `Co-Authored-By` trailer in every AI-assisted commit
- **Mandatory tests** — no new functionality without test coverage
- **License compliance** — contributors are responsible for verifying AI output doesn't reproduce incompatible licensed code

It also includes ready-to-use **GitHub Actions workflows** that enforce these rules automatically and block merges if they aren't met.

The maintainer stops being the filter. The process becomes the filter.

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
