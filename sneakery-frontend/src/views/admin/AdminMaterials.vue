<template>
  <div class="admin-materials admin-page">
    <!-- ===== PAGE HEADER ===== -->
    <div class="page-header">
      <div class="header-content">
        <div class="title-section">
          <h1 class="page-title">
            <span class="material-icons">layers</span>
            Quản lý Chất liệu
          </h1>
          <p class="page-subtitle">Quản lý danh sách chất liệu sản phẩm</p>
        </div>
        <div class="header-actions">
          <button class="btn btn-primary" @click="openCreateModal">
            <span class="material-icons">add</span>
            Thêm Chất liệu
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
          <div class="stat-value">{{ activeMaterialsCount }}</div>
          <div class="stat-label">ĐANG HOẠT ĐỘNG</div>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-icon" style="background: var(--gradient-warning)">
          <span class="material-icons">pause_circle</span>
        </div>
        <div class="stat-content">
          <div class="stat-value">{{ inactiveMaterialsCount }}</div>
          <div class="stat-label">TẠM NGƯNG</div>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-icon" style="background: var(--gradient-info)">
          <span class="material-icons">inventory</span>
        </div>
        <div class="stat-content">
          <div class="stat-value">{{ materials.length }}</div>
          <div class="stat-label">TỔNG CHẤT LIỆU</div>
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
            placeholder="Tìm theo tên chất liệu..."
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
    <div v-else-if="filteredMaterials.length === 0" class="admin-empty-state">
      <div class="empty-state-icon">
        <span class="material-icons">layers</span>
      </div>
      <h3 class="empty-state-title">Không có chất liệu nào</h3>
      <p class="empty-state-description">
        Bắt đầu thêm chất liệu đầu tiên cho cửa hàng của bạn
      </p>
      <button class="btn btn-primary" @click="openCreateModal">
        <span class="material-icons">add</span>
        Thêm Chất liệu
      </button>
    </div>

    <!-- ===== MATERIALS TABLE ===== -->
    <div v-else class="table-container">
      <table class="admin-table materials-table">
        <thead>
          <tr>
            <th style="width: 80px">ID</th>
            <th>Tên chất liệu</th>
            <th>Slug</th>
            <th style="width: 150px">Trạng thái</th>
            <th style="width: 180px">Ngày tạo</th>
            <th style="width: 150px">Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="material in paginatedMaterials" :key="material.id">
            <td>{{ material.id }}</td>
            <td>
              <strong>{{ material.name }}</strong>
              <p v-if="material.description" class="brand-desc">
                {{ truncateText(material.description, 50) }}
              </p>
            </td>
            <td>
              <code class="code-badge">{{ material.slug }}</code>
            </td>
            <td>
              <span
                class="status-badge"
                :class="material.isActive ? 'status-active' : 'status-inactive'"
              >
                <span class="material-icons">{{
                  material.isActive ? "check_circle" : "cancel"
                }}</span>
                {{ material.isActive ? "Hoạt động" : "Tạm ngưng" }}
              </span>
            </td>
            <td>{{ formatDate(material.createdAt) }}</td>
            <td>
              <div class="cell-actions">
                <button
                  class="btn-icon btn-edit"
                  @click="openEditModal(material)"
                  title="Chỉnh sửa"
                >
                  <span class="material-icons">edit</span>
                </button>
                <button
                  class="btn-icon btn-delete"
                  @click="confirmDelete(material)"
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
            {{ isEditMode ? "Chỉnh sửa Chất liệu" : "Thêm Chất liệu mới" }}
          </h2>
          <button class="modal-close" @click="closeModal">
            <span class="material-icons">close</span>
          </button>
        </div>

        <div class="modal-body">
          <form @submit.prevent="saveMaterial">
            <div class="form-row">
              <!-- 🔹 Tên chất liệu -->
              <div class="form-group">
                <label class="form-label required">Tên chất liệu</label>
                <input
                  v-model="formData.name"
                  type="text"
                  class="form-control"
                  placeholder="VD: Da thật, Vải mesh..."
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
                  placeholder="vd: da-that, vai-mesh..."
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
                placeholder="Nhập mô tả về chất liệu..."
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
                Kích hoạt chất liệu
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
      </div>
    </div>

    <!-- ===== DELETE CONFIRMATION MODAL ===== -->
    <ConfirmDialog
      v-model="showDeleteModal"
      type="danger"
      title="Xác nhận xóa chất liệu"
      :message="`Bạn có chắc chắn muốn xóa chất liệu '${materialToDelete?.name}'?`"
      description="Hành động này không thể hoàn tác."
      confirm-text="Xóa chất liệu"
      cancel-text="Hủy"
      :loading="deleting"
      @confirm="deleteMaterial"
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
const materials = ref([]);
const searchKeyword = ref("");
const filterStatus = ref("all");
const showModal = ref(false);
const showDeleteModal = ref(false);
const isEditMode = ref(false);
const materialToDelete = ref(null);
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

