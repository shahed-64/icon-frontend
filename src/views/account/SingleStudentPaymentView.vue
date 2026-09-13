<template>
  <RouterView />

  <AccountMenuView />

  <div class="main-content">
    <!-- ================= Header with Staff Profile ================= -->
    <div class="d-flex justify-content-between align-items-center flex-wrap mb-4 gap-3">
      <div>
        <h2 class="fw-bold mb-1 header-title">Account Dashboard</h2>
        <p class="text-muted mb-0">Manage class wise payment reports and financial activities</p>
      </div>

      <!-- Logged-in Staff/User Profile Info -->
      <div class="d-flex align-items-center gap-3 bg-white px-3 py-2 rounded-pill shadow-sm border">
        <div class="text-end">
          <h6 class="fw-bold text-dark mb-0">
            {{ displayUser.name }}
          </h6>
          <small class="text-muted text-capitalize">
            {{ displayUser.role }}
          </small>
        </div>

        <div class="profile-avatar-wrapper">
          <img
            :src="getImageUrl(displayUser)"
            alt="Staff Avatar"
            class="rounded-circle border border-2 border-primary object-fit-cover shadow-sm"
            width="45"
            height="45"
            @error="onImageError"
          />
        </div>
      </div>
    </div>

    <!-- ================= Search & Filters ================= -->
    <div class="d-flex justify-content-between align-items-center flex-wrap mb-4 gap-3">
      <div class="search-box">
        <div class="input-group">
          <span class="input-group-text">
            <i class="fa-solid fa-magnifying-glass"></i>
          </span>

          <input
            v-model="search"
            type="text"
            class="form-control"
            placeholder="Search Student ID or Name"
            @input="currentPage = 1"
          />
        </div>
      </div>
    </div>

    <!-- ================= Summary Cards ================= -->
    <div class="row g-4 mb-4">
      <div class="col-lg-3 col-md-6">
        <div class="dash-summary-card blue">
          <div>
            <small>Total Students</small>
            <h2>{{ totalStudents }}</h2>
          </div>

          <div class="summary-icon">
            <i class="fa-solid fa-user-graduate"></i>
          </div>
        </div>
      </div>

      <div class="col-lg-3 col-md-6">
        <div class="dash-summary-card green">
          <div>
            <small>Total Collection</small>
            <h2>৳ {{ totalCollection }}</h2>
          </div>

          <div class="summary-icon">
            <i class="fa-solid fa-money-bill-wave"></i>
          </div>
        </div>
      </div>

      <div class="col-lg-3 col-md-6">
        <div class="dash-summary-card red">
          <div>
            <small>Total Due</small>
            <h2>৳ {{ totalDue }}</h2>
          </div>

          <div class="summary-icon">
            <i class="fa-solid fa-circle-exclamation"></i>
          </div>
        </div>
      </div>

      <div class="col-lg-3 col-md-6">
        <div class="dash-summary-card orange">
          <div>
            <small>Total Unpaid Students</small>
            <h2>{{ totalUnpaidStudents }}</h2>
          </div>

          <div class="summary-icon">
            <i class="fa-solid fa-user-xmark"></i>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= Batch Buttons ================= -->
    <div class="card shadow-sm border-0 mb-4">
      <div class="card-body">
        <div class="d-flex flex-wrap gap-2">
          <button
            v-for="batch in batches"
            :key="batch"
            @click="selectBatch(batch)"
            class="btn"
            :class="selectedBatch === batch ? 'btn-primary' : 'btn-outline-primary'"
          >
            {{ batch }}
          </button>
        </div>
      </div>
    </div>

    <!-- ================= Table ================= -->
    <div class="card border-0 shadow-sm mb-4">
      <div
        class="card-header bg-white d-flex justify-content-between align-items-center py-3 flex-wrap gap-2"
      >
        <h5 class="fw-bold mb-0">Student Payment Report</h5>

        <span class="badge bg-primary fs-6 fw-normal"> {{ totalStudents }} Students </span>
      </div>

      <div class="table-responsive">
        <table class="table table-hover align-middle mb-0">
          <thead class="table-light">
            <tr>
              <th>ID</th>
              <th>Student Name</th>
              <th>Batch</th>
              <th>Monthly Fee</th>
              <th>Total Paid</th>
              <th>Total Due</th>
              <th>Status</th>
              <th>Action</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="student in paginatedStudents" :key="student.id">
              <td class="fw-bold">
                {{ student.student_id }}
              </td>

              <td>
                <div class="d-flex align-items-center gap-2">
                  <img
                    :src="getImageUrl(student)"
                    alt="Student Avatar"
                    class="student-table-img rounded-circle object-fit-cover border"
                    width="38"
                    height="38"
                    @error="onImageError"
                  />

                  <span class="fw-semibold text-dark">
                    {{ student.full_name }}
                  </span>
                </div>
              </td>

              <td>
                <span class="badge bg-light text-dark border">
                  {{ student.batch_name }}
                </span>
              </td>

              <td>৳ {{ Number(student.monthly_fee || 0).toLocaleString() }}</td>

              <td class="text-success fw-semibold">
                ৳
                {{
                  Number(
                    student.payments?.reduce((sum, p) => sum + Number(p.paid_amount || 0), 0) || 0,
                  ).toLocaleString()
                }}
              </td>

              <td class="text-danger fw-semibold">
                ৳
                {{
                  Number(
                    (student.payments?.reduce((sum, p) => sum + Number(p.due_amount || 0), 0) ||
                      0) +
                      (student.due_months?.length || 0) * Number(student.monthly_fee || 0),
                  ).toLocaleString()
                }}
              </td>

              <td>
                <span
                  class="badge"
                  :class="
                    student.due_months?.length > 0 ||
                    student.payments?.some((p) => p.status === 'due')
                      ? 'bg-danger'
                      : 'bg-success'
                  "
                >
                  {{
                    student.due_months?.length > 0 ||
                    student.payments?.some((p) => p.status === 'due')
                      ? 'Due'
                      : 'Paid'
                  }}
                </span>
              </td>

              <td>
                <button
                  class="btn btn-sm btn-primary"
                  @click="openSummary(student)"
                  data-bs-toggle="modal"
                  data-bs-target="#studentModal"
                >
                  View
                </button>
              </td>
            </tr>

            <tr v-if="filteredStudents.length === 0">
              <td colspan="8" class="text-center py-4 text-muted">No record found!</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pagination Footer -->
      <div
        class="card-footer bg-white py-3 border-0 d-flex justify-content-between align-items-center flex-wrap gap-3"
      >
        <span class="text-muted fs-7">
          Showing {{ showingStart }} to {{ showingEnd }} of {{ filteredStudents.length }} entries
        </span>

        <nav v-if="totalPages > 0">
          <ul class="pagination pagination-sm mb-0 align-items-center">
            <li class="page-item" :class="{ disabled: currentPage === 1 }">
              <button class="page-link px-3" @click="currentPage--" :disabled="currentPage === 1">
                <i class="fa-solid fa-chevron-left me-1"></i>
                Prev
              </button>
            </li>

            <li
              v-for="page in totalPages"
              :key="page"
              class="page-item"
              :class="{ active: currentPage === page }"
            >
              <button class="page-link" @click="currentPage = page">
                {{ page }}
              </button>
            </li>

            <li
              class="page-item"
              :class="{
                disabled: currentPage === totalPages || filteredStudents.length === 0,
              }"
            >
              <button
                class="page-link px-3"
                @click="currentPage++"
                :disabled="currentPage === totalPages || filteredStudents.length === 0"
              >
                Next
                <i class="fa-solid fa-chevron-right ms-1"></i>
              </button>
            </li>
          </ul>
        </nav>
      </div>
    </div>
  </div>

  <!-- Student Profile Modal -->
  <div
    class="modal fade"
    id="studentModal"
    tabindex="-1"
    aria-labelledby="studentModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog modal-lg modal-dialog-centered">
      <div class="modal-content border-0 shadow-lg">
        <div class="modal-header border-0 pb-0">
          <h4 class="fw-bold mb-0" id="studentModalLabel">Student Profile</h4>

          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>

        <div class="modal-body p-4">
          <div class="row g-4">
            <div class="col-md-4">
              <div class="profile-card text-center">
                <img
                  :src="getImageUrl(selectedStudent)"
                  class="profile-img mb-3 rounded-circle object-fit-cover"
                  alt="Profile Image"
                  @error="onImageError"
                />

                <h5 class="fw-bold mb-1">
                  {{ selectedStudent.full_name }}
                </h5>

                <p class="text-muted mb-2">
                  {{ selectedStudent.student_id }}
                </p>

                <span
                  class="badge"
                  :class="selectedStudent.status == 'active' ? 'bg-success' : 'bg-danger'"
                >
                  {{ selectedStudent.status }}
                </span>

                <hr class="my-3" />

                <div class="profile-info text-start">
                  <div>
                    <small class="d-block">Batch</small>
                    <h6 class="mb-0">
                      {{ selectedStudent.batch_name }}
                    </h6>
                  </div>

                  <div class="mt-3">
                    <small class="d-block">Monthly Fee</small>

                    <h6 class="mb-0">
                      ৳
                      {{ Number(selectedStudent.monthly_fee || 0).toLocaleString() }}
                    </h6>
                  </div>
                </div>
              </div>
            </div>

            <div class="col-md-8">
              <h6 class="fw-bold mb-3">Payment Summary</h6>

              <div class="row g-3">
                <div class="col-6 col-sm-6">
                  <div class="modal-summary-card">
                    <small>Total Paid</small>

                    <h4 class="text-success">
                      ৳
                      {{ Number(selectedStudent.total_paid || 0).toLocaleString() }}
                    </h4>
                  </div>
                </div>

                <div class="col-6 col-sm-6">
                  <div class="modal-summary-card">
                    <small>Total Due</small>

                    <h4 class="text-danger">
                      ৳
                      {{ Number(selectedStudent.total_due || 0).toLocaleString() }}
                    </h4>
                  </div>
                </div>

                <div class="col-6 col-sm-6">
                  <div class="modal-summary-card">
                    <small>Unpaid Month</small>

                    <h4 style="color: brown">
                      {{ selectedStudent.unpaid_months || 0 }}
                    </h4>
                  </div>
                </div>

                <div class="col-6 col-sm-6">
                  <div class="modal-summary-card">
                    <small>Unpaid Amount</small>

                    <h4 class="text-warning">
                      ৳
                      {{ Number(selectedStudent.unpaid_amount || 0).toLocaleString() }}
                    </h4>
                  </div>
                </div>
              </div>

              <div class="outstanding-card mt-4">
                <small>Total Outstanding</small>

                <h2 class="text-danger fw-bold mt-2 mb-0">
                  ৳
                  {{ Number(selectedStudent.total_outstanding || 0).toLocaleString() }}
                </h2>
              </div>
            </div>
          </div>
        </div>

        <div class="modal-footer border-0 pt-0">
          <button type="button" class="btn btn-secondary px-4 rounded-pill" data-bs-dismiss="modal">
            Close
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

