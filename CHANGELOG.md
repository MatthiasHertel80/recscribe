# Changelog

## 0.5.0 (2026-11-07), first public release

Recscribe backs up recordings from a USB voice recorder, transcribes them locally with speaker
separation, lets you name the speakers by listening to short samples, and turns transcripts into
minutes or meeting notes with your own AI key. It is freeware (see `LICENSE.txt`).

**Platforms:** macOS (Apple Silicon and Intel), Linux (x86_64 and aarch64), Windows (x86_64).
macOS on Apple Silicon is the most tested platform. Linux was tested with the automated test
suite. Windows is a beta: it builds and passes the platform checks, but has had little time on
real machines. The binaries are not code-signed (see `README.txt`).

### Recorder and library
- Detects the recorder by its structure (`RECORD` folder, `SET.bin`), not by its name.
- Copies `.ops` recordings with checksum verification, resumes safely after an interruption,
  and keeps a manifest of every file.
- Converts `.ops` to Ogg Opus without re-encoding (lossless repackaging).
- Optionally removes recordings from the recorder, by hand or after a verified sync. A recording
  is only removed after its local copy was checked against the stored SHA-256. macOS and Linux
  move it to the recorder's trash by default; Windows has no trash on removable drives, so
  removal there is permanent.

### Transcription and speakers
- Local speech recognition: Whisper large-v3-turbo via MLX on Apple Silicon, faster-whisper
  elsewhere. Local speaker separation (speakrs). Nothing is uploaded; models are downloaded once.
- Transcripts as Markdown, SRT and JSON, with speaker colours, timestamps and search in the
  reader.
- Name speakers from short audio samples; names carry over as suggestions to later recordings.
- A hint text for technical terms improves recognition.

### AI post-processing (optional, bring your own key)
- Minutes, meeting notes, summary, task list, technical notes, or your own templates.
- Anthropic, OpenAI-compatible services (OpenAI, OpenRouter) and local servers (Ollama,
  LM Studio). The key is kept in the macOS keychain or in a private file, never in the library.

### Interface
- Native window in English and German (follows the system language), command line for everything.
- Progress bars for syncing and transcribing, cancel that also stops the helper processes,
  stale locks from an aborted run are recovered automatically.
- macOS: `recscribe install-app` installs `Recscribe.app`; `install-agent` syncs when the
  recorder is plugged in.
