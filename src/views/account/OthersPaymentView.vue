<template>
  <RouterView />

  <AccountMenuView />

  <div class="box">
    <!-- =========================
        Page Header
    ========================= -->
    <div class="d-flex justify-content-between align-items-center mb-4">
      <div>
        <h3 class="fw-bold mb-1 text-dark">
          <i class="bi bi-cart-check-fill text-primary me-2"></i>Other Payments
        </h3>
        <p class="text-muted small mb-0">Manage all student other payments and fees.</p>
      </div>

      <button
        type="button"
        class="btn btn-primary d-inline-flex align-items-center gap-2 px-3 py-2 rounded-3 shadow-sm"
        data-bs-toggle="modal"
        data-bs-target="#otherPaymentModal"
        @click="resetForm"
      >
        <i class="bi bi-plus-circle-fill"></i>
        <span>Add Other Payment</span>
      </button>
    </div>

    <!-- =========================
        Summary Cards
    ========================= -->
    <div class="row g-3 mb-4">
      <div class="col-lg-3 col-md-6">
        <div class="card custom-card border-top-green border-0 shadow-sm rounded-4 h-100">
          <div class="card-body p-4 d-flex align-items-center justify-content-between">
            <div>
              <span class="card-title-text text-muted"> Total Other Payment </span>

              <h3 class="fw-bold mb-0 text-success mt-2">
                ৳ {{ totalOtherPayment.toLocaleString() }}
              </h3>
            </div>

            <div class="card-icon-box bg-green-light text-success">
              <i class="bi bi-receipt fs-4"></i>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-3 col-md-6">
        <div class="card custom-card border-top-red border-0 shadow-sm rounded-4 h-100">
          <div class="card-body p-4 d-flex align-items-center justify-content-between">
            <div>
              <span class="card-title-text text-muted"> This Month Collection </span>

              <h3 class="fw-bold mb-0 text-danger mt-2">
                ৳ {{ thisMonthOtherCollection.toLocaleString() }}
              </h3>
            </div>

            <div class="card-icon-box bg-red-light text-danger">
              <i class="bi bi-calendar-month fs-4"></i>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-3 col-md-6">
        <div class="card custom-card border-top-yellow border-0 shadow-sm rounded-4 h-100">
          <div class="card-body p-4 d-flex align-items-center justify-content-between">
            <div>
              <span class="card-title-text text-muted"> Total Items Sold </span>

              <h3 class="fw-bold mb-0 text-warning mt-2">
                {{ totalItems }}
              </h3>
            </div>

            <div class="card-icon-box bg-yellow-light text-warning">
              <i class="bi bi-bag-check-fill fs-4"></i>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-3 col-md-6">
        <div class="card custom-card border-top-blue border-0 shadow-sm rounded-4 h-100">
          <div class="card-body p-4 d-flex align-items-center justify-content-between">
            <div>
              <span class="card-title-text text-muted"> Last Purchase Date </span>

              <h5 class="fw-bold mb-0 text-primary mt-2">
                {{ lastPurchase }}
              </h5>
            </div>

            <div class="card-icon-box bg-blue-light text-primary">
              <i class="bi bi-calendar-check-fill fs-4"></i>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- =========================
        Main Table Card
    ========================= -->
    <div class="card border-0 shadow-sm rounded-4 overflow-hidden mb-4">
      <!-- Search Filter Bar -->
      <div class="card-header bg-white border-bottom py-3 px-4">
        <div class="row g-3 align-items-center justify-content-between">
          <div class="col-md-4">
            <div class="input-group">
              <span class="input-group-text bg-light border-end-0 text-muted">
                <i class="bi bi-search"></i>
              </span>

              <input
                type="text"
                class="form-control bg-light border-start-0 ps-0"
                placeholder="Search Student ID or Name..."
                v-model="tableSearch"
                @input="currentPage = 1"
              />
            </div>
          </div>

          <div class="col-md-3 text-end">
            <span class="badge bg-light text-dark border px-3 py-2 rounded-pill">
              Total Records: {{ filteredOtherPayments.length }}
            </span>
          </div>
        </div>
      </div>

      <!-- Table Content -->
      <div class="card-body p-0">
        <div class="table-responsive">
          <table class="my-custom-table w-100 align-middle">
            <thead>
              <tr>
                <th class="ps-4">#</th>
                <th>Student ID</th>
                <th>Student Name</th>
                <th>Item</th>
                <th>Qty</th>
                <th>Price</th>
                <th>Total</th>
                <th>Method</th>
                <th>Date</th>
                <th width="120" class="text-end pe-4">Action</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="(payment, index) in paginatedPayments" :key="payment.id" class="data-row">
                <td class="ps-4 fw-semibold text-muted">
                  {{ (currentPage - 1) * itemsPerPage + index + 1 }}
                </td>

                <td class="fw-bold text-dark">
                  {{ payment.student?.student_id || 'N/A' }}
                </td>

                <!-- Student Name with Avatar -->
                <td class="fw-medium">
                  <div class="d-flex align-items-center gap-2">
                    <img
                      :src="getImageUrl(payment.student)"
                      alt="Student Avatar"
                      class="student-table-img"
                    />

                    <span>
                      {{ payment.student?.full_name || 'N/A' }}
                    </span>
                  </div>
                </td>

                <td>
                  <span class="badge bg-light text-dark border px-2 py-1 fw-normal">
                    {{ payment.item_name }}
                  </span>
                </td>

                <td>{{ payment.quantity }}</td>

                <td>৳ {{ Number(payment.price).toLocaleString() }}</td>

                <td class="fw-bold text-success">
                  ৳ {{ Number(payment.total_amount).toLocaleString() }}
                </td>

                <td>
                  <span
                    class="badge bg-primary-subtle text-primary border border-primary-subtle rounded-pill px-3 py-1"
                  >
                    {{ payment.payment_method }}
                  </span>
                </td>

                <td class="text-muted small">
                  {{ new Date(payment.payment_date).toLocaleDateString() }}
                </td>

                <td class="text-end pe-4">
                  <button
                    class="btn btn-sm btn-light text-primary me-2 action-btn"
                    @click="editPayment(payment)"
                  >
                    <i class="bi bi-pencil"></i>
                  </button>

                  <button
                    class="btn btn-sm btn-light text-danger action-btn"
                    @click="deletePayment(payment.id)"
                  >
                    <i class="bi bi-trash"></i>
                  </button>
                </td>
              </tr>

              <tr v-if="paginatedPayments.length === 0">
                <td colspan="10" class="text-center py-5 text-muted">No other payments found.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- =========================
          PAGINATION FOOTER
      ========================= -->
      <div
        class="card-footer bg-white border-top py-3 px-4 d-flex justify-content-between align-items-center"
      >
        <div class="text-muted small">
          Showing
          <b>
            {{ paginatedPayments.length ? (currentPage - 1) * itemsPerPage + 1 : 0 }}
          </b>

          to

          <b>
            {{ Math.min(currentPage * itemsPerPage, filteredOtherPayments.length) }}
          </b>

          of

          <b>{{ filteredOtherPayments.length }}</b>

          entries
        </div>

        <!-- Pagination Controls -->
        <div class="d-flex align-items-center gap-1">
          <button class="pg-btn" :disabled="currentPage === 1" @click="currentPage--">
            <i class="bi bi-chevron-left"></i>
            Previous
          </button>

          <button
            v-for="page in totalPages"
            :key="page"
            class="pg-num-btn"
            :class="{ active: currentPage === page }"
            @click="currentPage = page"
          >
            {{ page }}
          </button>

          <button
            class="pg-btn"
            :disabled="currentPage === totalPages || totalPages === 0"
            @click="currentPage++"
          >
            Next
            <i class="bi bi-chevron-right"></i>
          </button>
        </div>
      </div>
    </div>

    <!-- =========================
        Add / Edit Modal
    ========================= -->
    <div
      class="modal fade"
      id="otherPaymentModal"
      tabindex="-1"
      aria-labelledby="otherPaymentModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-lg modal-dialog-centered">
        <div class="modal-content border-0 shadow-lg rounded-4">
          <div class="modal-header bg-primary text-white px-4 py-3">
            <h5
              class="modal-title d-flex align-items-center gap-2 fs-6 fw-bold"
              id="otherPaymentModalLabel"
            >
              <i class="bi" :class="isEdit ? 'bi-pencil-square' : 'bi-plus-circle-fill'"></i>

              {{ isEdit ? 'Edit Other Payment' : 'Add Other Payment' }}
            </h5>

            <button
              type="button"
              class="btn-close btn-close-white"
              data-bs-dismiss="modal"
              aria-label="Close"
              @click="resetForm"
            ></button>
          </div>

          <div class="modal-body p-4">
            <div class="row g-3">
              <!-- =========================
                  Student Search
              ========================= -->
              <div class="col-md-6 position-relative">
                <label class="form-label fw-semibold"> Student ID </label>

                <input
                  type="text"
                  class="form-control"
                  placeholder="Search Student ID..."
                  v-model="searchStudent"
                  autocomplete="off"
                />

                <div
                  v-if="filteredStudents.length"
                  class="list-group position-absolute w-100 shadow rounded-3 mt-1"
                  style="z-index: 1050; max-height: 180px; overflow-y: auto"
                >
                  <button
                    type="button"
                    class="list-group-item list-group-item-action py-2 text-start"
                    v-for="student in filteredStudents"
                    :key="student.id"
                    @click="selectStudent(student)"
                  >
                    <span class="fw-bold text-primary">
                      {{ student.student_id }}
                    </span>
                    -
                    {{ student.full_name }}
                  </button>
                </div>
              </div>

              <!-- Item Name -->
              <div class="col-md-6">
                <label class="form-label fw-semibold"> Item Name </label>

                <input
                  type="text"
                  class="form-control"
                  placeholder="Sheet / Card / Pen"
                  v-model="form.item_name"
                />
              </div>

              <!-- Quantity -->
              <div class="col-md-4">
                <label class="form-label fw-semibold"> Quantity </label>

                <input type="number" min="1" class="form-control" v-model.number="form.quantity" />
              </div>

              <!-- Price -->
              <div class="col-md-4">
                <label class="form-label fw-semibold"> Price </label>

                <input type="number" min="0" class="form-control" v-model.number="form.price" />
              </div>

              <!-- Total Amount -->
              <div class="col-md-4">
                <label class="form-label fw-semibold"> Total Amount </label>

                <input
                  type="text"
                  class="form-control bg-light fw-bold text-success"
                  :value="'৳ ' + totalPrice.toLocaleString()"
                  readonly
                />
              </div>

              <!-- Payment Method -->
              <div class="col-md-6">
                <label class="form-label fw-semibold"> Payment Method </label>

                <select class="form-select" v-model="form.payment_method">
                  <option value="Cash">Cash</option>
                  <option value="Bkash">Bkash</option>
                  <option value="Nagad">Nagad</option>
                  <option value="Bank">Bank</option>
                </select>
              </div>

              <!-- Payment Date -->
              <div class="col-md-6">
                <label class="form-label fw-semibold"> Payment Date </label>

                <input type="date" class="form-control" v-model="form.payment_date" />
              </div>

              <!-- Remarks -->
              <div class="col-12">
                <label class="form-label fw-semibold"> Remarks </label>

                <textarea
                  rows="2"
                  class="form-control"
                  placeholder="Optional remarks..."
                  v-model="form.remarks"
                ></textarea>
              </div>
            </div>
          </div>

          <div class="modal-footer bg-light px-4 py-3">
            <button
              type="button"
              class="btn btn-light text-secondary border"
              data-bs-dismiss="modal"
              @click="resetForm"
            >
              Cancel
            </button>

            <button
              type="button"
              class="btn btn-primary px-4 d-inline-flex align-items-center gap-2"
              @click="isEdit ? updatePayment() : savePayment()"
            >
              <i class="bi" :class="isEdit ? 'bi-check-circle-fill' : 'bi-save-fill'"></i>

              <span>
                {{ isEdit ? 'Update Payment' : 'Save Payment' }}
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import AccountMenuView from './AccountMenuView.vue'
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import api from '@/services/api'
import * as bootstrap from 'bootstrap'
import { getImageUrl } from '@/utils/img'

