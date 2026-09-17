<template>
  <dashPageView />

  <div class="final-result-page">
    <!-- Sidebar Space -->
    <div class="sidebar-space"></div>

    <!-- Main Content -->
    <div class="final-result-content">
      <!-- Page Header -->
      <div class="page-header">
        <div>
          <h2 class="page-title">Final Result</h2>
          <p class="page-subtitle">Configure examination percentages for the final result.</p>
        </div>

        <!-- Header Actions -->
        <div class="header-actions">
          <div class="status-badge" :class="isComplete ? 'complete' : 'incomplete'">
            <span class="status-dot"></span>

            {{ isComplete ? 'Configuration Complete' : 'Configuration Incomplete' }}
          </div>

          <!-- Download PDF -->
          <button type="button" class="download-pdf-btn" @click="goToFinalResultPdf">
            <i class="bi bi-file-earmark-pdf"></i>
            Download PDF
          </button>
        </div>
      </div>

      <!-- Summary Cards -->
      <div class="summary-grid">
        <!-- Examinations -->
        <div class="summary-card">
          <div class="summary-icon blue">
            <i class="bi bi-journal-text"></i>
          </div>

          <div>
            <div class="summary-label">Examinations</div>

            <div class="summary-value">
              {{ selectedYearExaminationCount }}
            </div>
          </div>
        </div>

        <!-- Total Percentage -->
        <div class="summary-card">
          <div class="summary-icon green">
            <i class="bi bi-percent"></i>
          </div>

          <div>
            <div class="summary-label">
              Total Percentage
              <span v-if="currentYear" style="font-size: 11px"> ({{ currentYear }}) </span>
            </div>

            <div class="summary-value">{{ totalPercentage.toFixed(2) }}%</div>
          </div>
        </div>

        <!-- Remaining -->
        <div class="summary-card">
          <div class="summary-icon orange">
            <i class="bi bi-hourglass-split"></i>
          </div>

          <div>
            <div class="summary-label">
              Remaining
              <span v-if="currentYear" style="font-size: 11px"> ({{ currentYear }}) </span>
            </div>

            <div class="summary-value">{{ remainingPercentage.toFixed(2) }}%</div>
          </div>
        </div>
      </div>

      <!-- Main Grid -->
      <div class="main-grid">
        <!-- Add Examination -->
        <div class="card-box">
          <div class="card-header">
            <div>
              <h5>Add Examination</h5>
              <p>Set the percentage contribution of an examination.</p>
            </div>

            <div class="header-icon">
              <i class="bi bi-plus-lg"></i>
            </div>
          </div>

          <form @submit.prevent="addFinalResult">
            <!-- Examination -->
            <div class="form-group">
              <label>
                Examination
                <span>*</span>
              </label>

              <select
                v-model="form.examination_id"
                class="form-control"
                :disabled="loading || saving"
              >
                <option value="">Select Examination</option>

                <option v-for="exam in availableExaminations" :key="exam.id" :value="exam.id">
                  {{ exam.examination_type }}
                  -
                  {{ exam.examination_year }}
                  ({{ exam.exam_mark || 'N/A' }} Marks)
                </option>
              </select>
            </div>

            <!-- Percentage -->
            <div class="form-group">
              <label>
                Percentage
                <span>*</span>
              </label>

              <div class="percentage-input">
                <input
                  v-model="form.percentage"
                  type="number"
                  min="0.01"
                  :max="remainingPercentage || 100"
                  step="0.01"
                  class="form-control"
                  placeholder="Enter percentage"
                  :disabled="loading || saving || !form.examination_id || remainingPercentage <= 0"
                />

                <span>%</span>
              </div>

              <small class="input-hint">
                <template v-if="currentYear">
                  Remaining percentage for
                  <strong>{{ currentYear }}</strong
                  >:
                  <strong> {{ remainingPercentage.toFixed(2) }}% </strong>
                </template>

                <template v-else> Select an examination to see the remaining percentage. </template>
              </small>
            </div>

            <!-- Error -->
            <div v-if="errorMessage" class="alert-error">
              <i class="bi bi-exclamation-circle"></i>

              <span>
                {{ errorMessage }}
              </span>
            </div>

            <!-- Add Button -->
            <button
              type="submit"
              class="btn-primary"
              :disabled="
                saving ||
                loading ||
                !form.examination_id ||
                !form.percentage ||
                remainingPercentage <= 0
              "
            >
              <span v-if="saving" class="spinner"></span>

              <i v-else class="bi bi-plus-lg"></i>

              {{ saving ? 'Adding...' : 'Add Examination' }}
            </button>
          </form>
        </div>

        <!-- Percentage Progress -->
        <div class="card-box progress-box">
          <div class="card-header">
            <div>
              <h5>Percentage Distribution</h5>

              <p>
                <template v-if="currentYear">
                  Final result configuration for {{ currentYear }}.
                </template>

                <template v-else> Select an examination to view year-wise progress. </template>
              </p>
            </div>

            <div class="percentage-circle" :class="{ complete: isComplete }">
              {{ totalPercentage.toFixed(0) }}%
            </div>
          </div>

          <div class="progress-wrapper">
            <div class="progress-track">
              <div
                class="progress-fill"
                :style="{
                  width: Math.min(totalPercentage, 100) + '%',
                }"
              ></div>
            </div>

            <div class="progress-info">
              <span>0%</span>
              <strong>100%</strong>
            </div>
          </div>

          <div class="completion-message" :class="isComplete ? 'success' : 'warning'">
            <i :class="isComplete ? 'bi bi-check-circle-fill' : 'bi bi-info-circle-fill'"></i>

            <div>
              <strong>
                {{ isComplete ? 'Ready for Final Result' : 'Configuration in Progress' }}
              </strong>

              <p>
                {{
                  isComplete
                    ? `All examination percentages for ${
                        currentYear || 'this year'
                      } are correctly configured to 100%.`
                    : `Add examinations until the ${
                        currentYear || 'selected year'
                      } total percentage reaches 100%.`
                }}
              </p>
            </div>
          </div>
        </div>
      </div>

      <!-- Examination List -->
      <div class="card-box table-card">
        <div class="card-header">
          <div>
            <h5>Final Result Examinations</h5>

            <p>Examinations included in the final result calculation.</p>
          </div>

          <button class="refresh-btn" @click="loadData" :disabled="loading">
            <i class="bi bi-arrow-clockwise" :class="{ rotating: loading }"></i>

            Refresh
          </button>
        </div>

        <!-- Loading -->
        <div v-if="loading" class="loading-state">
          <div class="large-spinner"></div>

          <p>Loading final result configuration...</p>
        </div>

        <!-- Empty -->
        <div v-else-if="finalResults.length === 0" class="empty-state">
          <div class="empty-icon">
            <i class="bi bi-journal-plus"></i>
          </div>

          <h6>No Examination Added</h6>

          <p>
            Add examinations and assign their percentages to create the final result configuration.
          </p>
        </div>

        <!-- Grouped Table -->
        <div v-else class="table-wrapper">
          <table class="result-table">
            <thead>
              <tr>
                <th>#</th>
                <th>Examination</th>
                <th>Year</th>
                <th>Percentage</th>
                <th class="text-end">Action</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="(group, index) in groupedFinalResults" :key="group.year">
                <!-- Serial -->
                <td>
                  <span class="serial">
                    {{ index + 1 }}
                  </span>
                </td>

                <!-- Examinations -->
                <td>
                  <div class="grouped-examinations">
                    <div class="grouped-exam-icon">
                      <i class="bi bi-journal-text"></i>
                    </div>

                    <div class="exam-list-content">
                      <div class="exam-names">
                        <span
                          v-for="(item, examIndex) in group.items"
                          :key="item.id"
                          class="exam-name-badge"
                        >
                          {{ item.examination?.examination_type || 'N/A' }}

                          <span v-if="examIndex < group.items.length - 1" class="plus-separator">
                            +
                          </span>
                        </span>
                      </div>

                      <small>
                        {{ group.items.length }}

                        {{ group.items.length === 1 ? 'Examination' : 'Examinations' }}

                        included
                      </small>
                    </div>
                  </div>
                </td>

                <!-- Year -->
                <td>
                  <span class="year-badge">
                    {{ group.year }}
                  </span>
                </td>

                <!-- Percentage -->
                <td>
                  <div class="percentage-group">
                    <div class="percentage-list">
                      <span v-for="item in group.items" :key="item.id" class="percentage-item">
                        {{ Number(item.percentage).toFixed(2) }}%
                      </span>
                    </div>

                    <span class="percentage-total"> {{ group.totalPercentage.toFixed(2) }}% </span>
                  </div>
                </td>

                <!-- Action -->
                <td class="text-end">
                  <div class="action-buttons">
                    <!-- Edit -->
                    <button class="action-btn edit" title="Edit" @click="openEditGroup(group)">
                      <i class="bi bi-pencil"></i>
                    </button>

                    <!-- PDF -->
                    <router-link
                      :to="{
                        path: '/SingularStudentFinalResultView',
                        query: {
                          year: String(group.year),
                        },
                      }"
                      class="action-btn pdf"
                      title="View / Download Final Result"
                    >
                      <i class="bi bi-file-earmark-pdf"></i>
                    </router-link>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- Edit Modal -->
    <div v-if="editingGroup" class="modal-overlay" @click.self="closeEdit">
      <div class="edit-modal">
        <!-- Modal Header -->
        <div class="modal-header">
          <div>
            <h5>Edit Final Result</h5>

            <p>Academic Year {{ editingGroup.year }}</p>
          </div>

          <button class="modal-close" @click="closeEdit">
            <i class="bi bi-x-lg"></i>
          </button>
        </div>

        <!-- Modal Body -->
        <div class="modal-body">
          <div class="modal-year-header">
            <div class="modal-year-icon">
              <i class="bi bi-calendar3"></i>
            </div>

            <div>
              <strong>
                {{ editingGroup.year }}
              </strong>

              <span>
                {{ editingGroup.items.length }}

                {{ editingGroup.items.length === 1 ? 'Examination' : 'Examinations' }}
              </span>
            </div>
          </div>

          <!-- Group Items -->
          <div class="edit-exam-list">
            <div v-for="item in editingGroup.items" :key="item.id" class="edit-exam-row">
              <div class="edit-exam-left">
                <div class="exam-icon large">
                  <i class="bi bi-journal-text"></i>
                </div>

                <div>
                  <strong>
                    {{ item.examination?.examination_type || 'N/A' }}
                  </strong>

                  <span>
                    {{ item.examination?.exam_mark || 'N/A' }}
                    Marks
                  </span>
                </div>
              </div>

              <div class="edit-exam-right">
                <div class="modal-percentage-input">
                  <input
                    v-model="editPercentages[item.id]"
                    type="number"
                    min="0.01"
                    max="100"
                    step="0.01"
                    class="form-control"
                  />

                  <span>%</span>
                </div>

                <button
                  type="button"
                  class="modal-delete-btn"
                  title="Remove Examination"
                  @click="deleteFromEdit(item)"
                >
                  <i class="bi bi-trash3"></i>
                </button>
              </div>
            </div>
          </div>

          <!-- Modal Total -->
          <div class="modal-total-box">
            <div>
              <span> Total Percentage </span>

              <strong :class="Math.abs(editGroupTotal - 100) < 0.001 ? 'valid' : 'invalid'">
                {{ editGroupTotal.toFixed(2) }}%
              </strong>
            </div>

            <div
              class="total-status"
              :class="Math.abs(editGroupTotal - 100) < 0.001 ? 'valid' : 'invalid'"
            >
              <i
                :class="
                  Math.abs(editGroupTotal - 100) < 0.001
                    ? 'bi bi-check-circle-fill'
                    : 'bi bi-exclamation-circle-fill'
                "
              ></i>

              {{ Math.abs(editGroupTotal - 100) < 0.001 ? 'Complete' : 'Must equal 100%' }}
            </div>
          </div>

          <!-- Error -->
          <div v-if="editError" class="alert-error">
            <i class="bi bi-exclamation-circle"></i>

            <span>
              {{ editError }}
            </span>
          </div>
        </div>

        <!-- Modal Footer -->
        <div class="modal-footer">
          <button class="btn-secondary" @click="closeEdit">Cancel</button>

          <button
            class="btn-primary modal-save"
            @click="updateFinalResultGroup"
            :disabled="updating || editingGroup.items.length === 0"
          >
            <span v-if="updating" class="spinner"></span>

            <i v-else class="bi bi-check-lg"></i>

            {{ updating ? 'Updating...' : 'Update Configuration' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import dashPageView from './dashPageView.vue'
import { useRouter } from 'vue-router'
import api from '@/services/api'

// =========================================================
// Router
// =========================================================

const router = useRouter()

// =========================================================
// State
// =========================================================

const examinations = ref([])
const finalResults = ref([])
const loading = ref(false)
const saving = ref(false)
const updating = ref(false)

const errorMessage = ref('')

const editingGroup = ref(null)
const editPercentages = reactive({})
const editError = ref('')

const activeYear = ref(null)

const form = reactive({
  examination_id: '',
  percentage: '',
})

// =========================================================
// Navigate To Final Result PDF
// =========================================================

const goToFinalResultPdf = () => {
  const latestGroup = groupedFinalResults.value[0]

  if (!latestGroup) {
    return
  }

  goToYearPdf(latestGroup.year)
}

// =========================================================
// Navigate To Specific Year Final Result PDF
// =========================================================

const goToYearPdf = (year) => {
  router.push({
    path: '/finalAvragePdfView',
    query: {
      year: String(year),
    },
  })
}

// =========================================================
// Group Final Results By Year
// =========================================================

const groupedFinalResults = computed(() => {
  const groups = {}

  finalResults.value.forEach((item) => {
    const year = item.examination?.examination_year || 'N/A'

    if (!groups[year]) {
      groups[year] = {
        year,
        items: [],
        totalPercentage: 0,
      }
    }

    groups[year].items.push(item)

    groups[year].totalPercentage += Number(item.percentage || 0)
  })

  return Object.values(groups).sort((a, b) => {
    if (a.year === 'N/A') return 1
    if (b.year === 'N/A') return -1

    return Number(b.year) - Number(a.year)
  })
})

// =========================================================
// Selected Examination Year
// =========================================================

const selectedExamYear = computed(() => {
  if (form.examination_id) {
    const selectedExam = examinations.value.find(
      (exam) => Number(exam.id) === Number(form.examination_id),
    )

    if (selectedExam?.examination_year) {
      return selectedExam.examination_year
    }
  }

  return activeYear.value
})

// =========================================================
// Current Year
// =========================================================

const currentYear = computed(() => {
  if (selectedExamYear.value) {
    return selectedExamYear.value
  }

  return groupedFinalResults.value[0]?.year || null
})

// =========================================================
// Current Year Total Percentage
// =========================================================

const totalPercentage = computed(() => {
  if (!currentYear.value) {
    return 0
  }

  const group = groupedFinalResults.value.find(
    (group) => String(group.year) === String(currentYear.value),
  )

  return group ? Number(group.totalPercentage || 0) : 0
})

// =========================================================
// Current Year Remaining Percentage
// =========================================================

const remainingPercentage = computed(() => {
  return Math.max(0, 100 - totalPercentage.value)
})

// =========================================================
// Current Year Completion
// =========================================================

const isComplete = computed(() => {
  return Math.abs(totalPercentage.value - 100) < 0.001
})

// =========================================================
// Current Year Examination Count
// =========================================================

const selectedYearExaminationCount = computed(() => {
  if (!currentYear.value) {
    return 0
  }

  return finalResults.value.filter(
    (item) => String(item.examination?.examination_year) === String(currentYear.value),
  ).length
})

// =========================================================
// Available Examinations
// =========================================================

const availableExaminations = computed(() => {
  const addedIds = finalResults.value.map((item) => Number(item.examination_id))

  return examinations.value.filter((exam) => !addedIds.includes(Number(exam.id)))
})

// =========================================================
// Edit Group Total
// =========================================================

const editGroupTotal = computed(() => {
  if (!editingGroup.value) {
    return 0
  }

  return editingGroup.value.items.reduce((total, item) => {
    return total + Number(editPercentages[item.id] || 0)
  }, 0)
})

// =========================================================
// Edit Maximum
// =========================================================

const editMaximum = computed(() => {
  if (!editingGroup.value) {
    return 100
  }

  return editingGroup.value.items.reduce((total, item) => {
    return total + Number(editPercentages[item.id] || 0)
  }, 0)
})

// =========================================================
// Load Examinations
// =========================================================

const loadExaminations = async () => {
  try {
    const response = await api.get('/examinations')

    examinations.value = response.data?.data || response.data || []
  } catch (error) {
    console.error('Failed to load examinations:', error)
  }
}

// =========================================================
// Load Final Result Configuration
// =========================================================

const loadFinalResults = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    const response = await api.get('/final-results')

    finalResults.value = response.data?.data || []
  } catch (error) {
    console.error('Failed to load final results:', error)

    errorMessage.value =
      error.response?.data?.message || 'Failed to load final result configuration.'
  } finally {
    loading.value = false
  }
}

