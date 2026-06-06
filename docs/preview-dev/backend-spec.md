# Backend Spec — Cieot v2

## Stack

- **Python 3.11+**
- **FastAPI** `0.116.1`
- **Uvicorn** `[standard]` `0.35.0` — port **5201**
- **SQLAlchemy 2** `2.0.43` (sync session)
- **psycopg** `[binary]` `3.2.13`
- **pydantic-settings** `2.10.1`
- **requests** + **BeautifulSoup4** — scrape TOEIC vocab
- **pypdf** + **pdfplumber** — parse transcript PDF BBC
- **pytest** — test tối thiểu (transcript format)

**Không dùng:** Alembic (schema qua `create_all` + startup patches).

## Cấu trúc

```
backend/
├── requirements.txt
├── pytest.ini
├── .env / .env.example
└── app/
    ├── main.py              # FastAPI app + routes (có thể tách api/routes/ khi implement, contract giữ nguyên)
    ├── models.py            # SQLAlchemy models (7 bảng)
    ├── schemas.py           # Pydantic DTO
    ├── database.py          # Engine, Session, Base
    ├── config.py            # Settings từ .env
    ├── seed.py              # Orchestrate TOEIC + BBC seed
    ├── scraper.py           # 600tuvungtoeic.com + fallback
    ├── toeic_topics.py      # 50 tên chủ đề
    ├── topic_thumbnail.py   # SVG thumbnail base64
    ├── bbc_listening_seed.py
    ├── transcript_format.py
    └── services/
        └── ai_grader.py     # Ollama + heuristic
tests/
└── test_transcript_format.py
```

## Cấu hình (.env)

| Biến | Mặc định | Mô tả |
|------|----------|-------|
| `APP_PORT` | `5201` | Port Uvicorn |
| `DATABASE_URL` | `postgresql+psycopg://cieot:cieot@localhost:5202/cieot` | Postgres |
| `TOEIC_BASE_URL` | `https://600tuvungtoeic.com` | Nguồn scrape vocab |
| `AI_ENABLE_OLLAMA` | `false` | Bật gọi Ollama |
| `AI_OLLAMA_URL` | `http://localhost:11434` | Ollama API |
| `AI_OLLAMA_MODEL` | `qwen3.5:4b` | Model chấm câu |
| `AI_COMPARE_MODEL` | `qwen3.5:4b` | Model so khớp nghĩa |
| `AI_TIMEOUT_SECONDS` | `35` | Timeout HTTP Ollama |

## Startup

1. `Base.metadata.create_all(bind=engine)`
2. `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` (patch cột legacy — xem [database-ai-local-spec.md](./database-ai-local-spec.md))
3. Data repair SQL (word_type normalize, xóa note attempt rỗng)
4. `seed_toeic_data(db)` — lần đầu scrape 50 topic + vocab; lần sau repair thumbnail/vocab
5. `seed_passive_listening_data(db)` — merge BBC static seed

## CORS

Dev: `allow_origins=["*"]`, `allow_credentials=True`, methods/headers `*`.

## Endpoints

### Health (phase 0)

| Method | Path | Response |
|--------|------|----------|
| GET | `/health` | `{ "status": "ok" }` |

### Vocabulary — content (phase 1)

| Method | Path | Body / Query | Response |
|--------|------|--------------|----------|
| GET | `/api/topics` | — | `TopicOut[]` |
| GET | `/api/topics/{topic_id}` | — | `TopicOut` |
| GET | `/api/topics/{topic_id}/vocabularies` | — | `VocabularyOut[]` |
| GET | `/api/vocabularies/{vocabulary_id}` | — | `VocabularyOut` |
| POST | `/api/topics/{topic_id}/vocabularies` | `VocabularyCreate` | `VocabularyOut` 201 |
| PUT | `/api/vocabularies/{vocabulary_id}` | `VocabularyUpdate` | `VocabularyOut` |
| DELETE | `/api/vocabularies/{vocabulary_id}` | — | 204; 403 nếu từ scrape gốc |

**TopicOut:** `id`, `slug`, `title_en`, `title_vi`, `thumbnail_base64`, `order_index`

**VocabularyOut:** `id`, `topic_id`, `word`, `phonetic`, `meaning_en`, `meaning_vi`, `word_type`, `example_en`, `example_vi`, `audio_url`, `image_url`, `source_ref`

### Assessment — Quiz (phase 2)

