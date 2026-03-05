<template>
  <!-- Show Login if not authenticated -->
  <Login v-if="!isAuthenticated" @login-success="onLoginSuccess" />

  <!-- Show Calculator if authenticated -->
  <div v-else class="app-container">
    <!-- Header -->
    <header class="app-header">
      <div class="header-content">
        <div class="header-icon">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="28" height="28">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1.41 16.09V20h-2.67v-1.93c-1.71-.36-3.16-1.46-3.27-3.4h1.96c.1 1.05.82 1.87 2.65 1.87 1.96 0 2.4-.98 2.4-1.59 0-.83-.44-1.61-2.67-2.14-2.48-.6-4.18-1.62-4.18-3.67 0-1.72 1.39-2.84 3.11-3.21V4h2.67v1.95c1.86.45 2.79 1.86 2.85 3.39H14.3c-.05-1.11-.64-1.87-2.22-1.87-1.5 0-2.4.68-2.4 1.64 0 .84.65 1.39 2.67 1.91s4.18 1.39 4.18 3.91c-.01 1.83-1.38 2.83-3.12 3.16z"/>
          </svg>
        </div>
        <div class="header-text">
          <h1 class="header-title">Loan Calculator</h1>
          <p class="header-subtitle">Calculate your loan repayment schedule</p>
        </div>
        <!-- User info + Logout -->
        <div class="header-user">
          <span class="header-username">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="16" height="16">
              <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
            </svg>
            {{ currentUser }}
          </span>
          <button class="btn-logout" @click="handleLogout" type="button">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="16" height="16">
              <path d="M17 7l-1.41 1.41L18.17 11H8v2h10.17l-2.58 2.58L17 17l5-5zM4 5h8V3H4c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h8v-2H4V5z"/>
            </svg>
            Logout
          </button>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="main-content">
      <!-- Calculator Form Card -->
      <div class="card">
        <div class="card-header">
          <h2 class="card-title">
            <span class="step-badge">1</span>
            Loan Details
          </h2>
        </div>
        <div class="card-body">
          <div class="form-grid">
            <!-- Loan Type -->
            <div class="form-group">
              <label class="form-label" for="loanType">
                Loan Type
                <span class="required">*</span>
              </label>
              <div class="select-wrapper">
                <select
                  id="loanType"
                  v-model="selectedProduct"
                  class="form-select"
                  :class="{ 'is-loading': loadingProducts, 'is-error': productError }"
                  :disabled="loadingProducts"
                  @change="onProductChange"
                >
                  <option value="" disabled>
                    {{ loadingProducts ? 'Loading products...' : 'Select a loan type' }}
                  </option>
                  <option
                    v-for="product in loanProducts"
                    :key="product.id"
                    :value="product"
                  >
                    {{ product.name }} — {{ formatRate(product.interest_rate) }}% interest
                  </option>
                </select>
                <div class="select-icon">
                  <svg v-if="loadingProducts" class="spinner" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                  </svg>
                  <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" width="16" height="16">
                    <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                  </svg>
                </div>
              </div>
              <p v-if="productError" class="form-error">{{ productError }}</p>
              <!-- Product Info Badge -->
              <div v-if="selectedProduct" class="product-info-badge">
                <div class="badge-item">
                  <span class="badge-label">Interest Rate</span>
                  <span class="badge-value primary">{{ formatRate(selectedProduct.interest_rate) }}%</span>
                </div>
                <div v-if="selectedProduct.min_amount" class="badge-item">
                  <span class="badge-label">Min Amount</span>
                  <span class="badge-value">{{ formatCurrency(selectedProduct.min_amount) }}</span>
                </div>
                <div v-if="selectedProduct.max_amount" class="badge-item">
                  <span class="badge-label">Max Amount</span>
                  <span class="badge-value">{{ formatCurrency(selectedProduct.max_amount) }}</span>
                </div>
              </div>
            </div>

            <!-- Loan Amount -->
            <div class="form-group">
              <label class="form-label" for="loanAmount">
                Loan Amount (KES)
                <span class="required">*</span>
              </label>
              <div class="input-wrapper">
                <span class="input-prefix">KES</span>
                <input
                  id="loanAmount"
                  v-model.number="loanAmount"
                  type="number"
                  class="form-input with-prefix"
                  :class="{ 'is-error': amountError }"
                  placeholder="e.g. 50,000"
                  min="0"
                  step="1000"
                />
              </div>
              <p v-if="amountError" class="form-error">{{ amountError }}</p>
            </div>

            <!-- Selected Period -->
            <div class="form-group">
              <label class="form-label" for="loanPeriod">
                Loan Period (Months)
                <span class="required">*</span>
              </label>
              <div class="input-wrapper">
                <input
                  id="loanPeriod"
                  v-model.number="loanPeriod"
                  type="number"
                  class="form-input"
                  :class="{ 'is-error': periodError }"
                  placeholder="e.g. 12"
                  min="1"
                  max="360"
                />
                <span class="input-suffix">months</span>
              </div>
              <p v-if="periodError" class="form-error">{{ periodError }}</p>
              <!-- Period Quick Select -->
              <div class="period-quick-select">
                <button
                  v-for="months in [3, 6, 12, 24, 36]"
                  :key="months"
                  class="period-btn"
                  :class="{ active: loanPeriod === months }"
                  @click="loanPeriod = months"
                  type="button"
                >
                  {{ months }}m
                </button>
              </div>
            </div>
          </div>

          <!-- Calculate Button -->
          <div class="form-actions">
            <button
              class="btn btn-primary btn-lg"
              @click="calculateLoan"
              :disabled="!canCalculate"
            >
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="20" height="20">
                <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm-7 3c1.93 0 3.5 1.57 3.5 3.5S13.93 13 12 13s-3.5-1.57-3.5-3.5S10.07 6 12 6zm7 13H5v-.23c0-.62.28-1.2.76-1.58C7.47 15.82 9.64 15 12 15s4.53.82 6.24 2.19c.48.38.76.97.76 1.58V19z"/>
              </svg>
              Calculate Loan
            </button>
            <button
              v-if="loanBreakdown"
              class="btn btn-outline"
              @click="resetCalculator"
              type="button"
            >
              Reset
            </button>
          </div>
        </div>
      </div>

      <!-- Loan Breakdown Card -->
      <transition name="slide-up">
        <div v-if="loanBreakdown" class="card">
          <div class="card-header">
            <h2 class="card-title">
              <span class="step-badge success">2</span>
              Loan Breakdown
            </h2>
          </div>
          <div class="card-body">
            <div class="breakdown-grid">
              <div class="breakdown-item">
                <div class="breakdown-icon primary">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="22" height="22">
                    <path d="M11.8 10.9c-2.27-.59-3-1.2-3-2.15 0-1.09 1.01-1.85 2.7-1.85 1.78 0 2.44.85 2.5 2.1h2.21c-.07-1.72-1.12-3.3-3.21-3.81V3h-3v2.16c-1.94.42-3.5 1.68-3.5 3.61 0 2.31 1.91 3.46 4.7 4.13 2.5.6 3 1.48 3 2.41 0 .69-.49 1.79-2.7 1.79-2.06 0-2.87-.92-2.98-2.1h-2.2c.12 2.19 1.76 3.42 3.68 3.83V21h3v-2.15c1.95-.37 3.5-1.5 3.5-3.55 0-2.84-2.43-3.81-4.7-4.4z"/>
                  </svg>
                </div>
                <div class="breakdown-content">
                  <span class="breakdown-label">Loan Amount</span>
                  <span class="breakdown-value">{{ formatCurrency(loanBreakdown.principal) }}</span>
                </div>
              </div>

              <div class="breakdown-item">
                <div class="breakdown-icon blue">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="22" height="22">
                    <path d="M11.99 2C6.47 2 2 6.48 2 12s4.47 10 9.99 10C17.52 22 22 17.52 22 12S17.52 2 11.99 2zM12 20c-4.42 0-8-3.58-8-8s3.58-8 8-8 8 3.58 8 8-3.58 8-8 8zm.5-13H11v6l5.25 3.15.75-1.23-4.5-2.67V7z"/>
                  </svg>
                </div>
                <div class="breakdown-content">
                  <span class="breakdown-label">Duration</span>
                  <span class="breakdown-value">{{ loanBreakdown.duration }} months</span>
                </div>
              </div>

              <div class="breakdown-item">
                <div class="breakdown-icon orange">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="22" height="22">
                    <path d="M11.5 2C6.81 2 3 5.81 3 10.5S6.81 19 11.5 19h.5v3c4.86-2.34 8-7 8-11.5C20 5.81 16.19 2 11.5 2zm1 14.5h-2v-2h2v2zm0-4h-2c0-3.25 3-3 3-5 0-1.1-.9-2-2-2s-2 .9-2 2h-2c0-2.21 1.79-4 4-4s4 1.79 4 4c0 2.5-3 2.75-3 5z"/>
                  </svg>
                </div>
                <div class="breakdown-content">
                  <span class="breakdown-label">Total Interest</span>
                  <span class="breakdown-value">{{ formatCurrency(loanBreakdown.totalInterest) }}</span>
                </div>
              </div>

              <div class="breakdown-item highlight">
                <div class="breakdown-icon green">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="22" height="22">
                    <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm-7 3c1.93 0 3.5 1.57 3.5 3.5S13.93 13 12 13s-3.5-1.57-3.5-3.5S10.07 6 12 6zm7 13H5v-.23c0-.62.28-1.2.76-1.58C7.47 15.82 9.64 15 12 15s4.53.82 6.24 2.19c.48.38.76.97.76 1.58V19z"/>
                  </svg>
                </div>
                <div class="breakdown-content">
                  <span class="breakdown-label">Monthly Installment</span>
                  <span class="breakdown-value large">{{ formatCurrency(loanBreakdown.monthlyInstallment) }}</span>
                </div>
              </div>
            </div>

            <!-- Summary Bar -->
            <div class="summary-bar">
              <div class="summary-item">
                <span class="summary-label">Total Repayment</span>
                <span class="summary-value">{{ formatCurrency(loanBreakdown.totalRepayment) }}</span>
              </div>
              <div class="summary-divider"></div>
              <div class="summary-item">
                <span class="summary-label">Interest Rate</span>
                <span class="summary-value">{{ formatRate(loanBreakdown.interestRate) }}% p.m.</span>
              </div>
              <div class="summary-divider"></div>
              <div class="summary-item">
                <span class="summary-label">Loan Product</span>
                <span class="summary-value">{{ loanBreakdown.productName }}</span>
              </div>
            </div>
          </div>
        </div>
      </transition>

      <!-- Repayment Schedule Card -->
      <transition name="slide-up">
        <div v-if="repaymentSchedule.length > 0" class="card">
          <div class="card-header">
            <h2 class="card-title">
              <span class="step-badge info">3</span>
              Expected Repayment Schedule
            </h2>
            <div class="card-header-actions">
              <span class="schedule-count">{{ repaymentSchedule.length }} installments</span>
            </div>
          </div>
          <div class="card-body no-padding">
            <div class="table-wrapper">
              <table class="data-table">
                <thead>
                  <tr>
                    <th class="th-number">#</th>
                    <th>Due Date</th>
                    <th class="th-right">Opening Balance</th>
                    <th class="th-right">Principal</th>
                    <th class="th-right">Interest</th>
                    <th class="th-right">Installment</th>
                    <th class="th-right">Closing Balance</th>
                    <th class="th-center">Status</th>
                  </tr>
                </thead>
                <tbody>
                  <tr
                    v-for="(row, index) in repaymentSchedule"
                    :key="index"
                    :class="{ 'row-highlight': row.isFirst, 'row-last': row.isLast }"
                  >
                    <td class="td-number">{{ row.installmentNumber }}</td>
                    <td class="td-date">
                      <div class="date-cell">
                        <span class="date-day">{{ formatDateDay(row.dueDate) }}</span>
                        <span class="date-month-year">{{ formatDateMonthYear(row.dueDate) }}</span>
                      </div>
                    </td>
                    <td class="td-right">{{ formatCurrency(row.openingBalance) }}</td>
                    <td class="td-right">{{ formatCurrency(row.principal) }}</td>
                    <td class="td-right interest">{{ formatCurrency(row.interest) }}</td>
                    <td class="td-right installment">{{ formatCurrency(row.installmentAmount) }}</td>
                    <td class="td-right">{{ formatCurrency(row.closingBalance) }}</td>
                    <td class="td-center">
                      <span class="status-badge" :class="getStatusClass(row.dueDate)">
                        {{ getStatusLabel(row.dueDate) }}
                      </span>
                    </td>
                  </tr>
                </tbody>
                <tfoot>
                  <tr class="table-footer">
                    <td colspan="3" class="tf-label">Totals</td>
                    <td class="td-right tf-value">{{ formatCurrency(totalPrincipal) }}</td>
                    <td class="td-right tf-value">{{ formatCurrency(totalInterestPaid) }}</td>
                    <td class="td-right tf-value">{{ formatCurrency(totalInstallments) }}</td>
                    <td colspan="2"></td>
                  </tr>
                </tfoot>
              </table>
            </div>
          </div>
        </div>
      </transition>
    </main>

    <!-- Footer -->
    <footer class="app-footer">
      <p>Loan Calculator &copy; {{ new Date().getFullYear() }} · Powered by PesaPal Credit</p>
    </footer>
  </div>
