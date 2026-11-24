# Docker Compose Server

Run the audio.cpp TTS server as configured in the server JSON files 
with Docker Compose.

## PocketTTS

### 1. Download the PocketTTS model

Get the English q8 model from
[audio-cpp/audio.cpp-gguf](https://huggingface.co/audio-cpp/audio.cpp-gguf/tree/main/PocketTTS-GGUF)
on Hugging Face.
Place the files from `english/` into `../models/PocketTTS-GGUF/english/`:

The directory should look like:
```
../models/PocketTTS-GGUF/english/
├── pocket-tts-english-q8_0.gguf
└── embeddings/
    ├── alba.safetensors
    └── ...
```

### 2. Start the server

Start **one** of:

```bash
docker compose -f pocket-tts-cuda12.yml up
docker compose -f pocket-tts-cuda13.yml up
docker compose -f pocket-tts-cpu.yml up
```

### 3. Optionally: Wait for server to be ready

```bash
./wait-for-server.sh
```

### 4. Generate speech

```bash
./pocket-tts.sh
```

This sends a request to `http://localhost:8080/v1/audio/speech` and saves
the result to `output/speech.wav`.

## Qwen3-TTS (with voice cloning)

### 1. Download the Qwen3-TTS model

Get the q8 model from
[audio-cpp/audio.cpp-gguf](https://huggingface.co/audio-cpp/audio.cpp-gguf/tree/main/Qwen3-TTS-12Hz-1.7B-Base-GGUF)
on Hugging Face.
Place the file into `../models/Qwen3-TTS-12Hz-1.7B-Base-GGUF/`:

The directory should look like:
```
../models/Qwen3-TTS-12Hz-1.7B-Base-GGUF/
└── qwen3-tts-12hz-1.7b-base-q8_0_v2.gguf
```

### 2. Add reference audio and transcription

Put a `ref_audio.wav` and `ref_text.txt` in `../references/`.

The directory should look like:
```
../references/
├── ref_audio.wav
└── ref_text.txt
```

### 3. Start the server

Start **one** of:

```bash
docker compose -f qwen3-tts-cuda12.yml up
docker compose -f qwen3-tts-cuda13.yml up
docker compose -f qwen3-tts-cpu.yml up
```

### 4. Optionally: Wait for server to be ready

```bash
./wait-for-server.sh
```

### 5. Generate speech

```bash
./qwen3-tts.sh
```

This sends a request to `http://localhost:8080/v1/audio/speech` with voice
cloning parameters and saves the result to `output/speech.wav`.

## Higgs Audio v3 TTS

The Higgs Audio v3 TTS weights use a research/non-commercial license. Check
the [model license notes](../../../docs/model_licenses.md) before deploying it.

### 1. Download the model

From the repository root, install the recommended Q8_0 package into the
Docker examples model directory:

```bash
python3 tools/model_manager_v2.py install \
  --models-root examples/docker/models higgs_audio_tts_4b_q8_0
```

For voice cloning, create `examples/docker/references/` and place a reference
WAV there.

### 2. Start the server

From this directory, choose either CUDA or CPU:

```bash
docker compose -f higgs-audio-v3-cuda12.yml up
# Or, without a usable NVIDIA GPU:
docker compose -f higgs-audio-v3-cpu.yml up
```

The server is available at `http://localhost:49112`. CPU inference is supported,
but will be substantially slower; the Q8 model weights alone are about 5 GB, so
leave additional system RAM for inference.

### 3. Generate speech

```bash
curl -f http://localhost:49112/v1/audio/speech \
  -H 'Content-Type: application/json' \
  -o higgs-audio-v3.wav \
  -d '{
    "model": "higgs-v3",
    "input": "Hello from Higgs Audio.",
    "voice_ref": "/references/ref.wav",
    "reference_text": "The words spoken in the reference audio."
  }'
```

Omit `voice_ref` and `reference_text` to generate without reference audio.