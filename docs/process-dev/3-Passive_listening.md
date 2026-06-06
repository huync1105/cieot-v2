# Phase 3 — Passive Listening

## Mục tiêu

Module **Nghe thụ động** (BBC 6 Minute English): danh sách bài, chi tiết 4 section, dictation đa lần, grade modal TipTap so transcript, tải transcript PDF/HTML on-demand.

## Phạm vi

### In scope

- Bảng `listening_lessons`, `listening_note_attempts`
- Seed ~10 episode BBC (static + merge)
- API passive lessons, notes CRUD, download transcript
- UI hub Listening → passive list → lesson detail
- Transcribe modal, grade modal, misheard table, correction marks
- `localStorage` vị trí audio theo attempt
- TipTap rich-text editor

### Out of scope

- Search/filter nâng cao (backlog tương lai)
- Progress listening toàn cục

## Checklist triển khai

### Backend / DB

- [ ] Models `ListeningLesson`, `ListeningNoteAttempt`
- [ ] Schemas listening (list, detail, notes, transcript download)
- [ ] `bbc_listening_seed.py` — static episode data
- [ ] `transcript_format.py` — normalize PDF/HTML text
- [ ] `seed_passive_listening_data` — merge, giữ transcript đã download
- [ ] Routes:
  - `GET /api/listening/passive-lessons`
  - `GET /api/listening/passive-lessons/{lesson_id}`
  - `GET/POST/PUT notes`
  - `PUT .../formatted`
  - `POST .../download-transcript`
- [ ] Note rules: ensure attempt 1; POST tạo attempt_no tiếp theo
- [ ] PDF parse: pypdf + pdfplumber

### Frontend

- [ ] `types/listening.ts`
- [ ] `lib/api.ts` — listening endpoints
- [ ] `pages/listening-page.tsx` — hub card Passive Listening
- [ ] `pages/listening-passive-page.tsx` — lesson list
- [ ] `pages/listening-passive-detail-page.tsx` — sections + audio + attempts
- [ ] Components:
  - `passive-listening-transcribe-modal.tsx`
  - `passive-listening-grade-modal.tsx`
  - `passive-listening-attempt-row.tsx`
  - `passive-listening-misheard-phrases-table.tsx`
  - `rich-text-editor.tsx` (TipTap)
  - `formatted-note-content.tsx`
  - `listening-correction-mark.ts`, `plain-text-to-html.ts`, `selection-edit-bubble.tsx`
- [ ] CSS `.listening-correction` trong `index.css`
- [ ] Audio localStorage key: `cieot-passive-listening-audio:{userId}:{lessonId}:{attemptNo}`

## Routes FE

| Path | Mô tả |
|------|-------|
| `/listening` | Hub — card "Nghe thụ động" |
| `/listening/passive` | Danh sách bài |
| `/listening/passive/:lessonId` | Chi tiết + notes |

## API contract

### GET `/api/listening/passive-lessons`

`ListeningLessonListItemOut[]` — không có transcript full (list nhẹ).

### GET `/api/listening/passive-lessons/{lesson_id}`

`ListeningLessonDetailOut` — thêm `introduction`, `weekly_question`, `vocabulary`, `transcript`, `transcript_source_url`.

### GET `/api/listening/passive-lessons/{lesson_id}/notes?user_id=guest-user`

`ListeningNoteAttemptOut[]` — auto ensure attempt 1 nếu chưa có.

### POST `/api/listening/passive-lessons/{lesson_id}/notes`

Body: `{ "user_id": "guest-user" }` → tạo attempt mới (attempt_no = max + 1).

### PUT `/api/listening/passive-lessons/{lesson_id}/notes/{attempt_no}`

Body: `{ "user_id": "guest-user", "content": "raw dictation text..." }`

Lưu kèm vị trí audio từ FE localStorage (FE-only).

### PUT `/api/listening/passive-lessons/{lesson_id}/notes/{attempt_no}/formatted`

Body:

```json
{
  "user_id": "guest-user",
  "formatted_content": "<p>HTML from TipTap</p>",
  "corrections_json": "{\"bookmarks\":[...],\"misheard\":[...]}"
}
```

### POST `/api/listening/passive-lessons/{lesson_id}/download-transcript`

Response `ListeningTranscriptDownloadOut`:

```json
{
  "lesson_id": 1,
  "transcript_source_url": "https://...",
  "transcript_length": 4500,
  "status": "ok"
}
```

Fetch PDF/HTML từ BBC, parse, cập nhật `listening_lessons.transcript`.

## Note attempt rules

- Scope: `user_id + lesson_id + attempt_no` (unique).
- Lần đọc/ghi đầu: đảm bảo attempt `1` tồn tại.
- `content`: text dictation thô.
- `formatted_content`: HTML sau chấm/so sánh transcript trong grade modal.
- `corrections_json`: JSON bookmarks, misheard phrases (optional).

## Seed strategy

- Nguồn listing: BBC 6 Minute English feature page.
- Chi tiết episode: static trong `bbc_listening_seed.py` (tránh scrape runtime không ổn định).
- Boot merge: cập nhật metadata; **không ghi đè** transcript đã download user/server.

## UI contract

- List: card thumbnail, title, ngày, summary ngắn.
- Detail: audio player sticky; 4 accordion/section rõ ràng.
- Nút "Ghi chép" → transcribe modal (textarea).
- Nút "Chấm bài" → grade modal: TipTap editor, highlight `.listening-correction` (đỏ pastel dark), hover hiện cụm đúng.
- Bảng cụm nghe sai; lưu formatted + corrections.
- Nút "Tải transcript" nếu transcript rỗng — loading state, thông báo lỗi mạng bằng tiếng Việt.

## Known limitations (port v1)

- BBC pages đôi khi không fetch được server-side — seed static bù phần lớn.
- Một số bài seed có placeholder transcript cho đến khi user gọi download.
- `user_id` cố định `guest-user`.

## Lệnh verify

```bash
curl http://127.0.0.1:5201/api/listening/passive-lessons
curl http://127.0.0.1:5201/api/listening/passive-lessons/1
curl "http://127.0.0.1:5201/api/listening/passive-lessons/1/notes?user_id=guest-user"

curl -X POST http://127.0.0.1:5201/api/listening/passive-lessons/1/notes \
  -H "Content-Type: application/json" \
  -d '{"user_id":"guest-user"}'

curl -X POST http://127.0.0.1:5201/api/listening/passive-lessons/1/download-transcript

# FE: /listening/passive → chọn bài → nghe → ghi chép → chấm → thấy highlight
# FE: tạo attempt 2, reload — data persist
```

## Backfill preview-spec sau khi code

- [database-ai-local-spec.md](../preview-dev/database-ai-local-spec.md)
- [backend-spec.md](../preview-dev/backend-spec.md)
- [frontend-spec.md](../preview-dev/frontend-spec.md)
- [project-master-spec.md](../preview-dev/project-master-spec.md)

## Liên kết

- Phase trước: [2-Vocabulary_quiz.md](./2-Vocabulary_quiz.md)
- [DESIGN.md](../../DESIGN.md) — listening-correction styles
