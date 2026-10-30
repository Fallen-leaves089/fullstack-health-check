# project-health

[中文](README.md) | [English](README.en.md)

一个面向已有软件项目的证据驱动体检技能包：先查项目自己的历史归档，再做「规范 / 规格」双轴审查，最后跑真实构建、数据库迁移、容器化回归和关键路径验证，输出 P0-P3 分级问题清单。适用于服务、Web / 移动客户端、数据库、容器与部署等不同形态的项目；全栈只是其中一种常见场景。

> **许可 / License**：PolyForm Noncommercial License 1.0.0。默认仅限非商业用途；商业使用需取得作者书面授权，可联系 `chenshuidelaye@qq.com` 协商。详见 [COMMERCIAL.md](COMMERCIAL.md)。

> 检查阶段默认只读。发现问题先报告，不擅自改代码。

## 先安装

这是一个通用 Skill 包，**不绑定 Codex**，也不是双击安装的软件。只要宿主支持 Agent Skills / `SKILL.md` 目录格式，就可以安装：解压后把整个 `project-health/` 目录复制到该宿主的技能目录。

| 宿主 | 常见技能目录 |
|---|---|
| Codex | `~/.codex/skills/` |
| Claude Code | `~/.claude/skills/` |
| 其他 Agent Skills 宿主 | 宿主配置的技能目录，以宿主文档为准 |

Windows PowerShell：

```powershell
Copy-Item -LiteralPath '.\project-health' -Destination '<宿主技能目录>' -Recurse -Force
```

macOS / Linux：

```bash
cp -R ./project-health "<宿主技能目录>/"
```

装好后目录根部必须有 `SKILL.md`，不能只复制这一个文件。具体示例见下方「安装」。

> **从 1.1.x 升级**：本项目已更名为 `project-health`。请先删除旧的 `fullstack-health-check` 技能目录，再安装新目录，避免两个技能同时存在。

## 它解决什么问题

很多“项目检查”最后变成两件事：读一遍代码，然后列一堆正确的废话。这个技能包强制做三件事：

1. **先查项目自己的历史**：上一轮遗留、构建基线、环境坑、用户禁区，比通用清单更值钱。
2. **双轴审查**：规范符合度（Standards）和需求符合度（Spec）分开看，避免把“代码写得丑”和“需求没做完”混成一锅。
3. **真实回归**：测试、构建、迁移、compose 配置都要跑，不只做静态阅读。

输出不是“建议关注”，而是带 `file:line` 证据、影响、建议和验证方式的分级清单。

它还会检查“检查方法本身是否有效”，并约束跨平台命令差异、安全底线、脱敏和修复边界；目标不是多列几条意见，而是让结论可复现、可追溯。

## 包内结构

```text
project-health/
├── SKILL.md / SKILL.en.md                # 技能本体（中文默认 / 英文镜像）
├── README.md / README.en.md              # 本文件
├── LICENSE                               # PolyForm Noncommercial 1.0.0 英文原文
├── LICENSE.zh-CN.md                      # 中文说明，不替代英文原文
├── COMMERCIAL.md / COMMERCIAL.en.md      # 商业授权协商说明
├── CONTRIBUTING.md / CONTRIBUTING.en.md  # 贡献指南与贡献授权
├── SANITIZATION.md / SANITIZATION.en.md  # 脱敏范围、保留内容与自检记录
├── SECURITY.md                           # 安全问题报告方式
├── SIGNING.md / SIGNING.en.md            # 下载校验与发布签名说明
├── release-signing-key.pub               # Ed25519 发布公钥
├── allowed_signers                       # OpenSSH 允许签名身份
├── NOTICE                                # 版权与 Required Notice
├── CHANGELOG.md                          # 版本记录
├── .gitattributes / .gitignore           # 换行符与忽略规则
├── .editorconfig                         # 编辑器编码与缩进约定
├── .github/workflows/verify.yml          # Windows / macOS / Linux CI
├── references/
│   ├── platform-commands.md / .en.md     # Windows / macOS / Linux / WSL2 命令对照
│   ├── windows-pitfalls.md / .en.md      # Windows 环境坑与绕法
│   ├── verification-playbook.md / .en.md # 纠错、定位与正反例验证
│   └── report-template.md / .en.md       # P0-P3 报告模板
└── scripts/
    ├── check_redaction.py                # 跨平台脱敏扫描器
    ├── verify_package.py                 # 跨平台总检
    ├── check-redaction.ps1               # Windows PowerShell 扫描器
    └── verify-package.ps1                # Windows PowerShell 总检
```

