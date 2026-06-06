# ✅ FE Code Review Checklist (Best Practices 2026)

## 1. 📦 Code Structure & Architecture
- [ ] Component được **tách nhỏ theo single responsibility**
- [ ] Không để toàn bộ logic trong 1 file/component lớn
- [ ] Có phân tầng rõ ràng: `UI / Business logic / API layer`
- [ ] Tái sử dụng component thay vì duplicate code
- [ ] Hooks / composables được tách riêng (nếu dùng React/Vue)
- [ ] Không đặt logic nghiệp vụ trực tiếp trong UI render
- [ ] **Số file thay đổi/tạo mới phù hợp quy mô feature** (xem bảng dưới); nếu lệch nhiều, đã giải thích hoặc tách PR
- [ ] **Mỗi file một trách nhiệm rõ ràng** — không trộn UI + API + types + constants trong cùng file (xem bảng loại file bên dưới)

### 1 file = 1 responsibility

Mỗi file chỉ đảm nhận **một vai trò** trong feature. Tách file khi trách nhiệm khác nhau; gom chung chỉ khi đoạn code quá nhỏ (< ~20 dòng) và gắn chặt một component duy nhất.

| Loại file | Trách nhiệm | Ví dụ / quy ước đặt tên |
|-----------|-------------|-------------------------|
| **Component UI** | Render, layout, event UI; gọi hook/service, **không** gọi `fetch` trực tiếp | `TopicListView.tsx`, `VocabFormDialog.tsx` |
| **Hook / logic** | State, side effect, orchestration flow; **không** JSX (trừ hook render-less hiếm) | `useTopicQuiz.ts`, `usePassiveListeningNotes.ts` |
| **Service / API** | HTTP/client, map request/response; **không** UI state | `vocabularyApi.ts`, `listeningApi.ts` |
| **Types** | Interface, type alias, enum domain; **không** runtime logic | `types/vocabulary.ts` |
| **Utils** | Pure function tái sử dụng; **không** phụ thuộc React/DOM nếu tránh được | `vocabulary-highlight.ts`, `formatDate.ts` |
| **Constants** | Hằng số, config tĩnh, magic string tập trung; **không** logic | `quizConstants.ts`, `GUEST_USER_ID` trong `lib/constants.ts` |

**Checklist nhanh khi review file:**

- [ ] File `.tsx` component: chủ yếu JSX + wiring props/callback; logic nặng → hook riêng.
- [ ] File `*Api.ts`: chỉ fetch + typing response; không `useState` / `useEffect`.
- [ ] File `types/*.ts`: export type/interface; không import component.
- [ ] Constants không rải rác trong component — tách `constants.ts` hoặc file domain khi dùng lại ≥2 nơi.

**Red flags:**

- Page 300+ dòng vừa CRUD vừa gọi API vừa khai báo type inline.
- `utils` import React hoặc gọi API.
- Cùng một interface copy-paste ở nhiều file thay vì `types/`.

### Quy mô feature và số file (khuyến nghị)

Áp dụng khi **planning** và **review** — đếm file FE liên quan trực tiếp (page, component, hook, `lib/*Api`, type, test, style module của feature đó). Không phải hard limit; mục tiêu là tránh gom quá nhiều vào ít file hoặc tách quá mảnh không cần thiết.

| Quy mô | Ví dụ | Số file FE (ước lượng) |
|--------|--------|-------------------------|
| **Nhỏ** | CRUD đơn giản (list + form/dialog) | **~3–6 files** |
| **Vừa** | Flow nhiều bước, vài component con, API layer riêng | **~6–12 files** |
| **Lớn** | Module/business phức tạp (master-detail, modal chain, nhiều sub-flow) | **~10–20 files** (có thể hơn nếu kiến trúc domain-driven rõ ràng) |

**Gợi ý phân file thường:**

- **Nhỏ:** `page` + `types` + `*Api.ts` + 1–2 component (form/list/dialog).
- **Vừa:** thêm hook/state helper, 2–4 component domain, có thể tách `*FormDialog` / `*ListView`.
- **Lớn:** folder `components/<feature>/`, tách presentational/container, utils pure, test file riêng — cấu trúc thư mục phản ánh domain, không tách file “cho đủ số lượng”.

**Red flags khi review:**

- Feature **nhỏ** nhưng **>8 file** mà không có lý do (duplicate, over-abstraction).
- Feature **lớn** nhưng **<5 file** — logic/UI dồn vào 1–2 file dài.
- Một PR gom nhiều feature không liên quan → nên tách PR theo quy mô trên.

