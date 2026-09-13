<template>
  <RouterView />
  <!-- Sidebar Component -->
  <dashPageView />
  <div class="content">
    <!-- ================= HEADER ================= -->
    <div class="staff-header">
      <div>
        <h2>Teachers Management</h2>
        <p>Manage Teachers academic and professional information</p>
      </div>

      <div class="staff-header-right">
        <div class="staff-summary">
          <div class="summary-icon">
            <i class="bi bi-person-badge-fill"></i>
          </div>
          <div>
            <h4>{{ teachers.length }}</h4>
            <span>Total Teachers</span>
          </div>
        </div>

        <button
          class="btn btn-light add-btn"
          data-bs-toggle="modal"
          data-bs-target="#addModal"
          @click="resetForm"
        >
          <i class="bi bi-person-plus-fill"></i> Add a Teacher
        </button>
      </div>
    </div>

    <!-- ================= MAIN CARD ================= -->
    <div class="staff-table-card">
      <!-- TOOLBAR -->
      <div class="staff-toolbar">
        <div class="search-box">
          <i class="bi bi-search"></i>
          <input v-model="search" type="text" placeholder="Search teacher..." />
        </div>

        <select v-model="selectedDepartment" class="class-filter">
          <option value="">All Departments</option>
          <option v-for="dept in uniqueDepartments" :key="dept" :value="dept">
            {{ dept }}
          </option>
        </select>
      </div>

      <!-- TABLE -->
      <div class="table-responsive">
        <table class="table staff-table align-middle">
          <thead>
            <tr>
              <th>#</th>
              <th>Photo</th>
              <th>Name & ID</th>
              <th>Designation</th>
              <th>Department</th>
              <th>Email</th>
              <th>Shifts</th>
              <th width="180">Action</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="(item, index) in paginatedTeachers" :key="item.id">
              <td>{{ (currentPage - 1) * perPage + index + 1 }}</td>
              <td>
                <img :src="item.image || defaultAvatar" class="staff-avatar" alt="Teacher Photo" />
              </td>
              <td>
                <div class="staff-name">
                  <strong>{{ item.full_name }}</strong>
                  <small>{{ item.teacher_id }}</small>
                </div>
              </td>
              <td>
                <span class="skill-badge">{{ item.designation }}</span>
              </td>
              <td>{{ item.department }}</td>
              <td>{{ item.email }}</td>
              <td>
                <div class="d-flex flex-wrap gap-1">
                  <span v-for="sh in item.shifts" :key="sh.id" class="badge bg-secondary">
                    {{ sh.name }}
                  </span>
                  <span v-if="!item.shifts || item.shifts.length === 0" class="text-muted small"
                    >No Shift</span
                  >
                </div>
              </td>
              <td>
                <div class="action-buttons">
                  <button
                    class="action-btn view"
                    @click="openView(item)"
                    data-bs-toggle="modal"
                    data-bs-target="#viewModal"
                  >
                    <i class="bi bi-eye"></i>
                  </button>

                  <button
                    class="action-btn edit"
                    @click="openEdit(item)"
                    data-bs-toggle="modal"
                    data-bs-target="#editModal"
                  >
                    <i class="bi bi-pencil"></i>
                  </button>

                  <button class="action-btn delete" @click="deleteTeacher(item.id)">
                    <i class="bi bi-trash"></i>
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- EMPTY STATE -->
      <div v-if="filteredTeachers.length === 0" class="empty-state">
        <i class="bi bi-person-x"></i>
        <h5>No Teacher Found</h5>
        <p>Try changing search or filter</p>
      </div>

      <!-- PAGINATION -->
      <div v-if="filteredTeachers.length > 0" class="pagination-box">
        <div>
          Showing <b>{{ (currentPage - 1) * perPage + 1 }}</b> to
          <b>{{ Math.min(currentPage * perPage, filteredTeachers.length) }}</b> of
          <b>{{ filteredTeachers.length }}</b>
        </div>

        <div class="page-buttons">
          <button class="page-btn" @click="prevPage" :disabled="currentPage === 1">
            <i class="bi bi-chevron-left"></i>
          </button>
          <span>{{ currentPage }} / {{ totalPages }}</span>
          <button class="page-btn" @click="nextPage" :disabled="currentPage === totalPages">
            <i class="bi bi-chevron-right"></i>
          </button>
        </div>
      </div>
    </div>

    <!-- ================= ADD MODAL ================= -->
    <div class="modal fade" id="addModal" ref="addModalRef">
      <div class="modal-dialog">
        <div class="modal-content teacher-modal">
          <div class="modal-header">
            <h5>Add Teacher</h5>
            <button class="btn-close" data-bs-dismiss="modal" id="closeAddModal"></button>
          </div>

          <div class="modal-body">
            <!-- IMAGE PREVIEW & INPUT -->
            <div class="text-center mb-3">
              <img
                :src="addPreview || defaultAvatar"
                class="image-preview mb-2"
                alt="Add Preview"
              />
              <input
                type="file"
                ref="addFileInput"
                class="form-control"
                @change="handleFileChange($event, 'add')"
                accept="image/*"
              />
            </div>

            <input v-model="form.full_name" class="form-control mb-3" placeholder="Full Name *" />
            <input
              v-model="form.designation"
              class="form-control mb-3"
              placeholder="Designation *"
            />

            <select v-model="form.department" class="form-control mb-3">
              <option value="" disabled selected>Select Department *</option>
              <option value="Computer Science">Computer Science</option>
              <option value="Electrical Engineering">Electrical Engineering</option>
              <option value="Physics">Physics</option>
              <option value="Mathematics">Mathematics</option>
              <option value="English">English</option>
            </select>

            <input
              v-model="form.qualification"
              class="form-control mb-3"
              placeholder="Qualification *"
            />
            <input v-model="form.phone" class="form-control mb-3" placeholder="Phone *" />
            <input v-model="form.email" class="form-control mb-3" placeholder="Email *" />
            <input v-model="form.joining_date" type="date" class="form-control mb-3" />
            <input
              v-model="form.salary"
              type="number"
              class="form-control mb-3"
              placeholder="Salary"
            />

            <!-- Shifts Checkbox List -->
            <label class="form-label fw-bold">Select Shifts *</label>
            <div class="border p-3 rounded mb-3 bg-light">
              <div v-for="shift in allShifts" :key="shift.id" class="form-check">
                <input
                  class="form-check-input"
                  type="checkbox"
                  :value="shift.id"
                  :id="'add_shift_' + shift.id"
                  v-model="form.shift_ids"
                />
                <label class="form-check-label" :for="'add_shift_' + shift.id">
                  {{ shift.name }}
                </label>
              </div>
              <div v-if="allShifts.length === 0" class="text-muted small">
                No shifts available. Please add shifts first.
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
            <button class="btn btn-primary" :disabled="loading" @click="teacher_create">
              {{ loading ? 'Saving...' : 'Save' }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= VIEW MODAL ================= -->
    <div class="modal fade" id="viewModal">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content teacher-modal">
          <div class="modal-header bg-primary text-white">
            <h5>Teacher Details</h5>
            <button class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
          </div>

          <div class="modal-body text-center">
            <img
              :src="selectedTeacher.image || defaultAvatar"
              class="teacher-modal-avatar mb-3"
              alt="Teacher Avatar"
            />
            <h4>{{ selectedTeacher.full_name }}</h4>
            <p class="text-muted">{{ selectedTeacher.teacher_id }}</p>
            <hr />

            <div class="text-start px-3">
              <p><strong>Designation:</strong> {{ selectedTeacher.designation }}</p>
              <p><strong>Department:</strong> {{ selectedTeacher.department }}</p>
              <p><strong>Qualification:</strong> {{ selectedTeacher.qualification }}</p>
              <p><strong>Email:</strong> {{ selectedTeacher.email }}</p>
              <p><strong>Phone:</strong> {{ selectedTeacher.phone }}</p>
              <p>
                <strong>Joining Date:</strong>
                {{ selectedTeacher.join_date || selectedTeacher.joining_date }}
              </p>
              <p><strong>Salary:</strong> {{ selectedTeacher.salary }}</p>
              <p>
                <strong>Shifts:</strong>
                <span
                  v-for="sh in selectedTeacher.shifts"
                  :key="sh.id"
                  class="badge bg-secondary me-1"
                >
                  {{ sh.name }}
                </span>
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= EDIT MODAL ================= -->
    <div class="modal fade" id="editModal">
      <div class="modal-dialog">
        <div class="modal-content teacher-modal">
          <div class="modal-header">
            <h5>Edit Teacher</h5>
            <button class="btn-close" data-bs-dismiss="modal" id="closeEditModal"></button>
          </div>

          <div class="modal-body">
            <!-- IMAGE PREVIEW & INPUT FOR EDIT -->
            <div class="text-center mb-3">
              <img
                :src="editPreview || selectedTeacher.image || defaultAvatar"
                class="image-preview mb-2"
                alt="Edit Preview"
              />
              <input
                type="file"
                ref="editFileInput"
                class="form-control"
                @change="handleFileChange($event, 'edit')"
                accept="image/*"
              />
            </div>

            <label class="form-label">Full Name</label>
            <input
              v-model="selectedTeacher.full_name"
              class="form-control mb-3"
              placeholder="Name"
            />

            <label class="form-label">Designation</label>
            <input
              v-model="selectedTeacher.designation"
              class="form-control mb-3"
              placeholder="Designation"
            />

            <label class="form-label">Department</label>
            <select v-model="selectedTeacher.department" class="form-control mb-3">
              <option value="Computer Science">Computer Science</option>
              <option value="Electrical Engineering">Electrical Engineering</option>
              <option value="Physics">Physics</option>
              <option value="Mathematics">Mathematics</option>
              <option value="English">English</option>
            </select>

            <label class="form-label">Qualification</label>
            <input
              v-model="selectedTeacher.qualification"
              class="form-control mb-3"
              placeholder="Qualification"
            />

            <label class="form-label">Phone</label>
            <input v-model="selectedTeacher.phone" class="form-control mb-3" placeholder="Phone" />

            <label class="form-label">Email</label>
            <input v-model="selectedTeacher.email" class="form-control mb-3" placeholder="Email" />

            <label class="form-label">Joining Date</label>
            <input v-model="selectedTeacher.joining_date" type="date" class="form-control mb-3" />

            <label class="form-label">Salary</label>
            <input v-model="selectedTeacher.salary" type="number" class="form-control mb-3" />

            <!-- Shifts Checkbox List for Edit -->
            <label class="form-label fw-bold">Select Shifts *</label>
            <div class="border p-3 rounded mb-3 bg-light">
              <div v-for="shift in allShifts" :key="shift.id" class="form-check">
                <input
                  class="form-check-input"
                  type="checkbox"
                  :value="shift.id"
                  :id="'edit_shift_' + shift.id"
                  v-model="selectedTeacher.shift_ids"
                />
                <label class="form-check-label" :for="'edit_shift_' + shift.id">
                  {{ shift.name }}
                </label>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
            <button class="btn btn-primary" :disabled="loading" @click="updateTeacher">
              {{ loading ? 'Updating...' : 'Update' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import api from '@/services/api'
import dashPageView from './dashPageView.vue'

// --- State Variables ---
const teachers = ref([])
const allShifts = ref([]) // 👈 সমস্ত শিফটের তালিকা রাখার জন্য
const loading = ref(false)
const search = ref('')
const selectedDepartment = ref('')
const currentPage = ref(1)
const perPage = ref(10)
const defaultAvatar = ref('https://via.placeholder.com/150')

// Add Form State
const form = ref({
  full_name: '',
  designation: '',
  department: '',
  qualification: '',
  phone: '',
  email: '',
  joining_date: new Date().toISOString().slice(0, 10), // 👈 admission_date থেকে joining_date করা হয়েছে
  salary: '',
  image: null,
  shift_ids: [], // 👈 একক শিফটের বদলে অ্যারে ব্যবহার করা হয়েছে
})
const addPreview = ref(null)

// Edit / View State
const selectedTeacher = ref({
  id: null,
  full_name: '',
  designation: '',
  department: '',
  qualification: '',
  phone: '',
  email: '',
  joining_date: '',
  salary: '',
  image: null,
  shift_ids: [], // 👈 শিফট আইডিগুলোর অ্যারে
})
const editImageFile = ref(null)
const editPreview = ref(null)

// Template Ref for File Input
const addFileInput = ref(null)

// --- Computed Properties ---
const filteredTeachers = computed(() => {
  return teachers.value.filter((teacher) => {
    const matchesSearch =
      teacher.full_name?.toLowerCase().includes(search.value.toLowerCase()) ||
      teacher.teacher_id?.toLowerCase().includes(search.value.toLowerCase()) ||
      teacher.email?.toLowerCase().includes(search.value.toLowerCase())

    const matchesDepartment =
      !selectedDepartment.value || teacher.department === selectedDepartment.value

    return matchesSearch && matchesDepartment
  })
})

const paginatedTeachers = computed(() => {
  const start = (currentPage.value - 1) * perPage.value
  return filteredTeachers.value.slice(start, start + perPage.value)
})

const totalPages = computed(() => {
  return Math.ceil(filteredTeachers.value.length / perPage.value) || 1
})

const uniqueDepartments = computed(() => {
  const depts = teachers.value.map((t) => t.department).filter(Boolean)
  return [...new Set(depts)]
})

// --- Clean Bootstrap Modals on Browser Back / Route Change ---
const cleanupModals = () => {
  // Remove modal-open from body
  document.body.classList.remove('modal-open')

  // Remove Bootstrap modal backdrops
  document.querySelectorAll('.modal-backdrop').forEach((backdrop) => {
    backdrop.remove()
  })

  // Reset body styles added by Bootstrap
  document.body.style.removeProperty('overflow')
  document.body.style.removeProperty('padding-right')

  // Hide any active modal
  document.querySelectorAll('.modal.show').forEach((modal) => {
    modal.classList.remove('show')
    modal.style.display = 'none'
    modal.removeAttribute('aria-modal')
    modal.setAttribute('aria-hidden', 'true')
  })
}
// --- Lifecycle Hook ---
onMounted(() => {
  fetchTeachers()
  fetchShifts()

  // Mobile/browser back button handle
  window.addEventListener('popstate', cleanupModals)
})

// --- Methods / Functions ---

// 1. Fetch Teachers List
const fetchTeachers = async () => {
  try {
    const response = await api.get('/teachers')
    if (response.data.status) {
      teachers.value = response.data.data
    }
  } catch (error) {
    console.error('Error fetching teachers:', error)
  }
}

// Fetch Shifts List (checkbox-এর জন্য শিফট ফেচ করা)
const fetchShifts = async () => {
  try {
    const response = await api.get('/shifts')
    if (response.data.status) {
      allShifts.value = response.data.data
    }
  } catch (error) {
    console.error('Error fetching shifts:', error)
  }
}

// 2. Image Selection & Preview
const handleFileChange = (event, type) => {
  const file = event.target.files[0]
  if (!file) return

  if (type === 'add') {
    form.value.image = file
    addPreview.value = URL.createObjectURL(file)
  } else if (type === 'edit') {
    editImageFile.value = file
    editPreview.value = URL.createObjectURL(file)
  }
}

// 3. Reset Form
const resetForm = () => {
  form.value = {
    full_name: '',
    designation: '',
    department: '',
    qualification: '',
    phone: '',
    email: '',
    joining_date: new Date().toISOString().slice(0, 10), // 👈 রিসেট করলেও আজকের ডেট থাকবে
    salary: '',
    image: null,
    shift_ids: [],
  }
  addPreview.value = null
  if (addFileInput.value) addFileInput.value.value = ''
}

// 4. Create Teacher
const teacher_create = async () => {
  loading.value = true
  try {
    const formData = new FormData()
    formData.append('full_name', form.value.full_name)
    formData.append('designation', form.value.designation)
    formData.append('department', form.value.department)
    formData.append('qualification', form.value.qualification)
    formData.append('phone', form.value.phone)
    formData.append('email', form.value.email)
    formData.append('joining_date', form.value.joining_date)
    formData.append('salary', form.value.salary || 0)

    // মাল্টিপল শিফট আইডি অ্যাপেন্ড করা
    form.value.shift_ids.forEach((id) => {
      formData.append('shift_ids[]', id)
    })

    if (form.value.image) {
      formData.append('image', form.value.image)
    }

    const response = await api.post('/teachers', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    })

    if (response.data.status) {
      alert(response.data.message)
      fetchTeachers()
      resetForm()
      document.getElementById('closeAddModal')?.click()
    }
  } catch (error) {
    console.log('Backend Error Details:', error.response?.data)
    if (error.response?.data?.errors) {
      const firstError = Object.values(error.response.data.errors)[0][0]
      alert(firstError)
    } else {
      alert(error.response?.data?.message || 'Error creating teacher')
    }
  } finally {
    loading.value = false
  }
}

// 5. Open View Modal
const openView = (teacher) => {
  selectedTeacher.value = { ...teacher }
}

// 6. Open Edit Modal
const openEdit = (teacher) => {
  selectedTeacher.value = {
    ...teacher,
    joining_date: teacher.join_date || teacher.joining_date,
    shift_ids: teacher.shifts ? teacher.shifts.map((s) => s.id) : [], // টিচারের শিফটগুলোকে আইডি অ্যারেতে কনভার্ট করা
  }
  editPreview.value = null
  editImageFile.value = null
}

// 7. Update Teacher
const updateTeacher = async () => {
  loading.value = true
  try {
    const formData = new FormData()
    formData.append('_method', 'PUT') // Laravel Method Spoofing
    formData.append('full_name', selectedTeacher.value.full_name)
    formData.append('designation', selectedTeacher.value.designation)
    formData.append('department', selectedTeacher.value.department)
    formData.append('qualification', selectedTeacher.value.qualification)
    formData.append('phone', selectedTeacher.value.phone)
    formData.append('email', selectedTeacher.value.email)
    formData.append('joining_date', selectedTeacher.value.joining_date)
    formData.append('salary', selectedTeacher.value.salary || 0)

    // আপডেট করার সময় শিফট আইডিগুলো অ্যারে আকারে পাঠানো
    if (selectedTeacher.value.shift_ids && selectedTeacher.value.shift_ids.length > 0) {
      selectedTeacher.value.shift_ids.forEach((id) => {
        formData.append('shift_ids[]', id)
      })
    }

    if (editImageFile.value) {
      formData.append('image', editImageFile.value)
    }

    const response = await api.post(`/teachers/${selectedTeacher.value.id}`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    })

    if (response.data.status) {
      alert(response.data.message)
      fetchTeachers()
      document.getElementById('closeEditModal')?.click()
    }
  } catch (error) {
    alert(error.response?.data?.message || 'Error updating teacher')
  } finally {
    loading.value = false
  }
}

// 8. Delete Teacher
const deleteTeacher = async (id) => {
  if (confirm('Are you sure you want to delete this teacher?')) {
    try {
      const response = await api.delete(`/teachers/${id}`)
      if (response.data.status) {
        alert(response.data.message)
        fetchTeachers()
      }
    } catch (error) {
      alert('Error deleting teacher')
    }
  }
}

// 9. Pagination Actions
const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++
}

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--
}
onBeforeUnmount(() => {
  cleanupModals()
  window.removeEventListener('popstate', cleanupModals)
})
</script>