/* =========================
   State
========================= */
const otherPayments = ref([])
const students = ref([])
const tableSearch = ref('')

// Pagination Settings
const currentPage = ref(1)
const itemsPerPage = ref(10)

const searchStudent = ref('')
const selectedStudent = ref(null)
const isEdit = ref(false)
const editId = ref(null)

// Student search loading state
const studentSearchLoading = ref(false)

/* =========================
   Form
========================= */
const form = ref({
  student_id: '',
  item_name: '',
  quantity: 1,
  price: '',
  total_amount: 0,
  payment_method: 'Cash',
  payment_date: new Date().toISOString().slice(0, 10),
  remarks: '',
})

/* =========================
   Computed Properties
========================= */
const filteredOtherPayments = computed(() => {
  if (!tableSearch.value) {
    return otherPayments.value
  }

  const query = tableSearch.value.toLowerCase()

  return otherPayments.value.filter((payment) => {
    const studentId = payment.student?.student_id?.toLowerCase() || ''

    const studentName = payment.student?.full_name?.toLowerCase() || ''

    const itemName = payment.item_name?.toLowerCase() || ''

    return studentId.includes(query) || studentName.includes(query) || itemName.includes(query)
  })
})

/* =========================
   Pagination Computations
========================= */
const totalPages = computed(() => {
  return Math.ceil(filteredOtherPayments.value.length / itemsPerPage.value) || 1
})

