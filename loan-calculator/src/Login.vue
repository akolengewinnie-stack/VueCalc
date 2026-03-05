<template>
  <div class="login-container">
    <!-- Header -->
    <header class="app-header">
      <div class="header-content">
        <div class="header-icon">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="28" height="28">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1.41 16.09V20h-2.67v-1.93c-1.71-.36-3.16-1.46-3.27-3.4h1.96c.1 1.05.82 1.87 2.65 1.87 1.96 0 2.4-.98 2.4-1.59 0-.83-.44-1.61-2.67-2.14-2.48-.6-4.18-1.62-4.18-3.67 0-1.72 1.39-2.84 3.11-3.21V4h2.67v1.95c1.86.45 2.79 1.86 2.85 3.39H14.3c-.05-1.11-.64-1.87-2.22-1.87-1.5 0-2.4.68-2.4 1.64 0 .84.65 1.39 2.67 1.91s4.18 1.39 4.18 3.91c-.01 1.83-1.38 2.83-3.12 3.16z"/>
          </svg>
        </div>
        <div>
          <h1 class="header-title">Loan Calculator</h1>
          <p class="header-subtitle">Powered by PesaPal Credit</p>
        </div>
      </div>
    </header>

    <!-- Login Card -->
    <div class="login-card">
      <div class="login-card-header">
        <div class="login-avatar">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="32" height="32">
            <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
          </svg>
        </div>
        <h2 class="login-title">Welcome Back</h2>
        <p class="login-subtitle">Sign in to access the loan calculator</p>
      </div>

      <div class="login-card-body">
        <!-- Error Alert -->
        <div v-if="loginError" class="alert alert-error">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
          </svg>
          <span>{{ loginError }}</span>
        </div>

        <!-- Login Form -->
        <form @submit.prevent="handleLogin" class="login-form">
          <!-- Username / Email -->
          <div class="form-group">
            <label class="form-label" for="username">
              Username / Email
              <span class="required">*</span>
            </label>
            <div class="input-wrapper">
              <span class="input-icon">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
                  <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
                </svg>
              </span>
              <input
                id="username"
                v-model="username"
                type="text"
                class="form-input with-icon"
                :class="{ 'is-error': usernameError }"
                placeholder="Enter your username or email"
                autocomplete="username"
                :disabled="loading"
              />
            </div>
            <p v-if="usernameError" class="form-error">{{ usernameError }}</p>
          </div>

          <!-- Password -->
          <div class="form-group">
            <label class="form-label" for="password">
              Password
              <span class="required">*</span>
            </label>
            <div class="input-wrapper">
              <span class="input-icon">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
                  <path d="M18 8h-1V6c0-2.76-2.24-5-5-5S7 3.24 7 6v2H6c-1.1 0-2 .9-2 2v10c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V10c0-1.1-.9-2-2-2zm-6 9c-1.1 0-2-.9-2-2s.9-2 2-2 2 .9 2 2-.9 2-2 2zm3.1-9H8.9V6c0-1.71 1.39-3.1 3.1-3.1 1.71 0 3.1 1.39 3.1 3.1v2z"/>
                </svg>
              </span>
              <input
                id="password"
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                class="form-input with-icon with-suffix"
                :class="{ 'is-error': passwordError }"
                placeholder="Enter your password"
                autocomplete="current-password"
                :disabled="loading"
              />
              <button
                type="button"
                class="input-toggle-btn"
                @click="showPassword = !showPassword"
                :disabled="loading"
              >
                <svg v-if="!showPassword" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
                  <path d="M12 4.5C7 4.5 2.73 7.61 1 12c1.73 4.39 6 7.5 11 7.5s9.27-3.11 11-7.5c-1.73-4.39-6-7.5-11-7.5zM12 17c-2.76 0-5-2.24-5-5s2.24-5 5-5 5 2.24 5 5-2.24 5-5 5zm0-8c-1.66 0-3 1.34-3 3s1.34 3 3 3 3-1.34 3-3-1.34-3-3-3z"/>
                </svg>
                <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="18" height="18">
                  <path d="M12 7c2.76 0 5 2.24 5 5 0 .65-.13 1.26-.36 1.83l2.92 2.92c1.51-1.26 2.7-2.89 3.43-4.75-1.73-4.39-6-7.5-11-7.5-1.4 0-2.74.25-3.98.7l2.16 2.16C10.74 7.13 11.35 7 12 7zM2 4.27l2.28 2.28.46.46C3.08 8.3 1.78 10.02 1 12c1.73 4.39 6 7.5 11 7.5 1.55 0 3.03-.3 4.38-.84l.42.42L19.73 22 21 20.73 3.27 3 2 4.27zM7.53 9.8l1.55 1.55c-.05.21-.08.43-.08.65 0 1.66 1.34 3 3 3 .22 0 .44-.03.65-.08l1.55 1.55c-.67.33-1.41.53-2.2.53-2.76 0-5-2.24-5-5 0-.79.2-1.53.53-2.2zm4.31-.78l3.15 3.15.02-.16c0-1.66-1.34-3-3-3l-.17.01z"/>
                </svg>
              </button>
            </div>
            <p v-if="passwordError" class="form-error">{{ passwordError }}</p>
          </div>

          <!-- Submit Button -->
          <button
            type="submit"
            class="btn btn-primary btn-lg btn-full"
            :disabled="loading"
          >
            <svg v-if="loading" class="spinner-sm" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="20" height="20">
              <path d="M11 7L9.6 8.4l2.6 2.6H2v2h10.2l-2.6 2.6L11 17l5-5-5-5zm9 12h-8v2h8c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2h-8v2h8v14z"/>
            </svg>
            {{ loading ? 'Signing in...' : 'Sign In' }}
          </button>
        </form>
      </div>
    </div>

    <!-- Footer -->
    <footer class="app-footer">
      <p>Loan Calculator &copy; {{ new Date().getFullYear() }} · Powered by PesaPal Credit</p>
    </footer>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Login',
  emits: ['login-success'],
  data() {
    return {
      username: '',
      password: '',
      showPassword: false,
      loading: false,
      loginError: null,
      usernameError: null,
      passwordError: null,
    }
  },
  methods: {
    validateForm() {
      let valid = true
      this.usernameError = null
      this.passwordError = null

      if (!this.username.trim()) {
        this.usernameError = 'Please enter your username or email'
        valid = false
      }

      if (!this.password) {
        this.passwordError = 'Please enter your password'
        valid = false
      }

      return valid
    },

    async handleLogin() {
      if (!this.validateForm()) return

      this.loading = true
      this.loginError = null

      try {
        const response = await axios.post(
          'https://mwdev.pesapal.credit/api/merchant/login',
          {
            username: this.username,
            password: this.password,
          },
          {
            timeout: 15000,
            headers: {
              'Content-Type': 'application/json',
              'Accept': 'application/json',
            },
          }
        )

        const data = response.data

        // Extract token from various possible response structures
        const token =
          data?.token ||
          data?.access_token ||
          data?.data?.token ||
          data?.data?.access_token ||
          null

        if (token) {
          // Store token in sessionStorage
          sessionStorage.setItem('auth_token', token)
          sessionStorage.setItem('auth_user', JSON.stringify(data?.user || data?.data?.user || { username: this.username }))
          this.$emit('login-success', { token, user: data?.user || data?.data?.user || { username: this.username } })
        } else if (response.status === 200 || response.status === 201) {
          // Login succeeded but no token in expected fields — store raw data and proceed
          sessionStorage.setItem('auth_token', 'session')
          sessionStorage.setItem('auth_user', JSON.stringify({ username: this.username }))
          this.$emit('login-success', { token: 'session', user: { username: this.username } })
        } else {
          this.loginError = 'Login failed. Please check your credentials and try again.'
        }
      } catch (error) {
        console.error('Login error:', error)
        if (error.response) {
          const status = error.response.status
          const msg = error.response.data?.message || error.response.data?.error || null
          if (status === 401 || status === 403) {
            this.loginError = msg || 'Invalid username or password. Please try again.'
          } else if (status === 422) {
            this.loginError = msg || 'Invalid input. Please check your credentials.'
          } else if (status >= 500) {
            this.loginError = 'Server error. Please try again later.'
          } else {
            this.loginError = msg || 'Login failed. Please try again.'
          }
        } else if (error.code === 'ECONNABORTED') {
          this.loginError = 'Request timed out. Please check your connection and try again.'
        } else {
          this.loginError = 'Unable to connect. Please check your internet connection.'
        }
      } finally {
        this.loading = false
      }
    },
  },
}
</script>

