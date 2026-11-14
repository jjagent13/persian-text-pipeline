# Persian Text Pipeline 🇮🇷

Persian text pipeline: **Sanitizer** + **Transformer Spell Corrector** + **REST API**

## Problem

Raw Persian text from various sources (web, OCR, ASR, chat) is full of noise:
- Arabic characters (ك، ي، ة) instead of Persian (ک، ی، ه)
- Extra diacritics, wrong ZWNJ
- Arabic/Persian digits instead of English
- Typos, dropped/swapped characters

## Solution

| Layer | Approach | Output |
|-------|----------|--------|
| 1. Sanitizer | Rule-based (regex + char mapping) | Normalized text |
| 2. Spell Corrector | Custom Transformer (4M params) | Corrected text |
| 3. Confidence Guard | 3 guards (char_sim, word_overlap, length_ratio) | Reject low-quality output |
| 4. REST API | FastAPI + cloudflared tunnel | Public endpoint |

**Design philosophy:** Never corrupt user input. If the model isn't confident, return the original text + reason.

## Project Structure

| Notebook | Description |
|----------|-------------|
| `pars_corrector.ipynb` | First attempt with mT5-small (NaN loss failure) — lesson learned |
| `Persian_Spell_Corrector.ipynb` | Transformer from scratch — caught shared-weights bug |
| `persian_text_pipeline.ipynb` | Final fixed version + sanitizer + API |
| `Text_Sanitizer_+_API.ipynb` | Checkpoint loading + Confidence Guard |

## Architecture
Input → Sanitizer → Tokenizer → Encoder (3 layers) → Decoder (3 layers) → Generator → Guard → Output


## Results

| Metric | Value |
|--------|-------|
| Best val loss | 1.73 |
| Model parameters | 4,061,579 |
| Guard precision (OOD) | 100% rejected (fail-safe) |

> **Honest note:** Trained on 120 seed sentences — this is a proof-of-concept, not SOTA. But the architecture and pipeline are production-ready.

## Usage

```bash
pip install -r requirements.txt
# Then run notebook 03 in Colab (T4 GPU)

Roadmap
See docs/ROADMAP.md.

License
MIT — free to use
