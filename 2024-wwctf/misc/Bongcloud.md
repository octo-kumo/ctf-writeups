---
ai_date: 2025-04-27 05:24:25
ai_summary: Script uses Stockfish engine for chess to win games against a bot, demonstrating AI exploitation.
ai_tags:
  - ai
  - stockfish
  - exploitation
created: 2024-11-30T15:25
points: 153
solves: 95
updated: 2025-07-14T09:46
---

You can always play it manually!

In hindsight maybe 5s think time was too short, should've made the bot more difficult.

## solve script

Uses max level stockfish to absolutely destroy my bot.

```python [solve.py]
import chess.engine
import random
import asyncio
from pwn import *


TOTAL_GAMES = 3


async def get_engine_move(engine, board, limit, game_id, multipv):
    if isinstance(engine, chess.engine.XBoardProtocol):
        play_result = await engine.play(board, limit, game=game_id)
        return play_result.move
    multipv = min(multipv, board.legal_moves.count())
    with await engine.analysis(
        board, limit, game=game_id, info=chess.engine.INFO_ALL, multipv=multipv or None
    ) as analysis:
        infos = [None for _ in range(multipv)]
        async for new_info in analysis:
            if multipv and "multipv" in new_info:
                infos[new_info["multipv"] - 1] = new_info
        return analysis.info["pv"][0]


async def play(engine, conn, pvs, time_limit):
    won_games = 0
    for i in range(TOTAL_GAMES):
        game_id = random.random()
        board = chess.Board("rnbqkbnr/pppp1ppp/8/4p3/4P3/8/PPPPKPPP/RNBQ1BNR b kq - 1 2")
        print(f"Game {i+1}/{TOTAL_GAMES}")
        conn.sendlineafter(b'[Enter]\r\n', b'')
        conn.recvuntil(b' \x1b[0;30;107m   h g f e d c b a  \x1b[0m\r\n\r\n')

        while True:
            board.set_fen(conn.recvline().decode().strip())
            print(board+"\n")
            move = await get_engine_move(engine, board, time_limit, game_id, pvs)
            conn.sendlineafter(b'Your move:', str(move).encode())
            conn.recvuntil(b' \x1b[0;30;107m   h g f e d c b a  \x1b[0m\r\n\r\n')
            line = conn.recvline()
            if line.decode().strip() == 'Wow, you won!':
                won_games += 1
                print("yay won", won_games)
                break
            elif line.decode().strip() == 'You lost!' or line.decode().strip() == 'It\'s a draw!':
                print("lost/draw")
                return
            elif line.decode().strip() == f'You won all {TOTAL_GAMES} games!':
                conn.interactive()
                return
            conn.recvuntil(b' \x1b[0;30;107m   h g f e d c b a  \x1b[0m\r\n\r\n')


async def main():
    _, engine = await chess.engine.popen_uci(["C:\\Users\\zy\\Downloads\\stockfish-windows-x86-64-avx2\\stockfish\\stockfish-windows-x86-64-avx2.exe"])
    conn = remote("chess.chal.wwctf.com", 1337)
    limit = chess.engine.Limit(white_clock=30, black_clock=30, white_inc=1, black_inc=1)
    try:
        await play(engine,  conn,  pvs=0,  time_limit=limit)
    except Exception as e:
        print(e)
    finally:
        await engine.quit()

print("solving")
asyncio.set_event_loop_policy(chess.engine.EventLoopPolicy())
try:
    asyncio.run(main())
except:
    pass
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733001528/2024/11/f1e40222ca4f363184a5a2d093273663.png)

```flag
wwf{y0u_4r3_4_7ru3_ch355_m4573r}
```