// =========================================================
// Load All Data
// =========================================================

const loadData = async () => {
  await Promise.all([loadExaminations(), loadFinalResults()])
}

// =========================================================
// Add Final Result Examination
// =========================================================

const addFinalResult = async () => {
  errorMessage.value = ''

  if (!form.examination_id) {
    errorMessage.value = 'Please select an examination.'
    return
  }

  const percentage = Number(form.percentage)

  if (!percentage || percentage <= 0) {
    errorMessage.value = 'Please enter a valid percentage.'
    return
  }

  // -------------------------------------------------------
  // Find Selected Examination
  // -------------------------------------------------------

  const selectedExam = examinations.value.find(
    (exam) => Number(exam.id) === Number(form.examination_id),
  )

  if (!selectedExam) {
    errorMessage.value = 'The selected examination could not be found.'
    return
  }

  const year = selectedExam.examination_year

  // -------------------------------------------------------
  // Calculate Percentage Only For Selected Year
  // -------------------------------------------------------

  const yearTotal = finalResults.value
    .filter((item) => String(item.examination?.examination_year) === String(year))
    .reduce((total, item) => total + Number(item.percentage || 0), 0)

  const yearRemaining = Math.max(0, 100 - yearTotal)

  // -------------------------------------------------------
  // Check Selected Year's Remaining Percentage
  // -------------------------------------------------------

  if (percentage > yearRemaining) {
    errorMessage.value = `Only ${yearRemaining.toFixed(2)}% is remaining for ${year}.`

    return
  }

  saving.value = true

  try {
    const response = await api.post('/final-results', {
      examination_id: Number(form.examination_id),

      percentage: percentage,
    })

    // -----------------------------------------------------
    // Set Active Year
    // -----------------------------------------------------

    activeYear.value = year

    // -----------------------------------------------------
    // Update Local Data
    // -----------------------------------------------------

    if (response.data?.data) {
      finalResults.value.push(response.data.data)
    } else {
      await loadFinalResults()
    }

    // -----------------------------------------------------
    // Reset Form
    // -----------------------------------------------------

    form.examination_id = ''
    form.percentage = ''
  } catch (error) {
    console.error('Failed to add final result:', error)

    errorMessage.value =
      error.response?.data?.message || 'Failed to add examination to final result configuration.'
  } finally {
    saving.value = false
  }
}

