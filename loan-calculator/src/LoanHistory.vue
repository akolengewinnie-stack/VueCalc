<template>
  <div class="loan-history">
    <!-- Search / Filter Bar -->
    <div class="history-toolbar">
      <div class="toolbar-left">
        <div class="search-wrapper">
          <span class="search-icon">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="16" height="16">
              <path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
            </svg>
          </span>
          <input
            v-model="searchQuery"
            type="text"
            class="search-input"
            placeholder="Search by External ID, status..."
          />
        </div>
        <div class="filter-group">
          <select v-model="statusFilter" class="filter-select">
            <option value="">All Statuses</option>
            <option value="approved">Approved</option>
            <option value="pending">Pending</option>
            <option value="rejected">Rejected</option>
            <option value="disbursed">Disbursed</option>
            <option value="completed">Completed</option>
          </select>
        </div>
      </div>
      <div class="toolbar-right">
        <button class="btn btn-outline btn-sm" @click="fetchHistory" :disabled="loading" type="button">
          <svg v-if="loading" class="spinner" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" width="14" height="14">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="14" height="14">
            <path d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/>
          </svg>
          Refresh
        </button>
        <div class="per-page-group">
          <label class="per-page-label">Show</label>
          <select v-model.number="perPage" class="filter-select per-page-select">
            <option :value="10">10</option>
            <option :value="25">25</option>
            <option :value="50">50</option>
            <option :value="100">100</option>
          </select>
          <label class="per-page-label">entries</label>
        </div>
      </div>
    </div>

    <!-- Error Banner -->
    <div v-if="error" class="error-banner">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
        <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
      </svg>
      <span>{{ error }}</span>
      <button class="error-dismiss" @click="error = null" type="button">×</button>
    </div>

    <!-- Loading Skeleton -->
    <div v-if="loading && loanRequests.length === 0" class="skeleton-wrapper">
      <div v-for="i in 5" :key="i" class="skeleton-row">
        <div class="skeleton-cell skeleton-sm"></div>
        <div class="skeleton-cell skeleton-md"></div>
        <div class="skeleton-cell skeleton-lg"></div>
        <div class="skeleton-cell skeleton-md"></div>
        <div class="skeleton-cell skeleton-sm"></div>
        <div class="skeleton-cell skeleton-sm"></div>
      </div>
    </div>

    <!-- Data Table -->
    <div v-else class="table-card">
      <div class="table-wrapper">
        <table class="data-table">
          <thead>
            <tr>
              <th class="th-sortable" @click="sortBy('external_id')">
                External ID
                <span class="sort-icon">{{ getSortIcon('external_id') }}</span>
              </th>
              <th class="th-sortable" @click="sortBy('loan_product')">
                Loan Product
                <span class="sort-icon">{{ getSortIcon('loan_product') }}</span>
              </th>
              <th class="th-right th-sortable" @click="sortBy('amount')">
                Amount
                <span class="sort-icon">{{ getSortIcon('amount') }}</span>
              </th>
              <th class="th-right th-sortable" @click="sortBy('duration')">
                Duration
                <span class="sort-icon">{{ getSortIcon('duration') }}</span>
              </th>
              <th class="th-sortable" @click="sortBy('created_at')">
                Date Applied
                <span class="sort-icon">{{ getSortIcon('created_at') }}</span>
              </th>
              <th class="th-center th-sortable" @click="sortBy('status')">
                Status
                <span class="sort-icon">{{ getSortIcon('status') }}</span>
              </th>
              <th class="th-center">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="paginatedRows.length === 0">
              <td colspan="7" class="empty-state">
                <div class="empty-content">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="40" height="40">
                    <path d="M20 6h-2.18c.07-.44.18-.88.18-1.36C18 2.06 15.94 0 13.36 0c-1.3 0-2.48.52-3.34 1.36L9 2.38 7.98 1.36C7.12.52 5.94 0 4.64 0 2.06 0 0 2.06 0 4.64c0 .48.11.92.18 1.36H0v14c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8l2 2V6h-2zm-6.64-4c.9 0 1.64.74 1.64 1.64 0 .9-.74 1.64-1.64 1.64-.9 0-1.64-.74-1.64-1.64C11.72 2.74 12.46 2 13.36 2zM4.64 2c.9 0 1.64.74 1.64 1.64 0 .9-.74 1.64-1.64 1.64C3.74 5.28 3 4.54 3 3.64 3 2.74 3.74 2 4.64 2zM18 20H2V8h16v12z"/>
                  </svg>
                  <p class="empty-title">No loan requests found</p>
                  <p class="empty-subtitle">
                    {{ searchQuery || statusFilter ? 'Try adjusting your search or filter.' : 'Enter an External ID above to look up loan requests.' }}
                  </p>
                </div>
              </td>
            </tr>
            <tr
              v-for="(row, index) in paginatedRows"
              :key="row.id || row.external_id || index"
              class="data-row"
              :class="{ 'row-selected': selectedRow === row }"
              @click="selectRow(row)"
            >
              <td class="td-id">
                <span class="id-badge">{{ row.external_id || row.id || '—' }}</span>
              </td>
              <td class="td-product">
                <div class="product-cell">
                  <span class="product-name">{{ row.loan_product || row.product_name || row.product || '—' }}</span>
                  <span v-if="row.interest_rate" class="product-rate">{{ formatRate(row.interest_rate) }}% p.m.</span>
                </div>
              </td>
              <td class="td-right td-amount">{{ formatCurrency(row.amount || row.loan_amount) }}</td>
              <td class="td-right td-duration">
                {{ row.duration || row.loan_period || row.period || '—' }}
                <span v-if="row.duration || row.loan_period" class="duration-unit">mo</span>
              </td>
              <td class="td-date">
                <div class="date-cell" v-if="row.created_at || row.date_applied || row.application_date">
                  <span class="date-day">{{ formatDateDay(row.created_at || row.date_applied || row.application_date) }}</span>
                  <span class="date-month-year">{{ formatDateMonthYear(row.created_at || row.date_applied || row.application_date) }}</span>
                </div>
                <span v-else>—</span>
              </td>
              <td class="td-center">
                <span class="status-badge" :class="getStatusClass(row.status)">
                  {{ formatStatus(row.status) }}
                </span>
              </td>
              <td class="td-center td-actions">
                <button
                  class="btn-icon"
                  title="View Details"
                  @click.stop="viewDetails(row)"
                  type="button"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="16" height="16">
                    <path d="M12 4.5C7 4.5 2.73 7.61 1 12c1.73 4.39 6 7.5 11 7.5s9.27-3.11 11-7.5c-1.73-4.39-6-7.5-11-7.5zM12 17c-2.76 0-5-2.24-5-5s2.24-5 5-5 5 2.24 5 5-2.24 5-5 5zm0-8c-1.66 0-3 1.34-3 3s1.34 3 3 3 3-1.34 3-3-1.34-3-3-3z"/>
                  </svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pagination -->
      <div class="pagination-bar" v-if="filteredRows.length > 0">
        <div class="pagination-info">
          Showing {{ paginationStart }}–{{ paginationEnd }} of {{ filteredRows.length }} entries
          <span v-if="searchQuery || statusFilter" class="filter-note">(filtered from {{ loanRequests.length }} total)</span>
        </div>
        <div class="pagination-controls">
          <button
            class="page-btn"
            :disabled="currentPage === 1"
            @click="currentPage = 1"
            type="button"
            title="First page"
          >«</button>
          <button
            class="page-btn"
            :disabled="currentPage === 1"
            @click="currentPage--"
            type="button"
            title="Previous page"
          >‹</button>
          <button
            v-for="page in visiblePages"
            :key="page"
            class="page-btn"
            :class="{ active: page === currentPage, ellipsis: page === '...' }"
            :disabled="page === '...'"
            @click="page !== '...' && (currentPage = page)"
            type="button"
          >{{ page }}</button>
          <button
            class="page-btn"
            :disabled="currentPage === totalPages"
            @click="currentPage++"
            type="button"
            title="Next page"
          >›</button>
          <button
            class="page-btn"
            :disabled="currentPage === totalPages"
            @click="currentPage = totalPages"
            type="button"
            title="Last page"
          >»</button>
        </div>
      </div>
    </div>

    <!-- Detail Modal -->
    <transition name="modal-fade">
      <div v-if="detailRow" class="modal-overlay" @click.self="detailRow = null">
        <div class="modal-box">
          <div class="modal-header">
            <h3 class="modal-title">Loan Request Details</h3>
            <button class="modal-close" @click="detailRow = null" type="button">
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="20" height="20">
                <path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>
              </svg>
            </button>
          </div>
          <div class="modal-body">
            <div class="detail-grid">
              <div class="detail-item" v-for="(value, key) in flattenedDetail" :key="key">
                <span class="detail-label">{{ formatKey(key) }}</span>
                <span class="detail-value">{{ formatDetailValue(key, value) }}</span>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button class="btn btn-outline" @click="detailRow = null" type="button">Close</button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'LoanHistory',
  props: {
    externalId: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      loanRequests: [],
      loading: false,
      error: null,

      // Search & filter
      searchQuery: '',
      statusFilter: '',

      // Sorting
      sortKey: 'created_at',
      sortDir: 'desc',

      // Pagination
      currentPage: 1,
      perPage: 10,

      // Detail modal
      detailRow: null,
      selectedRow: null,

      // External ID input (for lookup)
      lookupId: this.externalId || '',
    }
  },
  computed: {
    filteredRows() {
      let rows = [...this.loanRequests]

      // Text search
      if (this.searchQuery.trim()) {
        const q = this.searchQuery.toLowerCase()
        rows = rows.filter(r => {
          return (
            String(r.external_id || r.id || '').toLowerCase().includes(q) ||
            String(r.loan_product || r.product_name || r.product || '').toLowerCase().includes(q) ||
            String(r.status || '').toLowerCase().includes(q) ||
            String(r.amount || r.loan_amount || '').includes(q)
          )
        })
      }

      // Status filter
      if (this.statusFilter) {
        rows = rows.filter(r =>
          String(r.status || '').toLowerCase() === this.statusFilter.toLowerCase()
        )
      }

      // Sort
      rows.sort((a, b) => {
        let aVal = a[this.sortKey]
        let bVal = b[this.sortKey]

        if (this.sortKey === 'amount') {
          aVal = Number(a.amount || a.loan_amount || 0)
          bVal = Number(b.amount || b.loan_amount || 0)
        } else if (this.sortKey === 'created_at') {
          aVal = new Date(a.created_at || a.date_applied || 0).getTime()
          bVal = new Date(b.created_at || b.date_applied || 0).getTime()
        } else if (this.sortKey === 'duration') {
          aVal = Number(a.duration || a.loan_period || 0)
          bVal = Number(b.duration || b.loan_period || 0)
        } else {
          aVal = String(aVal || '').toLowerCase()
          bVal = String(bVal || '').toLowerCase()
        }

        if (aVal < bVal) return this.sortDir === 'asc' ? -1 : 1
        if (aVal > bVal) return this.sortDir === 'asc' ? 1 : -1
        return 0
      })

      return rows
    },

    totalPages() {
      return Math.max(1, Math.ceil(this.filteredRows.length / this.perPage))
    },

    paginatedRows() {
      const start = (this.currentPage - 1) * this.perPage
      return this.filteredRows.slice(start, start + this.perPage)
    },

    paginationStart() {
      if (this.filteredRows.length === 0) return 0
      return (this.currentPage - 1) * this.perPage + 1
    },

    paginationEnd() {
      return Math.min(this.currentPage * this.perPage, this.filteredRows.length)
    },

    visiblePages() {
      const total = this.totalPages
      const current = this.currentPage
      const pages = []

      if (total <= 7) {
        for (let i = 1; i <= total; i++) pages.push(i)
      } else {
        pages.push(1)
        if (current > 3) pages.push('...')
        for (let i = Math.max(2, current - 1); i <= Math.min(total - 1, current + 1); i++) {
          pages.push(i)
        }
        if (current < total - 2) pages.push('...')
        pages.push(total)
      }

      return pages
    },

    flattenedDetail() {
      if (!this.detailRow) return {}
      const flat = {}
      const flatten = (obj, prefix = '') => {
        for (const [k, v] of Object.entries(obj)) {
          const key = prefix ? `${prefix}_${k}` : k
          if (v !== null && typeof v === 'object' && !Array.isArray(v)) {
            flatten(v, key)
          } else {
            flat[key] = v
          }
        }
      }
      flatten(this.detailRow)
      return flat
    },
  },
  watch: {
    searchQuery() { this.currentPage = 1 },
    statusFilter() { this.currentPage = 1 },
    perPage() { this.currentPage = 1 },
    externalId(val) {
      if (val) {
        this.lookupId = val
        this.fetchHistory()
      }
    },
  },
  mounted() {
    this.fetchHistory()
  },
  methods: {
    async fetchHistory() {
      this.loading = true
      this.error = null

      try {
        const token = sessionStorage.getItem('auth_token')
        const headers = { 'Accept': 'application/json' }
        if (token && token !== 'session') {
          headers['Authorization'] = `Bearer ${token}`
        }

        const id = this.lookupId || this.externalId || ''
        const url = id
          ? `https://mwdev.pesapal.credit/api/merchant/loans/requests/${encodeURIComponent(id)}`
          : 'https://mwdev.pesapal.credit/api/merchant/loans/requests'

        const response = await axios.get(url, { timeout: 15000, headers })
        const data = response.data

        // Normalise to array
        if (Array.isArray(data)) {
          this.loanRequests = data
        } else if (data && Array.isArray(data.data)) {
          this.loanRequests = data.data
        } else if (data && Array.isArray(data.requests)) {
          this.loanRequests = data.requests
        } else if (data && Array.isArray(data.results)) {
          this.loanRequests = data.results
        } else if (data && typeof data === 'object') {
          // Single record returned
          this.loanRequests = [data]
        } else {
          this.loanRequests = []
        }
      } catch (err) {
        console.error('Loan history fetch error:', err)
        if (err.response) {
          const status = err.response.status
          const msg = err.response.data?.message || err.response.data?.error || null
          if (status === 401 || status === 403) {
            this.error = 'Session expired. Please log out and log in again.'
          } else if (status === 404) {
            this.error = 'No loan requests found for the given External ID.'
            this.loanRequests = []
          } else if (status === 422) {
            this.error = msg || 'Invalid request parameters.'
          } else {
            this.error = `Server error (${status}). Please try again.`
          }
        } else if (err.code === 'ECONNABORTED') {
          this.error = 'Request timed out. Please check your connection.'
        } else {
          this.error = 'Unable to load loan history. Please try again.'
        }
      } finally {
        this.loading = false
      }
    },

    sortBy(key) {
      if (this.sortKey === key) {
        this.sortDir = this.sortDir === 'asc' ? 'desc' : 'asc'
      } else {
        this.sortKey = key
        this.sortDir = 'asc'
      }
    },

    getSortIcon(key) {
      if (this.sortKey !== key) return '↕'
      return this.sortDir === 'asc' ? '↑' : '↓'
    },

    selectRow(row) {
      this.selectedRow = this.selectedRow === row ? null : row
    },

    viewDetails(row) {
      this.detailRow = row
    },

    formatCurrency(value) {
      if (value === null || value === undefined || value === '') return '—'
      return new Intl.NumberFormat('en-KE', {
        style: 'currency',
        currency: 'KES',
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      }).format(value)
    },

    formatRate(rate) {
      if (rate === null || rate === undefined) return '0'
      return parseFloat(rate).toFixed(2)
    },

    formatDateDay(dateStr) {
      if (!dateStr) return '—'
      return new Date(dateStr).getDate().toString().padStart(2, '0')
    },

    formatDateMonthYear(dateStr) {
      if (!dateStr) return ''
      return new Date(dateStr).toLocaleDateString('en-KE', { month: 'short', year: 'numeric' })
    },

    formatStatus(status) {
      if (!status) return 'Unknown'
      return status.charAt(0).toUpperCase() + status.slice(1).toLowerCase()
    },

    getStatusClass(status) {
      const s = String(status || '').toLowerCase()
      if (s === 'approved' || s === 'disbursed') return 'status-approved'
      if (s === 'pending' || s === 'processing') return 'status-pending'
      if (s === 'rejected' || s === 'declined') return 'status-rejected'
      if (s === 'completed' || s === 'paid') return 'status-completed'
      return 'status-unknown'
    },

    formatKey(key) {
      return key
        .replace(/_/g, ' ')
        .replace(/\b\w/g, c => c.toUpperCase())
    },

    formatDetailValue(key, value) {
      if (value === null || value === undefined) return '—'
      if (Array.isArray(value)) return `[${value.length} items]`
      const lk = key.toLowerCase()
      if (lk.includes('amount') || lk.includes('balance')) return this.formatCurrency(value)
      if (lk.includes('rate')) return `${parseFloat(value).toFixed(2)}%`
      if (lk.includes('date') || lk.includes('_at')) {
        try { return new Date(value).toLocaleString('en-KE') } catch { return value }
      }
      return String(value)
    },
  },
}
</script>

