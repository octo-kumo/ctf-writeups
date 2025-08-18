---
ai_date: 2025-08-17 20:46:51
ai_summary: Exploited app signature generation vulnerability to forge requests for
  flag
ai_tags:
- ssrf
- api-verification
- hmac-forge
created: 2025-08-16T12:23
points: 100
solves: 218
title: Sekai Bank - Signature
updated: 2025-08-17T20:46
---

Let's just first run the app in an emulator and track all the requests to the API.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755361482/20250816122441849.png/5ff826eae7e2cf0f5068fa38b6b50a52.png)

We realise that the signature is unique to each request path, to find it we need the signature data.

```java
private String generateSignature(Request request) throws IOException, GeneralSecurityException {
    Signature[] signatureArr;
    String str = request.method() + "/api".concat(getEndpointPath(request)) + getRequestBodyAsString(request);
    SekaiApplication instance = SekaiApplication.getInstance();
    PackageManager packageManager = instance.getPackageManager();
    String packageName = instance.getPackageName();
    try {
        if (Build.VERSION.SDK_INT >= 28) {
            PackageInfo packageInfo = packageManager.getPackageInfo(packageName, 134217728);
            SigningInfo signingInfo = packageInfo.signingInfo;
            if (signingInfo == null) {
                signatureArr = packageInfo.signatures;
            } else if (signingInfo.hasMultipleSigners()) {
                signatureArr = signingInfo.getApkContentsSigners();
            } else {
                signatureArr = signingInfo.getSigningCertificateHistory();
            }
        } else {
            signatureArr = packageManager.getPackageInfo(packageName, 64).signatures;
        }
        if (signatureArr == null || signatureArr.length <= 0) {
            throw new GeneralSecurityException("No app signature found");
        }
        MessageDigest instance2 = MessageDigest.getInstance("SHA-256");
        for (Signature byteArray : signatureArr) {
            instance2.update(byteArray.toByteArray());
        }
        return calculateHMAC(str, instance2.digest());
    } catch (PackageManager.NameNotFoundException | NoSuchAlgorithmException e) {
        throw new GeneralSecurityException("Unable to extract app signature", e);
    }
}
```

... which we can do easily with `apkverify`

```sh
root@android-pod:/opt/android-sdk/build-tools/35.0.1# ./apksigner verify --print-certs  /workspace/SekaiBank.apk
Signer #1 certificate DN: C=ID, ST=Bali, L=Indonesia, O=HYPERHUG, OU=Development, CN=Aimar S. Adhitya
Signer #1 certificate SHA-256 digest: 3f3cf8830acc96530d5564317fe480ab581dfc55ec8fe55e67dddbe1fdb605be
Signer #1 certificate SHA-1 digest: 2c9760ee9615adabdee0e228aed91e3d4ebdebdf
Signer #1 certificate MD5 digest: fcab4af1f7411b4ba70ec2fa915dee8e
```

Now we can just replicate the logic used in the code and we can forge signatures.

```python

import hashlib
import hmac
import requests

CERT_DIGEST_HEX = "3f3cf8830acc96530d5564317fe480ab581dfc55ec8fe55e67dddbe1fdb605be"
CERT_DIGEST = bytes.fromhex(CERT_DIGEST_HEX)

def generate_signature(method: str, endpoint_path: str, body: str) -> str:
    s = method.upper() + "/api" + endpoint_path + body
    sig = hmac.new(CERT_DIGEST, s.encode("utf-8"), hashlib.sha256).hexdigest()
    return sig
print("Signature:", generate_signature("GET", "/user/profile", '{}'))

r=requests.post("https://sekaibank-api.chals.sekai.team/api/flag",headers={
    'X-Signature': generate_signature("POST", "/flag", '{"unmask_flag":true}'),
},json={"unmask_flag":True})
print(r.text)

# SEKAI{are-you-ready-for-the-real-challenge?}
```

```flag
SEKAI{are-you-ready-for-the-real-challenge?}
```