# EZ Eraser

Free single-file Windows utility for wiping disks before you recycle, sell, or hand them off. I built it because I didn't want to pay for a bloated commercial wiper to do something Windows can already do, just with a worse UI.

<img width="921" height="631" alt="ez-eraser-main" src="https://github.com/user-attachments/assets/877e9ce7-6515-4529-950c-040a8689d6c0" />

## Features

* Wipes physical drives using NIST SP 800-88, DoD 5220.22-M, RCMP TSSIT OPS-II, HMG IS5 Enhanced, Schneier, VSITR, and Gutmann
* Hardware Secure Erase via ATA/NVMe firmware commands when the drive supports it
* Live mosaic visualizer with per-block wipe progress and disk health
* Verification pass after the wipe completes, reading every sector back to confirm zeros
* Audit mode for checking whether a drive is already clean without wiping it
* Resume support if the wipe gets interrupted by power loss, a crash, or a disconnect
* Tamper-evident certificate of destruction in HTML with a SHA-256 integrity hash
* USB keep-alive so Windows doesn't park drives mid-wipe

## Limitations

Disk 0 is locked out, so this won't wipe the system drive. It's also not a recovery tool, the binary isn't signed (see below on SmartScreen), and it's Windows only.

## Download

Grab the latest `DiskWipper.exe` from the [Releases](../../releases) page. Single-file portable executable, around 70 MB. No installer, no dependencies. Run it as administrator.

### About the SmartScreen warning

Windows shows "Windows protected your PC" the first time you run it because the binary isn't signed with a code signing certificate. Those run a few hundred dollars a year, which is a lot for a hobby project.

To run it anyway:

1. Click "More info"
2. Click "Run anyway"

### SHA-256

```
8E889B3590A1BDB9377BCBFB590375C20B3136B4BF49667DB1725CF9C8E3AFEF
```

Verify with PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 .\DiskWipper.exe
```

The output should match the hash above. If it doesn't, the file is corrupted or has been tampered with. Re-download.

## Usage

1. Run the `.exe` as administrator (UAC will prompt)
2. Select the disk you want to wipe from the list
3. Pick an algorithm. For most drives, NIST SP 800-88 Clear (single random pass) is fine and fast
4. Click the play button, confirm with the skull
5. Wait

The visualizer shows live progress, the speed graph shows throughput, and resume state is saved every 5 seconds in case something goes sideways.

For long wipes, leave the Prevent Sleep option enabled (it's on by default). It also keeps Windows from parking USB drives via power management.

## Algorithm guide

| Algorithm | Passes | Best for |
|-----------|--------|----------|
| Zero Fill | 1 | Quick erase for non-sensitive data |
| Random | 1 | General-purpose single-pass |
| NIST SP 800-88 Clear | 1 | Modern federal standard, recommended for SSDs |
| DoD 5220.22-M | 3 | Mechanical HDDs, older compliance frameworks |
| RCMP TSSIT OPS-II | 7 | Canadian government standard |
| HMG IS5 Enhanced | 3 | UK government standard |
| Schneier | 7 | High-security HDDs |
| VSITR | 7 | German BSI standard |
| Gutmann | 35 | Legacy MFM/RLL drives only, overkill for modern hardware |
| Secure Erase | 1 (firmware) | NVMe and SATA drives that support it |

For most users on modern drives, NIST SP 800-88 Clear or Zero Fill is sufficient. Multi-pass methods exist for compliance with specific regulatory frameworks. They don't add security on drives made after roughly 2001.

## Resume capability

If a wipe gets interrupted by power loss, an app crash, a USB disconnect, or a deliberate cancel, the operation resumes from where it stopped. State is saved to `%APPDATA%\EZEraser\` every 5 seconds. Reconnect the drive, select it, and the Resume button lights up.

Resume matches drives by serial number, model, and capacity. Swapping drives between sessions won't trigger a false resume; each drive's state is kept separately.

## Known issues

* USB-to-SATA bridges often block ATA Secure Erase commands. If Secure Erase fails on a USB drive, fall back to a software algorithm.
* Some BIOSes set a security freeze on SATA drives at boot. If Secure Erase reports a frozen drive, suspend and resume the system and try again.
* Progress during Secure Erase is synthetic. The drive does the erase internally and doesn't expose progress, so the UI counts up to avoid looking frozen.
* Audits over USB 2.0 are slow due to bus speed.

## License

MIT. See LICENSE. Use it however you want, including commercially. No warranty.

## Disclaimer

This tool destroys data permanently. There is no undo. Check the drive before you confirm. Disk 0 is locked out, but every other connected drive is fair game. Not responsible for data loss from picking the wrong drive.