</template>

<script>
import axios from 'axios'
import Login from './Login.vue'

export default {
  name: 'App',
  components: { Login },
  data() {
    return {
      // Auth state
      isAuthenticated: !!sessionStorage.getItem('auth_token'),
      authUser: JSON.parse(sessionStorage.getItem('auth_user') || 'null'),

      // Form fields
      selectedProduct: '',
      loanAmount: null,
      loanPeriod: null,

      // API data
      loanProducts: [],
      loadingProducts: false,
      productError: null,

      // Validation errors
      amountError: null,
      periodError: null,

      // Results
      loanBreakdown: null,
      repaymentSchedule: [],
    }
  },
  computed: {
    canCalculate() {
      return this.selectedProduct && this.loanAmount > 0 && this.loanPeriod > 0
    },
    totalPrincipal() {
      return this.repaymentSchedule.reduce((sum, row) => sum + row.principal, 0)
    },
    totalInterestPaid() {
      return this.repaymentSchedule.reduce((sum, row) => sum + row.interest, 0)
    },
    totalInstallments() {
      return this.repaymentSchedule.reduce((sum, row) => sum + row.installmentAmount, 0)
    },
    currentUser() {
      if (!this.authUser) return 'User'
      return this.authUser.name || this.authUser.username || this.authUser.email || 'User'
    },
  },
  mounted() {
    if (this.isAuthenticated) {
      this.fetchLoanProducts()
    }
  },
  methods: {
    onLoginSuccess({ token, user }) {
      this.isAuthenticated = true
      this.authUser = user
      this.fetchLoanProducts()
    },

    handleLogout() {
      sessionStorage.removeItem('auth_token')
      sessionStorage.removeItem('auth_user')
      this.isAuthenticated = false
      this.authUser = null
      // Reset calculator state
      this.selectedProduct = ''
      this.loanAmount = null
      this.loanPeriod = null
      this.loanBreakdown = null
      this.repaymentSchedule = []
      this.loanProducts = []
      this.productError = null
      this.amountError = null
      this.periodError = null
    },

    async fetchLoanProducts() {
      this.loadingProducts = true
      this.productError = null
      try {
        const response = await axios.get('https://mwdev.pesapal.credit/api/merchant/loans/products', {
          timeout: 10000,
          headers: {
            'Accept': 'application/json',
          }
        })
        // Handle various response structures
        const data = response.data
        if (Array.isArray(data)) {
          this.loanProducts = data
        } else if (data && Array.isArray(data.data)) {
          this.loanProducts = data.data
        } else if (data && Array.isArray(data.products)) {
          this.loanProducts = data.products
        } else if (data && Array.isArray(data.results)) {
          this.loanProducts = data.results
        } else {
          // Fallback: try to use the data directly
          this.loanProducts = data ? [data] : []
        }
      } catch (error) {
        console.error('Failed to fetch loan products:', error)
        this.productError = 'Failed to load loan products. Using demo data.'
        // Demo fallback data
        this.loanProducts = [
          { id: 1, name: 'Personal Loan', interest_rate: 2.5, min_amount: 10000, max_amount: 500000 },
          { id: 2, name: 'Business Loan', interest_rate: 3.0, min_amount: 50000, max_amount: 2000000 },
          { id: 3, name: 'Emergency Loan', interest_rate: 4.0, min_amount: 5000, max_amount: 100000 },
          { id: 4, name: 'Asset Finance', interest_rate: 1.8, min_amount: 100000, max_amount: 5000000 },
        ]
      } finally {
        this.loadingProducts = false
      }
    },

    onProductChange() {
      // Reset results when product changes
      this.loanBreakdown = null
      this.repaymentSchedule = []
    },

    validateForm() {
      let valid = true
      this.amountError = null
      this.periodError = null

      if (!this.loanAmount || this.loanAmount <= 0) {
        this.amountError = 'Please enter a valid loan amount greater than 0'
        valid = false
      } else if (this.selectedProduct) {
        if (this.selectedProduct.min_amount && this.loanAmount < this.selectedProduct.min_amount) {
          this.amountError = `Minimum amount for this product is ${this.formatCurrency(this.selectedProduct.min_amount)}`
          valid = false
        } else if (this.selectedProduct.max_amount && this.loanAmount > this.selectedProduct.max_amount) {
          this.amountError = `Maximum amount for this product is ${this.formatCurrency(this.selectedProduct.max_amount)}`
          valid = false
        }
      }

      if (!this.loanPeriod || this.loanPeriod <= 0) {
        this.periodError = 'Please enter a valid loan period greater than 0'
        valid = false
      } else if (this.loanPeriod > 360) {
        this.periodError = 'Loan period cannot exceed 360 months'
        valid = false
      }

      return valid
    },

    calculateLoan() {
      if (!this.validateForm()) return

      const principal = this.loanAmount
      const monthlyRate = this.selectedProduct.interest_rate / 100
      const months = this.loanPeriod

      // Calculate monthly installment using reducing balance method
      // M = P * [r(1+r)^n] / [(1+r)^n - 1]
      let monthlyInstallment
      if (monthlyRate === 0) {
        monthlyInstallment = principal / months
      } else {
        const factor = Math.pow(1 + monthlyRate, months)
        monthlyInstallment = principal * (monthlyRate * factor) / (factor - 1)
      }

      const totalRepayment = monthlyInstallment * months
      const totalInterest = totalRepayment - principal

      this.loanBreakdown = {
        principal,
        duration: months,
        interestRate: this.selectedProduct.interest_rate,
        monthlyInstallment,
        totalInterest,
        totalRepayment,
        productName: this.selectedProduct.name,
      }

      // Generate repayment schedule
      this.generateRepaymentSchedule(principal, monthlyRate, months, monthlyInstallment)
    },

    generateRepaymentSchedule(principal, monthlyRate, months, monthlyInstallment) {
      const schedule = []
      let balance = principal
      const startDate = new Date()
      startDate.setDate(1) // Start from 1st of next month
      startDate.setMonth(startDate.getMonth() + 1)

      for (let i = 1; i <= months; i++) {
        const dueDate = new Date(startDate)
        dueDate.setMonth(startDate.getMonth() + (i - 1))

        const openingBalance = balance
        const interest = balance * monthlyRate
        let principalPayment = monthlyInstallment - interest

        // Last installment adjustment for rounding
        if (i === months) {
          principalPayment = balance
        }

        const closingBalance = Math.max(0, balance - principalPayment)
        const actualInstallment = i === months
          ? principalPayment + interest
          : monthlyInstallment

        schedule.push({
          installmentNumber: i,
          dueDate: new Date(dueDate),
          openingBalance: Math.round(openingBalance * 100) / 100,
          principal: Math.round(principalPayment * 100) / 100,
          interest: Math.round(interest * 100) / 100,
          installmentAmount: Math.round(actualInstallment * 100) / 100,
          closingBalance: Math.round(closingBalance * 100) / 100,
          isFirst: i === 1,
          isLast: i === months,
        })

        balance = closingBalance
      }

      this.repaymentSchedule = schedule
    },

    resetCalculator() {
      this.selectedProduct = ''
      this.loanAmount = null
      this.loanPeriod = null
      this.loanBreakdown = null
      this.repaymentSchedule = []
      this.amountError = null
      this.periodError = null
    },

    // Formatting helpers
    formatCurrency(value) {
      if (value === null || value === undefined) return '—'
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

    formatDateDay(date) {
      return date.getDate().toString().padStart(2, '0')
    },

    formatDateMonthYear(date) {
      return date.toLocaleDateString('en-KE', { month: 'short', year: 'numeric' })
    },

    getStatusClass(dueDate) {
      const today = new Date()
      const due = new Date(dueDate)
      const diffDays = Math.ceil((due - today) / (1000 * 60 * 60 * 24))

      if (diffDays < 0) return 'status-overdue'
      if (diffDays <= 7) return 'status-due-soon'
      if (diffDays <= 30) return 'status-upcoming'
      return 'status-future'
    },

    getStatusLabel(dueDate) {
      const today = new Date()
      const due = new Date(dueDate)
      const diffDays = Math.ceil((due - today) / (1000 * 60 * 60 * 24))

      if (diffDays < 0) return 'Overdue'
      if (diffDays <= 7) return 'Due Soon'
      if (diffDays <= 30) return 'Upcoming'
      return 'Pending'
    },
  }
}
</script>

<style scoped>
/* App Container */
.app-container {
  max-width: 1100px;
  margin: 0 auto;
}

/* Header */
.app-header {
  margin-bottom: 28px;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 16px;
}

.header-text {
  flex: 1;
}

.header-user {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-left: auto;
}

.header-username {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.9);
  background: rgba(255, 255, 255, 0.15);
  padding: 6px 12px;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.25);
}

