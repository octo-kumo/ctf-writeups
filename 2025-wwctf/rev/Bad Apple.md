---
description: enjoy the apple ~
updated: 2025-07-28T06:34
created: 2025-07-28T01:39
solves: 0
points: 500
---

> Bad Apple
>
> 500
>
> `author: yun`
>
> My video player dies after half way through the video, I couldn't figure out why, could you fix it for me?
>
> - Run this in your terminal with `java -jar`
> - Try unplug and plug your earphones
> 
> _your decompiled code should be several MB! otherwise your decompiler failed!_
>
> hint
> The flag is in the second half of the video, so it is not in plaintext.

## post mortem

This challenge is meant to kill any hopes of using AI or automated tools for reverse engineering, but the actual underlying repetitive logic would be very simple to understand.

The main idea is to think "where would the video data be stored?" because videos take up a lot of space, and cannot be easily hidden.

Similarly, "VM logic will take up a lot of space", "why are there unused information that seems to have the same structure" and "why does the unused information take up half of the entire file".

Coincidentally, many decompilers would break when trying to decompile this challenge, mostly due to the huge stack in `main` (I had to increase stack size for the compiler to compile it too), or maybe the huge if-chains in the decoders.

Lastly, maybe the obfuscation was too much lol (for the initial challenge).

---

I am going to use https://github.com/Storyyeller/Krakatau and do it on the easy version.

```sh
./krak2 dis apple_easy.jar --out krakout.zip
```

*Doing this in JD GUI is easier, but with krak2 I can patch it.*
*Just note that you need to give JD GUI more RAM.*

## introduction

Obviously the data is stored somewhere, after all it is playing a full 3 min video.

*You might want to use tcpdump or smth to verify that it does not connect to network*

It cannot be stored as logic because that would be much larger. (and also there are string coded bytes inside the file).

So let's look for string / bytes inside the source code.

## the two (three?) byte streams

It is pretty clear that majority of the class file's size is dominated by the three groups of strings in the constants pool, two big one small.

And furthermore `main` is dominated by `StringBuilder.append` appending all of them together.

```java [line 20338]
... // snippet of the appending process
L28438: ldc_w [_4489]
L28441: invokespecial Method java/lang/String <init> (Ljava/lang/String;)V
L28444: invokevirtual Method java/lang/StringBuilder append (Ljava/lang/String;)Ljava/lang/StringBuilder;
L28447: new java/lang/String
L28450: dup
...
```

So let's clean up the code by removing that (regex too pro).

```java
L\d+:\s*invokespecial Method java/lang/String <init> \(Ljava/lang/String;\)V
L\d+:\s*invokevirtual Method java/lang/StringBuilder append \(Ljava/lang/String;\)Ljava/lang/StringBuilder;
L\d+:\s*new java/lang/String
L\d+:\s*dup
```

Then only keep the starting and ending `ldc` references. (replace the following with `$1\n$2`)

```java
(L\d+:\s*ldc(_w)? \[_\d+\]\s*)\n?
(L\d+:\s*ldc(_w)? \[_\d+\]\s*)\n?
(L\d+:\s*ldc(_w)? \[_\d+\]\s*)\n?
```

*I'm too lazy to use look-aheads/behinds*

We can roughly estimate the size of those strings by the number of lines they take up (this is not accurate but works), because it appears that all of the string constant declarations have the same byte size of `2048`.

*Note that constant ids for those strings are spaced by 2 mostly probably because of obfuscation.*

```java [line 26864]
.const [_3680] = Utf8 "'\u00F5UUUUUU\u0...\u00BE" // this is 2048 bytes
```

**Batch 1**

`1851 / 2 * 2048 = 1.9MB`

```java [line 9370]
L11: ldc [_79]
...
L11912: ldc_w [_1930]

"\u0082\u0091\u00BDUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU\u007F\u00FF\u00FF\u00FF\u00FF\u00FF\u00FF\u00FF\u00FF\u00FF\u00FF..."
```

**Batch 2**

`2924 / 2 * 2048 = 3MB`

```java [line 13998]
L11954: ldc_w [_1953]
...
L30960: ldc_w [_4877]

"\u00C6\u00D9EUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU..."
```

**Batch 3** (small)

`16 / 2 * 2048 = 16KB`

```java [line 21327]
L31005: ldc_w [_4879]
...
L31109: ldc_w [_4895]

"MThd\u0000\u0000\u0000\u0006\u0000\u0001\u0000\u0012\u0001\u00E0MTrk\u0000\u0000\u0000\u009D\u0000\u00FF\u0003,Touhou - Bad Appleii feat.nomico | Nonstop2k\u0000\u00FF\u0001,Touhou - Bad Appleii feat.nomico | Nonstop2k\u0000\u00FF\u0002\u001CCopyright 2024 Nonstop2k.com\u0000\u00FFQ..."

"MThd..........MTrk.......,Touhou - Bad Appleii feat.nomico | Nonstop2k...,Touhou - Bad Appleii feat.nomico | Nonstop2k....Copyright 2024 Nonstop2k.com..Q..."
```