const paginatedPayments = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value

  const end = start + itemsPerPage.value

  return filteredOtherPayments.value.slice(start, end)
})

const totalPrice = computed(() => {
  return Number(form.value.quantity || 0) * Number(form.value.price || 0)
})

const totalOtherPayment = computed(() => {
  return otherPayments.value.reduce((sum, payment) => sum + Number(payment.total_amount || 0), 0)
})

const totalItems = computed(() => {
  return otherPayments.value.reduce((sum, payment) => sum + Number(payment.quantity || 0), 0)
})

const lastPurchase = computed(() => {
  if (otherPayments.value.length === 0) {
    return '-'
  }

  return otherPayments.value[0].payment_date
})

/* =========================
   Student Search Results
========================= */
const filteredStudents = computed(() => {
  if (!searchStudent.value.trim()) {
    return []
  }

  if (selectedStudent.value && selectedStudent.value.student_id === searchStudent.value) {
    return []
  }

  return students.value
})

/* =========================
   This Month Collection
========================= */
const thisMonthOtherCollection = computed(() => {
  const currentMonth = new Date().getMonth()

  const currentYear = new Date().getFullYear()

  return otherPayments.value
    .filter((payment) => {
      const date = new Date(payment.payment_date)

      return date.getMonth() === currentMonth && date.getFullYear() === currentYear
    })
    .reduce((total, payment) => total + Number(payment.total_amount || 0), 0)
})

