<template>
  <RouterView />

  <!-- Sidebar Component -->
  <dashPageView />

  <div class="dashboard-layout">
    <!-- Main Content Area -->
    <main class="main-content">
      <!-- TOPBAR -->
      <div class="topbar">
        <div class="topbar-left">
          <h3 class="page-title">Dashboard</h3>
          <p class="page-subtitle">Welcome back! Here's today's coaching overview.</p>
        </div>

        <div class="topbar-right">
          <div class="profile-info">
            <span class="admin-name">
              {{ currentUser.name || 'Administrator' }}
            </span>
            <small>
              {{ currentUser.role || 'System Admin' }}
            </small>
          </div>

          <img
            :src="currentUser.image || 'https://i.pravatar.cc/100'"
            class="profile-img"
            alt="Profile Image"
          />
        </div>
      </div>

      <!-- DASHBOARD CONTENT -->
      <div class="dashboard-body">
        <!-- STATS CARDS -->
        <div class="row g-4 mb-4">
          <!-- TOTAL STAFF -->
          <div class="col-12 col-sm-6 col-lg-3">
            <div class="card-box blue">
              <div class="card-icon">
                <i class="fa-solid fa-users"></i>
              </div>

              <div class="card-content">
                <h3>{{ totalStaff }}</h3>
                <p>Total Staff</p>
              </div>
            </div>
          </div>

          <!-- TOTAL STUDENTS -->
          <div class="col-12 col-sm-6 col-lg-3">
            <div class="card-box green">
              <div class="card-icon">
                <i class="fa-solid fa-user-graduate"></i>
              </div>

              <div class="card-content">
                <h3>{{ totalStudents }}</h3>
                <p>Total Students</p>
              </div>
            </div>
          </div>

          <!-- TOTAL TEACHERS -->
          <div class="col-12 col-sm-6 col-lg-3">
            <div class="card-box red">
              <div class="card-icon">
                <i class="fa-solid fa-chalkboard-user"></i>
              </div>

              <div class="card-content">
                <h3>{{ totalTeachers }}</h3>
                <p>Total Teachers</p>
              </div>
            </div>
          </div>
          <!-- TOTAL PAYMENTS -->
          <div class="col-12 col-sm-6 col-lg-3">
            <div class="card-box orange">
              <div class="card-icon">
                <i class="fa-solid fa-wallet"></i>
              </div>

              <div class="card-content">
                <h3>{{ totalPayments }}</h3>
                <p>Total Payments</p>
              </div>
            </div>
          </div>
        </div>

        <!-- CHARTS & PROFILE ROW -->
        <div class="row g-4 mb-4">
          <!-- CHART -->
          <div class="col-12 col-lg-8">
            <div class="chart-card">
              <div class="chart-header">
                <div>
                  <h5>Traffic Analytics</h5>
                  <p>Monthly collection overview</p>
                </div>

                <div class="chart-filter">
                  <i class="fa-solid fa-calendar-days"></i>
                  Yearly
                </div>
              </div>

              <div class="chart-wrapper">
                <canvas id="trafficChart"></canvas>
              </div>
            </div>
          </div>

          <!-- DYNAMIC PROFILE CARD -->
          <div class="col-12 col-lg-4">
            <div class="profile-card">
              <img
                :src="currentUser.image || 'https://i.pravatar.cc/150?img=12'"
                alt="Profile Image"
              />

              <h4 class="mt-3">
                {{ currentUser.name || 'Jim Doe' }}
              </h4>

              <p class="text-muted">
                {{ currentUser.designation || currentUser.role || 'Project Manager' }}
              </p>

              <div class="row mt-4">
                <q>
                  "Empowering institutions through seamless administration and digital
                  transformation."
                </q>
              </div>
            </div>
          </div>
        </div>

        <!-- RECENT STAFF TABLE -->
        <div class="row">
          <div class="col-12">
            <div class="table-card">
              <div class="table-header">
                <div>
                  <h5>Recent Staff</h5>
                  <p>Latest registered staff members</p>
                </div>

                <div class="staff-count">
                  <i class="fa-solid fa-users"></i>
                  {{ recentStaff.length }}
                </div>
              </div>

              <div class="table-responsive">
                <table class="table custom-table">
                  <thead>
                    <tr>
                      <th>#</th>
                      <th>Staff</th>
                      <th>Email</th>
                      <th>Role</th>
                      <th>Status</th>
                    </tr>
                  </thead>

                  <tbody>
                    <tr v-for="(staff, index) in recentStaff.slice(0, 5)" :key="staff.id">
                      <td>{{ index + 1 }}</td>

                      <td>
                        <div class="staff-profile">
                          <span>{{ staff.name }}</span>
                        </div>
                      </td>

                      <td>{{ staff.email }}</td>

                      <td>
                        <span class="role-badge">
                          {{ staff.role }}
                        </span>
                      </td>

                      <td>
                        <span class="status-badge">
                          <i class="fa-solid fa-circle"></i>
                          Registered
                        </span>
                      </td>
                    </tr>

                    <tr v-if="recentStaff.length === 0">
                      <td colspan="5" class="text-center">No staff found</td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Chart from 'chart.js/auto'