import AccountMenuView from './AccountMenuView.vue'
import api from '@/services/api'
import { getImageUrl } from '@/utils/img'
import * as bootstrap from 'bootstrap'

/* =========================
   Search & State
========================= */

const search = ref('')

const totalStudents = ref(0)
const totalCollection = ref(0)
const totalDue = ref(0)
const totalUnpaidStudents = ref(0)

const payments = ref([])
const students = ref([])
const selectedStudent = ref({})

/* =========================
   Staff User
========================= */

const apiUser = ref(null)

const defaultAvatar = 'https://ui-avatars.com/api/?name=Staff&background=random'

const displayUser = computed(() => {
  const u = apiUser.value || {}

  return {
    name: u.name || 'Accountant / Staff',
    role: u.role || 'Staff',
    image: u.image || null,
  }
})

const onImageError = (e) => {
  e.target.onerror = null
  e.target.src = defaultAvatar
}

/* =========================
   Pagination
========================= */

const currentPage = ref(1)
const itemsPerPage = ref(10)

/* =========================
   Batch
========================= */

const batches = ref([
  'All',
  'Class-1',
  'Class-2',
  'Class-3',
  'Class-4',
  'Class-5',
  'Class-6',
  'Class-7',
  'Class-8',
  'Class-9',
  'Class-10',
  'Class-11',
  'Class-12',
])

