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
            <h4>{{ students.length }}</h4>
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

              <!-- <th>Course</th> -->

              <th width="180">Action</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="(item, index) in paginatedStudents" :key="item.id">
              <!-- SERIAL -->

              <td>
                {{ (currentPage - 1) * perPage + index + 1 }}
              </td>

              <!-- PHOTO -->

              <td>
                <img :src="getImageUrl(item)" class="staff-avatar" alt="Student Photo" />
              </td>

              <!-- NAME -->

              <td>
                <div class="staff-name">
                  <strong>
                    {{ item.full_name }}
                  </strong>

                  <small>
                    {{ item.student_id }}
                  </small>
                </div>
              </td>

              <!-- CLASS -->

              <td>
                <span class="skill-badge">
                  {{ item.class_info ? item.class_info.class_name : 'N/A' }}
                </span>
              </td>

              <!-- GROUP -->

              <td>
                <span v-if="item.class_group" class="group-badge">
                  {{ item.class_group.group_name }}
                </span>

                <span v-else class="text-muted"> N/A </span>
              </td>

              <!-- EMAIL -->

              <td>
                {{ item.email || 'N/A' }}
              </td>

              <!-- COURSE -->

              <!--
              <td>
                {{ item.course_name || 'N/A' }}
              </td>
              -->

              <!-- ACTION -->

              <td>
                <div class="action-buttons">
                  <!-- VIEW -->

                  <button
                    class="action-btn view"
                    @click="openView(item)"
                    data-bs-toggle="modal"
                    data-bs-target="#viewModal"
                    title="View"
                  >
                    <i class="bi bi-eye"></i>
                  </button>

                  <!-- EDIT -->

                  <button
                    class="action-btn edit"
                    @click="openEdit(item)"
                    data-bs-toggle="modal"
                    data-bs-target="#editModal"
                    title="Edit"
                  >
                    <i class="bi bi-pencil"></i>
                  </button>

                  <!-- DELETE -->

                  <button class="action-btn delete" @click="deleteStudent(item.id)" title="Delete">
                    <i class="bi bi-trash"></i>
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- EMPTY STATE -->

      <div v-if="filteredStudents.length === 0" class="empty-state">
        <i class="bi bi-person-x"></i>

        <h5>No Student Found</h5>

        <p>Try changing search or filter</p>
      </div>

      <!-- PAGINATION -->

      <div v-if="filteredStudents.length > 0" class="pagination-box">
        <div>
          Showing

          <b>
            {{ (currentPage - 1) * perPage + 1 }}
          </b>

          to

          <b>
            {{ Math.min(currentPage * perPage, filteredStudents.length) }}
          </b>

          of

          <b>
            {{ filteredStudents.length }}
          </b>
        </div>

        <div class="page-buttons">
          <button class="page-btn" @click="prevPage" :disabled="currentPage === 1">
            <i class="bi bi-chevron-left"></i>
          </button>

          <span> {{ currentPage }} / {{ totalPages }} </span>

          <button class="page-btn" @click="nextPage" :disabled="currentPage === totalPages">
            <i class="bi bi-chevron-right"></i>
          </button>
        </div>
      </div>
    </div>

    <!-- ===================================================== -->
    <!-- ================= ADD STUDENT MODAL ================= -->
    <!-- ===================================================== -->

    <div class="modal fade" id="addModal" ref="addModalRef">
      <div class="modal-dialog modal-lg">
        <div class="modal-content student-modal">
          <!-- HEADER -->

          <div class="modal-header">
            <h5>
              <i class="bi bi-person-plus-fill me-2"></i>
              Add Student
            </h5>

            <button class="btn-close" data-bs-dismiss="modal"></button>
          </div>

          <!-- BODY -->

          <div class="modal-body">
            <!-- IMAGE -->

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

            <!-- FULL NAME -->

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

            <!-- FATHER -->

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

            <!-- MOTHER -->

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

            <!-- PHONE -->

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

            <!-- EMAIL -->

            <div class="mb-3">
              <label class="form-label"> Email </label>

              <input
                v-model="form.email"
                type="email"
                class="form-control"
                placeholder="Enter email address"
              />
            </div>

            <!-- COURSE -->

            <!--
            <div class="mb-3">

              <label class="form-label">
                Course
              </label>

              <input
                v-model="form.course_name"
                type="text"
                class="form-control"
                placeholder="Enter course name"
              />

            </div>
            -->

            <!-- ================= CLASS GROUP ================= -->

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

            <!-- CLASS -->

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

            <!-- SECTION -->

            <div class="mb-3">
              <label class="form-label"> Section </label>

              <select v-model="form.section_id" class="form-select">
                <option value="">Select Section</option>

                <option v-for="sec in sections" :key="sec.id" :value="sec.id">
                  {{ sec.section_name }}
                </option>
              </select>
            </div>

            <!-- SHIFT -->

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

            <!-- ADMISSION DATE -->

            <div class="mb-3">
              <!--
              <label class="form-label">
                Admission Date
              </label>
              -->

              <input hidden v-model="form.admission_date" type="date" class="form-control" />
            </div>

            <!-- MONTHLY FEE -->

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

          <!-- FOOTER -->

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

    <!-- ===================================================== -->
    <!-- ================= VIEW STUDENT MODAL ================= -->
    <!-- ===================================================== -->

    <div class="modal fade" id="viewModal">
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
                {{ selectedStudent.class_info ? selectedStudent.class_info.class_name : 'N/A' }}
              </p>

              <!-- GROUP -->

              <p>
                <strong>Group:</strong>
                {{ selectedStudent.class_group ? selectedStudent.class_group.group_name : 'N/A' }}
              </p>

              <p>
                <strong>Section:</strong>
                {{ selectedStudent.section ? selectedStudent.section.section_name : 'N/A' }}
              </p>

              <p>
                <strong>Shift:</strong>
                {{ selectedStudent.shift ? selectedStudent.shift.name : 'N/A' }}
              </p>

              <p>
                <strong>Course:</strong>
                {{ selectedStudent.course_name || 'N/A' }}
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

    <!-- ===================================================== -->
    <!-- ================= EDIT STUDENT MODAL ================= -->
    <!-- ===================================================== -->

    <div class="modal fade" id="editModal">
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
            <!-- IMAGE -->

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

            <!-- NAME -->

            <div class="mb-3">
              <label class="form-label"> Full Name </label>

              <input v-model="selectedStudent.full_name" class="form-control" placeholder="Name" />
            </div>

            <!-- PHONE -->

            <div class="mb-3">
              <label class="form-label"> Phone </label>

              <input v-model="selectedStudent.phone" class="form-control" placeholder="Phone" />
            </div>

            <!-- EMAIL -->

            <div class="mb-3">
              <label class="form-label"> Email </label>

              <input v-model="selectedStudent.email" class="form-control" placeholder="Email" />
            </div>

            <!-- COURSE -->

            <div class="mb-3">
              <label class="form-label"> Course </label>

              <input
                v-model="selectedStudent.course_name"
                class="form-control"
                placeholder="Course Name"
              />
            </div>

            <!-- ================= CLASS GROUP ================= -->

            <div class="mb-3">
              <label class="form-label"> Class Group </label>

              <select v-model="selectedStudent.class_group_id" class="form-select">
                <option value="" disabled>Select Group</option>

                <option v-for="group in classGroups" :key="group.id" :value="group.id">
                  {{ group.group_name }}
                </option>
              </select>
            </div>

            <!-- CLASS -->

            <div class="mb-3">
              <label class="form-label"> Class </label>

              <select v-model="selectedStudent.class_id" class="form-select">
                <option value="" disabled>Select Class</option>

                <option v-for="cls in classes" :key="cls.id" :value="cls.id">
                  {{ cls.class_name }}
                </option>
              </select>
            </div>

            <!-- SECTION -->

            <div class="mb-3">
              <label class="form-label"> Section </label>

              <select v-model="selectedStudent.section_id" class="form-select">
                <option value="">Select Section</option>

                <option v-for="sec in sections" :key="sec.id" :value="sec.id">
                  {{ sec.section_name }}
                </option>
              </select>
            </div>

            <!-- SHIFT -->

            <div class="mb-3">
              <label class="form-label"> Shift </label>

              <select v-model="selectedStudent.shift_id" class="form-select">
                <option value="">Select Shift</option>

                <option v-for="shift in shifts" :key="shift.id" :value="shift.id">
                  {{ shift.name }}
                  ({{ shift.start_time }} - {{ shift.end_time }})
                </option>
              </select>
            </div>

            <!-- ADMISSION DATE -->

            <div class="mb-3">
              <label class="form-label"> Admission Date </label>

              <input v-model="selectedStudent.admission_date" type="date" class="form-control" />
            </div>

            <!-- MONTHLY FEE -->

            <div class="mb-3">
              <label class="form-label"> Monthly Fee </label>

              <input
                v-model="selectedStudent.monthly_fee"
                type="number"
                min="0"
                class="form-control"
                placeholder="Monthly Fee"
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

