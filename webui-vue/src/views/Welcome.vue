<template>
  <div class="welcome-page">
    <!-- 左侧：品牌 + 快速入口 -->
    <div class="left-panel">
      <a href="https://github.com/None9527/None_Z-image-Turbo_trainer" target="_blank" class="brand-link">
        <div class="brand">
          <div class="logo">
            <span>N</span>
          </div>
          <div class="brand-text">
            <h1><span class="gold">None</span> Trainer</h1>
            <p>Z-Image Turbo LoRA 训练工作室</p>
          </div>
        </div>
      </a>
      
      <div class="nav-grid">
        <div class="nav-card" @click="$router.push('/dataset')">
          <div class="nav-icon blue"><el-icon><Picture /></el-icon></div>
          <div class="nav-content">
            <h3>数据集管理</h3>
            <p>导入图片、生成缓存、Ollama 智能标注</p>
          </div>
          <el-icon class="nav-arrow"><ArrowRight /></el-icon>
        </div>
        
        <div class="nav-card" @click="$router.push('/config')">
          <div class="nav-icon green"><el-icon><Setting /></el-icon></div>
          <div class="nav-content">
            <h3>训练配置</h3>
            <p>AC-RF 参数、LoRA 设置、优化器选择</p>
          </div>
          <el-icon class="nav-arrow"><ArrowRight /></el-icon>
        </div>
        
        <div class="nav-card" @click="$router.push('/training')">
          <div class="nav-icon gold"><el-icon><VideoPlay /></el-icon></div>
          <div class="nav-content">
            <h3>开始训练</h3>
            <p>实时 Loss 曲线、进度监控、显存状态</p>
          </div>
          <el-icon class="nav-arrow"><ArrowRight /></el-icon>
        </div>
        
        <div class="nav-card" @click="$router.push('/generation')">
          <div class="nav-icon orange"><el-icon><MagicStick /></el-icon></div>
          <div class="nav-content">
            <h3>图像生成</h3>
            <p>加载训练好的 LoRA 模型测试效果</p>
          </div>
          <el-icon class="nav-arrow"><ArrowRight /></el-icon>
        </div>
      </div>
      
      <div class="footer">
        <span>Made with ❤️ by <strong>None</strong></span>
        <span class="divider">·</span>
        <span>基于 <strong>AC-RF</strong> 锚点耦合整流流算法</span>
      </div>
    </div>
    
    <!-- 右侧：状态面板 -->
    <div class="right-panel">
      <!-- 系统状态 -->
      <div class="status-card">
        <div class="card-header">
          <el-icon><Monitor /></el-icon>
          <span>系统环境</span>
          <el-tag v-if="wsConnected" type="success" size="small">在线</el-tag>
          <el-tag v-else type="danger" size="small">离线</el-tag>
        </div>
        <div class="env-grid" v-if="hasSystemInfo">
          <div class="env-item">
            <span class="env-label">Python</span>
            <span class="env-value">{{ systemInfo.python }}</span>
          </div>
          <div class="env-item">
            <span class="env-label">PyTorch</span>
            <span class="env-value">{{ systemInfo.pytorch }}</span>
          </div>
          <div class="env-item">
            <span class="env-label">CUDA</span>
            <span class="env-value">{{ systemInfo.cuda }}</span>
          </div>
          <div class="env-item">
            <span class="env-label">Diffusers</span>
            <span class="env-value">{{ systemInfo.diffusers }}</span>
          </div>
          <div class="env-item">
            <span class="env-label">Accelerate</span>
            <span class="env-value">{{ systemInfo.accelerate || 'N/A' }}</span>
          </div>
          <div class="env-item">
            <span class="env-label">Transformers</span>
            <span class="env-value">{{ systemInfo.transformers || 'N/A' }}</span>
          </div>
        </div>
        <div v-else class="loading-state">
          <el-icon class="is-loading"><Loading /></el-icon>
          <span>连接服务中...</span>
        </div>
      </div>
      
      <!-- 模型状态 -->
      <div class="status-card model-card">
        <div class="card-header">
          <el-icon><Box /></el-icon>
          <span>基础模型</span>
          <el-tag :type="modelStatus.exists ? 'success' : 'warning'" size="small">
            {{ modelStatus.exists ? '就绪' : '需下载' }}
          </el-tag>
        </div>
        
        <div class="model-status" v-if="modelStatus.summary">
          <div class="model-ring">
            <svg viewBox="0 0 100 100">
              <circle class="ring-bg" cx="50" cy="50" r="42" />
              <circle class="ring-progress" cx="50" cy="50" r="42" :style="{ strokeDashoffset: progressOffset }" />
            </svg>
            <div class="ring-text">
              <span class="ring-num">{{ validPercent }}</span>
              <span class="ring-unit">%</span>
            </div>
          </div>
          
          <div class="model-details">
            <div class="model-stat">
              <span class="stat-num success">{{ modelStatus.summary.valid_components }}</span>
              <span class="stat-label">有效组件</span>
            </div>
            <div class="model-stat">
              <span class="stat-num">{{ modelStatus.summary.total_components }}</span>
              <span class="stat-label">总组件</span>
            </div>
          </div>
        </div>
        
        <div class="component-tags" v-if="modelStatus.details">
          <span v-for="(comp, name) in modelStatus.details" :key="name" class="comp-tag" :class="{ valid: comp.valid, missing: !comp.exists }">
            <el-icon><CircleCheck v-if="comp.valid" /><Warning v-else-if="comp.exists" /><Close v-else /></el-icon>
            {{ getComponentName(name) }}
          </span>
        </div>
        
        <div class="model-action" v-if="!modelStatus.exists">
          <el-button v-if="!isDownloading" type="primary" @click="startDownload" :loading="startingDownload">
            <el-icon><Download /></el-icon>
            下载 Z-Image-Turbo 模型
          </el-button>
          <div v-else class="download-progress">
            <el-progress :percentage="downloadProgress" :stroke-width="12" />
            <span>{{ downloadSizeText }}</span>
          </div>
        </div>
      </div>
      
      <!-- 技术特性 -->
      <div class="feature-grid">
        <div class="feature-item">
          <span class="feature-icon">🎯</span>
          <span>锚点耦合采样</span>
        </div>
        <div class="feature-item">
          <span class="feature-icon">📉</span>
          <span>Min-SNR 加权</span>
        </div>
        <div class="feature-item">
          <span class="feature-icon">⚡</span>
          <span>Flash Attention</span>
        </div>
        <div class="feature-item">
          <span class="feature-icon">🔧</span>
          <span>自动硬件适配</span>
        </div>
      </div>
      
      <!-- 联系方式 -->
      <div class="contact-bar">
        <div class="contact-item" @click="copyEmail('lihaonan1082@gmail.com')">
          <span>📧</span>
          <span>lihaonan1082@gmail.com</span>
        </div>
        <div class="contact-item" @click="copyEmail('592532681@qq.com')">
          <span>📮</span>
          <span>592532681@qq.com</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useSystemStore } from '@/stores/system'
