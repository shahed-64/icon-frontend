<template>
  <RouterView />

  <!-- Sidebar -->
  <AccountMenuView />

  <!-- MAIN LAYOUT -->
  <div class="layout bgc">
    <!-- CONTENT AREA -->
    <div class="content">
      <!-- WELCOME BANNER HEADER -->
      <div
        class="mb-4 text-white d-flex justify-content-between align-items-center position-relative overflow-hidden"
        style="
          background: linear-gradient(135deg, #2563eb 0%, #3b82f6 50%, #4f46e5 100%);
          border-radius: 18px;
          padding: 24px 32px;
          box-shadow: 0 10px 25px -5px rgba(37, 99, 235, 0.3);
        "
      >
        <!-- Content Left -->
        <div class="d-flex align-items-center gap-3" style="z-index: 1">
          <div
            class="d-flex align-items-center justify-content-center"
            style="
              width: 52px;
              height: 52px;
              background: rgba(255, 255, 255, 0.18);
              border-radius: 14px;
              border: 1px solid rgba(255, 255, 255, 0.25);
              font-size: 22px;
            "
          >
            <i class="fa-solid fa-gauge-high"></i>
          </div>

          <div>
            <h3 class="fw-bold mb-1" style="font-size: 22px; color: #ffffff">
              Welcome Back, Administrator 👋
            </h3>

            <p class="mb-0" style="font-size: 13px; color: rgba(255, 255, 255, 0.85)">
              Manage your students, payments and coaching activities from one place.
            </p>
          </div>
        </div>

        <!-- Badge Right -->
        <div
          class="d-flex align-items-center gap-2"
          style="
            background: rgba(255, 255, 255, 0.18);
            padding: 10px 16px;
            border-radius: 12px;
            border: 1px solid rgba(255, 255, 255, 0.25);
            z-index: 1;
          "
        >
          <i class="fa-regular fa-calendar-days" style="font-size: 20px"></i>

          <div style="line-height: 1.2">
            <small style="font-size: 10px; display: block; opacity: 0.8"> Today </small>

            <span style="font-size: 13px; font-weight: 700"> Dashboard </span>
          </div>
        </div>

        <!-- Background Decorative Circle -->
        <div
          style="
            position: absolute;
            right: -40px;
            bottom: -40px;
            width: 220px;
            height: 220px;
            background: rgba(255, 255, 255, 0.08);
            border-radius: 50%;
            pointer-events: none;
          "
        ></div>
      </div>

      <!-- MAIN CARD -->
      <div class="card-box">
        <!-- TOP FILTER AREA -->
        <div class="student-topbar">
          <div class="topbar-left">
            <p class="section-subtitle">Search and collect monthly payments.</p>
          </div>

          <!-- RIGHT FILTER AREA -->
          <div class="student-filters">
            <!-- CLASS FILTER -->
            <div class="filter-item">
              <select v-model="selectedClass" class="form-select custom-select">
                <option value="">All Classes</option>

                <option v-for="cls in uniqueClasses" :key="cls" :value="cls">
                  {{ cls }}
                </option>
              </select>
            </div>

            <!-- SEARCH BOX -->
            <div class="filter-item">
              <input
                v-model="search"
                type="text"
                class="form-control custom-input"
                placeholder="Search ID / Name / Email"
              />
            </div>
          </div>
        </div>

        <!-- STUDENT TABLE SECTION -->
        <div class="student-table-wrapper mt-3">
          <div class="table-responsive">
            <table class="table student-table align-middle">
              <thead>
                <tr>
                  <th style="width: 50px">#</th>
                  <th>Student ID</th>
                  <th>Name</th>
                  <th>Class</th>
                  <th>Email</th>
                  <th>Due Months</th>
                  <th>Status</th>
                  <th class="text-center">Action</th>
                </tr>
              </thead>

              <tbody>
                <tr v-for="(s, index) in paginatedStudents" :key="s.id">
                  <!-- NUMBER -->
                  <td>
                    <span class="row-index">
                      {{ (currentPage - 1) * perPage + index + 1 }}
                    </span>
                  </td>

                  <!-- STUDENT ID -->
                  <td>
                    <span class="id-badge">
                      {{ s.student_id }}
                    </span>
                  </td>

                  <!-- STUDENT NAME WITH AVATAR IMAGE -->
                  <td>
                    <div class="student-info-inline">
                      <img :src="getImageUrl(s)" alt="Student Avatar" class="student-table-img" />

                      <span class="student-name">
                        {{ s.full_name }}
                      </span>
                    </div>
                  </td>

                  <!-- CLASS -->
                  <td>
                    <span class="class-text">
                      {{ s.batch_name }}
                    </span>
                  </td>

                  <!-- EMAIL -->
                  <td>
                    <div class="email-text">
                      {{ s.email || 'No Email' }}
                    </div>
                  </td>

                  <!-- DUE MONTH -->
                  <td>
                    <div v-if="s.due_months?.length">
                      <span class="due-badge">
                        {{ s.due_months.join(', ') }}
                      </span>
                    </div>

                    <div v-else>
                      <span class="paid-badge"> Paid </span>
                    </div>
                  </td>

                  <!-- STATUS -->
                  <td>
                    <span
                      class="status-badge"
                      :class="s.status === 'Active' ? 'status-active' : 'status-inactive'"
                    >
                      {{ s.status }}
                    </span>
                  </td>

                  <!-- ACTION BUTTON -->
                  <td class="text-center">
                    <div class="action-buttons-group">
                      <!-- Payment Button -->
                      <button
                        class="btn-action btn-payment"
                        data-bs-toggle="modal"
                        data-bs-target="#paymentModal"
                        @click="openPaymentModal(s)"
                        title="Payment"
                      >
                        <svg
                          width="15"
                          height="15"
                          viewBox="0 0 24 24"
                          fill="none"
                          stroke="currentColor"
                          stroke-width="2"
                          stroke-linecap="round"
                          stroke-linejoin="round"
                          class="me-1"
                        >
                          <rect width="20" height="14" x="2" y="5" rx="2" />

                          <line x1="2" x2="22" y1="10" y2="10" />
                        </svg>

                        Payment
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- EMPTY STATE -->
        <div v-if="filteredStudents.length === 0" class="empty-state">
          <div class="empty-icon">
            <i class="fa-solid fa-user-slash"></i>
          </div>

          <h5>No Student Found</h5>

          <p>Try searching with another student ID or name</p>
        </div>

        <!-- PAGINATION SECTION -->
        <div v-if="filteredStudents.length > 0" class="pagination-container">
          <!-- LEFT INFO -->
          <div class="pagination-info">
            Showing

            <strong>
              {{ (currentPage - 1) * perPage + 1 }} -
              {{ Math.min(currentPage * perPage, filteredStudents.length) }}
            </strong>

            of

            <strong>
              {{ filteredStudents.length }}
            </strong>

            Students
          </div>

          <!-- RIGHT BUTTONS -->
          <div class="pagination-control">
            <button class="pagination-btn" @click="prevPage" :disabled="currentPage === 1">
              Previous
            </button>

            <div class="page-number">{{ currentPage }} / {{ totalPages }}</div>

            <button class="pagination-btn" @click="nextPage" :disabled="currentPage === totalPages">
              Next
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- PAYMENT MODAL -->
  <div class="modal fade" id="paymentModal" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content payment-modal">
        <!-- HEADER -->
        <div class="modal-header payment-header">
          <div class="d-flex align-items-center gap-3">
            <div class="modal-icon">
              <i class="fa-solid fa-money-bill-transfer"></i>
            </div>

            <div>
              <h5 class="modal-title text-white">Student Payment</h5>

              <p class="mb-0 text-white-50">Record monthly payment</p>
            </div>
          </div>

          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>

        <!-- BODY -->
        <div class="modal-body p-4">
          <!-- STUDENT INFO CARD -->
          <div class="student-payment-card mb-4">
            <img
              :src="getImageUrl(selectedStudent)"
              alt="Student Avatar"
              class="student-avatar-large-img"
            />

            <div>
              <h6 class="mb-1 text-dark fw-bold">
                {{ selectedStudent.full_name }}
              </h6>

              <span class="badge bg-primary-subtle text-primary fw-medium">
                {{ selectedStudent.student_id }}
              </span>
            </div>
          </div>

          <!-- MONTHLY FEE -->
          <div class="form-group mb-3">
            <label class="form-label fw-semibold text-secondary"> Monthly Fee </label>

            <div v-if="!selectedStudent.monthly_fee">
              <input
                v-model="form.amount"
                type="number"
                class="form-control custom-input"
                placeholder="Enter monthly fee"
              />
            </div>

            <div v-else>
              <input
                :value="selectedStudent.monthly_fee"
                class="form-control custom-input bg-light"
                readonly
              />
            </div>
          </div>

          <!-- PAID AMOUNT -->
          <div class="form-group mb-3">
            <label class="form-label fw-semibold text-secondary"> Paid Amount </label>

            <input
              v-model="form.paid_amount"
              class="form-control custom-input"
              placeholder="Enter paid amount"
            />
          </div>

          <!-- MONTH -->
          <div class="form-group mb-3">
            <label class="form-label fw-semibold text-secondary"> Payment Month </label>

            <select v-model="form.month" class="form-select custom-select">
              <option disabled value="">Select Month</option>

              <option v-for="month in availableMonths" :key="month" :value="month">
                {{ month }}
              </option>
            </select>
          </div>

          <!-- EXTRA FEES -->
          <div class="row g-3 mb-3">
            <div class="col-md-6">
              <label class="form-label fw-semibold text-secondary"> Admission Fee </label>

              <input
                type="number"
                class="form-control custom-input"
                v-model="form.admission_fee"
                placeholder="Admission fee"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label fw-semibold text-secondary"> Exam Fee </label>

              <input
                type="number"
                class="form-control custom-input"
                v-model="form.exam_fee"
                placeholder="Exam fee"
              />
            </div>
          </div>

          <!-- PAYMENT METHOD -->
          <div class="form-group mb-2">
            <label class="form-label fw-semibold text-secondary"> Payment Method </label>

            <select v-model="form.payment_method" class="form-select custom-select">
              <option disabled value="">Select Method</option>

              <option>Bkash</option>
              <option>Nogod</option>
              <option>Cash</option>
            </select>
          </div>
        </div>

        <!-- FOOTER -->
        <div class="modal-footer payment-footer">
          <button class="btn btn-light rounded-3 px-4" data-bs-dismiss="modal">Close</button>

          <button class="btn save-btn" @click="savePayment">
            <i class="fa-solid fa-check me-1"></i>
            Save Payment
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onBeforeUnmount, computed, watch } from 'vue'