import { ref, reactive, computed, watch, onMounted, onBeforeUnmount } from 'vue'

import api from '@/services/api'

import * as bootstrap from 'bootstrap'

import 'bootstrap/dist/css/bootstrap.min.css'

import { getImageUrl } from '@/utils/img'

/* =====================================================
   DEFAULT IMAGE
===================================================== */

const defaultAvatar = 'https://i.pravatar.cc/150'

/* =====================================================
   STATES
===================================================== */

const students = ref([])

const sections = ref([])

const classes = ref([])

const classGroups = ref([])

const shifts = ref([])

const search = ref('')

const selectedClass = ref('')

const loading = ref(false)

/* =====================================================
   FILE STATES
===================================================== */

const addImageFile = ref(null)

const editImageFile = ref(null)

const addPreview = ref(null)

const editPreview = ref(null)

const addFileInput = ref(null)

const editFileInput = ref(null)

const addModalRef = ref(null)

/* =====================================================
   PAGINATION
===================================================== */

const currentPage = ref(1)

const perPage = ref(10)

/* =====================================================
   ADD STUDENT FORM
===================================================== */

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

/* =====================================================
   SELECTED STUDENT
===================================================== */

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

/* =====================================================
   GET SECTIONS
===================================================== */

const getSections = async () => {
  try {
    const res = await api.get('/sections')

    sections.value = res.data.sections || res.data.data || res.data || []
  } catch (error) {
    console.error('Error fetching sections:', error.response?.data || error)
  }
}

