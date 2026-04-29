# quran-model

Architectural design and reference whitepaper for a real-time, on-device,
pre-transformer ASR system specialised for Quranic Arabic recitation —
intended as a complementary track to general-purpose ASR (e.g. Whisper) for
applications that need to recognise Quranic recitation cleanly, with
near-perfect accuracy and sub-100&nbsp;ms latency.

---

## About this project

This is a personal endeavor to help create a new class of applications that
advance new Muslims' understanding of and connection to the deen.

The project is **completely free and open source**. Collaborators are warmly
welcomed — design feedback, data contributions, code, translations, reciter
recordings, or just sharing it with someone who might build on it.

---

## What's in this repo

- **[QuranicASR_Architecture_Whitepaper.pdf](QuranicASR_Architecture_Whitepaper.pdf)** —
  10-page whitepaper covering:
  - The architectural mismatch between transformer ASR (Whisper-class) and
    Quranic recitation as a problem domain
  - The constraint structure of the Quran as a closed corpus and why it
    enables a fundamentally different solution
  - A CTC + WFST hybrid pipeline (the "Kaldi-style" path) for real-time
    Quranic transcription
  - System architecture, memory/latency budget, phased implementation
    roadmap, risk register, and competitive/IP positioning

---

## TL;DR of the thesis

Whisper-class transformers are designed for **open-vocabulary** general
speech. The Quran is a **closed-vocabulary** problem (6,236 fixed ayat,
regular tajwid prosody, narrow style space). Replacing the transformer with
a small CTC acoustic model + a WFST decoder over Quranic-only paths gives:

- **Sub-100 ms** end-to-end latency (vs ~5–7 s for Whisper-base on phone)
- **~35 MB** on-disk footprint (vs 142 MB for Whisper-base)
- **Structural impossibility** of hallucinating non-canonical text — the
  decoder graph literally has no edges leading anywhere outside the Quran

This is a return to a pre-transformer architecture that is *correct for
this specific problem*, not a regression.

---

## How to contribute

This repo currently holds the design artefact only. Prototype implementation
work has not yet started. If you'd like to help:

- **Read the whitepaper** and open an issue with feedback, gaps, or
  alternative approaches you'd consider
- **Curate or contribute audio data** — phoneme-aligned Quranic recitations
  across reciters and qira'at variants are the largest moat to building this
- **Tooling experience** — Kaldi, OpenFst, NeMo, mobile ASR (TFLite/NCNN)
  all directly applicable
- **Translations** — verified canonical English/other-language translations
  with permissive licensing

Reach out via GitHub issues on this repo.

---

## License

The whitepaper and all design artefacts in this repo are released for free
public use. Implementation code (when added) will follow a permissive open
source license (likely MIT or Apache-2.0).