<style scoped>
.login-container {
  max-width: 480px;
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

/* Login Card */
.login-card {
  background: var(--white);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  overflow: hidden;
}

.login-card-header {
  padding: 32px 32px 24px;
  text-align: center;
  border-bottom: 1px solid var(--gray-100);
  background: var(--gray-50);
}

.login-avatar {
  width: 64px;
  height: 64px;
  background: var(--primary-light);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--primary);
  margin: 0 auto 16px;
}

.login-title {
  font-size: 22px;
  font-weight: 700;
  color: var(--gray-800);
  margin-bottom: 6px;
}

.login-subtitle {
  font-size: 14px;
  color: var(--gray-500);
}

.login-card-body {
  padding: 28px 32px 32px;
}

/* Alert */
.alert {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  padding: 12px 16px;
  border-radius: var(--radius);
  margin-bottom: 20px;
  font-size: 14px;
  font-weight: 500;
}

.alert-error {
  background: var(--danger-light);
  color: var(--danger);
  border: 1px solid rgba(224, 36, 36, 0.2);
}

.alert svg {
  flex-shrink: 0;
  margin-top: 1px;
}

/* Form */
.login-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-label {
  font-size: 14px;
  font-weight: 600;
  color: var(--gray-700);
  display: flex;
  align-items: center;
  gap: 4px;
}