import api from '@/services/api'
import dashPageView from './dashPageView.vue'

// ===============================
// State Variables
// ===============================

const totalStaff = ref(0)
const totalStudents = ref(0)
const totalPayments = ref(0)
const totalTeachers = ref(0)

const recentStaff = ref([])
const monthlyCollection = ref([])

// ===============================
// Current Logged-in User Info
// ===============================

const currentUser = ref({
  name: '',
  role: '',
  designation: '',
  image: '',
  tweets: 0,
  following: 0,
  followers: 0,
})

let chart = null

// ===============================
// Get Dashboard Data
// ===============================

const getDashboardData = async () => {
  try {
    // LocalStorage
    const storedUser = localStorage.getItem('user')

    if (storedUser) {
      const parsedUser = JSON.parse(storedUser)

      currentUser.value = {
        ...currentUser.value,
        ...parsedUser,
      }
    }

    // API
    const response = await api.get('/staff/dashboard')

    totalStaff.value = response.data.total_staff
    totalStudents.value = response.data.total_students
    totalPayments.value = response.data.total_payments
    totalTeachers.value = response.data.total_teachers

    recentStaff.value = response.data.recent_staff
    monthlyCollection.value = response.data.monthly_collection

    // User data from API
    if (response.data.user) {
      currentUser.value = {
        name: response.data.user.name,
        role: response.data.user.role || 'Admin',
        designation: response.data.user.designation || response.data.user.role,
        image: response.data.user.image || response.data.user.avatar,
        tweets: response.data.user.tweets || 750,
        following: response.data.user.following || 865,
        followers: response.data.user.followers || 3645,
      }
    }
  } catch (error) {
    console.error('Error fetching dashboard data:', error)
  }
}

// ===============================
// Create Chart
// ===============================

const createChart = () => {
  const months = Array(12).fill(0)

  if (monthlyCollection.value && monthlyCollection.value.length > 0) {
    monthlyCollection.value.forEach((item) => {
      months[item.month - 1] = Number(item.total)
    })
  }

  const ctx = document.getElementById('trafficChart')

  if (!ctx) {
    return
  }

  if (chart) {
    chart.destroy()
  }

  chart = new Chart(ctx, {
    type: 'bar',

    data: {
      labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],

      datasets: [
        {
          label: 'Collection',
          data: months,
          backgroundColor: '#2563eb',
          borderRadius: 10,
          barThickness: 18,
          hoverBackgroundColor: '#1d4ed8',
        },
      ],
    },

    options: {
      responsive: true,
      maintainAspectRatio: false,

      plugins: {
        legend: {
          display: false,
        },

        tooltip: {
          backgroundColor: '#111827',
          padding: 12,

          titleFont: {
            size: 14,
          },

          bodyFont: {
            size: 13,
          },

          callbacks: {
            label: function (context) {
              return ' ৳ ' + context.raw.toLocaleString()
            },
          },
        },
      },

      scales: {
        y: {
          beginAtZero: true,

          grid: {
            color: '#f1f5f9',
          },

          ticks: {
            callback: function (value) {
              return '৳ ' + value
            },
          },
        },

        x: {
          grid: {
            display: false,
          },
        },
      },
    },
  })
}

// ===============================
// Mounted
// ===============================

onMounted(async () => {
  await getDashboardData()
  createChart()
})
</script>

<style scoped>
/* ===============================
   Base Layout
================================ */

.dashboard-layout {
  display: flex;
  width: 100%;
  min-height: 100vh;
  background-color: #f8fafc;
}

.main-content {
  flex: 1;
  margin-left: 250px;
  padding: 24px;
  transition: all 0.3s ease;
  width: calc(100% - 250px);
}

/* ===============================
   Card Box
================================ */

.card-box {
  background: #ffffff;
  border-radius: 18px;
  padding: 25px 20px;
  box-shadow: 0 8px 20px rgba(15, 23, 42, 0.08);
  border: 1px solid #f1f5f9;
  transition: all 0.3s ease;
  text-align: center;
}

.card-box:hover {
  transform: translateY(-4px);
  box-shadow: 0 16px 30px rgba(15, 23, 42, 0.12);
}

.card-box i {
  font-size: 38px;
  margin-bottom: 15px;
}

.card-box h3 {
  font-size: 32px;
  font-weight: 700;
  color: #111827;
  margin-bottom: 8px;
}

.card-box p {
  color: #6b7280;
  font-size: 15px;
  font-weight: 500;
  margin: 0;
}

/* ===============================
   Icon Colors
================================ */

.blue i {
  color: #2563eb;
}

