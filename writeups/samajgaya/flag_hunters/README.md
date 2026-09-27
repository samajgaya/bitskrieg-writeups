# Flag Hunters - Reverse Enginerring (easy)

## Overview
Attached is a python script that when executed prints some sort of song lyrics.
```sh
flag_hunters $ python3 lyric-reader.py                         
Command line wizards, we’re starting it right,
Spawning shells in the terminal, hacking all night.
Scripts and searches, grep through the void,
Every keystroke, we're a cypher's envoy.
...
```

Inspecting the python script, the first few lines attach a _'secret_intro'_,
at the start of the song lyric, which contains the flag.

There is also a reader() function, that accepts the song lyric and
startLabel as arguments. It seems to "execute" the song lyric as a sort
of VM, with instructions like `RETURN` and `[REFRAIN]` / `REFRAIN`,
where `lip` is the "instruction pointer".  The first for loop assigns,
refrain and refrain_return, which are at the start (after secret_lyric)
of the song, while the second loop is the main logic of the VM.

```python
line_count = 0
lip = start
while not finished and line_count < MAX_LINES:
line_count += 1
for line in song_lines[lip].split(';'):
  if line == '' and song_lines[lip] != '':
    continue
  if line == 'REFRAIN':
    song_lines[refrain_return] = 'RETURN ' + str(lip + 1)
    lip = refrain
  elif re.match(r"CROWD.*", line):
    crowd = input('Crowd: ')
    song_lines[lip] = 'Crowd: ' + crowd
    lip += 1
  elif re.match(r"RETURN [0-9]+", line):
    lip = int(line.split()[1])
  elif line == 'END':
    finished = True
  else:
    print(line, flush=True)
    time.sleep(0.5)
    lip += 1
```

## Solving
As mentioned before lines containing `RETURN` and `REFRAIN` are treated
specially; but along with that lines starting with `CROWD` are also special
and `RETURN` takes a number argument for the position to return to.

From here it is evident that we need to somehow insert and read a `RETURN 0` to
get the secret_lyric to print. Lines starting with `CROWD` give an entry to
user input, that's our gateway to inserting the `RETURN 0`.

Though `RETURN [0-9]+` means that the line must start with "RETURN", and
`CROWD` always prepends 'Crowd: ' to our input (and since `input()` cannot
take in multiple lines), we need to get creative.

Pay attention to the loop statement
```python
for line in song_lines[lip].split(';'):
```
Each line is further split by a ';', which means that even in a single line,
we can insert multple "instructions". Since we want `RETURN` in its own line,
it just needs a ';' before it, as each line gets split by that delimiter in
the VM.
So our final payload is
``` ;RETURN 0 ```

Which prints the secret lyric, containing our flag!
