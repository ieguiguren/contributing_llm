# AI-Assisted Contributions Guide

> Quality standards don't drop. They get automated.

This document defines the requirements for contributions that involve AI-generated or AI-assisted code. It complements the project's existing `CONTRIBUTING.md` and applies to any pull request where code was partially or fully generated using LLMs, copilots, or AI coding tools.

The goal is simple: **same quality bar, transparent process, automated enforcement.**

---

## 1. Disclosure

Every PR that includes AI-generated or AI-assisted code **must** disclose it.

### Commit co-authorship

Add the AI tool as a co-author in your commit message using the standard Git trailer format:

```
Co-Authored-By: Claude Code <claude@anthropic.com>
Co-Authored-By: GitHub Copilot <copilot@github.com>
```

### PR description

Include in your PR description:

- Which AI tool(s) were used and for what purpose.
- Which files or sections were AI-generated vs. human-written.
- Whether you modified the AI output or used it as-is.

**Example:**

```markdown
## AI Disclosure

- **Tool:** Claude Code (Opus 4.6)
- **Scope:** Generated initial fix for NVIDIA GPU detection in `src/gpu.rs`.
  Manually adjusted error handling and added integration test.
- **Modified:** Yes — refactored match arms, added edge case for hybrid GPUs.
```

### Why this matters

Transparency is not about blame. It's about traceability. When a bug surfaces six months later, maintainers need to know the context in which code was written to debug it effectively.

---

## 2. Licensing Compliance

AI-generated code **must not** introduce license-incompatible material.

### Rules

- Do not paste code from AI outputs that reproduce substantial portions of copyrighted or licensed code from other projects.
- If the AI tool was trained on open-source code, the contributor is responsible for verifying that the output does not replicate identifiable code from projects with incompatible licenses.
- When in doubt, rewrite. A clean-room implementation guided by AI is always safer than a direct copy.

### Contributor responsibility

By submitting a PR with AI-assisted code, you affirm that:

1. You have reviewed the generated code for potential license issues.
2. The code does not knowingly reproduce substantial portions of any codebase with an incompatible license.
3. You accept the same responsibility for AI-generated code as you would for code you wrote manually.

---

## 3. Testing Requirements

Every PR that introduces new functionality or modifies existing behavior **must** include tests. No exceptions for AI-generated code.

### Minimum requirements

- **Unit tests** for every new function or method.
- **Integration tests** when the change affects component interaction.
- **Edge case coverage** — AI-generated code tends to handle the happy path well but miss boundary conditions. Explicitly test for:
  - Empty or null inputs
  - Error states and failure modes
  - Platform-specific behavior (if applicable)

### Test quality

Tests must be meaningful. A test that simply asserts `true == true` or mirrors the implementation without validating behavior will be rejected.

```
# Bad: mirrors implementation
def test_add():
    assert add(2, 3) == 2 + 3

# Good: validates expected behavior
def test_add_positive_numbers():
    assert add(2, 3) == 5

def test_add_negative_numbers():
    assert add(-1, -1) == -2

def test_add_overflow():
    assert add(sys.maxsize, 1) raises OverflowError
```

---

## 4. Code Quality Standards

AI-assisted code is held to the **same standards** as any other contribution.

### The contributor must understand the code

If a maintainer asks "why did you implement it this way?" during review, "the AI suggested it" is not an acceptable answer. You must be able to:

- Explain the logic and intent of every function.
- Justify architectural decisions.
- Identify potential failure modes.

If you cannot explain it, do not submit it.

### Formatting and linting

All code must pass the project's configured linters and formatters before submission. Do not rely on the AI tool's default formatting — run the project's toolchain.

### Documentation

- Public functions and methods must have docstrings or inline comments explaining their purpose.
- Complex logic blocks must include a comment explaining **why**, not **what**.
- If the AI-generated code solves a non-obvious problem, include a brief explanation of the approach in the PR description.

---

## 5. CI Enforcement

The following checks should be automated in the project's CI pipeline to enforce these guidelines.

### Conceptual checks

| Check | Purpose | Blocks merge? |
|-------|---------|:---:|
| Co-author trailer present | Ensures AI disclosure in commits | Yes |
| Test coverage threshold | No untested new code | Yes |
| Linter pass | Code style compliance | Yes |
| PR template validation | AI disclosure section filled | Yes |
| License scan | Detect potential license conflicts | Yes |

### GitHub Actions implementation

#### AI disclosure check