import { useWebSocketStore } from '@/stores/websocket'
import { 
  Picture, Setting, VideoPlay, Monitor, Box,
  CircleCheck, Warning, Close, Loading,
  ArrowRight, MagicStick, Download
} from '@element-plus/icons-vue'
import axios from 'axios'
import { ElMessage } from 'element-plus'

const systemStore = useSystemStore()
const wsStore = useWebSocketStore()

const modelStatus = ref({ exists: false, details: null as any, summary: null as any })
const startingDownload = ref(false)

const systemInfo = computed(() => systemStore.systemInfo)
const wsConnected = computed(() => wsStore.isConnected)
const hasSystemInfo = computed(() => systemStore.systemInfo.python !== '')

const downloadStatus = computed(() => systemStore.downloadStatus)
const isDownloading = computed(() => downloadStatus.value.status === 'running')
const downloadProgress = computed(() => downloadStatus.value.progress)
const downloadSizeText = computed(() => {
  const gb = downloadStatus.value.downloaded_size_gb || 0
  return gb > 0 ? `已下载 ${gb.toFixed(2)} GB` : '准备中...'
})

const validPercent = computed(() => {
  if (!modelStatus.value.summary) return 0
  const { valid_components, total_components } = modelStatus.value.summary
  return Math.round((valid_components / total_components) * 100)
})