.btn-logout {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.9);
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  padding: 7px 14px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-logout:hover {
  background: rgba(255, 255, 255, 0.25);
  color: white;
}

.header-icon {
  width: 56px;
  height: 56px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  flex-shrink: 0;
}

.header-title {
  font-size: 28px;
  font-weight: 700;
  color: white;
  letter-spacing: -0.5px;
}

.header-subtitle {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.8);
  margin-top: 2px;
}

/* Main Content */
.main-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* Cards */
.card {
  background: var(--white);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  overflow: hidden;
}

.card-header {
  padding: 20px 24px;
  border-bottom: 1px solid var(--gray-100);
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: var(--gray-50);
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--gray-800);
  display: flex;
  align-items: center;
  gap: 10px;
}

.card-header-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.schedule-count {
  font-size: 13px;
  color: var(--gray-500);
  background: var(--gray-100);
  padding: 4px 10px;
  border-radius: 20px;
  font-weight: 500;
}

.card-body {
  padding: 24px;
}

.card-body.no-padding {
  padding: 0;
}

/* Step Badge */
.step-badge {
  width: 26px;
  height: 26px;
  border-radius: 50%;
  background: var(--primary);
  color: white;
  font-size: 13px;
  font-weight: 700;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.step-badge.success { background: var(--secondary); }
.step-badge.info { background: #7c3aed; }

/* Form */
.form-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
}

@media (min-width: 640px) {
  .form-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 900px) {
  .form-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--gray-700);
  letter-spacing: 0.01em;
}

