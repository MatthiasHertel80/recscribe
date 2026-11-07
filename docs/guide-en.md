# Recscribe: User guide (English)

Recscribe backs up recordings from a USB voice recorder, transcribes them locally with speaker
separation, lets you name the speakers by ear, and turns transcripts into minutes or meeting
notes with your own AI key. No cloud account, no app, no Bluetooth. The recorder stays unchanged
unless you explicitly ask Recscribe to remove recordings. No audio leaves your computer; only
transcript text does, and only if you use the AI step.

The complete reference (in German) is the [Handbuch](handbuch.md). This page covers what you need
to get going.

Supported source today: USB recorders that store `.ops` files, tested with an AIREC recorder
(Jieli SoC, 64 GB, USB mass storage). Importing arbitrary audio files is planned.

## Install

Download the package for your system from the
[Releases](https://github.com/MatthiasHertel80/recscribe/releases) page and follow the
`README.txt` inside it.

| System | Package | Status |
|---|---|---|
| macOS 11+ (Apple Silicon, Intel) | `recscribe-<version>-macos-universal.zip` | best tested |
| Linux (x86_64, aarch64) | `recscribe-<version>-linux-<arch>.tar.gz` | automated tests; window needs Vulkan or OpenGL |
| Windows 10/11 (x86_64) | `recscribe-<version>-windows-x86_64.zip` | beta |

The programs are not code-signed. macOS (Gatekeeper) and Windows (SmartScreen) warn on first
start; the `README.txt` shows how to proceed. Check the download against `SHA256SUMS`.

For transcription you also need `uv` and `ffmpeg` (see the README). The first transcription
downloads the speech engine and models (about 2 GB); after that it works offline.

## Use

Open the window (double-click, or `recscribe gui`):

1. **Overview**: plug in the recorder, press *Sync now*. Files are copied, verified with SHA-256
   and listed. A progress bar shows the copy.
2. **Transcribe**: tick recordings, press start. Options: engine (auto, MLX, faster-whisper),
   model, language, a hint text for technical terms, speaker separation, offline mode.
   *Cancel* stops everything, including helper processes.
3. **Transcripts**: read with speaker colours and timestamps, search, copy, open Markdown or SRT.
4. **Speakers**: listen to short samples per speaker and type a name. Names from earlier
   recordings are offered as quick picks.
5. **Processing** (optional): choose a template (minutes, meeting notes, summary, tasks, technical
   notes, or your own), add context, press *Generate*. Needs your own API key.

Everything is also available on the command line:

```sh
recscribe detect                 # show the detected recorder
recscribe sync                   # copy new recordings (--dry-run to preview)
recscribe transcribe             # transcribe everything that has no transcript yet
recscribe speakers               # name the speakers by listening
recscribe ai run --template protokoll   # minutes from a transcript (see recscribe ai --help)
recscribe --help
```

The library defaults to `Aufnahmen/Recscribe` in your home folder (`--dest` or `RECSCRIBE_DEST`
to change it). Status files live in its hidden `.recscribe` folder.

## Removing recordings from the recorder

Off by default. You can remove recordings by hand (`recscribe delete <name>`, `delete --all`, or
the buttons in the window) or after each sync (`recscribe settings --delete-after-sync on`).
A recording is only removed after its local copy has been checked against the stored SHA-256, and
not if it changed in the meantime. macOS and Linux move it to the recorder's trash; Windows has
no trash for removable drives, so removal there is permanent. `--dry-run` shows what would happen.

## AI processing and your key

The program talks directly to the provider you choose: Anthropic, OpenAI-compatible services
(OpenAI, OpenRouter) or a local server (Ollama, LM Studio; then nothing leaves your computer).
With a cloud provider the whole transcript text is sent. `--dry-run --print-prompt` shows exactly
what would be sent. The key is read from an environment variable, the macOS keychain, or a private
file (`~/.config/recscribe/keys` on Linux, `%APPDATA%\recscribe\keys` on Windows). It is never
stored in the library folder.

Models can make mistakes or invent details. Read AI output before you share it, especially
tasks, numbers and dates.

## Privacy

No tracking, no analytics, no update check. Network access: downloading the speech engine and
models on first use, and sending transcript text to your AI provider when you ask for it.

## Notes and limits

- Transcripts and speaker labels are machine output and can be wrong, especially with overlapping
  speech, echo or similar voices. Check them before you rely on them.
- Very long recordings go to the AI in one request (safety limit about 150,000 tokens).
- Speaker names are per recording; there is no automatic voice recognition across recordings yet.
- Record only where you may, and inform people where the law requires it.

Terms of use: see `LICENSE.txt`. Third-party components: `THIRD-PARTY-NOTICES.txt`. Changes:
`CHANGELOG.md`.