const progressOffset = computed(() => {
  const circumference = 2 * Math.PI * 42
  return circumference - (validPercent.value / 100) * circumference
})

const componentNames: Record<string, string> = {
  'transformer': 'Transformer',
  'vae': 'VAE',
  'text_encoder': 'Text Encoder',
  'tokenizer': 'Tokenizer',
  'scheduler': 'Scheduler',
  'model_index.json': 'Index'
}

function getComponentName(name: string): string {
  return componentNames[name] || name
}

async function refreshModelStatus() {
  try {
    const res = await axios.get('/api/system/model-status')
    modelStatus.value = res.data
  } catch (e) {
    console.error('Failed to check model status:', e)
  }
}

async function startDownload() {
  startingDownload.value = true
  try {
    await axios.post('/api/system/download-model')
    ElMessage.success('下载任务已启动')
  } catch (e: any) {
    ElMessage.error('启动下载失败: ' + (e.response?.data?.detail || e.message))
  } finally {
    startingDownload.value = false
  }
}

refreshModelStatus()

function copyEmail(email: string) {
  navigator.clipboard.writeText(email)
  ElMessage.success(`已复制: ${email}`)
}
</script>

<style scoped>
.welcome-page {
  height: 100%;
  display: grid;
  grid-template-columns: 1fr 420px;
  gap: 0;
  background: var(--bg-primary);
  overflow: hidden;
}

/* 左侧面板 */
.left-panel {
  display: flex;
  flex-direction: column;
  padding: 48px;
  background: linear-gradient(135deg, rgba(240, 180, 41, 0.02) 0%, transparent 50%);
}

.brand-link {
  text-decoration: none;
  color: inherit;
  cursor: pointer;
  transition: transform 0.2s;
  display: inline-block;
  margin-bottom: 48px;
}

.brand-link:hover {
  transform: translateX(4px);
}

.brand-link:hover .logo {
  box-shadow: 0 12px 40px rgba(240, 180, 41, 0.4);
  transform: rotate(-5deg) scale(1.05);
}

.brand {
  display: flex;
  align-items: center;
  gap: 20px;
}