.required {
  color: var(--danger);
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon {
  position: absolute;
  left: 12px;
  color: var(--gray-400);
  display: flex;
  align-items: center;
  pointer-events: none;
  z-index: 1;
}

.form-input {
  width: 100%;
  padding: 10px 14px;
  border: 1.5px solid var(--gray-200);
  border-radius: var(--radius);
  font-size: 15px;
  color: var(--gray-800);
  background: var(--white);
  transition: border-color 0.15s, box-shadow 0.15s;
  outline: none;
}

.form-input.with-icon {
  padding-left: 42px;
}

.form-input.with-suffix {
  padding-right: 44px;
}

.form-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(26, 86, 219, 0.12);
}

.form-input.is-error {
  border-color: var(--danger);
}

.form-input.is-error:focus {
  box-shadow: 0 0 0 3px rgba(224, 36, 36, 0.12);
}

.form-input:disabled {
  background: var(--gray-50);
  color: var(--gray-400);
  cursor: not-allowed;
}

.input-toggle-btn {
  position: absolute;
  right: 12px;
  background: none;
  border: none;
  cursor: pointer;
  color: var(--gray-400);
  display: flex;
  align-items: center;
  padding: 2px;
  border-radius: 4px;
  transition: color 0.15s;
}

.input-toggle-btn:hover {
  color: var(--gray-600);
}

.input-toggle-btn:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

.form-error {
  font-size: 13px;
  color: var(--danger);
  display: flex;
  align-items: center;
  gap: 4px;
}

/* Buttons */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 10px 20px;
  border-radius: var(--radius);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.15s;
  text-decoration: none;
}

.btn-primary {
  background: var(--primary);
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: var(--primary-dark);
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

.btn-primary:disabled {
  opacity: 0.65;
  cursor: not-allowed;
  transform: none;
}

.btn-lg {
  padding: 13px 24px;
  font-size: 15px;
}

.btn-full {
  width: 100%;
}

/* Spinner */
.spinner-sm {
  width: 18px;
  height: 18px;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Footer */
.app-footer {
  text-align: center;
  padding: 20px 0 8px;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.65);
}
</style>