import AccountMenuView from './AccountMenuView.vue'
import { useRouter } from 'vue-router'
import api from '@/services/api'
import { getImageUrl } from '@/utils/img'
import * as bootstrap from 'bootstrap'

const router = useRouter()

const availableMonths = ref([])
const students = ref([])
const search = ref('')
const selectedClass = ref('')

/* Pagination */
const currentPage = ref(1)
const perPage = 10

/* Student */
const selectedStudent = reactive({
  student_id: '',
  full_name: '',
  monthly_fee: null,
  image: null,
})

/* Payment Form */
const form = reactive({
  student_id: '',
  amount: '',
  paid_amount: '',
  payment_method: '',
  payment_date: new Date().toISOString().slice(0, 10),
  month: '',
  admission_fee: null,
  exam_fee: null,
})

/* Search */
const filteredStudents = computed(() => {
  return students.value.filter((s) => {
    const keyword = search.value.toLowerCase()

    const matchSearch =
      s.student_id?.toLowerCase().includes(keyword) ||
      s.full_name?.toLowerCase().includes(keyword) ||
      s.email?.toLowerCase().includes(keyword)

    const matchClass = !selectedClass.value || s.batch_name === selectedClass.value

    return matchSearch && matchClass
  })
})

/* Total Pages */
const totalPages = computed(() => {
  return Math.ceil(filteredStudents.value.length / perPage) || 1
})