---

## 2. 🧠 Readability & Maintainability
- [ ] Code dễ đọc, dễ hiểu, naming rõ ràng
- [ ] Không có magic number / magic string
- [ ] Logic phức tạp được tách function nhỏ
- [ ] Tránh nested condition quá sâu
- [ ] File không quá dài (khuyến nghị < 300–400 lines)

---

## 3. 🏷️ Type Safety
- [ ] **Khai báo type đầy đủ cho tất cả props, state, function params**
- [ ] Không dùng `any` (hoặc hạn chế tối đa)
- [ ] Interface/type được reuse thay vì redefine
- [ ] API response có typing rõ ràng
- [ ] Guard type khi xử lý dữ liệu external (API, storage)

---

## 4. 🧩 Component Design
- [ ] Component được tách theo domain / feature
- [ ] Presentational vs Container pattern rõ ràng (nếu áp dụng)
- [ ] Props tối giản, không truyền quá nhiều props không cần thiết
- [ ] Không “prop drilling” quá sâu (dùng context/state management khi cần)
- [ ] Side effects không nằm trong render logic

---

## 5. ⚙️ State Management
- [ ] State được đặt đúng scope (local vs global)
- [ ] Tránh overuse global state
- [ ] Derived state không bị duplicate
- [ ] Không mutate state trực tiếp
- [ ] Async state handling rõ ràng (loading/error/success)

---

## 6. 🌐 API & Data Handling
- [ ] API call được tách riêng khỏi UI layer
- [ ] Có xử lý loading / error state đầy đủ
- [ ] Có retry / fallback nếu cần
- [ ] Validate response trước khi sử dụng
- [ ] Không assume data luôn tồn tại (safe access)

---

## 7. 🧪 Testing
- [ ] Có unit test cho logic quan trọng
- [ ] Có integration test cho flow chính
- [ ] Component test không phụ thuộc implementation detail
- [ ] Mock API hợp lý
- [ ] Coverage tối thiểu theo team standard

---

## 8. 🚀 Performance
- [ ] Tránh re-render không cần thiết
- [ ] Memoization đúng chỗ (useMemo / useCallback / memo)
- [ ] Lazy load component / route khi phù hợp
- [ ] Optimize list rendering (virtualization nếu cần)
- [ ] Không tạo object/function inline không cần thiết

---

## 9. 🔒 Security
- [ ] Không render trực tiếp HTML unsafe (XSS protection)
- [ ] Escape dữ liệu user input
- [ ] Không expose secret keys trên FE
- [ ] Validate input phía client + server
- [ ] Secure storage (tránh localStorage với sensitive data nếu không cần)

---

## 10. 🎨 UI/UX Quality
- [ ] UI handle đầy đủ states: loading / empty / error
- [ ] Responsive trên mobile/tablet/desktop
- [ ] Accessibility (a11y) cơ bản: aria-label, keyboard navigation
- [ ] Không hardcode text (i18n nếu có)
- [ ] Consistent UI behavior

---

## 11. 🧾 Documentation & Comments
- [ ] Có comment giải thích logic phức tạp
- [ ] **Có comment giải thích đầy đủ cho function quan trọng**
- [ ] README / docs cập nhật nếu thay đổi lớn
- [ ] JSDoc / TSDoc cho public functions
- [ ] Comment không thừa, tránh “noise comment”

---

## 12. 🧱 Change Safety (Critical when modifying code)
- [ ] **Nếu chỉnh sửa hoặc bổ sung logic, đã check kỹ impact tới các phần cũ**
- [ ] Đảm bảo không làm hỏng logic hiện tại của feature liên quan
- [ ] Kiểm tra UI các component liên quan có bị ảnh hưởng không
- [ ] Verify edge cases cũ vẫn hoạt động đúng
- [ ] Run lại flow chính (happy path + critical path)
- [ ] Có test hoặc manual check cho phần bị ảnh hưởng

---

## 13. 📌 Required Checklist Items
- [ ] **Có comment giải thích hàm đầy đủ**
- [ ] **Khai báo type đầy đủ**
- [ ] **Tách component đầy đủ, không để tất cả logic trong 1 file**
- [ ] **1 file = 1 responsibility** (UI / hook / API / types / utils / constants tách đúng vai trò)
- [ ] **Số file FE khớp quy mô feature** (nhỏ ~3–6, vừa ~6–12, lớn ~10–20+) hoặc có lý do kiến trúc rõ ràng