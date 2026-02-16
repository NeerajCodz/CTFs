# Caller Was Here

## Challenge Summary

A simulated incident response scenario presents multiple artifacts including logs, transcripts, and an image file. Among these artifacts is a token hidden in plain sight. The objective is to recover the final flag by correlating evidence across files and reversing an encoded token.

The challenge emphasizes analytical thinking over brute force. Key techniques involved include log correlation, string extraction from binary files, base64 decoding, hexadecimal processing, and XOR decryption.

---

## Final Flag

```
WTF{caller_was_here}
```

---

## Investigation Process

### 1. Identifying the Relevant Token

The file `pastebin_sim.txt` contains several token-like entries. The relevant entry is identified by searching for:

```
CALL-7777:
```

The line beginning with `CALL-7777:` contains a base64-encoded string. This is the encoded token that must be decoded.

---

### 2. Extracting the Caller Number

The file `sip_logs.json` contains call metadata. Searching for the entry:

```json
"call_id": "CALL-7777"
```

reveals the associated `caller_number`, for example:

```
+91-120-555-4321
```

---

### 3. Reversing the Last Four Digits

The instructions hint that the end of the phone number may “tell a different story when read the other way around.”

The last four digits:

```
4321
```

Reversed:

```
1234
```

This forms the first portion of the XOR key.

---

### 4. Extracting the KEYFRAG from the Image

The file `portal_image.png` contains hidden metadata. Running:

```bash
strings portal_image.png | grep KEYFRAG
```

returns:

```
KEYFRAG:05
```

The fragment extracted is:

```
05
```

---

### 5. Building the Combined XOR Key

Concatenating the reversed digits and the image fragment:

```
1234 + 05 → 123405
```

The ASCII bytes of `123405` are used as the XOR key.

---

### 6. Base64 Decoding the Token

The base64 token obtained from `pastebin_sim.txt`:

```
NjY2Njc1NGY1MzU0NWQ1ZTU2NDY2ZjQyNTA0MTZjNWM1NTQ3NTQ0Zg==
```

Decode it:

```bash
echo "NjY2Njc1NGY1MzU0NWQ1ZTU2NDY2ZjQyNTA0MTZjNWM1NTQ3NTQ0Zg==" | base64 -d > hex.txt
```

This produces a hexadecimal string.

---

### 7. Converting Hex to Raw Bytes

The decoded output is not the final plaintext. It is a hex-encoded string. Convert it to raw bytes:

```bash
xxd -r -p hex.txt > data.bin
```

This step is critical, as the challenge hints at using the raw value of the hex.

---

### 8. XOR Decryption

The resulting binary data is XOR-encrypted. Using the combined key `123405`, perform XOR decryption.

Example helper usage:

```bash
python3 decode.py "NjY2Njc1NGY1MzU0NWQ1ZTU2NDY2ZjQyNTA0MTZjNWM1NTQ3NTQ0Zg==" 123405
```

After XOR-decoding, the plaintext flag is revealed:

```
WTF{caller_was_here}
```

---

## Key Concepts Demonstrated

* Log correlation across multiple artifacts
* Hidden data extraction using `strings`
* Base64 decoding
* Hexadecimal to raw byte conversion with `xxd`
* XOR decryption with a derived key
* Clue interpretation and analytical reasoning

---

## Conclusion

The challenge required careful analysis of logs and artifacts, attention to subtle hints, and layered decoding steps. The solution involved:

1. Locating the correct token
2. Deriving a key from reversed caller digits and image metadata
3. Performing base64 decoding
4. Converting hex to raw bytes
5. Applying XOR decryption

The recovered flag confirms successful reconstruction of the encoded token.
