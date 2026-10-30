---
name: project-health
description: Evidence-driven health checks for existing software projects: history-first review, standards/spec two-axis audit, real build/migration/container regressions, and explicit verification, platform, security, and redaction boundaries, ending in an evidence-linked P0-P3 findings report. Use for periodic reviews, pre-release audits, project handovers, and whole-project diagnosis.
version: 1.2.0
license: PolyForm-Noncommercial-1.0.0
---

# Project Health Check

[中文](SKILL.md) | [English](SKILL.en.md)

> **License**: PolyForm Noncommercial License 1.0.0. Noncommercial use only by default; commercial use requires a written license from the author. Contact `chenshuidelaye@qq.com` to discuss terms. See [COMMERCIAL.en.md](COMMERCIAL.en.md).

> This is the English mirror. The default `SKILL.md` is Chinese; to use this version, copy it over `SKILL.md` in your skill directory.

Run a periodic health check on an existing software project and produce a **graded, evidence-linked, checkable** findings report. It works for services, web and mobile clients, database migrations, containerized deployment, and other project layers; full-stack is one common case, not the boundary of the skill. The workflow also covers verification discipline, platform command differences, security boundaries, redaction, and repair boundaries.

> The core workflow is platform-neutral. See `references/platform-commands.en.md` for side-by-side commands; Windows-only commands and traps live in `references/windows-pitfalls.en.md`, and macOS, Linux, and WSL2 can skip that section.

## Iron rules (higher priority than the rest of this file)

1. **Report before acting.** State what you understand, which rules you plan to use, what prerequisites are missing, and where the blockers are. Give the user the options and let them decide.
2. **Read-only during the review.** Report findings only (evidence + impact + recommendation). **Never change code on your own.** Wait for an explicit instruction.
3. **Project history outranks external rule packs.** Historical reports, `AGENTS.md`, `docs/`, change logs, and previous findings are closer to the truth than a generic checklist. If they conflict, ask the user.
4. **Do not invent methodology or fixes.** Prefer fixes that already exist in the project. Annotate the source when you reuse one.
5. **Put deliverables in the user-specified directory.** Desktop by default, named `description_date.md`; do not bury them in `docs/`.

## Workflow

### 0. Read the project's history first

Read, in order:

- `AGENTS.md` / `CLAUDE.md` (repository conventions and prohibitions)
- historical check reports, remediation logs, and ADRs under `docs/`
- the project's agent memory directory, if present
- previous findings, build baselines, environment traps, and user prohibitions

**This saves a lot of repeated trial and error.** If there is no history, say so; do not pretend there is.

### 1. Align with the user

Ask at least:

- Which rules: project conventions / external rule pack / both
- Baseline: which commit, branch, or full tree
- Uncommitted changes: commit first / keep read-only / handle after the review
- Permission to start Docker, databases, emulators, or other heavy resources

If the host does not support interactive questions, choose the most conservative option and state the assumption at the top of the report.

### 2. Pin the baseline

```bash
git rev-parse <fixed-point>           # must resolve; stop here if it does not
git log <fixed-point>..HEAD --oneline
git diff <fixed-point>...HEAD --stat  # three-dot: compare against merge-base
```

If the diff is empty or the ref is missing, stop and ask the user. Do not discover a bad baseline after starting subagents or builds.

### 3. Two-axis review (when using an external rule pack)

Run two **parallel** subagents, isolated from each other:

- **Standards axis**: coding-standard compliance + code-smell baseline. When the repository has no documented standard, the smell baseline is the de facto standard. Treat every item as an **opinion**, not a hard violation.
- **Spec axis**: compare against the spec / PRD / issue. Find "required but missing", "added but not requested", and "implemented incorrectly". If the spec is missing, say so; do not invent requirements.

Subagent prompts must be self-contained and explicitly **read-only**: no file changes, no `git commit`, no build that writes to the repository.
**Verify subagent claims** — a subagent's summary describes what it intended to do, not necessarily what it actually did.

If the host does not support subagents, run the two axes sequentially. Do not merge them into a vague "look at the code".

### 4. Real build and regression (do not stop at static review)

Run the project's baseline items and record the numbers:

- backend test suite
- frontend build + artifact checks (API base URL, debug flags, sourcemaps)
- mobile builds for every flavor
- containerized idempotency regression for database migrations
- compose / deployment configuration validation
- critical-path smoke tests

If a build fails, decide whether it is an **environment problem** or a **code problem**. Record the workaround for environment problems; do not automatically grade them as P0.

### 5. Graded output

`P0 breaks on deployment day` / `P1 must fix before deployment` / `P2 recommended before deployment` / `P3 hygiene and maintainability`.

For every item, include **evidence location (`file:line`) + impact + recommendation + verification method**.

Add separate sections for "verified OK" and "not covered in this round". Do not pretend to have full coverage.

Report template: `references/report-template.en.md`.

### 6. Fix only after an explicit instruction

