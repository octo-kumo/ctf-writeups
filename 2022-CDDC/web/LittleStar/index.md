---
ai_date: 2025-04-27 05:11:40
ai_summary: Cookie controls image source; exploit via input tampering
ai_tags:
  - xss
  - csrf
created: 2024-06-11T01:17
updated: 2025-07-14T09:46
---

Accessing the source we find this at the beginning

```html

<div class="content">
    <!-- twinkle_star -> little_star -> flag -->
    <div>
        <div class="button" onclick="star()">⭐</div>
    </div>
</div>
```

```javascript
function star() {
    $.get("/star", function (data, status) {
        html_code = '<div class="button">⭐</div>\n' + data['content'];
        $(".content")[0].innerHTML = html_code;
    })
}
```

Accessing `/star`

```json
{
  "content": "<div class=\"cookie\">twinkle_star</div> <br /><img class=\"image\" src=\"/static/img/starfall.gif\">"
}
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083589/2024/06/5bfa41cc5716bfd6be99ef65cfb5c1f8.gif)

It seems that the cookie sent determines the result of the image sent back

And we have the convenient hint `<!-- twinkle_star -> little_star -> flag -->`

## `little_star`

```json
{
  "content": "<div class=\"cookie\">little_star</div> <br /><img class=\"image\" src=\"/static/img/yellostar.gif\">"
}
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083590/2024/06/a053b83fe2a44c805308c22d7faa0d57.gif)

## `flag`

```json
{
  "content": "<div class=\"cookie\">FLAG!</div> <br /><div class=\"text-rainbow\">CDDC22{B4by_W3b_H4cking_3asy++}</div>"
}
```

And we got the flag

```flag
CDDC22{B4by_W3b_H4cking_3asy++}
```