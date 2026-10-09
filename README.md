# Voice Transcription Skill for Hermes Agent

A public, sanitized Hermes Agent skill for configuring and debugging voice/audio transcription workflows.

It covers:

- choosing local Whisper, Groq, or OpenAI transcription providers;
- avoiding wrong-language STT failures;
- safe secret handling;
- Telegram-style voice-message relay boundaries;
- verification steps for real transcripts.

No private paths, chat IDs, transcripts, API keys, or user-specific configuration are included.

## Files

- `SKILL.md` — the Hermes skill.

## Use

Copy `SKILL.md` into a Hermes skill directory or adapt it into an in-repo skill.

## License

MIT