- Touch only the named items. **Do not modify things that were already fine.**
- Before each fix, ask: will this break a flow that is still in use? Can we keep both behaviors, e.g. preserve the capability behind an explicit switch?
- Do not rewrite historical documents; add a dated header note instead.
- Re-run the regression after the fix.
- Keep the fix commit separate from the review report. Do not mix "found the problem" and "changed the code" in one unrecoverable commit.

## Verification discipline (most important)

**Every guard script must be tested with a positive and a negative case.** A guard that only reports green is not a guard.

Criteria:

- For an input that should fail, it must fail.
- For an input that should pass, it must pass.
- Both must be verified.

Choose the negative case carefully: a sample that "also fails for another reason" proves nothing. For example, using an old version that also has a syntax error to verify "seed accounts are detected" only proves the verification is invalid.

**Do not swallow the exit status of the command under test** with `>/dev/null 2>&1` or a pipeline. "The command crashed" and "the command finished cleanly" become indistinguishable, which is a common source of false negatives. In PowerShell, capture `$LASTEXITCODE` before formatting or filtering.

See `references/verification-playbook.en.md` for the full debugging and verification workflow.

## Platform command reference (identify the shell first)

The same task uses different commands on different systems. Do not put PowerShell syntax into bash/zsh, and do not copy Unix paths directly into Windows commands. See `references/platform-commands.en.md` for the full guide; common differences:

| Operation | Windows PowerShell | macOS / Linux / WSL2 |
|---|---|---|
| Current directory | `Get-Location` | `pwd` |
| Environment variable | `$env:NAME` | `$NAME` / `printenv NAME` |
| Python | `py -3` or `python` | `python3` |
| venv interpreter | `.\.venv\Scripts\python.exe` | `.venv/bin/python` |
| Command exists | `Get-Command` / `where.exe` | `command -v` |
| Inspect a port | `Get-NetTCPConnection -LocalPort <port>` | `ss -ltnp` / `lsof -iTCP:<port>` |

WSL2 uses Linux commands. Windows and WSL differ in paths, line endings, and how Docker starts; state the actual shell and interpreter in the report.

## Windows traps (check before trying)

The rest of this section is Windows-only. On other platforms, use the equivalent commands for the current shell; do not force PowerShell syntax into bash or zsh.

Full list: `references/windows-pitfalls.en.md`. High-frequency items:

| Trap | Workaround |
|---|---|
| `mvn` fails or behaves oddly in Git Bash | Run `mvn.cmd` from PowerShell |
| `mvn.cmd -q test` prints no summary | Parse `target/surefire-reports/*.xml`; the key is `tests=` (with the equals sign) |
| `python` is not on PATH in Git Bash | `where.exe python`, `py -3`, or `Get-Command python`; do not hard-code a user directory |
| `file` is not available | Use `Format-Hex` or Python bytes to detect encoding/BOM |
| Pipeline loses the exit code | Capture `$LASTEXITCODE` first, then `Select-String` / `Format-Table` |
| Recursive delete is blocked | Verify the absolute path; `Move-Item` to a temp directory, or use precise `Remove-Item -LiteralPath` |
| Docker daemon is not running | Start Docker Desktop and poll `docker info`; `docker compose config` does not need the daemon |
| `git mv` keeps the old blob | Edit the content first, then `git mv`; inspect the second column of `git status --short` |
| shell libraries use `${BASH_SOURCE[1]}` | It degenerates under `bash -c`; use `${BASH_SOURCE[0]}` |
| mysql:8.0 initialization starts two mysqld processes | Require N consecutive successful probes; reset the count on a drop |
| Cleaning test artifacts deletes placeholders | Delete exact files; run `git ls-files <dir>` first |

## Security baseline

**Never trust the current working directory (cwd).** After `cd`, validate a directory fingerprint (required files/directories). If anything is missing, fail closed.

Why: `docker compose` can operate on the host as root. Loading an untrusted compose file from the wrong directory with `volumes: - /:/host` plus a chroot is **executing the attacker's script**, not "a failed deployment".

Prefer stopping over silently going off track. Apply the same posture to environment-variable validation, artifact checks, empty-list failures, and `${VAR:?message}`.

## Redaction and privacy

- Redact commands, logs, packet captures, and configuration before showing them. Replace keys, tokens, cookies, private keys, and connection strings with `<REDACTED>`.
- Do not read or display the contents of `auth.json`, `.env`, SSH private keys, or similar files. Report only key names, existence, and permissions.
- Redaction must cover JSON, TOML, YAML, ENV, and command-line arguments. Replacing only `api_key:` will miss `"API_KEY": "..."`.
- If a real secret reaches the session or a report, stop spreading it, tell the user to rotate it, and keep only the key name in the report.
- Replace client names, real domains, private IPs, store/account identifiers with `<PROJECT>`, `<HOST>`, `<ACCOUNT>` as needed.

## Wrap-up

- Update the user-specified summary (desktop by default) and the in-repo report; keep their status in sync.
- Re-check line numbers after adding comments or moving files.
- Comments and code must agree (a comment claiming a "one-time **random** password" was actually a fixed value).
- Before committing: check for mojibake, UTF-8 without BOM, and `git diff --check`.
- After committing: confirm the worktree is clean.
