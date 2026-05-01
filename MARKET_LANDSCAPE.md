# Quranic ASR — Market & Open Resource Landscape

A scan of what already exists in the Quranic speech-recognition space —
products, open models, datasets, tooling, and academic work — and an honest
read on where the lane is for this project.

*Last updated: 2026-05-01.*

---

## TL;DR

Two players already do meaningful work in this space, and a third resource wave
arrived in the last ~30 days that significantly changes what a new project can
build:

- **Tarteel** is the established research leader (state-of-the-art ~4% WER on
  Quranic Arabic), but their best models are deliberately closed as core
  business assets. Their open contributions — `whisper-base-ar-quran` and the
  EveryAyah dataset — are valuable baselines, not the full stack.
- **RecitID** is the consumer-facing "Shazam for the Quran" — a polished
  freemium iOS/Android product ($9.99/mo, $59.99/yr) that does Quran
  identification, reciter ID, and live khutbah translation in 53 languages.
  **Cloud-only** — recognition requires internet.
- **The Tadabur dataset** (released April 2026) provides 1,400 hours of
  recitation audio across 600+ reciters with word-level alignments — a 3-5×
  scale jump over prior public corpora. Companion `tadabur-Whisper-Small/Medium`
  models are also published.
- **everyayah-phonemes + Quranic-Phonemizer** (commercial-friendly CC-BY-4.0)
  provide 1,248 hours of *phoneme-aligned* recitation across 45 reciters,
  with Tajweed-aware phoneme sequences. **This effectively solves Phase 1 of
  our roadmap.**

**The open lane this project occupies:** *free, fully-offline, mobile-first,
open-source Quranic captioning that runs without internet or subscription.*
Neither incumbent serves this lane.

---

## Direct competitive products

### RecitID — [recitid.ai](https://recitid.ai/)

The most direct competitor to the v2 vision. Already in market.

| Aspect | Details |
|---|---|
| Core feature | "Shazam for Quran" — identifies surah / ayah / reciter from audio |
| Other features | Live khutbah translation in **53 languages**, AI tafsir, voice-based reciter ID for 200+ reciters, Tajweed Mushaf reader |
| Inference | **Cloud-only** ("Audio detection and Live mode require internet") |
| Identification latency | ~18 seconds |
| Platforms | iOS + Android |
| Business model | Freemium — free tier; **$9.99/month or $59.99/year** for unlimited detection + premium AI features |
| Privacy | Audio "processed in real-time and immediately discarded" (their claim) |
| Tech stack | Not disclosed. Almost certainly cloud Whisper-class. |
| Status | Active in market as of 2026 |

**Strategic read:** RecitID has the consumer brand and a working business
model. They are not architecturally positioned to go fully offline (and have no
business incentive to — their subscription depends on cloud delivery). The
open lane is the offline / free / open positioning.

### Tarteel AI — [tarteel.ai](https://tarteel.ai/) / [github.com/TarteelAI](https://github.com/TarteelAI)

The research leader. Operates the largest open Quranic ASR effort, but
intentionally separates open and closed work.

| Aspect | Details |
|---|---|
| Reported best WER | **~4% WER** on Quranic Arabic (state-of-the-art) |
| Production stack | NVIDIA NeMo + Riva, recently moved to **Wav2Vec2Bert + CTranslate2 + int8 + LitServe** |
| Consumer app | "Tarteel: AI Quran Memorization" — iOS, real-time recitation feedback |
| Open contributions | `tarteel-ai/whisper-base-ar-quran` (5.75% WER, Apache 2.0, 12,970 downloads/month), `tarteel-ai/everyayah` dataset, the **Quranic Universal Library (QUL)** of Quran resources |
| Closed | Best production models, proprietary audio datasets — explicitly held back as *"essential base resources for Tarteel's sustainability"* |

**Strategic reads:**

1. **Tarteel's stack validates the v2 design direction.** Their move to
   Wav2Vec2Bert + int8 + small footprint is exactly what the whitepaper
   proposes. We're not pioneering risky territory — we're following the
   practical leader's path.
2. **The lane Tarteel leaves clear** is real-time + fully-offline + free + open.
   They cannot enter that lane without cannibalising their business.
3. **`tarteel-ai/whisper-base-ar-quran` is the obvious benchmark.** Apache 2.0,
   well-known, used in 57+ community Spaces. Any new model must beat its
   5.75% WER on real-imam audio (not just Quran-corpus audio) to justify
   shipping.

---

## Open pre-trained models

CTC / Wav2Vec2-based (architecture matches our v2 plan):

