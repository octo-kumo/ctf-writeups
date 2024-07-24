---
created: 2024-06-14T18:13
updated: 2024-06-15T04:23
---

> Flag will be printed out straight away when you run the binary.

Decompiling the binary gives us a simple loop with a sleep statement.

```cpp
undefined8 main(void) {
	int64_t var_ch;
	for (var_ch._0_4_ = 0; (int32_t)var_ch < 0x8ae; var_ch._0_4_ = (int32_t)var_ch + 0xca) {
		printf("%.*s\n", 0xca, flag + (int32_t)var_ch);
		sleep(0xb1aaf);
	}
	return 0;
}
```

We can just patch the executable by changing the value `0xb1aaf` to 1 or 0.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718439785/2024/06/ae6a7ae11a4d1b4d16b212e92ae58d13.png)

Voilà, the flag.

```flag
vsctf{1nTr0_r3v3R51ng!}
```