<style scoped>
/* Image Preview Style */
.image-preview {
  width: 90px;
  height: 90px;
  object-fit: cover;
  border-radius: 50%;
  border: 3px solid #2563eb;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

/* Base Styles */
.content {
  margin-left: 250px;
  padding: 25px;
  background: #f8fafc;
  min-height: 100vh;
}

.staff-header {
  background: linear-gradient(135deg, #2563eb, #4f46e5);
  padding: 28px 32px;
  border-radius: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
  margin-bottom: 25px;
  box-shadow: 0 10px 30px rgba(37, 99, 235, 0.25);
}

.staff-header h2 {
  font-weight: 700;
  margin-bottom: 6px;
}

.staff-header p {
  margin: 0;
  color: #dbeafe;
}

.staff-header-right {
  display: flex;
  align-items: center;
  gap: 20px;
}

.staff-summary {
  background: white;
  color: #111827;
  padding: 12px 18px;
  border-radius: 15px;
  display: flex;
  align-items: center;
  gap: 15px;
}

.summary-icon {
  width: 45px;
  height: 45px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #dbeafe;
  color: #2563eb;
  font-size: 22px;
}

.staff-summary h4 {
  margin: 0;
  font-weight: 700;
}

.staff-summary span {
  font-size: 13px;
  color: #6b7280;
}

.add-btn {
  color: #2563eb;
  font-weight: 600;
  border-radius: 12px;
  padding: 12px 20px;
}

.staff-table-card {
  background: white;
  border-radius: 20px;
  padding: 25px;
  box-shadow: 0 10px 25px rgba(15, 23, 42, 0.08);
}

.staff-toolbar {
  display: flex;
  justify-content: space-between;
  gap: 15px;
  margin-bottom: 20px;
}

.search-box {
  position: relative;
  width: 350px;
}

.search-box i {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  color: #9ca3af;
}

.search-box input {
  width: 100%;
  padding: 12px 15px 12px 45px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  outline: none;
}

.search-box input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15);
}

