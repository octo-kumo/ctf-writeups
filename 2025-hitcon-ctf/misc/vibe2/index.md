---
created: 2025-08-22T16:36
updated: 2025-08-24T10:22
points: 271
solves: 24
---
A random 64 length string is converted to their ASCII values and then plotted.

The plot is drawn as braille and sent to us, the task is to decode the original graph from the braille.


```
⡖⠖⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠒⠲⠲⡄
       ⡇                                                        ⢀                                   ⡇
      ⢠⡇                   ⡄     ⢠⡆                            ⣼⢹⡆                        ⣦         ⡇
120    ⡇                  ⢠⡇     ⢸⡇                            ⣿⠘⡇                        ⣿         ⡇
       ⡇                  ⣸⣷     ⢸⡇                     ⢀      ⣿ ⡇            ⡀           ⣿         ⡇
       ⡇                  ⡇⣿     ⢸⡇                    ⢠⢾⡄     ⡏ ⡇  ⢀         ⡇           ⣿⡄        ⡇
       ⡇                 ⢸⠃⢸     ⢸⡇                 ⢰  ⣾⠘⡇     ⡇ ⡇  ⢸⢧⢀⣤      ⡇ ⢠⡤⡄       ⡇⡇        ⡇
       ⡇                 ⢸ ⢸     ⢸⡇                 ⢸⡀ ⡇ ⡇     ⡇ ⣧  ⢸ ⠋⠸⡆     ⣷ ⢸⠇⣷      ⢰⡇⡇        ⡇
       ⡇     ⢀           ⡟ ⢸     ⢸⡇            ⣿    ⢸⡇ ⡇ ⡇     ⡇ ⢿  ⢸   ⢧     ⣿ ⢸ ⢸   ⣆  ⢸⡇⣷        ⡇
      ⢠⡇     ⢸   ⢀⡀      ⡇ ⢸     ⢸⣇            ⣿    ⣼⡇ ⡇ ⣷     ⡇ ⢸  ⢸   ⢸⡀    ⣿ ⢸ ⢸   ⣿  ⢸ ⢸        ⡇
100    ⡇     ⣾⡇  ⢸⡇     ⢰⠇ ⢸     ⢸⣿            ⣿    ⡿⡇ ⡇ ⢹     ⡇ ⢸  ⣿   ⢸⡇   ⢠⣿ ⢸ ⢸⡄ ⢠⣿  ⢸ ⢸        ⡇
       ⡇     ⡟⡇  ⢸⡇     ⢸  ⢸⡄    ⣼⢸       ⡀    ⣿    ⡇⡇⢸⡇ ⢸     ⡇ ⢸  ⡏    ⡇   ⢸⣿ ⢸ ⠘⡇ ⢸⢸⡄ ⢸ ⢸        ⡇
       ⡇    ⢰⠇⡇  ⢸⡇     ⣼  ⢸⡇    ⣿⢸      ⢸⡇    ⣿    ⡇⣇⢸⠃ ⢸     ⡇ ⢸⡀ ⡇    ⡇   ⢸⢻ ⢸  ⡇ ⢸⠈⡇ ⢸ ⢸⡄       ⡇
       ⡇    ⢸ ⡇  ⢸⡇     ⡟   ⡇  ⡄ ⡇⢸      ⢸⡇   ⢰⡿⡇   ⡇⣿⢸  ⢸⡆   ⢀⡇ ⠸⡇ ⡇    ⡇   ⢸⢸ ⣾  ⡇ ⣸ ⡇ ⢸ ⢸⡇       ⡇
       ⡇    ⣼ ⡇  ⢸⡇ ⢀   ⡇   ⡇  ⣇ ⡇⢸      ⢸⡇ ⢰ ⢸⠃⡇  ⢀⡇⢹⢸   ⡇   ⢸⡇  ⡇ ⡇    ⡇   ⢸⢸⡀⣿  ⢿ ⡟ ⣧ ⣼  ⡇       ⡇
       ⡇    ⡇ ⡇  ⣿⡇ ⣼   ⡇   ⡇  ⣿ ⡇⢸      ⣼⣧ ⢸ ⢸ ⡇  ⢸⡇⢸⢸   ⡇   ⢸⡇  ⡇⢠⡇    ⣇   ⢸⢸⡇⡏  ⢸ ⡇ ⢹ ⣿  ⡇       ⡇
      ⣀⡇    ⡇ ⡇  ⡏⡇ ⣿  ⢰⡇   ⡇  ⣿ ⡇⢸      ⡇⢻ ⣸⡇⢸ ⡇  ⢸ ⢸⣾   ⡇   ⢸⠁  ⣧⢸⠇    ⣿   ⢸⠸⡇⡇  ⠈⢧⡇ ⢸ ⡏  ⡇       ⡇
 80    ⡇   ⢰⠇ ⡇  ⡇⣧ ⣿⡀ ⢸    ⡇  ⣿ ⡇⢸      ⡇⢸ ⣿⡇⢸ ⡇  ⢸ ⢸⡟   ⣿   ⢸   ⢸⢸     ⢹   ⢸ ⡇⡇   ⠈⠁ ⠘⡇⡇  ⡇       ⡇
       ⡇   ⢸  ⣷  ⡇⣿ ⡿⡇ ⢸    ⣧ ⢸⣿ ⡇⢸    ⣄ ⡇⢸ ⡇⡇⢸ ⣿  ⢸ ⢸⡇   ⢸   ⢸   ⠸⣾     ⢸   ⢸ ⡇⡇       ⡇⡇  ⡇       ⡇
       ⡇   ⢸  ⣿  ⡇⢹ ⡇⡇ ⢸    ⣿ ⢸⢹ ⡇⢸⡇   ⣿⣸⠇⢸ ⡇⡇⢸ ⢹  ⢸ ⠈⡇   ⢸   ⢸    ⣿     ⢸   ⢸ ⡇⡇       ⢻⡇  ⣿       ⡇
       ⡇   ⡟  ⢹  ⡇⢸ ⡇⡇ ⡿    ⢹ ⢸⠸⡇⡇⢸⡇   ⡇⢿ ⢸ ⡇⣧⣸ ⢸  ⣾  ⠇   ⢸   ⢸    ⠻     ⢸   ⣾ ⣷⡇       ⢸⡇  ⢻ ⣦     ⡇
       ⡇   ⡇  ⢸ ⢠⡇⢸⢰⡇⣇ ⡇    ⢸ ⢸ ⣇⡇ ⡇   ⡇⠈ ⢸⣴⡇⢹⡿ ⢸  ⡏      ⠸⡇  ⢸          ⢸   ⣿ ⣿⡇       ⠸⠇  ⢸ ⣿     ⡇
       ⡇      ⢸ ⢸⡇⢸⢸⠇⣿⢠⡇    ⢸ ⢸ ⣿⡇ ⡇  ⢰⡇  ⠘⣿ ⢸⡇ ⢸⡄ ⡇       ⡇⢸ ⢸          ⢸⡇  ⡟ ⣿⠁           ⢸⢰⢿     ⡇
      ⣀⡇      ⢸ ⢸⠁⢸⢸ ⢸⢸     ⢸ ⢸ ⣿  ⡇  ⢸⠁   ⣿ ⢸⡇ ⠈⡇⢀⡇       ⡇⡾⡇⢸          ⠈⡇  ⡇ ⣿            ⢸⢸⠘⡇    ⡇
 60   ⠈⡇      ⢸ ⢸ ⢸⣸ ⢸⡾     ⢸⡇⢸ ⣿  ⡇  ⢸    ⣿ ⢸⡇  ⡇⢸⠃       ⣿⠃⡇⢸           ⡇  ⡇ ⢻            ⢸⣾ ⡇    ⡇
       ⡇      ⢸ ⢸ ⢸⣿ ⢸⡇     ⠈⡇⣾ ⣿  ⡇  ⢸    ⣿ ⠘⡇  ⣧⢸        ⠿ ⡇⣼           ⡇  ⡇ ⢸            ⢸⡏ ⣧    ⡇
       ⡇      ⢸ ⢸ ⠘⣿ ⠘⠁      ⡇⣿ ⢸  ⡇  ⢸    ⠋     ⢹⢸          ⡇⣿           ⡇  ⡇ ⠸            ⠘⡇ ⢸    ⡇
       ⡇      ⢸⡇⢸  ⣿         ⡇⡏ ⢸  ⢳  ⣾          ⢸⡟          ⢻⣿           ⢻  ⡇               ⠁ ⠈    ⡇
       ⡇      ⢸⡇⡟  ⡇         ⣇⡇    ⠈⡇ ⡏          ⢸⡇          ⢸⡇           ⢸  ⡇                      ⡇
       ⡇      ⠈⡇⡇  ⡇         ⢻⡇     ⢹ ⡇           ⠇          ⢸⡇           ⠘⡇ ⡇                      ⡇
      ⢀⡇       ⣿⠁  ⠁         ⢸⡇     ⠈⣇⡇                      ⢸⡇            ⡇⢰⡇                      ⡇
 40   ⠈⡇       ⣿             ⢸⡇      ⢸⡇                      ⠈⡇            ⢷⣸⠇                      ⡇
       ⡇       ⠇             ⢸⡇                               ⠇            ⠈                        ⡇
       ⡇                     ⠘⠃                                                                     ⡇
       ⡇                                                                                           ⢀⡇
       ⠉⠉⠉⠉⠋⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠙⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠋⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠙⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠋⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠋⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠙⠉⠉⠉⠉⠉⠉⠉⠉
          0           10            20           30            40           50            60
```


