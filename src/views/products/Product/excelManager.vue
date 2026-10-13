<template>
  <div class="variants-excel-page container mt-3 mt-md-4 px-2 px-md-3"
       v-if="checkPermission(['product_view'])">

    <!-- ===== هدر ===== -->
    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2">
          <h3 class="mb-0 page-title">
            <i class="bi bi-file-earmark-excel-fill"></i>
            <span>مدیریت گروهی تنوع‌ها با Excel</span>
          </h3>
          <router-link to="/products" class="btn btn-secondary back-btn">
            <i class="bi bi-arrow-right"></i>
            <span>بازگشت به محصولات</span>
          </router-link>
        </div>
      </div>

      <!-- توضیحات -->
      <div class="card-body p-2 p-md-3">
        <div class="info-box">
          <i class="bi bi-info-circle-fill"></i>
          <div>
            <strong>راهنما:</strong>
            ابتدا فایل اکسل تنوع‌ها را دانلود کنید. فقط ستون‌های
            <span class="badge-edit">موجودی</span>
            <span class="badge-edit">قیمت</span>
            <span class="badge-edit">قیمت حضوری</span>
            قابل ویرایش هستند. پس از ویرایش، فایل را آپلود کنید.
          </div>
        </div>
      </div>
    </div>

    <!-- ===== دو کارت اصلی ===== -->
    <div class="row g-3">

      <!-- کارت Export -->
      <div class="col-12 col-lg-6">
        <div class="card action-card export-card h-100">
          <div class="card-body p-3 p-md-4 d-flex flex-column">

            <div class="action-icon-wrap export-icon">
              <i class="bi bi-download"></i>
            </div>

            <h4 class="action-title">دانلود فایل اکسل</h4>
            <p class="action-desc">
              لیست تنوع‌ها را با اعمال فیلتر دسته‌بندی دانلود کنید.
              حداکثر ۱۰۰ محصول در هر فایل.
            </p>

            <!-- فیلتر دسته‌بندی -->
            <div class="mb-3">
              <label class="form-label">
                <i class="bi bi-funnel"></i>
                فیلتر دسته‌بندی
              </label>
              <multiselect
                v-model="selectedCategories"
                :options="categoryOptions"
                :multiple="true"
                :close-on-select="false"
                :clear-on-select="false"
                :preserve-search="true"
                track-by="id"
                label="title"
                placeholder="همه دسته‌بندی‌ها"
                :custom-label="categoryLabel"
                :loading="categoriesLoading"
                class="custom-multiselect"
              >
                <template #noResult>دسته‌بندی یافت نشد</template>
              </multiselect>
            </div>

            <!-- پیش‌نمایش تعداد -->
            <div v-if="previewCount !== null" class="preview-box mb-3">
              <i class="bi bi-box-seam-fill"></i>
              <span>
                تعداد تقریبی تنوع‌ها:
                <strong>{{ previewCount }}</strong>
              </span>
            </div>

            <button
              class="btn btn-success btn-lg w-100 mt-auto export-btn"
              :disabled="exporting"
              @click="exportExcel"
            >
              <template v-if="exporting">
                <span class="spinner-border spinner-border-sm me-2"></span>
                <span>در حال آماده‌سازی...</span>
              </template>
              <template v-else>
                <i class="bi bi-file-earmark-arrow-down"></i>
                <span>دانلود فایل اکسل</span>
              </template>
            </button>

          </div>
        </div>
      </div>

      <!-- کارت Import -->
      <div class="col-12 col-lg-6">
        <div class="card action-card import-card h-100">
          <div class="card-body p-3 p-md-4 d-flex flex-column">

            <div class="action-icon-wrap import-icon">
              <i class="bi bi-upload"></i>
            </div>

            <h4 class="action-title">آپلود فایل ویرایش‌شده</h4>
            <p class="action-desc">
              فایل اکسل را پس از ویرایش آپلود کنید.
              در صورت وجود خطا، هیچ تغییری اعمال نمی‌شود.
            </p>

            <!-- Dropzone -->
            <div
              class="dropzone mb-3"
              :class="{ 'dragging': isDragging, 'has-file': selectedFile }"
              @dragover.prevent="isDragging = true"
              @dragleave.prevent="isDragging = false"
              @drop.prevent="onDrop"
              @click="$refs.fileInput.click()"
            >
              <input
                ref="fileInput"
                type="file"
                accept=".xlsx,.xls,.csv"
                class="d-none"
                @change="onFileChange"
              />

              <template v-if="!selectedFile">
                <i class="bi bi-cloud-arrow-up dropzone-icon"></i>
                <div class="dropzone-text">
                  فایل را اینجا رها کنید یا کلیک کنید
                </div>
                <div class="dropzone-hint">
                  فرمت‌های مجاز: xlsx، xls، csv — حداکثر ۱۰ مگابایت
                </div>
              </template>

              <template v-else>
                <i class="bi bi-file-earmark-spreadsheet-fill file-icon"></i>
                <div class="file-info">
                  <div class="file-name">{{ selectedFile.name }}</div>
                  <div class="file-size">{{ formatFileSize(selectedFile.size) }}</div>
                </div>
                <button
                  type="button"
                  class="btn btn-sm btn-outline-danger remove-file-btn"
                  @click.stop="clearFile"
                >
                  <i class="bi bi-x-lg"></i>
                </button>
              </template>
            </div>

            <!-- خطاها -->
            <div v-if="importErrors.length" class="errors-box mb-3">
              <div class="errors-header">
                <i class="bi bi-exclamation-triangle-fill"></i>
                <strong>{{ importErrors.length }} خطا یافت شد</strong>
                <button
                  class="btn btn-sm btn-link ms-auto p-0 text-danger"
                  @click="importErrors = []"
                >
                  <i class="bi bi-x-circle"></i>
                </button>
              </div>
              <div class="errors-list">
                <div
                  v-for="(err, idx) in importErrors"
                  :key="idx"
                  class="error-item"
                >
                  <span class="error-row">ردیف {{ err.row }}</span>
                  <span class="error-col">{{ err.column }}</span>
                  <span class="error-msg">{{ err.message }}</span>
                </div>
              </div>
            </div>

            <button
              class="btn btn-primary btn-lg w-100 mt-auto import-btn"
              :disabled="!selectedFile || importing"
              @click="importExcel"
            >
              <template v-if="importing">
                <span class="spinner-border spinner-border-sm me-2"></span>
                <span>در حال آپلود...</span>
              </template>
              <template v-else>
                <i class="bi bi-cloud-arrow-up-fill"></i>
                <span>آپلود و بروزرسانی</span>
              </template>
            </button>

          </div>
        </div>
      </div>

    </div>

    <!-- ===== کارت راهنما ===== -->
    <div class="card mt-3 help-card">
      <div class="card-body p-3 p-md-4">
        <h5 class="help-title">
          <i class="bi bi-question-circle-fill"></i>
          نکات مهم
        </h5>
        <ul class="help-list">
          <li>
            <i class="bi bi-lock-fill"></i>
            ستون‌های <strong>شناسه تنوع</strong>، <strong>نام محصول</strong> و
            <strong>ویژگی‌ها</strong> قفل هستند و نباید تغییر کنند.
          </li>
          <li>
            <i class="bi bi-pencil-fill"></i>
            فقط سه ستون <strong>موجودی</strong>، <strong>قیمت</strong> و
            <strong>قیمت حضوری</strong> قابل ویرایش هستند.
          </li>
          <li>
            <i class="bi bi-shield-check"></i>
            اگر حتی یک سلول نامعتبر باشد، هیچ تغییری در سیستم اعمال نمی‌شود.
          </li>
          <li>
            <i class="bi bi-arrow-repeat"></i>
            موجودی کل محصول به‌صورت خودکار پس از بروزرسانی، بازمحاسبه می‌شود.
          </li>
        </ul>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import Multiselect from "vue-multiselect";
