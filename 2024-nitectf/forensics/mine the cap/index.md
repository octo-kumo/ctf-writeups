---
created: 2024-12-15T07:14
updated: 2024-12-15T08:06
tags:
  - minecraft
solves: 4
points: 480
---

The packets are minecraft packets!

wiki.vg is down for some reason so I had to refer to [Protocol - wiki.vg](https://web.archive.org/web/20241129034846/https://wiki.vg/index.php?title=Protocol&oldid=16681#Multi_Block_Change) on internet archive.

I will be using pyshark to automate the pcap part.

```python [solve.py]
import pyshark
import os
from proto import DataPackDumperProtocol

protocol = DataPackDumperProtocol()
path = os.path.join(os.path.dirname(__file__), 'traffic.pcap')
pkts = pyshark.FileCapture(path, display_filter='tcp')
for pkt in pkts:
    try:
        if pkt.tcp.payload:
            p = bytes.fromhex(pkt.tcp.payload.replace(':', ''))
            fromClient = pkt.tcp.srcport != '25565'
            protocol.data_received(p, 'upstream' if fromClient else 'downstream')
    except AttributeError as e:
        pass
```

## reading the buffer

Much of the code is taken from [barneygale/quarry: Python library that implements the Minecraft network protocol and data types](https://github.com/barneygale/quarry).

The library itself is used for connecting to actual servers and I figured I'll just yoink their source code and improvise a little to make it read off static captures.

```python [proto.py]
def data_received(self, data, direction='downstream'):
	self.recv_buff.add(data)
	self.recv_buff.save()
	try:
		buff = self.recv_buff.unpack_packet(Buffer1_14, self.compress_threshold)
		name = self.get_packet_name(buff.unpack_varint(), direction)
		self.packet_received(buff, name)
	except BufferUnderrun:
		self.recv_buff.restore()
		return
	except zlib.error:
		print("Decompression failed")
		self.recv_buff.discard()
		return
```

## handshake (protocol version + compression)

Minecraft packets could be compressed and encrypted (gratefully it wasn't).

```python [proto.py]
def packet_handshake(self, buff):
	proto, addr, port, state = buff.unpack_varint(), buff.unpack_string(), buff.unpack("H"), buff.unpack_varint()
	print("handshake", f"{proto=}", f"{addr}:{port}", f"{state=}")
	self.protocol_mode = "status" if state == 1 else "login"
	
def packet_login_start(self, buff):
	print("login_start name='{}'".format(buff.unpack_string()))

def packet_login_set_compression(self, buff):
	self.compress_threshold = buff.unpack_varint()
	print("login_set_compression threshold={}".format(self.compress_threshold))
	
"""
handshake proto=753 localhost:25565 state=2
login_start name='lvert'
login_set_compression threshold=256
"""
```

We can see that protocol version is `753` which corresponds to Minecraft version `1.16.3`.
## block updates

I realised that there were quite a lot of block change events, perhaps the author built the flag?

So I listened to all the chunk events and saved all the chunk section data to individual files, then I also saved all the block update events as a list of `(x,y,z,blockid)`.

I will use this information to reconstruct the world later.

```python [proto.py]
def packet_chunk_data(self, buff):
	x, z, full = buff.unpack('ii?')
	bitmask = buff.unpack_varint()
	heightmap = buff.unpack_nbt()
	biomes = [buff.unpack_varint() for _ in range(buff.unpack_varint())]
	sections_length = buff.unpack_varint()
	sections = buff.read(sections_length)
	block_entities = [buff.unpack_nbt() for _ in range(buff.unpack_varint())]
	# print(f"chunk_data {x=} {z=} {full=} BE:{len(block_entities)} BIT:{bitmask}")
	if not full:
		return
	p = os.path.join(os.path.dirname(__file__), 'visualize', 'chunks', f"chunk_{x}_{z}")
	with open(p, 'wb') as f:
		f.write(sections)
	self.chunks[f'{x}_{z}'] = {
		"bitmask": bitmask,
	}
	with open(os.path.join(os.path.dirname(__file__), 'visualize', 'chunks.json'), 'w') as f:
		json.dump(self.chunks, f)

def packet_player_position_and_look(self, buff):
	p_pos_look = buff.unpack('dddff')
	print(f"player_position_and_look {p_pos_look=}")

def packet_block_change(self, buff):
	pos = buff.unpack('q')
	block = buff.unpack_varint()
	x = (pos >> 38) & 0x3FFFFFF
	y = (pos << 52 >> 52) & 0xFFF
	z = (pos << 26 >> 38) & 0x3FFFFFF
	# print(f"block_change {x=} {y=} {z=} {block=}")
	if block == 0:
		return
	self.changed_blocks.append({"x": x, "y": y, "z": z, "b": block})
	self.update_changed_blocks()

def packet_multi_block_change(self, buff):
	secp = buff.unpack('q')
	sectionX = (secp >> 42) & 0x3FFFFF
	sectionY = (secp << 44 >> 44) & 0xFFFFF
	sectionZ = (secp << 22 >> 42) & 0x3FFFFF
	buff.unpack('?')
	recordCount = buff.unpack_varint()
	for i in range(recordCount):
		blockData = buff.unpack_varint(max_bits=64)
		blockStateId = blockData >> 12
		blockLocalX = (blockData >> 8) & 0xF
		blockLocalZ = (blockData >> 4) & 0xF
		blockLocalY = blockData & 0xF
		x = sectionX * 16 + blockLocalX
		y = sectionY * 16 + blockLocalY
		z = sectionZ * 16 + blockLocalZ
		# print(f"multi_block_change {x=} {y=} {z=} {blockStateId=}")
		if blockStateId == 0:
			continue
		self.changed_blocks.append({"x": x, "y": y, "z": z, "b": blockStateId})
	self.update_changed_blocks()

def update_changed_blocks(self):
	with open(os.path.join(os.path.dirname(__file__), 'visualize', 'blocks.json'), 'w') as f:
		json.dump(self.changed_blocks, f)
```

## display

I used [PrismarineJS/prismarine-viewer: Web based viewer for servers and bots](https://github.com/PrismarineJS/prismarine-viewer) to view the world itself.

It loads all the chunk data from the previous step and also change the blocks recorded from block events.

```js [view.js]
const fs = require('fs')
const path = require('path')
const standaloneViewer = require('prismarine-viewer').standalone
const { Vec3 } = require('vec3')

const version = '1.16.3'

const World = require('prismarine-world')(version)
const Chunk = require('prismarine-chunk')(version)

const CHUNK_INFO = require('./chunks.json')
const world = new World((chunkX, chunkZ) => {
    const chunk = new Chunk()
    const fp = path.join(__dirname, 'chunks', `chunk_${chunkX}_${chunkZ}`);
    if (fs.existsSync(fp) && CHUNK_INFO[`${chunkX}_${chunkZ}`]) {
        try {
            const data = fs.readFileSync(fp);
            chunk.load(data, CHUNK_INFO[`${chunkX}_${chunkZ}`].bitmask)
            console.log("wowo!")
        } catch (e) {
            console.log(e)
        }
    }
    return chunk
})

const BLOCK_INFO = require('./blocks.json');
for (let b of BLOCK_INFO) {
    await world.setBlockStateId(new Vec3(b.x, b.y, b.z), b.b);
}

const viewer = standaloneViewer({ version, viewDistance: 12, world, center: new Vec3(273, 76.43021160700744, 395), port: 3000 });

viewer.update()
```

## result

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734265847/2024/12/3c21a4390a76d4a788e203775a2b0423.png)

In the end I wasn't able to get the full url? There must've been some mistakes I made.

Oh welp, my teammate just brute forced the 3 missing letters.

[https://pastebin.com/raw/BnF81jrU](https://pastebin.com/raw/BnF81jrU "https://pastebin.com/raw/BnF81jrU")

```flag
nite{bl0ck_8y_bl0ck+chu7k_8y_chu7k}
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734266032/2024/12/4015c5c5f80ca302a3d33e9d77a6ed98.png)
