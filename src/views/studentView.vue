<template>
  <RouterView />

  <dashPageView />

  <div class="content">
    <!-- ================= HEADER ================= -->
    <div class="staff-header">
      <div>
        <h2>Student Management</h2>
        <p>Manage students, classes and academic information</p>
      </div>

      <div class="staff-header-right">
        <div class="staff-summary">
          <div class="summary-icon">
            <i class="bi bi-mortarboard-fill"></i>
          </div>

          <div>
            <h4>{{ totalStudents }}</h4>
            <span>Total Students</span>
          </div>
        </div>

        <button
          class="btn btn-light add-btn"
          data-bs-toggle="modal"
          data-bs-target="#addModal"
          @click="resetForm"
        >
          <i class="bi bi-person-plus-fill"></i>
          Add Student
        </button>
      </div>
    </div>

    <!-- ================= MAIN CARD ================= -->
    <div class="staff-table-card">
      <!-- TOOLBAR -->
      <div class="staff-toolbar">
        <div class="search-box">
          <i class="bi bi-search"></i>

          <input v-model="search" type="text" placeholder="Search student..." />
        </div>

        <!-- CLASS FILTER -->
        <select v-model="selectedClass" class="class-filter">
          <option value="">All Classes</option>

          <option v-for="cls in classes" :key="cls.id" :value="String(cls.id)">
            {{ cls.class_name }}
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
              <th>Name</th>
              <th>Class</th>
              <th>Group</th>
              <th>Email</th>
              <th width="180">Action</th>
            </tr>
          </thead>

          <tbody>
            <!-- LOADING -->
            <tr v-if="loading">
              <td colspan="7" class="text-center py-5">
                <div class="spinner-border text-primary" role="status"></div>

                <div class="mt-2 text-muted">Loading students...</div>
              </td>
            </tr>

            <!-- STUDENTS -->
            <tr v-for="(item, index) in students" :key="item.id" v-else>
              <td>
                {{ (currentPage - 1) * perPage + index + 1 }}
              </td>

              <td>
                <img :src="getImageUrl(item)" class="staff-avatar" alt="Student Photo" />
              </td>

              <td>
                <div class="staff-name">
                  <strong>{{ item.full_name }}</strong>
                  <small>{{ item.student_id }}</small>
                </div>
              </td>

              <td>
                <span class="skill-badge">
                  {{ item.class_info ? item.class_info.class_name : 'N/A' }}
                </span>
              </td>

              <td>
                <span v-if="item.class_group" class="group-badge">
                  {{ item.class_group.group_name }}
                </span>

                <span v-else class="text-muted"> N/A </span>
              </td>

              <td>
                {{ item.email || 'N/A' }}
              </td>

              <td>
                <div class="action-buttons">
                  <button
                    class="action-btn view"
                    @click="openView(item)"
                    data-bs-toggle="modal"
                    data-bs-target="#viewModal"
                    title="View"
                  >
                    <i class="bi bi-eye"></i>
                  </button>

                  <button
                    class="action-btn edit"
                    @click="openEdit(item)"
                    data-bs-toggle="modal"
                    data-bs-target="#editModal"
                    title="Edit"
                  >
                    <i class="bi bi-pencil"></i>
                  </button>

                  <button class="action-btn delete" @click="deleteStudent(item.id)" title="Delete">
                    <i class="bi bi-trash"></i>
                  </button>
                </div>
              </td>
            </tr>

            <!-- EMPTY STATE -->
            <tr v-if="!loading && students.length === 0">
              <td colspan="7" class="text-center py-5">
                <div class="empty-state">
                  <i class="bi bi-person-x"></i>

                  <h5>No Student Found</h5>

                  <p>Try changing search or filter</p>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- PAGINATION -->
      <div v-if="filteredTotal > 0" class="pagination-box">
        <div>
          Showing
          <b>{{ showingFrom }}</b>
          to
          <b>{{ showingTo }}</b>
          of
          <b>{{ filteredTotal }}</b>
        </div>

        <div class="page-buttons">
          <button class="page-btn" @click="prevPage" :disabled="currentPage === 1 || loading">
            <i class="bi bi-chevron-left"></i>
          </button>

          <span> {{ currentPage }} / {{ totalPages }} </span>

          <button
            class="page-btn"
            @click="nextPage"
            :disabled="currentPage === totalPages || loading"
          >
            <i class="bi bi-chevron-right"></i>
          </button>
        </div>
      </div>
    </div>

    <!-- ================= ADD STUDENT MODAL ================= -->
    <div class="modal fade" id="addModal" ref="addModalRef">
      <div class="modal-dialog modal-lg">
        <div class="modal-content student-modal">
          <div class="modal-header">
            <h5>
              <i class="bi bi-person-plus-fill me-2"></i>
              Add Student
            </h5>

            <button class="btn-close" data-bs-dismiss="modal"></button>
          </div>

          <div class="modal-body">
            <div class="text-center mb-4">
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

            <div class="mb-3">
              <label class="form-label">
                Full Name
                <span class="text-danger">*</span>
              </label>

              <input
                v-model="form.full_name"
                type="text"
                class="form-control"
                placeholder="Enter full name"
              />
            </div>

            <div class="mb-3">
              <label class="form-label">
                Father's Name
                <span class="text-danger">*</span>
              </label>

              <input
                v-model="form.fathers_name"
                type="text"
                class="form-control"
                placeholder="Enter father's name"
              />
            </div>

            <div class="mb-3">
              <label class="form-label">
                Mother's Name
                <span class="text-danger">*</span>
              </label>

              <input
                v-model="form.mothers_name"
                type="text"
                class="form-control"
                placeholder="Enter mother's name"
              />
            </div>

            <div class="mb-3">
              <label class="form-label">
                Phone
                <span class="text-danger">*</span>
              </label>

              <input
                v-model="form.phone"
                type="text"
                class="form-control"
                placeholder="Enter phone number"
              />
            </div>

            <div class="mb-3">
              <label class="form-label"> Email </label>

              <input
                v-model="form.email"
                type="email"
                class="form-control"
                placeholder="Enter email address"
              />
            </div>

            <div class="mb-3">
              <label class="form-label">
                Class Group
                <span class="text-danger">*</span>
              </label>

              <select v-model="form.class_group_id" class="form-select">
                <option value="" disabled>Select Group</option>

                <option v-for="group in classGroups" :key="group.id" :value="group.id">
                  {{ group.group_name }}
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label">
                Class
                <span class="text-danger">*</span>
              </label>

              <select v-model="form.class_id" class="form-select">
                <option value="" disabled>Select Class</option>

                <option v-for="cls in classes" :key="cls.id" :value="cls.id">
                  {{ cls.class_name }}
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label"> Section </label>

              <select v-model="form.section_id" class="form-select">
                <option value="">Select Section</option>

                <option v-for="sec in sections" :key="sec.id" :value="sec.id">
                  {{ sec.section_name }}
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label"> Assign Shift </label>

              <select v-model="form.shift_id" class="form-select">
                <option value="" disabled>Select Shift</option>

                <option v-for="shift in shifts" :key="shift.id" :value="shift.id">
                  {{ shift.name }}
                  ({{ shift.start_time }} - {{ shift.end_time }})
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label"> Monthly Fee </label>

              <input
                v-model="form.monthly_fee"
                type="number"
                min="0"
                class="form-control"
                placeholder="Enter monthly fee"
              />
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn btn-secondary" data-bs-dismiss="modal">Close</button>

            <button class="btn btn-primary" :disabled="loading" @click="student_create">
              <span v-if="loading"> Saving... </span>

              <span v-else>
                <i class="bi bi-person-plus-fill me-1"></i>
                Save Student
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= VIEW STUDENT MODAL ================= -->
    <div class="modal fade" id="viewModal" ref="viewModalRef">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content student-modal">
          <div class="modal-header bg-primary text-white">
            <h5>
              <i class="bi bi-person-vcard-fill me-2"></i>
              Student Details
            </h5>

            <button class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
          </div>

          <div class="modal-body text-center">
            <img
              :src="getImageUrl(selectedStudent)"
              class="student-modal-avatar mb-3"
              alt="Student Avatar"
            />

            <h4>
              {{ selectedStudent.full_name }}
            </h4>

            <p class="text-muted">
              {{ selectedStudent.student_id }}
            </p>

            <hr />

            <div class="text-start px-3">
              <p>
                <strong>Father's Name:</strong>
                {{ selectedStudent.fathers_name || 'N/A' }}
              </p>

              <p>
                <strong>Mother's Name:</strong>
                {{ selectedStudent.mothers_name || 'N/A' }}
              </p>

              <p>
                <strong>Email:</strong>
                {{ selectedStudent.email || 'N/A' }}
              </p>

              <p>
                <strong>Phone:</strong>
                {{ selectedStudent.phone || 'N/A' }}
              </p>

              <p>
                <strong>Class:</strong>
                {{ selectedStudent.class_info?.class_name || 'N/A' }}
              </p>

              <p>
                <strong>Group:</strong>
                {{ selectedStudent.class_group?.group_name || 'N/A' }}
              </p>

              <p>
                <strong>Section:</strong>
                {{ selectedStudent.section?.section_name || 'N/A' }}
              </p>

              <p>
                <strong>Shift:</strong>
                {{ selectedStudent.shift?.name || 'N/A' }}
              </p>

              <p>
                <strong>Monthly Fee:</strong>
                {{ selectedStudent.monthly_fee ? '৳ ' + selectedStudent.monthly_fee : 'N/A' }}
              </p>

              <p>
                <strong>Admission Date:</strong>
                {{ selectedStudent.admission_date || 'N/A' }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= EDIT STUDENT MODAL ================= -->
    <div class="modal fade" id="editModal" ref="editModalRef">
      <div class="modal-dialog modal-lg">
        <div class="modal-content student-modal">
          <div class="modal-header">
            <h5>
              <i class="bi bi-pencil-square me-2"></i>
              Edit Student
            </h5>

            <button class="btn-close" data-bs-dismiss="modal"></button>
          </div>

          <div class="modal-body">
            <div class="text-center mb-4">
              <img
                :src="editPreview || getImageUrl(selectedStudent)"
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

            <div class="mb-3">
              <label class="form-label"> Full Name </label>

              <input v-model="selectedStudent.full_name" class="form-control" placeholder="Name" />
            </div>

            <div class="mb-3">
              <label class="form-label"> Phone </label>

              <input v-model="selectedStudent.phone" class="form-control" placeholder="Phone" />
            </div>

            <div class="mb-3">
              <label class="form-label"> Email </label>

              <input v-model="selectedStudent.email" class="form-control" placeholder="Email" />
            </div>

            <div class="mb-3">
              <label class="form-label"> Class Group </label>

              <select v-model="selectedStudent.class_group_id" class="form-select">
                <option value="" disabled>Select Group</option>

                <option v-for="group in classGroups" :key="group.id" :value="group.id">
                  {{ group.group_name }}
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label"> Class </label>

              <select v-model="selectedStudent.class_id" class="form-select">
                <option value="" disabled>Select Class</option>

                <option v-for="cls in classes" :key="cls.id" :value="cls.id">
                  {{ cls.class_name }}
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label"> Section </label>

              <select v-model="selectedStudent.section_id" class="form-select">
                <option value="">Select Section</option>

                <option v-for="sec in sections" :key="sec.id" :value="sec.id">
                  {{ sec.section_name }}
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label"> Shift </label>

              <select v-model="selectedStudent.shift_id" class="form-select">
                <option value="">Select Shift</option>

                <option v-for="shift in shifts" :key="shift.id" :value="shift.id">
                  {{ shift.name }}
                </option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label"> Monthly Fee </label>

              <input
                v-model="selectedStudent.monthly_fee"
                type="number"
                min="0"
                class="form-control"
              />
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn btn-secondary" data-bs-dismiss="modal">Close</button>

            <button class="btn btn-primary" :disabled="loading" @click="updateStudent">
              {{ loading ? 'Updating...' : 'Update' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import dashPageView from './dashPageView.vue'
import { ref, reactive, computed, watch, onMounted, nextTick } from 'vue'

import api from '@/services/api'
import * as bootstrap from 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import { getImageUrl } from '@/utils/img'

const defaultAvatar = 'https://i.pravatar.cc/150'

const students = ref([])
const sections = ref([])
const classes = ref([])
const classGroups = ref([])
const shifts = ref([])

const search = ref('')
const selectedClass = ref('')
const loading = ref(false)

const currentPage = ref(1)
const perPage = ref(10)

const totalStudents = ref(0)
const filteredTotal = ref(0)
const totalPages = ref(1)

const addImageFile = ref(null)
const editImageFile = ref(null)

const addPreview = ref(null)
const editPreview = ref(null)

const addFileInput = ref(null)
const editFileInput = ref(null)

const addModalRef = ref(null)
const editModalRef = ref(null)

const form = reactive({
  full_name: '',
  fathers_name: '',
  mothers_name: '',
  phone: '',
  email: '',
  course_name: '',
  class_group_id: '',
  class_id: '',
  section_id: '',
  shift_id: '',
  admission_date: '',
  monthly_fee: '',
})

const selectedStudent = ref({
  id: null,
  full_name: '',
  fathers_name: '',
  mothers_name: '',
  student_id: '',
  phone: '',
  email: '',
  course_name: '',
  class_group_id: '',
  class_id: '',
  section_id: '',
  shift_id: '',
  admission_date: '',
  monthly_fee: '',
  image: null,
})

/* ================= PAGINATION INFO ================= */

const showingFrom = computed(() => {
  if (filteredTotal.value === 0) return 0

  return (currentPage.value - 1) * perPage.value + 1
})

const showingTo = computed(() => {
  return Math.min(currentPage.value * perPage.value, filteredTotal.value)
})

/* ================= IMAGE ================= */

const handleFileChange = (event, type) => {
  const file = event.target.files[0]

  if (!file) return

  if (type === 'add') {
    if (addPreview.value) {
      URL.revokeObjectURL(addPreview.value)
    }

    addImageFile.value = file
    addPreview.value = URL.createObjectURL(file)
  } else {
    if (editPreview.value) {
      URL.revokeObjectURL(editPreview.value)
    }

    editImageFile.value = file
    editPreview.value = URL.createObjectURL(file)
  }
}

/* ================= RESET ADD FORM ================= */

const resetForm = () => {
  Object.keys(form).forEach((key) => (form[key] = ''))

  addImageFile.value = null

  if (addPreview.value) {
    URL.revokeObjectURL(addPreview.value)
    addPreview.value = null
  }

  if (addFileInput.value) {
    addFileInput.value.value = ''
  }
}

/* ================= CLOSE MODAL ================= */
/* FIX: Bootstrap backdrop/blur cleanup */

const closeModal = async (modalId) => {
  const modalEl = document.getElementById(modalId)

  if (!modalEl) return

  const modalInstance =
    bootstrap.Modal.getInstance(modalEl) || bootstrap.Modal.getOrCreateInstance(modalEl)

  modalInstance.hide()

  /*
   * Wait for Bootstrap modal transition to finish.
   * Then force cleanup of any leftover backdrop/body state.
   */
  await new Promise((resolve) => {
    setTimeout(resolve, 400)
  })

  document.querySelectorAll('.modal-backdrop').forEach((backdrop) => {
    backdrop.remove()
  })

  document.body.classList.remove('modal-open')

  document.body.style.removeProperty('overflow')

  document.body.style.removeProperty('padding-right')

  modalEl.classList.remove('show')

  modalEl.style.removeProperty('display')

  modalEl.removeAttribute('aria-modal')
  modalEl.removeAttribute('role')

  await nextTick()
}

/* ================= GET STUDENTS ================= */

const getStudent = async (page = 1) => {
  try {
    const params = {
      page,
      per_page: perPage.value,
    }

    if (search.value.trim()) {
      params.search = search.value.trim()
    }

    if (selectedClass.value) {
      params.class_id = selectedClass.value
    }

    const res = await api.get('/students', {
      params,
    })

    const response = res.data

    students.value = response.students || response.data || []

    const pagination = response.pagination || {}

    currentPage.value = Number(pagination.current_page || response.current_page || page)

    totalPages.value = Number(pagination.last_page || response.last_page || 1)

    filteredTotal.value = Number(pagination.total || response.filtered_total || response.total || 0)

    totalStudents.value = Number(response.total_students || filteredTotal.value)

    if (students.value.length === 0 && currentPage.value > 1 && filteredTotal.value > 0) {
      currentPage.value = Math.max(1, currentPage.value - 1)

      await getStudent(currentPage.value)
    }
  } catch (error) {
    console.error('Error fetching students:', error.response?.data || error)

    students.value = []
  }
}

/* ================= CREATE STUDENT ================= */

const student_create = async () => {
  try {
    loading.value = true

    const formData = new FormData()

    Object.keys(form).forEach((key) => {
      if (form[key] !== '') {
        formData.append(key, form[key])
      }
    })

    if (addImageFile.value) {
      formData.append('image', addImageFile.value)
    }

    const res = await api.post('/students', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    })

    alert(res.data.message || 'Student Created Successfully')

    /*
     * IMPORTANT:
     * Close and fully cleanup Bootstrap modal
     * before refreshing the student list.
     */
    await closeModal('addModal')

    resetForm()

    currentPage.value = 1

    await getStudent(1)
  } catch (error) {
    alert(error.response?.data?.message || 'Failed to create student')
  } finally {
    loading.value = false

    /*
     * Extra safety cleanup.
     * This prevents page blur if Bootstrap leaves
     * a backdrop behind after modal close.
     */
    document.querySelectorAll('.modal-backdrop').forEach((backdrop) => {
      backdrop.remove()
    })

    document.body.classList.remove('modal-open')

    document.body.style.removeProperty('overflow')

    document.body.style.removeProperty('padding-right')
  }
}

/* ================= OPEN VIEW ================= */

const openView = (student) => {
  selectedStudent.value = {
    ...student,
  }
}

/* ================= OPEN EDIT ================= */

const openEdit = (student) => {
  selectedStudent.value = {
    ...student,
  }

  editImageFile.value = null
  editPreview.value = null

  if (editFileInput.value) {
    editFileInput.value.value = ''
  }
}

/* ================= UPDATE STUDENT ================= */

const updateStudent = async () => {
  try {
    loading.value = true

    const formData = new FormData()

    Object.keys(selectedStudent.value).forEach((key) => {
      if (selectedStudent.value[key] !== null && selectedStudent.value[key] !== undefined) {
        if (!['class_info', 'class_group', 'section', 'shift'].includes(key)) {
          formData.append(key, selectedStudent.value[key])
        }
      }
    })

    if (editImageFile.value) {
      formData.append('image', editImageFile.value)
    }

    formData.append('_method', 'PUT')

    const res = await api.post(`/students/${selectedStudent.value.id}`, formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    })

    alert(res.data.message || 'Student Updated Successfully')

    closeModal('editModal')

    await getStudent(currentPage.value)
  } catch (error) {
    alert(error.response?.data?.message || 'Failed to update student')
  } finally {
    loading.value = false
  }
}

/* ================= DELETE STUDENT ================= */

const deleteStudent = async (id) => {
  if (!confirm('Are you sure you want to delete this student?')) {
    return
  }

  try {
    loading.value = true

    await api.delete(`/students/${id}`)

    await getStudent(currentPage.value)
  } catch (error) {
    alert('Failed to delete student')
  } finally {
    loading.value = false
  }
}

/* ================= PAGINATION ================= */

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++

    getStudent(currentPage.value)
  }
}

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--

    getStudent(currentPage.value)
  }
}

