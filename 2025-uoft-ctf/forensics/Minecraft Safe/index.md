---
ai_date: '2025-04-27 05:28:24'
ai_summary: Identified weak crypto in server.jar's key generation, used for encryption.
  Exploitation involved analyzing server source and understanding the encryption algorithm.
ai_tags:
- rsa
- weak-key
- crypto-exploitation
created: 2025-01-11T01:17
solves: 3
tags:
- unsolved
updated: 2025-01-12T20:04
---

Another minecraft pcap challenge?

After a bit of scanning, I realised that the communication is encrypted, keys were exchanged and `login_encryption_request` was triggered.

My old writeup does not handle crypto so it won't work anymore.
## packet reading.

```python
def packet_login_encryption_request(self, buff):
	p_server_id = buff.unpack_string()
	def unpack_array(b): return b.read(b.unpack_varint(max_bits=16))
	p_public_key = unpack_array(buff)
	p_verify_token = unpack_array(buff)

	self.public_key = crypto.import_public_key(p_public_key)
	self.verify_token = p_verify_token
	print(f"login_encryption_request {p_server_id=} {len(p_public_key)=} {p_verify_token=}")
	print(f"{self.public_key.public_numbers()=}")
```

After `login_encryption_request` and `login_encryption_response` the bytes are encrypted.

Here we have the public key, but breaking RSA is basically impossible.

```
self.public_key.public_numbers()=<RSAPublicNumbers(e=65537, n=130461568758849331505036135484014114043992644491371593716583161711577760699108564114633147451910541252566337707008956518324286452684332343041659860224701474509989481060885383984791170320441956230799736307044647353927078847073897040487341909288238283432126001878262186673758416165688244773338744035128369896631)>
```

So I decided to look into the server's source code.

## server source

We can obtain the server's version via decompiling, `1.21.4`.

Let's check the provided `server.jar` file with the official one.

```bash
$ sha256sum server.jar
9429e99dc0cd0993ef1664549245d497ae1615bbee428d71bccbb0f35b20d026  server.jar
$ sha256sum server.real.jar
1066970b09e9c671844572291c4a871cc1ac2b85838bf7004fa0e778e10f1358  server.real.jar
```

The hashes are different!

Let's examine their keypair generation.

```java
protected void V() {
	l.info("Generating keypair");

	try {
		this.ad = axx.b();
	} catch (axy $$0) {
		throw new IllegalStateException("Failed to generate key pair", $$0);
	}
}
```

```java [axx.class]
public static KeyPair b() throws axy {
	try {
		BigInteger var0;
		BigInteger var1;
		do {
			var0 = BigInteger.probablePrime(512, new SecureRandom());
			var1 = var0.shiftRight(256).or(var0.mod(BigInteger.TWO.pow(256)).shiftLeft(256));
		} while(!var1.isProbablePrime(100));

		KeyFactory var2 = KeyFactory.getInstance("RSA");
		return new KeyPair(var2.generatePublic(new RSAPublicKeySpec(var0.multiply(var1), new BigInteger("65537"))), var2.generatePrivate(new RSAPrivateKeySpec(var0.multiply(var1), (new BigInteger("65537")).modInverse(var0.subtract(BigInteger.ONE).multiply(var1.subtract(BigInteger.ONE))))));
	} catch (Exception var4) {
		throw new axy(var4);
	}
}
```

Meanwhile the real minecraft server does this.

```java [real/axx.class]
public static KeyPair b() throws axy {
	try {
		KeyPairGenerator $$0 = KeyPairGenerator.getInstance("RSA");
		$$0.initialize(1024);
		return $$0.generateKeyPair();
	} catch (Exception $$1) {
		throw new axy($$1);
	}
}
```

The server.jar we got has weak crypto!

$$
\begin{align}
p  & = H_1 || H_0= H_1 \cdot 2^{256} + H_0 \\
q  & = H_0 || H_1= H_0 \cdot 2^{256} + H_1 \\
N  & = (H_1 \cdot 2^{256} + H_0) \cdot (H_0 \cdot 2^{256} + H_1) \\
 & = H_1 \cdot H_0 \cdot 2^{512} + (H_1^2 + H_0^2) \cdot 2^{256} + H_0 \cdot H_1
\end{align}
$$

## breaking the crypto

*"By the ancient keys of cipher and code, I call forth Maximxls, the solver of enigmas, the breaker of chains! Let the digital winds carry my plea, and may your brilliance illuminate the darkest of cryptographic realms! Step forth, Maximxls, and let the challenges bow before your unyielding might!"*

My teammate @Maximxls helped me.

