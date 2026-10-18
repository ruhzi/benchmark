# benchmark

A model is given a researcher-voiced description of an experiment, including the baseline numbers, and must predict the numeric outcome of the ablation.

## Files

| file | rows | contents |
|---|---|---|
| `full_corpus.jsonl` | 316 | every sample in the corpus, over 42 papers |
| `eval_set.jsonl` | 42 | the frozen evaluation set: one sample drawn per paper (split seed 20260815) |

Every eval row is also in the corpus; `in_eval_set` on each corpus row marks it.

## Record format

| field | meaning |
|---|---|
| `sample_id` | `<arxiv_id>-s<nnn>` |
| `arxiv_id`, `title` | source paper |
| `prompt` | the question given to the model; baseline values are stated, outcome values are withheld |
| `asked` | the metrics to predict, in order |
| `target` | answer key, positionally aligned with `asked`: `baseline_value` appears in the prompt, `method_value` is the hidden outcome |
| `meta` | component ablated, experimental setting, review flag |
| `in_eval_set` | corpus only: whether the row is in `eval_set.jsonl` |

## Scoring

Per predicted slot, capped relative error:

```
slot_error = min(|pred - true| / max(|true|, 1e-6), 1.0)
```

`true` is `method_value`. A slot the model leaves blank scores 1.0. A sample's score is the mean over its slots, and a model's score is the mean over samples. Lower is better. Provider or harness failures are excluded from the mean rather than penalised.

## Hugging Face

The same two files are published as a dataset at [huggingface.co/datasets/ruhzi/foresightbench](https://huggingface.co/datasets/ruhzi/foresightbench):

```python
from datasets import load_dataset

eval_set = load_dataset("ruhzi/foresightbench", "eval", split="test")
corpus   = load_dataset("ruhzi/foresightbench", "corpus", split="train")
```