// Computed
const filteredMaterials = computed(() => {
  let result = materials.value;
  if (searchKeyword.value) {
    const keyword = searchKeyword.value.toLowerCase();
    result = result.filter(
      (m) =>
        m.name.toLowerCase().includes(keyword) ||
        m.slug.toLowerCase().includes(keyword)
    );
  }
  if (filterStatus.value !== "all") {
    const isActive = filterStatus.value === "active";
    result = result.filter((m) => m.isActive === isActive);
  }
  return result;
});

const paginatedMaterials = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredMaterials.value.slice(start, end);
});

const totalPages = computed(() =>
  Math.ceil(filteredMaterials.value.length / itemsPerPage)
);

const activeMaterialsCount = computed(
  () => materials.value.filter((b) => b.isActive).length
);
const inactiveMaterialsCount = computed(
  () => materials.value.filter((b) => !b.isActive).length
);

// Methods
const fetchMaterials = async () => {
  loading.value = true;
  try {
    const result = await adminStore.fetchMaterials();
    materials.value = result.content || result || [];
  } catch (error) {
    console.error("Error fetching materials:", error);
    ElMessage.error({
      message: "Lỗi khi tải danh sách chất liệu",
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
  showModal.value = true;
};

const openEditModal = (material) => {
  isEditMode.value = true;
  formData.value = { ...material };
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
};

const generateSlug = () => {
  formData.value.slug = formData.value.name
    .toLowerCase()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .replace(/đ/g, "d")
    .replace(/[^a-z0-9\s-]/g, "")
    .replace(/\s+/g, "-")
    .replace(/-+/g, "-")
    .trim();
};

const saveMaterial = async () => {
  saving.value = true;
  try {
    if (isEditMode.value) {
      await adminStore.updateMaterial(formData.value.id, formData.value);
    } else {
      await adminStore.createMaterial(formData.value);
    }
    await fetchMaterials();
    closeModal();
    ElMessage.success({
      message: `${
        isEditMode.value ? "Cập nhật" : "Thêm"
      } chất liệu thành công!`,
      duration: 3000,
    });
  } catch (error) {
    console.error("Error saving material:", error);
    ElMessage.error({
      message: "Lỗi khi lưu chất liệu",
      duration: 3000,
    });
  } finally {
    saving.value = false;
  }
};

const confirmDelete = (material) => {
  materialToDelete.value = material;
  showDeleteModal.value = true;
};

const deleteMaterial = async () => {
  deleting.value = true;
  try {
    await adminStore.deleteMaterial(materialToDelete.value.id);
    await fetchMaterials();
    showDeleteModal.value = false;
    materialToDelete.value = null;
    ElMessage.success({
      message: "Xóa chất liệu thành công!",
      duration: 3000,
    });
  } catch (error) {
    console.error("Error deleting material:", error);
    ElMessage.error({
      message: "Lỗi khi xóa chất liệu",
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
  fetchMaterials();
});
</script>

<style scoped>
.brand-desc {
  color: var(--text-tertiary);
  font-size: var(--text-xs);
  margin: 0;
}
</style>