```python
N=130461568758849331505036135484014114043992644491371593716583161711577760699108564114633147451910541252566337707008956518324286452684332343041659860224701474509989481060885383984791170320441956230799736307044647353927078847073897040487341909288238283432126001878262186673758416165688244773338744035128369896631
p=12392410804664940156372899354158974363722257799949007097165521217543677200745499651061566614970955433161042932829539349273403482979240433003830514342967791     
q=10527537443298684008931726136394941222037097596968664524093225803422432813136644482794182361090071851369773768313662498322489166067545495287091879391949241     
d=120444591875645494952655925316390343931394097303056872878249907675954230604079583620792507844085742381960209697767320965885087077828791803343693331139593837503325523117472420712626828421711064671479313284923248605961051041209084189016216843605796126868866948541735980213741309193588914265655575714268414627473
p*q == N: True
```

## verification

Let's verify the author's uuid with his username.

```
login_start name='__toad_'
uuid='25f1fa23-0d8e-4e57-810d-e45f60c3bb74'
```

[Minecraft UUID / Username Converter](https://mcuuid.net/?q=__toad_)

```
25f1fa23-0d8e-4e57-810d-e45f60c3bb74
```

Yay it is correct!

## decoding

I got stuck here for 1 entire day.

I was able to salvage some info though.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736724521/2025/01/5a25efd5f9d0e4ffa410e702bf1945ce.png)

The chunks are intact in a ring pattern, not sure why.

The centre dots are blocks changed in block events.

I am forever getting bad packets, those packets start with bad lengths, causing the errors to cascade due to it consuming bytes of the next packet.

For example, right after compression headers start, I have to manually remove some data or it won't parse correctly (packet id not recognized.)

```python
if direction == 'upstream' and lost == b'/\x80-\x9be\x13,r)\x0f_\xf4\x9bs\x1f\x12:brand\x07vanilla\x10\x00\x00\x05en_us\x16\x00\x01\x7f\x01\x01\x01\x00g\xf3\xce\x06\xef\x8a\xc7\x84\xf4\xf4\xfe\xc9\x9d\x9f\x9fjore\x061.21.4':
	recv_buff.add(b'\x10\x00\x00\x05en_us\x16\x00\x01\x7f\x01\x01\x01\x00g\xf3\xce\x06\xef\x8a\xc7\x84\xf4\xf4\xfe\xc9\x9d\x9f\x9fjore\x061.21.4')
	recv_buff.save()
```

