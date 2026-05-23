---
description: Phân tích XML column definitions và tự động cập nhật Grid config (cols_all, cols_display, cols_labels, cols_types) trong Vue component
---

# /refactorxml - XML Column Definitions → Vue Grid Config

Workflow này giúp AI tự động phân tích đoạn XML column definitions (dạng `<coldgrtieuchuan>`) và cập nhật 4 blocks Grid config trong Vue component.

## Yêu cầu đầu vào từ User

User sẽ cung cấp:
1. **Đoạn XML** chứa các thẻ `<coldgrtieuchuan>` với các thuộc tính: `tblColName`, `title`, `hide`, `width`, `positionCol`
2. **4 vị trí code** cần chỉnh sửa trong file Vue component (thường dưới dạng `@[file:L-L]`):
   - `Grid_X_cols_all` — mảng tất cả column names
   - `Grid_X_cols_display` — mảng column names hiển thị (hide="false")
   - `Grid_X_cols_labels` — object map column → title
   - `Grid_X_cols_types` — object map column → data type

## Step 1: Phân tích XML

// turbo

```
Đọc đoạn XML và trích xuất thông tin từ mỗi thẻ <coldgrtieuchuan>:

1. tblColName → Chuyển toàn bộ sang UPPERCASE (ví dụ: NV_Ngayvao → NV_NGAYVAO, TCThoRapCU → TCTHORAPCU)
2. title → Giữ nguyên dùng cho labels
3. hide → Xác định true/false (case-insensitive: "TRUE", "True", "true" đều = ẩn)
4. Sắp xếp theo positionCol tăng dần
```

## Step 2: Phân loại data types

// turbo

```
Dựa vào title và tên cột, gán kiểu dữ liệu cho mỗi cột:

| Tiêu chí                         | Type       |
|-----------------------------------|------------|
| STT                               | text       |
| TT (Thao tác)                     | action     |
| Tên chứa "Ngày", "ngày", "Date"  | date       |
| Mã (NV_MA, DV_MA, T_MA...)       | text       |
| Tên (NV_TEN, CVU_TEN...)         | text       |
| Bậc lương (BACLUONG*)            | text       |
| Ghi chú (GHICHU)                 | text       |
| Hệ số, Lương, TC*, TONG*         | number     |
| Mặc định nếu không rõ            | text       |

Lưu ý:
- Các cột TC* (Trợ cấp) luôn là "number"
- Các cột TONG* (Tổng) luôn là "number"
- Các cột HESO* (Hệ số) luôn là "number"
- Các cột LUONG* (Lương) luôn là "number"
- BACLUONGCU, BACLUONGMOI là "text" (mã bậc lương, không phải số)
```

## Step 3: Tạo Grid_X_cols_all

// turbo

```
Tạo mảng chứa TẤT CẢ tblColName (đã UPPERCASE), bao gồm cả cột ẩn.
Luôn giữ lại "STT" và "TT" ở đầu mảng (nếu có trong code gốc).

Ví dụ:
const Grid_0_cols_all = [
  "STT",
  "TT",
  "NV_MA",
  "NV_TEN",
  ...tất cả cột từ XML...
];
```

## Step 4: Tạo Grid_X_cols_display

// turbo

```
Tạo mảng chỉ chứa các tblColName có hide="false" (case-insensitive).
Loại bỏ các cột có hide="true"/"TRUE".
Luôn giữ lại "STT" và "TT" ở đầu mảng (nếu có trong code gốc).

Ví dụ:
const Grid_0_cols_display = [
  "STT",
  "TT",
  "NV_MA",
  "NV_TEN",
  ...chỉ cột visible...
];
```

## Step 5: Tạo Grid_X_cols_labels

// turbo

```
Tạo object map từ tblColName (UPPERCASE) → title (giữ nguyên tiếng Việt từ XML).
Bao gồm cả "STT" và "TT" (nếu có trong code gốc).

Ví dụ:
const Grid_0_cols_labels = {
  STT: "STT",
  TT: "Thao tác",
  NV_MA: "Số thẻ",
  NV_TEN: "Họ Tên",
  ...
};
```

## Step 6: Tạo Grid_X_cols_types

// turbo

```
Tạo object map từ tblColName (UPPERCASE) → data type (theo bảng phân loại ở Step 2).
Bao gồm cả "STT" → "text" và "TT" → "action" (nếu có trong code gốc).

Ví dụ:
const Grid_0_cols_types = {
  STT: "text",
  TT: "action",
  NV_MA: "text",
  NV_NGAYVAO: "date",
  HESOCU: "number",
  ...
};
```

## Step 7: Áp dụng vào file Vue

// turbo

```
Sử dụng multi_replace_file_content để thay thế đồng thời cả 4 blocks trong file Vue component
tại đúng vị trí mà user đã chỉ định.

Lưu ý quan trọng:
- KHÔNG thay đổi các Grid_1_*, Grid_2_*, Grid_3_* (trừ khi user yêu cầu)
- KHÔNG thay đổi code logic bên ngoài 4 blocks được chỉ định
- Giữ nguyên format code (indent 2 spaces, trailing comma)
- Đảm bảo tất cả tblColName đều UPPERCASE trong code output
```

## Checklist cuối cùng

```
Sau khi hoàn thành, báo cáo cho user:
1. Tổng số cột trong cols_all
2. Số cột bị ẩn (hide=true) → liệt kê tên
3. Số cột hiển thị trong cols_display
4. Phân loại types: bao nhiêu text, number, date, action
5. Các cột đã được chuyển UPPERCASE (nếu tên gốc có chữ thường)
```
