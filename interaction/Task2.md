# Task 2 - Network Reconnaissance and Flutter Asset Extraction

## Objective
Recover Flag 2 linked to external communication behavior.

## Endpoint Discovery via Static Strings
```bash
grep -Eo 'https?://[^ ]+' strings.txt
```

Discovered endpoints:

```text
https://ccs26-appdev-server.vercel.app/api/status
https://ccs26-appdev-server.vercel.app/api/challenge
https://ccs26-appdev-server.vercel.app/api/submit
```

From discovered runtime artifacts, the target was confirmed as a Flutter desktop application.

<img width="1720" height="860" alt="Endpoint discovery" src="https://github.com/user-attachments/assets/2db9de4d-3f8b-4f39-a784-69c102930fbb" />

## TLS Capture Attempt (Non-viable Path)
```bash
tcpdump -i any -w ccs26.pcap
./ccs26_challenge
```

- Captured roughly 184 TLS packets.
- Decryption attempts in Wireshark did not provide useful plaintext due to the app/network stack behavior.

<img width="1919" height="1016" alt="TLS capture" src="https://github.com/user-attachments/assets/536e30a7-d59b-42ef-87d0-03a3f33d5789" />

## Successful Path - Flutter Asset Inspection
```bash
binwalk -e ccs26_challenge
cd _ccs26_challenge.extracted
tree
cd data/flutter_assets/assets
ls
```

Located `secrets.txt` containing encoded data:

```bash
cat secrets.txt
```

```ini
trace_id=Q1RGe3NvdXJjZV9pc190cnVzdF9tZV9icm99Cg==
```

<img width="1916" height="499" alt="secrets.txt" src="https://github.com/user-attachments/assets/8a748521-2251-4bbf-ac2a-da87f3936d43" />

Decoding:

```bash
echo Q1RGe3NvdXJjZV9pc190cnVzdF9tZV9icm99Cg== | base64 -d
```

## Result
```text
CTF{source_is_trust_me_bro}
```

## Key Insight
When network telemetry is encrypted and opaque, local packaged assets can still expose the exact values used by application logic.
