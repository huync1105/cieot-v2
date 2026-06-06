# Cieot v2 — Project Master Spec

## Tổng quan

**Cieot** là trợ lý học tập và ôn luyện TOEIC, hỗ trợ tra cứu từ vựng theo chủ đề và luyện nghe thụ động (BBC 6 Minute English). Phiên bản v2 port 1:1 stack và nghiệp vụ từ Cieot v1, tổ chức lại tài liệu theo [AGENTS.md](../../AGENTS.md) để agent code theo từng phase.

**UI:** dark-only, palette pastel (xem [DESIGN.md](../../DESIGN.md)).

**Auth:** guest mode — `user_id` cố định `guest-user`, không SSO/Keycloak.

## Stack & ports

| Thành phần | Công nghệ | Port (host) |
|------------|-----------|-------------|
| Frontend | React 19, Vite 8, TypeScript 6, React Router 7, Tailwind CSS v4, Lucide | **5200** |
| Backend | Python 3.11+, FastAPI 0.116, SQLAlchemy 2 (sync), psycopg3, Uvicorn | **5201** |
| Database | PostgreSQL 16 (Docker) | **5202** → 5432 |
| AI local | Ollama (tùy chọn, chạy riêng) | **11434** |

## Cấu trúc monorepo

```
cieot-v2/
├── frontend/              # Vite React SPA
├── backend/app/           # FastAPI application
├── docker/                # docker-compose.yml (Postgres)
├── docs/
│   ├── preview-dev/       # Đặc tả sống
│   ├── process-dev/       # Log từng phase
│   └── common-errors/
├── start.sh               # Khởi động Postgres + BE + FE
├── .env.example
├── AGENTS.md
└── DESIGN.md
```

## Kiến trúc domain

Tách logic theo 3 domain tái sử dụng (port từ v1):

| Domain | Trách nhiệm | Ví dụ |
|--------|-------------|-------|
| **content** | Dữ liệu học liệu | topics, vocabularies, listening_lessons |
| **progress** | Trạng thái học theo user | learning_progress |
| **assessment** | Quiz, chấm điểm AI | quiz_sessions, grade-example |

Luồng session (tham khảo developer-ascension-v3): init → resume → content → assessment → cập nhật progress.

## Phase roadmap

| Phase | Tên | Phạm vi tóm tắt | Process doc | Trạng thái |
|-------|-----|----------------|-------------|------------|
| **0** | Foundation | Monorepo, Docker Postgres, health API, App Shell dark pastel | [0-Cieot_foundation.md](../process-dev/0-Cieot_foundation.md) | Chưa code |
| **1** | Vocabulary browsing | 50 chủ đề TOEIC, CRUD từ, UI master-detail | [1-Vocabulary_browsing.md](../process-dev/1-Vocabulary_browsing.md) | Chưa code |
| **2** | Vocabulary Quiz | Quiz 3 bước/từ, AI chấm câu, progress | [2-Vocabulary_quiz.md](../process-dev/2-Vocabulary_quiz.md) | Chưa code |
| **3** | Passive Listening | BBC lessons, dictation notes, grade modal TipTap | [3-Passive_listening.md](../process-dev/3-Passive_listening.md) | Chưa code |

## Out of scope (lần rebuild này)

- Đăng nhập / auth thật / phân tích user đa tenant
- Alembic migration (giữ `create_all` + `ALTER TABLE IF NOT EXISTS` như v1)
- CI/Jenkins thực thi (chỉ stub tài liệu)

## API tổng hợp

Prefix backend: `/api` (Vite proxy từ FE). Chi tiết schema: [backend-spec.md](./backend-spec.md).

| Method | Path | Phase | Domain |
|--------|------|-------|--------|
| GET | `/health` | 0 | Health |
| GET | `/api/topics` | 1 | content |
| GET | `/api/topics/{topic_id}` | 1 | content |
| GET | `/api/topics/{topic_id}/vocabularies` | 1 | content |
| GET | `/api/vocabularies/{vocabulary_id}` | 1 | content |
| POST | `/api/topics/{topic_id}/vocabularies` | 1 | content |
| PUT | `/api/vocabularies/{vocabulary_id}` | 1 | content |
| DELETE | `/api/vocabularies/{vocabulary_id}` | 1 | content |
| POST | `/api/topics/{topic_id}/quiz-sessions` | 2 | assessment |
| POST | `/api/quiz-sessions/{session_id}/submit-step` | 2 | assessment |
| POST | `/api/vocabularies/{vocabulary_id}/grade-example` | 2 | assessment |
| POST | `/api/vocabularies/{vocabulary_id}/sentence-hint` | 2 | assessment |
| POST | `/api/topics/{topic_id}/quiz-sessions/{session_id}/complete` | 2 | assessment |
| GET | `/api/topics/{topic_id}/progress` | 2 | progress |
| GET | `/api/listening/passive-lessons` | 3 | content |
| GET | `/api/listening/passive-lessons/{lesson_id}` | 3 | content |
| GET | `/api/listening/passive-lessons/{lesson_id}/notes` | 3 | progress |
| POST | `/api/listening/passive-lessons/{lesson_id}/notes` | 3 | progress |
| PUT | `/api/listening/passive-lessons/{lesson_id}/notes/{attempt_no}` | 3 | progress |
| PUT | `/api/listening/passive-lessons/{lesson_id}/notes/{attempt_no}/formatted` | 3 | progress |
| POST | `/api/listening/passive-lessons/{lesson_id}/download-transcript` | 3 | content |

## Khởi chạy local

```bash
./start.sh
# Frontend: http://localhost:5200
# Backend:  http://127.0.0.1:5201/docs
# Postgres: localhost:5202
# Ollama:   chạy riêng → ollama serve
```

## Tài liệu liên quan

### Preview-spec

- [frontend-spec.md](./frontend-spec.md)
- [backend-spec.md](./backend-spec.md)
- [database-ai-local-spec.md](./database-ai-local-spec.md)
- [cicd-local-jenkins-spec.md](./cicd-local-jenkins-spec.md)

### Process-dev

- [0-Cieot_foundation.md](../process-dev/0-Cieot_foundation.md)
- [1-Vocabulary_browsing.md](../process-dev/1-Vocabulary_browsing.md)
- [2-Vocabulary_quiz.md](../process-dev/2-Vocabulary_quiz.md)
- [3-Passive_listening.md](../process-dev/3-Passive_listening.md)

### Khác

- [DESIGN.md](../../DESIGN.md) — design system dark pastel
- [AGENTS.md](../../AGENTS.md) — quy tắc agent
