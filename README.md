# SilverBullet Whisper Mic

A small fork of [SilverBullet](https://github.com/silverbulletmd/silverbullet) that adds a built-in microphone button to the top-left corner of the web UI.

The button records audio in the browser with `getUserMedia` and `MediaRecorder`, sends the recording to an OpenAI-compatible transcription endpoint, retries transient failures with exponential backoff, and inserts the returned transcript at the current cursor position.

## What this fork changes

- Adds a global microphone icon in the SilverBullet top bar.
- First click starts recording; the icon turns red while recording.
- Second click stops recording and starts transcription.
- Audio is not saved into the space and no file-upload/save dialog is shown.
- The browser sends `multipart/form-data` to an OpenAI-compatible endpoint.
- The default endpoint is same-origin:

```text
/v1/audio/transcriptions
```

This is intended to be served by your own reverse proxy, which can inject the API key server-side. The API key is not stored in browser code.

## Endpoint contract

The transcription endpoint should behave like OpenAI's audio transcription API:

```http
POST /v1/audio/transcriptions
Content-Type: multipart/form-data
```

Fields:

- `file`: recorded audio blob
- `model`: `whisper-1`
- `response_format`: `json`

Expected JSON response:

```json
{"text":"transcribed text"}
```

For compatibility, this fork also accepts `transcription` or `result` string fields.

## Runtime configuration

By default the UI posts to:

```text
/v1/audio/transcriptions
```

For testing, you can override it in the browser console:

```js
localStorage.setItem("silverbulletWhisperEndpoint", "https://your-proxy.example.com/v1/audio/transcriptions")
```

Then reload SilverBullet.

## Deployment note

Do not put OpenAI or Whisper API keys into browser JavaScript. Put the key in a server-side reverse proxy, for example nginx, and expose a same-origin `/v1/audio/transcriptions` endpoint to SilverBullet.

## Build

```bash
npm ci
npm run build
cargo build --release -p silverbullet --target x86_64-unknown-linux-musl
cp target/x86_64-unknown-linux-musl/release/silverbullet silverbullet-amd64
docker build -t silverbullet-whisper-mic:latest .
docker build -f Dockerfile.runtime-api --build-arg BASE_IMAGE=silverbullet-whisper-mic:latest -t silverbullet-whisper-mic:runtime-api .
```

## Tags

`silverbullet`, `whisper`, `openai`, `transcription`, `microphone`, `pwa`, `self-hosted`, `notes`

## License

MIT, same as upstream SilverBullet.