<style scoped>
.loan-history {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* Toolbar */
.history-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
  background: var(--white);
  padding: 16px 20px;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  border: 1px solid var(--gray-100);
}

.toolbar-left {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
  flex: 1;
}

.toolbar-right {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.search-wrapper {
  position: relative;
  min-width: 220px;
  flex: 1;
  max-width: 360px;
}

.search-icon {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--gray-400);
  display: flex;
  align-items: center;
  pointer-events: none;
}

.search-input {
  width: 100%;
  padding: 8px 12px 8px 34px;
  border: 1.5px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 13px;
  color: var(--gray-800);
  background: var(--white);
  font-family: inherit;
  transition: border-color 0.15s, box-shadow 0.15s;
  outline: none;
}

.search-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(26, 86, 219, 0.1);
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.filter-select {
  padding: 8px 12px;
  border: 1.5px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 13px;
  color: var(--gray-700);
  background: var(--white);
  font-family: inherit;
  cursor: pointer;
  outline: none;
  transition: border-color 0.15s;
}

.filter-select:focus {
  border-color: var(--primary);
}

.per-page-group {
  display: flex;
  align-items: center;
  gap: 6px;
}

.per-page-label {
  font-size: 13px;
  color: var(--gray-500);
  white-space: nowrap;
}

