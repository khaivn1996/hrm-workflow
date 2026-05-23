---
name: sql-query
description: SQL Server query patterns for HRM backend. Use when creating or modifying database queries, stored procedures, or Query Registry entries.
---

# SQL Query Skill

> Use this skill whenever working on SQL queries in `hrm_BE/app/queries/`.

---

## Query Registry Pattern

Every new query MUST follow these 3 steps:

### Step 1: Tạo query file (hoặc thêm vào file có sẵn)

```python
# hrm_BE/app/queries/tenmodule_queries.py

"""
Query Registry cho module TenModule
Naming convention: TENMODULE_{ACTION}_{DESCRIPTION}
"""

TENMODULE_QUERIES = {
    # ============================================================================
    # LOAD DATA
    # ============================================================================
    # Mô tả ngắn gọn
    # Params: param1, param2
    "TENMODULE_LOAD_DATA": """
        SELECT col1, col2
        FROM TABLE_NAME WITH (NOLOCK)
        WHERE col1 = :param1 AND col2 = :param2
        ORDER BY col1
    """,

    # ============================================================================
    # CRUD OPERATIONS
    # ============================================================================
    "TENMODULE_INSERT": """
        INSERT INTO TABLE_NAME (col1, col2)
        VALUES (:param1, :param2)
    """,

    "TENMODULE_UPDATE": """
        UPDATE TABLE_NAME
        SET col2 = :newValue
        WHERE col1 = :param1
    """,

    "TENMODULE_DELETE": """
        DELETE FROM TABLE_NAME
        WHERE col1 = :param1
    """,
}
```

### Step 2: Register trong `__init__.py`

```python
# hrm_BE/app/queries/__init__.py

# Thêm import
from .tenmodule_queries import TENMODULE_QUERIES

# Thêm vào ALL_QUERIES dict
ALL_QUERIES = {
    # ... existing ...
    **TENMODULE_QUERIES,
}
```

### Step 3: Gọi từ Frontend

```javascript
// Trong apiModule.js
import axios from "axios";

// SELECT query
export const loadData = async (params) => {
  return await axios.post("/api/tsql/query/registry/TENMODULE_LOAD_DATA", {
    params: { param1: params.value1, param2: params.value2 },
  });
};

// CRUD query (INSERT/UPDATE/DELETE)
export const saveData = async (params) => {
  return await axios.post("/api/tsql/query/registry/crud", {
    query_key: "TENMODULE_INSERT",
    params: { param1: params.value1, param2: params.value2 },
  });
};
```

---

## SQL Rules

| Rule         | Đúng ✅                         | Sai ❌                        |
| ------------ | ------------------------------- | ----------------------------- |
| Parameters   | `:paramName`                    | `'${value}'` (injection risk) |
| Read queries | `WITH (NOLOCK)` khi phù hợp     | Không dùng NOLOCK cho CRUD    |
| Date format  | `yyyy-MM-dd` hoặc SQL functions | Format tay                    |
| Naming       | `MODULE_ACTION_DESC`            | random names                  |
| Comments     | `# Params: param1, param2`      | Không document params         |

---

## Naming Convention

```
{MODULE}_{ACTION}_{DESCRIPTION}

Ví dụ:
- NGHIPHEP_LOAD_LOAI_PHEP      (load danh sách loại phép)
- NGHIPHEP_CHECK_THOI_VIEC     (kiểm tra thôi việc)
- NGHIPHEP_UPDATE               (cập nhật nghỉ phép)
- NGHIPHEP_DELETE               (xóa nghỉ phép)
```

Legacy keys (ví dụ `LoadDS_3_NP`, `InsertInfo_NP`) vẫn dùng được — KHÔNG đổi tên key cũ.

---

## Stored Procedure Call

Khi cần gọi stored procedure thay vì raw SQL:

```javascript
// Frontend
export const loadReport = async (params) => {
  return await axios.post("/api/tsql/procedure/select", {
    procedure: "sp_TenProcedure",
    params: { param1: params.value1 },
  });
};
```

Response check: `res.data?.data?.length > 0`

---

## Batch Insert

```javascript
// Frontend - batch insert nhiều rows
export const batchInsert = async (rows) => {
  return await axios.post("/api/tsql/batch-insert", {
    table: "TABLE_NAME",
    rows: rows,
  });
};
```

Response check: `res.data?.success && res.data?.rowsInserted > 0`

---

## Anti-Patterns (KHÔNG làm)

- ❌ String interpolation trong SQL: `f"WHERE NV_MA = '{ma}'"`
- ❌ Gửi raw SQL từ frontend
- ❌ Đổi tên query key đang dùng (breaking change)
- ❌ Thêm query mà không register trong `__init__.py`
- ❌ Hardcode date values — dùng `GETDATE()`, `YEAR()`, params
