---
name: voice-transcription
description: Configure reliable voice transcription.
version: 0.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [voice, stt, transcription, whisper, telegram, groq]
    related_skills: []
---

# Voice Transcription Skill

Use this skill when a user needs to configure, debug, or improve speech-to-text transcription for voice messages or audio notes. It focuses on reliable language selection, provider fallback, safe handling of secrets, and verification from real transcripts.

## When to Use

- Voice messages are transcribed in the wrong language.
- Transcripts look like broken English from non-English speech.
- A bot or assistant needs local or cloud STT configured.
- A Telegram-style voice-message relay needs safe scope and verification.
- A user wants Groq, OpenAI, Whisper, or local `faster-whisper` transcription guidance.

Don't use this for live call recording, covert transcription, or publishing private audio. Get explicit authorization before transcribing third-party conversations.

## Prerequisites

- A chosen STT provider:
  - local Whisper / `faster-whisper` for private fallback;
  - Groq Whisper models for fast low-cost transcription;
  - OpenAI transcription models for paid fallback.
- `ffmpeg` available when converting audio formats.
- API keys stored only in environment variables or a secure secret store. Never paste keys into chat, command history, source files, or public repositories.

Recommended environment variable names:

```text
GROQ_API_KEY
OPENAI_API_KEY
STT_PROVIDER=auto|local|groq|openai
STT_LANGUAGE=ru|en|auto
STT_MODEL=whisper-large-v3-turbo|whisper-large-v3|gpt-4o-mini-transcribe|small
```

## Provider Choice

1. **Start with language.** If Russian, Spanish, Thai, or another known language is expected, set it explicitly. A wrong `language: en` setting can make otherwise good models produce nonsense.
2. **Use local STT as the safe baseline.** Local Whisper avoids external API exposure, but can be slower and less accurate on small servers.
3. **Use Groq for fast Telegram-style replies.** `whisper-large-v3-turbo` is a good speed/quality tradeoff; `whisper-large-v3` can be better when latency matters less.
4. **Use OpenAI as fallback.** `gpt-4o-mini-transcribe` is the cost-conscious option; a larger model can be used when quality is more important than cost.
5. **Prefer `auto` mode.** Try the configured cloud provider first, then fall back to local transcription on provider errors so voice features keep working.

## Procedure

1. **Inspect current settings.** Check whether STT is enabled, which provider/model is selected, and whether `STT_LANGUAGE` is explicit. Completion: provider, model, language, and credential presence are known without exposing secret values.
2. **Verify audio conversion.** Confirm the input audio can be read or converted with `ffmpeg`. Completion: a short sample file converts to a standard format such as `.wav` or `.mp3`.
3. **Run one controlled transcript.** Transcribe a known sample and check language, punctuation, speaker intent, and truncation. Completion: transcript quality is accepted or the failure mode is named.
4. **Add formatting guidance when supported.** For Whisper-compatible APIs, pass a prompt asking for readable punctuation, sentence breaks, and the expected language. Completion: transcripts are readable enough for downstream automation.
5. **Bound the output.** Long audio should produce a concise transcript, a summary, or chunked output by explicit design. Completion: no automatic workflow can spam a chat with unbounded messages.
6. **Add fallback behavior.** If cloud STT fails, retry locally or report a clear provider error. Completion: one provider outage does not silently disable all voice handling.
7. **Verify in the exact target channel.** If the workflow replies in Telegram or another messenger, read back the sent reply/message ID. Completion: the transcript appears where the user expects it, not only in logs.

## Telegram Voice Relay Pattern

Use this pattern only when the user explicitly wants incoming voice messages transcribed and replied to in a messenger.

- Scope narrowly first: saved/test chat or explicit allowlist.
- Exclude groups, channels, bots, and assistant chats unless explicitly approved.
- Persist seen message IDs so restarts do not re-transcribe old audio.
- Log source message ID, transcript provider, and reply message ID.
- Do not delete or edit messages automatically.
- Treat replies from the user's own account as a write action that requires explicit scope approval.

## Safe Defaults

```yaml
stt:
  enabled: true
  provider: auto
  language: ru
  local:
    model: small
  groq:
    model: whisper-large-v3-turbo
  openai:
    model: gpt-4o-mini-transcribe
```

Adjust `language` and models for the user's actual context. Avoid publishing user-specific paths, chat IDs, phone numbers, usernames, API keys, transcripts, or audio samples.

## Pitfalls

- Do not treat a bad transcript as a writing-style issue until provider, language, and model settings are checked.
- Do not hard-code `en` unless English is the only expected language.
- Do not print API keys or include them in subprocess arguments visible to process listings.
- Do not assume a log line means the user saw the transcript; verify the destination message when possible.
- Do not use one successful saved-chat test as proof that personal or group chat relays work.
- Do not enable automatic replies broadly before an allowlist and dedupe state exist.

## Verification

A voice transcription setup is done only when:

- a fresh voice/audio sample was transcribed;
- the language is correct;
- the output is readable and bounded;
- the chosen provider and fallback are documented;
- secret values are not present in files, logs, commits, or public output;
- for chat relays, the reply was read back from the target chat or API.
