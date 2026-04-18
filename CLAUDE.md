# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

xiaozhi-esp32-server is a comprehensive backend system for ESP32-based smart hardware devices, providing voice interaction, AI service integration, IoT device management, and web/mobile admin interfaces.

## Architecture

The project consists of four main components:

```
main/
├─ xiaozhi-server/   # Python core AI engine (WebSocket with ESP32, ASR/LLM/TTS/VAD)
├─ manager-api/      # Java Spring Boot backend (RESTful API, MySQL, Redis)
├─ manager-web/      # Vue.js 2 web admin panel
└─ manager-mobile/   # uni-app + Vue 3 mobile admin (cross-platform)
```

**Ports:** xiaozhi-server:8000 (WebSocket), manager-web:8001, manager-api:8002, HTTP OTA:8003

**Data Flow:** ESP32 → WebSocket → xiaozhi-server (VAD→ASR→LLM→TTS) → ESP32. Configuration flows from manager-api to xiaozhi-server via HTTP.

## Key Commands

### xiaozhi-server (Python)

```bash
# Create environment
conda create -n xiaozhi-esp32-server python=3.10 -y
conda activate xiaozhi-esp32-server
conda install libopus ffmpeg -y

# Install dependencies
cd main/xiaozhi-server
pip install -r requirements.txt

# Run server
python app.py

# Performance testing
python performance_tester.py
```

### manager-api (Java)

```bash
cd main/manager-api
mvn clean install
mvn spring-boot:run
```

### manager-web (Vue.js)

```bash
cd main/manager-web
npm install
npm run serve    # Development
npm run build    # Production
```

### manager-mobile (uni-app)

```bash
cd main/manager-mobile
pnpm install
pnpm dev:h5      # H5 development
pnpm build:h5    # H5 production
```

### Docker Deployment

```bash
# Full deployment
docker compose -f docker-compose_all.yml up -d

# Server only
docker compose -f docker-compose.yml up -d
```

## Configuration

- **Main config:** `main/xiaozhi-server/config.yaml` (reference)
- **User config:** `main/xiaozhi-server/data/.config.yaml` (overrides, create this file)
- User config takes priority; only add keys you want to override.
- Required for startup: LLM api_key in `.config.yaml`

**Minimal config example:**
```yaml
selected_module:
  LLM: ChatGLMLLM

LLM:
  ChatGLMLLM:
    api_key: your_api_key_here
```

## Module Selection

Modules are selected via `selected_module` in config:
- **VAD:** SileroVAD (local)
- **ASR:** FunASR (local), XunfeiStreamASR, DoubaoStreamASR, etc.
- **LLM:** ChatGLMLLM (free), DoubaoLLM, AliLLM, DeepSeekLLM, OllamaLLM, etc.
- **TTS:** EdgeTTS (free), HuoshanDoubleStreamTTS, DoubaoTTS, etc.
- **Memory:** nomem (disabled), mem_local_short, mem0ai, powermem
- **Intent:** function_call, intent_llm, nointent

## Plugin System

Plugins are in `main/xiaozhi-server/plugins_func/functions/`. Enable via `Intent.function_call.functions` or `Intent.intent_llm.functions` in config.

Built-in plugins: `handle_exit_intent`, `play_music`. Additional: `get_weather`, `get_news_from_newsnow`, `change_role`, `search_from_ragflow`, `hass_*` (Home Assistant).

## Core Providers Structure

`main/xiaozhi-server/core/providers/`:
- `asr/` - Speech recognition adapters
- `llm/` - Language model adapters
- `tts/` - Speech synthesis adapters
- `vad/` - Voice activity detection
- `memory/` - Conversation memory
- `intent/` - Intent recognition
- `tools/` - MCP/IoT tool handlers

All providers follow a base class pattern (`base.py`) with type-specific implementations.

## Handle Flow

`main/xiaozhi-server/core/handle/` processes WebSocket messages:
- `helloHandle.py` - Initial connection
- `receiveAudioHandle.py` - Audio input processing
- `sendAudioHandle.py` - TTS output
- `intentHandler.py` - Intent/tool execution
- `abortHandle.py` - Interruption handling

## Testing

- **Audio test:** Open `main/xiaozhi-server/test/test_page.html` in Chrome
- **Performance test:** `python performance_tester.py` (tests ASR/LLM/TTS latency)

## Prerequisites

- **Model file:** Download `model.pt` to `models/SenseVoiceSmall/` for FunASR
  - Source: https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt
- **ffmpeg:** Required for audio processing (install via conda or system)