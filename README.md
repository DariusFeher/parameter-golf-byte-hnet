# Byte-Level H-Net Dynamic Chunking for Parameter Golf

This repository includes two non-record submissions I made to OpenAI's [Parameter Golf challenge](https://github.com/openai/parameter-golf), studying byte-level H-Net dynamic chunking under a ≤16 MB artifact constraint.

The first study, comparing byte-level H-Net to subword-level H-Net, was highlighted by OpenAI as one of its three favorite non-record submissions in its retrospective, [*What Parameter Golf taught us*](https://openai.com/index/what-parameter-golf-taught-us/), and was merged into the official Parameter Golf repo: [2026-03-29_HNet_ByteVsSubword_Study](https://github.com/openai/parameter-golf/tree/main/records/track_non_record_16mb/2026-03-29_HNet_ByteVsSubword_Study).

The project has two aims:

- Understand whether H-Net can learn useful chunk structure directly from raw bytes
- Test whether an improved byte-level H-Net can close the gap to a comparable subword-level H-Net and the official Parameter Golf baseline

> **OpenAI Parameter Golf Challenge:** train the best language model that fits in a **16 MB artifact**. The record track requires training in under **10 minutes on 8xH100s**; non-record submissions can explore longer runs or more experimental directions. Models are evaluated by compression on the FineWeb validation set using tokenizer-agnostic **bits per byte (BPB)**.


This work builds on Hwang et al. (2024), [*Dynamic Chunking for End-to-End Hierarchical Sequence Modeling*](https://arxiv.org/abs/2507.07955), adapting H-Net to the Parameter Golf setting.


## Studies

- [01_hnet_byte260_vs_sp1024](studies/01_hnet_byte260_vs_sp1024/README.md): H-Net study comparing `byte260` and `sp1024` H-Net variants and analyzing the learned chunk boundaries, quantitatively and qualitatively.
    > Recognized by OpenAI as one of its three favorite non-record submissions in [*What Parameter Golf taught us*](https://openai.com/index/what-parameter-golf-taught-us/).
    
- [02_improved_hnet_byte260_and_sp1024](studies/02_improved_hnet_byte260_and_sp1024/README.md): a follow-up improved version that reaches **1.2070 BPB** in a 4-hour `byte260` H-Net run, matching the official 4-hour baseline and a comparable `sp1024` H-Net.

## Tokenization Setup

- `byte260`: a byte-level tokenizer with a 260-token vocabulary. The model reads raw bytes directly, so there is no external subword tokenizer.
- `sp1024`: a 1024-vocabulary SentencePiece/BPE tokenizer. The model starts from pre-tokenized subword units instead of raw bytes.

## What This Repo Shows

- Byte-level H-Net can learn whitespace-aligned, word-like chunk boundaries directly from raw bytes ([Study 1](studies/01_hnet_byte260_vs_sp1024/README.md))
-  The initial byte-level H-Net demonstrates that this approach works under the challenge artifact budget (≤16 MB artifact size; [Study 1](studies/01_hnet_byte260_vs_sp1024/README.md))
- A follow-up improved version of byte-level H-Net closes the 4-hour gap to subword-level H-Net and the official competition baseline ([Study 2](studies/02_improved_hnet_byte260_and_sp1024/README.md)).

## Key Results

| Study | Setting | Tokenizer | BPB | Main takeaway |
|---|---|---|---:|---|
| [01_hnet_byte260_vs_sp1024](studies/01_hnet_byte260_vs_sp1024/README.md) | 10 min | byte260 H-Net | 1.4116 ± 0.013 | Learns word-like chunking from raw bytes |
| [01_hnet_byte260_vs_sp1024](studies/01_hnet_byte260_vs_sp1024/README.md) | 4 hours | byte260 H-Net | 1.3595 | Same architecture improves substantially with more optimization, with clear headroom for optimization |
| [01_hnet_byte260_vs_sp1024](studies/01_hnet_byte260_vs_sp1024/README.md) | 10 min | sp1024 H-Net | 1.3734 | Matched byte-vs-subword H-Net comparison |
| [02_improved_hnet_byte260_and_sp1024](studies/02_improved_hnet_byte260_and_sp1024/README.md) | 4 hours | byte260 H-Net | 1.2070 | Matches the official 4-hour baseline |
| [02_improved_hnet_byte260_and_sp1024](studies/02_improved_hnet_byte260_and_sp1024/README.md) | 4 hours | sp1024 H-Net | 1.2107 | Comparable subword-tokenized H-Net run |
||
| Reference baseline | 10 min | official baseline, subword tokenization, no H-Net | 1.2244 | Official record-track starting baseline |
| **Reference baseline** | 4 hours | official baseline, subword tokenization, no H-Net| 1.2074 | Official Parameter Golf baseline, not an H-Net model |

## Key Findings

  - **Byte-level H-Net learns word-like structure:** in the first study, the `byte260` router learns whitespace-aligned chunk boundaries directly from raw bytes, without an external tokenizer.
  - **The improved byte-level H-Net closes the 4-hour gap:** the follow-up `byte260` H-Net reaches **1.2070 BPB**, matching both the official 4-hour baseline (**1.2074**) and a comparable `sp1024` H-Net (**1.2107**).


## Repo Layout

- [studies/01_hnet_byte260_vs_sp1024](studies/01_hnet_byte260_vs_sp1024/README.md): original byte-vs-subword H-Net study, later highlighted by OpenAI in the [retrospective blog post](https://openai.com/index/what-parameter-golf-taught-us/)
- [studies/02_improved_hnet_byte260_and_sp1024](studies/02_improved_hnet_byte260_and_sp1024/README.md): follow-up improved byte-level H-Net, which closes the gap with the official baseline

## Reproduction Setup

Install dependencies:

```bash
python3 -m venv .venv-parameter-golf
source .venv-parameter-golf/bin/activate
pip install -r requirements.txt
```

Prepare the datasets:

```bash
# sp1024 is available from the published cached export.
python3 data/cached_challenge_fineweb.py --variant sp1024

# byte260 is not in the default cached manifest, so export it locally from the
# published document cache.
python3 data/download_hf_docs_and_tokenize.py \
  --output-root ./data/byte260_export \
  --tokenizer-config ./data/tokenizer_specs_byte260.json
```

The reported experiment runs are intended for the Parameter Golf evaluation setting, typically **8xH100**. For exact run commands, see the study READMEs:

- [Study 1 reproduction commands](studies/01_hnet_byte260_vs_sp1024/README.md#reproduction): original `byte260` vs `sp1024` H-Net study
- [Study 2 reproduction commands](studies/02_improved_hnet_byte260_and_sp1024/README.md#reproduction): improved `byte260` H-Net follow-up

----------