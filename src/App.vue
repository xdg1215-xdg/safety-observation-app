<template>
  <div class="app-container">
    <!-- Header with Logo -->
    <header class="app-header">
      <img :src="logoUrl" alt="LOGO" class="logo" />
      <h1>中盐常化STOP填报系统</h1>
    </header>

    <!-- Login Form -->
    <div v-if="!isLoggedIn" class="login-container">
      <div class="login-box">
        <h2>用户登录</h2>
        <div class="form-group">
          <label>用户名</label>
          <input v-model="loginForm.username" type="text" placeholder="请输入用户名" />
        </div>
        <div class="form-group">
          <label>密码</label>
          <input v-model="loginForm.password" type="password" placeholder="请输入密码" />
        </div>
        <button class="btn-primary" @click="handleLogin">登录</button>
      </div>
    </div>

    <!-- Main Form -->
    <div v-else class="form-container">
      <div class="form-section">
        <h2>安全观察与沟通记录表</h2>
        
        <!-- 基本信息 -->
        <div class="form-group">
          <label>观察日期</label>
          <input v-model="formData.observationDate" type="date" />
        </div>
        
        <div class="form-row">
          <div class="form-group">
            <label>观察地点</label>
            <input v-model="formData.location" type="text" placeholder="请输入观察地点" />
          </div>
          <div class="form-group">
            <label>观察者姓名</label>
            <input v-model="formData.observerName" type="text" placeholder="请输入观察者姓名" />
          </div>
        </div>

        <!-- 观察类型 -->
        <div class="form-group">
          <label>观察类型</label>
          <select v-model="formData.observationType">
            <option value="">请选择</option>
            <option value="behavior">行为观察</option>
            <option value="environment">环境观察</option>
            <option value="equipment">设备观察</option>
            <option value="operation">操作观察</option>
          </select>
        </div>

        <!-- STOP观察项 -->
        <div class="form-section-title">STOP观察项</div>
        <div class="checkbox-grid">
          <label v-for="item in stopItems" :key="item.value" class="checkbox-item">
            <input type="checkbox" :value="item.value" v-model="formData.stopItems" />
            <span>{{ item.label }}</span>
          </label>
        </div>

        <!-- 不安全行为描述 -->
        <div class="form-group">
          <label>不安全行为/状态描述</label>
          <textarea v-model="formData.unsafeBehavior" rows="3" placeholder="请描述观察到的不安全行为或状态"></textarea>
        </div>

        <!-- 风险等级 -->
        <div class="form-group">
          <label>风险等级</label>
          <select v-model="formData.riskLevel">
            <option value="">请选择</option>
            <option value="low">低风险</option>
            <option value="medium">中等风险</option>
            <option value="high">高风险</option>
            <option value="critical">重大风险</option>
          </select>
        </div>

        <!-- 建议措施 -->
        <div class="form-group">
          <label>建议整改措施</label>
          <textarea v-model="formData.suggestedMeasures" rows="3" placeholder="请输入建议的整改措施"></textarea>
        </div>

        <!-- 拍照上传 -->
        <div class="form-group">
          <label>现场照片</label>
          <div class="photo-upload" @click="takePhoto">
            <span v-if="!formData.photo">📷 点击拍照</span>
            <img v-else :src="formData.photo" alt="现场照片" />
          </div>
        </div>

        <!-- 提交按钮 -->
        <button class="btn-primary btn-submit" @click="submitForm">提交报表</button>
      </div>

      <!-- 历史记录 -->
      <div class="history-section">
        <h3>提交历史</h3>
        <div v-if="history.length === 0" class="empty-state">暂无提交记录</div>
        <div v-else class="history-list">
          <div v-for="(item, index) in history" :key="index" class="history-item">
            <div class="history-date">{{ item.date }}</div>
            <div class="history-location">{{ item.location }}</div>
            <div class="history-risk" :class="'risk-' + item.riskLevel">{{ getRiskLabel(item.riskLevel) }}</div>
          </div>
        </div>
      </div>

      <!-- 退出登录 -->
      <button class="btn-secondary" @click="handleLogout">退出登录</button>
    </div>

    <!-- Toast提示 -->
    <div v-if="toast.show" class="toast" :class="toast.type">{{ toast.message }}</div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import logoUrl from './assets/logo.png'

// Logo URL

// 登录状态
const isLoggedIn = ref(false)
const loginForm = reactive({
  username: '',
  password: ''
})

// 表单数据
const formData = reactive({
  observationDate: new Date().toISOString().split('T')[0],
  location: '',
  observerName: '',
  observationType: '',
  stopItems: [],
  unsafeBehavior: '',
  riskLevel: '',
  suggestedMeasures: '',
  photo: null
})

// STOP观察项
const stopItems = [
  { value: 'ppe', label: '个人防护装备(PPE)' },
  { value: 'body_position', label: '身体姿势' },
  { value: 'tools_equipment', label: '工具设备使用' },
  { value: ' housekeeping', label: '5S/现场管理' },
  { value: 'procedures', label: '作业程序' },
  { value: 'storage', label: '物料存储' },
  { value: 'signage', label: '标识标志' },
  { value: 'emergency', label: '应急设施' }
]

// 历史记录
const history = ref([])

// Toast提示
const toast = reactive({
  show: false,
  message: '',
  type: 'info'
})

// 显示Toast
const showToast = (message, type = 'info') => {
  toast.message = message
  toast.type = type
  toast.show = true
  setTimeout(() => {
    toast.show = false
  }, 3000)
}

