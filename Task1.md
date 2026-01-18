🏴‍☠️ **GDG CYBER RECRUITMENTS 2026**

# Task 1: (Level 1-3)

---

## Overview

This challenge followed a Capture The Flag (CTF) format consisting of **three independent parts**, each designed to test a different core cybersecurity skill:

* **Part 1:** Git forensics
* **Part 2:** Image steganography
* **Part 3:** Automation and data filtering

Each part contained one fragment of the final flag. The fragments had to be combined in the following format:

```
gdg{part1_part2_part3}
```

Each folder was approached independently. Instructions were carefully read, reconnaissance was performed first, and only non‑destructive techniques were used throughout the challenge.

---

## Part 1 – Git History Forensics (`gdg_part1`)

### Initial Observation

Upon reading the README file, the hint provided was:

> “Things aren’t always what they seem on the first glance.”

Listing the directory contents revealed the presence of a `.git` directory alongside two ordinary‑looking files:

* `app.py`
* `xotwod.txt`

The presence of a `.git` directory immediately indicated that Git history analysis would be relevant.

### Thought Process

* A `.git` directory is rarely present accidentally in CTF challenges
* The visible files appeared intentionally harmless and likely served as decoys
* Sensitive information is often removed from the working tree but remains recoverable in commit history

Based on this, the logical next step was to inspect the repository history for deleted or modified files.

### Actions Performed

1. Checked the repository status to identify changes
2. Inspected commit history using:

```
git log -p
```

3. Discovered a commit with the message:

```
Remove config file (oops, committed secrets!)
```

4. Examined the diff for that commit and found a deleted file named `config.txt`
5. The deleted file contained the following line:

```
FLAG_PART_1=gdg{sw1ss_
```

### Result

The extracted flag fragment was:

```
sw1ss_
```

### Key Learning

This part tested awareness of common Git hygiene mistakes and the ability to recover sensitive data from commit history rather than relying solely on the current working tree.

---

## Part 2 – Image Steganography (`gdg_part2`)

### Initial Observation

The README file stated:

> “At first glance it’s just an image; But in a CTF, appearances are almost always deceptive.”

The directory contained a single file:

* `heheheha.png`

This strongly suggested the use of steganography.

### Thought Process

Common image‑based hiding techniques considered:

* Metadata manipulation (EXIF data)
* Embedded files or appended archives
* Pixel‑level steganography (LSB techniques)

A layered analysis approach was chosen, starting with the simplest checks and moving toward more advanced techniques.

### Actions Performed

1. Verified file type:

```
file heheheha.png
```

2. Checked for metadata:

```
exiftool heheheha.png
```

No useful metadata was present.

3. Checked for embedded files:

```
binwalk heheheha.png
```

No hidden files or appended data were detected.

4. Performed pixel‑level steganography analysis using:

```
zsteg heheheha.png
```

### Discovery

`zsteg` revealed hidden text in the LSB RGB channel:

```
b1,rgb,lsb,xy .. text: "10:armykn1f3_"
```

The prefix `10:` was identified as a decoy and discarded.

### Result

The extracted flag fragment was:

```
armykn1f3_
```

### Key Learning

This part tested image forensics and steganography skills, particularly recognizing when to move beyond metadata analysis and into pixel‑level data extraction.

---

## Part 3 – QR Code Zipbomb (`gdg_part3`)

### Initial Observation

The README contained the hint:

> “Surely nobody’s going to sit and scan them one by one… Think smart, not hard.”

The directory contained:

* An archive with **3000 QR code images**
* Files named sequentially: `qr_1.png` through `qr_3000.png`

Manual analysis was clearly impractical.

### Thought Process

* Automation was required
* The majority of QR codes were expected to be decoys
* The real signal would be hidden through repetition or rarity

### Actions Performed

1. Extracted the archive
2. Automatically decoded all QR codes in order:

```
for i in $(ls qr_*.png | sort -V); do
  zbarimg "$i" | cut -d':' -f2
done > decoded.txt
```

3. Observed that each decoded line ended with a scan identifier such as:

```
(scan #XXXX)
```

4. Normalized the data by removing scan numbers:

```
sed 's/ (scan #[0-9]\+)//' decoded.txt > stripped.txt
```

5. Performed frequency analysis:

```
sort stripped.txt | uniq -c | sort -nr
```

### Discovery

One string appeared **only once**, unlike all others:

```
PDwtLS0tcGFydDM9Z2cxb2x9LS0tLT4+
```

This string was identified as Base64‑encoded.

### Decoding

```
echo "PDwtLS0tcGFydDM9Z2cxb2x9LS0tLT4+" | base64 -d
```

Decoded output:

```
<----part3=gg1ol}--->
```

### Result

The extracted flag fragment was:

```
gg1ol
```

### Key Learning

This part tested:

* Automation and scripting
* Data normalization
* Noise filtering
* Encoding recognition

---

## Final Flag

Combining all extracted fragments:

* **Part 1:** `sw1ss_`
* **Part 2:** `armykn1f3_`
* **Part 3:** `gg1ol`

Final flag:

```
gdg{sw1ss_armykn1f3_gg1ol}
```

---

## Final Takeaway

This CTF effectively simulated real‑world security scenarios and emphasized methodology over brute force. It reinforced practical skills in:

* Version control forensics
* Steganographic analysis
* Automation and scripting
* Logical problem solving

Each part required careful observation, disciplined analysis, and the ability to filter meaningful signals from noise—core competencies in real‑world cybersecurity work.
