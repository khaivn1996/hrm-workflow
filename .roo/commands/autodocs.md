---
description: Auto-update docs/implements.md sau khi hoàn thành feature/module
---

# /autodocs - Cập nhật tài liệu project HRM

Bạn đang chạy workflow `/autodocs` cho repo HRM.

Mục tiêu: cập nhật file:

```text
docs/implements.md
```

Đây là file tài liệu implementation chính của project. Không chỉnh source code. Không chỉnh file docs khác trừ khi user yêu cầu rõ.

## Quy tắc bắt buộc

- File output duy nhất được phép sửa là `docs/implements.md`.
- Không bịa endpoint, query key, component, permission code, schema, model, function hoặc behavior.
- Mọi nội dung phải dựa trên code hiện tại, git diff, hoặc tài liệu hiện có.
- Nếu chưa đủ thông tin, phải ghi rõ `Cần kiểm tra thêm`.
- Giữ style Markdown hiện tại của `docs/implements.md`.
- Ưu tiên tiếng Việt tự nhiên, ngắn gọn, rõ cho developer.
- Khi sửa incremental, không rewrite toàn bộ file nếu không cần.
- Khi full rescan, có thể rebuild toàn bộ `docs/implements.md`, nhưng phải cố gắng giữ/merge changelog cũ.

## Chọn chế độ

### INCREMENTAL — mặc định

Dùng chế độ này nếu user không nói rõ `full rescan`.

Mục tiêu:
- Chỉ cập nhật phần liên quan đến thay đổi gần đây.
- Chỉ đọc các file liên quan.
- Cập nhật đúng section trong `docs/implements.md`.
- Thêm changelog entry mới.
- Không overwrite toàn bộ file nếu không cần.

Cách xác định thay đổi:

```text
1. Nếu user chỉ định module/file → ưu tiên theo user.
2. Nếu không, chạy: git diff --name-only HEAD
3. Nếu danh sách quá ít hoặc chưa đủ ngữ cảnh, chạy thêm: git diff --name-only HEAD~5
4. Phân loại file thay đổi theo module:
   - CHAMCONG
   - NHANSU
   - DANHGIA
   - TINHLUONG
   - BAOCAO
   - Backend
   - Frontend shared
   - Auth/Security
   - Docs/Config
5. Đọc `docs/implements.md` hiện tại.
6. Chỉ cập nhật sections liên quan.
7. Thêm một dòng mới vào Changelog.
```

### FULL RESCAN — chỉ khi user yêu cầu rõ

Chỉ dùng khi user nói rõ một trong các ý sau:

```text
full rescan
quét lại toàn bộ
rebuild implements.md
overwrite implements.md
tạo lại docs/implements.md từ đầu
```

Mục tiêu:
- Quét toàn bộ project.
- Rebuild `docs/implements.md`.
- Overwrite nội dung cũ nếu cần.
- Giữ hoặc merge changelog cũ nếu còn phù hợp.

## Step 0 — Xác định phạm vi

Trước khi đọc file sâu, xác định:

```text
- Đây là incremental hay full rescan?
- Module nào bị ảnh hưởng?
- Có cần trace flow frontend → backend → query registry không?
- Có cần cập nhật Tech Stack, Architecture, API, Components, Tests, Changelog không?
```

Nếu user chỉ định module, ví dụ:

```text
/autodocs cập nhật ChamCongTrong
/autodocs cập nhật auth heartbeat
/autodocs cập nhật module TINHLUONG
```

thì chỉ tập trung vào module đó.

## Step 1A — Incremental Scan

Khi chạy incremental, chỉ đọc file liên quan trực tiếp.

### Backend paths cần xét

```text
hrm_BE/app/routers/*.py
hrm_BE/app/services/*.py
hrm_BE/app/queries/__init__.py
hrm_BE/app/queries/*_queries.py
hrm_BE/app/batch/*.py
hrm_BE/app/security.py
hrm_BE/app/session_cleanup.py
hrm_BE/app/models.py
hrm_BE/app/schemas.py
hrm_BE/app/middleware/*.py
hrm_BE/tests/*.py
hrm_BE/requirements.txt
```

### Frontend paths cần xét

```text
hrm_FE/src/apis/*.js
hrm_FE/src/components/**/*.vue
hrm_FE/src/stores/*.js
hrm_FE/src/router/index.ts
hrm_FE/src/utils/*.js
hrm_FE/src/workers/*.js
hrm_FE/src/views/*.vue
hrm_FE/src/Layout/*.vue
hrm_FE/src/main.js
```

### Trace flow khi có feature/module thay đổi

Nếu thay đổi liên quan feature, phải cố gắng trace đủ flow:

```text
Vue component
→ frontend API module
→ FastAPI router
→ service/database operation hoặc batch handler
→ query registry key
→ SQL query file
→ SQL Server interaction
→ permission/router/store nếu liên quan
```

Ví dụ cần map:

```text
Component ↔ API function ↔ Endpoint ↔ Query key ↔ Query file ↔ Permission code
```

## Step 1B — Full Rescan

Khi full rescan, quét toàn bộ project HRM.

### Backend full scan

Đọc và tổng hợp:

```text
hrm_BE/app/models.py
hrm_BE/app/schemas.py
hrm_BE/app/security.py
hrm_BE/app/database.py
hrm_BE/app/session_cleanup.py
hrm_BE/app/routers/*.py
hrm_BE/app/queries/__init__.py
hrm_BE/app/queries/*.py
hrm_BE/app/batch/*.py
hrm_BE/app/middleware/*.py
hrm_BE/app/services/*.py
hrm_BE/tests/*.py
hrm_BE/requirements.txt
hrm_BE/Dockerfile
hrm_BE/docker-compose.yml
```

