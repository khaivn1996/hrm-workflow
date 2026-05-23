---
name: vue-component
description: Vue 3 component patterns for HRM project. Use when creating, modifying, or reviewing Vue components under hrm_FE/.
---

# Vue Component Skill

> Use this skill whenever working on `.vue` files in `hrm_FE/`.

---

## Component Structure

Every component MUST follow this template:

```vue
<template>
  <!-- Element Plus components only (el-*) -->
</template>

<script setup>
// 1. Vue imports
import { ref, reactive, computed, onMounted, watch } from "vue";
// 2. Element Plus (nếu cần component/icon trực tiếp)
import { ElMessage } from "element-plus";
// 3. Pinia stores
import { useAuthStore } from "@/stores/authStores";
// 4. Composables
import { useExcelExport } from "@/composables/useExcelExport";
// 5. API modules
import { loadSomething, saveSomething } from "@/apis/apiModule";
// 6. Utils
import {
  showLoading,
  hideLoading,
  Success,
  Error,
  Confirm,
  Alert,
} from "@/utils/utils";
import { format } from "date-fns";

// === State ===
const form = reactive({ field1: "", field2: "" });
const tableData = ref([]);
const loading = ref(false);

// === API Calls ===
const loadData = async () => {
  try {
    showLoading();
    const res = await loadSomething(form);
    if (res.data?.data?.length > 0) {
      tableData.value = res.data.data;
    }
  } catch (error) {
    console.error("loadData error:", error);
    Error("Có lỗi xảy ra khi tải dữ liệu.");
  } finally {
    hideLoading();
  }
};

// === Lifecycle ===
onMounted(() => {
  loadData();
});
</script>
```

---

## Reactive State Rules

| Loại        | Dùng         | Ví dụ                                 |
| ----------- | ------------ | ------------------------------------- |
| Giá trị đơn | `ref()`      | `const count = ref(0)`                |
| Object/Form | `reactive()` | `const form = reactive({ name: "" })` |
| Danh sách   | `ref([])`    | `const items = ref([])`               |

---

## API Response Check Patterns

```javascript
// SELECT (registry hoặc procedure)
const res = await registrySelect("QUERY_KEY", params);
if (res.data?.data?.length > 0) {
  /* có data */
}

// CRUD (insert/update/delete)
const res = await registryCrud("QUERY_KEY", params);
if (res.data?.rows_affected > 0) {
  /* thành công */
}

// Batch insert
const res = await registryBatchInsert(tableName, rows);
if (res.data?.success && res.data?.rowsInserted > 0) {
  /* thành công */
}
```

---

## Grid/Table Components

| Loại bảng          | Component             | Khi nào dùng                           |
| ------------------ | --------------------- | -------------------------------------- |
| Danh sách đơn giản | `MultiSelectGrid.vue` | Bảng nhiều dòng, có chọn, search, sort |
| Bảng merge rows    | `GridWithRowSpan.vue` | Cần gộp ô theo cột phân nhóm           |
| Bảng cơ bản        | `Grid.vue`            | Bảng nhỏ, đơn giản                     |

---

## UI Patterns

- **Buttons**: `el-button` với `type="primary"`, `type="danger"`, etc.
- **Form**: `el-form` + `el-form-item` + `el-input` / `el-select` / `el-date-picker`
- **Dialog**: `el-dialog` với `v-model` binding
- **Messages**: Dùng `Success()`, `Error()`, `Confirm()`, `Alert()` từ utils — KHÔNG dùng `ElMessage` trực tiếp
- **Loading**: Luôn dùng `showLoading()` / `hideLoading()` — KHÔNG dùng `v-loading`

---

## Date Handling

```javascript
import { format, parse } from "date-fns";

// Format date
const formatted = format(new Date(), "yyyy-MM-dd");

// Parse "MM-yyyy" string
const [month, year] = form.Thang.split("-");
const firstDay = format(new Date(year, month - 1, 1), "yyyy-MM-dd");
```

---

## Excel Export

```javascript
import { useExcelExport } from "@/composables/useExcelExport";

const { exportExcel, exportExcelFormatted } = useExcelExport();

// Simple export
exportExcel(tableData.value, columns, "filename.xlsx");

// Formatted export (complex headers, merged cells)
exportExcelFormatted(tableData.value, config);
```

---

## Anti-Patterns (KHÔNG làm)

- ❌ Dùng Vuetify components (`v-btn`, `v-card`, etc.)
- ❌ Gọi `axios` trực tiếp từ component
- ❌ Hardcode colors — dùng CSS variables / theme tokens
- ❌ Dùng Options API
- ❌ `ElMessage.success()` trực tiếp — dùng `Success()` wrapper
