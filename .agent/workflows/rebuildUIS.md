---
description: Centralize UI theme từ hardcoded colors → 1 file tailwind.css (semantic tokens + dark mode). 6 bước - phân tích, thiết kế tokens, batch replace, swap hex, dark mode, verify.
---

# Rebuild UI Style — Centralize Theme từ Hardcoded → Single Source of Truth

> Workflow tổng hợp quy trình chuyển đổi giao diện từ hardcoded colors/fonts rải rác trong 60+ component → 1 file `tailwind.css` duy nhất với semantic CSS variables, hỗ trợ dark mode.
>
> **Áp dụng khi**: Muốn đổi theme toàn bộ app, thêm dark mode, hoặc refactor UI system cho project Vue 3 + Tailwind v4 + Element Plus.

---

## Tổng quan

```
TRƯỚC: 60+ files × hardcoded hex/rgba/Tailwind named colors → Đổi theme = sửa 60 files
SAU:   1 file tailwind.css × semantic tokens → Đổi theme = sửa 1 file
```

---

## Bước 1: Phân tích hiện trạng

### 1.1 Quét toàn bộ hardcoded colors

```bash
# Tìm Tailwind named-color classes (bg-white, text-blue-500, border-gray-200...)
grep -rn "bg-white\|bg-blue\|bg-indigo\|text-blue\|text-gray\|text-black\|border-gray\|border-blue" src/ --include="*.vue"

# Tìm hex hardcoded trong <style> blocks
grep -rn "#[0-9a-fA-F]\{3,8\}" src/ --include="*.vue"

# Tìm rgba hardcoded
grep -rn "rgba(" src/ --include="*.vue"
```

### 1.2 Tạo bảng mapping

Ghi ra file phân tích (ví dụ `theme_analysis.md`), bao gồm:

| Thông tin cần ghi                | Ví dụ                                              |
| -------------------------------- | -------------------------------------------------- |
| File nào dùng màu nào            | `Header.vue` → `text-blue-500`, `text-black`       |
| Giá trị hex xuất hiện nhiều nhất | `#3b82f6` (15 lần), `bg-white` (30 lần)            |
| Font đang load                   | Quicksand, Roboto, Poppins — cái nào thực sự dùng? |
| Element Plus theme mặc định      | `#409EFF` — có cần override không?                 |

### 1.3 Phân loại files theo layer

```
Layer 1: Global      → index.html, main.js, tailwind.css, vite.config
Layer 2: Layout      → Layout/index.vue, Header.vue, Sidemenu.vue, Login.vue
Layer 3: Shared      → Grid.vue, MultiSelectGrid.vue
Layer 4: Components  → CHAMCONG/*.vue, NHANSU/*.vue, DANHGIA/*.vue, ...
```

---

## Bước 2: Thiết kế semantic tokens

### 2.1 Quy tắc đặt tên

```
--color-{category}-{variant}

Categories:
  surface     → nền (bg)
  primary     → màu chủ đạo
  foreground  → text chính
  text-*      → text phụ (muted, light, placeholder)
  border      → viền
  active-*    → trạng thái active/hover trong sidebar
  shadow      → bóng đổ
  toggle-*    → dark mode toggle button
  login-*     → riêng cho trang login nếu có branding khác
```

### 2.2 Khai báo trong `tailwind.css` dùng `@theme`

```css
@import "tailwindcss";

@theme {
  /* === Surface / Background === */
  --color-surface: #ffffff;
  --color-surface-alt: #f8fafc;
  --color-surface-hover: #eef2ff;

  /* === Primary === */
  --color-primary: #3b82f6;
  --color-primary-dark: #2563eb;
  --color-primary-deeper: #1d4ed8;
  --color-primary-lighter: #93bbfd;

  /* === Text === */
  --color-foreground: #0f172a;
  --color-text-muted: #475569;
  --color-text-light: #4b5563;
  --color-text-placeholder: #94a3b8;

  /* === Border === */
  --color-border: #e5e7eb;
  --color-border-light: #e4e7ec;

  /* === Sidebar === */
  --color-active-bg-from: #dbeafe;
  --color-active-bg-to: #bfdbfe;
  --color-hover-bg-from: #f0f9ff;
  --color-hover-bg-to: #e0f2fe;

  /* === Misc === */
  --color-shadow: rgba(0, 0, 0, 0.1);
  --color-focus-ring: rgba(59, 130, 246, 0.3);
}
```

