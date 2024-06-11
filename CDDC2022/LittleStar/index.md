# Little Star

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

![](starfall.gif)

It seems that the cookie sent determines the result of the image sent back

And we have the convenient hint `<!-- twinkle_star -> little_star -> flag -->`

## `little_star`
```json
{
  "content": "<div class=\"cookie\">little_star</div> <br /><img class=\"image\" src=\"/static/img/yellostar.gif\">"
}
```

![](yellostar.gif)

## `flag`
```json
{
  "content": "<div class=\"cookie\">FLAG!</div> <br /><div class=\"text-rainbow\">CDDC22{B4by_W3b_H4cking_3asy++}</div>"
}
```
And we got the flag
```text
CDDC22{B4by_W3b_H4cking_3asy++}
```