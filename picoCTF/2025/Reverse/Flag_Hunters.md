picoCTF 2025 - Flag Hunters

Category: Reverse Engineering
Difficulty: Easy
Author: syreal

So the challenge gives you a python file (lyric-reader.py) and a netcat connection. The description says something about a hidden refrain that the program doesn't print by default and that there might be something in it for us, which is basically a big hint.

Looking at the code, the flag gets read in and stuck into a variable called secret_intro, then that gets glued onto the front of the whole song:

flag = open('flag.txt', 'r').read()

secret_intro = \
'''Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, '''\
+ flag + '\n'

So the flag is literally just sitting at the top of the lyrics. The problem is the reader function gets called like this:

reader(song_flag_hunters, '[VERSE1]')

and the function only starts printing from whatever line startLabel points to, going forward. Since [VERSE1] comes after the secret intro in the text, normally the program just skips right over the flag part and you never see it.

Reading through the reader function more, there's this bit:

elif re.match(r"CROWD.*", line):
    crowd = input('Crowd: ')
    song_lines[lip] = 'Crowd: ' + crowd
    lip += 1

This is a "singalong" spot where the program takes whatever you type and drops it directly into the lyrics array. That felt like the way in.

Then right under it:

elif re.match(r"RETURN [0-9]+", line):
    lip = int(line.split()[1])

So if you can get a line like "RETURN 0" into the lyrics, it'll jump execution back to line 0, which is right where the secret intro (and the flag) lives.

First try, I just typed RETURN 0 at the Crowd prompt. Didn't work. Then I noticed this line:

for line in song_lines[lip].split(';'):

The lines get split on semicolons too, not just newlines. So my input needed to actually look like a command in that split, not just plain text. I typed:

;RETURN 0

and that did it. The empty part before the semicolon gets skipped, and RETURN 0 gets parsed as a real jump instruction, sending the reader back to the top where the secret intro prints out.

Flag: picoCTF{70637h3r_f0r3v3r_0099cf61}
