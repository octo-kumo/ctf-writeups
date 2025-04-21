---
title: Bullet Dodge
solves: 70
points: 310
created: 2025-04-19T00:38
updated: 2025-04-20T20:26
tags:
  - unity
---

Game hacking.

First install melon loader, unity explorer and dnSpy.
## analysis

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745038070/2025/04/ebc6213d9cdc53d0812fecc9d354a39d.png)

We can see `f` inside `Canvas > Score`, viewing it in dnSpy confirms that its the logic behind the scoring.

```cs
using System;
using System.Text;
using TMPro;
using UnityEngine;
public class f: MonoBehaviour {
  private void Start() {
    this.b.text = this.j(this.l("U0NPUkU6IA==")) + this.k(this.e);
    this.d = true;
  }
  public string j(byte[] a) {
    return Encoding.UTF8.GetString(a);
  }
  private void Update() {
    string text = this.j(this.l("QlU=")) + this.w(this.j(this.l("Snp3PQ=="))) + this.w("LiQ=");
    if (this.d) {
      this.e += this.o;
      this.b.text = this.j(this.l("U0NPUkU6IA==")) + this.k(this.e);
    }
    foreach(char c in Input.inputString) {
      this.h += c.ToString();
      if (this.h.Length > text.Length) {
        this.h = this.h.Substring(1);
      } else if (this.h.Length == text.Length) {
        this.k(4);
      } else {
        this.j(this.l("b21vbW9tbW9t"));
      }
      if (this.h.ToUpper() != text) {
        this.k(13);
      } else {
        this.g();
      }
    }
    if (this.e >= this.s()) {
      this.a.b();
      this.o = 0;
      this.p = false;
    }
  }
  public byte[] l(string a) {
    return Convert.FromBase64String(a);
  }
  public string w(string e) {
    byte[] array = this.l(e);
    for (int i = 0; i < array.Length; i++) {
      array[i] = this.r((byte) f.q[i], array[i]);
    }
    return this.j(array);
  }
  public string k(int a) {
    return a.ToString();
  }
  private void g() {
    this.e += this.s();
  }
  public byte r(byte a, byte b) {
    return a ^ b;
  }
  public int s() {
    return 100000;
  }
  public TextMeshProUGUI b;
  public a a;
  public bool d;
  public int e;
  public bool p = true;
  public static string q = "kp";
  private int o = 1;
  private string h = "";
}
```

## patching

Deciphering what it is doing is too hard, we can just skip it by patching the binary.

```cs
private void Update()
	{
		this.j(this.l("QlU=")) + this.w(this.j(this.l("Snp3PQ=="))) + this.w("LiQ=");
		if (this.d)
		{
			this.e += this.o;
			this.b.text = this.j(this.l("U0NPUkU6IA==")) + this.k(this.e);
		}
		this.g();
		if (this.e >= this.s())
		{
			this.a.b();
			this.o = 0;
			this.p = false;
		}
	}
```

Now compile and save all in dnSpy, restart the game.

## flag

On game start the flag is shown to you.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745037522/2025/04/efe95d8e5d42bf449cf49795bc38770b.png)

The font was hard to read, so I grabbed it from the source.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745037772/2025/04/4c48caf26b2b0b684d78cab708b4cd92.png)

```flag
UMASS{R4N_FR0M_B1LL_o7}
```
