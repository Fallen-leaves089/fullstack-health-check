# Changelog

## 1.1.0 - 2026-10-29

### 中文

- 新增完整英文文档：`README.en.md`、`SKILL.en.md`、商业授权、贡献指南、脱敏说明和三份 `references/*.en.md`。
- 新增 Python 3.9+ 标准库实现的跨平台脚本：`check_redaction.py` 与 `verify_package.py`。
- 保留 PowerShell 5.1/7 脚本作为 Windows 的第二条自检路径。
- 新增 GitHub Actions，在 Windows、macOS、Ubuntu 上以 Python 3.9 和 3.12 验证包完整性；Windows 额外跑 PowerShell 5.1 与 PowerShell 7。
- 新增平台支持矩阵、`.gitattributes`、`.gitignore`、`.editorconfig` 和 `SECURITY.md`。
- 核心技能保持纯 Markdown + Git；Windows 专用命令继续隔离在 `references/windows-pitfalls*.md`。
- 说明 GitHub 许可证识别不支持 PolyForm 时显示 `NOASSERTION / Other` 属于正常现象，以 `LICENSE` 原文为准。

### English

- Added complete English companions for the README, skill body, commercial terms, contributing guide, sanitization notes, and all three reference documents.
- Added cross-platform Python 3.9+ standard-library tools: `check_redaction.py` and `verify_package.py`.
- Kept the PowerShell 5.1/7 scripts as a second Windows verification path.
- Added GitHub Actions for Windows, macOS, and Ubuntu with Python 3.9 and 3.12; Windows also runs PowerShell 5.1 and PowerShell 7.
- Added a platform support matrix, `.gitattributes`, `.gitignore`, `.editorconfig`, and `SECURITY.md`.
- The core skill remains plain Markdown plus Git; Windows-only commands stay isolated in `references/windows-pitfalls*.md`.
- Documented that GitHub may show `NOASSERTION / Other` for PolyForm because it is not in GitHub's recognized license list; `LICENSE` remains controlling.

## 1.0.0 - 2026-09-04 内部整理 / 2026-10-29 公开发布

- 从内部全栈体检技能整理出公开版。
- 移除个人路径、项目/客户标识、内部平台名称、真实域名与内网地址、密钥和具体事故数据。
- 保留全栈检查流程、Windows 环境坑、纠错与正反例验证纪律、安全底线。
- 新增 `references/windows-pitfalls.md`、`references/verification-playbook.md`、`references/report-template.md`。
- 新增 `scripts/check-redaction.ps1`，自带正反例自检。
- 新增 `scripts/verify-package.ps1`，检查文件完整性、UTF-8 无 BOM、相对链接和脱敏扫描。
- 采用 PolyForm Noncommercial License 1.0.0；商业授权可协商。
- 补充商业授权联系邮箱。
