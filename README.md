# Thai Call Center ASR

Colab-first Thai speech-to-text pipeline for the Super AI Engineer SS6 Individual Test: Thai Call Center ASR.

This repository is designed as both a competition workflow and an interview-ready portfolio project. It handles Thai banking call-center audio with phone compression, radio/static-like noise, fast/slow speech, pitch shifts, and singleton audio files.

## Why Notebook-First

The main workflow lives in one self-contained notebook:

- `Thai_Call_Center_ASR_Colab.ipynb`

All helper functions are inside the notebook so errors can be fixed cell-by-cell in Colab without uploading or importing new project files. The repo intentionally keeps data, model weights, checkpoints, submissions, and API keys out of Git.

## Open in Colab

Public repo URL:

```text
https://github.com/Up2mEz/Thai-call-center-ASR
```

Direct Colab URL after pushing:

```text
https://colab.research.google.com/github/Up2mEz/Thai-call-center-ASR/blob/main/Thai_Call_Center_ASR_Colab.ipynb
```

Clone in Colab:

```python
!git clone https://github.com/Up2mEz/Thai-call-center-ASR.git
%cd Thai-call-center-ASR
```

The notebook installs its own dependencies, so `requirements_colab.txt` is only a reference.

## Data Setup

Upload one of these to Google Drive:

```text
/content/drive/MyDrive/Thai-call-center-ASR/audio_final/
```

or:

```text
/content/drive/MyDrive/Thai-call-center-ASR/individual-test-thai-call-center-asr.zip
```

Expected unzipped layout:

```text
audio_final/
  sample_submission.csv
  audio/
    RSP_101_audio.wav
    RSP_101_audio_noise.wav
    ...
```

The notebook always uses `sample_submission.csv` as the source of truth. In the current local audit, the sample has 6,261 rows, including 975 complete 6-variant groups and 411 singleton `AU_*` files.

## API Key

OpenAI is optional but recommended for a strong baseline. Save your key in Colab Secrets as:

```text
OPENAI_API_KEY
```

The notebook also supports manual runtime input with `getpass`. Never commit API keys to GitHub.

## Pipeline

1. Mount Drive and locate dataset.
2. Audit filenames, variants, singleton files, and durations.
3. Run a small pilot on 50 complete groups and 25 singleton files.
4. Run OpenAI `gpt-4o-transcribe` on original + slow variants and singleton files.
5. Run `faster-whisper` with `large-v3-turbo`, falling back to `medium` if GPU memory is limited.
6. Optionally run hard variants: `phone`, `noise`, `fast`, plus a conservative noise-preprocessed candidate.
7. Normalize Thai text conservatively.
8. Choose a transcript per group using weighted Levenshtein consensus.
9. Generate submission CSVs while preserving exact sample row order.

## Outputs

Saved under Google Drive:

```text
/content/drive/MyDrive/Thai-call-center-ASR/work/
  checkpoints/
    openai_*.csv
    whisper_*.csv
  submissions/
    submission_keep_space.csv
    submission_no_punct_keep_space.csv
    submission_no_space_probe.csv
  audit_summary.csv
  experiment_candidates.csv
  group_consensus_debug.csv
```

## Model Tradeoffs

- OpenAI API: strong robustness and prompt control, but has cost and may clean filler words too aggressively.
- Faster-Whisper: open-weight, reproducible, cheap after setup, and uses the Whisper large-v3-turbo family through CTranslate2, but can struggle on Thai spacing and noisy/phone audio.
- Group consensus: uses the competition insight that original, phone, noise, fast, slow, and pitch variants share the same transcript.
- Singleton fallback: `AU_*` files have no augmentation group, so they are scored independently with the same candidate-quality rules.

## Noise Handling

The `_noise.wav` files can contain radio/static-like noise. The notebook treats noise-preprocessed transcription as an extra candidate, not an automatic replacement, so it cannot overwrite a stronger raw-audio result by accident.

## Checkpointing and Recovery

Every transcription pass writes checkpoint rows with:

```text
file_name, group_id, variant, model, prompt_version, audio_source,
text_raw, text_norm, duration_sec, runtime_sec, status, error, created_at
```

Rerunning a cell resumes from the checkpoint and skips completed files. OpenAI calls retry up to three times and continue after repeated failures.

## Interview Talking Points

- Designed an end-to-end Thai ASR pipeline for noisy banking call-center audio.
- Compared API-based ASR and open-weight Whisper models under cost and latency constraints.
- Built robust checkpointing for unreliable Colab sessions.
- Implemented conservative Thai normalization to preserve filler words like `อืม`, `เอ่อ`, `ค่ะ`, and `ครับ`.
- Used grouped-variant consensus to improve robustness without training labels.
- Prepared a production-minded serving demo cell with FastAPI while keeping competition inference reproducible.

## Git Notes

Do not commit:

- `audio_final/`
- zip datasets
- checkpoints
- submissions
- model weights
- `.env` files or API keys