/* Paginated Data */
const paginatedStudents = computed(() => {
  const start = (currentPage.value - 1) * perPage

  return filteredStudents.value.slice(start, start + perPage)
})

/* Page */
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

/* Reset page on search */
watch(search, () => {
  currentPage.value = 1
})

watch(selectedClass, () => {
  currentPage.value = 1
})

/* Payment Modal */
const openPaymentModal = (student) => {
  selectedStudent.student_id = student.student_id

  selectedStudent.full_name = student.full_name

  selectedStudent.monthly_fee = student.monthly_fee

  selectedStudent.image = student.image

  availableMonths.value = student.available_months || []

  form.student_id = student.id

  form.amount = student.monthly_fee ?? ''

  form.paid_amount = ''

  form.payment_method = ''

  form.payment_date = new Date().toISOString().slice(0, 10)

  form.month = ''

  form.admission_fee = null

  form.exam_fee = null
}

/* API Calls */

/* Get Students List */
const getStudents = async () => {
  try {
    const res = await api.get('/students')

    /* backend data response handle */
    if (Array.isArray(res.data)) {
      students.value = res.data
    } else if (res.data && Array.isArray(res.data.students)) {
      students.value = res.data.students
    } else {
      students.value = []
    }
  } catch (err) {
    console.error('Error fetching students:', err)
  }
}

