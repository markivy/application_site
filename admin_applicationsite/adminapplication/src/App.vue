<template>
  <div class="app">
    <!-- Animated Background -->
    <div class="background">
      <div class="gradient-orb orb-1"></div>
      <div class="gradient-orb orb-2"></div>
      <div class="gradient-orb orb-3"></div>
    </div>

    <!-- Navigation -->
    <nav class="navbar">
      <div class="nav-container">
        <div class="nav-brand">
          <div class="brand-icon"></div>
          <h1 class="brand-text">Xisto</h1>
        </div>
        <div class="nav-tabs">
          <div v-show="submitMessagevisibility" class="action-btn danger" name="SubmitMessage">
            <span>
              {{ submitMessage.text}}
            </span>
          </div>
          <button ref="loginbtn" v-show="currentView !== 'admin'" @click="currentView = 'login'"
            :class="['nav-tab', { active: currentView === 'login' }]">
            <span class="tab-text">Login</span>
            <div class="tab-indicator"></div>
          </button>
          <!-- <button v-if="authToken" @click="removeToken">
            remove token
          </button> -->
          <button ref="adminbtn" v-show="currentView !== 'login'" @click="currentView = 'admin'"
            :class="['nav-tab', { active: currentView === 'admin' }]">
            <span class="tab-text">Dashboard</span>
            <div class="tab-indicator"></div>
          </button>
          <button v-show="currentView == 'admin'" @click="removeToken()" class="nav-tab">Sair</button>
        </div>
      </div>
    </nav>



    <!-- Login View -->
    <div v-if="currentView === 'login'" class="main-content">
      <div class="login-container">
        <h1 class="login-title">Login</h1>
      </div>
      <div class="login-form">
        <div class="form-group">
          <label class="form-label">Email</label>
          <input v-model="email" name="email" type="text" class="modern-input" placeholder="digite seu email"
            required />
        </div>
        <div class="form-group">
          <label class="form-label">Senha</label>
          <input v-model="password" name="password" type="password" class="modern-input" placeholder="digite sua senha"
            required />
        </div>
        <button type="submit" class="submit-btn" @click="sendLogin">Login</button>
      </div>
    </div>

    <!-- Admin Dashboard View -->
    <div v-if="currentView === 'admin'" class="main-content">
      <div class="dashboard-header">
        <h1 class="dashboard-title">Painel de Candidaturas</h1>
        <div class="stats-card">
          <div class="stat-item">
            <span class="stat-number">{{ applications.length }}</span>
            <span class="stat-label">Total de curriculos</span>
          </div>
        </div>
      </div>

      <div class="applications-container">
        <div v-if="applications.length === 0" class="empty-state">
          <div class="empty-icon">
            <svg width="64" height="64" viewBox="0 0 24 24" fill="none">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" stroke="currentColor"
                stroke-width="1.5" />
              <polyline points="14,2 14,8 20,8" stroke="currentColor" stroke-width="1.5" />
              <line x1="16" y1="13" x2="8" y2="13" stroke="currentColor" stroke-width="1.5" />
              <line x1="16" y1="17" x2="8" y2="17" stroke="currentColor" stroke-width="1.5" />
            </svg>
          </div>
          <h3 class="empty-title">Sem curriculos novos</h3>
          <p class="empty-description">Os curriculos irão aparecer aqui quando as pessoas aplicarem.</p>
        </div>

        <div v-else class="applications-grid">
          <div v-for="(application, index) in applications" :key="application.index" class="application-card"
            :style="{ animationDelay: `${index * 0.1}s` }">
            <div class="card-header">
              <div class="applicant-info">
                <h3 class="applicant-name">{{ application.nome }}</h3>
                <p class="applicant-contact">{{ application.numero }}</p>
              </div>
              <div class="application-meta">
                <span class="application-date">{{ formatDate(application.CreatedAt) }}</span>
              </div>
            </div>

            <div class="card-body">
              <div class="file-info">
                <div class="file-icon">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                    <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" stroke="currentColor"
                      stroke-width="2" />
                    <polyline points="14,2 14,8 20,8" stroke="currentColor" stroke-width="2" />
                  </svg>
                </div>
                <span class="file-name">{{ application.nomearquivo }}</span>
              </div>
            </div>

            <div class="card-actions">
              <button @click="downloadCV(application)" class="action-btn primary">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                  <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" stroke="currentColor" stroke-width="2" />
                  <polyline points="7,10 12,15 17,10" stroke="currentColor" stroke-width="2" />
                  <line x1="12" y1="15" x2="12" y2="3" stroke="currentColor" stroke-width="2" />
                </svg>
                Download
              </button>
              <button @click="deleteApplication(application.id)" class="action-btn danger">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                  <polyline points="3,6 5,6 21,6" stroke="currentColor" stroke-width="2" />
                  <path d="m19,6v14a2,2 0 0,1 -2,2H7a2,2 0 0,1 -2,-2V6m3,0V4a2,2 0 0,1 2,-2h4a2,2 0 0,1 2,2v2"
                    stroke="currentColor" stroke-width="2" />
                </svg>
                Delete
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-if="currentView === 'loading'" class="loading-view">
    <div class="loading-spinner">
      <svg class="spinner" width="64" height="64" viewBox="0 0 50 50">
        <circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5"></circle>
      </svg>
      <div class="loading-text">
        Carregando...
        <span class="dot dot1">.</span>
        <span class="dot dot2">.</span>
        <span class="dot dot3">.</span>
      </div>
    </div>
  </div>


