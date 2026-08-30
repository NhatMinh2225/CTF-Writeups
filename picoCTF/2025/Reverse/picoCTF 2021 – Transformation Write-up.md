# picoCTF 2021 – Transformation Write-up

## Challenge Information

**Challenge:** Transformation  
**Category:** Reverse Engineering  
**Difficulty:** Easy  
**Author:** madStacks  
**CTF:** picoCTF 2021

### Description

> I wonder what this really is...

We are given an encoded file called `enc` and the following Python expression:

```python
''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])
```

Our goal is to recover the original flag.

---

## Initial Analysis

First, I downloaded the `enc` file and opened it in Notepad.

The content looked like this:

```text
灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸形㝦㘲捡㕽
```

At first, these looked like Chinese characters. I even tried putting them into Google Translate, but the result was meaningless.

That suggested that these characters were not actual Chinese text. Instead, they were probably Unicode characters produced by the encoding algorithm given in the challenge.

So I started analyzing the Python code.

---

## Understanding the Encoding

For readability, I rewrote the original one-liner like this:

```python
ans = []

for i in range(0, len(flag), 2):
    ans.append(chr((ord(flag[i]) << 8) + ord(flag[i + 1])))

''.join(ans)
```

The program processes the flag two characters at a time.

For every pair of characters:

```python
flag[i]
flag[i + 1]
```

it calculates:

```python
(ord(flag[i]) << 8) + ord(flag[i + 1])
```

The `<< 8` operation shifts the first character's ASCII value eight bits to the left.

Mathematically:

```text
x << 8 = x × 2^8 = x × 256
```

Therefore, each encoded Unicode character is created using:

```text
encoded_value = first_character × 256 + second_character
```

After producing one encoded character, the loop advances by two positions.

At this point, I assumed that the contents of `enc` were the output of this encoding process.

---

## Converting the Unicode Characters to Numbers

To understand the encoded data better, I wrote a small Python script to print the Unicode code point of every character:

```python
a = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸形㝦㘲捡㕽"

for i in a:
    print(ord(i))
```

Running it produced:

```text
28777
25455
17236
18043
12598
24418
26996
29535
26990
29556
13108
25695
28518
24376
24418
14182
13874
25441
13693
```

From the encoding formula, these values correspond to equations such as:

```text
x1 × 256 + x2 = 28777
x3 × 256 + x4 = 25455
x5 × 256 + x6 = 17236
...
x(n-1) × 256 + xn = 13693
```

where each `x` is the ASCII value of one character from the original flag.

---

## The Key Observation

At first, I thought this system could not be solved because every equation contains two unknown values, and the equations are independent from each other.

For example:

```text
256x1 + x2 = 28777
```

seems to have many possible integer solutions.

This confused me for a while.

Then I noticed an important property.

Every original flag character is an ASCII character, so its numerical value is less than `256`.

That means:

```text
0 <= x2 < 256
```

The equation:

```text
encoded_value = 256 × x1 + x2
```

is exactly the same form as Euclidean division:

```text
N = 256q + r
```

where:

```text
q = quotient
r = remainder
0 <= r < 256
```

For every positive integer `N`, dividing by `256` gives exactly one integer quotient and one remainder.

Therefore:

```text
x1 = encoded_value // 256
x2 = encoded_value % 256
```

So the encoding is completely reversible.

Another way to understand it is that the encoder combines two 8-bit characters into one 16-bit value:

```text
[first byte][second byte]
```

The first character occupies the upper 8 bits, and the second character occupies the lower 8 bits.

---

## Solving the Challenge

I wrote the following decoder:

```python
a = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸形㝦㘲捡㕽"

for i in a:
    x1 = ord(i) // 256
    x2 = ord(i) % 256

    print(chr(x1), chr(x2), sep="", end="")
```

Running the script gives:

```text
picoCTF{16_bits_inst34d_of_8_b7f62ca5}
```

---

## Flag

```text
picoCTF{16_bits_inst34d_of_8_b7f62ca5}
```

---

## Conclusion

The main idea of this challenge is understanding how two 8-bit character values are packed into a single 16-bit Unicode code point.

The encoder performs:

```python
encoded = (ord(char1) << 8) + ord(char2)
```

Since shifting left by eight bits is equivalent to multiplying by `256`, we can reverse it using integer division and modulo:

```python
char1 = chr(encoded // 256)
char2 = chr(encoded % 256)
```

This challenge was a useful introduction to bit shifting, Unicode code points, and the relationship between bit operations and Euclidean division.