Actually by examining batch 3 you will see copyright information, and you might realise that it is a MIDI file, so we can just ignore it. Because the flag is in the second half of the video, not the audio.

### main function

After cleaning up it also becomes a lot clearer what the code is doing.

```java [line 9363]
.method public static main : ([Ljava/lang/String;)V
    .code stack 6 locals 17
L0:     new java/lang/StringBuilder
L3:     dup
L4:     invokespecial Method java/lang/StringBuilder <init> ()V
L7:     new java/lang/String
L10:    dup
L11:    ldc [_79]
// ... batch 1
L11912: ldc_w [_1930]
// convert batch 1 into bytes
L11915: invokespecial Method java/lang/String <init> (Ljava/lang/String;)V
L11918: invokevirtual Method java/lang/StringBuilder append (Ljava/lang/String;)Ljava/lang/StringBuilder;
L11921: invokevirtual Method java/lang/StringBuilder toString ()Ljava/lang/String;
L11924: getstatic Field java/nio/charset/StandardCharsets ISO_8859_1 Ljava/nio/charset/Charset;
L11927: invokevirtual Method java/lang/String getBytes (Ljava/nio/charset/Charset;)[B
// asign it to `a` the byte array.
L11930: putstatic Field BadApplePlayer a [B

// some kind of anti debugger function
L11933: invokestatic Method BadApplePlayer a ()Z
L11936: ifeq L11943
L11939: iconst_1
L11940: invokestatic Method java/lang/System exit (I)V
// exits if triggers, so we can just ignore it

        .stack same_extended
L11943: new java/lang/StringBuilder
L11946: dup
L11947: invokespecial Method java/lang/StringBuilder <init> ()V
L11950: new java/lang/String
L11953: dup
L11954: ldc_w [_1953]
// batch 2
L30960: ldc_w [_4877]
// convert batch 2 into bytes
L30973: getstatic Field BadApplePlayer I [Ljava/lang/String;
L30976: iconst_2
L30977: aaload
L30978: invokespecial Method java/lang/String <init> (Ljava/lang/String;)V
L30981: invokevirtual Method java/lang/StringBuilder append (Ljava/lang/String;)Ljava/lang/StringBuilder;
L30984: invokevirtual Method java/lang/StringBuilder toString ()Ljava/lang/String;
L30987: getstatic Field java/nio/charset/StandardCharsets ISO_8859_1 Ljava/nio/charset/Charset;
L30990: invokevirtual Method java/lang/String getBytes (Ljava/nio/charset/Charset;)[B
L30993: pop
L30994: new java/lang/StringBuilder
L30997: dup
L30998: invokespecial Method java/lang/StringBuilder <init> ()V
L31001: new java/lang/String
L31004: dup
L31005: ldc_w [_4879]
// batch 3
L31109: ldc_w [_4895]

L31122: getstatic Field BadApplePlayer I [Ljava/lang/String;
L31125: iconst_3
L31126: aaload
L31127: invokespecial Method java/lang/String <init> (Ljava/lang/String;)V
L31130: invokevirtual Method java/lang/StringBuilder append (Ljava/lang/String;)Ljava/lang/StringBuilder;
L31133: invokevirtual Method java/lang/StringBuilder toString ()Ljava/lang/String;
L31136: getstatic Field java/nio/charset/StandardCharsets ISO_8859_1 Ljava/nio/charset/Charset;
L31139: invokevirtual Method java/lang/String getBytes (Ljava/nio/charset/Charset;)[B
L31142: astore_0
L31143: sipush 240
L31146: sipush 320
L31149: multianewarray [[Z 2
L31153: astore_1
L31154: invokestatic Method javax/sound/midi/MidiSystem getSequencer ()Ljavax/sound/midi/Sequencer;
L31157: dup
L31158: astore_2
L31159: invokeinterface InterfaceMethod javax/sound/midi/Sequencer "open" ()V 1
L31164: new java/io/ByteArrayInputStream
L31167: dup
L31168: aload_0
L31169: invokespecial Method java/io/ByteArrayInputStream <init> ([B)V
L31172: invokestatic Method javax/sound/midi/MidiSystem getSequence (Ljava/io/InputStream;)Ljavax/sound/midi/Sequence;
L31175: astore_0
L31176: aload_2
L31177: aload_0
L31178: invokeinterface InterfaceMethod javax/sound/midi/Sequencer setSequence (Ljavax/sound/midi/Sequence;)V 2
L31183: getstatic Field java/lang/System out Ljava/io/PrintStream;
L31186: getstatic Field BadApplePlayer I [Ljava/lang/String;
L31189: iconst_4
L31190: aaload
...
```

### afterwards
**Batch 1** is converted to bytes and used to set the field `a`, the byte array that is most likely storing the video information.

```java
.field private static a [B
...
L11:    ldc [_79]
L11912: ldc_w [_1930]
...
L11930: putstatic Field BadApplePlayer a [B
```

**Batch 2** is... ignored.
That is very sus.

You can compare it with batch 1.

