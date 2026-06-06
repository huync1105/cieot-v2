# ✅ FE Code Review Checklist (Best Practices 2026)

## 1. 📦 Code Structure & Architecture
- [ ] Component được **tách nhỏ theo single responsibility**
- [ ] Không để toàn bộ logic trong 1 file/component lớn
- [ ] Có phân tầng rõ ràng: `UI / Business logic / API layer`
- [ ] Tái sử dụng component thay vì duplicate code
- [ ] Hooks / composables được tách riêng (nếu dùng React/Vue)
- [ ] Không đặt logic nghiệp vụ trực tiếp trong UI render

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