.required {
  color: var(--danger);
  margin-left: 2px;
}

.select-wrapper {
  position: relative;
}

.form-select {
  width: 100%;
  padding: 10px 40px 10px 14px;
  border: 1.5px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 14px;
  color: var(--gray-800);
  background: var(--white);
  appearance: none;
  cursor: pointer;
  transition: border-color 0.15s, box-shadow 0.15s;
  font-family: inherit;
}

.form-select:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(26, 86, 219, 0.1);
}

.form-select:disabled {
  background: var(--gray-50);
  cursor: not-allowed;
  color: var(--gray-400);
}

.form-select.is-error {
  border-color: var(--danger);
}

.select-icon {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--gray-400);
  pointer-events: none;
  display: flex;
  align-items: center;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input-prefix {
  position: absolute;
  left: 14px;
  font-size: 13px;
  font-weight: 600;
  color: var(--gray-500);
  pointer-events: none;
}

.input-suffix {
  position: absolute;
  right: 14px;
  font-size: 13px;
  color: var(--gray-500);
  pointer-events: none;
}

.form-input {
  width: 100%;
  padding: 10px 14px;
  border: 1.5px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 14px;
  color: var(--gray-800);
  background: var(--white);
  transition: border-color 0.15s, box-shadow 0.15s;
  font-family: inherit;
}

