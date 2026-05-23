---
description: Smart context loading - KI-first, targeted lookup, tối ưu token
---

# /ref - Smart Context & Execute

Workflow tối ưu token: dùng KIs + implements.md thay vì quét toàn bộ project.

> **NGUYÊN TẮC VÀNG**: KHÔNG quét toàn bộ project files.
> Chỉ đọc file cụ thể khi đã xác định chính xác cần đọc gì.

// turbo-all

## Step 1: Đọc KIs (Knowledge Items)

```
Đọc KI summaries từ ~/.gemini/antigravity/knowledge/*/metadata.json
Chỉ đọc artifacts của KI liên quan đến task hiện tại:

- Frontend task → hrm-vue3-element-plus-patterns
- Backend task → hrm-fastapi-registry-pattern
- API/response task → hrm-api-service-layer
- Architecture/overview → hrm-project-architecture

KHÔNG đọc tất cả KIs. Chỉ đọc KI phù hợp với yêu cầu user.
```

## Step 2: Xác định component tương tự (Targeted Lookup)

```
Dùng docs/implements.md (chỉ đọc section liên quan, KHÔNG đọc toàn bộ 900 dòng):

- Nếu task liên quan module CHAMCONG → đọc section "CHAMCONG" trong implements.md
- Nếu task liên quan module NHANSU → đọc section "NHANSU"
- Nếu task liên quan backend API → đọc section "API Endpoints"
- Nếu cần biết query keys → đọc section "SQL Query Files"

Sau đó tìm component CÓ SẴN tương tự bằng grep_search (tìm theo tên function, pattern):
- Ví dụ: cần làm form CRUD → grep "registryCrud" trong module tương tự
- Ví dụ: cần batch processing → grep "batch" trong components/TINHLUONG/
- CHỈ đọc 1-2 file tương tự nhất, KHÔNG đọc hết module

Mục tiêu: tìm code mẫu thực tế trong project để follow pattern.
```

## Step 3: Context7 - Latest Library Docs

```
Dùng MCP context7 CHỈ KHI task liên quan đến:
- API syntax không chắc chắn (Element Plus props, Vue 3 API mới)
- Library version mới có breaking changes
- Pattern chưa có trong KIs

Quy trình:
1. resolve-library-id cho library cần tra cứu
2. query-docs với câu hỏi cụ thể (KHÔNG query chung chung)

Ví dụ tốt: "Element Plus el-table custom column render"
Ví dụ xấu: "Element Plus documentation"

KHÔNG tra cứu nếu KIs đã có đủ thông tin.
```

## Step 4: Phân tích & Lên kế hoạch

```
Dựa vào context đã thu thập (KIs + component mẫu + Context7 docs):

1. Xác định chính xác files cần tạo/sửa
2. Liệt kê các thay đổi cần làm
3. Xác nhận với user nếu có quyết định lớn
4. KHÔNG code ngay - phân tích trước
```

## Step 5: Thực thi

```
Code theo đúng conventions từ KIs:
- Frontend: Composition API, ref/reactive, showLoading/hideLoading
- API calls: registrySelect/registryCrud + response check patterns
- Backend: Query Registry, parameterized queries
- Error handling: try/catch/finally pattern

Sau khi hoàn thành, nhắc user chạy /autodocs nếu thay đổi lớn.
```
