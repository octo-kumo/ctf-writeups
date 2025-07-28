---
ai_date: 2025-04-27 05:12:24
ai_summary: Decrypted flag by observing chess piece positions and AES encryption
ai_tags:
  - aes
  - decryption
  - chess
created: 2024-12-27T12:26
points: 100
solves: 43
updated: 2025-07-14T09:46
---

## mod
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735320415/2024/12/0b089ef7463ae954d67c5361ac6b9b78.png)

## remove black pieces & move white pieces

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735320645/2024/12/57ee7c9ebf6f4cd63802d28dbbd8e686.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735320515/2024/12/8ec2e70a5c12e06f6fa690f06d89ca57.png)

Editing `xBoard` and `yBoard` allows us to move the piece.

We will edit only queen and rook.

## win?

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735320777/2024/12/b28bcafcf631618aa6f875fb89f0672c.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735320889/2024/12/62d8329f32bc45ca4eaa13208b13852e.png)

Hrm...

How about making the enemy queen my queen.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735321954/2024/12/3e602652bb679dcb506750efd90f4ad4.png)

Hrmm.... make the energy king ours?
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735322060/2024/12/e4f46f40f5152f67d59c6b38ef632ee1.png)

...

What if we set the positions via the controller?

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735322278/2024/12/9615628e0c548edecf06acb374496358.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735322349/2024/12/e1444581797e4b7295f8220d9c7f3967.png)

Hrmm...

## decompile

I decided to decompile it in dySpy to see what's actually happening.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735326595/2024/12/e3b1d4e3545383e10b5efe174c916ab9.png)

Turns out it was crafting the chess piece locations to form a key and iv, which is used to decrypt a piece of text.

Since the challenge itself says "Can you beat me in 1 move?", and the black king always eats our king directly, let's return the favours, and have our king eat black king in 1 move.

*Used friendly neighbourhood ChatGPT to convert the C-sharp code to python*

```python
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import numpy as np


class Component:
    def __init__(self, name, x, y):
        self.name = name
        self.x = x
        self.y = y
        self.active = name is not "black_king"

    def get_x_board(self):
        return self.x

    def get_y_board(self):
        return self.y


def Create(name, x, y):
    return Component(name, x, y)


class Game:
    def __init__(self):
        self.game_over = False
        self.player_white = [Create("white_rook", 0, 0),
                             Create("white_knight", 1, 0),
                             Create("white_bishop", 2, 0),
                             Create("white_queen", 3, 0),
                             Create("white_king", 4, 7),
                             Create("white_bishop", 5, 0),
                             Create("white_knight", 6, 0),
                             Create("white_rook", 7, 0),
                             Create("white_pawn", 0, 1),
                             Create("white_pawn", 1, 1),
                             Create("white_pawn", 2, 1),
                             Create("white_pawn", 3, 1),
                             Create("white_pawn", 4, 1),
                             Create("white_pawn", 5, 1),
                             Create("white_pawn", 6, 1),
                             Create("white_pawn", 7, 1)]
        self.player_black = [Create("black_rook", 0, 7),
                             Create("black_knight", 1, 7),
                             Create("black_bishop", 2, 7),
                             Create("black_queen", 3, 7),
                             Create("black_king", 4, 7),
                             Create("black_bishop", 5, 7),
                             Create("black_knight", 6, 7),
                             Create("black_rook", 7, 7),
                             Create("black_pawn", 0, 6),
                             Create("black_pawn", 1, 6),
                             Create("black_pawn", 2, 6),
                             Create("black_pawn", 3, 6),
                             Create("black_pawn", 4, 6),
                             Create("black_pawn", 5, 6),
                             Create("black_pawn", 6, 6),
                             Create("black_pawn", 7, 6)]

    def winner(self, player_winner):
        self.game_over = True
        print(f"{player_winner} wins")  # Simulating GUI text display

        if player_winner == "white":
            print("Not Good Enough")

            # Initialize arrays
            array = np.zeros((8, 8), dtype=int)
            array2 = np.zeros((8, 8), dtype=int)

            # Populate white pieces
            for game_object in self.player_white:
                if game_object is not None and game_object.active:
                    component = game_object  # Assuming it provides the required attributes
                    xboard, yboard = component.get_x_board(), component.get_y_board()
                    piece_name = component.name
                    array[xboard, yboard] = self.piece_value(piece_name, "white")

            # Populate black pieces
            for game_object in self.player_black:
                if game_object is not None and game_object.active:
                    component = game_object
                    xboard, yboard = component.get_x_board(), component.get_y_board()
                    piece_name = component.name
                    array2[xboard, yboard] = self.piece_value(piece_name, "black")

            # Transform matrices and decrypt text
            transformed_matrix = self.rm(array2)
            array5 = self.f1(array)
            array6 = self.f2(transformed_matrix)
            cipher_text = "LlfqPs1MOul1Jr09d6dZditrkXUgIfMDc3Lh6/z5Ufv6E2G8ARHNvE7xQ9jrGBRg"
            decrypted_text = self.fw(cipher_text, array5, array6)

            print("Check your clipboard!")
            print(decrypted_text)  # Simulate setting clipboard content

    def piece_value(self, name, color):
        values = {
            "king": 1,
            "queen": 2,
            "rook": 3,
            "bishop": 4,
            "knight": 5,
            "pawn": 6
        }
        return values.get(name.replace(f"{color}_", ""), 0)

    def f1(self, matrix):
        array = bytearray(32)
        num = 0
        for i in range(8):
            for j in range(8):
                if num < len(array):
                    array[num] = matrix[j, i] * 16 + j + i
                    num += 1
        return array

    def f2(self, matrix):
        array = bytearray(16)
        num = 0
        for num2 in range(8):
            for num3 in range(8):
                if num < len(array):
                    array[num] = matrix[num3, num2] * 2 + num3 % 2 + num2 % 2
                    num += 1
        return array

    def rm(self, original_matrix):
        length = original_matrix.shape[0]
        return np.flip(original_matrix)

    def fw(self, cipher_text, key, iv):
        aes = AES.new(key, AES.MODE_CBC, iv)
        encrypted_data = base64.b64decode(cipher_text)
        decrypted_data = unpad(aes.decrypt(encrypted_data), AES.block_size)
        return decrypted_data.decode('utf-8')


game = Game()
game.winner("white")
```

```flag
0xL4ugh{A_H0n0ur4ble_B4tt13_B3tw33n_K1NG5}
```