# Database & AI Local Spec — Cieot v2

## PostgreSQL

- Image: `postgres:16`
- Compose: [`docker/docker-compose.yml`](../../docker/docker-compose.yml) (tạo ở phase 0)
- Container: `cieot-postgres`
- Host port: **5202** → container **5432**

### Credentials (dev only)

| Biến | Giá trị |
|------|---------|
| POSTGRES_USER | `cieot` |
| POSTGRES_PASSWORD | `cieot` |
| POSTGRES_DB | `cieot` |

### Connection string

```
postgresql+psycopg://cieot:cieot@localhost:5202/cieot
```

### Volume

- Named volume: `cieot-postgres-data`

## Chiến lược schema

- **Không Alembic.** Khởi tạo qua `Base.metadata.create_all()` khi startup.
- **Startup patches:** `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` cho DB cũ nâng cấp từ v1-compatible state.

### Patches trên startup (port v1)

| Bảng | Cột thêm |
|------|----------|
| `vocabularies` | `image_url TEXT` |
| `topics` | `thumbnail_base64 TEXT DEFAULT ''` |
| `quiz_session_questions` | `user_meaning_input TEXT`, `user_word_type_input VARCHAR(80)` |
| `listening_lessons` | `audio_url TEXT DEFAULT ''`, `thumbnail_base64 TEXT DEFAULT ''`, `transcript_source_url TEXT DEFAULT ''` |
| `listening_note_attempts` | `formatted_content TEXT DEFAULT ''`, `formatted_updated_at TIMESTAMP NULL` |

Data repair bổ sung: normalize `word_type` prefix; xóa `listening_note_attempts` attempt 1 rỗng.

## Sơ đồ quan hệ

```mermaid
erDiagram
  topics ||--o{ vocabularies : has
  topics ||--o{ quiz_sessions : has
  quiz_sessions ||--o{ quiz_session_questions : contains
  vocabularies ||--o{ quiz_session_questions : references
  listening_lessons ||--o{ listening_note_attempts : has

  topics {
    int id PK
    string slug UK
    string title_en
    string title_vi
    text thumbnail_base64
    int order_index
  }

  vocabularies {
    int id PK
    int topic_id FK
    string word
    string phonetic
    text meaning_en
    text meaning_vi
    string word_type
    text example_en
    text example_vi
    text audio_url
    text image_url
    text source_ref
  }

  learning_progress {
    int id PK
    string user_id
    string item_type
    int item_id
    string status
    float confidence_score
    datetime last_reviewed_at
    datetime next_review_at
  }

  quiz_sessions {
    string session_id PK
    int topic_id FK
    string user_id
    datetime started_at
  }

  quiz_session_questions {
    int id PK
    string session_id FK
    int vocabulary_id FK
    int question_order
    string word
    text prompt
    int step_index
    float score
    bool done
  }

  listening_lessons {
    int id PK
    string slug UK
    string episode_code UK
    string title
    text transcript
    int order_index
  }

  listening_note_attempts {
    int id PK
    string user_id
    int lesson_id FK
    int attempt_no
    text content
    text formatted_content
    text corrections_json
  }
```

## Bảng chi tiết

### `topics` (phase 1)

| Cột | Kiểu | Ràng buộc |
|-----|------|-----------|
| `id` | INTEGER | PK, index |
| `slug` | VARCHAR(120) | UNIQUE, index |
| `title_en` | VARCHAR(180) | index |
| `title_vi` | VARCHAR(180) | default `''` |
| `thumbnail_base64` | TEXT | default `''` — SVG data URL |
| `order_index` | INTEGER | index |

Quan hệ: `1:N` → `vocabularies` (cascade delete orphan).

### `vocabularies` (phase 1)

| Cột | Kiểu | Ràng buộc |
|-----|------|-----------|
| `id` | INTEGER | PK |
| `topic_id` | INTEGER | FK → `topics.id`, index |
| `word` | VARCHAR(120) | index |
| `phonetic` | VARCHAR(200) | |
| `meaning_en` / `meaning_vi` | TEXT | |
| `word_type` | VARCHAR(80) | |
| `example_en` / `example_vi` | TEXT | |
| `audio_url` | TEXT | nullable |
| `image_url` | TEXT | nullable |
| `source_ref` | TEXT | scrape URL hoặc custom |

