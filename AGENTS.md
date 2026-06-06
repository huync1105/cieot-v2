# AGENTS Guidance - dev

## Mục tiêu

Đảm bảo mọi AI agent và contributor đọc đúng ngữ cảnh tài liệu trước khi bắt đầu code, giảm tràn context và tránh token rác.

## Quy tắc bắt buộc trước khi code tính năng

Trước mọi thay đổi liên quan đến code (frontend/backend/database/infra), AI **phải đọc** theo thứ tự:

1. `[docs/preview-dev/project-master-spec.md](./docs/preview-dev/project-master-spec.md)`
2. Tài liệu chuyên sâu tương ứng mảng công việc:
  - Frontend: `[docs/preview-dev/frontend-spec.md](./docs/preview-dev/frontend-spec.md)`; **luôn luôn code giao diện UI theo** `[DESIGN.md](./DESIGN.md)` (design system: màu, typography, spacing, bo góc, pattern component).
  - Backend: `[docs/preview-dev/backend-spec.md](./docs/preview-dev/backend-spec.md)`
  - Database + AI local: `[docs/preview-dev/database-ai-local-spec.md](./docs/preview-dev/database-ai-local-spec.md)`
  - CI/CD: `[docs/preview-dev/cicd-local-jenkins-spec.md](./docs/preview-dev/cicd-local-jenkins-spec.md)`
3. Khi xây dựng chức năng mới, đọc thêm toàn bộ file markdown trong thư mục `docs/process-dev`.
4. Luôn đọc toàn bộ file markdown trong thư mục `[docs/common-errors](./docs/common-errors/)` trước khi bắt đầu implementation để tránh lặp lại sai lầm cũ.

## Quy tắc thực thi trong quá trình code

- Không triển khai vượt ngoài phạm vi đã nêu trong tài liệu đặc tả hiện hành.
- Nếu phát sinh quyết định kỹ thuật mới, cập nhật lại tài liệu trước hoặc đồng thời với code.
- Nếu yêu cầu mới mâu thuẫn với đặc tả, ưu tiên xác nhận lại yêu cầu và sửa đặc tả trước.
- Sau mỗi phase có thay đổi kỹ thuật thực tế, bắt buộc backfill vào file preview-spec tương ứng (`backend-spec.md`, `database-ai-local-spec.md`, `frontend-spec.md`, ...), không chỉ ghi ở process docs.
- Nếu phase có thay đổi database (schema/table/index/FK/migration), bắt buộc cập nhật `docs/preview-dev/database-ai-local-spec.md` kèm:
  - danh sách bảng thay đổi,
  - quan hệ chính giữa các bảng,
  - chiến lược migration/index ở mức đặc tả.
- Với mọi hàm mới tạo hoặc hàm được chỉnh sửa, bắt buộc bổ sung comment chuẩn Javadoc `/** ... */` mô tả rõ:
  - Input (tham số, dữ liệu đầu vào),
  - Logic xử lý chính,
  - Output/kết quả trả về (và ngoại lệ nếu có),
  để giảm chi phí phân tích khi debug/fix bug ở các phase sau.
- Với mọi DTO/Entity/Model/Request/Response/type/interface mới (cả backend và frontend), bắt buộc chú thích cho từng field mới bằng comment phù hợp ngôn ngữ, sử dụng tiếng việt có dấu:
  - Backend: dùng `/** ... */` ngay trên field.
  - Frontend TypeScript/JavaScript: dùng comment rõ nghĩa ngay trên field/property.
  - Nội dung tối thiểu phải nêu mục đích field, ý nghĩa dữ liệu, và format/ràng buộc quan trọng (nếu có).
- Mỗi lần làm chức năng mới, tạo 1 file markdown mới trong `docs/process-dev` theo format `<phase number>-<Tên chức năng>` (ví dụ: `1-User_login`), tóm tắt đã làm những gì.
- Nếu bổ sung chức năng mới cho chức năng đã có, sau khi sửa code phải cập nhật lại file markdown phase cũ của chức năng đó, ghi rõ các phần thêm mới/chỉnh sửa.
- Nếu yêu cầu chỉ ở mức "dựng codebase/foundation", AI chỉ được tạo khung kỹ thuật tối thiểu (framework, config, folder layer tổng quát), **không** được scaffold sẵn module nghiệp vụ, màn hình nghiệp vụ, hoặc mock business flow khi chưa được yêu cầu rõ.
- Trước khi tạo thêm bất kỳ thành phần ngoài phạm vi tối thiểu, AI phải xác nhận lại phạm vi với người yêu cầu.
- Nếu trong quá trình làm phát sinh lỗi (code/config/build/deploy), AI phải cập nhật ngay vào file phù hợp trong thư mục `[docs/common-errors](./docs/common-errors/)` với nguyên nhân gốc và cách phòng tránh/verify.
- **Luôn luôn code giao diện UI theo** `[DESIGN.md](./DESIGN.md)` (component, layout, màn hình, style): dùng đúng token màu, typography, spacing, bo góc và pattern component đã định nghĩa; không tự ý hardcode giá trị visual lệch design system. Nếu yêu cầu mới mâu thuẫn `DESIGN.md`, ưu tiên xác nhận lại và cập nhật `DESIGN.md` đồng thời với code.
- Khi code UI, không hiển thị chi tiết kỹ thuật triển khai nội bộ (token, cookie httpOnly, interceptor, cơ chế auth backend, v.v.) trong nội dung dành cho end user. Text UI phải tập trung vào hành động/ngữ cảnh nghiệp vụ mà người dùng cần thực hiện.
- **Giữ pattern UI/UX đã có:** khi chỉnh sửa component/màn hình đã tồn tại, đọc process doc + preview-spec phase liên quan trước; **chỉ thay đổi đúng phạm vi yêu cầu**, không thay thế pattern cũ (ví dụ cây expand/collapse → select phẳng) trừ khi user xác nhận. Nếu bổ sung hành vi (ví dụ dropdown thu gọn), **gắn thêm** lên pattern cũ thay vì xóa hành vi đã có.
- Mỗi **nút mới** trên `AppToolbar` bắt buộc có **icon Lucide đặt trước** label (đầu nút), chọn icon phù hợp ngữ cảnh (ví dụ chat AI → `Bot`, tin nhắn → `MessageCircle`). Label tiếng Việt; không lộ tên model, URL hay cơ chế nội bộ cho end user.