| Method | Path | Body | Response |
|--------|------|------|----------|
| POST | `/api/topics/{topic_id}/quiz-sessions` | `QuizSessionCreate`: `user_id`, `question_count` (1–100, default 10) | `QuizSessionOut` 201 |
| POST | `/api/quiz-sessions/{session_id}/submit-step` | `QuizStepSubmitIn`: `vocabulary_id`, `step` (`fillAnswer`\|`compareOriginal`\|`writeExample`), optional inputs | `QuizStepSubmitOut` |
| POST | `/api/vocabularies/{vocabulary_id}/grade-example` | `SentenceGradeIn`: `user_id`, `sentence` | `SentenceGradeOut` |
| POST | `/api/vocabularies/{vocabulary_id}/sentence-hint` | `SentenceHintIn`: `word` | `SentenceHintOut` |
| POST | `/api/topics/{topic_id}/quiz-sessions/{session_id}/complete` | — | `QuizSessionCompleteOut` |
| GET | `/api/topics/{topic_id}/progress` | `user_id` query | `LearningProgressOut` |

**Quiz session:** persist PostgreSQL (`quiz_sessions`, `quiz_session_questions`); xóa session row khi complete.

**Pass threshold:** 75%.

**QuizStepSubmitOut** có thể trả thêm: `ai_match_result`, `ai_match_level`, `ai_reason_vi/en`, `system_word_type_status`, `system_word_type_reason_vi`.

**SentenceGradeOut:** `score` 0–100, `feedback`, `corrected_sentence`, `passed`, `ai_provider`, `model`.

### Listening (phase 3)

| Method | Path | Body / Query | Response |
|--------|------|--------------|----------|
| GET | `/api/listening/passive-lessons` | — | `ListeningLessonListItemOut[]` |
| GET | `/api/listening/passive-lessons/{lesson_id}` | — | `ListeningLessonDetailOut` |
| GET | `/api/listening/passive-lessons/{lesson_id}/notes` | `user_id` | `ListeningNoteAttemptOut[]` |
| POST | `/api/listening/passive-lessons/{lesson_id}/notes` | `ListeningNoteAttemptCreateIn`: `user_id` | `ListeningNoteAttemptOut` 201 (attempt tiếp theo) |
| PUT | `/api/listening/passive-lessons/{lesson_id}/notes/{attempt_no}` | `ListeningNoteAttemptUpdateIn`: `user_id`, `content` | `ListeningNoteAttemptOut` |
| PUT | `/api/listening/passive-lessons/{lesson_id}/notes/{attempt_no}/formatted` | `ListeningNoteAttemptFormattedUpdateIn`: `user_id`, `formatted_content`, `corrections_json?` | `ListeningNoteAttemptOut` |
| POST | `/api/listening/passive-lessons/{lesson_id}/download-transcript` | — | `ListeningTranscriptDownloadOut` |

**ListeningLessonDetailOut** thêm: `transcript_source_url`, `introduction`, `weekly_question`, `vocabulary`, `transcript`.

## AI Grader (`services/ai_grader.py`)

| Hàm | Mục đích | Fallback |
|-----|----------|----------|
| `grade_sentence` | Chấm câu ví dụ (grammar 40, usage 35, clarity 25) | Heuristic local |
| `compare_meaning` | So khớp nghĩa quiz bước `compareOriginal` | Heuristic |
| `generate_sentence_hint_vi` | Gợi ý câu tiếng Việt cho luyện dịch | Heuristic v2 |

Khi `AI_ENABLE_OLLAMA=false` hoặc Ollama unreachable → `ai_provider: heuristic`, model string mô tả fallback.

## Seed & scrape

- **TOEIC:** 50 topics từ `toeic_topics.py`; scrape `600tuvungtoeic.com` per lesson; fallback generate nếu scrape fail.
- **BBC:** static data `bbc_listening_seed.py` (~10 episodes); merge giữ transcript đã download.
- **Transcript download:** fetch PDF/HTML từ BBC, parse qua `transcript_format.py`.

## Liên kết

- [database-ai-local-spec.md](./database-ai-local-spec.md)
- [project-master-spec.md](./project-master-spec.md)
- Process: [1-Vocabulary_browsing.md](../process-dev/1-Vocabulary_browsing.md), [2-Vocabulary_quiz.md](../process-dev/2-Vocabulary_quiz.md), [3-Passive_listening.md](../process-dev/3-Passive_listening.md)
