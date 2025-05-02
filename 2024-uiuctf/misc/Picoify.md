---
ai_date: '2025-04-27 05:22:04'
ai_summary: Challenge required implementing a Huffman compression algorithm in PIC
  assembly, with a focus on bitwise manipulation for array access.
ai_tags:
- huffman
- bitwise
- compression
created: 2024-06-30T20:40
updated: 2024-07-07T23:06
---

**FAILED TO SOLVE**

This is a interesting challenge, because its a coding challenge instead of a cybersecurity one.

Basically we have to write a data compressor in PIC assembly.

## Huffman

After a simple frequency analysis of the possible output data, we can construct a Huffman tree.[^1]
[^1]: [Huffman Coding | Greedy Algo-3 - GeeksforGeeks](https://www.geeksforgeeks.org/huffman-coding-greedy-algo-3/)

```python
import heapq
import json

chrs = [32, 48, 49, 50, 51, 52, 53, 54, 55, 56, 65, 66, 67, 68, 69, 70, 71,
        72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90]

freq = {32: 17339, 82: 8411, 78: 8320, 51: 6678, 69: 6651, 76: 5575, 67: 5123, 49: 4717, 73: 4703, 65: 4633, 52: 4621, 83: 4477, 68: 4475, 53: 4349, 84: 4221, 55: 4075, 79: 3712,
        48: 3632, 80: 3485, 77: 3317, 85: 3234, 72: 2495, 89: 1795, 70: 1588, 71: 1437, 54: 1433, 86: 1431, 87: 1111, 75: 958, 66: 954, 56: 945, 88: 416, 74: 308, 81: 244, 90: 110, 50: 99}


class node:
    def __init__(self, freq, symbol, left=None, right=None):
        self.freq = freq
        self.symbol = symbol
        self.left = left
        self.right = right
        self.huff = ''

    def __lt__(self, nxt):
        return self.freq < nxt.freq


mapping = {}


def printNodes(node, val=''):
    newVal = val + str(node.huff)
    if (node.left):
        printNodes(node.left, newVal)
    if (node.right):
        printNodes(node.right, newVal)
    if (not node.left and not node.right):
        print(f"{node.symbol} -> {newVal}")
        mapping[node.symbol] = newVal


chars = [chr(x) for x in chrs]
freq = [freq[x] for x in chrs]

nodes = []

for x in range(len(chars)):
    heapq.heappush(nodes, node(freq[x], chars[x]))

while len(nodes) > 1:
    left = heapq.heappop(nodes)
    right = heapq.heappop(nodes)

    left.huff = 0
    right.huff = 1
    newNode = node(left.freq+right.freq, left.symbol+right.symbol, left, right)
    heapq.heappush(nodes, newNode)

printNodes(nodes[0])
json.dump(mapping, open('huffman-mapping.json', 'w'))
```

## Compression

Sadly I failed to learn assembly in the short span of time and contracted acute cephalalgia attempting so.

I did try my best to create a bitwise Huffman compressor. The array access would be possible with big continuous memory blocks and lookup functions that uses `addwf PCL,f` to jump forward onto a `retlw` instruction.

```python
mapping = [18, 7, 55, 4, 19, 9, 0, 6, 26, 0, 0, 0, 0, 0, 0, 0, 0, 11, 90, 15, 25, 8,
           28, 32, 23, 27, 183, 58, 31, 12, 14, 10, 2, 311, 1, 3, 22, 16, 119, 122, 439, 60, 567]
bit_len = [5, 5, 10, 4, 5, 5, 6, 5, 7, 0, 0, 0, 0, 0, 0, 0, 0, 5, 7, 5,
           5, 4, 6, 6, 6, 5, 9, 7, 5, 5, 4, 5, 5, 9, 4, 5, 5, 5, 7, 7, 9, 6, 10]

outgoing_stream = b''

buf = 0
buf_len = 0


def write_bit_to_buf(b):
    global buf, buf_len, outgoing_stream
    buf <<= 1
    buf |= b & 1
    buf_len += 1
    if buf_len == 8:
        outgoing_stream += bytes([buf])
        buf = 0
        buf_len = 0


def write_to_bytes(val, bits):
    while True:
        bits -= 1
        b = val & 1
        val >>= 1
        write_bit_to_buf(b)
        if bits == 0:
            break


def handle_byte(byte):
    if byte == 0x20:
        write_to_bytes(0b101, 3)
    else:
        write_to_bytes(mapping[byte-48], bit_len[byte-48])


def flush():
    write_to_bytes(0, 8-buf_len)


ori = generate_data(8)
print("ori =", ori)
for i in ori:
    handle_byte(i)
flush()
```

## ~~Depression~~ Decompression

Since decompression is done in python, it is much easier, just use bit strings.

```python
inp = data  # type: ignore
huffman_decoding = {'000000': '6', '000001': 'G', '00001': 'U', '0001': 'E', '0010': '3', '00110': 'M', '001110': 'F', '001111': 'Y', '01000': 'P', '01001': '0', '01010': 'O', '0101100': '8', '0101101': 'B', '0101110': 'K', '0101111': 'W', '01100': '7', '01101': 'T', '0111': 'N',
                    '1000': 'R', '10010': '5', '10011': 'D', '101': ' ', '11000': 'S', '11001': '4', '11010': 'A', '11011': 'I', '11100': '1', '111010': 'H', '1110110000': '2', '1110110001': 'Z', '111011001': 'Q', '111011010': 'J', '111011011': 'X', '1110111': 'V', '11110': 'C', '11111': 'L'}
def bytes_to_bit_string(byte_data):
    return ''.join(f'{byte:08b}' for byte in byte_data)
def decode_huffman(bit_string):
    decoded_str = ''
    buffer = ''
    for bit in bit_string:
        buffer += bit
        if buffer in huffman_decoding:
            decoded_str += huffman_decoding[buffer]
            buffer = ''
    return decoded_str
decoded_message = decode_huffman(bytes_to_bit_string(inp))
print(decoded_message)
```