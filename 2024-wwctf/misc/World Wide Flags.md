---
created: 2024-11-30T16:19
updated: 2024-12-08T01:44
solves: 119
points: 150
---

Intended solution is ~~write an AI~~ manually identifying all the flags.

I think everyone enjoyed it~

![anime-girl-happy.gif](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733003711/2024/11/a496abdc109a17b9e6891a9510d9f51e.gif)

![flag.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733004816/2024/11/e212104933871bd5f223a0cc0f7f8a43.png)

```flag
wwf{d1d_y0u_u53_41_70_r34d_fl465}
```

## to read a noisy flag

> Difficulty goes up in stages, blur, affine, low noise, more noise, random alignment.
>
> Since low difficulty flags can be solved with subset of high difficulty flag's steps, I will only discuss lv99 flags, i.e. with the highest level of random transformation, random alignment and high mean noise.

*Note that the solution below is not optimized at all, very slow at 1.3s / image*.
### 1. denoise

Since I'm the author, I know that the noise is produced with $\bar{x}=128$, $\sigma=30$, and is blended with the flag using `add` mode. Even if you don't know that you can probably guess it.

We can remove the noise (sort of) using OpenCV.

```python
img = cv2.fastNlMeansDenoisingColored(img, None, 10, 10, 7, 21)
img = (np.clip(img, 128, 255)-128)*2
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733097250/2024/12/5c9a2ce2060d3fca12bc018928efe80c.png)

Good progress so far!

### 2. find the rectangle

To find the rectangle we can use OpenCV's contour functions.

```python
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
_,thresh = cv2.threshold(gray, cv2.THRESH_BINARY)
contours, _ = cv2.findContours(thresh,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)
largest_contour = max(contours, key=cv2.contourArea)
```

After which we can find the corners like this.

```python
def get_corners(contour):
  approx = cv2.approxPolyDP(contour, 0.04 * cv2.arcLength(contour, True), True)
  if len(approx) != 4: return None
  corners = np.array(approx.reshape(4, 2), dtype="float32")
  rect = np.zeros((4, 2), dtype = "float32")
  s = np.sum(corners, axis=1)
  diff = np.diff(corners, axis=1)
  return corners[[np.argmin(s), np.argmin(diff), np.argmax(s), np.argmax(diff)]]

rect = get_corners(largest_contour)
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733097261/2024/12/50b7451e1dfb963f3a5e50bbbc3ef2cc.png)

### 3. reverse transform

We can then use OpenCV to transform our image back.

```python
def untransform(img, rect, width=250,height=175):
  dst = np.array([
    [0, 0],
    [width - 1, 0],
    [width - 1, height - 1],
    [0, height - 1]
  ], dtype="float32")
  return cv2.warpPerspective(img, cv2.getPerspectiveTransform(rect, dst), (width, height))
  
def untransformAffine(img, rect, width=250,height=175):
  dst = np.float32([
    [0, 0],
    [width - 1, 0],
    [width - 1, height - 1]
  ])
  return cv2.warpAffine(img, cv2.getAffineTransform(rect[:3], dst), (width, height))
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733097777/2024/12/dfa4c1f2696c99a33730f8db02df7c15.png)

`Perspective` vs `Affine` aren't really that different (the challenge itself uses affine transformation, but perspective seems to work a little better?).

### 4. flag matching

To match flags we simply use OpenCV's `matchTemplate` and run it against every known flag.

Highest score wins.

```python
def image_similarity(img1, img2):
  return cv2.matchTemplate(img1, img2, cv2.TM_CCOEFF_NORMED)[0][0]
  
WIDTH = 250
HEIGHT = 175
flags_dir = '/content/flags'
flags = [f for f in os.listdir(flags_dir) if f.endswith('.png')]
flag_imgs = [cv2.resize(cv2.imread(os.path.join(flags_dir, f)), (WIDTH, HEIGHT)) for f in flags]

def solve(warped):
  scores = [image_similarity(warped, flag_img) for flag_img in flag_imgs]
  best_index = scores.index(max(scores))
  best_similarity = scores[best_index]
  best_match = flags[best_index]
  return best_match[:2], best_similarity
```

> for the US flag that we used above, here are the results.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733101494/2024/12/81d3e0199242044e72bccdf8b25f4bb2.png)

> `um` and `us` are identical flags. One is *United States Minor Outlying Islands* while the other is *United States*.

## accuracy

Took 2 hours to test against all flags, accuracy is calculated by $\frac{\text{correct}}{\text{total}}$.

The average speed is around 1.3s / image.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733096445/2024/12/d7360351bf4b0db40029cf21b9e541bc.png)

```
accuracy=0.775330396475771
normalized_accuracy=0.8525999192508943
```

> Normalized accuracy is calculated after dividing each column by the sum of that column, this is because some countries had 100 generated flags while others only had 20.

### confusion matrix

From the confusion matrix we can see that for most countries its 100% accuracy, while for some other countries its not very stable.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733101724/2024/12/0442c8e92bdef8938aeccdba313f31b0.png)

Do note that ~300/55k flags failed and threw errors, I don't know if it was due to the rectangle step not finding rectangles, or other steps. It doesn't matter since the failure rate is $<1\%$.

## accuracy against hosted challenge

I ran my solve script against the hosted challenge 100 times, each time guessing 100 flags.

Note that the accuracy can be greatly improved even just by tweaking some of the OpenCV parameters, which I am too lazy to do.

### median time
Measures the time taken between progress change, ie includes time taken after fetching new flags.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733179235/2024/12/085833302e83fbd84731551dc931a82b.png)

> Median time below 20 is low due to skipping all the transformations and noise removal.

### skip rate
To avoid spending too long on one single flag, I'll skip it if match confidence is low, i.e. get a new flag.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733179434/2024/12/5d298e0fb989a535b552042fc0402275.png)

### failure rate
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733179482/2024/12/b79a92bc8c148f7b1e22e1080055ee08.png)

> Note that the failure rates are high due to some level taking multiple retries, each retry will add 1 to failure, France has 10 duplicate flags, each with different ISO code, so if I retry all of the top options I will add 9 to failure count.

### exception rate

Where some unknown error occurred and I have to just get a new flag.
Usually caused by flag being too unreadable or exotic flag not finding a match.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733179699/2024/12/beb412934485d917c8a6002203cffa0b.png)

### box plot of time taken

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733179964/2024/12/0f50f5352cffd740281a4985b9a9a496.png)

> The clump of outliers around 3s and 4.5s means they 1-2 extra flags were requested.
> Outliers beyond 9 are ignored.