import { useAdmin } from "@/stores/modules/admin";

const store = useAdmin();
const checkPermission = store.checkPermission;

// ===== دسته‌بندی‌ها =====
const categoriesLoading = ref(false);
const categoryOptions = ref([]);
const selectedCategories = ref([]);

async function loadCategories() {
  categoriesLoading.value = true;
  try {
    const { data } = await axios.get("/categories");
    // اگه API ساختار درختی می‌ده، flatten کن
    categoryOptions.value = flattenCategories(data.data ?? data);
  } catch (err) {
    console.error(err);
  } finally {
    categoriesLoading.value = false;
  }
}

function flattenCategories(tree, prefix = "") {
  const result = [];
  for (const node of tree) {
    result.push({
      id: node.id,
      title: prefix ? `${prefix} › ${node.title}` : node.title,
    });
    if (node.all_children && node.all_children.length) {
      result.push(...flattenCategories(node.all_children, node.title));
    }
  }
  return result;
}

function categoryLabel(option) {
  return option.title;
}

// ===== Export =====
const exporting = ref(false);

const categoryIdsParam = computed(() =>
  selectedCategories.value.map((c) => c.id).join(",")
);

async function exportExcel() {
  exporting.value = true;
  try {
    const params = {};
    if (categoryIdsParam.value) params.category_ids = categoryIdsParam.value;

    const response = await axios.get("/sync-product/export", {
      params,
      responseType: "blob",
    });

    // استخراج نام فایل از هدر Content-Disposition
    const disposition = response.headers["content-disposition"] || "";
    const match = disposition.match(/filename="?([^"]+)"?/);
    const filename = match ? match[1] : `variants-${Date.now()}.xlsx`;

    // ساخت لینک دانلود
    const blob = new Blob([response.data], {
      type:
        response.headers["content-type"] ||
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    });
    const url = window.URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = filename;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    window.URL.revokeObjectURL(url);

    Swal.fire({
      icon: "success",
      title: "آماده شد",
      text: "فایل اکسل دانلود شد",
      timer: 2000,
      showConfirmButton: false,
    });
  } catch (err) {
    console.error(err);
    Swal.fire("خطا", "مشکلی در دانلود فایل پیش آمد", "error");
  } finally {
    exporting.value = false;
  }
}

