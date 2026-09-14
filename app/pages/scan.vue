<template>
  <v-app class="dashboard-app">
    <v-navigation-drawer v-model="drawer" class="dashboard-drawer" width="250">
      <div class="brand-mark"></div>
      <div class="profile-mini">
        <v-avatar color="primary" size="42"><v-img v-if="user?.picture && !imageFailed" :src="user.picture" alt="Profile image" @error="imageFailed = true" /><span v-else>{{ initials }}</span></v-avatar>
        <div class="profile-mini-copy"><strong>{{ user?.name || 'Guest User' }}</strong><small>{{ user?.email || 'Not signed in' }}</small></div>
      </div>
      <v-list class="nav-list" nav density="comfortable">
        <v-list-item prepend-icon="mdi-view-dashboard-outline" title="Welcome" to="/" exact />
        <v-list-item prepend-icon="mdi-qrcode-scan" title="Scan" to="/scan" />
        <v-list-item prepend-icon="mdi-camera-outline" title="HTML 5 QR Scanner" to="/htmlscanner" />
      </v-list>
      <template #append><div class="drawer-footer"><v-btn block variant="text" prepend-icon="mdi-logout" @click="logout">Logout</v-btn><small>© 2026</small></div></template>
    </v-navigation-drawer>
    <v-app-bar class="dashboard-bar" elevation="0"><v-app-bar-nav-icon @click="drawer = !drawer" /><v-spacer /></v-app-bar>
    <v-main class="dashboard-main"><v-container class="dashboard-container"><section class="scanner-panel">
      <h1>QR Code Image Scanner</h1><div class="scanner-icon"><v-icon size="54">mdi-image-search-outline</v-icon></div><p class="scanner-status">{{ isScanning ? 'Reading image...' : 'Upload an image containing a QR code' }}</p>
      <input ref="fileInput" class="file-input" type="file" accept="image/*" @change="scanUploadedImage" />
      <v-btn block size="large" color="primary" prepend-icon="mdi-upload" :loading="isScanning" @click="chooseImage">Upload QR Image</v-btn>
      <div id="qr-reader" class="reader" />
      <div class="result-box">{{ result || 'Scan Result' }}</div><v-divider class="my-5" /><h2>Scan History</h2>
      <div v-if="history.length" class="history-list"><div v-for="entry in history" :key="entry.id" class="history-entry"><strong>{{ entry.text }}</strong><small>{{ entry.time }}</small></div></div><p v-else class="empty-history">No scans yet</p>
      <v-alert v-if="errorMessage" class="mt-4" type="error" variant="tonal">{{ errorMessage }}</v-alert>
    </section></v-container></v-main>
  </v-app>
</template>

<script setup lang="ts">
import type { Html5Qrcode } from 'html5-qrcode'
const drawer = ref(true)
const user = ref<any>(null)
const imageFailed = ref(false)
const isScanning = ref(false)
const result = ref('')
const errorMessage = ref('')
const history = ref<{ id: number; text: string; time: string }[]>([])
const fileInput = ref<HTMLInputElement | null>(null)
let scanner: Html5Qrcode | null = null
const initials = computed(() => (user.value?.name || 'Guest User').split(' ').map((part: string) => part[0]).join('').slice(0, 2).toUpperCase())
onMounted(() => { const savedUser = localStorage.getItem('google_user'); if (savedUser) user.value = JSON.parse(savedUser); const savedHistory = localStorage.getItem('scan_history'); if (savedHistory) history.value = JSON.parse(savedHistory) })
const logout = () => { localStorage.removeItem('google_user'); localStorage.removeItem('google_token'); navigateTo('/login') }
const chooseImage = () => fileInput.value?.click()
const scanUploadedImage = async (event: Event) => {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  if (!file) return
  errorMessage.value = ''
  result.value = ''
  isScanning.value = true
  try {
    const { Html5Qrcode } = await import('html5-qrcode')
    scanner = new Html5Qrcode('qr-reader')
    const decodedText = await scanner.scanFile(file, true)
    result.value = decodedText
    const entry = { id: Date.now(), text: decodedText, time: new Date().toLocaleString() }
    history.value = [entry, ...history.value].slice(0, 10)
    localStorage.setItem('scan_history', JSON.stringify(history.value))
  } catch {
    errorMessage.value = 'No QR code was found in that image. Please choose a clearer QR image.'
  } finally {
    scanner?.clear()
    scanner = null
    isScanning.value = false
    input.value = ''
  }
}
</script>

<style scoped>
.dashboard-app,.dashboard-main{background:#111;color:#f5f5f5}.dashboard-drawer,.dashboard-bar{background:#202020;color:#ddd;border-color:#303030}.brand-mark{display:flex;align-items:center;gap:10px;height:70px;padding:0 22px;font-size:14px;font-weight:700}.brand-mark span{display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#6c38c5;color:#fff;font-size:11px}.profile-mini{display:flex;align-items:center;gap:11px;padding:18px 20px;border-block:1px solid #303030}.profile-mini-copy{min-width:0;display:grid;gap:3px}.profile-mini-copy strong,.profile-mini-copy small{overflow:hidden;text-overflow:ellipsis;white-space:nowrap}.profile-mini-copy strong{font-size:13px}.profile-mini-copy small{color:#999;font-size:10px}.nav-list{padding:18px 10px}.nav-list :deep(.v-list-item){color:#aaa;margin:3px 0}.nav-list :deep(.v-list-item--active){color:#fff;background:#3a3a3a}.drawer-footer{padding:12px;color:#aaa;text-align:center}.drawer-footer small{display:block;padding-top:16px;font-size:10px}.dashboard-bar :deep(.v-toolbar-title){font-size:14px}.dashboard-container{max-width:1100px;padding:34px 28px 64px}.scanner-panel{width:min(100%,520px);margin:0 auto;padding:26px;background:#202020;border:1px solid #2e2e2e;color:#eee}.scanner-panel h1{text-align:center;font-size:20px;font-weight:500}.scanner-icon{display:grid;place-items:center;color:#aaa}.scanner-status{text-align:center;color:#aaa;font-size:13px}.file-input{display:none}.reader{display:none}.result-box{min-height:70px;display:grid;place-items:center;margin-top:14px;padding:16px;border:1px solid #333;color:#aaa;text-align:center;overflow-wrap:anywhere}.scanner-panel h2{font-size:14px;font-weight:500}.history-list{display:grid;gap:14px}.history-entry{display:grid;gap:5px;padding-bottom:14px;border-bottom:1px solid #333}.history-entry strong{font-size:14px;overflow-wrap:anywhere}.history-entry small,.empty-history{color:#aaa;font-size:12px}.empty-history{text-align:center}
</style>