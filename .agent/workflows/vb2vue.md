---
description: vb.net refactor to vue 3
---

# VB.NET → Vue 3 Migration Workflow

This workflow guides the migration of legacy VB.NET WinForms code to modern Vue 3 with Composition API.

## Prerequisites

- Source VB.NET code available
- Target Vue 3 component file exists

## Token Optimization

> **TRƯỚC KHI BẮT ĐẦU**: Đọc KIs thay vì quét project files.
>
> - `hrm-vue3-element-plus-patterns` → conventions, Grid/Dialog/Pinia patterns
> - `hrm-api-service-layer` → API helpers, response check patterns
> - Tìm 1 component tương tự cùng module bằng `grep_search` → dùng làm template
> - Dùng Context7 MCP chỉ khi cần syntax mới (Element Plus, Vue 3 API)

---

## Phase 1: Analysis

### 1.1 Analyze VB.NET Structure

// turbo

```
Identify in the VB.NET code:
- Event handlers (Sub xxx_Click, Sub xxx_Load)
- Form controls (TextBox, ComboBox, DateTimePicker, DataGridView)
- Global variables and objects (objXxx, MAIN.xxx)
- Data access patterns (stored procedures, datasets)
- Dialog forms (frm.ShowDialog())
```

### 1.2 Document Mapping Table

Create a mapping table:

| VB.NET Pattern           | Vue 3 Equivalent                                                   |
| ------------------------ | ------------------------------------------------------------------ |
| `Sub xxx_Click`          | `const xxx = async () => {}`                                       |
| `TextBox.Text`           | `ref('')` or `reactive({})`                                        |
| `DateTimePicker.Value`   | `el-date-picker` with `v-model`                                    |
| `ComboBox.SelectedValue` | `el-select` with `v-model`                                         |
| `DataGridView`           | MultiSelectGrid component                                          |
| `frm.ShowDialog()`       | `el-dialog` with `v-model:visible`                                 |
| `MAIN.userclient`        | `permissionStore.currentUser`                                      |
| `MsgBox()`               | `Success`or `Warning` or `Error` or custom `Success/Error/Warning` |
| `If...Then...Else`       | Standard JS `if...else`                                            |
| `While...End While`      | `for...of` or `while`                                              |
| `Try...Catch`            | `try...catch`                                                      |

---

## Phase 2: State Migration

### 2.1 Convert Global Objects to Reactive State

```javascript
// VB.NET: objChamCong with multiple properties
// Vue 3:
const form = reactive({
  Ngay: format(new Date(), "yyyy-MM-dd"),
  TuNgay: "",
  DenNgay: "",
  FlagChamCong: false,
});

// VB.NET: Global flags
// Vue 3:
const USER_KT = ref(false);
let TABLE_GQT = "";
let TABLE_VR = "";
```

### 2.2 Convert UI Control States

```javascript
// VB.NET: Control.Enabled = False
// Vue 3:
const isFieldDisabled = ref(false);

// VB.NET: Control.Visible = False
// Vue 3:
const showField = ref(true);
// Template: v-if="showField"
```

---

## Phase 3: Event Handler Migration

### 3.1 Button Click Events

```javascript
// VB.NET:
// Private Sub btCapnhat_Click(...)
//     ' logic here
// End Sub

// Vue 3:
const btCapnhat = async () => {
  // logic here
};
```

### 3.2 Form Load Events

```javascript
// VB.NET:
// Private Sub Form_Load(...)
//     ' initialization
// End Sub

// Vue 3:
onMounted(async () => {
  // initialization
});
```

### 3.3 Control Change Events

```javascript
// VB.NET:
// Private Sub cbDonvi_SelectedIndexChanged(...)

// Vue 3:
const oncbDonviChange = async (value) => {
  // handle change
};
// Template: @change="oncbDonviChange"
```

---

## Phase 4: Dialog Migration

### 4.1 Convert ShowDialog to el-dialog

