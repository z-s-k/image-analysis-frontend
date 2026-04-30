<template>
  <div class="app-container">
    <div class="header">
      <h1>👋 欢迎使用 作业出题</h1>
      <p class="subtitle">请上传作业票照片，我们将自动识别并生成对应考题。</p>
    </div>

    <div class="upload-container">
      <el-upload
        class="upload-area"
        action="#"
        :auto-upload="false"
        :show-file-list="true"
        :limit="1"
        :on-change="handleFileChange"
        :on-exceed="handleExceed"
        accept="image/*"
        ref="uploadRef"
      >
        <div class="upload-box">
          <el-icon class="el-icon--upload"><upload-filled /></el-icon>
          <div class="el-upload__text">
            <em>点击此处上传作业票图片</em>
          </div>
        </div>
      </el-upload>
    </div>

    <div class="action-container">
      <el-button type="primary" size="large" @click="submitUpload" :loading="loading" class="submit-btn">
        <el-icon style="margin-right: 5px"><Promotion /></el-icon> 开始生成
      </el-button>
    </div>

    <div v-if="result" class="result-container">
      <el-card class="box-card">
        <template #header>
          <div class="card-header">
            <span>识别结果</span>
            <span class="time-cost">耗时: {{ costTimeMs }} ms</span>
          </div>
        </template>
        <div class="result-content">
          <pre>{{ result }}</pre>
        </div>
        <div class="processed-image" v-if="processedImage">
          <p>处理后图片 (双边滤波):</p>
          <img :src="processedImage" alt="Processed Image" style="max-width: 100%; border-radius: 8px;" />
        </div>
      </el-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { UploadFilled, Promotion } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'
import type { UploadInstance, UploadProps, UploadFile } from 'element-plus'
import axios from 'axios'

const uploadRef = ref<UploadInstance>()
const fileToUpload = ref<File | null>(null)
const loading = ref(false)
const result = ref('')
const costTimeMs = ref(0)
const processedImage = ref('')

const handleFileChange: UploadProps['onChange'] = (uploadFile: UploadFile) => {
  if (uploadFile.raw) {
    fileToUpload.value = uploadFile.raw
  }
}

const handleExceed: UploadProps['onExceed'] = (files) => {
  uploadRef.value?.clearFiles()
  const file = files[0] as unknown as UploadFile
  uploadRef.value?.handleStart(file)
}

const submitUpload = async () => {
  if (!fileToUpload.value) {
    ElMessage.warning('请先选择一张图片')
    return
  }

  loading.value = true
  result.value = ''
  processedImage.value = ''

  const formData = new FormData()
  formData.append('file', fileToUpload.value)

  try {
    const response = await axios.post('http://localhost:7878/api/image/analyze', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })

    if (response.data.success) {
      result.value = response.data.result
      costTimeMs.value = response.data.costTimeMs
      processedImage.value = response.data.processedImage
      ElMessage.success('识别成功')
    } else {
      ElMessage.error(response.data.error || '识别失败')
    }
  } catch (error) {
    console.error(error)
    ElMessage.error('请求出错，请检查后端服务是否启动')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 20px;
  background-color: #fff;
  min-height: 100vh;
}

.header {
  text-align: center;
  margin-bottom: 40px;
}

.header h1 {
  font-size: 28px;
  color: #303133;
  margin: 0 0 10px 0;
  font-weight: 600;
}

.subtitle {
  color: #909399;
  font-size: 14px;
  margin: 0;
}

.upload-container {
  width: 100%;
  max-width: 600px;
  margin-bottom: 30px;
}

:deep(.el-upload) {
  display: block;
  width: 100%;
}

.upload-box {
  border: 1px dashed #dcdfe6;
  border-radius: 8px;
  background-color: #fafafa;
  padding: 40px 0;
  text-align: center;
  cursor: pointer;
  transition: border-color 0.3s;
}

.upload-box:hover {
  border-color: #409eff;
}

.el-icon--upload {
  font-size: 48px;
  color: #a8abb2;
  margin-bottom: 15px;
}

.action-container {
  margin-bottom: 40px;
}

.submit-btn {
  background: linear-gradient(90deg, #409eff, #36cfc9);
  border: none;
  border-radius: 20px;
  padding: 12px 30px;
  font-size: 16px;
  box-shadow: 0 4px 12px rgba(64, 158, 255, 0.3);
  transition: all 0.3s;
}

.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(64, 158, 255, 0.4);
  opacity: 0.9;
}

.result-container {
  width: 100%;
  max-width: 800px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.time-cost {
  color: #67c23a;
  font-size: 14px;
  font-weight: bold;
}

.result-content pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  background-color: #f5f7fa;
  padding: 15px;
  border-radius: 4px;
  color: #606266;
  font-size: 14px;
  line-height: 1.6;
}

.processed-image {
  margin-top: 20px;
  text-align: center;
}

.processed-image p {
  color: #606266;
  font-size: 14px;
  margin-bottom: 10px;
}
</style>
