# Cieot v2

Trợ lý học và ôn luyện TOEIC — từ vựng theo chủ đề và nghe thụ động (BBC 6 Minute English). Rebuild từ [Cieot v1](../Cieot) với tài liệu agent-driven.

## Stack

| Thành phần | Công nghệ | Port |
|------------|-----------|------|
| Frontend | React 19, Vite, TypeScript, Tailwind v4 | **5200** |
| Backend | FastAPI, SQLAlchemy, psycopg | **5201** |
| Database | PostgreSQL 16 (Docker) | **5202** |
| AI (tùy chọn) | Ollama local | 11434 |

## Chạy local (sau khi implement phase 0)

```bash
./start.sh
```

- Frontend: http://localhost:5200
- API docs: http://127.0.0.1:5201/docs
- Ollama: chạy riêng `ollama serve`

## Tài liệu (đọc trước khi code)

1. [AGENTS.md](./AGENTS.md) — quy tắc agent
2. [docs/preview-dev/project-master-spec.md](./docs/preview-dev/project-master-spec.md) — tổng quan
3. Spec theo mảng: [frontend](./docs/preview-dev/frontend-spec.md), [backend](./docs/preview-dev/backend-spec.md), [database+AI](./docs/preview-dev/database-ai-local-spec.md)
4. [docs/process-dev/](./docs/process-dev/) — checklist từng phase (0 → 3)
5. [DESIGN.md](./DESIGN.md) — UI dark pastel-only

## Phase roadmap

| Phase | Mô tả | Process doc |
|-------|--------|-------------|
| 0 | Foundation — shell, health, Postgres | [0-Cieot_foundation.md](./docs/process-dev/0-Cieot_foundation.md) |
| 1 | Vocabulary browsing | [1-Vocabulary_browsing.md](./docs/process-dev/1-Vocabulary_browsing.md) |
| 2 | Vocabulary Quiz + AI | [2-Vocabulary_quiz.md](./docs/process-dev/2-Vocabulary_quiz.md) |
| 3 | Passive Listening | [3-Passive_listening.md](./docs/process-dev/3-Passive_listening.md) |

Trạng thái hiện tại: **chỉ tài liệu** — chưa có code application.