/* =====================================================
   GET CLASSES
===================================================== */

const getClasses = async () => {
  try {
    const res = await api.get('/classes')

    classes.value = res.data.classes || res.data.data || res.data || []
  } catch (error) {
    console.error('Error fetching classes:', error.response?.data || error)
  }
}

/* =====================================================
   GET CLASS GROUPS
===================================================== */

const getClassGroups = async () => {
  try {
    const res = await api.get('/class_group')

    classGroups.value = res.data.classGroups || res.data.groups || res.data.data || res.data || []
  } catch (error) {
    console.error('Error fetching class groups:', error.response?.data || error)
  }
}

/* =====================================================
   GET SHIFTS
===================================================== */

const getShifts = async () => {
  try {
    const res = await api.get('/shifts')

    shifts.value = res.data.data || res.data.shifts || res.data || []
  } catch (error) {
    console.error('Error fetching shifts:', error.response?.data || error)
  }
}

/* =====================================================
   IMAGE CHANGE
===================================================== */

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

/* =====================================================
   RESET FORM
===================================================== */

const resetForm = () => {
  form.full_name = ''

  form.fathers_name = ''

  form.mothers_name = ''

  form.phone = ''

  form.email = ''

  form.course_name = ''

  form.class_group_id = ''

  form.class_id = ''

  form.section_id = ''

  form.shift_id = ''

  form.admission_date = ''

  form.monthly_fee = ''

  addImageFile.value = null

  if (addPreview.value) {
    URL.revokeObjectURL(addPreview.value)

    addPreview.value = null
  }

  if (addFileInput.value) {
    addFileInput.value.value = ''
  }
}

/* =====================================================
   GET ALL STUDENTS
===================================================== */

const getStudent = async () => {
  try {
    loading.value = true

    const res = await api.get('/students')

    students.value = res.data.students || res.data.data || res.data || []
  } catch (error) {
    console.error('Error fetching students:', error.response?.data || error)
  } finally {
    loading.value = false
  }
}

/* =====================================================
   CREATE STUDENT
===================================================== */

const student_create = async () => {
  try {
    loading.value = true

    const formData = new FormData()

    formData.append('full_name', form.full_name)

    formData.append('fathers_name', form.fathers_name)

    formData.append('mothers_name', form.mothers_name)

    formData.append('phone', form.phone)

    formData.append('email', form.email)

    formData.append('course_name', form.course_name)

    formData.append('class_group_id', form.class_group_id)

    formData.append('class_id', form.class_id)

    formData.append('section_id', form.section_id)

    formData.append('shift_id', form.shift_id)

    formData.append('admission_date', form.admission_date)

    formData.append('monthly_fee', form.monthly_fee)

    if (addImageFile.value) {
      formData.append('image', addImageFile.value)
    }

    const res = await api.post('/students', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    })

    alert(res.data.message || 'Student Created Successfully')

    closeModal('addModal')

    resetForm()

    await getStudent()
  } catch (error) {
    console.error('Create student error:', error.response?.data || error)

    alert(error.response?.data?.message || 'Failed to create student')
  } finally {
    loading.value = false
  }
}

/* =====================================================
   OPEN VIEW
===================================================== */

const openView = (student) => {
  selectedStudent.value = {
    ...student,
  }
}

/* =====================================================
   OPEN EDIT
===================================================== */

const openEdit = (student) => {
  selectedStudent.value = JSON.parse(JSON.stringify(student))

  if (!selectedStudent.value.class_group_id && selectedStudent.value.class_group) {
    selectedStudent.value.class_group_id = selectedStudent.value.class_group.id
  }

  editImageFile.value = null

  if (editPreview.value) {
    URL.revokeObjectURL(editPreview.value)

    editPreview.value = null
  }

  if (editFileInput.value) {
    editFileInput.value.value = ''
  }
}