const selectedBatch = ref('All')

const selectBatch = (batch) => {
  selectedBatch.value = batch
  currentPage.value = 1
}

/* =========================
   Modal Cleanup
========================= */

const cleanupModals = () => {
  document.querySelectorAll('.modal').forEach((modalElement) => {
    try {
      const modalInstance = bootstrap.Modal.getInstance(modalElement)

      if (modalInstance) {
        modalInstance.dispose()
      }
    } catch (error) {
      console.warn('Modal cleanup warning:', error)
    }

    modalElement.classList.remove('show')

    modalElement.style.removeProperty('display')
    modalElement.style.removeProperty('padding-right')

    modalElement.removeAttribute('aria-modal')
    modalElement.setAttribute('aria-hidden', 'true')

    modalElement.removeAttribute('role')
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

/* =========================
   Browser Back Fix
========================= */

const handleBrowserBack = () => {
  cleanupModals()
}

/* =========================
   Load Data
========================= */

onMounted(async () => {
  try {
    /* -------------------------------------------
       Get Payment Summary
    ------------------------------------------- */

    const paymentRes = await api.get('/payments')

    totalCollection.value = paymentRes.data.total_paid_amount || 0

    totalDue.value = paymentRes.data.total_due_amount || 0

    totalUnpaidStudents.value = paymentRes.data.total_unpaid_students || 0

    if (paymentRes.data.user) {
      apiUser.value = paymentRes.data.user
    } else {
      try {
        const dashRes = await api.get('/staff/dashboard')

        if (dashRes.data.user) {
          apiUser.value = dashRes.data.user
        }
      } catch (e) {
        console.log(e)
      }
    }

    /* -------------------------------------------
       Get All Students
    ------------------------------------------- */

    const studentRes = await api.get('/students')

    const allStudents = studentRes.data.students || []

    totalStudents.value = allStudents.length

    /* -------------------------------------------
       Merge Student + Payment
    ------------------------------------------- */

    const formattedStudents = allStudents.map((student) => {
      let latestPayment = null

      if (student.payments && student.payments.length > 0) {
        latestPayment = student.payments[student.payments.length - 1]
      }

      return {
        id: latestPayment ? latestPayment.id : student.id,

        student: {
          ...student,
        },

        amount: latestPayment ? latestPayment.amount : student.monthly_fee || 0,

        paid_amount: latestPayment ? latestPayment.paid_amount : 0,

        due_amount: latestPayment ? latestPayment.due_amount : student.monthly_fee || 0,

        status: latestPayment ? latestPayment.status : 'due',
      }
    })

    payments.value = formattedStudents

    students.value = allStudents

    /* -------------------------------------------
       Browser Back / Page Show Modal Cleanup
    ------------------------------------------- */

    window.addEventListener('popstate', handleBrowserBack)

    window.addEventListener('pageshow', handleBrowserBack)
  } catch (error) {
    console.error('API Error:', error)

    // Even if API fails, register modal cleanup
    window.addEventListener('popstate', handleBrowserBack)

    window.addEventListener('pageshow', handleBrowserBack)
  }
})

/* =========================
   Modal
========================= */

const openSummary = (student) => {
  const reportStudent = students.value.find((s) => Number(s.id) === Number(student.id))

  if (!reportStudent) return

  const totalPaid =
    reportStudent.payments?.reduce((sum, payment) => sum + Number(payment.paid_amount || 0), 0) || 0

  const totalDue =
    reportStudent.payments?.reduce((sum, payment) => sum + Number(payment.due_amount || 0), 0) || 0

  const unpaidMonths = reportStudent.due_months?.length || 0

  const unpaidAmount = unpaidMonths * Number(reportStudent.monthly_fee || 0)

  const totalOutstanding = totalDue + unpaidAmount

  selectedStudent.value = {
    ...reportStudent,

    total_paid: totalPaid,
    total_due: totalDue,
    unpaid_months: unpaidMonths,
    unpaid_amount: unpaidAmount,
    total_outstanding: totalOutstanding,
  }

  console.log('MODAL DATA:', selectedStudent.value)
}

/* =========================
   Filter
========================= */

const filteredStudents = computed(() => {
  return students.value.filter((student) => {
    const batchMatch = selectedBatch.value === 'All' || student.batch_name === selectedBatch.value

    const searchText = search.value.toLowerCase()

    const searchMatch =
      student.full_name?.toLowerCase().includes(searchText) ||
      student.student_id?.toLowerCase().includes(searchText)

    return batchMatch && searchMatch
  })
})

/* =========================
   Pagination
========================= */

const paginatedStudents = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value

  return filteredStudents.value.slice(start, start + itemsPerPage.value)
})

const totalPages = computed(() => {
  return Math.ceil(filteredStudents.value.length / itemsPerPage.value) || 1
})

const showingStart = computed(() => {
  if (filteredStudents.value.length === 0) {
    return 0
  }

  return (currentPage.value - 1) * itemsPerPage.value + 1
})

const showingEnd = computed(() => {
  const end = currentPage.value * itemsPerPage.value

  return end > filteredStudents.value.length ? filteredStudents.value.length : end
})

/* =========================
   Component Cleanup
========================= */

onBeforeUnmount(() => {
  window.removeEventListener('popstate', handleBrowserBack)

  window.removeEventListener('pageshow', handleBrowserBack)

  cleanupModals()
})
</script>

<style scoped>
.main-content {
  margin-left: 260px;
  width: calc(100% - 260px);
  min-height: 100vh;
  padding: 30px;
  background: #f4f7fb;
  transition: all 0.3s ease;
}

.search-box {
  width: 320px;
}

.search-box .input-group {
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.06);
}

