---
created: 2024-11-23T22:27
updated: 2024-11-23T22:29
solves: 61
points: 100
---

## part 1

We can solve this by just uploading the testing image from the pdf file.

![Screenshot 2024-11-23 124147.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1732418878/2024/11/041140ecbf92609f5ad115a8195bbf51.png)

## part 2

```python
from barcode import Code39
number = '65916-999999L'
code = Code39(number)
code.save("code")
```

I managed to generate bar codes that the website accepts, however the website simply says "Accepted". Got stuck there.