// =========================================================
// Open Edit Group
// =========================================================

const openEditGroup = (group) => {
  editingGroup.value = group

  editError.value = ''

  Object.keys(editPercentages).forEach((key) => delete editPercentages[key])

  group.items.forEach((item) => {
    editPercentages[item.id] = Number(item.percentage)
  })
}

// =========================================================
// Close Edit
// =========================================================

const closeEdit = () => {
  editingGroup.value = null

  editError.value = ''

  Object.keys(editPercentages).forEach((key) => delete editPercentages[key])
}

// =========================================================
// Update Final Result Group
// =========================================================

const updateFinalResultGroup = async () => {
  editError.value = ''

  if (!editingGroup.value) {
    return
  }

  const total = Number(editGroupTotal.value.toFixed(2))

  // -------------------------------------------------------
  // Total Must Be Exactly 100%
  // -------------------------------------------------------

  if (Math.abs(total - 100) > 0.001) {
    editError.value = `Total percentage must be exactly 100%. Current total is ${total.toFixed(2)}%.`

    return
  }

  // -------------------------------------------------------
  // Validate Every Percentage
  // -------------------------------------------------------

  for (const item of editingGroup.value.items) {
    const percentage = Number(editPercentages[item.id])

    if (!percentage || percentage <= 0) {
      editError.value = `Please enter a valid percentage for ${
        item.examination?.examination_type || 'the examination'
      }.`

      return
    }

    if (percentage > 100) {
      editError.value = 'Percentage cannot be greater than 100%.'

      return
    }
  }

  updating.value = true

  try {
    // -----------------------------------------------------
    // Update Each Examination
    // -----------------------------------------------------

    for (const item of editingGroup.value.items) {
      const percentage = Number(editPercentages[item.id])

      await api.put(`/final-results/${item.id}`, {
        percentage,
      })
    }

    // -----------------------------------------------------
    // Reload Configuration
    // -----------------------------------------------------

    await loadFinalResults()

    closeEdit()
  } catch (error) {
    console.error('Failed to update final result:', error)

    editError.value =
      error.response?.data?.message || 'Failed to update final result configuration.'
  } finally {
    updating.value = false
  }
}