```yaml
# .github/workflows/ai-contribution-check.yml
name: AI Contribution Standards

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  check-ai-disclosure:
    name: Verify AI disclosure
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Check for AI co-author disclosure
        run: |
          # List of known AI co-author patterns
          AI_PATTERNS=(
            "co-authored-by:.*claude"
            "co-authored-by:.*copilot"
            "co-authored-by:.*cursor"
            "co-authored-by:.*codewhisperer"
            "co-authored-by:.*gemini"
            "co-authored-by:.*ai"
          )

          # Get all commit messages in this PR
          COMMITS=$(git log --format="%B" origin/${{ github.base_ref }}..HEAD)

          # Check PR body for AI disclosure section
          PR_BODY="${{ github.event.pull_request.body }}"

          if echo "$PR_BODY" | grep -qi "AI Disclosure"; then
            echo "✅ AI disclosure section found in PR description."

            # If AI disclosure exists, verify co-author trailers
            HAS_COAUTHOR=false
            for pattern in "${AI_PATTERNS[@]}"; do
              if echo "$COMMITS" | grep -qi "$pattern"; then
                HAS_COAUTHOR=true
                break
              fi
            done

            if [ "$HAS_COAUTHOR" = false ]; then
              echo "⚠️  AI disclosure found in PR but no Co-Authored-By trailer in commits."
              echo "Please add: Co-Authored-By: <AI Tool> <email>"
              exit 1
            fi

            echo "✅ Co-author trailer found in commits."
          else
            echo "ℹ️  No AI disclosure section — treating as fully human-authored."
          fi

  check-tests:
    name: Verify test coverage
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check for test files in diff
        run: |
          # Get changed files
          CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD)

          # Check if source files were modified
          SRC_CHANGED=$(echo "$CHANGED" | grep -E '\.(rs|py|ts|js|go|java)$' | grep -v test || true)

          if [ -n "$SRC_CHANGED" ]; then
            # Check if test files were also modified/added
            TEST_CHANGED=$(echo "$CHANGED" | grep -iE '(test_|_test\.|\.test\.|tests/|spec/)' || true)

            if [ -z "$TEST_CHANGED" ]; then
              echo "❌ Source files modified but no test files changed or added."
              echo ""
              echo "Modified source files:"
              echo "$SRC_CHANGED"
              echo ""
              echo "Please add tests for your changes."
              exit 1
            fi

            echo "✅ Test files found in changeset."
          else
            echo "ℹ️  No source files modified — skipping test check."
          fi

  license-scan:
    name: License compatibility scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run license check
        run: |
          # Install scancode-toolkit or your preferred license scanner
          pip install scancode-toolkit --quiet

          # Scan only changed files for license issues
          CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD)

          for file in $CHANGED; do
            if [ -f "$file" ]; then
              scancode --license --only-findings --json-pp - "$file" 2>/dev/null | \
                python3 -c "
          import sys, json
          data = json.load(sys.stdin)
          for f in data.get('files', []):
              for lic in f.get('license_detections', []):
                  print(f'⚠️  {f[\"path\"]}: {lic[\"license_expression\"]}')
          " || true
            fi
          done

          echo "ℹ️  Review any license findings above manually."
```

#### PR template

Create `.github/PULL_REQUEST_TEMPLATE.md` to prompt contributors:

```markdown
## Description

<!-- Describe your changes -->

## AI Disclosure

<!-- If AI tools were used, fill in this section. Remove if not applicable. -->

- **Tool(s) used:**
- **Scope of AI assistance:**
- **Modified after generation:** Yes / No
- **Files affected:**

## Testing

- [ ] Unit tests added/updated
- [ ] Edge cases covered
- [ ] All existing tests pass

## Checklist

- [ ] Code follows project style guidelines
- [ ] I can explain every line of this PR
- [ ] No known license-incompatible code included
- [ ] Co-Authored-By trailer added to commits (if AI-assisted)
```

---

## Adoption

To adopt this guide in your project:

1. Copy this file to your repository root as `CONTRIBUTING_LLM.md`.
2. Reference it from your main `CONTRIBUTING.md`.
3. Add the GitHub Actions workflow to `.github/workflows/`.
4. Add the PR template to `.github/PULL_REQUEST_TEMPLATE.md`.
5. Communicate the new requirements to existing contributors.

---

## Recommended Tools

Enforcing these guidelines manually doesn't scale. Consider integrating an **AI-powered code review tool** into your workflow to automate part of the review process on every PR.

These tools typically provide:

- Automated line-by-line review with actionable suggestions.
- PR summaries and technical walkthroughs.
- Codebase-aware analysis that catches impact beyond the changed files.
- Conversational interface to discuss changes within the PR itself.

**Example:** [CodeRabbit](https://github.com/apps/coderabbitai) is free for open-source projects, integrates with GitHub and GitLab, and runs automatically on every new pull request. It can complement (not replace) the CI checks defined above by catching issues that static analysis misses — like logic errors, missing edge cases, or inconsistencies with the broader codebase.

When choosing a tool, evaluate based on:

- **Cost:** Free for OSS is a hard requirement for public projects.
- **Privacy:** Verify that your code is not used for model training and that reviews run in ephemeral environments.
- **Configurability:** The tool should respect your project's existing linting rules and coding standards, not impose its own.
- **Signal-to-noise ratio:** A tool that floods PRs with trivial comments will be ignored. Prefer tools that surface actionable findings.

---

## License

This document is released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) — use it, fork it, adapt it. No attribution required.

---

*Quality standards don't drop. They get automated.*