// ===== Import =====
const selectedFile = ref(null);
const isDragging = ref(false);
const importing = ref(false);
const importErrors = ref([]);

function onFileChange(e) {
  const file = e.target.files?.[0];
  if (file) setFile(file);
}

function onDrop(e) {
  isDragging.value = false;
  const file = e.dataTransfer.files?.[0];
  if (file) setFile(file);
}

function setFile(file) {
  const allowed = [
    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    "application/vnd.ms-excel",
    "text/csv",
  ];
  const ext = file.name.split(".").pop().toLowerCase();
  if (!["xlsx", "xls", "csv"].includes(ext)) {
    Swal.fire("فرمت نامعتبر", "فقط xlsx، xls یا csv مجاز است", "warning");
    return;
  }
  if (file.size > 10 * 1024 * 1024) {
    Swal.fire("حجم زیاد", "حجم فایل نباید بیشتر از ۱۰ مگابایت باشد", "warning");
    return;
  }
  selectedFile.value = file;
  importErrors.value = [];
}
function clearFile() {
  selectedFile.value = null;
  importErrors.value = [];
}

function formatFileSize(bytes) {
  if (bytes < 1024) return bytes + " B";
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + " KB";
  return (bytes / (1024 * 1024)).toFixed(2) + " MB";
}

