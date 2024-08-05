---
created: 2024-06-15T01:22
updated: 2024-08-04T19:45
solves: 18
points: 493
---

```
D:\AppData\Local\Temp\Roblox\sounds\RBX473C96F42BB81750C9BD26E83FE1D393 - text/plain
D:\AppData\Local\Temp\Roblox\sounds\RBX8E5A0F755FB6959D501DECD0B37B7675 - audio/x-wav
D:\AppData\Local\Temp\Roblox\sounds\RBXC7C8BF0AEB9C85B8C56D7C4468865B2C - text/plain
```

```python
"D:\AppData\Local\Temp\Roblox\sounds\RBXC7C8BF0AEB9C85B8C56D7C4468865B2C"
import os
import random
import string

def generate_hex_string(length=32):
    """Generate a random hexadecimal string of a given length."""
    return ''.join(random.choices('0123456789ABCDEF', k=length))

def rename_files_in_directory():
    """Rename all files in the current directory with a specific pattern, without retaining the extension."""
    for filename in os.listdir():
        if os.path.isfile(filename):  # Check if it's a file and not a directory
            # Generate new filename starting with 'RBX' and followed by a random hex string
            new_name = 'RBX' + generate_hex_string()
            # Rename the file
            os.rename(filename, new_name)
            print(f'Renamed "{filename}" to "{new_name}"')

# Run the function to rename files
rename_files_in_directory()

```

```python
"D:\AppData\Local\Temp\Roblox\sounds\RBX473C96F42BB81750C9BD26E83FE1D393"
import os
import random
import string
import time

def generate_hex_string(length=32):
    """Generate a random hexadecimal string of a given length."""
    return ''.join(random.choices('0123456789ABCDEF', k=length))

def rename_files_in_directory(script_name):
    """Rename all files in the current directory with a specific pattern, without retaining the extension,
    excluding the script itself, and update the last modified date to today's date."""
    # Get the current time in seconds since the epoch
    now = time.time()
    for filename in os.listdir():
        if os.path.isfile(filename) and filename != script_name:  # Check if it's a file and not the script itself
            # Generate new filename starting with 'RBX' and followed by a random hex string
            new_name = 'RBX' + generate_hex_string()
            # Rename the file
            os.rename(filename, new_name)
            # Update the last access and modification times
            os.utime(new_name, (now, now))
            print(f'Renamed "{filename}" to "{new_name}" and updated the last modified date to today.')

# Pass the script filename to the function
rename_files_in_directory('ahh.py')
```

Idk.