// 登录处理
const handleLogin = () => {
  if (!loginForm.username || !loginForm.password) {
    showToast('请输入用户名和密码', 'error')
    return
  }
  // 模拟登录
  isLoggedIn.value = true
  showToast('登录成功', 'success')
  loadHistory()
}

// 退出登录
const handleLogout = () => {
  isLoggedIn.value = false
  loginForm.username = ''
  loginForm.password = ''
}

// 加载历史记录
const loadHistory = () => {
  const saved = localStorage.getItem('stopReports')
  if (saved) {
    history.value = JSON.parse(saved)
  }
}

// 拍照
const takePhoto = () => {
  // 使用 Capacitor Camera API 或 HTML5 input
  const input = document.createElement('input')
  input.type = 'file'
  input.accept = 'image/*'
  input.capture = 'environment'
  input.onchange = (e) => {
    const file = e.target.files[0]
    if (file) {
      const reader = new FileReader()
      reader.onload = (e) => {
        formData.photo = e.target.result
      }
      reader.readAsDataURL(file)
    }
  }
  input.click()
}

// 提交表单
const submitForm = () => {
  // 表单验证
  if (!formData.location || !formData.observerName || !formData.observationType) {
    showToast('请填写必填项', 'error')
    return
  }

  // 保存记录
  const report = {
    date: formData.observationDate,
    location: formData.location,
    observer: formData.observerName,
    type: formData.observationType,
    riskLevel: formData.riskLevel,
    stopItems: [...formData.stopItems],
    unsafeBehavior: formData.unsafeBehavior,
    suggestedMeasures: formData.suggestedMeasures,
    photo: formData.photo,
    submitTime: new Date().toISOString()
  }

  // 保存到历史记录
  history.value.unshift(report)
  localStorage.setItem('stopReports', JSON.stringify(history.value))

  // 显示成功提示
  showToast('提交成功', 'success')

  // 重置表单
  resetForm()
}

// 重置表单
const resetForm = () => {
  formData.observationDate = new Date().toISOString().split('T')[0]
  formData.location = ''
  formData.observerName = ''
  formData.observationType = ''
  formData.stopItems = []
  formData.unsafeBehavior = ''
  formData.riskLevel = ''
  formData.suggestedMeasures = ''
  formData.photo = null
}

// 获取风险等级标签
const getRiskLabel = (level) => {
  const labels = {
    low: '低风险',
    medium: '中等风险',
    high: '高风险',
    critical: '重大风险'
  }
  return labels[level] || level
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background-color: #f5f5f5;
}

.app-container {
  min-height: 100vh;
  background-color: #f5f5f5;
}

.app-header {
  background: linear-gradient(135deg, #1a5fb4 0%, #0d47a1 100%);
  color: white;
  padding: 20px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}

.logo {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  object-fit: cover;
  margin-bottom: 10px;
  border: 3px solid white;
}

.app-header h1 {
  font-size: 20px;
  font-weight: 600;
}

.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 80vh;
  padding: 20px;
}

.login-box {
  background: white;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  width: 100%;
  max-width: 400px;
}

.login-box h2 {
  text-align: center;
  margin-bottom: 24px;
  color: #1a5fb4;
}

.form-container {
  padding: 16px;
}

.form-section {
  background: white;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}

.form-section h2 {
  color: #1a5fb4;
  margin-bottom: 20px;
  font-size: 18px;
  text-align: center;
}

.form-section-title {
  font-weight: 600;
  color: #333;
  margin: 16px 0 12px;
  padding-left: 8px;
  border-left: 4px solid #1a5fb4;
}

.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  color: #333;
  font-weight: 500;
  font-size: 14px;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #1a5fb4;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.checkbox-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.checkbox-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px;
  background: #f8f9fa;
  border-radius: 8px;
  font-size: 13px;
  cursor: pointer;
}

.checkbox-item input {
  width: 18px;
  height: 18px;
}

.photo-upload {
  border: 2px dashed #ddd;
  border-radius: 8px;
  padding: 30px;
  text-align: center;
  cursor: pointer;
  transition: border-color 0.3s;
}

.photo-upload:hover {
  border-color: #1a5fb4;
}

.photo-upload img {
  max-width: 100%;
  max-height: 200px;
  border-radius: 8px;
}

.btn-primary {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #1a5fb4 0%, #0d47a1 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(26, 95, 180, 0.4);
}

.btn-secondary {
  width: 100%;
  padding: 12px;
  background: #f5f5f5;
  color: #666;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  cursor: pointer;
  margin-top: 16px;
}

.history-section {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}

.history-section h3 {
  margin-bottom: 16px;
  color: #333;
}

.empty-state {
  text-align: center;
  color: #999;
  padding: 30px;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.history-item {
  display: flex;
  align-items: center;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  gap: 12px;
}

.history-date {
  color: #666;
  font-size: 13px;
}

.history-location {
  flex: 1;
  color: #333;
}

.history-risk {
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.risk-low { background: #e8f5e9; color: #2e7d32; }
.risk-medium { background: #fff3e0; color: #ef6c00; }
.risk-high { background: #ffebee; color: #c62828; }
.risk-critical { background: #f3e5f5; color: #7b1fa2; }

.toast {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  border-radius: 8px;
  color: white;
  font-size: 14px;
  z-index: 1000;
  animation: slideDown 0.3s ease;
}

.toast.success { background: #4caf50; }
.toast.error { background: #f44336; }
.toast.info { background: #2196f3; }

@keyframes slideDown {
  from { opacity: 0; transform: translate(-50%, -20px); }
  to { opacity: 1; transform: translate(-50%, 0); }
}

@media (max-width: 480px) {
  .form-row, .checkbox-grid {
    grid-template-columns: 1fr;
  }
}
</style>