async function importExcel() {
  if (!selectedFile.value) return;

  const confirm = await Swal.fire({
    title: "تأیید بروزرسانی",
    text: "آیا از آپلود و اعمال تغییرات مطمئن هستید؟",
    icon: "question",
    showCancelButton: true,
    confirmButtonText: "بله، اعمال شود",
    cancelButtonText: "انصراف",
    confirmButtonColor: "#16a34a",
  });
  if (!confirm.isConfirmed) return;

  importing.value = true;
  importErrors.value = [];

  try {
    const formData = new FormData();
    formData.append("file", selectedFile.value);

    const { data } = await axios.post("/sync-product/import", formData, {
      headers: { "Content-Type": "multipart/form-data" },
    });

    Swal.fire({
      icon: "success",
      title: "بروزرسانی موفق",
      html: `
        <div class="text-start">
          <p class="mb-1">✅ <strong>${data.data.updated}</strong> تنوع بروزرسانی شد</p>
          <p class="mb-1">⏭️ <strong>${data.data.skipped}</strong> تنوع بدون تغییر</p>
          <p class="mb-0">🔄 <strong>${data.data.synced_products}</strong> محصول sync شد</p>
        </div>
      `,
      confirmButtonText: "عالی",
      confirmButtonColor: "#16a34a",
    });

    clearFile();
  } catch (err) {
    console.log(err);
    
    if (err.response?.status === 422 && err.response.data.errors) {
      importErrors.value = err.response.data.errors;
      Swal.fire({
        icon: "warning",
        title: "خطا در فایل",
        text: err.response.data.message || "فایل نامعتبر است",
        confirmButtonText: "مشاهده جزئیات",
      });
    } else {
      Swal.fire(
        "خطا",
        err.response?.data?.message || "مشکلی در آپلود پیش آمد",
        "error"
      );
    }
  } finally {
    importing.value = false;
  }
}

// ===== Preview (اختیاری - اگه خواستی endpoint بسازی) =====
const previewCount = ref(null);

// اگه خواستی endpoint preview بسازی، اینجا فراخوانی کن
watch(selectedCategories, () => {
  previewCount.value = null;
});

// ===== Init =====
onMounted(() => {
  loadCategories();
});
</script>

<style scoped>
/* ============ هدر ============ */
.header-card .card-header {
  padding: 16px 20px;
  background: transparent;
  border-bottom: 2px solid #f8f9fa;
}

.page-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1.4rem;
  display: flex;
  align-items: center;
  gap: 8px;
}

.back-btn {
  white-space: nowrap;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  justify-content: center;
}

/* ============ Info box ============ */
.info-box {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  color: #1e40af;
  padding: 12px 16px;
  border-radius: 10px;
  font-size: 0.88rem;
  line-height: 1.6;
}

.info-box > i {
  font-size: 1.15rem;
  color: #3b82f6;
  margin-top: 2px;
  flex-shrink: 0;
}

.badge-edit {
  display: inline-block;
  background: #fde68a;
  color: #92400e;
  padding: 1px 8px;
  border-radius: 6px;
  font-weight: 700;
  font-size: 0.78rem;
  margin: 0 2px;
}

/* ============ کارت‌ها ============ */
.action-card {
  border: 1px solid #e9ecef;
  border-radius: 14px;
  overflow: hidden;
  transition: all 0.2s ease;
}

.action-card:hover {
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.07);
}

.export-card {
  border-top: 4px solid #16a34a;
}

.import-card {
  border-top: 4px solid #3b82f6;
}

.action-icon-wrap {
  width: 56px;
  height: 56px;
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.6rem;
  margin-bottom: 14px;
}

.export-icon {
  background: #dcfce7;
  color: #16a34a;
}

.import-icon {
  background: #dbeafe;
  color: #3b82f6;
}

.action-title {
  font-size: 1.15rem;
  font-weight: 700;
  color: #2d3436;
  margin-bottom: 8px;
}

.action-desc {
  color: #6c757d;
  font-size: 0.86rem;
  line-height: 1.7;
  margin-bottom: 16px;
}

/* ============ Multiselect ============ */
.form-label {
  font-size: 0.85rem;
  font-weight: 600;
  color: #495057;
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 6px;
}

.custom-multiselect :deep(.multiselect) {
  min-height: 44px;
  border-radius: 10px;
  border-color: #e0e0e0;
}

.custom-multiselect :deep(.multiselect__tags) {
  border-radius: 10px;
  padding: 6px 12px;
  min-height: 44px;
  border-color: #e0e0e0;
  display: flex;
  align-items: center;
}

.custom-multiselect :deep(.multiselect__placeholder) {
  margin-bottom: 0;
  padding-top: 0;
}

/* ============ Preview box ============ */
.preview-box {
  display: flex;
  align-items: center;
  gap: 8px;
  background: #f0fdf4;
  border: 1px solid #bbf7d0;
  color: #166534;
  padding: 10px 14px;
  border-radius: 10px;
  font-size: 0.86rem;
}

