# Frontend Spec — Cieot v2

## Stack

- **Vite** `^8.0.9` + **React** `^19.2.5` + **TypeScript** `~6.0.2`
- **React Router** v7 (`react-router-dom` `^7.14.2`) — SPA
- **Tailwind CSS** v4 (`@tailwindcss/vite` `^4.2.4`)
- **Lucide React** `^1.8.0` — icon sidebar, toolbar, nút hành động
- **TipTap** `^3.23.6` — rich-text editor (phase 3, grade modal)
- **react-markdown** + **remark-gfm** — hiển thị markdown
- **clsx**, **tailwind-merge**, **class-variance-authority** — utility styling (không bắt buộc folder `components/ui/` như shadcn; v1 dùng component inline)
- Dev server port: **5200** (`strictPort` khuyến nghị)

**Không dùng:** Zustand theme store (v2 dark-only, không toggle light/dark).

## Theme

- **Dark-only** — `class="dark"` cố định trên `<html>` hoặc chỉ định nghĩa token dark trong CSS (không light mode, không `ThemeToggle`).
- Token CSS: xem [DESIGN.md](../../DESIGN.md) và `frontend/src/index.css` (port từ v1 `.dark` block).
- Font: **Inter**, system fallback.

## Cấu trúc thư mục

```
frontend/src/
├── main.tsx
├── App.tsx
├── router.tsx
├── index.css                 # Tailwind + CSS variables dark pastel
├── layouts/
│   └── app-layout.tsx        # SideMenu + AppToolbar + breadcrumb + Outlet
├── pages/
│   ├── dashboard-page.tsx
│   ├── vocabulary-page.tsx           # Hub module Vocabulary
│   ├── vocabulary-topics-page.tsx    # Danh sách 50 chủ đề
│   ├── vocabulary-topic-detail-page.tsx
│   ├── listening-page.tsx            # Hub module Listening
│   ├── listening-passive-page.tsx
│   └── listening-passive-detail-page.tsx
├── components/
│   ├── app-breadcrumb.tsx
│   ├── vocabulary/
│   │   └── topic-quiz-modal.tsx      # Phase 2
│   └── listening/
│       ├── passive-listening-transcribe-modal.tsx
│       ├── passive-listening-grade-modal.tsx
│       ├── passive-listening-attempt-row.tsx
│       ├── passive-listening-misheard-phrases-table.tsx
│       ├── rich-text-editor.tsx
│       ├── formatted-note-content.tsx
│       ├── listening-correction-mark.ts
│       ├── plain-text-to-html.ts
│       └── selection-edit-bubble.tsx
├── lib/
│   ├── api.ts                # fetch wrapper, GUEST_USER_ID
│   ├── utils.ts              # cn(), helpers
│   └── vocabulary-highlight.ts  # highlight từ trong example
└── types/
    ├── vocabulary.ts
    └── listening.ts
```

## Layout shell (phase 0+)

- Grid: sidebar **280px** + main column; mobile stack 1 cột.
- **SideMenu** (trái): logo Cieot + nav items:
  - Dashboard `/` — icon `LayoutDashboard`
  - Listening `/listening` — icon `Headphones`
  - Vocabulary `/vocabulary` — icon `NotebookPen`
- **AppToolbar** (trên main): tiêu đề chào + badge user placeholder (`guest-user` / text "Người dùng").
- **Breadcrumb** sticky phía trên content scroll.
- **ScrollContent:** `overflow-y-auto`, padding `p-6`.

Quy ước AGENTS: mọi **nút mới** trên AppToolbar phải có icon Lucide đặt **trước** label, tiếng Việt.

## Routes

| Path | Page | Phase |
|------|------|-------|
| `/` | Dashboard (welcome, link tới Vocabulary) | 0 |
| `/vocabulary` | Hub — card dẫn tới TOEIC topics | 1 |
| `/vocabulary/toeic-topics` | Grid 50 chủ đề + thumbnail | 1 |
| `/vocabulary/topics/:topicId` | Master-detail từ vựng + CRUD | 1–2 |
| `/listening` | Hub — card Passive Listening | 3 |
| `/listening/passive` | Danh sách bài BBC | 3 |
| `/listening/passive/:lessonId` | Chi tiết bài + notes | 3 |

## Module Vocabulary (phase 1)

### Hub `/vocabulary`

- Card gradient pastel (`pastel-gradient`) dẫn tới `/vocabulary/toeic-topics`.
- Label tiếng Việt: "Từ vựng TOEIC".

### Topics list `/vocabulary/toeic-topics`

- Grid card: thumbnail base64, `title_vi` / `title_en`, link tới detail.
- Loading skeleton, empty state nếu chưa seed.

### Topic detail `/vocabulary/topics/:topicId`

- Layout 2 cột: danh sách từ (trái) + panel chi tiết (phải).
- Chi tiết: word, phonetic, meaning EN/VI, word_type, example EN/VI, highlight từ trong example, nút phát âm (audio_url hoặc `SpeechSynthesis`).
- CRUD từ tùy chỉnh: thêm/sửa/xóa (xóa bảo vệ từ scrape gốc — `source_ref` từ 600tuvungtoeic).
- Phase 2: nút "Làm quiz" mở `TopicQuizModal`.

## Module Vocabulary Quiz (phase 2)

- Modal full-screen hoặc overlay lớn, flow 3 bước/từ: `fillAnswer` → `compareOriginal` → `writeExample`.
- Gọi API quiz session; hiển thị kết quả pass/fail (ngưỡng 75%), gợi ý từ ôn lại.
- Bước `writeExample`: gọi `grade-example`; nút gợi ý câu tiếng Việt qua `sentence-hint`.
- Text UI tiếng Việt; không hiển thị tên model Ollama.

## Module Listening (phase 3)

### Hub `/listening`

- Card dẫn tới Passive Listening.

### Passive list `/listening/passive`

- Danh sách bài: thumbnail, title, episode_code, published_date, summary.

### Passive detail `/listening/passive/:lessonId`

- 4 section: Introduction, This week's question, Vocabulary, Transcript.
- Audio player; nút transcribe → modal nhập dictation.
- Panel attempts: nhiều lần ghi chú; tạo attempt mới; grade modal (TipTap) so sánh với transcript.
- `.listening-correction` highlight chỗ nghe sai; bảng misheard phrases; bookmark trong `corrections_json`.
- `localStorage` key audio: `cieot-passive-listening-audio:{userId}:{lessonId}:{attemptNo}`.

## API client

- File `lib/api.ts`: base URL từ env `VITE_API_URL` hoặc proxy `/api`.
- Hằng `GUEST_USER_ID = "guest-user"` — gửi trên mọi request cần `user_id`.
- Không React Query/SWR — `fetch` trực tiếp (giữ pattern v1).

## Dev proxy

`vite.config.ts`:

```ts
server: {
  port: 5200,
  proxy: {
    "/api": { target: "http://127.0.0.1:5201", changeOrigin: true },
  },
}
```

Alias `@` → `./src`.

## Dependencies (runtime)

| Gói | Mục đích |
|-----|----------|
| react, react-dom | UI |
| react-router-dom | Routing |
| lucide-react | Icons |
| @tiptap/react, starter-kit, pm | Rich text (listening grade) |
| react-markdown, remark-gfm | Markdown render |
| clsx, tailwind-merge, cva | Class utilities |

## Liên kết

- [project-master-spec.md](./project-master-spec.md)
- [backend-spec.md](./backend-spec.md)
- [DESIGN.md](../../DESIGN.md)