.class-filter {
  width: 200px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  padding: 10px;
}

.staff-table {
  border-collapse: separate;
  border-spacing: 0 10px;
}

.staff-table thead th {
  background: #f1f5f9;
  border: none;
  padding: 15px;
  color: #374151;
  font-size: 14px;
}

.staff-table tbody tr {
  background: white;
  box-shadow: 0 3px 12px rgba(0, 0, 0, 0.05);
  transition: 0.3s;
}

.staff-table tbody tr:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
}

.staff-table td {
  padding: 15px;
  border: none;
}

.staff-avatar {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid #dbeafe;
}

.staff-name {
  display: flex;
  flex-direction: column;
}

.staff-name small {
  color: #6b7280;
  font-size: 12px;
}

.skill-badge {
  background: #eff6ff;
  color: #2563eb;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 13px;
  font-weight: 600;
}

.action-buttons {
  display: flex;
  gap: 8px;
}

.action-btn {
  width: 38px;
  height: 38px;
  border: none;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  transition: 0.3s;
}

.action-btn:hover {
  transform: translateY(-3px);
}

.action-btn.view {
  background: #dbeafe;
  color: #2563eb;
}

.action-btn.edit {
  background: #fef3c7;
  color: #d97706;
}

.action-btn.delete {
  background: #fee2e2;
  color: #dc2626;
}

