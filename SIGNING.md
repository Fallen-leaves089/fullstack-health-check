# 发布签名与下载校验

[中文](SIGNING.md) | [English](SIGNING.en.md)

发布包使用 **OpenSSH Ed25519** 签名。这样 Windows 10/11、macOS 和大多数 Linux 不需要额外安装 GPG，就能验证下载文件。

## 发布文件

从 GitHub Releases 下载时，请同时下载：

- `fullstack-health-check-<version>.zip`：技能包
- `SHA256SUMS`：zip 的 SHA256 校验和
- `SHA256SUMS.sig`：`SHA256SUMS` 的 OpenSSH 签名
- `allowed_signers`：允许的签名身份和公钥
- `release-signing-key.pub`：用于核对公钥指纹，内容与 `allowed_signers` 中的公钥一致

仓库和 Release 都提供 `release-signing-key.pub`。

`SHA256SUMS` 和 `SHA256SUMS.sig` 是在 zip 生成之后创建的发布资产，不放进 zip 内部，避免“包含自身校验和”导致哈希循环变化。

## 签名身份

- Principal：`fullstack-health-check-release`
- 算法：Ed25519
- 公钥指纹：`SHA256:DBDrYR50w0G+UuqtNAIgKZsXX4AjZAh3nvv0/zMCN8M`
- 公钥文件：`release-signing-key.pub`

先核对指纹，再验证签名：

```bash
ssh-keygen -lf release-signing-key.pub
```

## macOS 验证

macOS 通常自带 `shasum`，但不一定有 `sha256sum`。在下载目录运行：

```bash
shasum -a 256 -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers \
  -I fullstack-health-check-release \
  -n file -s SHA256SUMS.sig < SHA256SUMS
```

## Linux 验证

GNU coreutils 发行版通常使用 `sha256sum`。在下载目录运行：

```bash
sha256sum -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers \
  -I fullstack-health-check-release \
  -n file -s SHA256SUMS.sig < SHA256SUMS
```

在 macOS 或 Linux 上，两条命令都成功，才说明文件完整且校验和由发布密钥签名。只看到 `Good "file" signature` 或 `Signature verified` 之类的成功信息还不够，应同时确认 `SHA256SUMS` 中的文件名与下载文件一致。

## Windows PowerShell 验证

Windows 内置 OpenSSH 通常已经包含 `ssh-keygen.exe`。在下载目录运行：

```powershell
$zip = '.\fullstack-health-check-<version>.zip'
$expected = ((Get-Content .\SHA256SUMS | Where-Object { $_ -match [regex]::Escape([IO.Path]::GetFileName($zip)) } | Select-Object -First 1) -split '\s+')[0].ToLowerInvariant()
$actual = (Get-FileHash -LiteralPath $zip -Algorithm SHA256).Hash.ToLowerInvariant()
if ($actual -ne $expected) { throw 'SHA256 mismatch' }

cmd /c "ssh-keygen.exe -Y verify -f allowed_signers -I fullstack-health-check-release -n file -s SHA256SUMS.sig < SHA256SUMS"
if ($LASTEXITCODE -ne 0) { throw 'Signature verification failed' }
```

PowerShell 5.1 不直接支持 `<` 输入重定向，所以这里用 `cmd /c` 只负责把 `SHA256SUMS` 送给 `ssh-keygen`。不要用 `Get-Content | ssh-keygen` 代替，管道编码可能改变被签名的字节。

## 只验证 zip 的 SHA256

如果暂时没有公钥，至少可以核对哈希。

Windows PowerShell：

```powershell
Get-FileHash -LiteralPath .\fullstack-health-check-<version>.zip -Algorithm SHA256
```

macOS：

```bash
shasum -a 256 fullstack-health-check-<version>.zip
```

Linux：

```bash
sha256sum fullstack-health-check-<version>.zip
```

哈希匹配只能说明文件在下载过程中没有损坏，不能证明发布者身份；身份验证需要 `SHA256SUMS.sig`。

## 维护者签名流程

私钥只保存在维护者本机，**不要提交到仓库、不要上传到 Release、不要写进 CI 日志**。发布时：

1. 用固定的发布密钥签署 `SHA256SUMS`。
2. 生成 `SHA256SUMS.sig`。
3. 将 zip、`SHA256SUMS`、`SHA256SUMS.sig` 一起上传到同一个 Release。
4. 复核公钥指纹和签名验证结果。

更换密钥时，发布新的 `release-signing-key.pub` 和 `allowed_signers`，并在 `CHANGELOG.md` 记录轮换日期；旧签名仍然对应旧公钥，不能悄悄覆盖身份。