/* ================= SEARCH ================= */

let searchTimer = null

watch(search, () => {
  clearTimeout(searchTimer)

  searchTimer = setTimeout(() => {
    currentPage.value = 1

    getStudent(1)
  }, 400)
})

watch(selectedClass, () => {
  currentPage.value = 1

  getStudent(1)
})

/* ================= ON MOUNTED ================= */

onMounted(async () => {
  loading.value = true

  try {
    const cachedSections = sessionStorage.getItem('cache_sections')

    const cachedClasses = sessionStorage.getItem('cache_classes')

    const cachedGroups = sessionStorage.getItem('cache_groups')

    const cachedShifts = sessionStorage.getItem('cache_shifts')

    if (cachedSections && cachedClasses && cachedGroups && cachedShifts) {
      sections.value = JSON.parse(cachedSections)

      classes.value = JSON.parse(cachedClasses)

      classGroups.value = JSON.parse(cachedGroups)

      shifts.value = JSON.parse(cachedShifts)

      await getStudent(1)
    } else {
      const [secRes, clsRes, grpRes, shfRes] = await Promise.all([
        api.get('/sections'),
        api.get('/classes'),
        api.get('/class_group'),
        api.get('/shifts'),
        getStudent(1),
      ])

      sections.value = secRes.data.sections || secRes.data.data || secRes.data || []

      classes.value = clsRes.data.classes || clsRes.data.data || clsRes.data || []

      classGroups.value =
        grpRes.data.classGroups || grpRes.data.groups || grpRes.data.data || grpRes.data || []

      shifts.value = shfRes.data.data || shfRes.data.shifts || shfRes.data || []

      sessionStorage.setItem('cache_sections', JSON.stringify(sections.value))

      sessionStorage.setItem('cache_classes', JSON.stringify(classes.value))

      sessionStorage.setItem('cache_groups', JSON.stringify(classGroups.value))

      sessionStorage.setItem('cache_shifts', JSON.stringify(shifts.value))
    }
  } catch (error) {
    console.error('Initialization error:', error)
  } finally {
    loading.value = false
  }
})
</script>