## 平台支持

| 平台 | 核心技能 | 自检方式 |
|---|---|---|
| Windows 10/11 + PowerShell 5.1/7 | 支持 | `scripts/verify-package.ps1` 或 `python scripts\verify_package.py` |
| macOS / Linux + Python 3.9+ | 支持 | `python3 scripts/verify_package.py` |
| WSL2 | 支持 | `python3 scripts/verify_package.py` 或 PowerShell 7 |
| 其他类 Unix 系统 + Python 3.9+ | 支持 | `python3 scripts/verify_package.py` |
| 任意 Agent Skills 宿主（Codex、Claude Code 等） | 支持 | 纯 Markdown 技能格式 |

核心技能只依赖 Markdown 和 Git 命令，不绑定 Windows，也不绑定某个特定宿主；Python 自检只用标准库，不需要联网安装依赖。常见操作的三平台写法见 `references/platform-commands.md` / `.en.md`；Windows 专用坑集中在 `references/windows-pitfalls.md` / `.en.md`，其他平台可以跳过。

## 安装

### 通用安装：任意 Agent Skills 宿主

本包不依赖 Codex 专有格式。宿主只要读取 `SKILL.md`，并能访问同目录的 `references/` 和 `scripts/`，就可以使用。

1. 解压发布包。
2. 确认该宿主的技能目录。
3. 把整个 `project-health/` 目录复制进去。
4. 确认 `<技能目录>/project-health/SKILL.md` 存在。

| 宿主 | 常见技能目录 | 说明 |
|---|---|---|
| Codex | `~/.codex/skills/` | 示例，不是唯一支持宿主 |
| Claude Code | `~/.claude/skills/` | 示例 |
| 其他 Agent Skills 宿主 | 宿主配置的技能目录 | 以宿主文档为准 |

Windows PowerShell：

```powershell
$source = '.\project-health'
$skills = '<宿主技能目录>'
New-Item -ItemType Directory -Force -Path $skills | Out-Null
Copy-Item -LiteralPath $source -Destination $skills -Recurse -Force
Test-Path (Join-Path $skills 'project-health\SKILL.md')
```

macOS / Linux：

```bash
mkdir -p "<宿主技能目录>"
cp -R ./project-health "<宿主技能目录>/"
test -f "<宿主技能目录>/project-health/SKILL.md"
```

### 示例：Codex

```bash
mkdir -p ~/.codex/skills
cp -R ./project-health ~/.codex/skills/
```

Windows 下把目标改成 `$env:USERPROFILE\.codex\skills`。

### 示例：Claude Code

```bash
mkdir -p ~/.claude/skills
cp -R ./project-health ~/.claude/skills/
```

Windows 下把目标改成 `$env:USERPROFILE\.claude\skills`。

### 其他宿主

把整个目录放进宿主配置的技能目录即可。不同宿主的目录名、加载方式和是否需要重启都不同，以宿主文档为准；不要假设所有宿主都使用 `.codex` 或 `.claude`。

### Git clone

```bash
git clone https://github.com/Fallen-leaves089/project-health.git "<宿主技能目录>/project-health"
```

### 手动安装

解压后把 `project-health/` 整个目录复制到技能目录，不要只复制 `SKILL.md`——`references/` 和 `scripts/` 也要一起带走。

### 使用英文技能本体

技能宿主默认加载 `SKILL.md`（中文）。如需英文版，把安装目录中的 `SKILL.md` 替换为 `SKILL.en.md`：

```bash
cp SKILL.en.md SKILL.md
```

仓库同时保留两份文件，方便对照和更新。

## 下载与签名

