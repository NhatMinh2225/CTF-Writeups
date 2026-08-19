# picoCTF 2025 - Flag Hunters

- **Category:** Reverse Engineering
- **Difficulty:** Easy
- **Author:** syreal

---

## Challenge Description

> Lyrics jump from verses to the refrain kind of like a subroutine call. There's a hidden refrain this program doesn't print by default. Can you get it to print it? There might be something in it for you.

- **Files provided:** `lyric-reader.py`
- **Instance connection:** Netcat (`nc verbal-sleep.picoctf.net <port>`)

---

## Static Analysis

Inspecting the provided source code `lyric-reader.py`:

1. **Hidden Flag Location:**  
   The flag is loaded into `secret_intro` at the very beginning of the full lyrics string (`song_flag_hunters`):

   ```python
   flag = open('flag.txt', 'r').read()

   secret_intro = \
   '''Pico warriors rising, puzzles laid bare,
   Solving each challenge with precision and flair.
   With unity and skill, flags we deliver,
   The ether’s ours to conquer, '''\
   + flag + '\n'

   song_flag_hunters = secret_intro + '''...'''
   ```

2. **Execution Flow:**  
   At the bottom of the script, execution starts at the `[VERSE1]` label instead of the beginning:

   ```python
   reader(song_flag_hunters, '[VERSE1]')
   ```
   Because `secret_intro` is placed before `[VERSE1]` (lines 0 to 4), the default execution flow completely skips the flag.

3. **Vulnerability in `reader()`:**  
   - The interpreter parses each line into sub-commands using semicolon delimiters:
     ```python
     for line in song_lines[lip].split(';'):
     ```
   - It supports explicit jumps via regex pattern `RETURN [0-9]+`, which updates the line pointer `lip`:
     ```python
     elif re.match(r"RETURN [0-9]+", line):
         lip = int(line.split()[1])
     ```
   - User input at the `CROWD` prompt is written directly into `song_lines[lip]` without sanitization:
     ```python
     elif re.match(r"CROWD.*", line):
         crowd = input('Crowd: ')
         song_lines[lip] = 'Crowd: ' + crowd
         lip += 1
     ```

---

## Exploitation

Since input is not sanitized and the interpreter splits commands by `;`, we can inject a control command.

When prompted with `Crowd: `, entering:
```text
;RETURN 0
```
modifies the current line buffer to:
```text
Crowd: ;RETURN 0
```

When execution later iterates through this line, `split(';')` executes `RETURN 0`, resetting the line pointer `lip` to line `0` (the beginning of `secret_intro`).

### Steps to Reproduce

1. Connect to the challenge instance using Netcat:
   ```bash
   nc verbal-sleep.picoctf.net 56894
   ```
2. Wait until the first refrain reaches the `Crowd: ` prompt.
3. Send the payload:
   ```text
   ;RETURN 0
   ```
4. The program loops back to line 0 and prints the full lyrics including the flag.

---

## Flag

```text
picoCTF{70637h3r_f0r3v3r_0099cf61}
```