.per-page-select {
  width: 70px;
}

/* Buttons */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  border-radius: var(--radius);
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.15s;
  font-family: inherit;
}

.btn-sm {
  padding: 7px 12px;
  font-size: 12px;
}

.btn-outline {
  background: transparent;
  color: var(--gray-600);
  border: 1.5px solid var(--gray-200);
}

.btn-outline:hover:not(:disabled) {
  border-color: var(--gray-400);
  color: var(--gray-800);
}

.btn-outline:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-icon {
  width: 30px;
  height: 30px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  background: var(--gray-100);
  border-radius: 6px;
  cursor: pointer;
  color: var(--gray-600);
  transition: all 0.15s;
}

.btn-icon:hover {
  background: var(--primary-light);
  color: var(--primary);
}

/* Error Banner */
.error-banner {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  background: rgba(239, 68, 68, 0.08);
  border: 1px solid rgba(239, 68, 68, 0.25);
  border-radius: var(--radius);
  color: #dc2626;
  font-size: 13px;
}

.error-banner svg { flex-shrink: 0; }

.error-dismiss {
  margin-left: auto;
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  color: #dc2626;
  line-height: 1;
  padding: 0 4px;
}

/* Skeleton */
.skeleton-wrapper {
  background: var(--white);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  overflow: hidden;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.skeleton-row {
  display: flex;
  gap: 16px;
  align-items: center;
}

.skeleton-cell {
  height: 14px;
  background: linear-gradient(90deg, var(--gray-100) 25%, var(--gray-50) 50%, var(--gray-100) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: 4px;
}

.skeleton-sm { width: 80px; }
.skeleton-md { width: 140px; }
.skeleton-lg { width: 200px; flex: 1; }

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Table Card */
.table-card {
  background: var(--white);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  border: 1px solid var(--gray-100);
  overflow: hidden;
}

.table-wrapper {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}

.data-table thead {
  background: var(--gray-800);
  position: sticky;
  top: 0;
  z-index: 1;
}

.data-table th {
  padding: 12px 16px;
  text-align: left;
  font-size: 11px;
  font-weight: 700;
  color: var(--gray-300);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  white-space: nowrap;
}

.th-right { text-align: right; }
.th-center { text-align: center; }

.th-sortable {
  cursor: pointer;
  user-select: none;
  transition: color 0.15s;
}

.th-sortable:hover {
  color: white;
}

.sort-icon {
  margin-left: 4px;
  font-size: 10px;
  opacity: 0.7;
}

.data-table tbody tr {
  border-bottom: 1px solid var(--gray-100);
  transition: background 0.1s;
  cursor: pointer;
}

.data-table tbody tr:hover {
  background: var(--gray-50);
}

.data-table tbody tr:last-child {
  border-bottom: none;
}

.data-row.row-selected {
  background: var(--primary-light) !important;
}

.data-table td {
  padding: 12px 16px;
  color: var(--gray-700);
  white-space: nowrap;
}

.td-right { text-align: right; font-variant-numeric: tabular-nums; }
.td-center { text-align: center; }

.td-id .id-badge {
  font-family: monospace;
  font-size: 12px;
  background: var(--gray-100);
  padding: 3px 8px;
  border-radius: 4px;
  color: var(--gray-700);
  font-weight: 600;
}

.product-cell {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.product-name {
  font-weight: 500;
  color: var(--gray-800);
}

.product-rate {
  font-size: 11px;
  color: var(--gray-500);
}

.td-amount {
  font-weight: 600;
  color: var(--gray-800);
}

.td-duration {
  font-weight: 500;
}

.duration-unit {
  font-size: 11px;
  color: var(--gray-500);
  margin-left: 2px;
}

.td-date .date-cell {
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.date-day {
  font-size: 15px;
  font-weight: 700;
  color: var(--gray-800);
  line-height: 1;
}

.date-month-year {
  font-size: 11px;
  color: var(--gray-500);
}

.td-actions {
  min-width: 60px;
}

/* Status Badges */
.status-badge {
  display: inline-flex;
  align-items: center;
  padding: 3px 10px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 600;
  white-space: nowrap;
}

.status-approved {
  background: rgba(14, 159, 110, 0.12);
  color: #065f46;
}

.status-pending {
  background: rgba(245, 158, 11, 0.12);
  color: #92400e;
}

.status-rejected {
  background: rgba(239, 68, 68, 0.12);
  color: #991b1b;
}

.status-completed {
  background: rgba(26, 86, 219, 0.12);
  color: #1e40af;
}

.status-unknown {
  background: var(--gray-100);
  color: var(--gray-500);
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 48px 24px !important;
}

.empty-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: var(--gray-400);
}

.empty-title {
  font-size: 15px;
  font-weight: 600;
  color: var(--gray-600);
  margin-top: 8px;
}

.empty-subtitle {
  font-size: 13px;
  color: var(--gray-400);
  max-width: 320px;
}

/* Pagination */
.pagination-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 20px;
  border-top: 1px solid var(--gray-100);
  flex-wrap: wrap;
  gap: 12px;
}

.pagination-info {
  font-size: 13px;
  color: var(--gray-500);
}

.filter-note {
  color: var(--gray-400);
  font-size: 12px;
  margin-left: 4px;
}

.pagination-controls {
  display: flex;
  align-items: center;
  gap: 4px;
}

.page-btn {
  min-width: 32px;
  height: 32px;
  padding: 0 8px;
  border: 1.5px solid var(--gray-200);
  border-radius: 6px;
  background: var(--white);
  font-size: 13px;
  font-weight: 500;
  color: var(--gray-600);
  cursor: pointer;
  transition: all 0.15s;
  font-family: inherit;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.page-btn:hover:not(:disabled):not(.ellipsis) {
  border-color: var(--primary);
  color: var(--primary);
}

.page-btn.active {
  background: var(--primary);
  border-color: var(--primary);
  color: white;
}

.page-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.page-btn.ellipsis {
  border: none;
  cursor: default;
}

/* Spinner */
.spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 20px;
}

.modal-box {
  background: var(--white);
  border-radius: var(--radius-lg);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 600px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px;
  border-bottom: 1px solid var(--gray-100);
  background: var(--gray-50);
}

.modal-title {
  font-size: 16px;
  font-weight: 700;
  color: var(--gray-800);
}

.modal-close {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--gray-400);
  display: flex;
  align-items: center;
  padding: 4px;
  border-radius: 6px;
  transition: all 0.15s;
}

.modal-close:hover {
  background: var(--gray-100);
  color: var(--gray-700);
}

.modal-body {
  padding: 24px;
  overflow-y: auto;
  flex: 1;
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

@media (max-width: 480px) {
  .detail-grid {
    grid-template-columns: 1fr;
  }
}

.detail-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding: 12px;
  background: var(--gray-50);
  border-radius: var(--radius);
  border: 1px solid var(--gray-100);
}

.detail-label {
  font-size: 10px;
  font-weight: 700;
  color: var(--gray-500);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.detail-value {
  font-size: 13px;
  font-weight: 500;
  color: var(--gray-800);
  word-break: break-all;
}

.modal-footer {
  padding: 16px 24px;
  border-top: 1px solid var(--gray-100);
  display: flex;
  justify-content: flex-end;
}

/* Modal Transition */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.2s ease;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}

.modal-fade-enter-active .modal-box,
.modal-fade-leave-active .modal-box {
  transition: transform 0.2s ease;
}

.modal-fade-enter-from .modal-box {
  transform: scale(0.95) translateY(-10px);
}

.modal-fade-leave-to .modal-box {
  transform: scale(0.95) translateY(-10px);
}

/* Responsive */
@media (max-width: 640px) {
  .history-toolbar {
    flex-direction: column;
    align-items: stretch;
  }

  .toolbar-left,
  .toolbar-right {
    width: 100%;
  }

  .search-wrapper {
    max-width: 100%;
  }

  .pagination-bar {
    flex-direction: column;
    align-items: center;
  }
}
</style>
