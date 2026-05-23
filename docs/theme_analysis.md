# 🎨 Phân tích Theme Architecture — HRM Frontend

> Cập nhật: 2026-04-17

## Tổng quan

| Metric                          | Giá trị                               |
| ------------------------------- | ------------------------------------- |
| Tổng số components (`.vue`)     | 78 files                              |
| CSS file duy nhất               | `src/styles/tailwind.css` (~340 dòng) |
| Số semantic tokens              | 53 tokens (light + dark)              |
| Components dùng semantic tokens | 78/78 files (100%)                    |
| Hardcoded hex còn lại           | 0                                     |

```
Centralization Score: ██████████ 100%
```

---

## Cơ chế hoạt động

> [!IMPORTANT]
> **`@theme` là directive của Tailwind CSS v4**, không phải tên tự đặt — KHÔNG thể đổi thành `@light` hay tên khác.

### Light Mode = Mặc định (không cần class)

```css
/* ☀️ LIGHT MODE — Default tokens */
@theme {
  --color-surface: #ffffff;
  --color-primary: #3b82f6;
  /* ... 53 tokens */
}
```

- `@theme { }` khai báo CSS vars ở `:root` level → **luôn có hiệu lực**
- Đồng thời **đăng ký** tokens để Tailwind generate utility classes (`bg-surface`, `text-foreground`...)
- Nếu bỏ ra khỏi `@theme` → utility classes sẽ **không hoạt động**

### Dark Mode = Override bằng class `.dark`

```css
/* 🌙 DARK MODE — Override all tokens */
html.dark {
  --color-surface: #0f172a;
  --color-primary: #60a5fa;
  /* ... ghi đè cùng tên biến */
}
```

- Khi `<html>` **không có** class `dark` → trình duyệt đọc giá trị từ `@theme` → **light**
- Khi `<html>` **có** class `dark` → CSS cascade ghi đè → **dark**
- Toggle chỉ cần 1 dòng JS: `document.documentElement.classList.toggle("dark")`

### Early Init (chống flash trắng khi reload)

```js
// index.html — chạy TRƯỚC khi Vue mount
if (localStorage.getItem("hrm-dark-mode") === "true") {
  document.documentElement.classList.add("dark");
}
```

---

## Kiến trúc Theme (3 lớp)

| Lớp                       | File                     | Vai trò                               | Khi đổi theme          |
| ------------------------- | ------------------------ | ------------------------------------- | ---------------------- |
| 1. `@theme { }`           | `tailwind.css` L9–L80    | Khai báo 53 tokens (light)            | ✅ Swap hex values     |
| 2. `html.dark { }`        | `tailwind.css` L86–L181  | Override tokens (dark) + EP vars      | ✅ Swap hex values     |
| 3. EP component overrides | `tailwind.css` L187–L330 | `!important` cho el-card, el-table... | ⚡ Tự động theo tokens |

> Tất cả 3 lớp nằm trong **1 file duy nhất** (`tailwind.css`). Components không chứa hardcoded colors.

---

## Hướng dẫn đổi theme

### Scenario A: Đổi color palette (blue → purple)

**Chỉ sửa `tailwind.css`** — swap hex trong `@theme` (light) và `html.dark` (dark):

```diff
  @theme {
-   --color-primary: #3b82f6;
+   --color-primary: #8b5cf6;
  }
  html.dark {
-   --color-primary: #60a5fa;
+   --color-primary: #a78bfa;
  }
```

Effort: **~30 phút**, 1 file duy nhất.

### Scenario B: Thêm multi-theme (user-selectable)

1. Tạo thêm bộ `html.theme-xxx { }` overrides trong `tailwind.css`
2. Update `index.html` early-init script để load đúng theme class
3. Thêm UI selector trong Header

---

## Đánh giá

| Tiêu chí           | Trạng thái                        |
| ------------------ | --------------------------------- |
| Token system       | ✅ 53 tokens, naming rõ ràng      |
| Component adoption | ✅ 78/78 (100%)                   |
| Dark mode          | ✅ EP overrides + FOUC prevention |
| Hardcoded leakage  | ✅ 0 remaining                    |
| Multi-theme ready  | ⚠️ Chưa — chỉ light/dark          |

> [!TIP]
> Mọi thay đổi theme chỉ cần sửa `tailwind.css` duy nhất.