.preview-box > i {
  color: #16a34a;
}

/* ============ Dropzone ============ */
.dropzone {
  position: relative;
  border: 2px dashed #cbd5e1;
  border-radius: 12px;
  padding: 24px 16px;
  text-align: center;
  cursor: pointer;
  transition: all 0.2s ease;
  background: #f8fafc;
  min-height: 160px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.dropzone:hover {
  border-color: #3b82f6;
  background: #eff6ff;
}

.dropzone.dragging {
  border-color: #3b82f6;
  background: #dbeafe;
  transform: scale(1.01);
}

.dropzone.has-file {
  border-color: #16a34a;
  background: #f0fdf4;
  border-style: solid;
}

.dropzone-icon {
  font-size: 2.5rem;
  color: #94a3b8;
  margin-bottom: 10px;
}

.dropzone:hover .dropzone-icon {
  color: #3b82f6;
}

.dropzone-text {
  font-weight: 600;
  color: #475569;
  margin-bottom: 4px;
  font-size: 0.9rem;
}

.dropzone-hint {
  font-size: 0.75rem;
  color: #94a3b8;
}

.file-icon {
  font-size: 2.5rem;
  color: #16a34a;
  margin-bottom: 10px;
}

.file-info {
  text-align: center;
}

.file-name {
  font-weight: 700;
  color: #166534;
  font-size: 0.9rem;
  word-break: break-all;
}

.file-size {
  font-size: 0.75rem;
  color: #6c757d;
  margin-top: 4px;
}

.remove-file-btn {
  position: absolute;
  top: 8px;
  left: 8px;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}

/* ============ Errors box ============ */
.errors-box {
  background: #fef2f2;
  border: 1px solid #fecaca;
  border-radius: 10px;
  overflow: hidden;
}

.errors-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  background: #fee2e2;
  color: #991b1b;
  font-size: 0.85rem;
}

.errors-header > i {
  color: #dc2626;
}

.errors-list {
  max-height: 200px;
  overflow-y: auto;
  padding: 8px 0;
}

.error-item {
  display: grid;
  grid-template-columns: 70px 90px 1fr;
  gap: 8px;
  padding: 6px 14px;
  font-size: 0.78rem;
  color: #7f1d1d;
  border-bottom: 1px solid #fee2e2;
}

.error-item:last-child {
  border-bottom: none;
}

.error-row {
  font-weight: 700;
  color: #991b1b;
}

.error-col {
  color: #b91c1c;
  font-family: monospace;
}

.error-msg {
  color: #7f1d1d;
}

/* ============ Buttons ============ */
.export-btn,
.import-btn {
  font-weight: 700;
  border-radius: 10px;
  padding: 12px 20px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

/* ============ Help card ============ */
.help-card {
  border: 1px solid #e9ecef;
  border-radius: 14px;
}

.help-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 1.05rem;
  font-weight: 700;
  color: #2d3436;
  margin-bottom: 14px;
}

.help-title > i {
  color: #6c5ce7;
}

.help-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.help-list li {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  font-size: 0.88rem;
  color: #495057;
  line-height: 1.6;
}

.help-list li > i {
  color: #6c5ce7;
  font-size: 0.9rem;
  margin-top: 3px;
  flex-shrink: 0;
}

/* ============ Responsive ============ */
@media (max-width: 767.98px) {
  .header-card .card-header {
    padding: 12px 14px;
  }

  .header-card .card-body {
    padding: 12px 14px;
  }

  .page-title {
    font-size: 1.05rem;
    justify-content: center;
    text-align: center;
    width: 100%;
  }

  .back-btn {
    width: 100%;
  }

  .action-icon-wrap {
    width: 48px;
    height: 48px;
    font-size: 1.35rem;
  }

  .action-title {
    font-size: 1rem;
  }

  .dropzone {
    padding: 20px 12px;
    min-height: 140px;
  }

  .dropzone-icon,
  .file-icon {
    font-size: 2rem;
  }

  .error-item {
    grid-template-columns: 1fr;
    gap: 2px;
    padding: 8px 12px;
  }

  .error-row {
    font-size: 0.72rem;
  }
}
</style>