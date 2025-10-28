<template>
  <div class="admin-shoesoles admin-page">
    <!-- ===== PAGE HEADER ===== -->
    <div class="page-header">
      <div class="header-content">
        <div class="title-section">
          <h1 class="page-title">
            <span class="material-icons">view_day</span>
            Quản lý Loại đế giày
          </h1>
          <p class="page-subtitle">
            Quản lý danh sách loại đế giày cho sản phẩm
          </p>
        </div>
        <div class="header-actions">
          <button class="btn btn-primary" @click="openCreateModal">
            <span class="material-icons">add</span>
            Thêm Loại đế
          </button>
        </div>
      </div>
    </div>

    <!-- ===== STATS GRID ===== -->
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-icon" style="background: var(--gradient-success)">
          <span class="material-icons">check_circle</span>
        </div>
        <div class="stat-content">
          <div class="stat-value">{{ activeShoeSolesCount }}</div>
          <div class="stat-label">ĐANG HOẠT ĐỘNG</div>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-icon" style="background: var(--gradient-warning)">
          <span class="material-icons">pause_circle</span>
        </div>
        <div class="stat-content">
          <div class="stat-value">{{ inactiveShoeSolesCount }}</div>
          <div class="stat-label">TẠM NGƯNG</div>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-icon" style="background: var(--gradient-info)">
          <span class="material-icons">inventory</span>
        </div>
        <div class="stat-content">
          <div class="stat-value">{{ shoeSoles.length }}</div>
          <div class="stat-label">TỔNG LOẠI ĐẾ</div>
        </div>
      </div>
    </div>

    <!-- ===== FILTERS ===== -->
    <div class="content-section">
      <div class="search-filters">
        <div class="search-box">
          <span class="material-icons search-icon">search</span>
          <input
            type="text"
            class="search-input"
            v-model="searchKeyword"
            placeholder="Tìm theo tên loại đế..."
          />
        </div>

        <select class="filter-select" v-model="filterStatus">
          <option value="all">Tất cả trạng thái</option>
          <option value="active">Đang hoạt động</option>
          <option value="inactive">Tạm ngưng</option>
        </select>

        <button class="btn btn-ghost" @click="resetFilters">
          <span class="material-icons">refresh</span>
          Làm mới
        </button>
      </div>
    </div>

    <!-- ===== LOADING STATE ===== -->
    <div v-if="loading" class="admin-loading">
      <div class="loading-spinner"></div>
      <p class="loading-text">Đang tải dữ liệu...</p>
    </div>

    <!-- ===== EMPTY STATE ===== -->
    <div v-else-if="filteredShoeSoles.length === 0" class="admin-empty-state">
      <div class="empty-state-icon">
        <span class="material-icons">view_day</span>
      </div>
      <h3 class="empty-state-title">Không có loại đế nào</h3>
      <p class="empty-state-description">
        Bắt đầu thêm loại đế đầu tiên cho cửa hàng của bạn
      </p>
      <button class="btn btn-primary" @click="openCreateModal">
        <span class="material-icons">add</span>
        Thêm Loại đế
      </button>
    </div>

    <!-- ===== SHOE SOLES TABLE ===== -->
    <div v-else class="table-container">
      <table class="admin-table shoesoles-table">
        <thead>
          <tr>
            <th style="width: 80px">ID</th>
            <th>Tên loại đế</th>
            <th>Slug</th>
            <th style="width: 150px">Trạng thái</th>
            <th style="width: 180px">Ngày tạo</th>
            <th style="width: 150px">Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="sole in paginatedShoeSoles" :key="sole.id">
            <td>{{ sole.id }}</td>
            <td>
              <strong>{{ sole.name }}</strong>
              <p v-if="sole.description" class="desc">
                {{ truncateText(sole.description, 50) }}
              </p>
            </td>
            <td>
              <code class="code-badge">{{ sole.slug }}</code>
            </td>
            <td>
              <span
                class="status-badge"
                :class="sole.isActive ? 'status-active' : 'status-inactive'"
              >
                <span class="material-icons">{{
                  sole.isActive ? "check_circle" : "cancel"
                }}</span>
                {{ sole.isActive ? "Hoạt động" : "Tạm ngưng" }}
              </span>
            </td>
            <td>{{ formatDate(sole.createdAt) }}</td>
            <td>
              <div class="cell-actions">
                <button
                  class="btn-icon btn-edit"
                  @click="openEditModal(sole)"
                  title="Chỉnh sửa"
                >
                  <span class="material-icons">edit</span>
                </button>
                <button
                  class="btn-icon btn-delete"
                  @click="confirmDelete(sole)"
                  title="Xóa"
                >
                  <span class="material-icons">delete</span>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- ===== PAGINATION ===== -->
    <div v-if="totalPages > 1" class="table-pagination">
      <button
        class="pagination-btn"
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        <span class="material-icons">chevron_left</span>
        Trước
      </button>

      <div class="pagination-info">
        Trang {{ currentPage }} / {{ totalPages }}
      </div>

      <button
        class="pagination-btn"
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        Sau
        <span class="material-icons">chevron_right</span>
      </button>
    </div>

    <!-- ===== CREATE/EDIT MODAL ===== -->
    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal modal-lg" @click.stop>
        <div class="modal-header">
          <h2 class="modal-title">
            <span class="material-icons">{{
              isEditMode ? "edit" : "add"
            }}</span>
            {{ isEditMode ? "Chỉnh sửa Loại đế" : "Thêm Loại đế mới" }}
          </h2>
        </div>

        <div class="modal-body">
          <form @submit.prevent="saveShoeSole">
            <div class="form-row">
              <!-- 🔹 Tên loại đế -->
              <div class="form-group">
                <label class="form-label required">Tên loại đế</label>
                <input
                  v-model="formData.name"
                  type="text"
                  class="form-control"
                  placeholder="VD: Đế cao su, Đế EVA..."
                  @input="generateSlug"
                  required
                />
                <span v-if="formErrors?.name" class="form-error">{{
                  formErrors.name
                }}</span>
              </div>

              <!-- 🔹 Slug -->
              <div class="form-group">
                <label class="form-label required">Slug</label>
                <input
                  v-model="formData.slug"
                  type="text"
                  class="form-control"
                  placeholder="vd: de-cao-su, de-eva..."
                  required
                />
                <span v-if="formErrors?.slug" class="form-error">{{
                  formErrors.slug
                }}</span>
                <span class="form-help"
                  >URL thân thiện (tự động tạo từ tên)</span
                >
              </div>
            </div>

            <div class="form-group">
              <label class="form-label">Mô tả</label>
              <textarea
                class="form-control"
                v-model="formData.description"
                placeholder="Nhập mô tả về loại đế..."
                rows="4"
              ></textarea>
            </div>

            <div class="form-check">
              <input
                type="checkbox"
                class="form-check-input"
                v-model="formData.isActive"
                id="isActive"
              />
              <label class="form-check-label" for="isActive">
                Kích hoạt loại đế
              </label>
            </div>

            <div class="form-actions right">
              <button type="button" class="btn btn-outline" @click="closeModal">
                <span class="material-icons">close</span>
                Hủy
              </button>
              <button
                type="submit"
                class="btn btn-primary"
                :class="{ 'btn-loading': saving }"
                :disabled="saving"
              >
                <span class="material-icons">{{
                  saving ? "hourglass_empty" : "save"
                }}</span>
                {{ saving ? "Đang lưu..." : "Lưu" }}
              </button>
            </div>
          </form>
        </div>

        <div class="modal-footer">
          <button class="modal-close btn btn-secondary" @click="closeModal">
            Đóng
          </button>
        </div>
      </div>
    </div>

    <!-- ===== DELETE CONFIRMATION MODAL ===== -->
    <ConfirmDialog
      v-model="showDeleteModal"
      type="danger"
      title="Xác nhận xóa loại đế"
      :message="`Bạn có chắc chắn muốn xóa loại đế '${shoeSoleToDelete?.name}'?`"
      description="Hành động này không thể hoàn tác."
      confirm-text="Xóa loại đế"
      cancel-text="Hủy"
      :loading="deleting"
      @confirm="deleteShoeSole"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { ElMessage } from "element-plus";