// =========================================================
// Delete From Edit Modal
// =========================================================

const deleteFromEdit = async (item) => {
  const examName = item.examination?.examination_type || 'this examination'

  const confirmed = window.confirm(
    `Are you sure you want to remove ${examName} from the final result configuration?`,
  )

  if (!confirmed) {
    return
  }

  editError.value = ''

  try {
    await api.delete(`/final-results/${item.id}`)

    // -----------------------------------------------------
    // Remove From Main List
    // -----------------------------------------------------

    finalResults.value = finalResults.value.filter((result) => result.id !== item.id)

    // -----------------------------------------------------
    // Remove From Edit Group
    // -----------------------------------------------------

    if (editingGroup.value) {
      editingGroup.value.items = editingGroup.value.items.filter((result) => result.id !== item.id)
    }

    delete editPercentages[item.id]

    // -----------------------------------------------------
    // Close Modal If No Item Remains
    // -----------------------------------------------------

    if (!editingGroup.value || editingGroup.value.items.length === 0) {
      closeEdit()
    }
  } catch (error) {
    console.error('Failed to delete final result:', error)

    editError.value = error.response?.data?.message || 'Failed to remove examination.'
  }
}

// =========================================================
// Initial Load
// =========================================================

onMounted(() => {
  loadData()
})
</script>
<style scoped>
/* =========================================================
   Page
========================================================= */

