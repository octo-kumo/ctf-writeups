---
ai_date: '2025-04-27 05:14:28'
ai_summary: Used testing image from PDF, attempted to generate valid barcode for input
  validation bypass
ai_tags:
- img-search
- barcodes
- input-validation
created: 2024-11-23T22:27
points: 100
solves: 61
updated: 2024-11-23T22:29
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