---
description: Auto-update docs/implements.md sau khi hoàn thành feature/module
---

# /autodocs - Tổng hợp & cập nhật tài liệu project

Workflow này cập nhật file `docs/implements.md`.
Đây là workflow DUY NHẤT cho documentation. Output DUY NHẤT là `docs/implements.md`.

## Chọn Mode

> **INCREMENTAL (mặc định)**: Chỉ cập nhật phần thay đổi gần đây.
> Dùng `git diff --name-only HEAD~5` hoặc yêu cầu user cho biết module nào thay đổi.
> Chỉ quét files đã thay đổi, cập nhật sections tương ứng trong implements.md.
>
> **FULL RESCAN**: Chỉ khi user yêu cầu rõ ràng (ví dụ: "full rescan").
> Quét toàn bộ project, overwrite implements.md.

// turbo-all

## Step 0 (Incremental Mode): Xác định thay đổi

```
1. Chạy git diff --name-only HEAD~5 (hoặc hỏi user module nào thay đổi)
2. Phân loại files thay đổi theo module: CHAMCONG, NHANSU, DANHGIA, TINHLUONG, BAOCAO, Backend
3. CHỈ quét các files trong danh sách thay đổi
4. Đọc implements.md hiện tại, chỉ cập nhật sections liên quan
5. Thêm changelog entry
```

## Step 1 (Full Rescan Only): Quét cấu trúc project

```
Quét toàn bộ project HRM bằng list_dir, view_file_outline, view_file:

1. Backend `hrm_BE/`:
   - `app/models.py` → liệt kê tất cả model classes + columns
   - `app/schemas.py` → liệt kê tất cả Pydantic schemas
   - `app/security.py` → liệt kê tất cả functions (tên + mô tả ngắn)
   - `app/database.py` → thông tin kết nối DB
   - `app/session_cleanup.py` → mô tả background tasks
   - `app/routers/*.py` → liệt kê tất cả endpoints (method, path, mô tả)
   - `app/queries/__init__.py` → đọc registry mapping (ALL_QUERIES)
   - `app/queries/*.py` → liệt kê query files + số lượng queries
   - `app/batch/*.py` → liệt kê batch processing files
   - `app/middleware/*.py` → liệt kê middleware
   - `app/services/*.py` → liệt kê services + schemas/functions
   - `tests/*.py` → liệt kê test files + số test cases
   - `requirements.txt` → dependencies

2. Frontend `hrm_FE/src/`:
   - `apis/*.js` → liệt kê API modules + exported functions
   - `components/**/*.vue` → liệt kê components theo module
   - `stores/*.js` → Pinia stores + state/actions
   - `utils/*.js` → utility files + functions
   - `views/*.vue` → shared view components
   - `Layout/*.vue` → layout components
   - `router/index.ts` → routes + permission codes
   - `workers/*.js` → web worker actions
   - `main.js` → plugins, middleware, startup sequence

3. Architecture & Patterns (QUAN TRỌNG):
   - Xác định design patterns: Registry Pattern, Service Layer, Dependency Injection, etc.
   - Trace data flow: Component → API module → Backend router → Service → Query → SQL Server
   - Xác định authentication flow: Login → JWT → Heartbeat → Session cleanup
   - Xác định permission flow: Login → Load permissions → Route guard → Component render
   - Liệt kê error handling patterns (Frontend + Backend)
   - Phát hiện kỹ thuật đặc biệt: Web Worker, Batch Processing, Infinite Scroll, etc.
   - Liệt kê coding conventions: naming, folder structure, API response check patterns

4. Component Interaction Flows:
   - Trace flow của mỗi module chính (CHAMCONG, NHANSU, DANHGIA...)
   - Xác định shared dependencies giữa components
   - Map: Component ↔ API ↔ Backend endpoint ↔ Query key relationships
   - Ghi nhận design decisions, known gotchas, edge cases
```

## Step 2: Tổng hợp vào docs/implements.md

```
Ghi kết quả vào `docs/implements.md`. OVERWRITE toàn bộ nội dung cũ.
ĐÂY LÀ FILE DUY NHẤT CHỨA TOÀN BỘ TÀI LIỆU PROJECT.

Format cần có TẤT CẢ sections sau:

# HRM Project - Implementation Details
> Auto-generated bởi /autodocs workflow
> Cập nhật lần cuối: [YYYY-MM-DD]

## Tech Stack
(Bảng technology stack)

## Architecture Patterns & Techniques
### Design Patterns
Liệt kê MỌI pattern đang dùng, gồm:
- Tên pattern
- Nơi áp dụng (files cụ thể)
- Cách hoạt động (mô tả ngắn)
- Lý do chọn

### Data Flow Diagrams
Vẽ mermaid diagrams cho CÁC FLOW CHÍNH:
1. Authentication Flow (Login → Token → Heartbeat → Session)
2. CRUD Data Flow (Component → API → Router → Service → SQL Server)
3. Query Registry Flow (Frontend key → Backend registry → SQL execution)
4. Batch Processing Flow
5. Report Flow (FastAPI → ASP.NET Crystal Reports)
6. Worker Flow (Main thread → Web Worker → API)

### Frontend Architecture
### Backend Architecture
### Security Architecture
### Error Handling Patterns
### API Response Check Conventions
### Known Design Decisions & Gotchas

## Project Tree
(Cây thư mục chi tiết)

## Backend Details
### Database Models (bảng)
### Pydantic Schemas (bảng)
### API Endpoints (nhóm theo router, bảng)
### SQL Query Files (bảng)
### Batch Processing (bảng)
### Middleware & Security (bảng)
### Background Tasks (bảng)
### Testing (bảng)

## Frontend Details
### API Modules (bảng)
### Vue Components (bảng theo module: CHAMCONG, NHANSU, DANHGIA...)
### Pinia Stores (bảng)
### Shared Views (bảng)
### Utilities (bảng)
### Layout (bảng)
### Router & Permissions (bảng)
### Workers (bảng)

## Component Interaction Flows
### Module Flows
Cho mỗi module chính, mô tả:
- User Action → Component → API → Backend → Query → DB
- Shared components sử dụng
- Permission requirements

### Shared Component Usage Map
(Bảng: Shared Component | Props/Events | Modules sử dụng)

## Dependencies (Backend + Frontend)

## Thống kê (Counts tổng hợp)

## Changelog
Thêm dòng mới cho thay đổi vừa thực hiện.
Chỉ THÊM dòng mới, KHÔNG xóa dòng cũ.
Format: | Ngày | Module | Thay đổi | Conversation |
```

## Step 3: Thông báo kết quả

```
Thông báo cho user:
- Tóm tắt cập nhật trong docs/implements.md
- Highlight: patterns mới, endpoints mới, modules mới
- Thống kê: components, endpoints, queries, tests
```
