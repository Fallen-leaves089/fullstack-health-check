# fullstack-health-check

[中文](README.md) | [English](README.en.md)

A periodic health-check skill for existing full-stack projects. It combines project-specific history, a two-axis review (standards vs. spec), and real build/container regressions, then produces a P0-P3 findings report with evidence locations.

> **License**: PolyForm Noncommercial License 1.0.0. Noncommercial use only by default; commercial use requires a written license from the author. Contact `chenshuidelaye@qq.com` to discuss terms. See [COMMERCIAL.en.md](COMMERCIAL.en.md).

> Read-only by default. Report first; do not change code without an explicit instruction.

## Install first

This is a **host-neutral** skill package, not tied to Codex, and not a double-click installer. Any host that supports the Agent Skills / `SKILL.md` directory format can use it: extract the archive and copy the entire `fullstack-health-check/` directory into that host's skills directory.

| Host | Common skills directory |
|---|---|
| Codex | `~/.codex/skills/` |
| Claude Code | `~/.claude/skills/` |
| Other Agent Skills hosts | The host's configured skills directory; check its documentation |

Windows PowerShell:

```powershell
Copy-Item -LiteralPath '.\fullstack-health-check' -Destination '<HOST_SKILLS_DIR>' -Recurse -Force
```

macOS / Linux:

```bash
cp -R ./fullstack-health-check "<HOST_SKILLS_DIR>/"
```

The directory root must contain `SKILL.md`; do not copy only that file. See "Install" below for examples.

## What it solves

Many "full-stack checks" turn into two things: reading some code, then listing generic advice. This skill enforces three things instead:

1. **Read the project's own history first** — previous findings, build baselines, environment traps, and user prohibitions are more valuable than a generic checklist.
2. **Two-axis review** — standards compliance and spec compliance are reviewed separately, so "ugly code" and "missing requirement" do not get mixed into one bucket.
3. **Real regression** — tests, builds, migrations, and compose configuration are actually run, not just read.

The output is not "please pay attention"; it is a graded list with `file:line` evidence, impact, recommendation, and verification method.

## What's inside

```text
fullstack-health-check/
├── SKILL.md / SKILL.en.md                # skill body (Chinese default / English mirror)
├── README.md / README.en.md              # this file
├── LICENSE                               # PolyForm Noncommercial 1.0.0 (English original)
├── LICENSE.zh-CN.md                      # Chinese summary, not a substitute
├── COMMERCIAL.md / COMMERCIAL.en.md      # commercial licensing
├── CONTRIBUTING.md / CONTRIBUTING.en.md  # contribution guide
├── SANITIZATION.md / SANITIZATION.en.md  # sanitization scope and audit
├── SECURITY.md                           # security reporting
├── SIGNING.md / SIGNING.en.md            # download verification and release signing
├── release-signing-key.pub               # Ed25519 release public key
├── allowed_signers                       # permitted OpenSSH signing identity
├── NOTICE                                # copyright and Required Notice
├── CHANGELOG.md                          # version history
├── .gitattributes / .gitignore           # line endings and ignore rules
├── .editorconfig                         # editor encoding and indentation
├── .github/workflows/verify.yml          # 3-OS CI
├── references/
│   ├── platform-commands.md / .en.md     # Windows / macOS / Linux / WSL2 commands
│   ├── windows-pitfalls.md / .en.md      # Windows traps and workarounds
│   ├── verification-playbook.md / .en.md # debugging and verification discipline
│   └── report-template.md / .en.md       # P0-P3 report template
└── scripts/
    ├── check_redaction.py                # cross-platform redaction scanner
    ├── verify_package.py                 # cross-platform package verifier
    ├── check-redaction.ps1               # Windows PowerShell scanner
    └── verify-package.ps1                # Windows PowerShell verifier
```

## Platform support

| Platform | Core skill | Verification |
|---|---|---|
| Windows 10/11 + PowerShell 5.1/7 | Yes | `scripts/verify-package.ps1` or `python scripts/verify_package.py` |
| macOS / Linux + Python 3.9+ | Yes | `python3 scripts/verify_package.py` |
| WSL2 | Yes | `python3 scripts/verify_package.py` or PowerShell 7 |
| Other Unix-like systems + Python 3.9+ | Yes | `python3 scripts/verify_package.py` |
| Any Agent Skills host (Codex, Claude Code, etc.) | Yes | Plain Markdown skill format |

The core skill is plain Markdown and Git commands, so it is not tied to Windows or a specific host. The Python verifier uses only the standard library and does not need network access. See `references/platform-commands.md` / `.en.md` for side-by-side Windows, macOS, Linux, and WSL2 commands; Windows-specific traps are isolated in `references/windows-pitfalls.md` / `.en.md`.

## Install

### Generic install: any Agent Skills host

This package does not depend on a Codex-specific format. A host only needs to read `SKILL.md` and allow access to `references/` and `scripts/` in the same directory.

1. Extract the release archive.
2. Find that host's skills directory.
3. Copy the entire `fullstack-health-check/` directory into it.
4. Confirm that `<SKILLS_DIR>/fullstack-health-check/SKILL.md` exists.

| Host | Common skills directory | Notes |
|---|---|---|
| Codex | `~/.codex/skills/` | Example, not the only supported host |
| Claude Code | `~/.claude/skills/` | Example |
| Other Agent Skills hosts | The host's configured skills directory | Follow the host documentation |

Windows PowerShell:

```powershell
$source = '.\fullstack-health-check'
$skills = '<HOST_SKILLS_DIR>'
New-Item -ItemType Directory -Force -Path $skills | Out-Null
Copy-Item -LiteralPath $source -Destination $skills -Recurse -Force
Test-Path (Join-Path $skills 'fullstack-health-check\SKILL.md')
```