/* Modal Cleanup */
const cleanupModals = () => {
  document.querySelectorAll('.modal').forEach((modalEl) => {
    try {
      const instance = bootstrap.Modal.getInstance(modalEl)

      if (instance) {
        instance.dispose()
      }
    } catch (error) {
      console.warn('Modal cleanup warning:', error)
    }

    modalEl.classList.remove('show')

    modalEl.style.removeProperty('display')
    modalEl.style.removeProperty('padding-right')

    modalEl.removeAttribute('aria-modal')

    modalEl.setAttribute('aria-hidden', 'true')

    modalEl.removeAttribute('role')
  })

  document.querySelectorAll('.modal-backdrop').forEach((backdrop) => {
    backdrop.remove()
  })

  document.body.classList.remove('modal-open')

  document.body.style.removeProperty('overflow')

  document.body.style.removeProperty('padding-right')

  document.documentElement.style.removeProperty('overflow')

  document.documentElement.style.removeProperty('padding-right')
}

/* Browser Back */
const handleBrowserBack = () => {
  cleanupModals()
}

/* Save Payment API */
const savePayment = async () => {
  try {
    const res = await api.post('/payments', form)

    const paymentId = res.data?.payment?.id || res.data?.id

    const modalEl = document.getElementById('paymentModal')

    if (modalEl) {
      const modalInstance =
        bootstrap.Modal.getInstance(modalEl) || bootstrap.Modal.getOrCreateInstance(modalEl)

      modalInstance.hide()
    }

    cleanupModals()

    if (paymentId) {
      router.push(`/singlePayment/${paymentId}`)
    } else {
      await getStudents()
    }
  } catch (err) {
    console.error('Payment error:', err)

    if (err.response?.data?.message) {
      alert(err.response.data.message)
    } else {
      alert('Payment save failed')
    }
  }
}

/* Unique Classes */
const uniqueClasses = computed(() => {
  const classes = students.value.map((s) => s.batch_name).filter(Boolean)

  return [...new Set(classes)]
})

/* Mounted */
onMounted(() => {
  getStudents()

  window.addEventListener('popstate', handleBrowserBack)

  window.addEventListener('pageshow', handleBrowserBack)
})

/* Before Unmount */
onBeforeUnmount(() => {
  window.removeEventListener('popstate', handleBrowserBack)

  window.removeEventListener('pageshow', handleBrowserBack)

  cleanupModals()
})
</script>

<style scoped>
/* =========================
   MAIN LAYOUT & FONTS
========================= */

.bgc {
  background: #f8fafc;
  min-height: 100vh;

  font-family:
    'Inter',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    sans-serif;
}

.layout {
  display: flex;
  min-height: 100vh;
}

.content {
  flex: 1;
  margin-left: 250px;
  padding: 30px;
}

/* =========================
   CARD & TOPBAR
========================= */

.card-box {
  background: #ffffff;
  border-radius: 16px;
  padding: 24px;
  border: 1px solid #f1f5f9;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.03);
}

.student-topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
}

.section-subtitle {
  color: #64748b;
  font-size: 14px;
  margin: 0;
}

.student-filters {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.custom-select,
.custom-input {
  border: 1px solid #e2e8f0;
  background-color: #ffffff;
  border-radius: 8px;
  padding: 8px 14px;
  font-size: 14px;
  color: #334155;
  transition: all 0.2s ease-in-out;
}

.custom-select:hover,
.custom-input:hover {
  border-color: #cbd5e1;
}

.custom-select:focus,
.custom-input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
  outline: none;
}

/* =========================
   TABLE DESIGN
========================= */

.student-table-wrapper {
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #f1f5f9;
}

.student-table {
  margin: 0;
}

.student-table thead th {
  background: #2563eb;
  color: #ffffff;
  padding: 14px 16px;
  font-size: 13px;
  font-weight: 600;
  border: none;
  text-transform: capitalize;
}