```
loading... kernel32
loading... kernel32

--- upstream init --- s_set_protocol 0x0
handshake proto=769 localhost:25565 state=2

--- upstream login --- s_login_start 0x0
login_start name='__toad_'

--- downstream login --- c_encryption_begin 0x1
login_encryption_request p_server_id='' len(p_public_key)=162
validate keys True
p_verify_token.hex()='0ec1d269'

--- upstream login --- s_encryption_begin 0x1

--- downstream login --- c_compress 0x3
login_set_compression threshold=256

--- downstream login --- c_success 0x2
uuid='25f1fa23-0d8e-4e57-810d-e45f60c3bb74'
name='__toad_'
property_name='textures'
property_value='ewogICJ0aW1lc3RhbXAiIDogMTczNDMwMjY5MjE3OSwKICAicHJvZmlsZUlkIiA6ICIyNWYxZmEyMzBkOGU0ZTU3ODEwZGU0NWY2MGMzYmI3NCIsCiAgInByb2ZpbGVOYW1lIiA6ICJfX3RvYWRfIiwKICAic2lnbmF0dXJlUmVxdWlyZWQiIDogdHJ1ZSwKICAidGV4dHVyZXMiIDogewogICAgIlNLSU4iIDogewogICAgICAidXJsIiA6ICJodHRwOi8vdGV4dHVyZXMubWluZWNyYWZ0Lm5ldC90ZXh0dXJlLzUwMWY1YmE1YTM2OGYwNTQ2OGU2NzU4MDg2NDQ0ZmUzMzEwZmIxODkxMjczZjAwNGEwODRmMTA4OGRlMWVkZDQiLAogICAgICAibWV0YWRhdGEiIDogewogICAgICAgICJtb2RlbCIgOiAic2xpbSIKICAgICAgfQogICAgfQogIH0KfQ=='
property_signature='sYCm/hA/Dr7fX0N5lpFGfrAF2KM7pEs5XLnoVoSeby745lHmVLT7U6LYp4xd0dpPbM4WcYYiM5F5H92GMDz/HhXY9GJYTG03t3KKMPbzj2xoAw6VfyJ96BrBqgpldYb8s0nc6jQhj+d9pETmFXaVtb1Fu2tERJiOy5Ms7QvOMQmqg4eEU7LS+X3oFggQ3vMrCYt/di6jSPc2o0IK1nPLwxzDzY7sPZUq8RgbZIAiqxoU6vLoChUfzCg1g8DLloICEwKaZiLSriSiINcwnLlkG6yBeQ2lDcOh85KKJlU1CR33YqoLI/wp+zTDdB6qkiOghYmmhRlInW97agwpdMJUFFpd6ohPnO6cpnxu1R9u9b0eSUMtp32TVAtULlDRgsgk1rvY8qn+VpJafIkaLXKwIFUTnndy4jS0hbKDsYpVdLxNRmnzLRDWbTb8gaZ7RJLGnDIeWb3/S7pRTj1UxQVmHMweHlu5xvR3Z4AVmsF/5cxvJgDD0VRLAuPSCDOpyutHy+eIIBqLiyA8Xs2U79GXDnouJAP1flDFwgYXFQ0uTTem6wrsZlRxXZxX+LnlezV80jLlogTbpNxFLJV76gjQgl9i19RJ90rQPIvoLnOUGYY4iZioBjAiJrr5beI3AOPZ0DDNdU6JHwTE8bXDYgUkxodNMazuburfFdKqg3YwvYE='

(data fix skipped 73 bytes)

--- upstream configuration --- s_settings 0x0
locale en_us
view_distance 22
chat_mode 0
chat_colors True
displayed_skin_parts 127
main_hand 1
enable_text_filtering True
allow_server_listing True
particle 0

(data fix skipped 27 bytes)

--- downstream configuration --- c_feature_flags 0xc

--- downstream configuration --- c_select_known_packs 0xe
downstream configuration KeyError losing 3069 bytes
b'f>\x94K\x8a\xba#\x9e\xfbZ?\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\...
663e944b8aba239efb5a3f084fc8f44d090cf300a6a628e607d8b3e34b5cea...
possible zip at  746 b'\x07\x16minecraft:trim_pattern\x12\x0eminecraft:bolt\x00\x0fminecraf'
possible zip at  1304 b'\x07\x1aminecraft:painting_variant2\x0fminecraft:alban\x00\x0fmin'
possible zip at  1773 b'\x07\x15minecraft:damage_type1\x0fminecraft:arrow\x00\x1bminecraf'
possible zip at  2194 b'\x07\x18minecraft:banner_pattern+\x0eminecraft:base\x00\x10minecr'
possible zip at  2527 b'\x07\x15minecraft:enchantment*\x17minecraft:aqua_affinity\x00\x1c'
possible zip at  2912 b'\x07\x16minecraft:jukebox_song\x13\x0cminecraft:11\x00\x0cminecraft:'

--- downstream configuration --- c_registry_data 0x7

--- downstream configuration --- c_tags 0xd
```

`KeyError` means the packet id is not recognized.

I don't know why that is the case, due to me fixing the data on previous packets this shouldn't happen but for some reason it does.

I had to manually sieve through all the broken packets to guess where the next packet should start.

