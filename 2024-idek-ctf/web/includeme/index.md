---
ai_date: '2025-04-27 05:18:38'
ai_summary: Brute-forced file upload exploiting a test script, revealing race condition
  in /home/ctf/.julia/packages/Genie/yQwwj/test/tests_AppServer.jl
ai_tags:
- brute
- file-upload
- race-condition
created: 2024-08-17T23:23
updated: 2024-08-18T21:34
---

By brute forcing every `.jl` file on the system, I managed to launch a file upload window that worked!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1723952876/2024/08/a5902c24e5b95776a76cd20440add995.png)

After checking the logs I found out that it was `/home/ctf/.julia/packages/Genie/yQwwj/test/tests_AppServer.jl`

But I got stuck here.

After the event ended I found out that the solution is a race condition, to include `app.jl` again.

Damn, I feel dumb.