.student-table tbody tr {
  border-bottom: 1px solid #f1f5f9;
  transition: all 0.2s ease-in-out;
}

.student-table tbody tr:hover {
  background-color: #f1f5f9;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.02);
}

.student-table td {
  padding: 14px 16px;
  color: #334155;
  font-size: 14px;
}

/* TABLE ELEMENTS */

.row-index {
  color: #64748b;
  font-weight: 500;
}

.id-badge {
  background-color: #eff6ff;
  color: #2563eb;
  padding: 4px 10px;
  border-radius: 6px;
  font-weight: 600;
  font-size: 13px;
  display: inline-block;
  transition: all 0.2s ease;
}

.student-table tbody tr:hover .id-badge {
  background-color: #dbeafe;
}

.student-info-inline {
  display: flex;
  align-items: center;
  gap: 10px;
}

/* Student Profile Image */

.student-table-img {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.student-name {
  font-weight: 600;
  color: #0f172a;
}

.class-text {
  color: #475569;
}

.email-text {
  color: #64748b;
}

/* =========================
   BADGES
========================= */

.paid-badge {
  background: #16a34a;
  color: white;
  padding: 4px 12px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 600;
}

.due-badge {
  background: #ef4444;
  color: white;
  padding: 4px 12px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 600;
}

.status-badge {
  padding: 4px 12px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 600;
}

.status-active {
  background: #16a34a;
  color: white;
}

.status-inactive {
  background: #94a3b8;
  color: white;
}

/* =========================
   ACTION BUTTONS
========================= */

.action-buttons-group {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.btn-action {
  border: none;
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.btn-action:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
}

.btn-action:active {
  transform: translateY(0);
}

.btn-payment {
  background: #16a34a;
  color: white;
}

.btn-payment:hover {
  background: #15803d;
}

/* =========================
   PAGINATION
========================= */

.pagination-container {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
}

.pagination-info {
  color: #64748b;
  font-size: 14px;
}

.pagination-control {
  display: flex;
  align-items: center;
  gap: 8px;
}

.pagination-btn {
  background: #f1f5f9;
  color: #334155;
  border: none;
  padding: 6px 14px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.pagination-btn:hover:not(:disabled) {
  background: #e2e8f0;
  color: #0f172a;
  transform: translateY(-1px);
}

.pagination-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.page-number {
  background: #eff6ff;
  color: #2563eb;
  padding: 6px 12px;
  border-radius: 6px;
  font-weight: 600;
  font-size: 13px;
}

/* =========================
   EMPTY STATE
========================= */

.empty-state {
  text-align: center;
  padding: 40px 20px;
}

.empty-icon {
  width: 60px;
  height: 60px;
  margin: auto;
  border-radius: 50%;
  background: #f1f5f9;
  color: #94a3b8;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
}

.empty-state h5 {
  margin-top: 12px;
  color: #334155;
}

.empty-state p {
  color: #94a3b8;
  font-size: 14px;
}

/* =========================
   PAYMENT MODAL
========================= */

.payment-modal {
  border-radius: 16px;
  border: none;
  overflow: hidden;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
}

.payment-header {
  padding: 20px 24px;
  border: none;
  background: linear-gradient(135deg, #2563eb 0%, #3b82f6 50%, #4f46e5 100%);
}

.modal-icon {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 18px;
}

.student-payment-card {
  background: #f8fafc;
  border-radius: 12px;
  padding: 12px 16px;
  display: flex;
  align-items: center;
  gap: 12px;
  border: 1px solid #f1f5f9;
}

.student-avatar-large-img {
  width: 46px;
  height: 46px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid #e2e8f0;
}

.payment-footer {
  padding: 16px 24px;
  border-top: 1px solid #f1f5f9;
  background: #f8fafc;
}

.save-btn {
  background: #16a34a;
  color: white;
  padding: 8px 20px;
  border-radius: 8px;
  font-weight: 500;
  border: none;
  transition: all 0.2s ease;
}

.save-btn:hover {
  background: #15803d;
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(22, 163, 74, 0.2);
}

/* =========================
   RESPONSIVE DESIGN
========================= */

@media (max-width: 768px) {
  .content {
    margin-left: 0;
    padding: 15px;
  }

  .student-topbar {
    flex-direction: column;
    align-items: stretch;
  }

  .student-filters {
    flex-direction: column;
  }

  .pagination-container {
    flex-direction: column;
    gap: 12px;
  }
}
</style>