import { useAdminStore } from "@/stores/admin";
import ConfirmDialog from "@/assets/components/common/ConfirmDialog.vue";

const adminStore = useAdminStore();

// State
const loading = ref(false);
const saving = ref(false);
const deleting = ref(false);
const shoeSoles = ref([]);
const searchKeyword = ref("");
const filterStatus = ref("all");
const showModal = ref(false);
const showDeleteModal = ref(false);
const isEditMode = ref(false);
const shoeSoleToDelete = ref(null);
const currentPage = ref(1);
const itemsPerPage = 10;

// Form data
const formData = ref({
  id: null,
  name: "",
  slug: "",
  description: "",
  isActive: true,
});

// (Optional) form errors if bạn có validate từ BE
const formErrors = ref({});

// Computed
const filteredShoeSoles = computed(() => {
  let result = shoeSoles.value;

  // Filter by search
  if (searchKeyword.value) {
    const keyword = searchKeyword.value.toLowerCase();
    result = result.filter(
      (item) =>
        item.name.toLowerCase().includes(keyword) ||
        item.slug.toLowerCase().includes(keyword)
    );
  }

  // Filter by status
  if (filterStatus.value !== "all") {
    const isActive = filterStatus.value === "active";
    result = result.filter((item) => item.isActive === isActive);
  }

  return result;
});