.green i {
  color: #16a34a;
}

.orange i {
  color: #f59e0b;
}

.red i {
  color: #ef4444;
}

/* ===============================
   Topbar
================================ */

.topbar {
  background: #ffffff;
  border-radius: 16px;
  padding: 18px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  box-shadow: 0 8px 20px rgba(15, 23, 42, 0.08);
}

.page-title {
  margin: 0;
  font-size: 26px;
  font-weight: 700;
  color: #111827;
}

.page-subtitle {
  margin-top: 4px;
  margin-bottom: 0;
  color: #6b7280;
  font-size: 14px;
}

.topbar-right {
  display: flex;
  align-items: center;
  gap: 15px;
}

.profile-info {
  text-align: right;
}

.admin-name {
  display: block;
  font-weight: 600;
  color: #111827;
}

.profile-info small {
  color: #6b7280;
}

.profile-img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: 3px solid #2563eb;
  object-fit: cover;
}

/* ===============================
   Chart Card
================================ */

.chart-card {
  background: #ffffff;
  border-radius: 20px;
  padding: 25px;
  height: 420px;
  border: 1px solid #f1f5f9;
  box-shadow: 0 8px 20px rgba(15, 23, 42, 0.08);
  transition: all 0.35s ease;
}

.chart-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 18px 40px rgba(37, 99, 235, 0.15);
  border-color: #dbeafe;
}

.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chart-card h5 {
  font-size: 20px;
  font-weight: 700;
  color: #111827;
  margin: 0;
}

.chart-header p {
  color: #6b7280;
  font-size: 14px;
  margin-top: 5px;
  margin-bottom: 0;
}

.chart-filter {
  background: #eff6ff;
  color: #2563eb;
  padding: 8px 14px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 600;
  transition: 0.3s ease;
}

.chart-wrapper {
  height: 300px;
  position: relative;
}

/* ===============================
   Profile Card
================================ */

.profile-card {
  background: #ffffff;
  border-radius: 20px;
  padding: 30px 20px;
  text-align: center;
  height: 420px;
  border: 1px solid #f1f5f9;
  box-shadow: 0 8px 20px rgba(15, 23, 42, 0.08);
  transition: all 0.35s ease;
}

.profile-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 18px 35px rgba(37, 99, 235, 0.18);
  border-color: #dbeafe;
}

.profile-card img {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  object-fit: cover;
  border: 4px solid #eff6ff;
  transition: 0.3s ease;
}

.profile-card h4 {
  font-weight: 700;
  color: #111827;
  margin-bottom: 4px;
}

.profile-card p {
  font-size: 14px;
}

.profile-card .row {
  background: #f8fafc;
  border-radius: 15px;
  padding: 15px 5px;
  margin-left: 0;
  margin-right: 0;
}

.profile-card h5 {
  font-size: 20px;
  font-weight: 700;
  color: #111827;
  margin-bottom: 3px;
}

.profile-card small {
  color: #6b7280;
  font-size: 13px;
}

/* ===============================
   Table Card
================================ */

.table-card {
  background: #ffffff;
  border-radius: 20px;
  padding: 25px;
  border: 1px solid #f1f5f9;
  box-shadow: 0 8px 20px rgba(15, 23, 42, 0.08);
  transition: 0.3s ease;
}

.table-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.table-header h5 {
  margin: 0;
  font-size: 20px;
  font-weight: 700;
  color: #111827;
}

.table-header p {
  margin: 5px 0 0;
  font-size: 14px;
  color: #6b7280;
}

.staff-count {
  background: #eff6ff;
  color: #2563eb;
  padding: 8px 15px;
  border-radius: 12px;
  font-weight: 600;
  font-size: 14px;
}

.custom-table {
  margin: 0;
}

.custom-table thead th {
  background: #f8fafc;
  color: #64748b;
  font-size: 13px;
  font-weight: 600;
  border: none;
  padding: 14px;
}

.custom-table tbody td {
  padding: 16px 14px;
  vertical-align: middle;
  color: #374151;
  border-bottom: 1px solid #f1f5f9;
}

.staff-profile {
  display: flex;
  align-items: center;
  gap: 12px;
}

.staff-profile img {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid #dbeafe;
}

.staff-profile span {
  font-weight: 600;
  color: #111827;
}

.role-badge {
  background: #ede9fe;
  color: #7c3aed;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
}

.status-badge {
  background: #dcfce7;
  color: #16a34a;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
}

.status-badge i {
  font-size: 8px;
  margin-right: 5px;
}

/* ===============================
   RESPONSIVE DESIGN
================================ */

@media (max-width: 991px) {
  .main-content {
    margin-left: 0;
    width: 100%;
    padding: 16px;
  }
}

@media (max-width: 576px) {
  .topbar {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }

  .topbar-right {
    width: 100%;
    justify-content: space-between;
  }
}
</style>