<style scoped>
/* =====================================================
   IMAGE
===================================================== */

.image-preview {
  width: 90px;
  height: 90px;
  object-fit: cover;
  border-radius: 50%;
  border: 3px solid #2563eb;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

/* =====================================================
   BASE
===================================================== */

.content {
  margin-left: 250px;
  padding: 25px;
  background: #f8fafc;
  min-height: 100vh;
}

/* =====================================================
   HEADER
===================================================== */

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

/* =====================================================
   SUMMARY
===================================================== */

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

/* =====================================================
   CARD
===================================================== */

.staff-table-card {
  background: white;

  border-radius: 20px;

  padding: 25px;

  box-shadow: 0 10px 25px rgba(15, 23, 42, 0.08);
}

/* =====================================================
   TOOLBAR
===================================================== */

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

/* =====================================================
   TABLE
===================================================== */

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

/* =====================================================
   AVATAR
===================================================== */

.staff-avatar {
  width: 45px;
  height: 45px;

  border-radius: 50%;

  object-fit: cover;

  border: 3px solid #dbeafe;
}

/* =====================================================
   NAME
===================================================== */

.staff-name {
  display: flex;
  flex-direction: column;
}

.staff-name small {
  color: #6b7280;
  font-size: 12px;
}

/* =====================================================
   CLASS BADGE
===================================================== */

.skill-badge {
  background: #eff6ff;

  color: #2563eb;

  padding: 6px 12px;

  border-radius: 20px;

  font-size: 13px;

  font-weight: 600;
}

/* =====================================================
   GROUP BADGE
===================================================== */

.group-badge {
  background: #ecfdf5;

  color: #059669;

  padding: 6px 12px;

  border-radius: 20px;

  font-size: 13px;

  font-weight: 600;
}

/* =====================================================
   ACTION
===================================================== */

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

/* =====================================================
   EMPTY
===================================================== */

.empty-state {
  text-align: center;

  padding: 50px;

  color: #6b7280;
}

.empty-state i {
  font-size: 45px;

  color: #9ca3af;
}

/* =====================================================
   PAGINATION
===================================================== */

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

/* =====================================================
   MODAL
===================================================== */

.student-modal {
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

.modal .form-control,
.modal .form-select {
  border-radius: 12px;

  padding: 12px;
}

.student-modal-avatar {
  width: 100px;
  height: 100px;

  border-radius: 50%;

  border: 5px solid #dbeafe;

  object-fit: cover;
}

/* =====================================================
   MOBILE
===================================================== */

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
    min-width: 1100px;
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
