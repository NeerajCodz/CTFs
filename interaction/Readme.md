# CCS26 CTF - Solution Write-Up

## Answer Flags
```text
CTF{its_3am_might_just_git_push_force}
CTF{source_is_trust_me_bro}
CTF{touch_grass}
```

## Overview
This challenge included Windows and Linux executables with three hidden flags. Solving required a layered approach across static analysis, runtime/network observation, and remote API interaction.

The binaries were built with Flutter, which introduced obfuscation and TLS-encrypted communication that made direct packet inspection less useful.

## Tools Used
- `strings`
- `file`
- `base64`
- `tcpdump`
- `curl`
- `binwalk`

`OS Used: Arch Linux`

## Flag Summary
| Flag | Method | Description |
| --- | --- | --- |
| Flag 1 | Static binary analysis | Hardcoded string recovered from executable |
| Flag 2 | Asset and network reconnaissance | Extracted from Flutter asset data |
| Flag 3 | Remote API interaction | Retrieved via challenge/submit flow |

## Task Writeups
- `Task1.md` - Static binary analysis and first flag extraction
- `Task2.md` - Network-oriented reconnaissance and Flutter asset extraction
- `Task3.md` - Remote challenge solve and final flag retrieval

## Final Takeaway
The intended path rewarded methodical analysis over brute force. When direct TLS decryption was blocked, shifting to Flutter bundle inspection exposed the required artifact and kept momentum toward complete flag recovery.
