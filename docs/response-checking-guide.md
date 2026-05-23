# API Response Checking Guide

Hướng dẫn kiểm tra response cho từng loại helper API trong hệ thống HRM.

> [!IMPORTANT]
> Tất cả response đều qua axios, nên luôn truy cập qua `res.data.*` (axios wraps response trong `.data`).

---

## 1. `registrySelect` — Load dữ liệu (SELECT)

**Endpoint:** `POST /api/tsql/query/registry/{queryKey}`

**BE Response Schema** (`QueryResponse`):

```json
{
  "status": "success",
  "query": "SELECT ...",
  "parameters": { "NV_MA": "001031" },
  "data": [ { "MA": "001031", "TEN": "Nguyễn Văn A" }, ... ],
  "rows_affected": 2,
  "message": "Query executed successfully"
}
```

### ✅ Check có dữ liệu:

```javascript
const res = await registrySelect("LoadDS_1_XXX", { NV_MA: "001031" });
const data = Array.isArray(res?.data?.data) ? res.data.data : [];

if (data.length > 0) {
  // Có dữ liệu → xử lý
  const firstRow = data[0];
} else {
  // Không có dữ liệu
}
```

---

## 2. `registryCrud` — Thao tác INSERT / UPDATE / DELETE

**Endpoint:** `POST /api/tsql/query/registry/{queryKey}/crud`

**BE Response Schema** (`QueryResponse`):

```json
{
  "status": "success",
  "query": "UPDATE ...",
  "parameters": { "NV_MA": "001031" },
  "data": null,
  "rows_affected": 1,
  "message": "Query executed successfully"
}
```

### ✅ Check thành công:

```javascript
const res = await registryCrud("UpdateInfo_1_XXX", { NV_MA: "001031", ... });
const rowsAffected = res?.data?.rows_affected ?? 0;

if (rowsAffected > 0) {
  Success("Cập nhật thành công");
} else {
  Error("Cập nhật thất bại");
}
```

## 3. `registrySelectPagination` — Load dữ liệu có phân trang

**Endpoint:** `POST /api/tsql/query/registry/{queryKey}/pagination?offset=0&limit=100`

**BE Response Schema** (`QueryResponseINF`):

```json
{
  "status": "success",
  "data": [ { "MA": "001031", ... }, ... ],
  "offset": 0,
  "limit": 100,
  "has_more": true,
  "total": 500,
  "message": "Loaded 100 records, total: 500"
}
```

### ✅ Check có dữ liệu:

```javascript
const res = await registrySelectPagination(
  "LoadDS_4_XXX",
  { NGAY, DONVI },
  { offset, limit },
);
const data = Array.isArray(res?.data?.data) ? res.data.data : [];
const total = res?.data?.total || data.length;
```

---

## 4. `registryBatchInsert` — Insert nhiều row cùng lúc

**Endpoint:** `POST /api/tsql/batch-insert/registry/{queryKey}`

**BE Response Schema** (`BatchInsertRegistryResponse`):

```json
{
  "success": true,
  "message": "Đã insert thành công 10 dòng vào Table_Name",
  "rowsInserted": 10,
  "errors": []
}
```

> [!IMPORTANT]
> Helper `registryBatchInsert` đã wrap response → trả về trực tiếp object, **KHÔNG** qua `res.data`.

### ✅ Check thành công:

```javascript
const result = await registryBatchInsert("InsertBulk_1_XXX", dataArray);

if (result.success && result.rowsInserted > 0) {
  Success(`Đã insert ${result.rowsInserted} dòng`);
} else {
  Error(result.message || "Insert thất bại");
}
```

---

## 5. `procedure/select` — Gọi Stored Procedure (SELECT)

**Endpoint:** `POST /api/tsql/procedure/select`

**BE Response Schema** (`ProcedureResponse`):

```json
{
  "status": "success",
  "procedure": "SP_LOADDS_XXX",
  "parameters": { "nv_ma": "001031" },
  "data": [ { ... }, ... ],
  "rows_affected": 5,
  "message": "Procedure executed successfully"
}
```

### ✅ Check có dữ liệu:

```javascript
const res = await axios.post(`/api/tsql/procedure/select`, {
  procedure_name: "SP_LOADDS_XXX",
  params: { nv_ma: "001031" },
});
const data = Array.isArray(res?.data?.data) ? res.data.data : [];

if (data.length > 0) {
  // Có dữ liệu
}
```

---

## 6. `procedure/crud` — Gọi Stored Procedure (INSERT/UPDATE/DELETE)

**Endpoint:** `POST /api/tsql/procedure/crud`

**BE Response Schema** (`ProcedureResponse`):

```json
{
  "status": "success",
  "procedure": "SP_INSERT_XXX",
  "parameters": { "nv_ma": "001031" },
  "data": null,
  "rows_affected": 1,
  "message": "Procedure executed successfully"
}
```

### ✅ Check thành công:

```javascript
const res = await axios.post(`/api/tsql/procedure/crud`, {
  procedure_name: "SP_INSERT_XXX",
  params: { nv_ma: "001031" },
});
const rowsAffected = res?.data?.rows_affected ?? 0;

if (rowsAffected > 0) {
  Success("Thành công");
} else {
  Error("Thất bại");
}
```

---

## 7. `procedure/select-pagination` — Gọi SP có phân trang

**Endpoint:** `POST /api/tsql/procedure/select-pagination?offset=0&limit=100`

**BE Response Schema** (`ProcedureResponseINF`):

```json
{
  "status": "success",
  "data": [ { ... }, ... ],
  "offset": 0,
  "limit": 100,
  "has_more": true,
  "total": 500,
  "message": "Loaded 100 records, total: 500"
}
```

### ✅ Check có dữ liệu:

```javascript
const res = await axios.post(
  `/api/tsql/procedure/select-pagination`,
  { procedure_name: "SP_LOADDS_XXX", params: { ... } },
  { params: { offset: 0, limit: 100 } }
);
const data = Array.isArray(res?.data?.data) ? res.data.data : [];
const total = res?.data?.total || data.length;
```

---

## Bảng Tóm Tắt Nhanh

| Helper                        | Check dữ liệu tồn tại         | Check thao tác thành công                   |
| ----------------------------- | ----------------------------- | ------------------------------------------- |
| `registrySelect`              | `res?.data?.data?.length > 0` | —                                           |
| `registryCrud`                | —                             | `res?.data?.rows_affected > 0`              |
| `registrySelectPagination`    | `res?.data?.data?.length > 0` | —                                           |
| `registryBatchInsert`         | —                             | `result.success && result.rowsInserted > 0` |
| `procedure/select`            | `res?.data?.data?.length > 0` | —                                           |
| `procedure/crud`              | —                             | `res?.data?.rows_affected > 0`              |
| `procedure/select-pagination` | `res?.data?.data?.length > 0` | —                                           |

> [!CAUTION]
> **Sai lầm phổ biến:**
>
> - Dùng `res.data.length` thay vì `res.data.data.length` (thiếu 1 tầng `.data`)
> - Quên check null: luôn dùng optional chaining `?.` và nullish coalescing `??`
> - `registryBatchInsert` trả về trực tiếp object (đã wrap), các helper khác trả về axios response