</template>

<script setup>
import { ref, reactive, onMounted, watch, computed } from 'vue'

const currentView = ref('loading')
const authToken = ref(localStorage.getItem('authToken'))
console.log('authToken', authToken.value)
const submitMessage = ref('text', 'type')
const applications = ref([])
const email = ref('')
const password = ref('')

const loginbtn = ref(null)
const adminbtn = ref(null)

watch(currentView, (newView) => {
  if (newView === 'admin') {
    console.log('loadApplications')
    loadApplications()
  }
  if (newView === 'login') {
    console.log('existsvalidtoken')
    existsvalidtoken()
  }
})

const existsvalidtoken = async () => {
  const token = localStorage.getItem('authToken')
  if (!token) {
    currentView.value = 'login'
    return
  }
  try {
    const response = await fetch('http://147.79.104.229:5678/webhook/existsvalidtoken', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        payload: {
          token: token
        },
        action: 'existsvalidtoken'
      })
    })
    if (!response.ok) {
      throw new Error('Erro na requisição')
    }
    const data = await response.status
    console.log('data', data)
    if (data === 200) {
      console.log('existsvalidtoken ok')
      currentView.value = 'admin'
    } else {
      console.log('existsvalidtoken not ok')
      currentView.value = 'login'
    }
  } catch (error) {
    console.error('Erro ao verificar o token:', error)
    currentView.value = 'login'
  }
}
const removeToken = () => {
  localStorage.removeItem('authToken')
  authToken.value = null
  currentView.value = 'login'
}

const submitMessagevisibility = ref(false)
const showMessage = (text, type) => {
  submitMessagevisibility.value = true
  submitMessage.value = { text, type }
  setTimeout(() => {
    submitMessage.value = "",""
    submitMessagevisibility.value = false
  }, 5000)
}

const loadApplications = async () => {

  if (!authToken.value) {
    currentView.value = 'login'
    alert('Por favor, faça login para acessar o painel.')
    return
  }
  console.log('authToken', authToken.value)
  const response = await fetch('http://147.79.104.229:5678/webhook/loadapplications', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      payload: {
        token: authToken.value
      },
      action: 'loadapplications'
    })
  }).then(res => {
    if (!res.ok) {
      throw new Error('Erro ao carregar candidaturas')
    }
    return res.json()
  }).then(data => {
    console.log('data', data)
    applications.value = data || []
    currentView.value = 'admin'
  }).catch(error => {
    console.error('Erro ao carregar candidaturas:', error)
    showMessage('Erro ao carregar candidaturas. Por favor, tente novamente.', 'error')
  })
}

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const downloadCV = (application) => {
  const link = document.createElement('a')
  link.href = application.curriculo
  link.download = application.nomearquivo
  link.click()
}

