# Phase 1 — Vocabulary browsing

## Mục tiêu

Triển khai module **Từ vựng TOEIC**: 50 chủ đề, danh sách/chi tiết từ, CRUD từ tùy chỉnh, seed từ 600tuvungtoeic.com, UI master-detail dark pastel.

## Phạm vi

### In scope

- Bảng `topics`, `vocabularies`
- Seed 50 topic + vocab (scrape + fallback generation)
- Thumbnail SVG base64 per topic
- API list/detail/CRUD vocabulary
- UI hub → topics grid → topic detail master-detail
- Phát âm: `audio_url` hoặc browser SpeechSynthesis
- Highlight từ trong example (inflection regex)
- Bảo vệ xóa từ scrape gốc (`source_ref` từ 600tuvungtoeic)

### Out of scope

- Quiz (phase 2)
- Auth đa user

## Checklist triển khai

### Backend / DB

- [ ] `models.py` — `Topic`, `Vocabulary`
- [ ] `schemas.py` — `TopicOut`, `VocabularyOut`, `VocabularyCreate`, `VocabularyUpdate`
- [ ] `toeic_topics.py` — 50 tên chủ đề
- [ ] `scraper.py` — scrape lesson pages + fallback
- [ ] `topic_thumbnail.py` — generate SVG thumbnail
- [ ] `seed.py` — `seed_toeic_data`, repair pass
- [ ] Startup: gọi seed sau patches
- [ ] Routes:
  - `GET /api/topics`
  - `GET /api/topics/{topic_id}`
  - `GET /api/topics/{topic_id}/vocabularies`
  - `GET /api/vocabularies/{vocabulary_id}`
  - `POST /api/topics/{topic_id}/vocabularies`
  - `PUT /api/vocabularies/{vocabulary_id}`
  - `DELETE /api/vocabularies/{vocabulary_id}` — 403 nếu protected

### Frontend

- [ ] `types/vocabulary.ts`
- [ ] `lib/api.ts` — client topics/vocab
- [ ] `pages/vocabulary-page.tsx` — hub card "Từ vựng TOEIC"
- [ ] `pages/vocabulary-topics-page.tsx` — grid thumbnails
- [ ] `pages/vocabulary-topic-detail-page.tsx` — list + detail panel
- [ ] Form dialog thêm/sửa từ; confirm xóa
- [ ] `lib/vocabulary-highlight.ts` — highlight trong example
- [ ] Nút phát âm từ
- [ ] Loading / empty / error states

## Quyết định kỹ thuật

| Quyết định | Chi tiết |
|------------|----------|
| Nguồn dữ liệu | `TOEIC_BASE_URL=https://600tuvungtoeic.com` |
| Slug topic | Từ title EN, unique |
| `source_ref` | URL bài scrape; empty/custom cho từ user thêm |
| Delete guard | Từ có `source_ref` chứa domain 600tuvungtoeic → HTTP 403 |
| Thumbnail | Base64 inline SVG, không CDN |

## API contract

### GET `/api/topics`

Response: `TopicOut[]` sorted by `order_index`.

### GET `/api/topics/{topic_id}/vocabularies`

Response: `VocabularyOut[]` sorted alphabetically by `word`.

### POST `/api/topics/{topic_id}/vocabularies`

Body `VocabularyCreate`:

```json
{
  "word": "allocate",
  "phonetic": "/ˈæləkeɪt/",
  "meaning_en": "to distribute",
  "meaning_vi": "phân bổ",
  "word_type": "verb",
  "example_en": "We allocate resources carefully.",
  "example_vi": "Chúng tôi phân bổ nguồn lực cẩn thận."
}
```

### DELETE `/api/vocabularies/{id}`

- 204 nếu từ custom
- 403 nếu từ hệ thống scrape

## UI contract

- Hub `/vocabulary`: 1 card gradient pastel → `/vocabulary/toeic-topics`.
- Topics grid: `rounded-2xl` card, thumbnail, title VI ưu tiên.
- Detail: cột trái scroll list từ; cột phải sticky detail — phonetic, loại từ, nghĩa, ví dụ highlight.
- Nút "Thêm từ", "Sửa", "Xóa" trên panel detail.
- Copy UI tiếng Việt cho end user.

## Lệnh verify

```bash
curl http://127.0.0.1:5201/api/topics | head
curl http://127.0.0.1:5201/api/topics/1/vocabularies | head
# FE: mở /vocabulary/toeic-topics — thấy 50 chủ đề
# FE: mở /vocabulary/topics/1 — chọn từ, nghe phát âm, thêm từ custom, xóa custom OK
# FE: thử xóa từ scrape → thông báo lỗi phù hợp
```

## Backfill preview-spec sau khi code

- [database-ai-local-spec.md](../preview-dev/database-ai-local-spec.md) — bảng topics, vocabularies
- [backend-spec.md](../preview-dev/backend-spec.md) — endpoints phase 1
- [frontend-spec.md](../preview-dev/frontend-spec.md) — pages/components
- [project-master-spec.md](../preview-dev/project-master-spec.md) — phase 1 status

## Liên kết

- [backend-spec.md](../preview-dev/backend-spec.md)
- [frontend-spec.md](../preview-dev/frontend-spec.md)
- Phase tiếp: [2-Vocabulary_quiz.md](./2-Vocabulary_quiz.md)
