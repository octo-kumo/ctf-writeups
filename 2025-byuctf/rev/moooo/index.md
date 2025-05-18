---
ai_date: 2025-05-17 21:38:50
ai_summary: Brute-forced Cow language by symbol manipulation and integer arithmetic to solve for missing characters
ai_tags:
  - brute-force
  - symbolic
  - arithmetics
created: 2025-05-17T07:11
points: 481
solves: 53
title: moooo
updated: 2025-05-17T21:52
---

This is the Cow language: [COW - Esolang](https://esolangs.org/wiki/COW).

I made a char-by-char bruteforcer based on maximum instruction index.

Ok maybe not really a char-by-char because I am more doing something like a Dijkstra with a priority queue.

I keep the last character and repeat it as input (`}`) because otherwise for some reason it will just keep going `byuctf{!!!!!!!!!!!!`.

However, after a while, I get stuck and can't get the just the last part.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1747483320/2025/05/6901bff22161b5664a6f088076f6d912.png)

It appears that all inputs works perfectly fine, and the search never ends.

Idk how to improve my bruteforcer so I am just going to actually brute force the last few characters.

![Screenshot 2025-05-17 091626.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1747488830/2025/05/9be5d6e05d172893931ff099456dfb01.png)

I tried parallelized brute force but it just didn't work.

Depressed, I decided to import `sympy` and manually go through each character.

```python
def _handle_current_as_instruction(self):
	idx = self._cells[self._ptr]
	if not isinstance(idx, int):
		print(idx)
	if idx < 0 or idx >= len(self.available_commands) or idx == 3:
		raise Exception(f"Invalid command index {idx} in cell for mOO")
	self._commands_to_functions[idx]()
	
def _print_or_read_char(self):
	if self._cells[self._ptr] == 0:
		if len(self.buffer) == 0:
			x = symbols('x'+str(len(self.unknowns)))
			self._cells[self._ptr] = x
		else:
			self._cells[self._ptr] = self.buffer[0]
			self.buffer = self.buffer[1:]
	else:
		if not isinstance(self._cells[self._ptr], int):
			print(self._cells[self._ptr])
		self.output += bytes([self._cells[self._ptr]])
		print(self.output)
		pass
```

Basically after the first `b'byuctf{moo_mooo_moooo_mooooooo_mo'`, I will feed it a sympy symbol instead of actual data.

And on output, if I get a sympy symbol back, I just print it out.

Interestingly, it was some very simple calculation, just adding / subtracting by some fixed amount.

Since I know that $x_{0}+\Delta\in{\{\text{gotem}\}}$, I can just calculate the correct character for `x0`.

```bash
$ python solver.py
x0 - 109 # i assumed this to be o = 111
$ python solver.py
x0 + 8  # -> _
$ python solver.py
x0 + 3  # -> l
$ python solver.py
x0 + 5  # -> o
$ python solver.py
x0 - 7  # -> l
$ python solver.py
x0 - 16 # -> }
$ python solver.py
b'Moo. Moo moo mooo!\ngotem'
```

And I got it!

```flag
byuctf{moo_mooo_moooo_mooooooo_moo_lol}
```

## solver

After realising that I was pretty close with the original solver, I modified it slightly such that it can solve from start to finish in just a few seconds.

```python
from queue import PriorityQueue
import string
from typing import List, Optional

# mostly takne from https://github.com/Mikhail57/cow-interpreter/blob/master/interpreter.py
class CowInterpreter:
    # Twelve COW commands, mapped by index
    available_commands = [
        "moo",  # 0: loop end ("moo")
        "mOo",  # 1: move pointer left
        "moO",  # 2: move pointer right
        "mOO",  # 3: execute cell as instruction
        "Moo",  # 4: input/output char
        "MOo",  # 5: decrement cell
        "MoO",  # 6: increment cell
        "MOO",  # 7: loop start
        "OOO",  # 8: zero cell
        "MMM",  # 9: copy/paste register
        "OOM",  # 10: print int
        "oom",  # 11: read int
    ]
    max_idx = 0
    output = b""

    def __init__(self, buffer=b"byuctf{") -> None:
        self.buffer = buffer
        self._cells: List[int] = [0] * 30000
        self._commands: List[int] = []
        self._ptr: int = 0         # data pointer
        self._cmd_ptr: int = 0     # instruction pointer
        self._register: Optional[int] = None

        # map command indices to handler methods
        self._commands_to_functions = {
            0: self._handle_loop_end,
            1: self._move_to_prev_cell,
            2: self._move_to_next_cell,
            3: self._handle_current_as_instruction,
            4: self._print_or_read_char,
            5: self._decrement_current_cell,
            6: self._increment_current_cell,
            7: self._handle_loop_start,
            8: self._zero_current_cell,
            9: self._copy_or_paste_register,
            10: self._print_int,
            11: self._read_int,
        }

    def interpret(self, code: str):
        # tokenize program: split on whitespace and filter valid commands
        tokens = code.split()
        self._commands = [self.available_commands.index(t)
                          for t in tokens if t in self.available_commands]
        self._ptr = 0
        self._cmd_ptr = 0

        while self._cmd_ptr < len(self._commands):
            cmd = self._commands[self._cmd_ptr]
            handler = self._commands_to_functions.get(cmd)
            if handler is None:
                raise Exception(f"Unknown command index {cmd}")
            handler()
            self._cmd_ptr += 1
            self.max_idx = max(self.max_idx, self._cmd_ptr)

    def _handle_loop_start(self):
        # "MOO": if current cell is zero, skip to matching loop end ("moo")
        if self._cells[self._ptr] == 0:
            # find matching loop-end
            self._cmd_ptr = self._get_loop_end(self._cmd_ptr)

    def _handle_loop_end(self):
        # "moo": if current cell != 0, jump back to matching loop start ("MOO")
        if self._cells[self._ptr] != 0:
            self._cmd_ptr = self._get_loop_start(self._cmd_ptr)

    def _zero_current_cell(self):
        self._cells[self._ptr] = 0

    def _move_to_prev_cell(self):
        if self._ptr == 0:
            raise IndexError("Data pointer moved before start of tape")
        self._ptr -= 1

    def _move_to_next_cell(self):
        self._ptr += 1
        if self._ptr >= len(self._cells):
            # expand tape if needed
            self._cells.append(0)

    def _handle_current_as_instruction(self):
        # "mOO": treat cell value as command index and execute it
        idx = self._cells[self._ptr]
        if idx < 0 or idx >= len(self.available_commands) or idx == 3:
            # idx == 3 would recurse infinitely
            raise Exception(f"Invalid command index {idx} in cell for mOO")
        # execute without advancing cmd_ptr twice
        self._commands_to_functions[idx]()

    def _print_or_read_char(self):
        # "Moo": if cell != 0, output char, else read char
        if self._cells[self._ptr] == 0:
            self._cells[self._ptr] = self.buffer[0]
            if len(self.buffer) > 1:
                self.buffer = self.buffer[1:]
        else:
            self.output += bytes([self._cells[self._ptr]])
            pass

            # print(chr(self._cells[self._ptr]), end='')

    def _decrement_current_cell(self):
        self._cells[self._ptr] -= 1

    def _increment_current_cell(self):
        self._cells[self._ptr] += 1

    def _copy_or_paste_register(self):
        # "MMM": copy into register or paste from register
        if self._register is None:
            self._register = self._cells[self._ptr]
        else:
            self._cells[self._ptr] = self._register
            self._register = None

    def _print_int(self):
        # "OOM": print integer with no newline

        pass
        # print(self._cells[self._ptr], end='')

    def _read_int(self):
        # "oom": read integer into cell
        val = chr(self.buffer[0])
        if len(self.buffer) > 1:
            self.buffer = self.buffer[1:]
        self._cells[self._ptr] = int(val)

    def _get_loop_end(self, start_idx: int) -> int:
        # find matching "moo" for "MOO" at start_idx
        depth = 1
        i = start_idx
        while depth > 0:
            i += 1
            if i >= len(self._commands):
                raise IndexError("Loop start without matching loop end")
            if self._commands[i] == 7:  # another MOO
                depth += 1
            elif self._commands[i] == 0:  # moo
                depth -= 1
        return i

    def _get_loop_start(self, end_idx: int) -> int:
        # find matching "MOO" for "moo" at end_idx
        depth = 1
        i = end_idx
        while depth > 0:
            i -= 1
            if i < 0:
                raise IndexError("Loop end without matching loop start")
            if self._commands[i] == 0:  # moo
                depth += 1
            elif self._commands[i] == 7:  # MOO
                depth -= 1
        return i


with open("main.cow", 'r') as f:
    code = f.read()
prog = b"byuctf{"
queue = PriorityQueue()
queue.put((0, prog))

prev_max = 0

max_by_pos = dict()

while not queue.empty():
    idx, prog = queue.get()
    for i in string.digits + string.ascii_letters + string.punctuation:
        inp = prog + i.encode() + b'}'
        interpreter = CowInterpreter(inp)
        try:
            interpreter.interpret(code)
            print(inp, '->', interpreter.output[19:], interpreter.max_idx)
        except Exception as e:
            pass
        if b'gotem' in interpreter.output:
            print("Found flag:", inp.decode())
            exit()
        ans = b"gotem"
        extra = 0
        for j in range(len(interpreter.output)-19):
            if interpreter.output[19+j] == ans[j]:
                extra += 2
            else:
                break
        interpreter.max_idx += extra
        if interpreter.max_idx > prev_max:
            print('new best', interpreter.max_idx, inp.decode())
            prev_max = interpreter.max_idx
        if interpreter.max_idx >= prev_max and len(prog + i.encode()) <= 38:
            queue.put((-interpreter.max_idx+len(prog + i.encode()), prog + i.encode()))
```