## parsing the braille
No matter what we do, we will need to parse the braille.

My idea is simple, just convert the entire string into a pixel graph with size `2*w` and `4*h`.
Loop over the string and set values accordingly, AI fulfilled my requirements pretty quickly.

```python
def braille_to_pixels(braille_char: str) -> list[bool]:
    code_point = ord(braille_char)
    if code_point < 0x2800 or code_point > 0x28FF:
        return [False] * 8
    byte_val = code_point - 0x2800
    return [
        (byte_val >> 0) & 1 == 1,
        (byte_val >> 1) & 1 == 1,
        (byte_val >> 2) & 1 == 1,
        (byte_val >> 3) & 1 == 1,
        (byte_val >> 4) & 1 == 1,
        (byte_val >> 5) & 1 == 1,
        (byte_val >> 6) & 1 == 1,
        (byte_val >> 7) & 1 == 1
    ]

def extract_pixels_from_braille(braille_text: str) -> list[list[bool]]:
    lines = braille_text.strip().split('\n')
    if not lines:
        return []
    h = len(lines)
    w = max((len(line) for line in lines), default=0)
    if w == 0:
        return []
    pixel_height = h * 4
    pixel_width = w * 2
    pixels = [[False] * pixel_width for _ in range(pixel_height)]
    for y, line in enumerate(lines):
        for x, char in enumerate(line):
            pixel_values = braille_to_pixels(char)
            px = x * 2
            py = y * 4
            if py < pixel_height and px < pixel_width:
                pixels[py][px] = pixel_values[0]
            if py + 1 < pixel_height and px < pixel_width:
                pixels[py + 1][px] = pixel_values[1]
            if py + 2 < pixel_height and px < pixel_width:
                pixels[py + 2][px] = pixel_values[2]
            if py < pixel_height and px + 1 < pixel_width:
                pixels[py][px + 1] = pixel_values[3]
            if py + 1 < pixel_height and px + 1 < pixel_width:
                pixels[py + 1][px + 1] = pixel_values[4]
            if py + 2 < pixel_height and px + 1 < pixel_width:
                pixels[py + 2][px + 1] = pixel_values[5]
            if py + 3 < pixel_height and px < pixel_width:
                pixels[py + 3][px] = pixel_values[6]
            if py + 3 < pixel_height and px + 1 < pixel_width:
                pixels[py + 3][px + 1] = pixel_values[7]
    return pixels, pixel_width, pixel_height
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/e_negate/v1755896191/20250822165631760.png/ee7e4367de0f5f15677f563009189697.png)

## parsing the graph
LLM was kind useless for this part so its handwritten + copilot on comments.

> Side note but this somehow increases my code readability as I have a line of comment showing my intent then the implementation.

### axis
Notice that the chart axes only take up 1 pixel, so we can just look for some row / column of pixels that is largely filled.

```python
# y axis is the first column that has a lot of trues throughout the column
y_axis_x = next(x for x in range(len(pixels[0])) if sum(pixels[y][x] for y in range(h)) > h // 2)
# x axis is the first row that has a lot of trues throughout the row (from bottom)
x_axis_y = next(y for y in range(len(pixels) - 1, -1, -1) if sum(pixels[y]) > w // 2)
```

Then notice that the axis markers take up 1 pixel for x axis and 2 pixel for y axis.

And that the order of numbers appearing in the string happens to be `(y markers top down) | (x markers left to right)`, which means we can just pop from the front and assign each marker their data space value.

```python
all_numbers = re.findall(r'\b\d+\b', graph)
all_numbers = list(map(int, all_numbers))
y_axis_markers = [(y-0.5, all_numbers.pop(0)) for y in range(1, h) if pixels[y][y_axis_x - 1] and pixels[y-1][y_axis_x-1]]
x_axis_markers = [(x, all_numbers.pop(0)) for x in range(w) if pixels[x_axis_y + 1][x]]
```

Because pixel space and data space are linear, we can find the parameters like this.

```python
def compute_linear_params(markers):
    graph_coords, real_coords = zip(*markers)
    A = np.vstack([graph_coords, np.ones(len(graph_coords))]).T
    m, b = np.linalg.lstsq(A, real_coords, rcond=None)[0]
    return m, b
m_x, b_x = compute_linear_params(x_axis_markers)
m_y, b_y = compute_linear_params(y_axis_markers)
# estimate data space coordinate from pixel coordinate
def est_x(graph_x):
	return float(m_x * graph_x + b_x)
def est_y(graph_y):
	return float(m_y * graph_y + b_y)
```

Now that we can convert from pixel space to data space, we can read the graph itself.

### graph

Each column of the pixel graph has a start and end y value, defined by the first pixel filled and last pixel filled.

```python
# now extract the graph as a list of ranges
graph_ranges = []
# print(f"Extracting graph ranges from x={y_axis_x + 1} to x={w-2}")
for x in range(y_axis_x + 1, w - 2):
	column = [pixels[y][x] for y in range(round(y_axis_markers[0][0] - 1.5), x_axis_y - 1)]
	ys = [i for i, val in enumerate(column) if val]
	if ys:
		start_y, end_y = ys[0], ys[-1]
		graph_ranges.append((est_x(x),est_y(start_y + y_axis_markers[0][0] - 1.5+BIAS),est_y(end_y + y_axis_markers[0][0] - 1.5+BIAS)))
```

If we just take those values and plot it we get something like this.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/e_negate/v1755897033/20250822171033591.png/21c7e0f3720e7fc595941a30982afc12.png)

OwO, we are really close! (or is it?)

#### extrema
Notice how the extrema are very good estimators? The $y_{low}$ of a local minima is almost exactly equal to the truth and vice versa for local maxima.

(Orange is truth)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/e_negate/v1755897598/20250822171958155.png/7fe9bb1bc4aa50ce27389b382249bee8.png)

We can estimate ~3/4 of the points to 0.01 accuracy just with this.

```python
guesses = {}
for i,(x, y1, y2) in enumerate(graph_ranges):
    # check left and right neighbors centers
    if i > 0 and i < len(graph_ranges) - 1:
        left_x, left_y1, left_y2 = graph_ranges[i - 1]
        right_x, right_y1, right_y2 = graph_ranges[i + 1]
        if y1 >= left_y1 and y1 >= right_y1:
            print(f"Local maximum at x={round(x)}, y={y1}, truth={truth[round(x)]}")
            guesses[x] = y1
        elif y2 <= left_y2 and y2 <= right_y2:
            print(f"Local minimum at x={round(x)}, y={y2}, truth={truth[round(x)]}")
            guesses[x] = y2
    elif i == 0:
        # if lower than the next one, then it's a local minimum
        right_x, right_y1, right_y2 = graph_ranges[i + 1]
        if y2 <= right_y2:
            print(f"Local minimum at x={round(x)}, y={y2}, truth={truth[round(x)]}")
            guesses[x] = y2
        elif y1 >= right_y1:
            print(f"Local maximum at x={round(x)}, y={y1}, truth={truth[round(x)]}")
            guesses[x] = y1
    elif i == len(graph_ranges) - 1:
        # if higher than the previous one, then it's a local maximum
        left_x, left_y1, left_y2 = graph_ranges[i - 1]
        if y1 >= left_y1:
            print(f"Local maximum at x={round(x)}, y={y1}, truth={truth[round(x)]}")
            guesses[x] = y1
        elif y2 <= left_y2:
            print(f"Local minimum at x={round(x)}, y={y2}, truth={truth[round(x)]}")
            guesses[x] = y2

error = np.mean([guesses[i] - truth[round(i)] for i in guesses if i < len(truth)])
error_no_truth = np.mean([guesses[i] - round(guesses[i]) for i in guesses if i < len(truth)])
print(f"Mean error: {error:f}")
print(f"Mean error without truth: {error_no_truth:f}")
error_x = np.mean([abs(i - round(i)) for i in guesses if i < len(truth)])
print(f"Mean X error: {error_x:f}")

any_error = any(abs(guesses[i] - truth[round(i)]) > 0.5 for i in guesses if i < len(truth))
print(f"Any error > 0.5: {any_error}")
print(f"Missing guesses: {len(truth)-len(guesses)}")
```

```
Mean error: 0.0117
Mean error without truth: -0.0283
Mean X error: 0.1358
Any error > 0.5: True
Missing guesses: 14
```

#### edges
Next I tried to use various cases of pixel ranges, such as going up, going down etc to estimate the y value of the touching edge, or +0.5 pixel x.

However it actually decreased accuracy so I moved on.

#### asking AI
I then asked the LLMs to come up with a algorithm to find the points. I told it about the minima / maxima stuff.

```python
y1 = np.array([y1 for x, y1, y2 in graph_ranges])
y2 = np.array([y2 for x, y1, y2 in graph_ranges])
x = np.array([x for x, y1, y2 in graph_ranges])
mid_y = (y1 + y2) / 2
adjusted_y = mid_y.copy()

# Step 1: Adjust for local extrema
for i in range(1, len(x) - 1):
    if mid_y[i-1] > mid_y[i] < mid_y[i+1]:
        adjusted_y[i] = y2[i]
    elif mid_y[i-1] < mid_y[i] > mid_y[i+1]:
        adjusted_y[i] = y1[i]

# Step 2: Compute slopes and delta_slopes for curvature adjustment
if len(x) > 2:
    slopes = (mid_y[1:] - mid_y[:-1]) / (x[1:] - x[:-1])
    delta_slopes = slopes[1:] - slopes[:-1]  # length len(x)-3
    abs_deltas = np.abs(delta_slopes)
    threshold = np.mean(abs_deltas) if len(abs_deltas) > 0 else 0

    # Adjust at points with significant curvature (associate with i+1, the middle point)
    for j in range(len(delta_slopes)):
        i = j + 1  # index in adjusted_y
        if abs_deltas[j] > threshold:
            if delta_slopes[j] > 0:
                adjusted_y[i] = y2[i]
            else:
                adjusted_y[i] = y1[i]

# Interpolate/extrapolate to integer x (adjust range based on your min/max x)
f = interp1d(x, adjusted_y, kind='linear', fill_value='extrapolate')
integer_x = np.arange(0, 64)  # Example up to 3; use np.arange(0, 61) for full
estimated_y = f(integer_x)
```

Looking at its code it seems to just be doing what I told it that I did.

But surprisingly it worked pretty well, except for inaccuracies on the extrema.

#### combining
Which means, by combing my data (very accurate on extrema) and the AI's result (accurate on other points),  we can get a very close approximation of the graph!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755898534/20250822173534108.png/39e640e7bf979f26846caea5d28c991f.png)

```python
not_equal_count = sum(1 for i in range(len(truth)) if truth[i] != estimated_y[i])
print(f"Number of points not equal to truth: {not_equal_count}")
# Number of points not equal to truth: 17
```

## optimization

However there is still 25% inaccuracy from a perfect match, and we only get 1 chance at guessing the password so it has to be 100% accurate.

My next idea was to optimise the password by tweaking characters based on pixel differences.

We can generate the text chart like this, then compare it with the one we received and try to optimise the numbers, then compare again and repeat.

```python
def get_chart(seq):
    plt.figure()
    plt.plot(seq)
    g = ""
    for manager in Gcf.get_all_fig_managers():
        canvas = manager.canvas
        canvas.draw()
        g = canvas.to_txt()
    plt.close()
    return g
```

- `parse_chart` is parsing the brailles into `pixels, width, height`.
- `solve_chart` tries to solve the graph and returns `(64,), extra_stuff`.

~~Now do vibe coding to fit the theme of the challenge~~

- The AI first gave me a completely random hill climb, which was basically brute forcing the $77^{64}$ possible password space.
- The AI did not use the information about x coordinate of pixel differences at first.
- It tried to search over every possible `ALPHA` value for each proposed solution and was too slow, I made it search locally.
- The AI was using some complicated `resid.raval` and a whole lot of other stuff at first, idk what it was doing but it was not working, so I just changed it to a simple `resid.sum`, which worked lol.
- I added some logic to increase search space as the round number increases to avoid getting stuck.

Anyways after rounds of manually tweaking and simplification and more *vibe coding*, I got to this.

```python
import os
import time
from typing import Callable, List, Tuple

import numpy as np
from chal import ALPHA, get_chart
from parse import draw_diff, parse_chart

alpha_list = list(ALPHA)
alpha_index_of = {ord(c): i for i, c in enumerate(alpha_list)}

def targeted_search(
    attempt: List[int],
    target_pixels: np.ndarray,
    target_w: int,
    target_h: int,
    *,
    est_x: Callable[[int, int], int] = None,
    max_rounds: int = 30,
    draw: bool = True,
) -> Tuple[List[int], int]:
    if draw:
        os.makedirs("frames", exist_ok=True)
        [os.remove(os.path.join("frames", f)) for f in os.listdir("frames")]

    best = list(attempt)
    pix, w, h = parse_chart(get_chart(tuple(best)))
    assert (w, h) == (target_w, target_h), "initial attempt changed dims"

    best_arr = np.array(pix, dtype=np.int16)
    T = np.array(target_pixels, dtype=np.int16)
    best_diff = int(np.sum(np.abs(T - best_arr)))

    print(f"[targeted] start diff={best_diff} shape={T.shape}")

    start_time = time.time()
    frame_counter = 0

    for rnd in range(1, max_rounds + 1):
        max_local_offset = 10 * rnd
        round_start = time.time()
        improved = False
        cache = {}

        resid = np.abs(T - best_arr)
        diff_sum_per_col = np.sum(resid, axis=0)
        xs = np.nonzero(diff_sum_per_col)[0]

        deltas_for_pos = (-1, 0, 1) if rnd == 1 else ((-2, -1, 0, 1, 2) if rnd == 2 else tuple(range(-3, 4)))
        candidate_positions = {
            p
            for x in xs
            for p in (int(round(est_x(float(x)))) + d for d in deltas_for_pos)
            if 0 <= p < 64
        }

        if not candidate_positions:
            print("[targeted] no candidate positions inferred, stopping")
            break

        pos_scores = {p: 0 for p in candidate_positions}
        for x in xs:
            guessed = int(round(est_x(float(x))))
            for delta in (-1, 0, 1):
                p = guessed + delta
                if p in pos_scores:
                    pos_scores[p] += diff_sum_per_col[x]

        positions = sorted(candidate_positions, key=lambda p: -pos_scores[p])
        print(f"[targeted] candidate positions: {candidate_positions}")

        for pos in positions:
            cur_byte = best[pos]
            offsets = [0] + [d for o in range(1, max_local_offset + 1) for d in (-o, o)]
            if cur_byte in alpha_index_of:
                cur_idx = alpha_index_of[cur_byte]
                candidates = [ord(alpha_list[cur_idx + off]) for off in offsets if 0 <= cur_idx + off < len(alpha_list)]
            else:
                candidates = [cur_byte + off for off in offsets if 0 <= cur_byte + off <= 255]
            best_local_byte = cur_byte
            best_local_diff = best_diff
            for b in candidates:
                if b == cur_byte:
                    continue
                key = (pos, b)
                if key in cache:
                    diff = cache[key][0]
                else:
                    proposal = list(best)
                    proposal[pos] = b
                    try:
                        r = get_chart(tuple(proposal))
                        ppix, pw, ph = parse_chart(r)
                    except Exception:
                        continue
                    if (pw, ph) != (target_w, target_h):
                        continue
                    parr = np.array(ppix, dtype=np.int16)
                    diff = int(np.sum(np.abs(T - parr)))
                    cache[key] = (diff, parr)

                if diff < best_local_diff:
                    best_local_diff = diff
                    best_local_byte = b
                    if diff == 0:
                        break
            if best_local_byte != cur_byte and best_local_diff < best_diff:
                best[pos] = best_local_byte
                best_diff = best_local_diff
                best_arr = cache[(pos, best_local_byte)][1]
                improved = True
                if draw:
                    frame_counter += 1
                    img, _, _ = draw_diff(target_pixels, best_arr, target_w, target_h)
                    img_up = img.resize((img.width * 4, img.height * 4), resample=0)
                    img_up.save("frames/frame_{:04d}.png".format(frame_counter))
                print(f"[round {rnd}] pos {pos} improved -> diff={best_diff} ; byte={cur_byte} -> {best_local_byte}")
                if best_diff == 0:
                    print("Exact match found!")
                    print(f"Done in {time.time() - start_time:f}s")
                    return best, 0
        print(f"[targeted] finished round {rnd} improved={improved} best_diff={best_diff} round_time={time.time() - round_start:f}s")
        if not improved:
            print("[targeted] no improvement this round, stopping")
            break
    print(f"[targeted] finished: best_diff={best_diff} time={time.time() - start_time:f}s")
    return best, best_diff
```

### the driver

Now the solve script itself.

You might notice that I'm repeatedly getting new graphs until I get one that is favourable.

This is because after analysing the distribution of total number of pixel differences, I realised that it is either 3000+ (initial solve failed miserably), ~500 (likely axes marker mismatch) and <400 (very nice match).

Instead of actually implementing logic to handle those, it is easier to just get a new graph, because diff<400 is actually pretty common.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755900473/20250822180753627.png/5ab91c40a067b4c4c088997fc2a21edb.png)


```python
import traceback
from pwn import *
context.log_level = 'error'
import random
from chal import get_chart
from parse import parse_chart, solve_chart, draw_diff
from search import targeted_search
import numpy as np
import signal
ALPHA = (
    "abcdefghijklmnopqrstuvwxyz" +
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ" +
    "0123456789" +
    "_:-+@!#$%^&*,./"
)

LOCAL = True

def rem_chart():
    conn = remote("vibe2.chal.hitconctf.com", 3000)
    conn.recvuntil(b"I will draw the password out for you.\n\n\n")
    chart = conn.recvuntil(b"\n\n\n",drop=True)
    return chart.decode(), conn

while True:
    # just keep trying until we get a favorable chart
    try:
        if LOCAL:
            ans = list(bytes(random.choices(ALPHA.encode(), k=64))) # 64
            chart = get_chart(tuple(ans))
        else:
            chart, conn = rem_chart()
        c1, w, h = parse_chart(chart)
        attempt, extra = solve_chart(chart) # attempt to solve the chart into 64
        _c = get_chart(tuple(attempt))
        c2, w2, h2 = parse_chart(_c)
        if not w == w2 and h == h2:
            if not LOCAL: conn.close()
            print(f"Width or height mismatch: {w}x{h} vs {w2}x{h2}")
            continue
    except Exception as e:
        traceback.print_exc()
        continue
    img, diff, per = draw_diff(c1, c2, w, h)
    if diff > 400:
        print(f"Diff too high: {diff} ({per:f}%)")
        if not LOCAL: conn.close()
        continue
    y_marker_column = extra["y_axis_x"]
    if any(c2[y][y_marker_column-1] != c1[y][y_marker_column-1] for y in range(h)):
        print("Y axis column changed")
        if not LOCAL: conn.close()
        continue
    print(f"Current diff: {diff} ({per:f}%)")
    img.save("diff.png")
    # functions to map (image space -> data space)
    est_x = extra["est_x"]
    est_y = extra["est_y"]
    def handle_sigint(signum, frame):
        print("Interrupted by user")
        if not LOCAL:
            try:
                conn.close()
            except Exception:
                pass
        exit(0)
    signal.signal(signal.SIGINT, handle_sigint)

    seq, diff = targeted_search(
        attempt, np.array(c1), w, h, est_x=est_x
    )
    
    if diff != 0:
        print(f"Failed to find exact match, diff={diff}")
        if not LOCAL: conn.close()
        continue
    # this little shit crashed a solved challenge 3 times
    seq = list(seq)
    seq = [int(s) for s in seq]
    if LOCAL:
        print(ans)
        for i in range(64):
            if seq[i] != ans[i]:
                print(f"Mismatch at index {i}: {seq[i]} != {ans[i]}")
                exit(1)
    if not LOCAL:
        password = bytes(list(seq)).decode()
        conn.sendline(password.encode())
        print(conn.recvall().decode())
        conn.close()
    break
```


```
❯ python solve.py
Width or height mismatch: 202x148 vs 210x148
Width or height mismatch: 202x148 vs 206x148
Width or height mismatch: 202x148 vs 210x148
Current diff: 388 (1.30%)
[targeted] start diff=388 shape=(148, 202)
[targeted] candidate positions: {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62}
[round 1] pos 14 improved -> diff=335 ; byte=82.0 -> 75
[round 1] pos 43 improved -> diff=310 ; byte=87.0 -> 90
[round 1] pos 26 improved -> diff=270 ; byte=61.0 -> 54.0
[round 1] pos 27 improved -> diff=264 ; byte=99.0 -> 100
[round 1] pos 36 improved -> diff=259 ; byte=89.0 -> 90
[round 1] pos 49 improved -> diff=221 ; byte=73.0 -> 68
[round 1] pos 32 improved -> diff=214 ; byte=36.0 -> 35
[round 1] pos 33 improved -> diff=191 ; byte=103.0 -> 100
[round 1] pos 8 improved -> diff=175 ; byte=80.0 -> 78
[round 1] pos 9 improved -> diff=167 ; byte=76.0 -> 75
[round 1] pos 30 improved -> diff=158 ; byte=67.0 -> 66
[round 1] pos 29 improved -> diff=148 ; byte=76.0 -> 75
[round 1] pos 54 improved -> diff=134 ; byte=69.0 -> 67
[round 1] pos 3 improved -> diff=121 ; byte=103.0 -> 105
[round 1] pos 21 improved -> diff=117 ; byte=47.0 -> 46
[round 1] pos 59 improved -> diff=108 ; byte=50.0 -> 49
[round 1] pos 56 improved -> diff=99 ; byte=72.0 -> 73
[round 1] pos 23 improved -> diff=92 ; byte=36.0 -> 35
[round 1] pos 38 improved -> diff=84 ; byte=102.0 -> 103
[round 1] pos 11 improved -> diff=77 ; byte=53.0 -> 52
[round 1] pos 61 improved -> diff=70 ; byte=96.0 -> 95.0
[targeted] finished round 1 improved=True best_diff=70 round_time=90.67s
[targeted] candidate positions: {18, 19, 20, 21, 22, 23, 24, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47}
[round 2] pos 36 improved -> diff=38 ; byte=90 -> 94
[round 2] pos 43 improved -> diff=16 ; byte=90 -> 94
[round 2] pos 44 improved -> diff=9 ; byte=38.0 -> 37
[round 2] pos 21 improved -> diff=0 ; byte=46 -> 45
Exact match found!
Done in 107.6s
```

> Testing this locally yield ~100s average time, could probably make it faster with threads but the allocated time is 600s.

### the flag
```flag
hitcon{well_this_is_just_a_ppc_challenge_now_that_the_ai_have_been_evicted}
```

And there we have it!

I also generated a GIF to visualise the optimisation steps.

![output.gif](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755900841/20250822181401559.gif/40ed54fd78efa13a0b8bbed4ef673103.gif)
