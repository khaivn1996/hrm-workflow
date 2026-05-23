---
description: Workflow debug/fix bug có hệ thống cho HRM project
---

# Workflow: Bugfix

> Quy trình debug từ symptom → root cause → fix → verify.

---

## Phase 1: Thu thập thông tin

1. **Xác định symptom** — mô tả chính xác lỗi: UI sai? API trả lỗi? Data không đúng?
2. **Xác định scope** — lỗi ở đâu?
   - Frontend (Vue component, API call, state)?
   - Backend (FastAPI endpoint, SQL query, registry)?
   - Database (data sai, stored procedure)?
3. **Check console/network** — xem browser DevTools:
   - Console errors (JavaScript)
   - Network tab: API request/response, status code
   - Vue DevTools: component state, Pinia store

---

## Phase 2: Reproduce & Trace

4. **Reproduce lỗi** — tìm bước tái hiện cụ thể
5. **Trace call flow**:

   ```
   Component → API module (src/apis/) → Backend endpoint → Query Registry → SQL
   ```

   - Frontend: tìm hàm gọi API trong component → check API module
   - Backend: tìm endpoint trong `routers/` → check service layer → check query key
   - SQL: tìm query trong `queries/*_queries.py` → check params, logic

6. **Check API response pattern**:

   ```javascript
   // SELECT → check data
   res.data?.data?.length > 0;

   // CRUD → check rows affected
   res.data?.rows_affected > 0;

   // Batch → check success
   res.data?.success && res.data?.rowsInserted > 0;
   ```

---

## Phase 3: Root Cause

7. **Xác định root cause** — phân loại:
   - **Data bug**: SQL query trả sai data → fix SQL
   - **Logic bug**: Frontend xử lý sai data → fix JS logic
   - **API bug**: Endpoint trả sai format → fix backend
   - **State bug**: Pinia store / reactive state sai → fix state management
   - **Session bug**: JWT expired, heartbeat fail → check auth flow

8. **Kiểm tra liên quan** — lỗi có ảnh hưởng module khác không?

---

## Phase 4: Fix

9. **Minimal fix** — chỉ sửa đúng root cause
10. **Không refactor** — không sửa code không liên quan
11. **Không đổi API contract** — giữ nguyên response format trừ khi được yêu cầu
12. **Match style** — code mới phải theo pattern hiện có

---

## Phase 5: Verify

13. **Test fix** — lỗi đã hết chưa?
14. **Test regression** — các chức năng liên quan có bị ảnh hưởng không?
15. **Backend test** (nếu có):
    ```bash
    cd hrm_BE
    python -m pytest tests/ -v
    ```

---

## Checklist trước khi báo done

- [ ] Root cause đã được xác định và giải thích
- [ ] Fix là minimal — không có thay đổi không liên quan
- [ ] Đã test chức năng bị lỗi
- [ ] Đã check regression trên chức năng liên quan
- [ ] Đã tóm tắt: file changed, what changed, risks