.empty-state {
  text-align: center;
  padding: 50px;
  color: #6b7280;
}

.empty-state i {
  font-size: 45px;
  color: #9ca3af;
}

.pagination-box {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
  color: #6b7280;
}

.page-buttons {
  display: flex;
  align-items: center;
  gap: 10px;
}

.page-btn {
  border: none;
  background: #2563eb;
  color: white;
  width: 38px;
  height: 38px;
  border-radius: 10px;
}

.page-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.teacher-modal {
  border: none;
  border-radius: 20px;
  overflow: hidden;
}

.modal-header {
  padding: 20px 25px;
}

.modal-body {
  padding: 25px;
}

.modal-footer {
  padding: 18px 25px;
}

.modal .form-control {
  border-radius: 12px;
  padding: 12px;
}

.teacher-modal-avatar {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  border: 5px solid #dbeafe;
  object-fit: cover;
}

@media (max-width: 768px) {
  .content {
    margin-left: 0;
    padding: 15px;
  }

  .staff-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 20px;
  }

  .staff-header-right {
    width: 100%;
    justify-content: space-between;
  }

  .staff-toolbar {
    flex-direction: column;
  }

  .search-box,
  .class-filter {
    width: 100%;
  }

  .staff-table {
    min-width: 900px;
  }

  .staff-table-card {
    overflow: hidden;
  }

  .pagination-box {
    flex-direction: column;
    gap: 15px;
  }
}
</style>
