---
created: 2025-03-22T23:54
updated: 2025-04-19T08:07
points: 975
---

My first attempt was to use a evolution like algorithm to find the original text, the model I used was `SentenceTransformer('sentence-transformers/gtr-t5-base')`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1742702140/2025/03/cb531dce84f212dfa080d735717cada3.png)

`0.1707 Text: 'dyneysoof dlo bcema'`
`0.1671 Text: 'bde mfia '`

Many attempts and modifications were made but the similarity never hit 0.2.

Then after some research I found the word2vec and vec2text.

[vec2text/vec2text: utilities for decoding deep representations (like sentence embeddings) back to text](https://github.com/vec2text/vec2text)

```python
import vec2text
import torch
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

corrector = vec2text.load_pretrained_corrector("gtr-base")
ground_truth = np.load('gtr_embeddings.npy')
vec2text.invert_embeddings(
  embeddings=torch.from_numpy(ground_truth).cuda(),
  corrector=corrector,
  num_steps=10,
)
# ['            The secret terminalphrase is terminalin']
```

The result is promising! It is semantically saying `the secret password is ...`.

```python
from transformers import AutoModel, AutoTokenizer, PreTrainedTokenizer, PreTrainedModel
def get_gtr_embeddings(text_list,
                       encoder: PreTrainedModel,
                       tokenizer: PreTrainedTokenizer) -> torch.Tensor:
    inputs = tokenizer(text_list,
                       return_tensors="pt",
                       max_length=128,
                       truncation=True,
                       padding="max_length",)
    with torch.no_grad():
        model_output = encoder(input_ids=inputs['input_ids'], attention_mask=inputs['attention_mask'])
        hidden_state = model_output.last_hidden_state
        embeddings = vec2text.models.model_utils.mean_pool(hidden_state, inputs['attention_mask'])
    return embeddings

corrector = vec2text.load_pretrained_corrector("gtr-base")
encoder = AutoModel.from_pretrained("sentence-transformers/gtr-t5-base").encoder
tokenizer = AutoTokenizer.from_pretrained("sentence-transformers/gtr-t5-base")

embeddings = get_gtr_embeddings([
       '            The secret terminalphrase is terminalin'
], encoder, tokenizer)
cosine_similarity(embeddings, ground_truth)
# array([[0.8059379]], dtype=float32)
```

They are very similar too!

However `terminalin` is not correct.

Let's see how `num_steps` affects the result.

```python
for i in range(10):
  print(i, vec2text.invert_embeddings(
      embeddings=torch.from_numpy(ground_truth).cuda(),
      corrector=corrector,
      num_steps=i+1,
  ))
```

```
0 ['           secret terminalphrase is passinit']
1 ['            The secret terminalphrase is terminalin']
2 ['           The secret terminalphrase is init']
3 ['            The secret terminalphrase is terminalin']
4 ['           The secret terminalphrase is init']
5 ['            The secret terminalphrase is terminalin']
6 ['           The secret terminalphrase is init']
7 ['            The secret terminalphrase is terminalin']
8 ['           The secret terminalphrase is init']
9 ['            The secret terminalphrase is terminalin']
```

Interesting...

I got stuck here but my teammate @RJCyber managed to find it.

The password was `terminalinit` lmao.

```flag
HTB{AI_S3cr3ts_Unve1l3d}
```
