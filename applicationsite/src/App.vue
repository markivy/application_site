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
          <h1 class="brand-text">XISTO</h1>
        </div>
        <div class="nav-tabs">
          <button 
            @click="currentView = 'apply'" 
            :class="['nav-tab', { active: currentView === 'apply' }]"
          >
            <span class="tab-text">Candidatar-se</span>
            <div class="tab-indicator"></div>
          </button>
        </div>
      </div>
    </nav>

    <!-- Application Form View -->
    <div v-if="currentView === 'apply'" class="main-content">
      <div class="hero-section">
        <h1 class="hero-title">Faça parte de nossa equipe!</h1>
        <p class="hero-subtitle">Envie seu currículo e vamos construir algo incrível juntos.</p>
      </div>

      <div class="form-container">
        <div class="glass-card">
          <form @submit.prevent="submitApplication" class="application-form">
            <div class="form-grid">
              <div class="input-group">
                <label class="input-label">Nome Completo</label>
                <div class="input-wrapper">
                  <input
                    v-model="form.name"
                    type="text"
                    class="modern-input"
                    placeholder="Digite seu nome completo"
                    required
                  />
                  <div class="input-border"></div>
                </div>
              </div>

              <div class="input-group">
                <label class="input-label">Número de Contato</label>
                <div class="input-wrapper">
                  <input
                    v-model="form.contact"
                    type="tel"
                    class="modern-input"
                    placeholder="Digite seu número de contato"
                    required
                  />
                  <div class="input-border"></div>
                </div>
              </div>
            </div>

            <div class="input-group file-group">
              <label class="input-label">Currículo</label>
              <div class="file-upload-zone" :class="{ 'has-file': form.cvFile, 'drag-over': isDragOver }"
                   @dragover.prevent="isDragOver = true"
                   @dragleave.prevent="isDragOver = false"
                   @drop.prevent="handleFileDrop">
                <input
                  type="file"
                  @change="handleFileUpload"
                  accept=".pdf,.doc,.docx"
                  class="file-input"
                  id="cv-upload"
                  required
                />
                <label for="cv-upload" class="file-upload-label">
                  <div class="upload-icon">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                      <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" stroke="currentColor" stroke-width="2"/>
                      <polyline points="7,10 12,15 17,10" stroke="currentColor" stroke-width="2"/>
                      <line x1="12" y1="15" x2="12" y2="3" stroke="currentColor" stroke-width="2"/>
                    </svg>
                  </div>
                  <div class="upload-text">
                    <span v-if="!form.cvFile" class="upload-primary">Arraste seu arquivo aqui ou clique para navegar</span>
                    <span v-else class="upload-success">{{ form.cvFile.name }}</span>
                    <span class="upload-secondary">PDF, DOC, DOCX até 5MB</span>
                  </div>
                </label>
              </div>
            </div>

            <button type="submit" class="submit-btn" :disabled="isSubmitting" @click="sendApplication">
              <span class="btn-text">
                {{ isSubmitting ? 'Enviando...' : 'Enviar Candidatura' }}
              </span>
              <div class="btn-shine"></div>
            </button>
          </form>

          <div v-if="submitMessage" :class="['notification', submitMessage.type]">
            <div class="notification-content">
              <div class="notification-icon">
                <svg v-if="submitMessage.type === 'success'" width="20" height="20" viewBox="0 0 24 24" fill="none">
                  <path d="M9 12l2 2 4-4" stroke="currentColor" stroke-width="2"/>
                  <circle cx="12" cy="12" r="9" stroke="currentColor" stroke-width="2"/>
                </svg>
                <svg v-else width="20" height="20" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="12" r="9" stroke="currentColor" stroke-width="2"/>
                  <line x1="15" y1="9" x2="9" y2="15" stroke="currentColor" stroke-width="2"/>
                  <line x1="9" y1="9" x2="15" y2="15" stroke="currentColor" stroke-width="2"/>
                </svg>
              </div>
              <span>{{ submitMessage.text }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'

const currentView = ref('apply')
const isSubmitting = ref(false)
const submitMessage = ref(null)
const applications = ref([])
const isDragOver = ref(false)

const form = reactive({
  name: '',
  contact: '',
  cvFile: null
})

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  processFile(file)
}

const handleFileDrop = (event) => {
  isDragOver.value = false
  const file = event.dataTransfer.files[0]
  processFile(file)
}

const processFile = (file) => {
  if (!file) return
  
  if (file.size > 5 * 1024 * 1024) {
    showMessage('O tamanho do arquivo deve ser menor que 5MB', 'error')
    return
  }
  
  const allowedTypes = ['application/pdf', 'application/msword', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document']
  if (!allowedTypes.includes(file.type)) {
    showMessage('Por favor, envie um arquivo PDF, DOC ou DOCX', 'error')
    return
  }
  
  form.cvFile = file
}
const fileToBinary = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsArrayBuffer(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = error => reject(error)
  })
}


const fileToBase64 = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = error => reject(error)
  })
}

const showMessage = (text, type) => {
  submitMessage.value = { text, type }
  setTimeout(() => {
    submitMessage.value = null
  }, 5000)
}

const loadApplications = () => {
  const saved = localStorage.getItem('jobApplications')
  if (saved) {
    applications.value = JSON.parse(saved)
  }
}


const sendApplication = async () => {
  try {
    const response = await fetch('http://147.79.104.229:5678/webhook/SendContext', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: form.name,
        contact: form.contact,
        cvFileName: form.cvFile.name,
        cvFileData: await fileToBase64(form.cvFile),
        submittedAt: new Date().toISOString()
      })
    })
    const data = await response.json()
    console.log(data)
    showMessage('Candidatura enviada com sucesso!', 'success')
  } catch (error) {
    console.error('Erro ao enviar candidatura:', error)
    showMessage('Erro ao enviar candidatura. Por favor, tente novamente.', 'error')
  }
}

onMounted(() => {
  loadApplications()

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
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  33% { transform: translateY(-30px) rotate(120deg); }
  66% { transform: translateY(20px) rotate(240deg); }
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

.modern-input:focus + .input-border {
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
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
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
}
</style>