/* =====================================================
   UPDATE STUDENT
===================================================== */

const updateStudent = async () => {
  try {
    loading.value = true

    const formData = new FormData()

    formData.append('full_name', selectedStudent.value.full_name || '')

    formData.append('phone', selectedStudent.value.phone || '')

    formData.append('email', selectedStudent.value.email || '')

    formData.append('course_name', selectedStudent.value.course_name || '')

    formData.append('class_group_id', selectedStudent.value.class_group_id || '')

    formData.append('class_id', selectedStudent.value.class_id || '')

    formData.append('section_id', selectedStudent.value.section_id || '')

    formData.append('shift_id', selectedStudent.value.shift_id || '')

    formData.append('admission_date', selectedStudent.value.admission_date || '')

    formData.append('monthly_fee', selectedStudent.value.monthly_fee || '')

    formData.append('_method', 'PUT')

    if (editImageFile.value) {
      formData.append('image', editImageFile.value)
    }

    await api.post(`/students/${selectedStudent.value.id}`, formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    })

    alert('Updated successfully')

    closeModal('editModal')

    await getStudent()
  } catch (error) {
    console.error('Error updating student:', error.response?.data || error)

    alert(error.response?.data?.message || 'Update failed')
  } finally {
    loading.value = false
  }
}

/* =====================================================
   DELETE STUDENT
===================================================== */

const deleteStudent = async (id) => {
  if (!confirm('Are you sure you want to delete this student?')) {
    return
  }

  try {
    await api.delete(`/students/${id}`)

    students.value = students.value.filter((item) => item.id !== id)

    alert('Deleted successfully')
  } catch (error) {
    console.error('Error deleting student:', error.response?.data || error)

    alert('Failed to delete student')
  }
}

/* =====================================================
   CLOSE MODAL
===================================================== */

const closeModal = (modalId) => {
  const modalEl = document.getElementById(modalId)

  if (modalEl) {
    const modal = bootstrap.Modal.getInstance(modalEl) || new bootstrap.Modal(modalEl)

    modal.hide()
  }

  setTimeout(() => {
    document.querySelectorAll('.modal-backdrop').forEach((el) => el.remove())

    document.body.classList.remove('modal-open')

    document.body.style.overflow = ''

    document.body.style.paddingRight = ''
  }, 300)
}

/* =====================================================
   FILTER STUDENTS
===================================================== */

const filteredStudents = computed(() => {
  const keyword = search.value.toLowerCase().trim()

  return students.value.filter((student) => {
    const matchSearch =
      student.full_name?.toLowerCase().includes(keyword) ||
      student.email?.toLowerCase().includes(keyword) ||
      student.student_id?.toLowerCase().includes(keyword)

    const matchClass =
      !selectedClass.value || String(student.class_id) === String(selectedClass.value)

    return matchSearch && matchClass
  })
})

/* =====================================================
   PAGINATION
===================================================== */

const totalPages = computed(() => {
  return Math.ceil(filteredStudents.value.length / perPage.value) || 1
})

const paginatedStudents = computed(() => {
  const start = (currentPage.value - 1) * perPage.value

  return filteredStudents.value.slice(start, start + perPage.value)
})

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++
  }
}

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--
  }
}

/* =====================================================
   WATCH FILTER
===================================================== */

watch([search, selectedClass], () => {
  currentPage.value = 1
})

/* =====================================================
   MODAL CLEANUP
   MOBILE / BROWSER BACK FIX
===================================================== */

const cleanupModals = () => {
  // Remove stuck Bootstrap backdrop
  document.querySelectorAll('.modal-backdrop').forEach((el) => el.remove())

  // Close any opened modal
  document.querySelectorAll('.modal.show').forEach((modalEl) => {
    const modalInstance = bootstrap.Modal.getInstance(modalEl)

    if (modalInstance) {
      modalInstance.hide()
    }

    modalEl.classList.remove('show')

    modalEl.style.display = 'none'

    modalEl.removeAttribute('aria-modal')

    modalEl.setAttribute('aria-hidden', 'true')
  })

  // Restore body state
  document.body.classList.remove('modal-open')

  document.body.style.removeProperty('overflow')

  document.body.style.removeProperty('padding-right')
}

/* =====================================================
   ON MOUNTED
===================================================== */

onMounted(() => {
  getStudent()

  getSections()

  getClasses()

  getClassGroups()

  getShifts()

  // Browser / Mobile Back button
  window.addEventListener('popstate', cleanupModals)
})

/* =====================================================
   ON BEFORE UNMOUNT
===================================================== */

onBeforeUnmount(() => {
  cleanupModals()

  window.removeEventListener('popstate', cleanupModals)
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