> **Kết quả**: Tailwind tự sinh utility classes: `bg-surface`, `text-primary`, `border-border`, `text-foreground`...

---

## Bước 3: Batch replace — Tailwind classes trong template

### 3.1 Bảng replace chính

| Cũ (hardcoded)                        | Mới (semantic)                                | Lý giải         |
| ------------------------------------- | --------------------------------------------- | --------------- |
| `bg-white`                            | `bg-surface`                                  | Nền chính       |
| `bg-slate-50` / `bg-gray-50`          | `bg-surface-alt`                              | Nền phụ         |
| `bg-indigo-50` / `hover:bg-indigo-50` | `bg-surface-hover` / `hover:bg-surface-hover` | Hover state     |
| `text-blue-500`                       | `text-primary`                                | Accent chính    |
| `text-blue-600`                       | `text-primary-dark`                           | Accent đậm      |
| `text-blue-700`                       | `text-primary-deeper`                         | Accent đậm nhất |
| `text-black` / `text-slate-900`       | `text-foreground`                             | Text chính      |
| `text-gray-600` / `text-gray-700`     | `text-text-light`                             | Text phụ        |
| `border-gray-200`                     | `border-border`                               | Viền chuẩn      |
| `border-[#e4e7ec]`                    | `border-border-light`                         | Viền nhẹ        |
| `border-blue-300`                     | `border-primary-lighter`                      | Viền accent     |

### 3.2 Thực hiện

```bash
# Ví dụ replace bằng IDE hoặc script
# Search: bg-white  →  Replace: bg-surface
# Search: text-blue-500  →  Replace: text-primary
# ...

# SAU KHI REPLACE, verify không còn named colors:
grep -rn "bg-white\|bg-blue\|bg-indigo\|text-blue\|text-gray\|text-black\|border-gray\|border-blue" src/ --include="*.vue"
# Kỳ vọng: 0 kết quả
```

> ⚠️ **Cẩn thận**: Một số `bg-white` nằm trong comment hoặc string → check từng kết quả trước khi replace all.

---

## Bước 4: Swap hex/rgba trong `<style scoped>`

Một số component (đặc biệt Login.vue, Sidemenu.vue) có hex/rgba hardcoded trong CSS scoped. Thay bằng `var(--color-*)`:

```css
/* CŨ */
background: linear-gradient(135deg, #f8fafc 0%, #eef2ff 100%);
box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
border: 1px solid #e4e7ec;
color: #475569;

/* MỚI */
background: linear-gradient(
  135deg,
  var(--color-surface-alt) 0%,
  var(--color-surface-hover) 100%
);
box-shadow: 0 4px 6px var(--color-shadow);
border: 1px solid var(--color-border-light);
color: var(--color-text-muted);
```

### Kỹ thuật `color-mix()` cho shadow/opacity

Khi cần rgba với opacity động:

```css
/* Thay vì rgba(0,0,0,0.1) */
box-shadow: 0 2px 4px
  color-mix(in srgb, var(--color-foreground) 10%, transparent);
```

### Element Plus component overrides

Trong `tailwind.css`, override EP vars để consistent:

```css
/* Element Plus overrides */
:root {
  --el-color-primary: var(--color-primary);
  --el-border-color: var(--color-border);
  --el-text-color-regular: var(--color-foreground);
}
```

---

## Bước 5: Dark Mode

### 5.1 Override tokens trong `html.dark`

```css
html.dark {
  --color-surface: #1e293b;
  --color-surface-alt: #0f172a;
  --color-surface-hover: #334155;
  --color-primary: #60a5fa;
  --color-foreground: #f1f5f9;
  --color-text-muted: #94a3b8;
  --color-border: #334155;
  --color-shadow: rgba(0, 0, 0, 0.3);
  /* ... override TẤT CẢ tokens */
}
```