.logo {
  width: 72px;
  height: 72px;
  background: linear-gradient(135deg, #f0b429 0%, #e67e22 100%);
  border-radius: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 32px rgba(240, 180, 41, 0.3);
  transition: all 0.3s ease;
}

.logo span {
  font-size: 40px;
  font-weight: 800;
  color: #1a1a1d;
}

.brand-text h1 {
  font-size: 2.8rem;
  font-weight: 800;
  margin: 0;
  letter-spacing: -1px;
}

.brand-text h1 .gold {
  background: linear-gradient(135deg, #f0b429 0%, #e67e22 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.brand-text p {
  margin: 4px 0 0 0;
  color: var(--text-muted);
  font-size: 1rem;
}

/* 导航网格 */
.nav-grid {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.nav-card {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 24px;
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.25s ease;
}

.nav-card:hover {
  border-color: var(--primary);
  background: rgba(240, 180, 41, 0.05);
  transform: translateX(8px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

.nav-icon {
  width: 56px;
  height: 56px;
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  flex-shrink: 0;
}

.nav-icon.blue { background: rgba(64, 158, 255, 0.15); color: #409eff; }
.nav-icon.green { background: rgba(103, 194, 58, 0.15); color: #67c23a; }
.nav-icon.gold { background: rgba(240, 180, 41, 0.15); color: #f0b429; }
.nav-icon.orange { background: rgba(230, 126, 34, 0.15); color: #e67e22; }

.nav-content {
  flex: 1;
}

.nav-content h3 {
  margin: 0 0 4px 0;
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--text-primary);
}

.nav-content p {
  margin: 0;
  font-size: 0.9rem;
  color: var(--text-muted);
}

.nav-arrow {
  font-size: 20px;
  color: var(--text-muted);
  transition: all 0.2s;
}

.nav-card:hover .nav-arrow {
  color: var(--primary);
  transform: translateX(4px);
}

/* 底部 */
.footer {
  margin-top: auto;
  padding-top: 24px;
  color: var(--text-muted);
  font-size: 0.85rem;
}

.footer strong {
  color: var(--primary);
}

.footer .divider {
  margin: 0 12px;
  opacity: 0.3;
}

/* 右侧面板 */
.right-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 32px;
  background: var(--bg-secondary);
  border-left: 1px solid var(--border-color);
  overflow-y: auto;
}

/* 状态卡片 */
.status-card {
  padding: 20px;
  background: var(--bg-darker);
  border-radius: 14px;
  border: 1px solid var(--border-color);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 16px;
  font-weight: 600;
  color: var(--text-primary);
}

.card-header .el-icon {
  color: var(--primary);
}

.card-header .el-tag {
  margin-left: auto;
}

/* 环境信息 */
.env-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.env-item {
  display: flex;
  justify-content: space-between;
  padding: 10px 12px;
  background: var(--bg-primary);
  border-radius: 8px;
}

.env-label {
  color: var(--text-muted);
  font-size: 12px;
}

.env-value {
  color: var(--text-primary);
  font-size: 12px;
  font-weight: 500;
  font-family: var(--font-mono);
}

.loading-state {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 32px;
  color: var(--text-muted);
}

/* 模型状态 */
.model-status {
  display: flex;
  align-items: center;
  gap: 24px;
  margin-bottom: 16px;
}

.model-ring {
  position: relative;
  width: 90px;
  height: 90px;
  flex-shrink: 0;
}

.model-ring svg {
  transform: rotate(-90deg);
}

.model-ring circle {
  fill: none;
  stroke-width: 8;
  stroke-linecap: round;
}

.ring-bg {
  stroke: var(--bg-primary);
}

.ring-progress {
  stroke: var(--success);
  stroke-dasharray: 264;
  transition: stroke-dashoffset 0.5s ease;
}

.ring-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.ring-num {
  font-size: 24px;
  font-weight: 700;
  color: var(--text-primary);
}

.ring-unit {
  font-size: 12px;
  color: var(--text-muted);
}

.model-details {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.model-stat {
  display: flex;
  flex-direction: column;
}

.stat-num {
  font-size: 28px;
  font-weight: 700;
  color: var(--text-primary);
  line-height: 1;
}

.stat-num.success {
  color: var(--success);
}

.stat-label {
  font-size: 12px;
  color: var(--text-muted);
  margin-top: 2px;
}

/* 组件标签 */
.component-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 16px;
}

.comp-tag {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 10px;
  background: var(--bg-primary);
  border-radius: 6px;
  font-size: 11px;
  color: var(--text-muted);
}

.comp-tag.valid {
  background: rgba(103, 194, 58, 0.1);
  color: var(--success);
}

.comp-tag.missing {
  opacity: 0.5;
}

.comp-tag .el-icon {
  font-size: 12px;
}

/* 模型操作 */
.model-action {
  text-align: center;
}

.download-progress {
  max-width: 100%;
}

.download-progress span {
  display: block;
  margin-top: 8px;
  font-size: 12px;
  color: var(--text-muted);
  text-align: center;
}

/* 技术特性 */
.feature-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 16px;
  background: var(--bg-darker);
  border-radius: 10px;
  border: 1px solid var(--border-color);
  font-size: 13px;
  color: var(--text-secondary);
}

.feature-icon {
  font-size: 18px;
}

/* 联系方式 */
.contact-bar {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: auto;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  background: var(--bg-darker);
  border-radius: 10px;
  border: 1px solid var(--border-color);
  font-size: 13px;
  color: var(--text-secondary);
  font-family: var(--font-mono);
  cursor: pointer;
  transition: all 0.2s;
}

.contact-item:hover {
  border-color: var(--primary);
  color: var(--text-primary);
  background: rgba(240, 180, 41, 0.05);
}

/* 响应式 */
@media (max-width: 1000px) {
  .welcome-page {
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr;
  }
  
  .left-panel {
    padding: 32px;
  }
  
  .right-panel {
    border-left: none;
    border-top: 1px solid var(--border-color);
  }
  
  .brand-text h1 {
    font-size: 2rem;
  }
  
  .nav-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
</style>
