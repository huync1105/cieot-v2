# CI/CD Local Jenkins Spec — Cieot v2

## Trạng thái

**Chưa áp dụng.** Cieot v1 không có pipeline CI/CD. v2 giữ stub tài liệu để agent không tự triển khai Jenkins sớm.

## Dự kiến (tham chiếu, không implement trong phase 0–3)

| Bước | Mô tả |
|------|-------|
| Lint FE | `cd frontend && npm run lint` |
| Build FE | `cd frontend && npm run build` |
| Test BE | `cd backend && pytest` |
| Integration | Docker Compose Postgres + smoke `curl /health` |

## Out of scope hiện tại

- Jenkinsfile, GitHub Actions, deploy production
- Docker image cho backend/frontend (chỉ Postgres container)

## Liên kết

- [project-master-spec.md](./project-master-spec.md)