.search-box .input-group-text {
  background: #fff;
  border: none;
}

.search-box .form-control {
  border: none;
  box-shadow: none;
}

.dash-summary-card {
  border-radius: 18px;
  padding: 25px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
  transition: all 0.3s ease;
  cursor: pointer;
  min-height: 120px;
}

.dash-summary-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 20px 35px rgba(0, 0, 0, 0.18);
}

.dash-summary-card small {
  opacity: 0.9;
  font-size: 15px;
}

.dash-summary-card h2 {
  margin-top: 10px;
  font-size: 28px;
  font-weight: 700;
  color: #ffffff !important;
  margin-bottom: 0;
}

.blue {
  background: linear-gradient(135deg, #3b82f6, #2563eb);
}

.green {
  background: linear-gradient(135deg, #22c55e, #16a34a);
}

.red {
  background: linear-gradient(135deg, #ef4444, #dc2626);
}

.orange {
  background: linear-gradient(135deg, #f59e0b, #ea580c);
}

.summary-icon {
  width: 65px;
  height: 65px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  flex-shrink: 0;
}

.card {
  border: none;
  border-radius: 18px;
  overflow: hidden;
}

.btn-outline-primary,
.btn-primary {
  padding: 9px 22px;
  border-radius: 30px;
  font-weight: 600;
  transition: all 0.25s ease;
}

.btn-outline-primary:hover {
  transform: translateY(-2px);
}

.btn-sm {
  padding: 7px 16px;
  border-radius: 10px;
}

.table thead th {
  background: #f8fafc;
  font-weight: 700;
  white-space: nowrap;
  padding: 15px;
}

.table td {
  padding: 15px;
  vertical-align: middle;
  white-space: nowrap;
}

.table tbody tr:hover {
  background: #f7fbff;
}

.badge {
  padding: 8px 14px;
  border-radius: 30px;
  font-size: 13px;
}

.pagination .page-item .page-link {
  border: 1px solid #e5e7eb;
  padding: 6px 14px;
  margin: 0 3px;
  border-radius: 8px;
  color: #4b5563;
  font-weight: 600;
  box-shadow: none;
}

.pagination .page-item.active .page-link {
  background-color: #2563eb;
  border-color: #2563eb;
  color: #fff;
}

.profile-card {
  background: #f8fafc;
  border: 1px solid #e9ecef;
  border-radius: 18px;
  padding: 25px;
  text-align: center;
  height: 100%;
}

.profile-img {
  width: 110px;
  height: 110px;
  border-radius: 50%;
  object-fit: cover;
  border: 4px solid white;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
}

.modal-summary-card {
  background: white;
  border: 1px solid #ececec;
  border-radius: 15px;
  padding: 18px;
  transition: all 0.25s ease;
}

.modal-summary-card:hover {
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.outstanding-card {
  background: #f8f9fa;
  border-radius: 16px;
  border-left: 5px solid #dc3545;
  padding: 20px;
}

@media (max-width: 992px) {
  .main-content {
    margin-left: 0 !important;
    width: 100% !important;
    padding: 18px;
  }

  .search-box {
    width: 100%;
  }
}
</style>