Cần ghi nhận:

```text
- Database models/classes/columns
- Pydantic schemas
- Auth/session/security functions
- Routers/endpoints/method/path/auth requirements
- Query Registry keys/files
- Batch handlers
- Services/functions
- Middleware
- Background tasks
- Tests/test count
- Dependencies
```

### Frontend full scan

Đọc và tổng hợp:

```text
hrm_FE/src/apis/*.js
hrm_FE/src/components/**/*.vue
hrm_FE/src/stores/*.js
hrm_FE/src/utils/*.js
hrm_FE/src/views/*.vue
hrm_FE/src/Layout/*.vue
hrm_FE/src/router/index.ts
hrm_FE/src/workers/*.js
hrm_FE/src/main.js
hrm_FE/package.json
hrm_FE/vite.config.*
```

Cần ghi nhận:

```text
- API modules/exported functions
- Vue components theo module
- Pinia stores/state/actions
- Router paths/permissions
- Shared views/components
- Utilities
- Worker actions
- Layout/Login/Header/Sidemenu
- i18n/theme nếu liên quan
- Dependencies/build config
```

## Step 2 — Architecture & Patterns

Luôn kiểm tra và cập nhật nếu thay đổi:

```text
- Query Registry Pattern
- Service Layer
- Dependency Injection
- Centralized API Layer
- Permission-based Routing
- Batch Processing
- Batch Insert Registry
- Web Worker Excel flow
- Heartbeat Session
- Axios Interceptor
- Audit Trail
- Memory Pagination
- Error handling conventions
- API response check conventions
- Known gotchas/design decisions
```

Khi mô tả pattern, ghi:

```text
- Pattern name
- Files chính
- Cách hoạt động
- Lý do/ảnh hưởng thực tế
```

## Step 3 — Cập nhật docs/implements.md

`docs/implements.md` cần giữ các sections chính sau. Nếu incremental thì chỉ cập nhật section liên quan. Nếu full rescan thì đảm bảo đủ tất cả.

```text
# HRM Project - Implementation Details

## Tech Stack

## Architecture Patterns & Techniques
### Design Patterns
### Data Flow Diagrams
### Frontend Architecture
### Backend Architecture
### Security Architecture
### Error Handling Patterns
### API Response Check Conventions
### Known Design Decisions & Gotchas

## Project Tree

## Backend Details
### Database Models
### Pydantic Schemas
### API Endpoints
### SQL Query Files
### Batch Processing
### Middleware & Security
### Background Tasks
### Testing

## Frontend Details
### API Modules
### Vue Components
### Pinia Stores
### Shared Views
### Utilities
### Layout
### Router & Permissions
### Workers

## Component Interaction Flows
### Module Flows
### Shared Component Usage Map

## Dependencies

## Thống kê

## Changelog
```

## Step 4 — Data Flow Diagrams

Nếu full rescan hoặc flow chính thay đổi, cập nhật Mermaid diagrams cho các flow liên quan:

```text
1. Authentication Flow
2. CRUD Data Flow
3. Query Registry Flow
4. Batch Processing Flow
5. Report Flow
6. Worker Flow
```

Không cần vẽ lại tất cả nếu incremental không đụng tới flow đó.

## Step 5 — Module Flow format

Khi cập nhật module flow, dùng format ngắn gọn:

```text
User action
→ Vue Component
→ API module/function
→ Backend endpoint/router
→ Service/batch/query registry
→ SQL Server
→ Response handling/UI update
```

Nếu biết permission, ghi thêm:

```text
Permission: `1frm...`
```

Nếu chưa chắc:

```text
Permission: Cần kiểm tra thêm trong `router/index.ts` hoặc `permissionStore.js`
```

## Step 6 — Changelog

Luôn thêm dòng mới vào `## Changelog`.

Format:

```markdown
| Ngày | Module | Thay đổi | Conversation |
|---|---|---|---|
| YYYY-MM-DD | MODULE | Mô tả ngắn thay đổi docs | /autodocs |
```

Quy tắc:

```text
- Không xóa changelog cũ khi incremental.
- Dòng mới đặt ở đầu bảng changelog nếu bảng đang sắp xếp mới → cũ.
- Module nên dùng: CHAMCONG, NHANSU, DANHGIA, TINHLUONG, BAOCAO, Backend, Frontend, Auth, Security, Docs, Testing, Worker, Utils.
- Conversation dùng `/autodocs` nếu không có mã commit/conversation cụ thể.
```

## Step 7 — Kiểm tra trước khi kết thúc

Trước khi trả lời user, kiểm tra:

```text
- `docs/implements.md` vẫn render Markdown hợp lệ.
- Không có section bị duplicate vô lý.
- Không xóa nhầm changelog cũ.
- Không ghi thông tin không kiểm chứng.
- Nếu có bảng Markdown, cột phải cân đối và dễ đọc.
- Nếu có Mermaid, syntax cơ bản hợp lệ.
```

## Step 8 — Trả lời user

Sau khi cập nhật xong, trả lời ngắn gọn bằng tiếng Việt:

```text
Đã cập nhật docs/implements.md.

Tóm tắt:
- ...

Sections đã cập nhật:
- ...

Cần kiểm tra thêm:
- ...
```

Nếu không có gì cần kiểm tra thêm, ghi:

```text
Cần kiểm tra thêm: Không có.
```