### 5.2 Override EP components cho dark mode

```css
html.dark {
  /* Dialog */
  .el-dialog {
    background: var(--color-surface) !important;
  }
  .el-dialog__title {
    color: var(--color-foreground) !important;
  }

  /* Table */
  .el-table,
  .el-table th,
  .el-table td {
    background-color: var(--color-surface) !important;
    color: var(--color-foreground) !important;
  }

  /* Input */
  .el-input__inner {
    background: var(--color-surface-alt);
    color: var(--color-foreground);
  }

  /* ... các EP component khác */
}
```

### 5.3 Toggle button (Header.vue)

```javascript
// Persist preference
function toggleDarkMode() {
  isDark.value = !isDark.value;
  document.documentElement.classList.toggle("dark", isDark.value);
  localStorage.setItem("hrm-dark-mode", isDark.value ? "true" : "false");
}

// Restore on mount
onMounted(() => {
  if (localStorage.getItem("hrm-dark-mode") === "true") {
    isDark.value = true;
    document.documentElement.classList.add("dark");
  }
});
```

### 5.4 FOUC prevention (index.html)

Thêm script inline vào `<head>` để tránh flash trắng khi reload:

```html
<script>
  if (localStorage.getItem("hrm-dark-mode") === "true") {
    document.documentElement.classList.add("dark");
  }
</script>
```

---

## Bước 6: Verification — Kiểm tra 100%

### 6.1 Grep đảm bảo 0 hardcoded

```bash
# Kiểm tra Tailwind named colors
grep -rn "bg-white\|bg-blue\|bg-indigo\|text-blue\|text-gray\|text-black\|border-gray" src/ --include="*.vue" | grep -v "node_modules" | grep -v "//.*bg-white"
# Kỳ vọng: 0 kết quả (trừ comments)

# Kiểm tra hex hardcoded (trừ trong <script>)
grep -rn "#[0-9a-fA-F]\{6\}" src/ --include="*.vue" | grep -v "<script" | grep -v "//"
# Kỳ vọng: 0 kết quả
```

### 6.2 Visual check

1. **Login page** — gradient, input focus, button hover, dropdown
2. **Layout** — Header, Sidebar hover/active, content area
3. **Grid** — sticky header shadow, row hover
4. **Dark mode** — toggle ON → verify tất cả trang đều dark, toggle OFF → về light
5. **Element Plus** — dialog, table, input, select, dropdown đều theo theme

### 6.3 Đổi thử theme

Thay giá trị `--color-primary` trong `@theme` → toàn bộ app phải đổi màu chủ đạo mà **không sửa file nào khác**.

---

## Checklist tóm tắt

```
[ ] Bước 1: Quét + phân tích hardcoded colors → tạo theme_analysis.md
[ ] Bước 2: Thiết kế semantic tokens trong tailwind.css @theme block
[ ] Bước 3: Batch replace Tailwind classes (bg-white → bg-surface, ...)
[ ] Bước 4: Swap hex/rgba trong <style scoped> → var(--color-*)
[ ] Bước 5: Thêm Dark Mode (html.dark overrides + toggle + FOUC prevention)
[ ] Bước 6: Verify 0 hardcoded + visual check + thử đổi theme
```

---

## Lưu ý quan trọng

> **Tất cả thay đổi là CSS-only** — không ảnh hưởng logic JavaScript hay data flow. Nếu làm đúng, giao diện phải trông **giống hệt trước** (cùng màu, cùng spacing), chỉ khác ở cách khai báo.

> **Nguyên tắc naming**: Dùng tên semantic (`surface`, `primary`, `foreground`) thay vì tên màu (`white`, `blue`, `gray`). Như vậy khi đổi theme Navy → Red, class `bg-primary` vẫn đúng ngữ nghĩa.

> **Thứ tự thực hiện**: Layout → Shared views → Components. Làm từ global → cụ thể để dễ verify từng bước.