```java
// batch 1
L11927: invokevirtual Method java/lang/String getBytes (Ljava/nio/charset/Charset;)[B
L11930: putstatic Field BadApplePlayer a [B

// batch 2
L30990: invokevirtual Method java/lang/String getBytes (Ljava/nio/charset/Charset;)[B
L30993: pop // just ignored lol
```

**Batch 3** is the MIDI.

## the decoding functions

Well, you might try to decode the byte streams, but they do not fit any known file types.

They also can't be storing frames directly, 6500 frames × 320 × 240 = 499200000 bits = 62,400,000 bytes = 62 megabytes.

It must be a custom compressed video codec.

*or maybe they contain the source code to a VM, who knows VMs are popular now-a-days, but you can't find any function that seems to be executing arbitrary code.*

So they are video data, where is the decoding function? It can't be in main, it doesn't have enough unknown logic left.

There appears to be two very suspicious functions `a ()I` and `b ()I` (because they are the biggest functions apart from `main`).

Although they do not access `a [B` (the byte array) directly, they both call `a (I)V` at the start, that function does access `a [B`.

```java [line 119]
.method private static a : ()I
    .code stack 4 locals 0
L0:     bipush 20
L2:     invokestatic Method BadApplePlayer a (I)V
L5:     getstatic Field BadApplePlayer a J
L8:     getstatic Field BadApplePlayer a I
L11:    iconst_1
... // wow so big! 7000+ lines!
L7526:  putstatic Field BadApplePlayer a I
L7529:  bipush 48
L7531:  ireturn
L7532:  
    .end code
.end method
```

```java [line 4741]
.method private static b : ()I
    .code stack 4 locals 0
L0:     bipush 19
L2:     invokestatic Method BadApplePlayer a (I)V
L5:     getstatic Field BadApplePlayer a J
L8:     getstatic Field BadApplePlayer a I
... // wow so big! 7000+ lines!
L7528:  putstatic Field BadApplePlayer a I
L7531:  bipush 15
L7533:  ireturn
L7534:  
    .end code
.end method
```

Only `b ()I` is used however.

```java [back to main method]
...
.stack full
            locals Object java/lang/Object Object [[Z Object javax/sound/midi/Sequencer Top Top Top Long
            stack
        .end stack
L31616: getstatic Field BadApplePlayer b I
L31619: getstatic Field BadApplePlayer a [B
L31622: arraylength
L31623: if_icmpge L32220
L31626: invokestatic Method BadApplePlayer b ()I
L31629: sipush 255
L31632: iand
L31633: invokestatic Method BadApplePlayer b ()I
L31636: sipush 255
L31639: iand
...
```

## patching

Now to patch the .class file.

Take a guess, maybe `a` is the decoding function for bytes in batch 2, who knows, maybe it is true.

*Maybe this challenge was guessy after all.*

In that case we replace the `pop` after the definition of batch 2 with `putstatic Field BadApplePlayer a [B` after batch 1, effectively now batch 1 is being ignored while batch 2 is used to initialise the global `a [B`.

Then we replace every invocation of `b ()I` with `a ()I`.

```
.*invokestatic Method BadApplePlayer b \(\)I
.*invokestatic Method BadApplePlayer a \(\)I
```

### diff

```dif
$ diff *.old.j *.patched.j
13986,13990c13986
< L11930: putstatic Field BadApplePlayer a [B
< L11933: invokestatic Method BadApplePlayer a ()Z
< L11936: ifeq L11943
< L11939: iconst_1
< L11940: invokestatic Method java/lang/System exit (I)V
---
> L11930: pop
21321c21317
< L30993: pop
---
> L30993: putstatic Field BadApplePlayer a [B
21641c21637
< L31626: invokestatic Method BadApplePlayer b ()I
---
> L31626: invokestatic Method BadApplePlayer a ()I
21644c21640
< L31633: invokestatic Method BadApplePlayer b ()I
---
> L31633: invokestatic Method BadApplePlayer a ()I
21666c21662
< L31667: invokestatic Method BadApplePlayer b ()I
---
> L31667: invokestatic Method BadApplePlayer a ()I
21726c21722
< L31754: invokestatic Method BadApplePlayer b ()I
---
> L31754: invokestatic Method BadApplePlayer a ()I
```

```bash
$ ./krak2 asm --out patched/BadApplePlayer.class BadApplePlayer.patched.j
$ cd patched
$ java BadApplePlayer
```

Try it...

And done, we have the second half of the video.

## flag
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1753678028/20250728004707948.png/fc61871a74e0d650969eb0e9aabeca66.png)

Continue playing and we have the flag.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1753678115/20250728004834841.png/1669fd01dbe5a80d804f0bd8e80452c1.png)

```flag
wwf{nagareteku_toki_no_naka_de_demo_FLAG_ga_hora_guruguru_mawatte}
```

Sadly the audio is broken now (because the midi will start from the beginning).

It is a exercise left to the viewer to figure out: how to patch the audio player to start from midway point.

> wait but what about the harder one?

It is just packing this whole thing, and decoding it on the fly. 😃
