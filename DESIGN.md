---
version: v2
name: Cieot Dark Pastel
description: Design system dark-only cho Cieot v2 — nền navy đậm, chữ sáng dịu, accent pastel cam/xanh/tím/vàng cho học TOEIC. Không light mode.

colors:
  background: "#0b1220"
  foreground: "#dbeafe"
  card: "#111827"
  card-foreground: "#e2e8f0"
  primary: "#fb923c"
  primary-foreground: "#1c1917"
  secondary: "#38bdf8"
  secondary-foreground: "#e0f2fe"
  tertiary: "#a78bfa"
  tertiary-foreground: "#f5f3ff"
  quaternary: "#fbbf24"
  quaternary-foreground: "#fffbeb"
  soft-blue: "#0f2740"
  soft-purple: "#2e2250"
  soft-yellow: "#3d3417"
  soft-orange: "#40200f"
  muted: "#1e293b"
  muted-foreground: "#94a3b8"
  border: "#334155"
  destructive: "#f87171"
  destructive-muted: "rgb(127 29 29 / 0.45)"
  success: "#22c55e"

typography:
  font-family: Inter, ui-sans-serif, system-ui, sans-serif
  display:
    fontSize: 32px
    fontWeight: 700
    lineHeight: 1.15
  heading-1:
    fontSize: 24px
    fontWeight: 700
    lineHeight: 1.25
  heading-2:
    fontSize: 20px
    fontWeight: 600
    lineHeight: 1.3
  body:
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5
  body-sm:
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.43
  caption:
    fontSize: 12px
    fontWeight: 500
    lineHeight: 1.33

rounded:
  sm: 8px
  md: 12px
  lg: 16px
  xl: 20px
  full: 9999px

spacing:
  xs: 8px
  sm: 12px
  md: 16px
  lg: 24px
  xl: 32px

layout:
  sidebar-width: 280px
  toolbar-height: 72px
  content-max-width: 1200px
---

# Cieot v2 — Design System (Dark Pastel)

Giao diện **chỉ dark mode**. Palette pastel trên nền navy giúp mắt nghỉ khi học lâu; accent cam là CTA chính, xanh/tím/vàng phân biệt module và trạng thái.

Agent **bắt buộc** tuân thủ token dưới đây — không hardcode màu lệch design system.

## CSS variables

Đặt trong `frontend/src/index.css` (không block `:root` light):

```css
:root {
  --background: #0b1220;
  --foreground: #dbeafe;
  --card: #111827;
  --card-foreground: #e2e8f0;
  --primary: #fb923c;
  --primary-foreground: #1c1917;
  --secondary: #38bdf8;
  --secondary-foreground: #e0f2fe;
  --tertiary: #a78bfa;
  --tertiary-foreground: #f5f3ff;
  --quaternary: #fbbf24;
  --quaternary-foreground: #fffbeb;
  --soft-blue: #0f2740;
  --soft-purple: #2e2250;
  --soft-yellow: #3d3417;
  --soft-orange: #40200f;
  --muted: #1e293b;
  --muted-foreground: #94a3b8;
  --border: #334155;
}
```

Dùng trong Tailwind: `bg-[var(--card)]`, `text-[var(--foreground)]`, v.v.

## Typography

- Font: **Inter** (Google Fonts hoặc system).
- Tiêu đề trang: `text-2xl font-bold`.
- Section: `text-lg font-semibold`.
- Body: `text-sm` / `text-base`.
- Muted meta: `text-[var(--muted-foreground)]`.

## Spacing & radius

- Card/panel: `rounded-xl` hoặc `rounded-2xl`, padding `p-4`–`p-6`.
- Nút nav sidebar: `rounded-xl px-4 py-3`.
- Input: `rounded-lg border bg-[var(--card)]`.
- Gap grid module hub: `gap-4` / `gap-6`.

## App Shell