.final-result-page {
  min-height: 100vh;
  background: #f5f7fb;
  display: flex;
}

/* =========================================================
   Sidebar Space
========================================================= */

.sidebar-space {
  width: 250px;
  flex-shrink: 0;
}

/* =========================================================
   Main Content
========================================================= */

.final-result-content {
  flex: 1;
  padding: 30px;
  min-width: 0;
}

/* =========================================================
   Header
========================================================= */

.page-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 25px;
}

.page-title {
  margin: 0;
  font-size: 28px;
  font-weight: 700;
  color: #172033;
}

.page-subtitle {
  margin: 6px 0 0;
  color: #7b8497;
  font-size: 14px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.status-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 9px 15px;
  border-radius: 30px;
  font-size: 13px;
  font-weight: 600;
}

.status-badge.complete {
  background: #e8f8ef;
  color: #16834b;
}

.status-badge.incomplete {
  background: #fff5df;
  color: #b87900;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: currentColor;
}

/* =========================================================
   Download PDF Button
========================================================= */

.download-pdf-btn {
  height: 40px;
  padding: 0 15px;
  border: 0;
  border-radius: 9px;
  background: #dc4c4c;
  color: #fff;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;

  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;

  transition: 0.2s ease;
}

.download-pdf-btn:hover {
  background: #c83f3f;
  transform: translateY(-1px);
}

/* =========================================================
   Summary
========================================================= */

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 18px;
  margin-bottom: 22px;
}

.summary-card {
  background: #fff;
  border: 1px solid #e8ebf1;
  border-radius: 14px;
  padding: 20px;

  display: flex;
  align-items: center;
  gap: 15px;

  box-shadow: 0 3px 12px rgba(20, 31, 56, 0.04);
}

.summary-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 21px;
}

.summary-icon.blue {
  background: #eaf2ff;
  color: #3d76e8;
}

.summary-icon.green {
  background: #e8f8ef;
  color: #19945a;
}

.summary-icon.orange {
  background: #fff3df;
  color: #dc8b17;
}

.summary-label {
  font-size: 13px;
  color: #7d8799;
  margin-bottom: 4px;
}

.summary-value {
  font-size: 22px;
  font-weight: 700;
  color: #172033;
}

/* =========================================================
   Main Grid
========================================================= */

.main-grid {
  display: grid;

  grid-template-columns:
    minmax(0, 1.05fr)
    minmax(0, 0.95fr);

  gap: 20px;
  margin-bottom: 22px;
}

.card-box {
  background: #fff;
  border: 1px solid #e8ebf1;
  border-radius: 15px;
  padding: 23px;

  box-shadow: 0 3px 12px rgba(20, 31, 56, 0.04);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;

  gap: 15px;
  margin-bottom: 22px;
}

.card-header h5 {
  margin: 0;
  color: #172033;
  font-size: 17px;
  font-weight: 700;
}

.card-header p {
  margin: 5px 0 0;
  color: #8a92a3;
  font-size: 13px;
}

.header-icon {
  width: 38px;
  height: 38px;

  border-radius: 10px;

  background: #edf3ff;
  color: #3974e5;

  display: flex;
  align-items: center;
  justify-content: center;
}

/* =========================================================
   Form
========================================================= */

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;

  font-size: 13px;
  font-weight: 600;

  color: #354052;

  margin-bottom: 8px;
}

.form-group label span {
  color: #e34b4b;
}