/* =========================
   API Requests
========================= */

const getOtherPayments = async () => {
  try {
    const response = await api.get('/other-payments')

    otherPayments.value = response.data.data || response.data

    // Keep pagination valid after reload
    if (currentPage.value > totalPages.value) {
      currentPage.value = totalPages.value
    }
  } catch (error) {
    console.error('Error fetching payments:', error)
  }
}

/*
|--------------------------------------------------------------------------
| Search Students From Backend
|--------------------------------------------------------------------------
| StudentController supports:
| /students?search=...&per_page=20
|
| This avoids loading all students into browser.
*/
const searchStudents = async (search = '') => {
  const query = search.trim()

  if (!query) {
    students.value = []
    studentSearchLoading.value = false
    return
  }

  studentSearchLoading.value = true

  try {
    const response = await api.get('/students', {
      params: {
        search: query,
        per_page: 20,
        page: 1,
      },
    })

    students.value = response.data?.students || response.data?.data || response.data || []
  } catch (error) {
    console.error('Error searching students:', error)

    students.value = []
  } finally {
    studentSearchLoading.value = false
  }
}

/* =========================
   Debounced Student Search
========================= */
let studentSearchTimer = null

watch(searchStudent, (newValue) => {
  clearTimeout(studentSearchTimer)

  // If selected student is cleared
  if (selectedStudent.value && newValue !== selectedStudent.value.student_id) {
    selectedStudent.value = null
    form.value.student_id = ''
  }

  if (!newValue.trim()) {
    students.value = []
    return
  }

  studentSearchTimer = setTimeout(() => {
    searchStudents(newValue)
  }, 300)
})

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
   Actions & Methods