```
loading... kernel32
varint(0) 102 0x66 b'>\x94K\x8a\xba#\x9e\xfbZ?\x08O\xc8\xf4M'    ?
varint(1) 62 0x3e b'\x94K\x8a\xba#\x9e\xfbZ?\x08O\xc8\xf4M\t'    ?
varint(2) 9620 0x2594 b'\x8a\xba#\x9e\xfbZ?\x08O\xc8\xf4M\t\x0c\xf3'     ?
varint(3) 75 0x4b b'\x8a\xba#\x9e\xfbZ?\x08O\xc8\xf4M\t\x0c\xf3'         ?
varint(4) 580874 0x8dd0a b'\x9e\xfbZ?\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6'       ?
varint(5) 4538 0x11ba b'\x9e\xfbZ?\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6'          ?
varint(6) 35 0x23 b'\x9e\xfbZ?\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6'      ?
varint(7) 1490334 0x16bd9e b'?\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6(\xe6\x07'     ?
varint(8) 11643 0x2d7b b'?\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6(\xe6\x07'         ?
varint(9) 90 0x5a b'?\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6(\xe6\x07'      ?
varint(10) 63 0x3f b'\x08O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6(\xe6\x07\xd8'          ?
varint(11) 8 0x8 b'O\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6(\xe6\x07\xd8\xb3'    ?
varint(12) 79 0x4f b'\xc8\xf4M\t\x0c\xf3\x00\xa6\xa6(\xe6\x07\xd8\xb3\xe3'       ?
varint(13) 1276488 0x137a48 b'\t\x0c\xf3\x00\xa6\xa6(\xe6\x07\xd8\xb3\xe3K\\\xea'        ?
varint(14) 9972 0x26f4 b'\t\x0c\xf3\x00\xa6\xa6(\xe6\x07\xd8\xb3\xe3K\\\xea'     ?
varint(15) 77 0x4d b'\t\x0c\xf3\x00\xa6\xa6(\xe6\x07\xd8\xb3\xe3K\\\xea'         ?
varint(16) 9 0x9 b'\x0c\xf3\x00\xa6\xa6(\xe6\x07\xd8\xb3\xe3K\\\xeaX'    ?
varint(17) 12 0xc b'\xf3\x00\xa6\xa6(\xe6\x07\xd8\xb3\xe3K\\\xeaX\x9d'   c_feature_flags
         array

varint(18) 115 0x73 b'\xa6\xa6(\xe6\x07\xd8\xb3\xe3K\\\xeaX\x9dx\xda'    ?
varint(19) 0 0x0 b'\xa6\xa6(\xe6\x07\xd8\xb3\xe3K\\\xeaX\x9dx\xda'       ?
varint(20) 660262 0xa1326 b'\xe6\x07\xd8\xb3\xe3K\\\xeaX\x9dx\xda\xaf\x92\x9c'   ?
varint(21) 5158 0x1426 b'\xe6\x07\xd8\xb3\xe3K\\\xeaX\x9dx\xda\xaf\x92\x9c'      ?
varint(22) 40 0x28 b'\xe6\x07\xd8\xb3\xe3K\\\xeaX\x9dx\xda\xaf\x92\x9c'          ?
varint(23) 998 0x3e6 b'\xd8\xb3\xe3K\\\xeaX\x9dx\xda\xaf\x92\x9c\xb8\x9a'        ?
varint(24) 7 0x7 b'\xd8\xb3\xe3K\\\xeaX\x9dx\xda\xaf\x92\x9c\xb8\x9a'    c_registry_data
         string array

varint(25) 158915032 0x978d9d8 b'\\\xeaX\x9dx\xda\xaf\x92\x9c\xb8\x9a\xaf\xc7\x8d\x93'   ?
varint(26) 1241523 0x12f1b3 b'\\\xeaX\x9dx\xda\xaf\x92\x9c\xb8\x9a\xaf\xc7\x8d\x93'      ?
varint(27) 9699 0x25e3 b'\\\xeaX\x9dx\xda\xaf\x92\x9c\xb8\x9a\xaf\xc7\x8d\x93'   ?
varint(28) 75 0x4b b'\\\xeaX\x9dx\xda\xaf\x92\x9c\xb8\x9a\xaf\xc7\x8d\x93'       ?
varint(29) 92 0x5c b'\xeaX\x9dx\xda\xaf\x92\x9c\xb8\x9a\xaf\xc7\x8d\x93j'        ?
```

## find all

The crypto part is definitely working due to correct exchange of bytes right after the encryption packets.

So instead I dumped all info to `client.all.txt` and `server.all.txt` and simply looked for all matching headers.

For uncompressed data I couldn't find anything useful.
### zlib header

```python
p = re.compile(rb'\x78[\x01\x5e\x9c\xda]')
```

For each point I will run the following checks to ensure it is indeed a valid packet.
Then I will handle it.

```python
s1, s2 = varint_lookback(by, st-1, 2)
buff = Buffer(by[s2:])
# print('varints', by[s2:s1].hex(), by[s1:st].hex())
packet_len = buff.unpack_varint()
buff = Buffer(buff.read(packet_len))
data_len = buff.unpack_varint()
data = buff.read()
data = zlib.decompress(data)
if 'uoft' in data:
	print(data)
	exit()
assert len(data) == data_len
unzipped_out.append(data)
buff = Buffer(data)
pid = buff.unpack_varint()
name = get_packet_name(pid, 'play', direction)
pids.add(pid)
fn = globals().get('packet_'+name, None)
if fn:
	fn(buff)
else:
	print()
	print(hex(pid), name)
	print(f"{packet_len=} {data_len=}")
	print(buff.buff[:50])
```

More chunks are salvaged, however a lot of data is still missing.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736724771/2025/01/a8de8866ae7c1fb160b2c45387796737.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736724962/2025/01/40f92cde7e061be655d611383c7d6364.png)

The individual block update events are combined with data I got before and mod 100.
(some positions are y=-2539520, which is obviously wrong but I included them just in case)

## conclusion

I couldn't solve it.

I tried a lot of fixes but none of them work, the packets just keep getting misaligned.

Maybe it is coz the library I share much of the serialisation logic with has discontinued and is stuck in version 1.19, or maybe more steps are needed in the capture phase, like some tcp packets should be discarded or something.

Welp.

## after

After the event ended, I realised that I was really close.

I just had to use two ciphers, 1 for the client 1 for server, that's it.

No error correction needed, packets all behave nicely.

Everything works flawlessly.

![200w.gif](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736727242/2025/01/cdd29fbe6702a911d79e99121cf3bb62.gif)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736729941/2025/01/1d8b4684a1a6c9f6e39d8ded4550782d.png)

Not a single error was thrown.

I spent 24 hours in vane trying to manually fix the data.