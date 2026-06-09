# betterthantoon.com

**GCF is better than TOON on every dimension.** Their datasets, their tokenizer, their methodology. Full benchmark data proving it.

## Why TOON fails

TOON claims ~40% token savings over JSON on tabular data. What they don't measure: whether LLMs can actually read it at scale, or produce valid output in it.

- **Comprehension**: TOON averages 68.5% at 500 records. GCF averages 90.7%.
- **Generation**: TOON's own decoder rejects LLM-generated output on 7 of 9 models. The error is always the same: `toon: cannot assign string to int`. The model writes "target" where TOON expects the integer `0`.
- **Token efficiency**: GCF wins all 6 datasets on TOON's own benchmark.

TOON's flat tabular design forces two failure modes that GCF eliminates structurally:

1. **Distance grouping failure**: without section headers, models must scan 500 rows and filter by column value. They guess round numbers instead. TOON median error: 53.
2. **Semantic-to-integer encoding**: TOON encodes categories as integers with no structural cue. Models can't perform this mapping unprompted.

## Token efficiency (TOON's own benchmark)

Using TOON's benchmark code, TOON's tokenizer (o200k_base), and TOON's datasets:

| Dataset | GCF | TOON | Result |
|:---|:---|:---|:---|
| Semi-uniform event logs | **108,158** | 154,032 | **GCF 42% smaller** |
| E-commerce orders | **61,593** | 73,246 | **GCF 19% smaller** |
| Deeply nested config | **616** | 618 | **GCF 0.3% smaller** |
| Employee records | **49,055** | 49,966 | **GCF 2% smaller** |
| Analytics time-series | **8,398** | 9,127 | **GCF 8% smaller** |
| GitHub repos | **8,576** | 8,744 | **GCF 2% smaller** |

GCF wins all 6. Including the flat tabular datasets TOON was designed for.

## Comprehension (can the model read it?)

500 symbols, 200 edges, 13 extraction questions. No format instructions. No system prompt.

| Model | GCF | TOON | JSON |
|:---|:---|:---|:---|
| Claude Opus 4.6 | **96.2%** | 84.6% | 73.1% |
| Claude Sonnet 4.6 | **100%** | 73.1% | 53.8% |
| Claude Haiku 4.5 | **96.2%** | 69.2% | 57.7% |
| GPT-5.5 | **84.1%** | 67.7% | 45.8% |
| GPT-5.4 | **78.0%** | 56.0% | 44.1% |
| GPT-5.4-mini | **71.8%** | 64.1% | 54.2% |
| Gemini 2.5 Pro | **100%** | 76.9% | 58.3% |
| Gemini 3.1 Pro | **100%** | 76.9% | 46.2% |
| Gemini 3.5 Flash | **100%** | 61.5% | 46.2% |
| Gemini 2.5 Flash | **80.6%** | 54.6% | 57.0% |

23 runs, 10 models, 3 providers. GCF wins 22, ties 1, loses 0. Four models hit 100%.

## Generation (can the model write it?)

3-line primer, validated through real decoders. Same data, same prompt structure.

| Model | GCF | TOON | JSON |
|:---|:---|:---|:---|
| Claude Opus 4.6 | **5/5** | 0/5 | 5/5 |
| Claude Sonnet 4.6 | **5/5** | 2-3/5 | 5/5 |
| Claude Haiku 4.5 | **5/5** | 1-3/5 | 5/5 |
| GPT-5.5 | **4-5/5** | 1-2/5 | 5/5 |
| GPT-5.4 | **5/5** | 0/5 | 5/5 |
| GPT-5.4-mini | **5/5** | 0/5 | 5/5 |
| Gemini 2.5 Pro | **5/5** | 1/5 | 5/5 |
| Gemini 3.1 Pro | **5/5** | 0/5 | 5/5 |

No model has ever been trained on GCF. Every frontier model produces valid output on first exposure. TOON has been published for months and fails on 7 of 9 models.

## The TOON generation failure

Every TOON generation failure produces the same error:

```
INVALID: symbols: index 0: distance: toon: cannot assign string to int
```

The model writes `target` in the distance column. TOON expects `0`. The model would need to know, unprompted, that "target" maps to 0, "related" maps to 1, "extended" maps to 2. No model does this because the format gives no structural cue.

GCF expresses distance through section placement: targets go in `## targets`, related in `## related`. No integer mapping required.

Even when TOON is hand-held (pre-encoded integers in the prompt), GCF output is still 28% smaller.

## What GCF has that TOON doesn't

- **Session deduplication**: symbols from prior calls become bare references. 92.7% savings by the 5th call. TOON re-serializes everything every time.
- **Delta encoding**: 81.2% additional savings on re-queries. TOON has no equivalent.
- **Hierarchical grouping**: section headers eliminate the distance-filtering failures that plague TOON at scale.
- **Local IDs**: edges cost ~4 tokens each regardless of identifier length. TOON repeats full qualified names.
- **Streaming**: zero-buffering with trailer summary. TOON has no streaming support.

## We opened a PR on TOON's repo

We added GCF as a formatter to TOON's official benchmark. 9 lines of code. The PR is public:

[toon-format/toon#319](https://github.com/toon-format/toon/pull/319)

The fork with full results:

[blackwell-systems/toon](https://github.com/blackwell-systems/toon)

## How to try GCF

```bash
pip install gcf-proxy
```

Wrap any MCP server. Zero code changes. Or use the libraries directly:

```bash
pip install gcf-python        # Python
npm install @blackwell-systems/gcf  # TypeScript
go get github.com/blackwell-systems/gcf-go  # Go
cargo add gcf                 # Rust
```

## Links

- [Live site](https://betterthantoon.com)
- [Full benchmarks](https://gcformat.com/guide/benchmarks.html)
- [GCF vs TOON (detailed)](https://gcformat.com/guide/vs-toon.html)
- [Full eval data](https://gcformat.com/guide/eval-results.html)
- [GCF Specification](https://github.com/blackwell-systems/gcf)
- [GCF Proxy](https://github.com/blackwell-systems/gcf-proxy)
- [Playground (live three-way comparison)](https://gcformat.com/playground.html)
- [Whitepaper (DOI: 10.5281/zenodo.20579817)](https://doi.org/10.5281/zenodo.20579817)
- [TOON benchmark fork](https://github.com/blackwell-systems/toon)

## License

MIT - [Dayna Blackwell](https://github.com/blackwell-systems)
