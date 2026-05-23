---
description: Workflow tạo module/report mới end-to-end cho HRM project
---

# Workflow: New Module

> Checklist tạo module mới từ frontend đến backend, đảm bảo không bỏ sót bước nào.

---

## Phase 1: Phân tích yêu cầu

1. **Xác định loại module**:
   - CRUD form (quản lý dữ liệu)?
   - Report (báo cáo, thống kê)?
   - Batch operation (xử lý hàng loạt)?

2. **Xác định data sources**:
   - Bảng nào trong SQL Server?
   - Stored procedure hay raw SQL?
   - Cần join những bảng nào?

3. **Xác định UI**:
   - Grid loại nào? (`MultiSelectGrid` / `GridWithRowSpan` / `Grid`)
   - Có form nhập liệu không?
   - Có export Excel không?
   - Có report PDF không?

---

## Phase 2: Backend

4. **Tạo query file**:

   ```
   hrm_BE/app/queries/tenmodule_queries.py
   ```

   - Naming: `TENMODULE_{ACTION}_{DESC}`
   - Dùng `:paramName` cho parameters
   - Comment mô tả + params cho mỗi query

5. **Register queries**:

   ```
   hrm_BE/app/queries/__init__.py
   ```

   - Import module queries
   - Spread vào `ALL_QUERIES`

6. **Tạo endpoint** (nếu cần logic phức tạp):
   ```
   hrm_BE/app/routers/tenmodule_router.py
   ```

   - Đa số trường hợp dùng registry endpoint có sẵn là đủ
   - Chỉ tạo router mới khi cần xử lý phức tạp (multi-query, transaction, etc.)

// turbo 7. **Test backend**:

```bash
cd hrm_BE && python -m pytest tests/ -v
```

---

## Phase 3: Frontend

8. **Tạo API module** (hoặc thêm vào file có sẵn):

   ```
   hrm_FE/src/apis/apiTenmodule.js
   ```

   - Import axios + helpers
   - Export async functions cho mỗi API call

9. **Tạo Vue component**:

   ```
   hrm_FE/src/views/MODULE/TenComponent.vue
   ```

   - `<script setup>` + Composition API
   - Import order: vue → element-plus → pinia → composables → apis → utils
   - State: `ref()` / `reactive()`
   - API calls với `showLoading()` / `hideLoading()`

10. **Thêm route**:

    ```
    hrm_FE/src/router/index.js
    ```

    - Thêm route mới với `meta: { requiresAuth: true, permission: "MA_QUYEN" }`
    - Lazy load: `component: () => import("@/views/MODULE/TenComponent.vue")`

11. **Thêm menu item** (nếu cần):
    - Cập nhật sidebar/navigation component
    - Check permission hiển thị

---

## Phase 4: Tích hợp

12. **Wire up UI + API**:
    - Form submit → gọi API → check response → show message
    - Load data on mount hoặc on event

13. **Excel export** (nếu cần):

    ```javascript
    import { useExcelExport } from "@/composables/useExcelExport";
    const { exportExcel } = useExcelExport();
    ```

14. **PDF report** (nếu cần):
    ```javascript
    import { loadReport } from "@/apis/apiMain";
    const blob = await loadReport("TenReport", "pdf", params);
    ```

---

## Phase 5: Verify

15. **Test CRUD flow**: Create → Read → Update → Delete
16. **Test validation**: Required fields, date ranges, edge cases
17. **Test permissions**: User không có quyền → redirect/hide
18. **Test export**: Excel output đúng format

---

## Checklist trước khi done

- [ ] Backend query file created + registered in `__init__.py`
- [ ] Frontend API module created
- [ ] Vue component created with `<script setup>`
- [ ] Route added with permission guard
- [ ] CRUD hoạt động đầy đủ
- [ ] Response checks đúng pattern (data.length / rows_affected / success)
- [ ] `showLoading` / `hideLoading` bao đủ
- [ ] Messages tiếng Việt cho user
- [ ] Nhắc user chạy `/autodocs` để cập nhật docs
