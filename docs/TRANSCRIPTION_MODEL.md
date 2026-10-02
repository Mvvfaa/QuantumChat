# Voice transcription model selection and licensing

This project uses a browser-side Whisper transcription flow that stays local to the client and never sends audio or plaintext transcript to the backend.

## Exact artifact

- Runtime library: `@huggingface/transformers` version `4.3.0`
- Whisper model: `Xenova/whisper-tiny`
- Base model: `openai/whisper-tiny`
- Pinned revision: `5332fcc35e32a33b86612b9a57a89be7906102b1`

## Why this artifact

The `Xenova/whisper-tiny` repo is the ONNX-converted browser-compatible version of the official `openai/whisper-tiny` checkpoint, intended for Transformers.js. The model page explicitly states that it is the OpenAI Whisper tiny model with ONNX weights for compatibility with Transformers.js.

## License verification

- `openai/whisper-tiny`: Apache-2.0
- `Xenova/whisper-tiny`: Apache-2.0

The repository and model card both declare the same Apache-2.0 license, which is compatible with the project’s existing open-source distribution model and does not require a separate backend speech API.

## Safety constraints for this repo

- The model is never loaded during normal application startup.
- The runtime is lazy-loaded inside a dedicated Web Worker only after the user explicitly chooses to transcribe.
- The model is pinned to an exact commit SHA rather than a floating `main` or latest tag.
- The backend never receives plaintext audio or plaintext transcript content.

## Acceptance criteria used for this implementation

1. Verify exact model repo and revision before install.
2. Verify license compatibility before using the artifact.
3. Pin the exact revision.
4. Keep model download and execution local-only.
5. Keep backend plaintext-free and preserve existing sealed-envelope message flow.
