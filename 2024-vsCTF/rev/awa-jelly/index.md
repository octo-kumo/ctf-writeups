---
ai_date: '2025-04-27 05:22:54'
ai_summary: Challenge involves character rearrangement, likely a Caesar cipher or
  similar substitution technique.
ai_tags:
- substitution
- decryption
- ciphertext
created: 2024-06-14T20:10
points: 395
solves: 184
updated: 2024-08-04T19:46
---

> [JellyCTF](https://jellyc.tf/) has some amazing challenges on [Jelly Hoshiumi](https://x.com/jellyhoshiumi) (星海ジェリー), one of the few VTubers who loves CTF. Inspired by a challenge, I made one based on [AWA5.0](https://github.com/TempTempai/AWA5.0). Can you find the redacted input that matches the screenshot?

Some examination of the input/output made me realise that the program simply rearrange the characters of the input.

So I just modified some parts of the original code to allow sync operations and used the following script to check the position mapping of the program.

```js
function runCode() {
    var codeStr = "awa awa awa awawawa awa awawawawa awa awawawa awa awa awa awawa awa awa awawawa awa awa awa awawawa awa awawawa awa awa awawa awa awa awa awawawa awa awa awa awa awawa awa awawawa awa awa awawawa awa awa awawawa awa awa awawa awawa awa awawawa awa awa awa awawawa awa awawawa awa awawa awawa awa awa awawawa awawa awawa awa awa awa awawawa awawa awawawa awa awa awawawa awawawa awa awawa awa awawawa awa awa awa awawawa awa awawawa awa awa awa awa awa awa awa awawawa awa awa awa awa awa awa awa awawawa awa awa awa awawa awa awa awawawa awa awa awa awawawa awa awawawa awa awa awawa awa awa awa awawawa awa awa awa awa awawa awa awawawa awa awa awawawa awa awa awawawa awa awa awawa awawa awa awawawa awa awa awa awawawa awa awawawa awa awawa awawa awa awa awawawa awawa awawa awa awa awa awawawa awawa awawawa awa awa awawawa awawawa awa awawa awa awawawa awa awa awa awawawa awa awawawa awa awa awa awa awa awa awa awawawa awa awa awa awa awa awa awa awawawa awa awa awa awa awa awa awa awawawa awawa awa awa awa awa awa awawawa awawawa awawa awa awa awawawa awawawawawawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa awa awa awa awawa";
    commandsList = ReadAwaTalk(codeStr);

    const out = "1o1i_awlaw_aowsay3wa0awa!iJlooHi";
    const replaceCharAt = (str, index, replacement) => str.substr(0, index) + replacement + str.substr(index + 1);
    const dec = [];
    for (let i = 0; i < out.length; i++) {
        const t = replaceCharAt("0".repeat(out.length), i, "a");
        const newI = codeLoop(t, []).indexOf("a");
        if (newI === -1) throw new Error("'a' not found in output");
        dec[i] = out.charAt(newI);
    }
    console.log(dec.join(""));
    // J3lly_0oooosHii11i_awawawawaawa!
}
```