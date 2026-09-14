<template>
  <v-app>
    <v-main class="login-page">
      <v-container class="fill-height d-flex align-center justify-center pa-4">
        <v-card class="login-card" width="672" elevation="8">
          <v-card-title class="text-center text-h3 font-weight-bold pa-0 mb-3">
            LOGIN
          </v-card-title>
          <v-card-subtitle class="text-center text-body-1 mb-7">
            Welcome back, please login
          </v-card-subtitle>

          <v-card-text class="pa-0">
            <v-text-field
              v-model="email"
              class="login-input mb-4"
              placeholder="Email"
              type="email"
              prepend-inner-icon="mdi-email"
              variant="solo"
              flat
              hide-details
            />
            <v-text-field
              v-model="password"
              class="login-input mb-7"
              placeholder="Password"
              type="password"
              prepend-inner-icon="mdi-lock"
              variant="solo"
              flat
              hide-details
            />

            <v-btn block class="email-button mb-6" size="large" @click="emailLoginMessage">
              Sign In
            </v-btn>

            <div class="or-divider mb-6"><span>OR</span></div>

            <v-btn
              block
              class="google-button"
              :loading="isSigningIn"
              size="large"
              prepend-icon="mdi-google"
              @click="loginWithGoogle"
            >
              Sign in with google
            </v-btn>
            <v-alert v-if="errorMessage" class="mt-4" type="error" variant="tonal">
              {{ errorMessage }}
            </v-alert>
          </v-card-text>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script lang="ts" setup>
// @ts-nocheck
const config = useRuntimeConfig()
const email = ref('')
const password = ref('')
const isSigningIn = ref(false)
const errorMessage = ref('')

declare global {
  interface Window {
    google: any
  }
}

const waitForGoogle = () => new Promise((resolve, reject) => {
  const startedAt = Date.now()
  const check = () => {
    if (window.google?.accounts?.oauth2) return resolve(window.google)
    if (Date.now() - startedAt > 10000) return reject(new Error('Google sign-in could not be loaded.'))
    window.setTimeout(check, 100)
  }
  check()
})

const loginWithGoogle = async () => {
  errorMessage.value = ''
  isSigningIn.value = true

  if (!config.public.googleClientId) {
    errorMessage.value = 'Google sign-in is not configured yet.'
    isSigningIn.value = false
    return
  }

  try {
    await waitForGoogle()
    const client = window.google.accounts.oauth2.initTokenClient({
      client_id: config.public.googleClientId,
      scope: 'openid email profile',
      callback: async (response: any) => {
        try {
          const userInfo = await $fetch('https://www.googleapis.com/oauth2/v3/userinfo', {
            headers: {
              Authorization: `Bearer ${response.access_token}`,
            },
          })

          localStorage.setItem('google_user', JSON.stringify(userInfo))
          localStorage.setItem('google_token', response.access_token)
          await navigateTo('/')
        } catch {
          errorMessage.value = 'Google sign-in could not load your profile.'
          isSigningIn.value = false
        }
      },
      error_callback: () => {
        errorMessage.value = 'Google sign-in was cancelled or could not be completed.'
        isSigningIn.value = false
      },
    })

    client.requestAccessToken()
  } catch (error: any) {
    errorMessage.value = error.message || 'Google sign-in could not be completed.'
    isSigningIn.value = false
  }
}

const emailLoginMessage = () => {
  errorMessage.value = 'Email and password sign-in is not connected yet. Use Google sign-in.'
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  background: #fff;
}

.login-card {
  width: min(672px, calc(100vw - 48px));
  min-height: 750px;
  padding: 86px 40px 78px !important;
  border: 1px solid #eeeeee;
  border-radius: 28px;
  box-shadow: 0 12px 28px rgba(0, 0, 0, 0.18) !important;
}

.login-card :deep(.v-card-title) {
  color: #111111;
  font-size: 42px !important;
  font-weight: 700 !important;
  letter-spacing: 0;
  line-height: 1.2;
}

.login-card :deep(.v-card-subtitle) {
  color: #333333;
  font-size: 22px !important;
  font-weight: 400;
  line-height: 1.35;
  opacity: 1;
}

.login-input :deep(.v-field) {
  min-height: 72px;
  border-radius: 0;
  background: #f4f4f4;
}

.login-input :deep(input) {
  font-size: 21px;
}

.email-button {
  min-height: 52px;
  color: #4c82b8;
  background: #e3ebf8;
  font-size: 20px;
}

.google-button {
  min-height: 54px;
  color: #ffffff;
  background: #ed1c24;
  font-size: 20px;
}

.or-divider {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #333333;
  font-size: 21px;
}

@media (max-width: 600px) {
  .login-card {
    width: calc(100vw - 32px);
    min-height: 680px;
    padding: 56px 20px 48px !important;
  }
}
</style>