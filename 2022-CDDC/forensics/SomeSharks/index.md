---
ai_date: 2025-04-27 05:10:59
ai_summary: GET request with If-None-Match header exploited a 304 NOT MODIFIED response, potentially revealing a cached PDF file.
ai_tags:
  - http-hdr
  - cache-p
  - pdf
created: 2024-06-11T01:17
updated: 2025-07-14T09:46
---

```
GET /systemic-therapy-site/Documents/Policy%20and%20Forms/Benefit%20Drug%20List.pdf HTTP/1.1
Host: www.bccancer.bc.ca
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: ko-KR,ko;q=0.9
Cookie: TS012d7c21=01f089f5435cc9aed4ef77938cdbee76b972dda66a550675349b5f23f83162dba6983965badebbe1de0f662293383f3dde97ca470b
If-None-Match: "{990DF044-92D3-4A02-B133-D8FA07A0E335},135pub"
If-Modified-Since: Mon, 02 May 2022 23:52:27 GMT
```

```
HTTP/1.1 304 NOT MODIFIED
Cache-Control: private,max-age=0
Content-Length: 0
Expires: Sun, 24 Apr 2022 07:20:29 GMT
X-SharePointHealthScore: 0
Public-Extension: http://schemas.microsoft.com/repl-2
SPRequestGuid: db9d3ba0-4ae2-8080-e9e2-b8b2f74ae61e
request-id: db9d3ba0-4ae2-8080-e9e2-b8b2f74ae61e
X-FRAME-OPTIONS: SAMEORIGIN
SPRequestDuration: 8
SPIisLatency: 0
MicrosoftSharePointTeamServices: 15.0.0.4797
X-Content-Type-Options: nosniff
X-MS-InvokeApp: 1; RequireReadOnly
Date: Mon, 09 May 2022 07:20:29 GMT
Set-Cookie: TS012d7c21=01f089f54303d18ab40c8103e7161535bd165ddce0991b5d4efd78c4b5178ac6b9c0b9c1918d570a55987ded36fe8a08d7fb83029f; Path=/; Domain=.www.bccancer.bc.ca
```