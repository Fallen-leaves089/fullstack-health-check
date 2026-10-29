# fullstack-health-check

一个面向已有项目的全栈体检技能包：先查项目自己的历史归档，再做「规范 / 规格」双轴审查，最后跑真实构建、数据库迁移和容器化回归，输出 P0-P3 分级问题清单。

> 检查阶段默认只读。发现问题先报告，不擅自改代码。

## 它解决什么问题

很多“全栈检查”最后变成两件事：读一遍代码，然后列一堆正确的废话。这个技能包强制做三件事：

1. **先查项目自己的历史**：上一轮遗留、构建基线、环境坑、用户禁区，比通用清单更值钱。
2. **双轴审查**：规范符合度（Standards）和需求符合度（Spec）分开看，避免把“代码写得丑”和“需求没做完”混成一锅。
3. **真实回归**：测试、构建、迁移、compose 配置都要跑，不只做静态阅读。

输出不是“建议关注”，而是带 `file:line` 证据、影响、建议和验证方式的分级清单。

## 包内结构

```text
fullstack-health-check/
├── SKILL.md                              # 技能本体，宿主加载这个
├── README.md                             # 本文件
├── LICENSE                               # PolyForm Noncommercial 1.0.0 英文原文
├── LICENSE.zh-CN.md                      # 中文说明，不替代英文原文
├── COMMERCIAL.md                         # 商业授权协商说明
├── NOTICE                                # 版权与 Required Notice
├── SANITIZATION.md                       # 脱敏范围、保留内容与自检记录
├── CHANGELOG.md                          # 版本记录
├── CONTRIBUTING.md                       # 贡献指南与贡献授权
├── references/
│   ├── windows-pitfalls.md               # Windows 环境坑与绕法
│   ├── verification-playbook.md          # 纠错、定位与正反例验证
│   └── report-template.md                # P0-P3 报告模板
└── scripts/
    ├── check-redaction.ps1               # 脱敏扫描器，自带正反例自检
    └── verify-package.ps1                # 包完整性、编码、链接、脱敏总检
```

## 安装

### Codex / 兼容 Agent Skills 的宿主

PowerShell：

```powershell
$skills = Join-Path $env:USERPROFILE '.codex\skills'
New-Item -ItemType Directory -Force -Path $skills | Out-Null
Copy-Item -LiteralPath '.\fullstack-health-check' -Destination $skills -Recurse -Force
```

装完后技能目录应包含 `SKILL.md`。

### Claude Code 或其他技能目录

把整个 `fullstack-health-check/` 目录放进宿主的技能目录即可，例如 `~/.claude/skills/`。不同宿主的目录名不同，以宿主文档为准。

### 手动安装

解压后把 `fullstack-health-check/` 整个目录复制到技能目录，不要只复制 `SKILL.md`——`references/` 和 `scripts/` 也要一起带走。

## 触发方式

对话里说这些会命中：

- 「做个全栈检查」
- 「全面问题检查」
- 「上线前体检」
- 「检查全栈项目」

也可以直接说「用 fullstack-health-check 检查这个仓库」。

## 使用前提

- 项目用 git；技能会先 `git rev-parse` 校验基准点，解析失败就地停下。
- 需要跑容器化回归时，Docker 守护进程要能启动。
- 检查阶段默认只读；要修代码，等用户明确指令。
- 需要 Windows 专项排障时，先读 `references/windows-pitfalls.md`，不要凭记忆试。

## 自检

在包根目录运行：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\verify-package.ps1
```

> 非 Windows 环境把 `powershell` 换成 `pwsh`。

它会检查：

- 必需文件是否齐全
- 文本文件是否为 UTF-8 无 BOM
- Markdown 相对链接是否可解析
- 脱敏扫描器的正反例自检是否通过
- 包内是否残留个人路径、私钥、令牌、内网 IP 等敏感模式

只跑脱敏扫描：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\check-redaction.ps1 -SelfTest
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\check-redaction.ps1 -Path .
```

## 脱敏说明

这个公开版从一份内部技能整理而来，已移除个人绝对路径、用户名、项目/客户标识、内部平台名称、真实域名与内网地址、密钥和事故细节。保留下来的是可复用的方法、Windows 环境坑和纠错纪律。

具体移除与保留范围见 `SANITIZATION.md`。如果你发现仍有可疑信息，请通过仓库 Issue 或维护者公开渠道联系，不要直接贴出敏感值。

## 参与贡献

见 `CONTRIBUTING.md`。新增环境坑请写清“坑 / 绕法 / 验证方式”；新增守护脚本必须自带正反例。

## 许可

本作品采用 **PolyForm Noncommercial License 1.0.0**，仅允许非商业用途。英文原文见 `LICENSE`，中文说明见 `LICENSE.zh-CN.md`。

商业使用需要作者书面授权，但**可以和作者商量**。如需商业化，可以联系 `chenshuidelaye@qq.com`；授权方式、范围和费用按实际使用场景协商，见 `COMMERCIAL.md`。

## 版本

- 版本：1.0.0
- 对应内部整理日期：2026-09-04
- 公开发布整理：2026-10-29