### `learning_progress` (phase 2)

| Cột | Kiểu | Ràng buộc |
|-----|------|-----------|
| `id` | INTEGER | PK |
| `user_id` | VARCHAR(100) | index |
| `item_type` | VARCHAR(50) | index — ví dụ `topic_quiz`, `vocabulary` |
| `item_id` | INTEGER | index |
| `status` | VARCHAR(30) | default `new` |
| `confidence_score` | FLOAT | default 0 |
| `last_reviewed_at` | TIMESTAMP | nullable |
| `next_review_at` | TIMESTAMP | nullable |

**Unique:** `(user_id, item_type, item_id)`.

### `quiz_sessions` (phase 2)

| Cột | Kiểu | Ràng buộc |
|-----|------|-----------|
| `session_id` | VARCHAR(32) | PK |
| `topic_id` | INTEGER | FK → `topics.id` |
| `user_id` | VARCHAR(100) | index |
| `started_at` | TIMESTAMP | index |

Quan hệ: `1:N` → `quiz_session_questions` (cascade, order by `question_order`).

### `quiz_session_questions` (phase 2)

| Cột | Kiểu | Ràng buộc |
|-----|------|-----------|
| `id` | INTEGER | PK |
| `session_id` | VARCHAR(32) | FK → `quiz_sessions.session_id` |
| `vocabulary_id` | INTEGER | FK → `vocabularies.id` |
| `question_order` | INTEGER | |
| `word`, `phonetic`, `prompt` | | snapshot câu hỏi |
| `expected_meaning_vi`, `expected_word_type` | TEXT/VARCHAR | |
| `user_meaning_input`, `user_word_type_input` | | nullable |
| `step_index` | INTEGER | default 0 |
| `score` | FLOAT | default 0 |
| `done` | BOOLEAN | default false |

**Unique:** `(session_id, vocabulary_id)`.

### `listening_lessons` (phase 3)

| Cột | Kiểu | Ràng buộc |
|-----|------|-----------|
| `id` | INTEGER | PK |
| `slug` | VARCHAR(160) | UNIQUE |
| `title` | VARCHAR(255) | index |
| `episode_code` | VARCHAR(20) | UNIQUE |
| `published_date` | VARCHAR(30) | |
| `summary` | TEXT | |
| `source_url` | TEXT | BBC page |
| `audio_url` | TEXT | |
| `thumbnail_base64` | TEXT | |
| `transcript_source_url` | TEXT | PDF/HTML URL |
| `introduction` | TEXT | |
| `weekly_question` | TEXT | |
| `vocabulary` | TEXT | section vocab |
| `transcript` | TEXT | full transcript |
| `order_index` | INTEGER | index |

### `listening_note_attempts` (phase 3)

| Cột | Kiểu | Ràng buộc |
|-----|------|-----------|
| `id` | INTEGER | PK |
| `user_id` | VARCHAR(100) | index |
| `lesson_id` | INTEGER | FK → `listening_lessons.id` |
| `attempt_no` | INTEGER | default 1 |
| `content` | TEXT | raw dictation |
| `formatted_content` | TEXT | HTML sau grade |
| `formatted_updated_at` | TIMESTAMP | nullable |
| `corrections_json` | TEXT | nullable — JSON bookmarks/misheard |
| `created_at` / `updated_at` | TIMESTAMP | |

**Unique:** `(user_id, lesson_id, attempt_no)`.

## Seed

| Nguồn | Trigger | Ghi chú |
|-------|---------|---------|
| TOEIC 50 topics | Startup nếu `topics` rỗng | Scrape + fallback |
| BBC passive lessons | Startup merge | Static seed, giữ transcript đã tải |

## Ollama (AI local)

- **Không có bảng DB** cho AI.
- Chạy riêng: `ollama serve` (port 11434).
- Model khuyến nghị dev: `qwen3.5:4b`.
- Bật qua `AI_ENABLE_OLLAMA=true` trong `backend/.env`.
- Health: manual `curl http://localhost:11434/api/tags`.

## Liên kết

- [backend-spec.md](./backend-spec.md)
- [project-master-spec.md](./project-master-spec.md)