## Quy tắc logic, thuật toán và best practices

- **Đơn giản, ngắn gọn**: ưu tiên luồng đọc được, ít nhánh/phụ thuộc không cần thiết; tránh abstraction hoặc design pattern khi chỉ phục vụ một chỗ duy nhất; không “chuẩn bị cho tương lai” ngoài những gì đặc tả/process đã nêu. Nội dung chi tiết hành vi (thinking trước khi code, surgical changes…) đồng thuận với các rule trong `[.cursor/rules](./.cursor/rules/)` (ví dụ guideline behavioral của dự án).
- **Best practices theo stack**: trong phạm vi task, làm đúng chuẩn ngôn ngữ và framework đang dùng (đặt tên, xử lý async/lỗi, idioms TypeScript/Java, React/Next, v.v.—ưu tiên tài liệu chính thức); không vi phạm guideline của toolchain (ESLint, compiler, formatter đã cấu hình).
- **Độ phức tạp thuật toán**: trước khi finalize, đánh giá **time / space complexity** và khi có ý nghĩa thì chi phí **I/O** hoặc **round-trip** database/API; chọn giải pháp đủ nhanh với khối lượng input thực tế của hệ thống, đơn giản hóa khi chưa cần tối ưu sớm. Nếu phải tối ưu hoặc dùng cấu trúc đặc biệt (`Map`, `Set`, chỉ mục, batch…), **ghi ngắn** trong comment Javadoc của hành vi liên quan hoặc trong process doc của phase để reviewer và agent sau nắm được lý do và độ phức tạp mong đợi.
- **Cân bằng với đặc tả**: không vì “tối giản” mà bỏ qua validation, tính toàn vẹn dữ liệu/transaction, hay hợp đồng API/request-response đã nêu trong preview-spec/process.

## Quy ước commit Git sau khi hoàn thành một phase

Sau khi làm xong một phase và cần `git commit`, dùng **một** trong các khung sau. Tiền tố `<feat>`, `<fix>`, `<front>`, `<back>`, `<config>` (**có** cặp `<` `>` như ví dụ) là **nguyên văn** trong tiêu đề commit; chỉ thay các phần tên người / tên tính năng / tên phần bổ sung (không để placeholder kiểu chữ `<developer_name>` trong commit thực tế).


| Tình huống                                                                                                                            | Khung tiêu đề commit                                                |
| ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Chức năng mới **fullstack** (backend + frontend)                                                                                      | `<feat>(<developer_name>): <feature_name>`                          |
| Sửa lỗi (bugfix)                                                                                                                      | `<fix>(<developer_name>): Fix <bug_name>`                           |
| Chỉ **frontend / UI** (không có thay đổi luồng backend tương ứng trong cùng commit)                                                   | `<front>(<developer_name>): Code UI <layout_name hoặc module_name>` |
| Chỉ **backend**                                                                                                                       | `<back>(<developer_name>): Flow <logic_name hoặc flow_name>`        |
| **Cấu hình** (config tool/CI/env, v.v.) hoặc **sửa tài liệu phục vụ agent** (AGENTS.md, preview-spec, process docs, common-errors, …) | `<config>(<developer_name>): Bổ sung <additional_part_name>`        |


**Ví dụ (sau khi thay placeholder):**

- `<feat>(huync1105): Đăng nhập SSO`
- `<fix>(huync1105): Fix không refresh token khi hết hạn`
- `<front>(huync1105): Code UI trang Dashboard`
- `<back>(huync1105): Flow đồng bộ vai trò từ Keycloak`
- `<config>(cursor): Bổ sung quy ước commit trong AGENTS.md`

## Quy tắc tối ưu context

- Luôn nạp tài liệu theo đúng mảng công việc, tránh nạp toàn bộ tài liệu không liên quan.
- Tóm tắt các quyết định kỹ thuật chính thành checklist ngắn trước khi bắt đầu implementation.
- Hạn chế suy diễn ngoài tài liệu khi chưa có xác nhận từ yêu cầu chính thức.