========================= */

const selectStudent = (student) => {
  selectedStudent.value = student

  form.value.student_id = student.id

  searchStudent.value = student.student_id

  // Hide dropdown after selection
  students.value = []
}

const closeModal = () => {
  const modalElement = document.getElementById('otherPaymentModal')

  if (modalElement) {
    const modalInstance =
      bootstrap.Modal.getInstance(modalElement) || bootstrap.Modal.getOrCreateInstance(modalElement)

    modalInstance.hide()
  }

  setTimeout(() => {
    cleanupModals()
  }, 300)
}

const savePayment = async () => {
  try {
    form.value.total_amount = totalPrice.value

    await api.post('/other-payments', form.value)

    await getOtherPayments()

    resetForm()
    closeModal()
  } catch (error) {
    console.error('Save failed:', error.response?.data || error)
  }
}

const editPayment = async (payment) => {
  isEdit.value = true
  editId.value = payment.id

  /*
   * First try to use student data already
   * attached to the payment.
   */
  let matchedStudent = students.value.find((s) => s.id === payment.student_id) || payment.student

  /*
   * If the student is not currently available,
   * search the backend using student ID.
   */
  if (!matchedStudent && payment.student_id) {
    try {
      const response = await api.get('/students', {
        params: {
          search: payment.student_id,
          per_page: 20,
          page: 1,
        },
      })

      const result = response.data?.students || response.data?.data || response.data || []

      matchedStudent =
        result.find((student) => Number(student.id) === Number(payment.student_id)) || result[0]

      if (matchedStudent) {
        students.value = [matchedStudent]
      }
    } catch (error) {
      console.error('Error loading payment student:', error)
    }
  }

  if (matchedStudent) {
    selectedStudent.value = matchedStudent

    searchStudent.value = matchedStudent.student_id
  }

  form.value = {
    student_id: payment.student_id,
    item_name: payment.item_name,
    quantity: payment.quantity,
    price: payment.price,
    total_amount: payment.total_amount,
    payment_method: payment.payment_method,
    payment_date: payment.payment_date,
    remarks: payment.remarks || '',
  }

  const modal = bootstrap.Modal.getOrCreateInstance(document.getElementById('otherPaymentModal'))

  modal.show()
}

const updatePayment = async () => {
  try {
    form.value.total_amount = totalPrice.value

    await api.put(`/other-payments/${editId.value}`, form.value)

    await getOtherPayments()

    resetForm()
    closeModal()
  } catch (error) {
    console.error('Update failed:', error.response?.data || error)
  }
}

const deletePayment = async (id) => {
  if (!confirm('Are you sure you want to delete this payment?')) {
    return
  }

  try {
    await api.delete(`/other-payments/${id}`)

    await getOtherPayments()
  } catch (error) {
    console.error('Delete failed:', error)
  }
}

