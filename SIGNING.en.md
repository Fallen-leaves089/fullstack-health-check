# Release Signing and Download Verification

[中文](SIGNING.md) | **English**

Release artifacts use **OpenSSH Ed25519** signatures. Windows 10/11, macOS, and most Linux systems can verify them without installing GPG.

## Release files

When downloading from GitHub Releases, get all of these:

- `project-health-<version>.zip`: the skill package
- `SHA256SUMS`: the zip checksum
- `SHA256SUMS.sig`: the OpenSSH signature over `SHA256SUMS`
- `allowed_signers`: the permitted signing identity and public key
- `release-signing-key.pub`: the public key used to check the fingerprint; it matches the key in `allowed_signers`

Both the repository and the Release provide `release-signing-key.pub`.

`SHA256SUMS` and `SHA256SUMS.sig` are release assets created after the zip. They are not placed inside the zip, which avoids a recursive hash change caused by embedding the archive's own checksum.

## Signing identity

- Principal: `project-health-release`
- Algorithm: Ed25519
- Public key fingerprint: `SHA256:DBDrYR50w0G+UuqtNAIgKZsXX4AjZAh3nvv0/zMCN8M`
- Public key file: `release-signing-key.pub`

Check the fingerprint before verifying:

```bash
ssh-keygen -lf release-signing-key.pub
```

## macOS verification

macOS normally bundles `shasum` but does not always provide `sha256sum`. From the download directory:

```bash
shasum -a 256 -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers \
  -I project-health-release \
  -n file -s SHA256SUMS.sig < SHA256SUMS
```

## Linux verification

GNU coreutils distributions normally provide `sha256sum`. From the download directory:

```bash
sha256sum -c SHA256SUMS
ssh-keygen -Y verify -f allowed_signers \
  -I project-health-release \
  -n file -s SHA256SUMS.sig < SHA256SUMS
```

On macOS or Linux, both commands must succeed. A successful signature check alone is not enough; also confirm that the filename in `SHA256SUMS` matches the file you downloaded.

## Windows PowerShell verification

Windows OpenSSH normally includes `ssh-keygen.exe`. From the download directory:

```powershell
$zip = '.\project-health-<version>.zip'
$expected = ((Get-Content .\SHA256SUMS | Where-Object { $_ -match [regex]::Escape([IO.Path]::GetFileName($zip)) } | Select-Object -First 1) -split '\s+')[0].ToLowerInvariant()
$actual = (Get-FileHash -LiteralPath $zip -Algorithm SHA256).Hash.ToLowerInvariant()
if ($actual -ne $expected) { throw 'SHA256 mismatch' }

cmd /c "ssh-keygen.exe -Y verify -f allowed_signers -I project-health-release -n file -s SHA256SUMS.sig < SHA256SUMS"
if ($LASTEXITCODE -ne 0) { throw 'Signature verification failed' }
```

Windows PowerShell 5.1 does not support `<` input redirection directly, so `cmd /c` is used only to feed `SHA256SUMS` to `ssh-keygen`. Do not replace it with `Get-Content | ssh-keygen`; pipeline encoding can change the signed bytes.

## Check the zip hash only

If the public key is not available yet, at least compare the hash.

Windows PowerShell:

```powershell
Get-FileHash -LiteralPath .\project-health-<version>.zip -Algorithm SHA256
```

macOS:

```bash
shasum -a 256 project-health-<version>.zip
```

Linux:

```bash
sha256sum project-health-<version>.zip
```

A matching hash proves only that the download was not corrupted. It does not prove the publisher's identity; identity verification requires `SHA256SUMS.sig`.

## Maintainer signing procedure

Keep the private key on the maintainer's machine. **Never commit it, upload it to a Release, or print it in CI logs.** For each release:

1. Sign `SHA256SUMS` with the fixed release key.
2. Generate `SHA256SUMS.sig`.
3. Upload the zip, `SHA256SUMS`, and `SHA256SUMS.sig` to the same Release.
4. Recheck the public-key fingerprint and signature verification.

When rotating the key, publish a new `release-signing-key.pub` and `allowed_signers` and record the rotation date in `CHANGELOG.md`. Old signatures remain tied to the old key; do not silently overwrite the identity.
