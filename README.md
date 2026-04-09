# PowerShell Shellcode Injector

> Windows shellcode loader that downloads position-independent code from GitHub Releases and executes it in-process via function pointer with W^X memory discipline.

![Language](https://img.shields.io/badge/language-PowerShell-blue?logo=powershell&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6?logo=windows&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)
![Architecture](https://img.shields.io/badge/arch-x86__64%20%7C%20i386%20%7C%20aarch64%20%7C%20armv7a-lightgrey)
![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B%20%7C%207%2B-5391FE?logo=powershell&logoColor=white)

---

## Features

- **In-process execution** -- shellcode runs inside the PowerShell process via `Func<int>` delegate, no child process created.
- **W^X memory discipline** -- memory is never simultaneously writable and executable. Allocates RW, copies payload, flips to RX, then invokes.
- **Automatic architecture detection** -- reads `PROCESSOR_ARCHITECTURE` to select the correct binary (`x86_64`, `i386`, `aarch64`, `armv7a`).
- **Zero dependencies** -- pure PowerShell with inline C# P/Invoke. No modules, no NuGet packages, no external tools.
- **Structured logging** -- color-coded, timestamped log output with severity levels (DBG, INF, OK, WRN, ERR).
- **Payload validation** -- rejects payloads that are too small or that contain HTML (proxy/CDN error page detection).

## Requirements

| Requirement | Details |
|---|---|
| **OS** | Windows 7 / Server 2008 R2 or later |
| **PowerShell** | 5.1+ (Desktop) or 7+ (Core) |
| **Privileges** | Standard user (no admin required) |
| **Network** | HTTPS access to `github.com` |
| **.NET** | .NET Framework 3.5+ (ships with Windows) |

No third-party modules or dependencies are needed.

## Usage

```powershell
# Run the loader -- auto-detects architecture, downloads, and executes
.\loader.ps1
```

### How It Works

1. Auto-detect architecture from `PROCESSOR_ARCHITECTURE`
2. Download the matching `windows-<arch>.bin` asset from GitHub Releases
3. `VirtualAlloc` with `PAGE_READWRITE` (RW)
4. Copy shellcode into the allocated region
5. `VirtualProtect` to `PAGE_EXECUTE_READ` (RX)
6. Invoke the entry point as a `Func<int>` delegate (function pointer call)
7. `VirtualFree` on return

### Supported Architectures

Architecture is determined automatically from the running PowerShell process:

| PowerShell Process | `PROCESSOR_ARCHITECTURE` | Asset Downloaded |
|---|---|---|
| 64-bit on x86_64 | AMD64 | `windows-x86_64.bin` |
| 64-bit on ARM64 | ARM64 | `windows-aarch64.bin` |
| 32-bit on x86 | x86 | `windows-i386.bin` |
| 32-bit on ARM | ARM | `windows-armv7a.bin` |

### SSL Note

SSL certificate verification is intentionally disabled during download. The loader fetches unsigned shellcode from public GitHub Releases, so TLS certificate pinning adds no security value and would break on hosts with outdated CA certificate stores.

## Project Structure

```
.
├── loader.ps1          # Main shellcode loader script
├── LICENSE             # MIT License
├── RESPONSIBLE_USE.md  # Responsible use policy
├── .gitignore          # Git ignore rules
└── README.md           # This file
```

## Disclaimer

This project is provided strictly for **authorized security testing**, **educational purposes**, **Capture The Flag (CTF) competitions**, and **security research**. Use of this tool against systems without explicit written authorization is illegal and unethical.

The authors assume no liability for misuse. You are solely responsible for ensuring that your use complies with all applicable laws and regulations.

By using this software, you agree to the terms outlined in [RESPONSIBLE_USE.md](RESPONSIBLE_USE.md).

## License

This project is licensed under the [MIT License](LICENSE).
