# The Entropy Silence

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Memory Dump
- Final Flag: geek{x0r_m3m_f0r3ns1cs_master}

## Executive Summary
The key detail is that the attachment was refreshed. The new core dump contains the real flag directly in memory, so a simple string scan solves the challenge.

## Solution Walkthrough
### Artifact Update
The old attachment `trace_qfcpWye.zip` led to a rejected candidate. The new
attachment `trace_2.zip` is a different core dump and contains the real flag.

### Recon
Unzip the new file and identify the artifact:

```bash
unzip trace_2.zip -d trace_2_extracted
file trace_2_extracted/trace.bin
readelf -n trace_2_extracted/trace.bin
```

This shows a normal x86-64 ELF core from `./ghost_machine`. The note section
also records the original path:

```text
/home/admin/Cybergeek CTF/Task 4/ghost_machine
```

### Extraction
The flag is present directly in the dumped memory. A simple string scan is
enough:

```bash
strings -a -n 4 trace_2_extracted/trace.bin | grep 'geek{'
```

Output:

```text
geek{x0r_m3m_f0r3ns1cs_master}
```

The raw byte location is at offset `0x15518` inside the new core.

### Validation
The live challenge page was previously saved locally in an unsolved state with a
visible `Submit Your Flag` form. After submitting
`geek{x0r_m3m_f0r3ns1cs_master}`, the live page changed to:

```text
THIS CHALLENGE HAS BEEN SOLVED BY YOU OR YOUR TEAM MEMBER
You submitted a flag at April 19, 2026, 2:09 a.m.
```

A follow-up control submission with an obviously wrong flag did not change that
timestamp, so the solve record is consistent with the accepted submission of the
new candidate.

### Conclusion
The refreshed artifact was the key. The old trace preserved an intermediate
buffer; `trace_2.zip` preserved the real flag directly in memory.

## Flag
```text
geek{x0r_m3m_f0r3ns1cs_master}
```