const deleteApplication = (id) => {
  if (confirm('Are you sure you want to delete this application?')) {
    applications.value = applications.value.filter(app => app.id !== id)
    localStorage.setItem('jobApplications', JSON.stringify(applications.value))
  }
}

// Para garantir um login seguro, é importante enviar as credenciais (usuário e senha) de forma segura, preferencialmente sobre HTTPS, e nunca armazenar ou trafegar senhas em texto puro. Além disso, recomenda-se implementar autenticação baseada em tokens (como JWT), aplicar rate limiting, e proteger contra ataques de força bruta. O ideal é que a senha seja enviada já em hash, mas normalmente o hash é feito no backend. Aqui está um exemplo melhorado, mas lembre-se que o endpoint deve ser HTTPS e o backend deve tratar a segurança:
const sendLogin = async () => {
  try {
    // Validação básica no frontend
    if (!email.value || !password.value) {
      console.log('faltando usuario ou senha');
      showMessage('Por favor, preencha usuário e senha.');
      return;
    }

    // --- ATENÇÃO: USE SEMPRE HTTPS EM PRODUÇÃO! ---
    const response = await fetch('http://147.79.104.229:5678/webhook/easyauth', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        payload: {
          email: email.value,
          // A senha é enviada em texto puro via HTTPS para o backend,
          // onde ela deve ser hashed e comparada com o hash armazenado.
          password: password.value
        },
        action: 'login'
      })
    });

    if (!response.ok) {
      let errorData = {};
      try {
        // Tenta parsear o corpo da resposta como JSON.
        // Se o backend enviar HTML ou texto puro em caso de erro, isso falhará.
        errorData = await response.json();
      } catch (jsonParseError) {
        // Captura o erro se o corpo não for um JSON válido.
        console.warn('Erro ao parsear JSON da resposta de erro:', jsonParseError);
        errorData.message = 'Formato de resposta de erro inválido do servidor.';
      }

      // Lança um erro com mais detalhes, usando a mensagem do backend se disponível.
      const errorMessage = errorData.message || 'Erro sem mensagem específica.';
      throw new Error(`${response.status} - ${response.statusText} - ${errorMessage}`);
    }

    const data = await response.json();

    if (data.token) {
      localStorage.setItem('authToken', data.token);
      authToken.value = data.token
      console.log('Login realizado com sucesso!', 'success');
      currentView.value = 'admin';
    } else {
      // Este bloco só será alcançado se response.ok for true (status 2xx)
      // mas o token não estiver presente. Isso pode indicar uma lógica de backend falha
      // ou um tipo de "erro lógico" retornado com status 200.
      console.log('Não foi possível obter o token. Usuário ou senha inválidos ou problema no servidor.', 'error');
    }
  } catch (error) {
    console.error('Erro ao enviar login:', error);
    // showMessage pode exibir '401 - Unauthorized - Usuário ou senha inválidos.'
    // ou 'Erro ao enviar login. Por favor, tente novamente.' para erros de rede.
    console.log(`Erro no login: ${error.message || 'Por favor, tente novamente.'}`, 'error');
  }
};

onMounted(() => {
  if (authToken.value) {
    currentView.value = 'admin'
  } else {
    currentView.value = 'login'
  }
})
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.app {
  min-height: 100vh;
  background: #000;
  color: #fff;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
  position: relative;
  overflow-x: hidden;
}

/* Animated Background */
.background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}

.gradient-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
  opacity: 0.3;
  animation: float 20s ease-in-out infinite;
}

.orb-1 {
  width: 400px;
  height: 400px;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  top: -200px;
  left: -200px;
  animation-delay: 0s;
}