.form-control {
  width: 100%;
  height: 45px;

  border: 1px solid #dfe4ec;
  border-radius: 9px;

  padding: 0 13px;

  background: #fff;
  color: #263247;

  font-size: 14px;
  outline: none;

  transition: 0.2s ease;
}

.form-control:focus {
  border-color: #5685e9;

  box-shadow: 0 0 0 3px rgba(86, 133, 233, 0.1);
}

.form-control:disabled {
  background: #f4f6f9;
  cursor: not-allowed;
}

.percentage-input {
  position: relative;
}

.percentage-input .form-control {
  padding-right: 45px;
}

.percentage-input > span {
  position: absolute;

  right: 14px;
  top: 50%;

  transform: translateY(-50%);

  color: #697386;

  font-weight: 600;
  font-size: 14px;
}

.input-hint {
  display: block;

  margin-top: 7px;

  color: #8992a3;
  font-size: 12px;
}

.input-hint strong {
  color: #4f6f9f;
}

.btn-primary {
  width: 100%;
  height: 45px;

  border: 0;
  border-radius: 9px;

  background: #3f75e5;
  color: #fff;

  font-size: 14px;
  font-weight: 600;

  cursor: pointer;

  display: flex;
  align-items: center;
  justify-content: center;

  gap: 8px;

  transition: 0.2s ease;
}

.btn-primary:hover:not(:disabled) {
  background: #3268d6;
  transform: translateY(-1px);
}

.btn-primary:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

/* =========================================================
   Error
========================================================= */

.alert-error {
  display: flex;
  align-items: flex-start;

  gap: 9px;

  padding: 11px 13px;

  border-radius: 8px;

  background: #fff0f0;
  border: 1px solid #ffd4d4;

  color: #c33d3d;

  font-size: 13px;

  margin-bottom: 16px;
}

/* =========================================================
   Progress
========================================================= */

.percentage-circle {
  min-width: 58px;
  height: 58px;

  border-radius: 50%;

  background: #edf3ff;
  color: #3e73df;

  display: flex;
  align-items: center;
  justify-content: center;

  font-weight: 700;
  font-size: 14px;
}

.percentage-circle.complete {
  background: #e7f8ee;
  color: #16834b;
}

.progress-wrapper {
  margin-top: 12px;
}

.progress-track {
  height: 13px;
  width: 100%;

  background: #edf0f5;

  border-radius: 20px;

  overflow: hidden;
}

.progress-fill {
  height: 100%;

  border-radius: 20px;

  background: #4d7fe5;

  transition: width 0.35s ease;
}

.progress-info {
  display: flex;
  justify-content: space-between;

  margin-top: 8px;

  color: #9aa2b1;
  font-size: 12px;
}

.progress-info strong {
  color: #596477;
}

.completion-message {
  display: flex;

  gap: 12px;

  padding: 14px;
  margin-top: 25px;

  border-radius: 10px;
}

.completion-message.success {
  background: #ebf9f0;
  color: #18814c;
}

.completion-message.warning {
  background: #fff7e7;
  color: #b47a12;
}

.completion-message > i {
  font-size: 18px;
  margin-top: 1px;
}

.completion-message strong {
  font-size: 13px;
}

.completion-message p {
  margin: 4px 0 0;

  font-size: 12px;
  line-height: 1.5;
}

/* =========================================================
   Table
========================================================= */

.table-card {
  padding: 0;
  overflow: hidden;
}

.table-card .card-header {
  padding: 22px 23px 18px;

  margin: 0;

  border-bottom: 1px solid #edf0f4;
}

.refresh-btn {
  border: 1px solid #dfe4ec;

  background: #fff;
  color: #5f6b7d;

  height: 37px;

  padding: 0 13px;

  border-radius: 8px;

  font-size: 13px;

  display: flex;
  align-items: center;

  gap: 7px;

  cursor: pointer;
}