```javascript
// VB.NET:
// Dim frm As New frmCapnhatnghiphep
// frm.ShowDialog()
// If objChamCong.FlagChamCong = True Then...

// Vue 3:
// State
const dialogVisible = ref(false);
const formDialog = reactive({
  TuNgay: "",
  DenNgay: "",
  FlagSuccess: false,
});

// Open dialog
const openDialog = () => {
  formDialog.TuNgay = form.Ngay;
  formDialog.DenNgay = form.Ngay;
  formDialog.FlagSuccess = false;
  dialogVisible.value = true;
};

// Handle confirm
const onDialogConfirm = async () => {
  formDialog.FlagSuccess = true;
  dialogVisible.value = false;
  // Continue processing...
};
```

### 4.2 Dialog Template

```html
<el-dialog
  v-model="dialogVisible"
  title="Dialog Title"
  width="450"
  :close-on-click-modal="false"
>
  <div class="flex flex-col gap-4">
    <!-- Form fields -->
  </div>
  <template #footer>
    <el-button type="primary" @click="onDialogConfirm">Cập nhật</el-button>
    <el-button @click="dialogVisible = false">Hủy</el-button>
  </template>
</el-dialog>
```

---

## Phase 5: Data Access Migration

### 5.1 Convert Stored Procedure Calls

```javascript
// VB.NET:
// objChamCong.ChonBang = 5
// Me.b.UpdateInfo(objChamCong)

// Vue 3:
const res = await apiModule.UpdateInfo({
  NV_MA: form.NV_MA,
  Ngay: ngayValue,
  ChonBang: 5,
  // other params
});
```

### 5.2 Convert DataGridView Loop

```javascript
// VB.NET:
// While i <= Me.dgrBC.Rows.Count - 1
//     objChamCong.NV_MA = Me.dgrBC.Rows(i).Cells("NV_MA").Text
//     ' process
//     i = i + 1
// End While

// Vue 3:
for (const row of tableData.value) {
  const NV_MA = row.NV_MA?.toString() || "";
  // process
}
```

---

## Phase 6: Error Handling & Messages

### 6.1 Convert Message Boxes

```javascript
// VB.NET: MsgBox("Success!", MsgBoxStyle.Information)
// Vue 3:
Success("Thành công!");

// VB.NET: MsgBox("Error!", MsgBoxStyle.Critical)
// Vue 3:
Error("Có lỗi xảy ra!");

// VB.NET: MsgBox("Confirm?", MsgBoxStyle.YesNo)
// Vue 3:
const confirmed = await Confirm("Bạn có chắc chắn?", "Xác nhận");
if (confirmed) {
  /* proceed */
}
```

### 6.2 Error Handling Pattern

```javascript
// VB.NET:
// Try
//     ' code
// Catch ex As Exception
//     MsgBox(ex.Message)
// End Try

// Vue 3:
try {
  showLoading();
  // async operations
  Success("Hoàn thành!");
} catch (error) {
  console.error("Operation error:", error);
  Error("Có lỗi xảy ra. Vui lòng thử lại.");
} finally {
  hideLoading();
}
```

---

## Phase 7: Verification

### 7.1 Checklist

- [ ] All event handlers converted to Vue 3 functions
- [ ] All form controls mapped to reactive state
- [ ] All dialogs converted to el-dialog
- [ ] All API calls use async/await pattern
- [ ] Error handling implemented
- [ ] Loading states handled
- [ ] Grid refresh after operations

### 7.2 Testing

1. Verify UI renders correctly
2. Test all button click handlers
3. Test dialog open/close/confirm flow
4. Test API calls with success/error scenarios
5. Verify grid data updates after operations

---

## Quick Reference: Common Patterns

```javascript
// Date formatting
const ngayValue = format(new Date(form.Ngay), "yyyy/MM/dd");

// Audit fields
const now = new Date();
const NGAYCN = format(now, "yyyy/MM/dd HH:mm:ss");
const NGUOICN = permissionStore.currentUser || "system";

// Batch processing with error tracking
let successCount = 0;
let errorCount = 0;

for (const row of tableData.value) {
  try {
    const res = await apiModule.processRow(row);
    if (res?.data?.success) successCount++;
    else errorCount++;
  } catch (err) {
    console.error("Error:", err);
    errorCount++;
  }
}

// Result message
if (errorCount === 0) {
  Success(`Đã xử lý thành công ${successCount} bản ghi`);
} else {
  Warning(`Thành công: ${successCount}, Lỗi: ${errorCount}`);
}
```
