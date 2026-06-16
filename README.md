# DHTMLX Scheduler - AI Scheduler Manager Demo

This demo shows how to connect **DHTMLX Scheduler** with an **AI-powered chatbot** that can manage a Scheduler Timeline with natural language instructions.
The chatbot can inspect state, prepare scheduling previews, schedule incoming maintenance requests, reschedule work orders, move work orders back to the request queue, and adjust Scheduler date, zoom, and skin settings.

The sample domain is **office building maintenance**. A facilities coordinator manages scheduled maintenance work orders across staff/team rows and a separate Incoming Requests panel for unscheduled work.

The setup combines **DHTMLX Scheduler** for Timeline visualization, a **frontend app (Vite + TypeScript)** for Scheduler, chat, and local command execution, and a **backend (Express + Socket.IO)** for communication with an LLM through the OpenAI API or a compatible function-calling service. Everything can be run locally or with Docker.

## Features

- **AI-driven Scheduler control** - interact with the Scheduler via chat using natural language instructions.
- **Timeline scheduling** - view scheduled maintenance work orders in a DHTMLX Scheduler Timeline with maintenance staff/resources as rows.
- **Incoming Requests panel** - keep unscheduled maintenance requests outside Scheduler until they receive a resource and time.
- **Preview Mode** - AI scheduling changes are prepared as a visual preview first. Live state changes only after the user clicks Apply.
- **Work order management** - add, update, delete, reschedule, and unschedule work orders through chat tools.
- **Scheduler view control** - change visible date, Timeline zoom, and Scheduler skin through chat commands.
- **Browser voice input** - dictate a chat command with the browser SpeechRecognition API, review/edit the transcript, then send manually.

## How it works

This demo shows how a Scheduler can be managed using natural language commands processed by an LLM. When the user types something like:

> _Generate today's schedule from pending maintenance requests._

the user's request provided via the chatbot, is sent to LLM, which then calls a function. The function returns a command and some data that is processed on the client. The user then reviews the visual preview and applies or discards it.

### The main flow works like this:

1. **Function calling with LLM**

- The backend uses OpenAI-compatible function calling.
- Available tools are defined with Zod in `backend/schemaList.ts`.
- Zod schemas are converted to JSON Schema with `zod-to-json-schema`.
- Runtime validation uses the same schemas before a tool call is emitted to the browser.

2. **Client-side command runner**

- Frontend tool calls are handled by `frontend/src/command-runner.ts`.

3. **System prompt and history management**

- Per-client session history is maintained on the backend for each Socket.IO connection.
- Conversation history is trimmed to stay within token limits while preserving complete assistant tool-call cycles with their tool results.
- `generateSystemPrompt()` guides the model to inspect Scheduler state, use availability facts, preserve incoming request ids, respect working hours/lunch behavior, and follow Preview Mode rules.
- Frontend tool summaries return compact state facts after each command, including preview scheduled/unscheduled ids and recovery guidance for failed or repeated tool calls.
- Apply and Cancel emit `scheduler_state_event` notes back to the backend so later turns know whether a preview was committed or discarded.

4. **Models and limitations**

- The default model is `gpt-5-nano`.
- `MODEL` can be used to choose another model; `OPENAI_MODEL` is also supported as a fallback.
- If using another provider, make sure it supports OpenAI-compatible chat completions and function calling.

## Quick start

### Option 1: Production-style mode (Docker)

```bash
git clone https://github.com/DHTMLX/scheduler-maker-ai-demo.git
cd scheduler-maker-ai-demo
cp .env.example .env
# Edit .env with your API keys
docker compose up --build
```

Open http://localhost:3000 in your browser. The frontend runs on port 3000, backend on port 3001. Make sure you have a valid OpenAI API key or another compatible LLM provider configured in `.env`.

### Option 2: Development mode (Docker)

Run with hot reload for development:

```bash
git clone https://github.com/DHTMLX/scheduler-maker-ai-demo.git
cd scheduler-maker-ai-demo
cp .env.dev.example .env
# Edit .env with your API keys
docker compose -f docker-compose.dev.yml up --build
```

Open **http://localhost:3000** in your browser. The backend runs on **http://localhost:3001**.

### Option 3: Local development (without Docker)

If you prefer running locally without Docker:

```bash
npm install
cp .env.dev.example .env
# Edit .env with your API keys

npm run dev:backend    # http://localhost:3001
npm run dev:frontend   # http://localhost:3000
```

---

## Environment Variables

```bash
# LLM API configuration
OPENAI_API_KEY=YOUR_OPENAI_API_KEY
OPENAI_BASE_URL=YOUR_OPENAI_BASE_URL
MODEL=gpt-5-nano

# Docker Compose configuration
VITE_SOCKET_URL_DOCKER=http://localhost:3001
FRONTEND_ORIGIN_DOCKER=http://localhost:3000
```

## Repo structure:

frontend/  
 ├─ src/  
 │ ├─ chat-widget/  
 │ ├─ command-runner/  
 │ ├─ incoming-panel/  
 │ ├─ preview/  
 │ ├─ scheduler/  
 │ ├─ app-state.ts  
 │ ├─ command-runner.ts  
 │ ├─ main.ts  
 │ └─ style.css  
 ├─ Dockerfile  
 ├─ Dockerfile.dev  
 ├─ index.html  
 ├─ package.json  
 └─ vite.config.ts

backend/  
 ├─ Dockerfile  
 ├─ Dockerfile.dev  
 ├─ constants.ts  
 ├─ helper.ts  
 ├─ logger.ts  
 ├─ prompt.ts  
 ├─ requestGuards.ts  
 ├─ schemaList.ts  
 ├─ server.ts  
 ├─ types.ts  
 ├─ package.json  
 └─ tsconfig.json

docker-compose.yml  
docker-compose.dev.yml  
.env.example  
.env.dev.example  
package.json  
README.md

## Scripts (without Docker)

```bash
npm install
cp .env.example .env

# Backend
npm run dev:backend    # http://localhost:3001

# Frontend
npm run dev:frontend   # http://localhost:3000

# Production build
npm run build
```

## License

Source code in this repo is released under the MIT License.

**DHTMLX Scheduler** is a commercial library. This demo uses `@dhx/trial-scheduler`; use it under a valid [DHTMLX license](https://dhtmlx.com/docs/products/licenses.shtml) or evaluation agreement.
Usage of the OpenAI API or other LLM providers is subject to their terms of service and billing.

## Useful links

- [DHTMLX Scheduler Product Page](https://dhtmlx.com/docs/products/dhtmlxScheduler/)
- [DHTMLX Scheduler Documentation](https://docs.dhtmlx.com/scheduler/)
- [OpenAI API Docs](https://platform.openai.com/docs/)
- [Socket.IO Docs](https://socket.io/docs/v4/)
- [DHTMLX technical support forum](https://forum.dhtmlx.com/)
