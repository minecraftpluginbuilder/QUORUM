# Quorum

**An open-source, OpenAI-compatible AI server that runs on a free Google Colab T4. Several small models check each other's work before you get an answer.**

Quorum runs open GGUF models locally with `llama-cpp-python`, serves them through FastAPI, and exposes them to the internet with an ngrok tunnel. Any app that speaks the OpenAI API (chat completions and image generation) can use it by changing the base URL.

## Features

- **OpenAI-compatible API**: `/v1/chat/completions` (with streaming), `/v1/models` and `/v1/images/generations`
- **Four modes**, from quick answers to multi-model review (see below)
- **Image generation** with a small diffusion model that fits on the T4
- **API-key protection**, so a public ngrok URL isn't open to everyone
- **Single file server** (`quorum_server.py`) plus a **single file chat page** (`quorum_chat.html`)
- **Context protection**: long prompts are trimmed instead of crashing, and cut-off replies are continued automatically

## Modes

| Mode | Models | How it works |
|---|---|---|
| `Quorum-UltraFast` | 1 | One model answers directly |
| `Quorum-Fast` | 2 | One model answers, a second reviews it, and the first fixes any problems |
| `Quorum-Code` | 3 | An explainer plans, a coder writes, a checker reviews. Repeats until it passes |
| `Quorum-Max` | 4 | An architect splits the task into parts. A builder writes each part. An auditor and a tester must both approve it before it is kept |

Only one model is in GPU memory at a time, so the server swaps models as needed. Higher modes are slower but more reliable.

### Models used

| Role | Model |
|---|---|
| Main / Builder / Coder | DeepSeek-Coder-V2-Lite-Instruct (Q4_K_M) |
| Architect / Explainer | Gemma 3 12B (QAT Q4_0) |
| Auditor | gpt-oss-20b (MXFP4) |
| Tester / Checker | Phi-3.5-mini-instruct (Q5_K_L) |
| Images | SDXL-Turbo |

You can change which model plays which role in the settings near the top of `quorum_server.py`.

## Requirements

- A Google Colab notebook with a **T4 GPU** runtime
- A free [ngrok](https://ngrok.com) account and authtoken
- A [Hugging Face](https://huggingface.co) account and access token (Gemma is gated, so accept its license on its model page)

## Setup

1. Copy `.env.example` to `.env` and fill it in:

   ```
   NGROK_AUTHTOKEN=your_ngrok_token
   HF_TOKEN=your_huggingface_token
   QUORUM_API_KEY=choose_any_password
   ```

   `QUORUM_API_KEY` is a password you make up. Clients send it as `Authorization: Bearer <key>`.

2. In Colab, switch the runtime to **T4 GPU**, upload `.env` to `/content/.env`, and upload `quorum_server.py`.
3. Run it in a cell:

   ```python
   %run quorum_server.py
   ```

   Colab Secrets also work instead of a `.env` file.
4. The first run installs packages and downloads the models, which takes a while. When it finishes you'll see **Quorum is live** and your public URL.

Never commit your `.env` file. Colab deletes uploaded files when the runtime resets, so re-upload it each session. The ngrok URL also changes every time you restart.

## Usage

### Chat

```bash
curl YOUR_URL/v1/chat/completions \
  -H "Authorization: Bearer YOUR_QUORUM_API_KEY" \
  -H "ngrok-skip-browser-warning: true" \
  -H "Content-Type: application/json" \
  -d '{"model": "Quorum-Fast", "messages": [{"role": "user", "content": "Hello!"}]}'
```

Send `ngrok-skip-browser-warning: true` on every request, or ngrok may return a warning page instead of your data.

### Images

```bash
curl YOUR_URL/v1/images/generations \
  -H "Authorization: Bearer YOUR_QUORUM_API_KEY" \
  -H "ngrok-skip-browser-warning: true" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "a red fox in snow", "size": "512x512"}'
```

The response contains base64 PNGs in `data[].b64_json`. In any chat mode you can also type `/image a red fox in snow` to get a picture back in the reply.

### With OpenAI-compatible apps

Use `YOUR_URL/v1` as the base URL, your `QUORUM_API_KEY` as the API key, and one of the mode names above as the model.

### Chat page

Open `quorum_chat.html` in a browser (or host it on GitHub Pages), tap **Settings**, and paste your URL and API key. They are saved only in your own browser.

## Configuration

All settings are at the top of `quorum_server.py`:

| Setting | What it does |
|---|---|
| `FAST_*`, `CODE_*` | Which model plays each role in Fast and Code |
| `MAX_MODELS` | The four Max models and their context sizes |
| `MAX_ROUNDS`, `MAX_PART_TRIES`, `MAX_PARTS` | Safety limits for the review loops |
| `ENABLE_IMAGES`, `IMAGE_STEPS`, `IMAGE_MAX_SIDE` | Turn image generation on or off and tune speed and size |

## Limitations

- Free Colab sessions disconnect after a while, so this is for development and personal use, not production.
- Requests are handled one at a time, and swapping models takes time. Max can take minutes.
- Small models make mistakes. The review loops reduce them but don't remove them.
- Image quality is below large hosted generators, and text inside images is weak.
- Telling the models to answer as "Quorum" is a prompt instruction, not a guarantee.

## Licenses

The Quorum code is released under the license in this repository. The models it downloads each have their own license, and you are responsible for following them. In particular, SDXL-Turbo is free for non-commercial use, and commercial use needs a Stability AI membership. Set `ENABLE_IMAGES = False` to turn it off.


## Website / Host

Use this URL: https://quorumbeta.netlify.app/
to use the ai

or

Use the html and host your self to use the ai

## END
