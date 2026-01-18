🏴‍☠️ **GDG CYBER RECRUITMENTS 2026**

# Task 1B

## Challenge Overview

As part of **GDG Cybersecurity Recruitment 2026**, we were provided with a Linux executable named `apple_pie`. The binary was advertised as being protected using modern security mechanisms along with basic obfuscation techniques.

The objective of the challenge was to analyze the binary and recover a hidden flag in the following format:

```
gdg{flag}
```

No source code was provided, and exploitation was not explicitly required. This strongly suggested a **reverse‑engineering–focused challenge** rather than a memory corruption exploit.

---

## Initial Understanding from the Challenge Description

From the README and problem statement, the following assumptions were made:

* The target is a **Linux binary analysis** challenge
* The binary likely implements:

  * PIE (Position Independent Executable)
  * NX (Non‑Executable stack)
  * Basic obfuscation
* Source code is not available
* The goal is to **recover hidden data**, not necessarily gain shell access

Based on this, the intended methodology was:

* Binary reconnaissance
* Static reverse engineering
* Selective dynamic analysis (only where useful)

---

## Step 1: Basic Reconnaissance

### 1.1 File Type Identification

The first step was to identify the binary type:

```
file apple_pie
```

**Output:**

```
ELF 32-bit LSB pie executable, Intel i386, dynamically linked, not stripped
```

#### Key Observations

* 32‑bit ELF binary (i386)
* PIE enabled
* Dynamically linked
* **Not stripped** → symbols and function names are preserved (a major advantage for reverse engineering)

---

### 1.2 Security Protections

We then analyzed enabled security mitigations:

```
checksec --file=apple_pie
```

**Summary:**

* PIE: Enabled
* NX: Enabled
* Stack Canary: Not present

#### Implications

* Code addresses are randomized (PIE)
* Stack memory is non‑executable (NX)
* Lack of stack canary is irrelevant here since no obvious overflow vectors exist

Overall, traditional buffer overflow exploitation is unlikely. This reinforced the idea that the challenge is logic‑ or analysis‑based rather than exploit‑driven.

---

## Step 2: Executing the Binary

The binary was executed to observe runtime behavior:

```
./apple_pie
```

**Output:**

```
=== The Elusive PIE ===
Enter password:
```

After entering any input:

```
Wrong password... but maybe there's still a way in.
Say something to the program:
```

#### Observations

* The password is always rejected
* Execution continues regardless of correctness
* The program accepts a **second user input** and echoes it back

This behavior suggested that:

* The password check is likely a decoy
* The real logic lies elsewhere

---

## Step 3: Strings Analysis

To identify embedded strings, we ran:

```
strings apple_pie
```

### Notable Strings

* `not_the_password`
* `You found the hidden treasure: %s`
* `Wrong password... but maybe there's still a way in.`
* `Say something to the program:`

#### Interpretation

* `not_the_password` is clearly misleading and likely a fake check
* `You found the hidden treasure: %s` strongly implies:

  * There exists hidden data
  * It is printed via `printf` with a string argument

At this stage, we suspected:

* A hidden or unused function
* Obfuscated data decoded at runtime

---

## Step 4: Format String Vulnerability (Red Herring)

While testing the second input, it was observed that the program directly passed user input to `printf`:

```c
printf(user_input);
```

This confirmed the presence of a **format string vulnerability**.

### Experiments

Inputs tested included:

* `%x %x %x`
* `%p %p %p`
* `%n$s`

### Outcome

* Stack values could be leaked
* The program frequently crashed
* No reliable memory disclosure of the flag was achieved
* The vulnerable path never referenced the "hidden treasure" string

#### Conclusion

The format string bug is **intentional but misleading**. It serves as a red herring and is not the intended solution path.

---

## Step 5: Static Analysis Using Ghidra

Given the binary was not stripped, we moved to full static analysis using **Ghidra**.

### 5.1 Decompiling `main()`

The decompiled `main()` revealed:

* A password comparison against the hardcoded string `not_the_password`
* Identical execution flow regardless of match result
* A vulnerable `printf`, but **no flag logic**

This confirmed:

* The password check is meaningless
* The flag is not handled in `main()`

---

## Step 6: Investigating the `.rodata` Section

Using Ghidra’s **Defined Strings** window, we located:

```
"You found the hidden treasure: %s"
```

### Cross‑Reference Analysis

Examining XREFs showed that this string is referenced **only once**, by a function named:

```
def_nothing_important
```

#### Critical Insight

* `main()` never calls this function
* The function name is intentionally misleading
* This unused function is the likely flag container

This was the turning point of the challenge.

---

## Step 7: Reverse Engineering `def_nothing_important`

The decompiled function appeared as:

```c
void def_nothing_important(void)
{
    byte local_26[22];

    local_26[i] = <hardcoded hex values>;

    for (i = 0; i < 22; i++) {
        local_26[i] ^= 0x05;
    }

    printf("You found the hidden treasure: %s\n", local_26);
}
```

### Analysis

* A 22‑byte array contains hardcoded hexadecimal values
* Each byte is XOR‑decoded using key `0x05`
* The decoded buffer is printed as a string
* The function is **never invoked**, preventing the flag from appearing at runtime

Thus, the flag must be recovered **manually** through static analysis.

---

## Step 8: Manual XOR Decoding

Each byte was XOR‑decoded using the key `0x05`.

### Example

* `0x62 ^ 0x05 = 0x67` → `g`
* `0x61 ^ 0x05 = 0x64` → `d`
* `0x62 ^ 0x05 = 0x67` → `g`
* `0x7e ^ 0x05 = 0x7b` → `{`

Repeating this process for all 22 bytes yielded the final string.

---

## Final Flag

```
gdg{P1E_3xpl01ted_lol}
```

---

## Final Thoughts and Lessons Learned

### Obstacles and Intentional Misdirection

* Fake password check (`not_the_password`)
* Deliberate format string vulnerability
* Hidden, unused function
* XOR‑encoded flag stored in `.rodata`

### Key Takeaways

* Not every vulnerability is meant to be exploited
* Static analysis often outperforms dynamic exploitation
* Cross‑references to unused functions are extremely powerful
* Always inspect `.rodata` and uncalled code paths

---

## Tools Used

* `file`
* `checksec`
* `strings`
* Ghidra
* Manual XOR decoding

---

## Conclusion

This challenge effectively tested:

* Binary reconnaissance
* Reverse‑engineering discipline
* The ability to ignore red herrings
* Understanding of basic obfuscation techniques

The flag was successfully recovered through **static reverse engineering**, demonstrating that careful analysis often matters more than exploitation.