正式下载请从 [GitHub Releases](https://github.com/Fallen-leaves089/project-health/releases) 获取，并同时下载：

- `project-health-<version>.zip`
- `SHA256SUMS`
- `SHA256SUMS.sig`
- `allowed_signers`
- `release-signing-key.pub`

先核对公钥指纹：

```bash
ssh-keygen -lf release-signing-key.pub
```

macOS 自带 `shasum`：

```bash
shasum -a 256 -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers -I project-health-release -n file -s SHA256SUMS.sig < SHA256SUMS
```

Linux（GNU coreutils）：

```bash
sha256sum -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers -I project-health-release -n file -s SHA256SUMS.sig < SHA256SUMS
```

Windows PowerShell 5.1 / 7：

```powershell
$zip = '.\project-health-<version>.zip'
$line = Get-Content .\SHA256SUMS | Where-Object { $_ -match [regex]::Escape([IO.Path]::GetFileName($zip)) } | Select-Object -First 1
if (-not $line) { throw 'Checksum entry not found' }
$expected = ($line -split '\s+')[0].ToLowerInvariant()
$actual = (Get-FileHash -LiteralPath $zip -Algorithm SHA256).Hash.ToLowerInvariant()
if ($actual -ne $expected) { throw 'SHA256 mismatch' }

cmd /c "ssh-keygen.exe -Y verify -f allowed_signers -I project-health-release -n file -s SHA256SUMS.sig < SHA256SUMS"
if ($LASTEXITCODE -ne 0) { throw 'Signature verification failed' }
```

完整说明见 `SIGNING.md`。`SHA256SUMS` 和 `SHA256SUMS.sig` 是 zip 生成后的独立发布资产，不放进 zip 内部。

## 触发方式

对话里说这些会命中：

- 「做个项目体检」
- 「全面问题检查」
- 「上线前审计」或「上线前体检」
- 「项目交接前检查」
- 「检查这个软件项目」或「检查全栈项目」

也可以直接说「用 project-health 检查这个仓库」。

## 使用前提

- 项目用 git；技能会先 `git rev-parse` 校验基准点，解析失败就地停下。
- 需要跑容器化回归时，Docker 守护进程要能启动。
- 检查阶段默认只读；要修代码，等用户明确指令。
- 需要 Windows 专项排障时，先读 `references/windows-pitfalls.md`，不要凭记忆试。

## 自检

在包根目录运行（推荐，跨平台）：

```bash
python3 scripts/verify_package.py
```

Windows 也可以运行：

```powershell
python scripts\verify_package.py
```

PowerShell 版（Windows）：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\verify-package.ps1
```

> 非 Windows 环境如已安装 PowerShell 7，可把 `powershell` 换成 `pwsh`。

它会检查：

- 必需文件是否齐全
- 文本文件是否为 UTF-8 无 BOM
- Markdown 相对链接是否可解析
- 脱敏扫描器的正反例自检是否通过
- 包内是否残留个人路径、私钥、令牌、内网 IP 等敏感模式
- `SKILL.md` / `SKILL.en.md` 的 frontmatter、版本和语言链接是否一致

只跑脱敏扫描（跨平台）：

```bash
python3 scripts/check_redaction.py --self-test
python3 scripts/check_redaction.py --path .
```

Windows PowerShell 版：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\check-redaction.ps1 -SelfTest
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\check-redaction.ps1 -Path .
```

## 脱敏说明

这个公开版从一份内部技能整理而来，已移除个人绝对路径、用户名、项目/客户标识、内部平台名称、真实域名与内网地址、密钥和事故细节。保留下来的是可复用的方法、Windows 环境坑和纠错纪律。英文文档是中文原文的翻译，不引入新的内部信息。

具体移除与保留范围见 `SANITIZATION.md`。如果你发现仍有可疑信息，请通过仓库 Issue 或维护者公开渠道联系，不要直接贴出敏感值。

## 参与贡献

见 `CONTRIBUTING.md`。新增环境坑请写清“坑 / 绕法 / 验证方式”；新增守护脚本必须自带正反例。

## 许可

本作品采用 **PolyForm Noncommercial License 1.0.0**，仅允许非商业用途。英文原文见 `LICENSE`，中文说明见 `LICENSE.zh-CN.md`。

商业使用需要作者书面授权，但**可以和作者商量**。如需商业化，可以联系 `chenshuidelaye@qq.com`；授权方式、范围和费用按实际使用场景协商，见 `COMMERCIAL.md`。

> GitHub 的许可证识别列表不包含 PolyForm，因此仓库页面可能显示 `NOASSERTION / Other`。这不代表没有许可证；以仓库中的 `LICENSE` 英文原文为准。

## 版本

- 版本：1.2.0
- 对应内部整理日期：2026-09-04
- 公开发布整理：2026-10-30
