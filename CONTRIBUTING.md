# Contributing

Thanks for considering a contribution. This repo has a clear, consistently applied merge policy — the maintainer has stated it across merge reviews ([#30], [#43], [#54]), and this document collects it in one place so you can tell *before* building whether something will land.

## The policy in one sentence

> "This repo's policy is to stay a universal, market- and person-agnostic template — country-specific portal skills and personal-profile changes get declined (see [#31], [#17]). What gets in are features that make the fork-and-customize path better for everyone." *([#30] merge review)*

Your fork is where personalization lives — your profile, your market's job portals, your templates. Upstream is the machinery that makes any fork work.

## What gets merged

Every merged feature so far fits one of these shapes:

- **Generators over instances.** A command that helps every forker build their own market-specific thing beats the thing itself. `/add-portal` ([#37]) — a country-agnostic generator for portal skills — is what got merged; individual country portals ([#31]) are what got declined. Same logic for `/add-template` ([#30]) vs. shipping more templates.
- **Bridges that connect what exists without modifying it.** `/rank` ([#43]) connects `/scrape` to `/apply`; `/outcome` ([#54]) writes the archive `/setup` already read; `/interview` ([#58]) consumes that archive. As the [#43] merge review put it: *"fills the real gap between /scrape and /apply without touching either."*
- **Verification infrastructure.** The ATS text-layer check ([#40]), the CI workflow ([#59]). Anything that catches broken output before a user (or the maintainer) has to.
- **Bug fixes with a demonstrated failure**, not a suspected one ([#35], [#55], [#56], the `\closing{}` fix in [#59]).

## What gets declined

- **Country- or market-specific skills** ([#31]) — build them with `/add-portal` and keep them in your fork. If your market needs something the generator can't produce, improve the generator.
- **Personal-profile content** ([#17]) — anything that would bake one person's data, sector, or preferences into the template. CI's placeholder-integrity job enforces this mechanically.
- **Features that add a dependency the workflow can't degrade without.** Optional tools follow the graceful-skip pattern (`salary_lookup.py`, `pdftotext` in the ATS check): if it's missing, the step warns and continues.

## Design rules that earn merges

These are drawn directly from what merge reviews have called out as the reason for merging:

1. **Respect ownership boundaries.** One owner per file or state. If your command produces data another command interprets, hand off — don't duplicate the interpreting logic (`/outcome` writes, `/setup` calibrates).
2. **Additive schema changes only.** New statuses and fields extend existing schemas without restructuring them, and the change lands in the same file that documents the schema (`seen_jobs.json` statuses, the `outcome.md` enum).
3. **Explicit failure paths, never guesses.** A dead URL becomes `expired` or an "unavailable" stub — not content reconstructed from memory. A missing optional tool becomes a warning and a documented fallback.
4. **Idempotent re-runs.** Running a command twice appends or updates; it never duplicates folders, rows, or history.
5. **The honesty rule.** Nothing in this framework fabricates: skill gaps are acknowledged, keywords are never stuffed, postings are never invented, scores aren't bent. If your feature generates user-facing content, carry this rule through it.

## Before you build

- **Open an issue first for anything substantial.** Recent contributors have asked "would this be welcome?" before building ([#62]) — that's the right instinct, and it protects your time. Small fixes can go straight to PR.
- **Check open PRs and issues** so you don't build something already claimed.

## PR expectations

- **Make your claims verifiable.** PRs here are verified line-by-line before merging — weights checked against the framework files, schema claims checked against the documented schemas. Write the PR body so each claim maps to a file, a line, or a CI run someone can check. A claim that survives verification is worth more than a paragraph of description.
- **CI must pass.** The workflow (`.github/workflows/ci.yml`) runs LaTeX smoke compiles, the skill/command lint, and CLI typechecks on every PR. Two jobs (placeholder integrity, exact page counts) run only on this repo, so don't be surprised when they skip on your fork.
- **No personal data.** Tracked template files keep their `[PLACEHOLDER]` tokens; personal outputs (`cv/main_*.tex`, `documents/applications/**`, the tracker) are gitignored — never weaken those rules to get something committed.
- **Keep diffs single-purpose.** One feature or fix per PR, with docs updated in the same PR (README command list and file tree, plus whichever schema-documenting file your change touches).
- **Commit style:** `feat:` / `fix:` / `docs:` / `ci:` prefixes, imperative subject.

## Local checks (what CI will run)

```bash
# Lint skills, commands, settings.json
python tools/lint_skills.py

# LaTeX smoke tests (CV must be exactly 2 pages, cover letter exactly 1)
cd cv && lualatex -interaction=nonstopmode main_example.tex && cd ..
cd cover_letters && xelatex -interaction=nonstopmode cover_example.tex && cd ..

# Typecheck a portal CLI
cd .agents/skills/<tool>/cli && bun install && bun run typecheck
```

**Live portal requests stay out of CI and out of automated tests** — they're network-flaky, and `linkedin-search` is personal-use-only per its own ToS warning. Test portal CLIs live only locally and sparingly.

## Conventions

- **Commands** live in `.claude/commands/<name>.md`, start with a `# /<name> - Title` line, and are written as ordered steps with explicit rules sections. Read `apply.md` or `outcome.md` for the house style.
- **Skills** live in `.claude/skills/<name>/` (workflow skills) or `.agents/skills/<name>/` (portal CLI skills) with YAML frontmatter: `name`, a `description` written for trigger matching (include local-language trigger phrases for market skills), and `allowed-tools` scoped as narrowly as possible.
- **Portal skills** follow the contract in `.claude/commands/add-portal.md`: `search`/`detail` commands, the shared flag set, `{ meta, results }` JSON output, errors to stderr as `{ "error", "code" }` with exit 1, backoff on 429/5xx, zero runtime dependencies by default.
- **Permissions** (`.claude/settings.json`) stay narrow — one scoped entry per tool, in the spirit of [#27].

[#17]: https://github.com/MadsLorentzen/ai-job-search/issues/17
[#27]: https://github.com/MadsLorentzen/ai-job-search/issues/27
[#30]: https://github.com/MadsLorentzen/ai-job-search/issues/30
[#31]: https://github.com/MadsLorentzen/ai-job-search/issues/31
[#35]: https://github.com/MadsLorentzen/ai-job-search/issues/35
[#37]: https://github.com/MadsLorentzen/ai-job-search/issues/37
[#40]: https://github.com/MadsLorentzen/ai-job-search/issues/40
[#43]: https://github.com/MadsLorentzen/ai-job-search/issues/43
[#54]: https://github.com/MadsLorentzen/ai-job-search/issues/54
[#55]: https://github.com/MadsLorentzen/ai-job-search/issues/55
[#56]: https://github.com/MadsLorentzen/ai-job-search/issues/56
[#58]: https://github.com/MadsLorentzen/ai-job-search/issues/58
[#59]: https://github.com/MadsLorentzen/ai-job-search/issues/59
[#62]: https://github.com/MadsLorentzen/ai-job-search/issues/62
