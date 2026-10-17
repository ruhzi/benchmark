# benchmark

The 42-item evaluation set: one sample drawn per paper (split seed 20260815) from a corpus of 316 ablation samples extracted from arXiv papers.

Each line of `benchmark.jsonl` is one sample:

| field | meaning |
|---|---|
| `sample_id` | `<arxiv_id>-s<nnn>` |
| `arxiv_id`, `title` | source paper |
| `prompt` | the researcher-voiced question given to the model (baseline values are stated; outcome values are withheld) |
| `asked` | the metrics the model must predict, in order |
| `target` | answer key, positionally aligned with `asked`: `baseline_value` is in the prompt, `method_value` is the hidden outcome |
| `meta` | component ablated, experimental setting, review flag |

Scoring: capped relative error per metric, cap 1.0, averaged over metrics then over samples.
