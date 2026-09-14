<template>
  <v-app class="dashboard-app">
    <v-navigation-drawer v-model="drawer" class="dashboard-drawer" width="250">
      <div class="brand-mark"></div>
      <div class="profile-mini"><v-avatar color="primary" size="42">
        <v-img v-if="user?.picture && !imageFailed" :src="user.picture" @error="imageFailed = true" />
        <span v-else>{{ initials }}</span></v-avatar>
        <div class="profile-mini-copy"><strong>{{ user?.name || 'Guest User' }}</strong><small>{{ user?.email || 'Not signed in' }}</small></div></div><v-list class="nav-list" nav density="comfortable"><v-list-item prepend-icon="mdi-view-dashboard-outline" title="Welcome" to="/" exact /><v-list-item prepend-icon="mdi-qrcode-scan" title="Scan" to="/scan" /><v-list-item prepend-icon="mdi-camera-outline" title="HTML 5 QR Scanner" to="/htmlscanner" /></v-list><template #append><div class="drawer-footer"><v-btn block variant="text" prepend-icon="mdi-logout" @click="logout">Logout</v-btn><small>© 2026</small></div></template></v-navigation-drawer>
    <v-app-bar class="dashboard-bar" elevation="0"><v-app-bar-nav-icon @click="drawer = !drawer" /><v-spacer /></v-app-bar>
    <v-main class="dashboard-main">
      <v-container class="dashboard-container">
        <v-card v-if="user" class="profile-card pa-6">
          <div class="d-flex align-center ga-4">
            <v-avatar size="64">
              <v-img :src="user.picture" />
            </v-avatar>
            <div>
              <h2>{{ user.name }}</h2>
              <p>{{ user.email }}</p>
            </div>
          </div>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
const user = ref<any>(null)
const imageFailed = ref(false)
const drawer = ref(true)

const initials = computed(() => (user.value?.name || 'User')
  .split(' ').map((part: string) => part[0]).join('').slice(0, 2).toUpperCase())

onMounted(() => {
  const savedUser = localStorage.getItem('google_user')
  if (savedUser) {
    user.value = JSON.parse(savedUser)
  }
})

const logout = () => { localStorage.removeItem('google_user'); localStorage.removeItem('google_token'); navigateTo('/login') }

</script>

<style scoped>
.dashboard-app,.dashboard-main{background:#111;color:#f5f5f5}.dashboard-drawer,.dashboard-bar{background:#202020;color:#ddd;border-color:#303030}.brand-mark{display:flex;align-items:center;gap:10px;height:70px;padding:0 22px;font-size:14px;font-weight:700}.brand-mark span{display:grid;place-items:center;width:32px;height:32px;border-radius:50%;background:#6c38c5;color:#fff;font-size:11px}.profile-mini{display:flex;align-items:center;gap:11px;padding:18px 20px;border-block:1px solid #303030}.profile-mini-copy{min-width:0;display:grid;gap:3px}.profile-mini-copy strong,.profile-mini-copy small{overflow:hidden;text-overflow:ellipsis;white-space:nowrap}.profile-mini-copy strong{font-size:13px}.profile-mini-copy small{color:#999;font-size:10px}.nav-list{padding:18px 10px}.nav-list :deep(.v-list-item){color:#aaa;margin:3px 0}.nav-list :deep(.v-list-item--active){color:#fff;background:#3a3a3a}.drawer-footer{padding:12px;color:#aaa;text-align:center}.drawer-footer small{display:block;padding-top:16px;font-size:10px}.dashboard-bar :deep(.v-toolbar-title){font-size:14px}.dashboard-container{max-width:1100px;padding:34px 28px 64px}.welcome-header{margin-bottom:28px}.eyebrow,.profile-label{color:#8e6ed0;font-size:11px;font-weight:700;letter-spacing:.14em}.welcome-header h1{margin:8px 0;font-size:clamp(27px,4vw,38px);font-weight:500}.welcome-header p:last-child{color:#999}.profile-card,.quick-card{background:#202020;border:1px solid #303030;color:#f5f5f5}.profile-content{display:flex;align-items:center;gap:22px;padding:28px}.profile-label{margin:0 0 7px}.profile-content h2{margin:0 0 5px;font-size:23px;font-weight:500}.email{margin:0;color:#aaa}.initials{font-size:25px;font-weight:700}.quick-links{display:grid;grid-template-columns:repeat(2,1fr);gap:18px;margin-top:22px}.quick-card{display:flex;align-items:center;gap:17px;padding:24px;cursor:pointer}.quick-card h3{margin:0 0 5px;font-size:16px;font-weight:500}.quick-card p{margin:0;color:#999;font-size:12px}.quick-card .arrow{margin-left:auto;color:#888}@media(max-width:600px){.quick-links{grid-template-columns:1fr}.profile-content{padding:22px}}
</style>