| Model | Architecture | Reported WER | License | Notes |
|---|---|---|---|---|
| [`Nuwaisir/Quran_speech_recognizer`](https://huggingface.co/Nuwaisir/Quran_speech_recognizer) | Wav2Vec2-XLS-R-53 Arabic, fine-tuned on Quran ASR Challenge | not stated on card | — | Quran-specific fine-tune of XLS-R |
| [`jonatasgrosman/wav2vec2-large-xlsr-53-arabic`](https://huggingface.co/jonatasgrosman/wav2vec2-large-xlsr-53-arabic) | Wav2Vec2-XLS-R-53 | general Arabic baseline | — | Most popular general Arabic Wav2Vec2 fine-tune |
| [`elgeish/wav2vec2-large-xlsr-53-arabic`](https://huggingface.co/elgeish/wav2vec2-large-xlsr-53-arabic) | Wav2Vec2-XLS-R-53, Common Voice | 23.39% on Common Voice Arabic | — | General Arabic; useful as fall-through |
| [`facebook/mms-1b-all`](https://huggingface.co/docs/transformers/model_doc/mms) | MMS-1B, 1107 languages | n/a per-language | CC-BY-NC 4.0 | Broadest voice prior; non-commercial license |
| NeMo Conformer-CTC small | Conformer + CTC, streaming-native | n/a multilingual | Apache 2.0 | Smallest viable; designed for mobile |

Whisper-based (transformer; useful as alignment oracles + benchmarks):

| Model | Base | Reported WER | License | Notes |
|---|---|---|---|---|
| [`tarteel-ai/whisper-base-ar-quran`](https://huggingface.co/tarteel-ai/whisper-base-ar-quran) | Whisper-base | **5.75%** on internal Quranic eval | Apache 2.0 | Most-used open Quran ASR model. **Primary benchmark to beat.** |
| `tadabur-Whisper-Small` | Whisper-small | not stated | published with Tadabur dataset | New (April 2026) |
| `tadabur-whisper-medium` | Whisper-medium | not stated | community variant | New (April 2026) |

**Recommended starting points for the v2 acoustic model:**

1. **Default**: NeMo Conformer-CTC small, fine-tune on `everyayah-phonemes`
2. **Higher accuracy ceiling**: Wav2Vec2-XLS-R Arabic (e.g. Jonatas Grosman's),
   fine-tune on `everyayah-phonemes`
3. **Largest backbone**: MMS-1B-all, fine-tune (heavier, license caveat)

---

## Open datasets

| Dataset | Hours | Reciters | Annotations | License | Released |
|---|---|---|---|---|---|
| [**Tadabur**](https://huggingface.co/datasets/FaisaI/tadabur) | **1,400+** | **600+** | Word-level WhisperX alignments, simplified + Uthmani text, murattal + mujawwad | **CC-BY-NC 4.0** (research only) | April 2026 |
| [**everyayah-phonemes**](https://huggingface.co/datasets/hetchyy/everyayah-phonemes) | 1,248 | 45 | **Tajweed-aware phoneme labels**, Uthmani text, train/dev splits | **CC-BY-4.0** (commercial OK) | recent |
| [`tarteel-ai/everyayah`](https://huggingface.co/datasets/tarteel-ai/everyayah) | — | 26 | Word-level transcriptions, diacritized | varies | mature |
| `Salama1429/tarteel-ai-everyayah-Quran` | — | — | mirrored variant | — | — |
| `Buraaq/quran-audio-text-dataset` | — | 30 | text only | — | — |
| `FaisaI/tadabur` | (same as above) | | | | |
| `Sadique5/arabic_quranic_asr` | small | — | verses + unique words | — | teaching-oriented |

**License watch:** if the project wants downstream collaborators to be able
to ship commercial products built on top, the **primary training corpus
should be `everyayah-phonemes` (CC-BY-4.0)**, not Tadabur (CC-BY-NC-4.0).
Tadabur can be used for ablations and academic comparisons but cannot
contaminate weights destined for commercial use.

---

## Open tooling

| Tool | Purpose | License | Status |
|---|---|---|---|
| [Quranic-Phonemizer](https://github.com/Hetchy/Quranic-Phonemizer) | Open-source Tajweed-aware phonemizer; converts Uthmani text to phoneme sequences with optional Tajweed rules | open | active (used to produce everyayah-phonemes) |
| [quran-align](https://github.com/cpfair/quran-align) | CMU Sphinx-based word-level alignment; <73ms accuracy | MIT | unmaintained since 2016, but functional |
| WhisperX | (used by Tadabur) word-level alignment of Whisper output | BSD-4 | active |
| Kaldi (decoder libs) | Production-grade WFST decoder + tooling | Apache 2.0 | active |
| OpenFst | WFST construction + composition | Apache 2.0 | active |
| ONNX Runtime Mobile | On-device inference engine | MIT | active |

---

## Academic work worth knowing

- **"Bringing Tajweed-Aware Phonemes to Qur'anic Machine [Learning]"** —
  [OpenReview](https://openreview.net/pdf?id=hZt0JK28iV). Directly aligned
  with our approach; describes the phonemizer that powers
  `everyayah-phonemes`.
- **"Quran Recitation Recognition using End-to-End Deep Learning"** — arXiv
  [2305.07034](https://arxiv.org/pdf/2305.07034). Earlier end-to-end approach.
- **"Enhanced Neural Speech Recognition of Quranic Recitations via a Large
  Audio Model"** — [MDPI 2025](https://www.mdpi.com/2076-3417/15/17/9521).
- **"The Tarteel Dataset: Crowd-Sourced and Labeled Quranic Recitation"** —
  [OpenReview](https://openreview.net/pdf?id=TAdzPkgnnV8).
- **"Speech Recognition Models for Holy Quran Recitation Based on Modern
  Approaches and Tajweed Rules: A Comprehensive Overview"** —
  [ResearchGate 2024](https://www.researchgate.net/publication/376960906).
- **"Automatic Pronunciation Error Detection and Correction"** for Quran —
  arXiv [2509.00094](https://www.arxiv.org/pdf/2509.00094).
- **NVIDIA case study on Tarteel** — production ASR architecture insight —
  [nvidia.com](https://www.nvidia.com/en-gb/case-studies/automating-real-time-arabic-speech-recognition/).

---

## Where this project fits

The market shakes out cleanly along three axes:

| | Cloud + Paid | Cloud + Open | **Mobile + Free + Open** |
|---|---|---|---|
| **Identification only** | RecitID free tier | — | (open lane) |
| **Identification + Translation** | RecitID Premium ($60/yr) | — | **← this project** |
| **ASR for memorization / feedback** | Tarteel app | Tarteel open Whisper baseline | (open lane) |

No incumbent serves the **mobile + free + open** column, and the largest
incumbent (Tarteel) cannot enter it without breaking their own business model.
RecitID has the consumer brand and the multi-language translation feature but
is structurally cloud-bound.

The mission framing ("free project to help new Muslims connect to deen")
matches the architectural choice (offline, open, no subscription) — they're
the same thing said two different ways. This is a coherent and defensible
position, not a marketing veneer.

---

## Implications for the v2 roadmap

The whitepaper (v0.2) budgets **8-11 weeks** to a working prototype, with
Phase 1 (data prep / force-alignment) consuming 2-3 weeks. Given what now
exists publicly:

| Phase | Whitepaper estimate | Revised estimate | Why |
|---|---|---|---|
| 1. Data prep / alignment | 2-3 wks | **2-5 days** | `everyayah-phonemes` already exists with Tajweed-aware phoneme labels. Just evaluate quality and possibly augment with a small real-imam set. |
| 2. Acoustic model fine-tune | 1-2 wks | 1-2 wks | Unchanged. |
| 3. WFST construction | 1 wk | 1 wk | Unchanged — bespoke per phoneme inventory. |
| 4. Mobile runtime / ONNX bundling | 2 wks | 2 wks | Unchanged. |
| 5. Integration + confidence routing | 1-2 wks | 1-2 wks | Unchanged. |
| **Total** | **8-11 wks** | **~5-7 wks** | |

The real bottlenecks become (a) the **real-imam evaluation set** (the
methodological non-negotiable from v0.2) and (b) the **WFST + JNI mobile
integration** work. The "neural model training" piece, traditionally the
hardest part, is now the smallest piece.

---

## Recommended next moves

1. **Pull and inspect the new datasets locally.** Sample 20-30 random clips
   from `everyayah-phonemes` and `Tadabur` and verify alignment quality
   against text by ear.
2. **Benchmark `tarteel-ai/whisper-base-ar-quran`** on a small real-imam
   sample (e.g. recordings of ~3-5 distinct local imams reciting Al-Fatihah +
   short surahs). This gives the baseline number any new model has to beat,
   and confirms whether the bottleneck is really the model vs. the audio
   conditions (per the sim2real lesson in whitepaper §5).
3. **Reach out to [Hetchy](https://huggingface.co/hetchyy)**, the creator of
   `everyayah-phonemes` and the Quranic-Phonemizer. Their work is exactly
   what we'd otherwise build, under a compatible license. Natural collaborator.
4. **Decide on a posture toward Tarteel.** They run the most active research
   programme in this space; their open assets (whisper-base-ar-quran +
   EveryAyah + QUL) are a foundation we'd build on. Worth being explicit
   about positioning as *complementary, not competitive*: Tarteel optimises
   memorisation and accuracy at the research frontier; this project optimises
   offline mobile deployment and free public access.
5. **Track RecitID.** It's the sharpest existing test of "is there demand for
   this?" — and the answer is clearly yes (they have a paid product). Their
   feature set telegraphs what users actually want.

---

## How to keep this current

This file should be reviewed every ~3 months. Pace of new dataset / model
releases on HuggingFace is high; the most consequential additions in the
last 90 days (Tadabur, everyayah-phonemes) substantially changed the picture.

To refresh:

1. Search HuggingFace for `quranic asr`, `quran recitation`, `quran phoneme`
2. Check `github.com/TarteelAI` for new releases
3. Check `recitid.ai` (and similar competitor surfaces) for new features /
   architectural disclosure
4. Search arXiv for new "Quranic ASR" / "Tajweed-aware" papers
5. Update this file. Commit.