const resetForm = () => {
  isEdit.value = false
  editId.value = null
  searchStudent.value = ''
  selectedStudent.value = null
  students.value = []

  form.value = {
    student_id: '',
    item_name: '',
    quantity: 1,
    price: '',
    total_amount: 0,
    payment_method: 'Cash',
    payment_date: new Date().toISOString().slice(0, 10),
    remarks: '',
  }
}

/* =========================
   Lifecycle Hooks
========================= */
onMounted(() => {
  getOtherPayments()

  // Mobile / browser Back button
  // modal backdrop cleanup
  window.addEventListener('popstate', handleBrowserBack)

  window.addEventListener('pageshow', handleBrowserBack)
})

onBeforeUnmount(() => {
  clearTimeout(studentSearchTimer)

  window.removeEventListener('popstate', handleBrowserBack)

  window.removeEventListener('pageshow', handleBrowserBack)

  cleanupModals()
})
</script>
<style scoped>
.box {
  width: 85%;
  margin: auto;
  margin-left: 264px;
}

/* Top Cards Styling */

.custom-card {
  background-color: #ffffff;
  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease;
  cursor: pointer;
}

.custom-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.08) !important;
}

.border-top-green {
  border-top: 4px solid #10b981 !important;
}

.border-top-red {
  border-top: 4px solid #ef4444 !important;
}

.border-top-yellow {
  border-top: 4px solid #f59e0b !important;
}

.border-top-blue {
  border-top: 4px solid #3b82f6 !important;
}

.card-icon-box {
  width: 50px;
  height: 50px;
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bg-green-light {
  background-color: #d1fae5;
}

.bg-red-light {
  background-color: #fee2e2;
}

.bg-yellow-light {
  background-color: #fef3c7;
}

.bg-blue-light {
  background-color: #dbeafe;
}

.card-title-text {
  font-size: 0.85rem;
  font-weight: 500;
}

/* ===================================================
   TABLE & HOVER STYLING
=================================================== */

.my-custom-table {
  border-collapse: collapse;
}

.my-custom-table thead tr {
  background-color: #f8fafc;
  color: #64748b;
  font-size: 0.85rem;
  text-transform: uppercase;
  border-bottom: 2px solid #e2e8f0;
}

.my-custom-table th,
.my-custom-table td {
  padding: 14px 16px;
}

/* Row Styling */

.my-custom-table tbody tr.data-row {
  border-bottom: 1px solid #f1f5f9;
  background-color: #ffffff;
  transition:
    background-color 0.2s ease,
    transform 0.1s ease;
  cursor: pointer;
}

.my-custom-table tbody tr.data-row:hover {
  background-color: #f0fdf4 !important;
  box-shadow: inset 4px 0 0 #10b981;
}

/* Student Profile Image in Table */

.student-table-img {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

/* ===================================================
   PAGINATION BUTTONS STYLING
=================================================== */

.pg-btn {
  background: #ffffff;
  border: 1px solid #cbd5e1;
  color: #475569;
  padding: 6px 14px;
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: 500;
  transition: all 0.2s ease;
}

.pg-btn:hover:not(:disabled) {
  background-color: #0d6efd;
  border-color: #0d6efd;
  color: #ffffff;
}

.pg-btn:disabled {
  background-color: #f1f5f9;
  color: #94a3b8;
  cursor: not-allowed;
  border-color: #e2e8f0;
}

.pg-num-btn {
  background: #ffffff;
  border: 1px solid #cbd5e1;
  color: #475569;
  width: 32px;
  height: 32px;
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.pg-num-btn:hover {
  background-color: #e2e8f0;
}

.pg-num-btn.active {
  background-color: #0d6efd;
  border-color: #0d6efd;
  color: #ffffff;
}

@media (max-width: 768px) {
  .box {
    width: 100%;
    margin-left: 0;
    padding: 15px;
  }
}
</style>
