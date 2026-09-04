# Reverse Engineering – Medium

**Author:** Abhishek Agarwal

## Challenge Description

> This program has constructed the flag using hex ASCII values. Identify the flag text by disassembling the program.

The challenge provides an executable file named `asciiftw`.

## Initial Analysis

First, I downloaded the challenge file. When I opened it with Notepad, I saw a lot of unreadable characters, which immediately suggested that it was a binary file rather than a normal text file.

I then opened WSL and used the `file` command to identify the file type:

```bash
nhatminh@DESKTOP-OJURB2M:~$ file /mnt/c/Users/nhatt/Downloads/asciiftw
/mnt/c/Users/nhatt/Downloads/asciiftw: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c29491782ee13aa7c5734d77b281865b608e46e9, for GNU/Linux 3.2.0, not stripped
```

The output confirmed that the file was a **64-bit ELF executable**, so I tried running it directly:

```bash
nhatminh@DESKTOP-OJURB2M:~$ /mnt/c/Users/nhatt/Downloads/asciiftw
The flag starts with 70
```

The output was not very useful by itself, so I decided to analyze the program more closely using GDB.

## Disassembling `main`

I opened the executable in GDB:

```bash
gdb /mnt/c/Users/nhatt/Downloads/asciiftw
```

I first tried:

```gdb
set dis intel
```

but GDB reported that the command was ambiguous. The correct command was:

```gdb
set disassembly-flavor intel
```

Then I disassembled the `main` function:

```gdb
disas main
```

The important part of the disassembly looked like this:

```asm
0x0000000000001184 <+27>:    mov    BYTE PTR [rbp-0x30],0x70
0x0000000000001188 <+31>:    mov    BYTE PTR [rbp-0x2f],0x69
0x000000000000118c <+35>:    mov    BYTE PTR [rbp-0x2e],0x63
0x0000000000001190 <+39>:    mov    BYTE PTR [rbp-0x2d],0x6f
0x0000000000001194 <+43>:    mov    BYTE PTR [rbp-0x2c],0x43
0x0000000000001198 <+47>:    mov    BYTE PTR [rbp-0x2b],0x54
0x000000000000119c <+51>:    mov    BYTE PTR [rbp-0x2a],0x46
0x00000000000011a0 <+55>:    mov    BYTE PTR [rbp-0x29],0x7b
0x00000000000011a4 <+59>:    mov    BYTE PTR [rbp-0x28],0x41
0x00000000000011a8 <+63>:    mov    BYTE PTR [rbp-0x27],0x53
0x00000000000011ac <+67>:    mov    BYTE PTR [rbp-0x26],0x43
0x00000000000011b0 <+71>:    mov    BYTE PTR [rbp-0x25],0x49
0x00000000000011b4 <+75>:    mov    BYTE PTR [rbp-0x24],0x49
0x00000000000011b8 <+79>:    mov    BYTE PTR [rbp-0x23],0x5f
0x00000000000011bc <+83>:    mov    BYTE PTR [rbp-0x22],0x49
0x00000000000011c0 <+87>:    mov    BYTE PTR [rbp-0x21],0x53
0x00000000000011c4 <+91>:    mov    BYTE PTR [rbp-0x20],0x5f
0x00000000000011c8 <+95>:    mov    BYTE PTR [rbp-0x1f],0x45
0x00000000000011cc <+99>:    mov    BYTE PTR [rbp-0x1e],0x41
0x00000000000011d0 <+103>:   mov    BYTE PTR [rbp-0x1d],0x53
0x00000000000011d4 <+107>:   mov    BYTE PTR [rbp-0x1c],0x59
0x00000000000011d8 <+111>:   mov    BYTE PTR [rbp-0x1b],0x5f
0x00000000000011dc <+115>:   mov    BYTE PTR [rbp-0x1a],0x33
0x00000000000011e0 <+119>:   mov    BYTE PTR [rbp-0x19],0x43
0x00000000000011e4 <+123>:   mov    BYTE PTR [rbp-0x18],0x46
0x00000000000011e8 <+127>:   mov    BYTE PTR [rbp-0x17],0x34
0x00000000000011ec <+131>:   mov    BYTE PTR [rbp-0x16],0x42
0x00000000000011f0 <+135>:   mov    BYTE PTR [rbp-0x15],0x46
0x00000000000011f4 <+139>:   mov    BYTE PTR [rbp-0x14],0x41
0x00000000000011f8 <+143>:   mov    BYTE PTR [rbp-0x13],0x44
0x00000000000011fc <+147>:   mov    BYTE PTR [rbp-0x12],0x7d
```

This immediately looked interesting because the program was storing individual hexadecimal byte values sequentially on the stack.

The stack frame reserves `0x30` bytes:

```asm
sub rsp,0x30
```

and the values from `[rbp-0x30]` onward are filled one byte at a time.

Since these values are ASCII codes, they can be directly converted into characters. For example:

```text
0x70 = 'p'
0x69 = 'i'
0x63 = 'c'
0x6f = 'o'
0x43 = 'C'
0x54 = 'T'
0x46 = 'F'
0x7b = '{'
```

This already gives:

```text
picoCTF{
```

which strongly indicates that this region of memory contains the flag.

## Inspecting the Stack

I also noticed these instructions:

```asm
0x0000000000001200 <+151>:   movzx  eax,BYTE PTR [rbp-0x30]
0x0000000000001204 <+155>:   movsx  eax,al
```

At first, I placed a breakpoint at:

```gdb
break *main+158
```

because I wanted to inspect the value of `eax`. However, this turned out not to be necessary for solving the challenge.

I also checked the memory near `[rbp-0x8]`:

```gdb
x/8xb $rbp-0x8
```

which produced:

```text
0x7fffffffdeb8: 0x00    0xb4    0xaa    0xbe    0xbe    0x30    0x9b    0x76
```

There was nothing particularly useful there, so I focused on the region starting at `[rbp-0x30]`.

I inspected the bytes as characters using:

```gdb
x/18cb $rbp-0x30
```

GDB showed:

```text
0x7fffffffde90: 112 'p' 105 'i' 99 'c' 111 'o' 67 'C' 84 'T' 70 'F' 123 '{'
0x7fffffffde98: 65 'A' 83 'S' 67 'C' 73 'I' 73 'I' 95 '_' 73 'I' 83 'S'
0x7fffffffdea0: 95 '_' 69 'E'
```

This confirmed that the bytes were indeed forming a readable ASCII string.

Instead of examining every byte individually, I realized that GDB could display the entire memory region as a null-terminated string.

I used:

```gdb
x/s $rbp-0x30
```

and obtained:

```text
0x7fffffffde90: "picoCTF{ASCII_IS_EASY_3CF4BFAD}"
```

## Flag

```text
picoCTF{ASCII_IS_EASY_3CF4BFAD}
```

## Conclusion

The program constructs the flag directly on the stack by storing each character as its hexadecimal ASCII value. By disassembling `main`, identifying the sequence of `mov BYTE PTR` instructions, and examining the stack memory as a string, the entire flag can be recovered easily.

This challenge is a good introduction to reading x86-64 assembly and understanding how ASCII values can be used to construct strings in memory.