macOS / Linux:

```bash
mkdir -p "<HOST_SKILLS_DIR>"
cp -R ./fullstack-health-check "<HOST_SKILLS_DIR>/"
test -f "<HOST_SKILLS_DIR>/fullstack-health-check/SKILL.md"
```

### Example: Codex

```bash
mkdir -p ~/.codex/skills
cp -R ./fullstack-health-check ~/.codex/skills/
```

On Windows, use `$env:USERPROFILE\.codex\skills` as the destination.

### Example: Claude Code

```bash
mkdir -p ~/.claude/skills
cp -R ./fullstack-health-check ~/.claude/skills/
```

On Windows, use `$env:USERPROFILE\.claude\skills` as the destination.

### Other hosts

Put the whole directory in the skills directory configured by that host. Directory names, loading behavior, and whether a restart is required vary by host; follow its documentation. Do not assume every host uses `.codex` or `.claude`.

### Git clone

```bash
git clone https://github.com/Fallen-leaves089/fullstack-health-check.git "<HOST_SKILLS_DIR>/fullstack-health-check"
```

### Manual install

After extracting, copy the entire `fullstack-health-check/` directory into the skills directory. Do not copy only `SKILL.md`; `references/` and `scripts/` must travel with it.

### Use the English skill body

Skill hosts load `SKILL.md` (Chinese) by default. To use the English version, replace `SKILL.md` in your installed skill directory with `SKILL.en.md`:

```bash
cp SKILL.en.md SKILL.md
```

Keep both files in the repository so readers can compare them.

## Download and signatures

Download official artifacts from [GitHub Releases](https://github.com/Fallen-leaves089/fullstack-health-check/releases) and get all of these files:

- `fullstack-health-check-<version>.zip`
- `SHA256SUMS`
- `SHA256SUMS.sig`
- `allowed_signers`
- `release-signing-key.pub`

Check the public-key fingerprint first:

```bash
ssh-keygen -lf release-signing-key.pub
```

macOS (bundled `shasum`):

```bash
shasum -a 256 -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers -I fullstack-health-check-release -n file -s SHA256SUMS.sig < SHA256SUMS
```

Linux (GNU coreutils):

```bash
sha256sum -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers -I fullstack-health-check-release -n file -s SHA256SUMS.sig < SHA256SUMS
```

Windows PowerShell 5.1 / 7:

```powershell
$zip = '.\fullstack-health-check-<version>.zip'
$line = Get-Content .\SHA256SUMS | Where-Object { $_ -match [regex]::Escape([IO.Path]::GetFileName($zip)) } | Select-Object -First 1
if (-not $line) { throw 'Checksum entry not found' }
$expected = ($line -split '\s+')[0].ToLowerInvariant()
$actual = (Get-FileHash -LiteralPath $zip -Algorithm SHA256).Hash.ToLowerInvariant()
if ($actual -ne $expected) { throw 'SHA256 mismatch' }

cmd /c "ssh-keygen.exe -Y verify -f allowed_signers -I fullstack-health-check-release -n file -s SHA256SUMS.sig < SHA256SUMS"
if ($LASTEXITCODE -ne 0) { throw 'Signature verification failed' }
```

See `SIGNING.en.md` for the complete instructions. `SHA256SUMS` and `SHA256SUMS.sig` are separate assets created after the zip; they are not inside the zip.

## Triggers

Say any of these:

- "Run a full-stack health check"
- "Check this full-stack project"
- "Pre-release audit"
- "Find P0/P1 issues before deployment"
- "Use fullstack-health-check on this repo"

## Requirements

- The project uses Git. The skill validates the baseline with `git rev-parse` and stops if it cannot resolve.
- Docker must be available if you want containerized migration/regression checks.
- The review phase is read-only by default; code changes require an explicit instruction.
- For Windows-specific troubleshooting, read `references/windows-pitfalls.en.md` first.

## Self-check

From the package root:

```bash
python3 scripts/verify_package.py
```

On Windows:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\verify-package.ps1
```

Or with Python on Windows:

```powershell
python scripts\verify_package.py
```

The verifier checks:

- required files are present
- text files are UTF-8 without BOM
- relative Markdown links resolve
- the redaction scanner passes its positive/negative self-test
- the package has no personal paths, private keys, tokens, or private IPs
- `SKILL.md` / `SKILL.en.md` frontmatter and language links are present

Run only the redaction scanner:

```bash
python3 scripts/check_redaction.py --self-test
python3 scripts/check_redaction.py --path .
```

## Sanitization

This public version was derived from an internal skill and rewritten to remove personal absolute paths, usernames, project/client identifiers, internal platform names, real domains and private addresses, secrets, and incident-specific data.

The review scope is documented in `SANITIZATION.en.md`. If you find a possible leak, open an issue titled `[Security]` and describe only the file and the type of information. Do not paste the sensitive value.

## Contributing

See `CONTRIBUTING.en.md`. New environment traps should include "trap / workaround / verification"; new guard scripts must include positive and negative cases.

## License

This work is licensed under the **PolyForm Noncommercial License 1.0.0**. The English original is in `LICENSE`; the Chinese summary is in `LICENSE.zh-CN.md`.

Commercial use requires a written license from the author, but **the author is open to discussing it**. For commercial licensing, contact `chenshuidelaye@qq.com`; terms and scope are negotiable. See `COMMERCIAL.en.md`.

> GitHub's license detector does not list PolyForm, so the repository page may show `NOASSERTION / Other`. That does not mean the package is unlicensed; the `LICENSE` text controls.

## Version

- Version: 1.1.2
- Internal source date: 2026-09-04
- Public release: 2026-10-29
