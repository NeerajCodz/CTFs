# Task 1 - Static Binary Analysis

## Objective
Recover Flag 1 from the Linux binary without relying on runtime exploitation.

## Initial Recon
```bash
file ccs26_challenge
strings ccs26_challenge > strings.txt
grep -Ei "flag|ctf" strings.txt
```

The output revealed a configuration-like section:

```text
kernel_flag=CTF{its_3am_might_just_git_push_force}
deploy_code=0xCTF_1_3aSy_t3xt
```

## Analysis
- `kernel_flag` was directly in valid CTF flag format.
- `deploy_code` did not match expected flag format and was treated as auxiliary data.
- No execution or network interaction was required for this step.

## Result
```text
CTF{its_3am_might_just_git_push_force}
```

## Key Insight
Always perform string-level static triage first. In many CTF binaries, at least one flag is intentionally recoverable from embedded constants.
