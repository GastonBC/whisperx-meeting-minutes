# WhisperX & Gemini Meeting Minutes Automation Pipeline

An end-to-end, multi-part Google Colab notebook workflow designed to automate the lifecycle of raw audio or video recordings into polished, professional meeting minutes.

The pipeline combines precise speech-to-text transcription and speaker diarization via **WhisperX**, multi-model fallback execution using **Google Gemini**, dynamic project management skill loading, customizable domain context via an external `context.md` file, push notifications via **ntfy.sh**, and seamless packaging for local download.

## Complete Pipeline Architecture

### Part 0: Environment Setup (`Dependencies & Kernel`)
* **Clean Dependency Installation:** Forces compatible versions of PyTorch (`2.8.0`), TorchAudio, TorchVision, and WhisperX (`3.8.5`) to prevent runtime crashes.
* **Pandas/Numpy Sync:** Upgrades and syncs pandas and numpy to eliminate circular import errors stemming from Colab's default C-extensions.
* **Automatic Kernel Restart:** Programmatically kills and restarts the Python kernel post-installation so the environment loads fresh packages cleanly.

### Part 1: Transcription & Diarization (`WhisperX`)
* **FFmpeg Audio Normalization:** Automatically converts input formats (`.mp4`, `.m4a`, `.mp3`, `.wav`, `.mkv`, `.webm`) into lightweight **32kbps mono OGG** files (`16kHz`) for optimized processing.
* **VRAM Safeguards:** Executes explicit garbage collection (`gc.collect()`) and PyTorch CUDA cache clearing (`torch.cuda.empty_cache()`) between pipeline stages (transcription $\rightarrow$ alignment $\rightarrow$ diarization) to prevent Out-Of-Memory (OOM) errors.
* **Idempotent Batch Processing:** Scans the active workspace directory, automatically skipping media files that already have matching `.txt` transcripts.
* **Speaker-Grouped Output:** Flattens word-level alignments into structured dialogue blocks grouped cleanly by speaker turns (`SPEAKER_00: text...`).

### Part 2: AI Meeting Minutes Generation (`Google Gemini`)
* **Dynamic Skill Prompting:** Pulls the latest project management meeting analyzer framework dynamically from GitHub (`SKILL.md`).
* **Custom Context Injection (`context.md`):** Reads project-specific rules, entity spellings, or domain constraints directly from a local `context.md` file, keeping the source code clean and generic.
* **Model Fallback Chain:** Automatically attempts generation with prioritized models (e.g., `gemini-3.8-flash`, falling back to `gemini-3.7-flash`) with built-in retry timers.
* **Structured Deliverables:** Generates comprehensive markdown minutes (`_MINUTES_*.md`) complete with initial metadata, topic-by-topic overviews, structured Q&A, and clear action items.

## Required Colab Secrets

Before running the notebook cells, configure the following secrets in your Google Colab **Secrets (key icon)** panel:

1. **`HF_TOKEN`**: Your Hugging Face user access token (required for `pyannote-audio` speaker diarization; ensure you have accepted the model terms on Hugging Face).
2. **`GEMINI_API_KEY`**: Your Google GenAI API key.

## Customizing Domain Context (`context.md`)

To adapt the minutes generator to your specific industry or project (e.g., architecture, software development, legal), create a file named `context.md` in your working directory.

Example `context.md`:
```
Act as a technical writer specialized in construction and architectural coordination meetings.

Common company or technical terms that might be mistranscribed: (examples)
```

## Step-by-Step Usage Guide

0. **Run Part 0:** Installs compatible environment dependencies (`torch`, `whisperx`, `pandas`, `numpy`) and automatically restarts the Google Colab kernel.
1. **Upload Media & Context:** Upload your meeting recordings (`.mp4`, `.mp3`, `.wav`, etc.) and optional `context.md` file into your Google Colab workspace.
2. **Run Part 1:** Converts media, transcribes speech via WhisperX, aligns timestamps, diarizes speakers, and saves text logs.
3. **Run Part 2:** Pulls the analysis skill, reads `context.md`, sends transcripts to Google Gemini, and creates formatted markdown minutes.
4. **Download:** Package and download your processed files using standard Colab archive utilities.

> This project is personal and vibe coded
