# Phase 0 — Cieot foundation

## Mục tiêu

Dựng monorepo tối thiểu: Docker Postgres, backend health, frontend App Shell dark pastel. **Không** scaffold module nghiệp vụ (Vocabulary/Listening/Quiz).

## Phạm vi

### In scope

- Cấu trúc thư mục `frontend/`, `backend/`, `docker/`
- `start.sh` — Postgres → Uvicorn → Vite (ports 5202, 5201, 5200)
- `.env.example` root + `backend/.env.example`
- Backend: FastAPI, CORS, kết nối DB, `GET /health`
- Frontend: Vite React TS, router, App Shell (SideMenu 3 item + AppToolbar + breadcrumb + Outlet)
- Theme dark-only pastel theo [DESIGN.md](../../DESIGN.md)
- Trang Dashboard placeholder chào mừng

### Out of scope

- Bảng nghiệp vụ, seed TOEIC/BBC
- API `/api/topics`, listening, quiz
- Ollama integration
- Zustand / theme toggle

## Checklist triển khai

### Infra

- [ ] `docker/docker-compose.yml` — Postgres 16, port 5202, user/db `cieot`
- [ ] `start.sh` — port cleanup, venv backend, npm install FE
- [ ] `.env.example` — biến môi trường tham chiếu

### Backend

- [ ] `backend/requirements.txt` — fastapi, uvicorn, sqlalchemy, psycopg, pydantic-settings
- [ ] `backend/app/config.py` — Settings (`database_url`, `app_port`)
- [ ] `backend/app/database.py` — engine, SessionLocal, Base
- [ ] `backend/app/main.py` — FastAPI, CORS, startup `create_all` (chưa seed), `GET /health`
- [ ] `backend/.env.example`

### Frontend

- [ ] `frontend/package.json` — react 19, vite 8, tailwind 4, react-router 7, lucide
- [ ] `frontend/vite.config.ts` — port 5200, alias `@`, proxy `/api`
- [ ] `frontend/src/index.css` — token dark pastel (không block `:root` light)
- [ ] `frontend/src/layouts/app-layout.tsx` — nav: Dashboard, Listening, Vocabulary
- [ ] `frontend/src/components/app-breadcrumb.tsx` — breadcrumb từ route
- [ ] `frontend/src/pages/dashboard-page.tsx` — welcome, link gợi ý Vocabulary (disabled/placeholder route OK)
- [ ] `frontend/src/router.tsx` — routes shell only
- [ ] `<html class="dark">` hoặc tương đương

## Quyết định kỹ thuật

| Quyết định | Lý do |
|------------|-------|
| Port 5200/5201/5202 | Giữ y hệt v1, tránh xung đột doc |
| Dark-only | Yêu cầu sản phẩm v2; bỏ ThemeToggle |
| SideMenu 3 module | Dashboard, Listening, Vocabulary |
| Không Alembic phase 0 | Port pattern v1 |
| Guest user placeholder trên toolbar | Chuẩn bị phase sau, chưa gọi API user |

## UI contract (phase 0)

- Sidebar rộng 280px, logo "Cieot", subtitle tiếng Việt ngắn.
- Nav item active: nền `--primary`, chữ `--primary-foreground`.
- Toolbar: "Chào mừng trở lại" + badge "Người dùng" (không lộ guest-user technical id trừ khi dev debug).
- Dashboard: card pastel gradient, text hướng dẫn bắt đầu từ Vocabulary (phase 1).

## Lệnh verify

```bash
# Từ root repo
./start.sh
# Hoặc từng bước:
docker compose -f docker/docker-compose.yml up -d
curl http://127.0.0.1:5201/health          # {"status":"ok"}
cd frontend && npm run build               # TypeScript + Vite build OK
cd frontend && npm run dev -- --port 5200  # UI shell dark, 3 nav items
```

Postgres: `docker compose -f docker/docker-compose.yml exec postgres pg_isready -U cieot -d cieot`

## Backfill preview-spec sau khi code

- [project-master-spec.md](../preview-dev/project-master-spec.md) — đánh dấu phase 0 done
- [frontend-spec.md](../preview-dev/frontend-spec.md) — cập nhật path file thực tế nếu khác
- [backend-spec.md](../preview-dev/backend-spec.md) — health endpoint
- [database-ai-local-spec.md](../preview-dev/database-ai-local-spec.md) — compose path

## Liên kết

- [project-master-spec.md](../preview-dev/project-master-spec.md)
- [DESIGN.md](../../DESIGN.md)
