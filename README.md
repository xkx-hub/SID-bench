# Semantic Interruption Detection Benchmark

A multilingual benchmark for evaluating **semantic interruption detection** in conversational speech. The dataset contains annotated audio recordings in English and Mandarin Chinese, along with a silence/noise negative set, designed to test models that decide *when* (and *whether*) to interrupt a speaker.

---

## Overview

In spoken dialogue systems and voice assistants, knowing when to interrupt a speaker — and detecting interruption signals in real time — is a critical yet under-studied task. This benchmark provides:

- **3,700 audio clips** with frame-level interruption timestamps
- **Bilingual coverage**: English and Mandarin Chinese conversational speech
- **Negative samples**: 500 silence and environmental noise clips for robustness evaluation
- **Three standardized metrics**: FIR, IRL, and APT (defined below)

---

## Dataset Statistics

| Split | Language | Total Samples | With Interruption | Without Interruption |
|-------|----------|:-------------:|:-----------------:|:--------------------:|
| `en_test_lines.jsonl` | English | 1,600 | 1,100 (68.8%) | 500 (31.3%) |
| `zh_test_lines.jsonl` | Chinese | 1,600 | 1,100 (68.8%) | 500 (31.3%) |
| `silence_noise_test.jsonl` | — | 500 | 0 (0%) | 500 (100%) |
| **Total** | | **3,700** | **2,200** | **1,500** |

**Audio duration:**
- English: 0.36s – 22.0s (avg 3.74s)
- Chinese: similar range (avg 3.85s)
- Silence/noise: 0.09s – 182.0s (avg 13.0s)

---

## Directory Structure

```
Semantic_Interaption/
├── README.md
├── test.ipynb                        # Evaluation notebook
└── test_wavs/
    ├── en_test_lines.jsonl           # English ground-truth annotations
    ├── zh_test_lines.jsonl           # Chinese ground-truth annotations
    ├── silence_noise_test.jsonl      # Silence / noise annotations
    ├── en/                           # English WAV files (1,600)
    ├── zh/                           # Chinese WAV files (1,600)
    └── silence_or_noise/             # Silence / noise WAV files (500)
```

---

## Annotation Format

Each `.jsonl` file contains one JSON object per line.

### Speech splits (`en_test_lines.jsonl`, `zh_test_lines.jsonl`)

```json
{
  "audio":           "MDT_ASR_AI106_A0016_S0005_0_G3265_0318890-0324670.wav",
  "total_nonbreak":  false,
  "duration":        5.78,
  "break_time":      0.125,
  "text_with_break": "<break> Experiencing different cultures, learning new languages, ..."
}
```

| Field | Type | Description |
|-------|------|-------------|
| `audio` | `str` | Filename of the corresponding WAV file |
| `total_nonbreak` | `bool` | `true` if no interruption occurs in the entire clip |
| `duration` | `float` | Total audio duration in seconds |
| `break_time` | `float` | Timestamp (seconds) of the first interruption signal; `-1` when `total_nonbreak=true` |
| `text_with_break` | `str` | Transcript with `<break>` token inserted at the interruption point |

### Silence / noise split (`silence_noise_test.jsonl`)

All samples have `total_nonbreak=true` and `text_with_break=null`. These clips contain no speech and serve as a robustness / false-alarm test.

```json
{
  "audio":           "wind_wind_heavy_clean_stdy_air.wav",
  "total_nonbreak":  true,
  "duration":        152.52,
  "break_time":      152.52,
  "text_with_break": null
}
```

---

## Evaluation Metrics

The benchmark defines three complementary metrics. Reference implementation is in `test.ipynb`.

### FIR — False Interruption Rate

The fraction of non-interruption samples for which the model incorrectly predicts an interruption.

```
FIR = |false positives| / |total_nonbreak samples|
```

Lower is better.

### IRL — Interruption Response Latency

Average absolute timing error (in seconds) over true-positive detections (model correctly identifies a break and the timestamp is within 50 ms of ground truth).

```
IRL = mean(|predicted_break_time - gt_break_time|)   [over true positives]
```

Lower is better.

### APT — Average Penalty Time

A unified penalty that captures both detection errors and timing inaccuracies:

| Prediction vs. Ground Truth | Penalty |
|-----------------------------|---------|
| Both predict no break | 0 |
| Predicted time within ±50 ms of ground truth | 0 |
| False alarm (predict break, no GT break) | Full clip duration |
| Missed detection (no prediction, GT has break) | `duration − break_time` |
| Late detection | `predicted_time − break_time` |

```
APT = mean(penalty)   [over all samples]
```

Lower is better.

---

## Download

The full dataset (audio + annotations) is hosted on Hugging Face:

**[https://huggingface.co/datasets/kxxia/SID-bench](https://huggingface.co/datasets/kxxia/SID-bench)**

```bash
# Option 1: huggingface_hub CLI
huggingface-cli download kxxia/SID-bench --repo-type dataset --local-dir ./SID-bench

# Option 2: Python
from huggingface_hub import snapshot_download
snapshot_download(repo_id="kxxia/SID-bench", repo_type="dataset", local_dir="./SID-bench")
```

---

## Quick Start

```python
import json

# Load ground-truth annotations
with open("test_wavs/en_test_lines.jsonl") as f:
    en_samples = [json.loads(line) for line in f]

# Each sample
sample = en_samples[0]
print(sample["audio"])           # WAV filename
print(sample["total_nonbreak"])  # bool
print(sample["break_time"])      # float (seconds)
print(sample["text_with_break"]) # transcript with <break> marker
```

Run `test.ipynb` to reproduce the evaluation pipeline and compute FIR / IRL / APT against your own model predictions.

### Prediction format

Your model should produce a JSONL file with one prediction per line:

```json
{"audio": "MDT_ASR_AI106_A0016_S0005_0_G3265_0318890-0324670.wav", "total_nonbreak": false, "break_time": 0.13}
```

---

## Use Cases

- **Conversational AI / dialogue systems**: training models to detect natural turn-taking signals
- **Voice assistants**: knowing when a user intends to interrupt or be interrupted
- **Speech-to-speech translation**: preserving interruption timing across languages
- **Contact center analytics**: segmenting overlapping speech

---

## License

Please check the `LICENSE` file for usage terms. The audio data may originate from multiple source corpora; refer to their respective licenses before use in commercial applications.

---

## Acknowledgement

The English conversational speech data used in this benchmark were derived from the **English Duplex Conversation Training Dataset** provided by MagicData.

> MagicData. (2025). *MDT-AF069 English Duplex Conversation Training Dataset*.
>https://www.magicdatatech.com/datasets/asr/mdt-af069-multi-stream-spontaneous-conversation-training-datasets-english-1733294791

---

## Citation

If you use this benchmark in your research, please cite:

```bibtex
@misc{semantic_interruption_benchmark,
      title={Semantic-Aware Interruption Detection in Spoken Dialogue Systems: Benchmark, Metric, and Model}, 
      author={Kangxiang Xia and Bingshen Mu and Xian Shi and Jin Xu and Lei Xie},
      year={2026},
      eprint={2603.24144},
      archivePrefix={arXiv},
      primaryClass={cs.SD},
      url={https://arxiv.org/abs/2603.24144}, 
}
```