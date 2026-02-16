# CTF Writeup – GDB Reverse Engineering Challenges

## Challenge 1 – gdb_call

### Objective

Analyze a stripped binary using GDB and recover the hidden flag.

---

### Initial Enumeration

Download and inspect the binary:

```bash
wget https://thebinaryfilelink
file gdb_call
chmod +x gdb_call
./gdb_call
```

Execution output:

```
Hello my cute lil red flags<3
```

No flag is displayed during normal execution.

---

### Debugging with GDB

Launch GDB:

```bash
gdb gdb_call
```

Check for debugging symbols:

```gdb
(gdb) info functions
```

Even though the binary is stripped, visible functions include:

* main
* func1
* func2
* func3
* func4
* func5

---
<img width="680" height="695" alt="image" src="https://github.com/user-attachments/assets/2740adf1-b131-4d91-acfd-a0282a8c4257" />

### Function Flow Analysis

Disassemble `main`:

```gdb
(gdb) disas main
```

`main` calls `func1`.

Repeat the process:

```gdb
(gdb) disas func1
(gdb) disas func2
(gdb) disas func3
(gdb) disas func4
```

Inside `func4`, the following instruction appears:

```assembly
imul   $0x4479,%eax,%eax
```

The hexadecimal constant `0x4479` is significant.

---

### Converting the Hex Value

Convert the hex value to decimal inside GDB:

```gdb
(gdb) p/d 0x4479
```

Result:

```
17529
```

---

### Final Flag

```
WTF{17529}
```

---

