# Walkthrough: Heartbeat Session Management

## Tổng quan

Triển khai cơ chế heartbeat để quản lý session đăng nhập. Frontend gửi heartbeat mỗi 60 giây. Server xóa session nếu không nhận heartbeat trong 3 phút → user phải đăng nhập lại khi mở trình duyệt.

## Changes Made

### Backend (6 files)

| File                                                                            | Change | Mô tả                                                                   |
| ------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------------- |
| [models.py](file:///c:/Source/Gitea/HRM/hrm_BE/app/models.py)                   | MODIFY | Thêm `ActiveSession` model                                              |
| [schemas.py](file:///c:/Source/Gitea/HRM/hrm_BE/app/schemas.py)                 | MODIFY | Thêm `HeartbeatResponse`                                                |
| [security.py](file:///c:/Source/Gitea/HRM/hrm_BE/app/security.py)               | MODIFY | Token 8h, session validation, `hash_token()`, `create/delete_session()` |
| [auth.py](file:///c:/Source/Gitea/HRM/hrm_BE/app/routers/auth.py)               | MODIFY | `POST /heartbeat`, login tạo session, logout xóa session                |
| [session_cleanup.py](file:///c:/Source/Gitea/HRM/hrm_BE/app/session_cleanup.py) | NEW    | Background task xóa expired sessions mỗi 60s                            |
| [main.py](file:///c:/Source/Gitea/HRM/hrm_BE/main.py)                           | MODIFY | Register cleanup startup task                                           |

### Frontend (3 files)

| File                                                                                    | Change | Mô tả                                                                     |
| --------------------------------------------------------------------------------------- | ------ | ------------------------------------------------------------------------- |
| [apiLogin.js](file:///c:/Source/Gitea/HRM/hrm_FE/src/apis/apiLogin.js)                  | MODIFY | Thêm `heartbeat()` API                                                    |
| [authStores.js](file:///c:/Source/Gitea/HRM/hrm_FE/src/stores/authStores.js)            | MODIFY | Heartbeat timer: `startHeartbeat()`, `stopHeartbeat()`, `sendHeartbeat()` |
| [axiosInterceptor.js](file:///c:/Source/Gitea/HRM/hrm_FE/src/utils/axiosInterceptor.js) | MODIFY | Skip heartbeat 401 (let authStore handle)                                 |

### Testing (4 files)

| File                                                                            | Change | Mô tả                                                             |
| ------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------- |
| [requirements.txt](file:///c:/Source/Gitea/HRM/hrm_BE/requirements.txt)         | MODIFY | Thêm `pytest`, `pytest-asyncio`                                   |
| [pytest.ini](file:///c:/Source/Gitea/HRM/hrm_BE/pytest.ini)                     | NEW    | Pytest config                                                     |
| [conftest.py](file:///c:/Source/Gitea/HRM/hrm_BE/tests/conftest.py)             | NEW    | Fixtures: SQLite in-memory, test user, auth token, active session |
| [test_heartbeat.py](file:///c:/Source/Gitea/HRM/hrm_BE/tests/test_heartbeat.py) | NEW    | 15 unit tests                                                     |

---

## Test Results

```
======================== 15 passed, 64 warnings in 3.16s ========================
```

| Test Class              | Tests                                                       | Status |
| ----------------------- | ----------------------------------------------------------- | ------ |
| `TestLogin`             | login tạo session, login fail không tạo session             | ✅ 2/2 |
| `TestHeartbeat`         | success, update timestamp, no session → 401, no token → 403 | ✅ 4/4 |
| `TestLogout`            | logout xóa session                                          | ✅ 1/1 |
| `TestSessionValidation` | API với/không có session                                    | ✅ 2/2 |
| `TestSessionCleanup`    | expired detection, active not expired, cleanup only expired | ✅ 3/3 |
| `TestHashToken`         | deterministic, different tokens, length 64                  | ✅ 3/3 |

---

## Cách chạy test

```bash
cd hrm_BE
pip install pytest pytest-asyncio
python -m pytest tests/ -v
```

## Flow hoạt động

```
Login → Server tạo ActiveSession → Frontend start heartbeat (60s)
  ↓
Browser mở → Heartbeat OK → Session alive
  ↓
Browser đóng → Heartbeat dừng → 3 phút → Server cleanup → Session xóa
  ↓
Browser mở lại → API call → Không có session → 401 → Redirect Login
```

## Lưu ý deploy

> [!IMPORTANT]
>
> - Lần deploy đầu tiên, backend sẽ tự tạo bảng `active_sessions` nhờ `Base.metadata.create_all()`
> - Tất cả user hiện tại sẽ cần đăng nhập lại (vì chưa có active session trong DB)
> - Token TTL giảm từ 365 ngày → 8 giờ (backup safety)
