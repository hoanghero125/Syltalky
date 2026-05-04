# Syltalky

Real-time accessible video conferencing for Vietnamese speakers — with AI-powered live captions, voice cloning, and sign language translation built in.

---

## What it is

Syltalky is a full-stack video conferencing platform designed around accessibility. Every meeting is transcribed live in Vietnamese, participants can speak through a cloned or designed AI voice, and sign language video is translated automatically. After the meeting ends, an LLM-generated summary (in Vietnamese Markdown) is created from the transcript and delivered as a notification.

---

## Repositories

| Repo | Stack | Purpose |
|---|---|---|
| [`Syltalky_API/`](./Syltalky_API/) | Python · FastAPI · CUDA | AI services: STT, TTS, sign language translation |
| [`Syltalky_BE/`](./Syltalky_BE/) | Python · FastAPI · PostgreSQL | Backend: auth, meetings, voice profiles, notifications |
| [`Syltalky_FE/`](./Syltalky_FE/) | React · Vite · LiveKit | Web app: all user-facing screens |

Each repo is its own git repository with its own README, commit history, and deployment lifecycle.

---

## Architecture

```
Browser (Syltalky_FE)
    │
    ├── HTTP/WS  →  Syltalky_BE (port 8001)
    │                   │
    │                   ├── PostgreSQL  (port 5432)
    │                   ├── MinIO       (port 9000)
    │                   ├── LiveKit     (port 7880)
    │                   ├── Redis       (port 6379)
    │                   └── HTTP  →  Syltalky_API (port 8000)
    │
    └── WebRTC   →  LiveKit (port 7880 / 7882 UDP)
```

The frontend talks only to the backend. The backend proxies all AI work to the AI API. The AI API requires a CUDA-capable GPU.

---

## Feature overview

### Live captions
The backend taps each participant's LiveKit audio track, streams PCM chunks to the AI API's `/ws/stt` WebSocket, and broadcasts the resulting Vietnamese text back to the meeting room in real time. Captions appear as a subtitle overlay on each speaker's video tile and accumulate in a scrollable Captions panel.

### Voice cloning & design
Users can record or upload a 5–15s audio clip to clone their voice. The backend runs it through STT (to get the transcript), then sends both to the AI API to register a voice. At meeting time, the TTS panel lets a user type text and hear it read aloud in their cloned voice (or a designed voice built from style tags). The audio is broadcast to all participants.

### Sign language translation
Upload an ASL video from the meeting interface. The AI API extracts pose keypoints with RTMPose, runs them through Uni-Sign (ASL → English), and translates the result to Vietnamese with EnViT5.

### Meeting history & AI summaries
When a meeting ends, a post-processing job builds a full transcript from the saved captions, summarises it using the configured LLM, and pushes a `summary_ready` notification to the host via WebSocket. The Library screen shows all past meetings with their summaries and transcripts.

### Meeting extras
Live meetings also support: pinned messages, polls (single/multiple choice), collaborative notes (Tiptap + Yjs CRDT), co-host promotion, a waiting room, and an AI chat assistant powered by a configurable LLM.

---

## Deployment

- Domain: **syltalky.pro.vn** (Cloudflare DNS)
- Frontend → `https://syltalky.pro.vn`
- Backend → `https://api.syltalky.pro.vn`
- MinIO public files → `https://minio.syltalky.pro.vn`

Both Docker Compose stacks (`Syltalky_API` and `Syltalky_BE`) run on the same machine. The frontend is a static Vite build served by a web server (Nginx or Caddy).

---

## External services

| Service | Purpose |
|---|---|
| LiveKit (self-hosted) | WebRTC audio/video |
| MinIO (self-hosted) | Avatars, reference audio |
| Resend | Transactional email (verify, reset password) |
| Qwen3.5-35B-A3B (OpenAI-compatible proxy) | Meeting summarisation + AI chat assistant |
| Google OAuth | Sign-in with Google |

---

## Development phases

| Phase | What was built |
|---|---|
| 1 | Project scaffold — FastAPI, Vite, Alembic, Docker Compose |
| 2 | Auth — register, login, JWT, email verify, forgot/reset password |
| 3 | User profile, voice config, Settings modal |gu
| 4 | Voice clone — upload/record, waveform trim, STT→TTS pipeline |
| 5 | Meetings core — create, join, LiveKit, device check, meeting room grid |
| 6 | Real-time captions — audio tap, WebSocket STT, subtitle overlay |
| 7 | TTS in meeting — text input, voice synthesis, audio broadcast |
| 8 | Post-processing — LLM summary, notifications, Library |
| 9 | Meeting extras — pins, polls, notes, co-host, waiting room, AI chat |
