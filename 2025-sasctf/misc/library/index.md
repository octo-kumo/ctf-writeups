---
created: 2025-05-24T17:50
updated: 2025-05-24T18:04
---

I already worked on a client mod for the other minecraft challenge so I just added a snippet to export all item framed books.

(In fact I tried to make my minecraft forensics project work on new version mca but is faced with dead library after dead library, I don't want to re-implement the entire NBT parser so I moved on)

```java
ClientTickEvents.END_CLIENT_TICK.register(client -> {
	if (client.world == null || client.player == null) return;
	if (++tickCounter % 20 != 0) return;
	Box detectionBox = Box.of(client.player.getPos(), 10, 10, 10);
	for (ItemFrameEntity frame : client.world.getEntitiesByClass(ItemFrameEntity.class, detectionBox, e -> true)) {
		UUID id = frame.getUuid();
		if (!processedFrames.add(id)) continue;
		var stack = frame.getHeldItemStack();
		if (stack.getItem() == Items.WRITTEN_BOOK) {
			WrittenBookContentComponent writtenBookContentComponent = stack.get(DataComponentTypes.WRITTEN_BOOK_CONTENT);
			if (writtenBookContentComponent != null) {
				StringBuilder sb = new StringBuilder();
				sb.append("Title: ").append(writtenBookContentComponent.title()).append("\n");
				sb.append("Author: ").append(writtenBookContentComponent.author()).append("\n\n");
				List<RawFilteredPair<Text>> pages = writtenBookContentComponent.pages();
				for (int i = 0; i < pages.size(); i++) {
					sb.append("Page ").append(i + 1).append(":\n").append(pages.get(i).raw().getLiteralString()).append("\n\n");
				}

				try {
					Path configDir = FabricLoader.getInstance().getConfigDir();
					Path outputDir = configDir.resolve("book_frames");
					Files.createDirectories(outputDir);
					Path filePath = outputDir.resolve(id + ".txt");
					Files.writeString(filePath, sb.toString(), StandardCharsets.UTF_8, StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING);
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}
});
```

Couldn't find the flag in plaintext so I looked around, then I found the suspicious pattern of some books having pages like this.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1748123456/2025/05/19eaf8603b98dba32f758417cec54614.png)

Wrote a script to solve it.

```python
import os
import re
import glob

pattern = re.compile(
    r"Page (\d+):\nLet's append (.) to the end of the new book and continue reading part (\d+) on page (\d+)"
)

base = re.compile(
    r"Title: RawFilteredPair\[raw=Moby-Dick; or, The Whale - (\d+), filtered=Optional.empty\]"
)
script_dir = os.path.dirname(os.path.abspath(__file__))
txt_files = glob.glob(os.path.join(script_dir, '*.txt'))

char_map = {}
for filepath in txt_files:
    with open(filepath, 'r', encoding='utf-8') as f:
        content = f.read()
    page_no = list(base.finditer(content))[0].group(1)
    for match in pattern.finditer(content):
        _p, ch, part, page = match.group(1), match.group(2), match.group(3), match.group(4)
        char_map[page_no+"_"+_p] = (ch, part+"_"+page)
print("char_map:", char_map)

successors = set(s for _, s in char_map.values())
nodes = set(char_map.keys())
heads = nodes - successors

start = list(heads)[0]
s = ""
node = start
visited = set()
while node in char_map and node not in visited:
    visited.add(node)
    ch, nxt = char_map[node]
    s += ch
    node = nxt
print(s)
```

```flag
SAS{0ld_5ch00l_r37r13v4l_4ugm3nt3d_g3n3r4t10n}
```
