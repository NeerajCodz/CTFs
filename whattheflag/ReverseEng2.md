# CTF Writeup – GDB Reverse Engineering Challenges

## Challenge 2 – addr

```
WTF{0xa1b2c3d4}
```

### Objective

Bypass execution delay and recover a statically stored value in memory.

---

### Initial Enumeration

Download and execute the binary:

```bash
wget https://thebinaryfilelink
file addr
chmod +x addr
./addr
```

Behavior observed:

* Program delays for 101 seconds
* Prints: "Nice Try.The flag- "

This indicates a deliberate delay mechanism.

---

### Debugging with GDB

Launch GDB:

```bash
gdb addr
```

List functions:

```gdb
(gdb) info functions
```

The presence of a `sleep` call explains the delay.

Disassemble main:

```gdb
(gdb) disas main
```
<img width="552" height="551" alt="image" src="https://github.com/user-attachments/assets/c3e6123c-e201-4931-9f65-4535fcf08319" />

Set a breakpoint:

```gdb
(gdb) break main
(gdb) run
```

Skip the sleep call using:

```gdb
(gdb) jump *(main+29)
```

The program now prints immediately:

```
Nice Try.The flag-
```

---

### Inspecting Static Memory

`main` references a function named `load_st`. Disassemble it:

```gdb
(gdb) disas load_st
```

A static address is referenced:

```
0x404020
```
<img width="774" height="398" alt="image" src="https://github.com/user-attachments/assets/af1e2b5f-a6a5-4c2e-b144-b4315cf04813" />


Examine the memory contents:

```gdb
(gdb) x/4xb 0x404020
```

Output:

```
0xd4 0xc3 0xb2 0xa1
```

---

### Interpreting Endianness

The system uses little-endian ordering.

Bytes in memory:

```
d4 c3 b2 a1
```

Reversed for correct interpretation:

```
a1 b2 c3 d4
```

Final reconstructed value:

```
0xa1b2c3d4
```

---

### Final Flag

```
WTF{0xa1b2c3d4}
```

---

## Key Concepts Demonstrated

* Reverse engineering stripped binaries
* Function flow tracing in GDB
* Identifying hardcoded constants in assembly
* Hexadecimal to decimal conversion
* Bypassing execution delays using `jump`
* Static memory inspection
* Understanding little-endian representation

---
