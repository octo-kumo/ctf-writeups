---
ai_date: '2025-04-27 05:21:30'
ai_summary: Optimized image using adversarial attack to exploit model's weakness,
  submitting manipulated signature for flag
ai_tags:
- adversarial
- model-exploitation
- input-optimization
created: 2024-08-02T20:15
points: 499
solves: 4
tags:
- AI
updated: 2024-08-05T19:35
---

We are given a `h5` file, and the website asks for a signature.

Ah, adversarial attacks.

## analysis

```python
import numpy as np
from tensorflow.keras.models import load_model
model = load_model('model.h5')
model.input_shape # (None, 96, 128, 1)
model.summary()
```

```text
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┓
┃ Layer (type)                         ┃ Output Shape                ┃         Param # ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━┩
│ conv2d (Conv2D)                      │ (None, 90, 122, 64)         │           3,200 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ max_pooling2d (MaxPooling2D)         │ (None, 45, 61, 64)          │               0 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ conv2d_1 (Conv2D)                    │ (None, 41, 57, 64)          │         102,464 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ max_pooling2d_1 (MaxPooling2D)       │ (None, 20, 28, 64)          │               0 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ conv2d_2 (Conv2D)                    │ (None, 18, 26, 128)         │          73,856 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ max_pooling2d_2 (MaxPooling2D)       │ (None, 9, 13, 128)          │               0 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ conv2d_3 (Conv2D)                    │ (None, 7, 11, 256)          │         295,168 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ flatten (Flatten)                    │ (None, 19712)               │               0 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ dense (Dense)                        │ (None, 512)                 │      10,093,056 │
├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
│ dense_1 (Dense)                      │ (None, 1)                   │             513 │
└──────────────────────────────────────┴─────────────────────────────┴─────────────────┘

 Total params: 10,568,259 (40.31 MB)

 Trainable params: 10,568,257 (40.31 MB)

 Non-trainable params: 0 (0.00 B)

 Optimizer params: 2 (12.00 B)
```

It seems to output 1 or values near 1 for almost everything, that means that is probably the wrong value.

We would want a image that will give us lower outputs.

## optimize
I used almighty Adam with a learning rate of 0.01.
It was able to converge very quickly within 100 epoch, but I ran it til 2000 epochs to be safe.

```python
import tensorflow as tf

input_tensor = tf.Variable(input_data, dtype=tf.float32)

def loss_fn(input_tensor):
    prediction = model(input_tensor[tf.newaxis, ...])
    return prediction
optimizer = tf.keras.optimizers.Adam(learning_rate=0.01)
epochs = 2000
for epoch in range(epochs):
    with tf.GradientTape() as tape:
        loss = loss_fn(input_tensor)
    gradients = tape.gradient(loss, [input_tensor])
    optimizer.apply_gradients(zip(gradients, [input_tensor]))
    if epoch % 100 == 0:
        print(f'Epoch {epoch}, Loss: {loss.numpy()}')

optimized_input = input_tensor.numpy()
```

```
Epoch 0, Loss: [[0.99548364]]
Epoch 100, Loss: [[0.00178202]]
Epoch 200, Loss: [[0.00101832]]
Epoch 300, Loss: [[0.0006519]]
Epoch 400, Loss: [[0.0004537]]
Epoch 500, Loss: [[0.00033495]]
Epoch 600, Loss: [[0.00025783]]
Epoch 700, Loss: [[0.0002047]]
Epoch 800, Loss: [[0.00016647]]
Epoch 900, Loss: [[0.0001381]]
Epoch 1000, Loss: [[0.00011637]]
Epoch 1100, Loss: [[9.9308294e-05]]
Epoch 1200, Loss: [[8.564472e-05]]
Epoch 1300, Loss: [[7.45256e-05]]
Epoch 1400, Loss: [[6.5351305e-05]]
Epoch 1500, Loss: [[5.7699173e-05]]
Epoch 1600, Loss: [[5.1251296e-05]]
Epoch 1700, Loss: [[4.5772984e-05]]
Epoch 1800, Loss: [[4.107793e-05]]
Epoch 1900, Loss: [[3.7016132e-05]]
```

## send payload

The final optimized image is very sketchy but it works. (I made 2)

![optimized2.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722644989/2024/08/c71c10265c77abbd4b138ad8d7e557f7.png)

![optimized_output.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722644998/2024/08/4c0dd7941f3f19d8c9f6c6baa47fb7a4.png)

```python
from PIL import Image
import numpy as np
import base64
import requests

image_data = optimized_input.reshape((96, 128))
image_data = (image_data - image_data.min()) / (image_data.max() - image_data.min()) * 255
image_data = image_data.astype(np.uint8)
image = Image.fromarray(image_data, mode='L')
image.save('optimized_output.png')

with open('optimized_output.png', 'rb') as image_file:
    encoded_image = base64.b64encode(image_file.read()).decode('utf-8')
r = requests.post("http://challs.tfcctf.com:30403/", data={'signature': "data:image/png;base64,"+encoded_image})
print(r.text)
```

```flag
CTF{f1ght1n9_4rt1fic1al_in73ll1g3nce}
```