.refresh-btn:hover {
  background: #f6f8fb;
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.rotating {
  animation: rotate 0.8s linear infinite;
}

.table-wrapper {
  width: 100%;
  overflow-x: auto;
}

.result-table {
  width: 100%;

  border-collapse: collapse;

  min-width: 800px;
}

.result-table th {
  background: #f8f9fb;

  color: #7d8798;

  font-size: 12px;
  font-weight: 600;

  text-align: left;

  padding: 14px 18px;

  border-bottom: 1px solid #e9edf2;

  white-space: nowrap;
}

.result-table td {
  padding: 17px 18px;

  border-bottom: 1px solid #edf0f4;

  color: #485366;

  font-size: 13px;

  vertical-align: middle;
}

.result-table tbody tr:last-child td {
  border-bottom: 0;
}

.result-table tbody tr:hover {
  background: #fafbfc;
}

.text-end {
  text-align: right !important;
}

/* =========================================================
   Serial
========================================================= */

.serial {
  width: 31px;
  height: 31px;

  border-radius: 8px;

  background: #f0f3f8;

  display: inline-flex;

  align-items: center;
  justify-content: center;

  color: #657186;

  font-size: 12px;
  font-weight: 600;
}

/* =========================================================
   Grouped Examination
========================================================= */

.grouped-examinations {
  display: flex;
  align-items: center;
  gap: 12px;
}

.grouped-exam-icon {
  width: 40px;
  height: 40px;

  flex-shrink: 0;

  border-radius: 10px;

  background: #edf3ff;
  color: #3f76e5;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 16px;
}

.exam-list-content {
  min-width: 0;
}

.exam-names {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 5px;

  line-height: 1.7;
}

.exam-name-badge {
  color: #263247;

  font-size: 13px;
  font-weight: 650;
}

.plus-separator {
  color: #9aa3b2;

  font-weight: 500;

  margin-left: 4px;
}

.exam-list-content small {
  display: block;

  margin-top: 3px;

  color: #929baa;

  font-size: 11px;
}

/* =========================================================
   Year
========================================================= */

.year-badge {
  padding: 6px 10px;

  border-radius: 7px;

  background: #f2f4f8;
  color: #606b7d;

  font-size: 12px;
  font-weight: 650;
}

/* =========================================================
   Percentage Group
========================================================= */

.percentage-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

.percentage-list {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}

.percentage-item {
  padding: 5px 8px;

  border-radius: 6px;

  background: #f3f6fb;

  color: #667286;

  font-size: 11px;
  font-weight: 600;

  white-space: nowrap;
}

.percentage-total {
  display: inline-flex;
  align-items: center;
  justify-content: center;

  min-width: 62px;

  padding: 6px 9px;

  border-radius: 7px;

  background: #eaf2ff;
  color: #3b71dc;

  font-weight: 750;
  font-size: 12px;

  white-space: nowrap;
}

/* =========================================================
   Action
========================================================= */

.action-buttons {
  display: inline-flex;
  align-items: center;
  justify-content: flex-end;
  gap: 6px;
  white-space: nowrap;
  position: relative;
  z-index: 1;
}

.action-btn {
  width: 34px;
  height: 34px;
  min-width: 34px;
  flex-shrink: 0;
  border: 0;
  border-radius: 7px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: 0.2s ease;
  position: relative;
}

.action-btn {
  width: 34px;
  height: 34px;

  border: 0;
  border-radius: 7px;

  cursor: pointer;

  display: inline-flex;

  align-items: center;
  justify-content: center;

  transition: 0.2s ease;
}

.action-btn.edit {
  background: #edf3ff;
  color: #3f76df;
}

.action-btn.edit:hover {
  background: #dfeaff;
}

.action-btn.pdf {
  background: #fff0f0;
  color: #dc4c4c;
}

.action-btn.pdf:hover {
  background: #ffe1e1;
}

/* =========================================================
   Loading / Empty
========================================================= */

.loading-state,
.empty-state {
  min-height: 230px;

  display: flex;

  flex-direction: column;

  align-items: center;
  justify-content: center;

  padding: 30px;

  text-align: center;
}

.loading-state p,
.empty-state p {
  color: #8992a2;

  font-size: 13px;

  margin: 12px 0 0;
}

.empty-icon {
  width: 58px;
  height: 58px;

  border-radius: 50%;

  background: #edf3ff;
  color: #4a7ce0;

  display: flex;

  align-items: center;
  justify-content: center;

  font-size: 23px;
}

.empty-state h6 {
  margin: 14px 0 0;

  color: #354052;

  font-size: 15px;
}

.empty-state p {
  max-width: 400px;

  line-height: 1.6;
}

/* =========================================================
   Spinner
========================================================= */

.spinner {
  width: 15px;
  height: 15px;

  border: 2px solid rgba(255, 255, 255, 0.45);

  border-top-color: #fff;

  border-radius: 50%;

  animation: spin 0.7s linear infinite;
}

.large-spinner {
  width: 31px;
  height: 31px;

  border: 3px solid #e7ebf2;

  border-top-color: #477ae2;

  border-radius: 50%;

  animation: spin 0.7s linear infinite;
}

/* =========================================================
   Edit Modal
========================================================= */

.modal-overlay {
  position: fixed;

  inset: 0;

  background: rgba(19, 28, 45, 0.48);

  display: flex;

  align-items: center;
  justify-content: center;

  padding: 20px;

  z-index: 9999;
}

.edit-modal {
  width: 100%;
  max-width: 650px;

  max-height: 90vh;

  background: #fff;

  border-radius: 15px;

  box-shadow: 0 18px 60px rgba(18, 30, 52, 0.2);

  overflow: hidden;

  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;

  align-items: flex-start;

  justify-content: space-between;

  padding: 21px 22px;

  border-bottom: 1px solid #edf0f4;

  flex-shrink: 0;
}

.modal-header h5 {
  margin: 0;

  color: #202b3d;

  font-size: 17px;
}

.modal-header p {
  margin: 5px 0 0;

  color: #8992a2;

  font-size: 12px;
}

.modal-close {
  width: 32px;
  height: 32px;

  border: 0;

  border-radius: 7px;

  background: #f2f4f7;

  color: #6d7685;

  cursor: pointer;
}

.modal-body {
  padding: 22px;

  overflow-y: auto;
}

/* =========================================================
   Modal Year
========================================================= */

.modal-year-header {
  display: flex;

  align-items: center;

  gap: 12px;

  padding: 13px;

  background: #f7f9fc;

  border: 1px solid #edf0f4;

  border-radius: 10px;

  margin-bottom: 17px;
}

.modal-year-icon {
  width: 42px;
  height: 42px;

  border-radius: 9px;

  background: #edf3ff;
  color: #3f76e5;

  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-year-header strong {
  display: block;

  color: #273246;

  font-size: 15px;
}

.modal-year-header span {
  display: block;

  color: #8b94a4;

  font-size: 11px;

  margin-top: 3px;
}

/* =========================================================
   Edit Exam List
========================================================= */

.edit-exam-list {
  display: flex;
  flex-direction: column;

  gap: 9px;
}

.edit-exam-row {
  display: flex;

  align-items: center;
  justify-content: space-between;

  gap: 15px;

  padding: 12px;

  border: 1px solid #edf0f4;

  border-radius: 10px;

  background: #fff;

  transition: 0.2s ease;
}

.edit-exam-row:hover {
  border-color: #dfe6f0;
  background: #fafbfc;
}

.edit-exam-left {
  display: flex;

  align-items: center;

  gap: 11px;

  min-width: 0;
}

.exam-icon.large {
  width: 42px;
  height: 42px;

  border-radius: 9px;

  background: #edf3ff;
  color: #3f76e5;

  display: flex;
  align-items: center;
  justify-content: center;

  flex-shrink: 0;
}

.edit-exam-left strong {
  display: block;

  color: #273246;

  font-size: 13px;
}

.edit-exam-left span {
  display: block;

  color: #8b94a4;

  font-size: 11px;

  margin-top: 3px;
}

.edit-exam-right {
  display: flex;

  align-items: center;

  gap: 7px;

  flex-shrink: 0;
}

.modal-percentage-input {
  position: relative;

  width: 105px;
}

.modal-percentage-input .form-control {
  height: 40px;

  padding-right: 34px;

  font-size: 13px;
}

.modal-percentage-input > span {
  position: absolute;

  right: 11px;
  top: 50%;

  transform: translateY(-50%);

  color: #697386;

  font-size: 12px;
  font-weight: 600;
}

.modal-delete-btn {
  width: 36px;
  height: 36px;

  border: 0;

  border-radius: 7px;

  background: #fff0f0;

  color: #dc5050;

  cursor: pointer;

  display: flex;

  align-items: center;
  justify-content: center;

  transition: 0.2s ease;
}

.modal-delete-btn:hover {
  background: #ffe1e1;
}

/* =========================================================
   Modal Total
========================================================= */

.modal-total-box {
  display: flex;

  align-items: center;
  justify-content: space-between;

  gap: 15px;

  padding: 14px;

  margin-top: 17px;

  border-radius: 9px;

  background: #f7f9fc;

  border: 1px solid #e8edf3;
}

.modal-total-box span {
  display: block;

  color: #8992a2;

  font-size: 11px;

  margin-bottom: 3px;
}

.modal-total-box strong {
  font-size: 18px;
}

.modal-total-box strong.valid {
  color: #18814c;
}

.modal-total-box strong.invalid {
  color: #c33d3d;
}

.total-status {
  display: inline-flex;

  align-items: center;

  gap: 5px;

  font-size: 11px;
  font-weight: 650;
}

.total-status.valid {
  color: #18814c;
}

.total-status.invalid {
  color: #c33d3d;
}

/* =========================================================
   Modal Footer
========================================================= */

.modal-footer {
  padding: 16px 22px;

  border-top: 1px solid #edf0f4;

  display: flex;

  justify-content: flex-end;

  gap: 10px;

  flex-shrink: 0;
}

.btn-secondary {
  height: 42px;

  padding: 0 17px;

  border: 1px solid #dfe4ec;

  border-radius: 8px;

  background: #fff;

  color: #606b7c;

  font-size: 13px;
  font-weight: 600;

  cursor: pointer;
}

.btn-secondary:hover {
  background: #f7f8fa;
}

.modal-save {
  width: auto;

  padding: 0 18px;

  height: 42px;
}

/* =========================================================
   Animations
========================================================= */

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes rotate {
  to {
    transform: rotate(360deg);
  }
}

/* =========================================================
   Responsive
========================================================= */

@media (max-width: 1100px) {
  .sidebar-space {
    width: 220px;
  }

  .final-result-content {
    padding: 22px;
  }

  .summary-grid {
    grid-template-columns: 1fr;
  }

  .main-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .final-result-page {
    display: block;
  }

  .sidebar-space {
    width: 0;
    display: none;
  }

  .final-result-content {
    padding: 16px;
  }

  .page-header {
    align-items: flex-start;
    flex-direction: column;
  }

  .page-title {
    font-size: 23px;
  }

  .header-actions {
    width: 100%;

    align-items: flex-start;

    flex-direction: column;
  }

  .status-badge {
    align-self: flex-start;
  }

  .download-pdf-btn {
    width: 100%;
  }

  .summary-grid {
    gap: 12px;
  }

  .card-box {
    padding: 17px;
  }

  .table-card {
    padding: 0;
  }

  .table-card .card-header {
    padding: 17px;
  }

  .edit-exam-row {
    align-items: flex-start;
    flex-direction: column;
  }

  .edit-exam-right {
    width: 100%;
  }

  .modal-percentage-input {
    flex: 1;
  }

  .modal-delete-btn {
    flex-shrink: 0;
  }
}

@media (max-width: 480px) {
  .modal-footer {
    flex-direction: column-reverse;
  }

  .modal-footer .btn-secondary,
  .modal-footer .modal-save {
    width: 100%;
  }

  .modal-total-box {
    align-items: flex-start;
    flex-direction: column;
  }
}
.card-box i {
  position: absolute;
  right: 0px;
  top: 0px;
  font-size: 25px;
  opacity: 0.3;
}
</style>