const paginatedShoeSoles = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredShoeSoles.value.slice(start, end);
});

const totalPages = computed(() =>
  Math.ceil(filteredShoeSoles.value.length / itemsPerPage)
);

const activeShoeSolesCount = computed(
  () => shoeSoles.value.filter((b) => b.isActive).length
);
const inactiveShoeSolesCount = computed(
  () => shoeSoles.value.filter((b) => !b.isActive).length
);

// Methods
const fetchSoles = async () => {
  loading.value = true;
  try {
    const result = await adminStore.fetchSoles();
    shoeSoles.value = result.content || result || [];
  } catch (error) {
    console.error("Error fetching shoe soles:", error);
    ElMessage.error({
      message: "Lỗi khi tải danh sách loại đế",
      duration: 3000,
    });
  } finally {
    loading.value = false;
  }
};

const openCreateModal = () => {
  isEditMode.value = false;
  formData.value = {
    id: null,
    name: "",
    slug: "",
    description: "",
    isActive: true,
  };
  formErrors.value = {};
  showModal.value = true;
};

const openEditModal = (sole) => {
  isEditMode.value = true;
  formData.value = { ...sole };
  formErrors.value = {};
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
  formData.value = {
    id: null,
    name: "",
    slug: "",
    description: "",
    isActive: true,
  };
  formErrors.value = {};
};

const generateSlug = () => {
  formData.value.slug = formData.value.name
    .toLowerCase()
    .normalize("NFD") // bỏ dấu tiếng Việt
    .replace(/[\u0300-\u036f]/g, "") // loại bỏ ký tự tổ hợp
    .replace(/đ/g, "d")
    .replace(/[^a-z0-9\s-]/g, "") // chỉ giữ chữ cái, số, dấu cách và dấu -
    .replace(/\s+/g, "-") // thay khoảng trắng bằng dấu -
    .replace(/-+/g, "-") // gộp dấu - liên tiếp
    .trim();
};

const saveShoeSole = async () => {
  saving.value = true;
  try {
    if (isEditMode.value) {
      await adminStore.updateSole(formData.value.id, formData.value);
    } else {
      await adminStore.createSole(formData.value);
    }
    await fetchSoles();
    closeModal();
    ElMessage.success({
      message: `${isEditMode.value ? "Cập nhật" : "Thêm"} loại đế thành công!`,
      duration: 3000,
    });
  } catch (error) {
    console.error("Error saving shoe sole:", error);

    // Nếu BE trả về lỗi validate, bạn có thể map vào formErrors
    // ví dụ: if (error.response?.data?.validationErrors) { ... }

    ElMessage.error({
      message: "Lỗi khi lưu loại đế",
      duration: 3000,
    });
  } finally {
    saving.value = false;
  }
};

const confirmDelete = (sole) => {
  shoeSoleToDelete.value = sole;
  showDeleteModal.value = true;
};

const deleteShoeSole = async () => {
  deleting.value = true;
  try {
    await adminStore.deleteShoeSole(shoeSoleToDelete.value.id);
    await fetchSoles();
    showDeleteModal.value = false;
    shoeSoleToDelete.value = null;
    ElMessage.success({
      message: "Xóa loại đế thành công!",
      duration: 3000,
    });
  } catch (error) {
    console.error("Error deleting shoe sole:", error);
    ElMessage.error({
      message: "Lỗi khi xóa loại đế",
      duration: 3000,
    });
  } finally {
    deleting.value = false;
  }
};

const resetFilters = () => {
  searchKeyword.value = "";
  filterStatus.value = "all";
  currentPage.value = 1;
};

const formatDate = (dateString) => {
  if (!dateString) return "—";
  const date = new Date(dateString);
  return date.toLocaleDateString("vi-VN");
};

const truncateText = (text, maxLength) => {
  if (!text) return "";
  return text.length > maxLength ? text.substring(0, maxLength) + "..." : text;
};

// Lifecycle
onMounted(() => {
  fetchSoles();
});
</script>

<style scoped>
/* Mượn style từ Brand/Material để đồng bộ */
.desc {
  color: var(--text-tertiary);
  font-size: var(--text-xs);
  margin: 0;
}

/* Badge code dùng chung */
.code-badge {
  background: rgba(167, 139, 250, 0.1);
  color: var(--primary-light);
  padding: var(--space-1) var(--space-2);
  border-radius: var(--radius-sm);
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  border: 1px solid rgba(167, 139, 250, 0.2);
}

/* Table spacing cân đối như Brand */
.table-container .admin-table th,
.table-container .admin-table td {
  vertical-align: middle;
}
</style>