```
┌──────────────┬────────────────────────────────────┐
│ SideMenu     │ AppToolbar                         │
│ 280px        ├────────────────────────────────────┤
│              │ Breadcrumb (sticky)                │
│ Dashboard    ├────────────────────────────────────┤
│ Listening    │                                    │
│ Vocabulary   │ ScrollContent / Outlet             │
│              │                                    │
└──────────────┴────────────────────────────────────┘
```

### SideMenu

- Nền `--card`, border phải `--border`.
- Logo "Cieot" màu `--primary`.
- Nav item: icon Lucide 16px + label; active = nền `--primary`, chữ `--primary-foreground`; hover = `--muted`.

### AppToolbar

- Nền `--card`, border dưới.
- Badge user bên phải (rounded-xl, nền `--muted`).
- **Mọi nút mới:** icon Lucide **trước** label, tiếng Việt.

### Breadcrumb

- Sticky trên vùng scroll, nền `--background` 95% + backdrop blur.
- Separator `/` hoặc chevron muted.

## Components

### Module hub card

- Class `pastel-gradient` — gradient 135° từ soft-blue → soft-purple → soft-orange (color-mix nhẹ).
- `rounded-2xl`, border `--border`, hover shadow nhẹ.
- Title `--heading-2`, mô tả `--muted-foreground`.

```css
.pastel-gradient {
  background-image: linear-gradient(
    135deg,
    color-mix(in oklab, var(--soft-blue) 85%, #0b1220 15%) 0%,
    color-mix(in oklab, var(--soft-purple) 85%, #0b1220 15%) 45%,
    color-mix(in oklab, var(--soft-orange) 85%, #0b1220 15%) 100%
  );
}
```

### Primary button

- Nền `--primary`, chữ `--primary-foreground`, `rounded-xl`, font-medium.
- Hover: brightness 110%; disabled: opacity 50%, `cursor-not-allowed`.

### Secondary button

- Nền `--muted` hoặc transparent + border `--border`.
- Chữ `--foreground`.

### Topic / lesson card

- Nền `--card`, thumbnail `rounded-lg`, title `--card-foreground`.
- Master-detail: list trái scroll; panel phải sticky top.

### Modal (quiz, transcribe, grade)

- Overlay `bg-black/60`.
- Panel `--card`, `rounded-2xl`, max-width `lg`/`xl`, padding `p-6`.
- Header `--heading-2`, nút đóng góc phải.

### Listening correction mark

Highlight đoạn nghe sai trong grade modal:

```css
.listening-correction {
  background-color: rgb(127 29 29 / 0.45);
  color: rgb(252 165 165);
  border-radius: 0.125rem;
  padding: 0 0.125rem;
}
```

Hover tooltip cụm đúng: nền `#22c55e`, chữ trắng.

## Trạng thái UI

| Trạng thái | Pattern |
|------------|---------|
| Loading | Skeleton `--muted` animate-pulse |
| Empty | Icon Lucide muted + câu tiếng Việt gợi ý hành động |
| Error | Border/text `--destructive`, message tiếng Việt, không stack trace |
| Success quiz | Accent `--quaternary` hoặc `--secondary` cho badge đậu |

## Copy & accessibility

- Text end-user **tiếng Việt**; không lộ Ollama, model, URL nội bộ, cookie, token.
- `button:not(:disabled) { cursor: pointer }`.
- Contrast: foreground trên background đạt đọc được; primary cam trên nền tối dùng cho CTA lớn.
- Focus ring: `ring-2 ring-[var(--secondary)]` cho input/button.

## Do / Don't

**Do**

- Dùng CSS variables cho mọi màu nền/chữ/viền.
- Giữ sidebar + toolbar + breadcrumb xuyên suốt app.
- Modal cho flow dài (quiz, chấm listening).

**Don't**

- Thêm light theme hoặc theme toggle.
- Hardcode `#fff` / `#000` ngoài token.
- Thay master-detail bằng select phẳng khi đã có pattern cây/list (AGENTS).
- Hiển thị tên model AI trên UI.

## Liên kết

- [frontend-spec.md](./docs/preview-dev/frontend-spec.md)
- [AGENTS.md](./AGENTS.md)
