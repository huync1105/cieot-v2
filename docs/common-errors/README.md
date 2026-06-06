# Common Errors — Cieot v2

Thư mục ghi lại lỗi lặp lại khi phát triển Cieot v2. Agent **bắt buộc đọc toàn bộ file `.md` trong thư mục này** trước mỗi lần implementation (theo [AGENTS.md](../../AGENTS.md)).

## Quy ước ghi lỗi

Mỗi file mới đặt tên ngắn gọn theo triệu chứng hoặc module, ví dụ: `uvicorn-stale-port-5201.md`.

Cấu trúc bắt buộc:

```markdown
# <Tiêu đề ngắn>

## Triệu chứng
Mô tả lỗi người dùng/dev gặp.

## Nguyên nhân gốc
Giải thích kỹ thuật.

## Cách sửa
Các bước cụ thể.

## Cách phòng tránh / verify
Lệnh hoặc checklist để không lặp lại.
```

## Trạng thái hiện tại

Chưa có lỗi được ghi — thư mục sẽ được backfill khi implement từng phase.