.orb-2 {
  width: 300px;
  height: 300px;
  background: linear-gradient(45deg, #a8e6cf, #ffd93d);
  top: 50%;
  right: -150px;
  animation-delay: -10s;
}

.orb-3 {
  width: 500px;
  height: 500px;
  background: linear-gradient(45deg, #ff8a80, #82b1ff);
  bottom: -250px;
  left: 50%;
  transform: translateX(-50%);
  animation-delay: -5s;
}

@keyframes float {

  0%,
  100% {
    transform: translateY(0px) rotate(0deg);
  }

  33% {
    transform: translateY(-30px) rotate(120deg);
  }

  66% {
    transform: translateY(20px) rotate(240deg);
  }
}

/* Navigation */
.navbar {
  position: sticky;
  top: 0;
  z-index: 100;
  backdrop-filter: blur(20px);
  background: rgba(0, 0, 0, 0.8);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.nav-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.nav-brand {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.brand-icon {
  width: 32px;
  height: 32px;
  background: linear-gradient(45deg, #fff, #888);
  border-radius: 8px;
}

.brand-text {
  font-size: 1.5rem;
  font-weight: 700;
  margin: 0;
  background: linear-gradient(45deg, #fff, #888);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.nav-tabs {
  display: flex;
  gap: 0.5rem;
}

.nav-tab {
  position: relative;
  background: none;
  border: none;
  color: #888;
  padding: 0.75rem 1.5rem;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 500;
}

.nav-tab:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.05);
}

.nav-tab.active {
  color: #fff;
  background: rgba(255, 255, 255, 0.1);
}

.tab-indicator {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 2px;
  background: linear-gradient(45deg, #fff, #888);
  transition: width 0.3s ease;
}

.nav-tab.active .tab-indicator {
  width: 80%;
}

/* Main Content */
.main-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

/* Hero Section */
.hero-section {
  text-align: center;
  margin-bottom: 4rem;
  padding: 4rem 0;
}

.hero-title {
  font-size: 4rem;
  font-weight: 800;
  margin: 0 0 1rem 0;
  background: linear-gradient(45deg, #fff, #888);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  line-height: 1.1;
}

.hero-subtitle {
  font-size: 1.25rem;
  color: #888;
  margin: 0;
  max-width: 600px;
  margin: 0 auto;
}

/* Form Container */
.form-container {
  max-width: 600px;
  margin: 0 auto;
}

.glass-card {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 24px;
  padding: 3rem;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
}

.application-form {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.file-group {
  grid-column: 1 / -1;
}

.input-label {
  font-weight: 600;
  color: #fff;
  font-size: 0.875rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.input-wrapper {
  position: relative;
}

.modern-input {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 1rem;
  color: #fff;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.modern-input::placeholder {
  color: #666;
}

.modern-input:focus {
  outline: none;
  border-color: rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.08);
}

.input-border {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 2px;
  background: linear-gradient(45deg, #fff, #888);
  transition: width 0.3s ease;
}

.modern-input:focus+.input-border {
  width: 100%;
}

/* File Upload */
.file-upload-zone {
  position: relative;
  border: 2px dashed rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  padding: 2rem;
  text-align: center;
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.02);
}

.file-upload-zone:hover,
.file-upload-zone.drag-over {
  border-color: rgba(255, 255, 255, 0.4);
  background: rgba(255, 255, 255, 0.05);
}

.file-upload-zone.has-file {
  border-color: #4ade80;
  background: rgba(74, 222, 128, 0.1);
}

.file-input {
  position: absolute;
  opacity: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.file-upload-label {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  cursor: pointer;
}

.upload-icon {
  color: #888;
  transition: color 0.3s ease;
}

.file-upload-zone:hover .upload-icon {
  color: #fff;
}

.upload-text {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.upload-primary {
  color: #fff;
  font-weight: 500;
}

.upload-success {
  color: #4ade80;
  font-weight: 600;
}

.upload-secondary {
  color: #666;
  font-size: 0.875rem;
}

/* Submit Button */
.submit-btn {
  position: relative;
  background: linear-gradient(45deg, #fff, #888);
  color: #000;
  border: none;
  border-radius: 12px;
  padding: 1rem 2rem;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  overflow: hidden;
  margin-top: 1rem;
}

.submit-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 30px rgba(255, 255, 255, 0.2);
}

.submit-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-shine {
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  transition: left 0.5s ease;
}

.submit-btn:hover .btn-shine {
  left: 100%;
}

/* Notifications */
.notification {
  margin-top: 1.5rem;
  padding: 1rem;
  border-radius: 12px;
  border: 1px solid;
  animation: slideIn 0.3s ease;
}

.notification.success {
  background: rgba(74, 222, 128, 0.1);
  border-color: rgba(74, 222, 128, 0.3);
  color: #4ade80;
}

.notification.error {
  background: rgba(239, 68, 68, 0.1);
  border-color: rgba(239, 68, 68, 0.3);
  color: #ef4444;
}

.notification-content {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Dashboard */
.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 3rem;
}

.dashboard-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0;
  background: linear-gradient(45deg, #fff, #888);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.stats-card {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 1.5rem;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
}

.stat-number {
  font-size: 2rem;
  font-weight: 700;
  color: #fff;
}

.stat-label {
  font-size: 0.875rem;
  color: #888;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

/* Applications */
.applications-container {
  min-height: 400px;
}

.empty-state {
  text-align: center;
  padding: 4rem 2rem;
}

.empty-icon {
  color: #333;
  margin-bottom: 1.5rem;
}

.empty-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #fff;
  margin: 0 0 0.5rem 0;
}

.empty-description {
  color: #888;
  margin: 0;
}

.applications-grid {
  display: grid;
  gap: 1.5rem;
}

.application-card {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 1.5rem;
  transition: all 0.3s ease;
  animation: fadeInUp 0.5s ease forwards;
  opacity: 0;
}

.application-card:hover {
  transform: translateY(-4px);
  border-color: rgba(255, 255, 255, 0.2);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
}

@keyframes fadeInUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
}

.applicant-name {
  font-size: 1.25rem;
  font-weight: 600;
  color: #fff;
  margin: 0 0 0.25rem 0;
}

.applicant-contact {
  color: #888;
  margin: 0;
  font-size: 0.875rem;
}

.application-date {
  color: #666;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.card-body {
  margin-bottom: 1.5rem;
}

.file-info {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
}

.file-icon {
  color: #888;
}

.file-name {
  color: #fff;
  font-weight: 500;
}

.card-actions {
  display: flex;
  gap: 0.75rem;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: 1px solid;
  border-radius: 8px;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.action-btn.primary {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.2);
  color: #fff;
}

.action-btn.primary:hover {
  background: rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 255, 255, 0.3);
}

.action-btn.danger {
  background: rgba(239, 68, 68, 0.1);
  border-color: rgba(239, 68, 68, 0.3);
  color: #ef4444;
}

.action-btn.danger:hover {
  background: rgba(239, 68, 68, 0.2);
  border-color: rgba(239, 68, 68, 0.4);
}

/* Responsive Design */
@media (max-width: 768px) {
  .nav-container {
    flex-direction: column;
    gap: 1rem;
  }

  .main-content {
    padding: 1rem;
  }

  .hero-title {
    font-size: 2.5rem;
  }

  .glass-card {
    padding: 2rem;
  }

  .form-grid {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }

  .dashboard-header {
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }

  .card-header {
    flex-direction: column;
    gap: 0.5rem;
  }

  .card-actions {
    flex-direction: column;
  }

  .loading-view {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.85);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }

  .loading-spinner {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .spinner {
    animation: spin 1s linear infinite;
    margin-bottom: 20px;
  }

  @keyframes spin {
    100% {
      transform: rotate(360deg);
    }
  }

  .path {
    stroke: #6c63ff;
    stroke-linecap: round;
    animation: dash 1.5s ease-in-out infinite;
  }

  @keyframes dash {
    0% {
      stroke-dasharray: 1, 150;
      stroke-dashoffset: 0;
    }

    50% {
      stroke-dasharray: 90, 150;
      stroke-dashoffset: -35;
    }

    100% {
      stroke-dasharray: 90, 150;
      stroke-dashoffset: -124;
    }
  }

  .loading-text {
    color: #fff;
    font-size: 1.5rem;
    letter-spacing: 2px;
    display: flex;
    align-items: center;
  }

  .dot {
    opacity: 0;
    animation: blink 1.4s infinite both;
    font-size: 2rem;
    margin-left: 2px;
  }

  .dot1 {
    animation-delay: 0s;
  }

  .dot2 {
    animation-delay: 0.2s;
  }

  .dot3 {
    animation-delay: 0.4s;
  }

  @keyframes blink {

    0%,
    80%,
    100% {
      opacity: 0;
    }

    40% {
      opacity: 1;
    }
  }
}
</style>