.form-input.with-prefix {
  padding-left: 48px;
}

.form-input:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(26, 86, 219, 0.1);
}

.form-input.is-error {
  border-color: var(--danger);
}

.form-error {
  font-size: 12px;
  color: var(--danger);
  display: flex;
  align-items: center;
  gap: 4px;
}

/* Product Info Badge */
.product-info-badge {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  padding: 10px 12px;
  background: var(--primary-light);
  border-radius: var(--radius);
  border: 1px solid rgba(26, 86, 219, 0.15);
}

.badge-item {
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.badge-label {
  font-size: 10px;
  font-weight: 600;
  color: var(--gray-500);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.badge-value {
  font-size: 13px;
  font-weight: 600;
  color: var(--gray-700);
}

.badge-value.primary {
  color: var(--primary);
}

/* Period Quick Select */
.period-quick-select {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.period-btn {
  padding: 4px 10px;
  border: 1.5px solid var(--gray-200);
  border-radius: 20px;
  font-size: 12px;
  font-weight: 500;
  color: var(--gray-600);
  background: var(--white);
  cursor: pointer;
  transition: all 0.15s;
  font-family: inherit;
}

.period-btn:hover {
  border-color: var(--primary);
  color: var(--primary);
}

.period-btn.active {
  background: var(--primary);
  border-color: var(--primary);
  color: white;
}

/* Form Actions */
.form-actions {
  display: flex;
  gap: 12px;
  margin-top: 24px;
  padding-top: 20px;
  border-top: 1px solid var(--gray-100);
}

/* Buttons */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  border-radius: var(--radius);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s;
  border: none;
  font-family: inherit;
  text-decoration: none;
}

.btn-lg {
  padding: 12px 28px;
  font-size: 15px;
}

.btn-primary {
  background: var(--primary);
  color: white;
  box-shadow: 0 1px 3px rgba(26, 86, 219, 0.3);
}

.btn-primary:hover:not(:disabled) {
  background: var(--primary-dark);
  box-shadow: 0 4px 12px rgba(26, 86, 219, 0.4);
  transform: translateY(-1px);
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none;
}

.btn-outline {
  background: transparent;
  color: var(--gray-600);
  border: 1.5px solid var(--gray-200);
}

.btn-outline:hover {
  border-color: var(--gray-400);
  color: var(--gray-800);
}

/* Breakdown Grid */
.breakdown-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
  margin-bottom: 20px;
}

@media (min-width: 768px) {
  .breakdown-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

.breakdown-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 16px;
  background: var(--gray-50);
  border-radius: var(--radius);
  border: 1px solid var(--gray-100);
  transition: transform 0.15s;
}

.breakdown-item:hover {
  transform: translateY(-2px);
}

.breakdown-item.highlight {
  background: var(--secondary-light);
  border-color: rgba(14, 159, 110, 0.2);
}

.breakdown-icon {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.breakdown-icon.primary { background: var(--primary-light); color: var(--primary); }
.breakdown-icon.blue { background: #dbeafe; color: #1d4ed8; }
.breakdown-icon.orange { background: #fef3c7; color: #d97706; }
.breakdown-icon.green { background: var(--secondary-light); color: var(--secondary); }

.breakdown-content {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 0;
}

.breakdown-label {
  font-size: 11px;
  font-weight: 600;
  color: var(--gray-500);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.breakdown-value {
  font-size: 15px;
  font-weight: 700;
  color: var(--gray-800);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.breakdown-value.large {
  font-size: 17px;
  color: var(--secondary);
}

/* Summary Bar */
.summary-bar {
  display: flex;
  align-items: center;
  gap: 0;
  background: var(--gray-800);
  border-radius: var(--radius);
  overflow: hidden;
}

.summary-item {
  flex: 1;
  padding: 14px 20px;
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.summary-label {
  font-size: 11px;
  font-weight: 600;
  color: var(--gray-400);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.summary-value {
  font-size: 15px;
  font-weight: 700;
  color: white;
}

.summary-divider {
  width: 1px;
  height: 40px;
  background: rgba(255, 255, 255, 0.1);
}

/* Data Table */
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
.th-number { text-align: center; width: 48px; }
.th-center { text-align: center; }

.data-table tbody tr {
  border-bottom: 1px solid var(--gray-100);
  transition: background 0.1s;
}

.data-table tbody tr:hover {
  background: var(--gray-50);
}

.data-table tbody tr:last-child {
  border-bottom: none;
}

.row-highlight {
  background: var(--primary-light) !important;
}

.row-last {
  background: var(--secondary-light) !important;
}

.data-table td {
  padding: 12px 16px;
  color: var(--gray-700);
  white-space: nowrap;
}

.td-number {
  text-align: center;
  font-weight: 600;
  color: var(--gray-500);
  font-size: 12px;
}

.td-right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

.td-center {
  text-align: center;
}

.td-date {
  min-width: 100px;
}

.date-cell {
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.date-day {
  font-size: 16px;
  font-weight: 700;
  color: var(--gray-800);
  line-height: 1;
}

.date-month-year {
  font-size: 11px;
  color: var(--gray-500);
}

.interest {
  color: #d97706;
  font-weight: 500;
}

.installment {
  color: var(--primary);
  font-weight: 600;
}

/* Table Footer */
.table-footer {
  background: var(--gray-800) !important;
  border-top: 2px solid var(--gray-700) !important;
}

.tf-label {
  padding: 14px 16px;
  font-weight: 700;
  color: var(--gray-300);
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.tf-value {
  text-align: right;
  font-weight: 700;
  color: white;
  font-size: 13px;
  font-variant-numeric: tabular-nums;
}

/* Status Badges */
.status-badge {
  display: inline-flex;
  align-items: center;
  padding: 3px 8px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 600;
  white-space: nowrap;
}

.status-overdue {
  background: var(--danger-light);
  color: var(--danger);
}

.status-due-soon {
  background: var(--warning-light);
  color: var(--warning);
}

.status-upcoming {
  background: var(--primary-light);
  color: var(--primary);
}

.status-future {
  background: var(--gray-100);
  color: var(--gray-500);
}

/* Spinner */
.spinner {
  width: 16px;
  height: 16px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Transitions */
.slide-up-enter-active {
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

.slide-up-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

/* Footer */
.app-footer {
  text-align: center;
  padding: 24px 0;
  color: rgba(255, 255, 255, 0.6);
  font-size: 13px;
}

/* Responsive */
@media (max-width: 480px) {
  .breakdown-grid {
    grid-template-columns: 1fr;
  }

  .summary-bar {
    flex-direction: column;
  }

  .summary-divider {
    width: 100%;
    height: 1px;
  }

  .form-actions {
    flex-direction: column;
  }

  .btn-lg {
    width: 100%;
    justify-content: center;
  }
}
</style>
