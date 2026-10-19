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
| `source_license` | the license the source paper carries on arXiv (see [License](#license)) |
| `prompt` | the question given to the model; baseline values are stated, outcome values are withheld |
| `asked` | the metrics to predict, in order |
| `target` | answer key, positionally aligned with `asked`: `baseline_value` appears in the prompt, `method_value` is the hidden outcome |
| `meta` | `base_id`, `title`, `area` (research-area tag, see below), component ablated, experimental setting, review flag |
| `in_eval_set` | corpus only: whether the row is in `eval_set.jsonl` |

## Research areas

`meta.area` is a per-paper tag, assigned by hand from the paper's title and
abstract and applied to every sample of that paper. It is the area split used
in the paper's per-area results. Six values:

| `area` | meaning | papers | corpus rows | eval rows |
|---|---|---|---|---|
| `vision` | Vision & multimodal | 9 | 101 | 9 |
| `safety-interp` | Safety, attacks, interpretability & auditing | 9 | 61 | 9 |
| `agents` | LLM agents, memory & context systems | 8 | 27 | 8 |
| `applied-sci` | Applied & scientific ML (time series, control, neuro, reliability, recsys) | 8 | 65 | 8 |
| `llm-training` | LLM pre-/post-training & architecture | 5 | 56 | 5 |
| `llm-eval` | LLM evaluation & measurement studies | 3 | 6 | 3 |

## Scoring

Per predicted slot, capped relative error:

```
slot_error = min(|pred - true| / max(|true|, 1e-6), 1.0)
```

`true` is `method_value`. A slot the model leaves blank scores 1.0. A sample's score is the mean over its slots, and a model's score is the mean over samples. Lower is better. Provider or harness failures are excluded from the mean rather than penalised.

## License

ForesightBench has mixed licensing; [LICENSE](LICENSE) has the full terms.

- **The dataset author's contribution** is licensed CC BY 4.0. This covers
  the sample selection and decomposition, the question framing, the schema,
  the `asked` / `target` / `meta` annotations and the area tags.
- **Material from the source papers** stays under the license its authors
  chose on arXiv. Each `prompt` adapts passages of one paper, and `title` is
  that paper's title. The license is recorded per sample in `source_license`:

| `source_license` | papers | corpus rows | eval rows |
|---|---|---|---|
| `cc-by-4.0` | 18 | 143 | 18 |
| `cc-by-sa-4.0` | 2 | 7 | 2 |
| `cc-by-nc-sa-4.0` | 1 | 2 | 1 |
| `cc-by-nc-nd-4.0` | 1 | 1 | 1 |
| `arxiv-nonexclusive-distrib-1.0` | 20 | 163 | 20 |

If you use a sample, you are responsible for respecting its source license.
To keep only material whose source is CC BY 4.0 (with `corpus` loaded as in the Hugging Face section below):

```python
cc_by = corpus.filter(lambda r: r["source_license"] == "cc-by-4.0")
```

Authors of a source paper who want its samples removed can email
ruhaanchopra2005@gmail.com with the arXiv id. When you use ForesightBench,
please also cite the source papers listed in the paper's Appendix D.

## Hugging Face

The same two files are published as a dataset at [huggingface.co/datasets/ruhzi/foresightbench](https://huggingface.co/datasets/ruhzi/foresightbench):

```python
from datasets import load_dataset

eval_set = load_dataset("ruhzi/foresightbench", "eval", split="test")
corpus   = load_dataset("ruhzi/foresightbench", "corpus", split="train")
```
