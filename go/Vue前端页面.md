# Vue 留言系统前端

> 🔗 **关联笔记**：[[留言系统项目实战]]
> 📅 **创建时间**：2024-12-24

---

## 📖 概述

使用 Vue 3 + Vite 为 Go 留言系统创建前端页面，实现完整的用户交互界面。

---

## 🛠️ 项目初始化

### 1. 创建 Vue 项目

```bash
npm create vite@latest message-board-frontend -- --template vue
cd message-board-frontend
npm install
```

### 2. 安装依赖

```bash
npm install axios vue-router@4 pinia
```

---

## 📁 项目结构

```
src/
├── api/
│   └── message.js      # API 接口
├── components/
│   ├── MessageCard.vue # 留言卡片
│   ├── MessageForm.vue # 留言表单
│   └── MessageList.vue # 留言列表
├── views/
│   └── Home.vue        # 首页
├── stores/
│   └── message.js      # Pinia 状态管理
├── App.vue
└── main.js
```

---

## 🔌 API 封装

### `src/api/message.js`

```javascript
import axios from 'axios'

const api = axios.create({
  baseURL: 'http://localhost:8080/api',
  timeout: 5000,
})

// 获取所有留言
export const getMessages = () => api.get('/messages')

// 获取单个留言
export const getMessage = (id) => api.get(`/messages/${id}`)

// 创建留言
export const createMessage = (data) => api.post('/messages', data)

// 更新留言
export const updateMessage = (id, data) => api.put(`/messages/${id}`, data)

// 删除留言
export const deleteMessage = (id) => api.delete(`/messages/${id}`)

// 搜索留言
export const searchMessages = (params) => api.get('/messages/search', { params })

export default api
```

---

## 🎨 组件实现

### `src/components/MessageCard.vue`

```vue
<template>
  <div class="message-card">
    <div class="card-header">
      <span class="author">{{ message.author }}</span>
      <span class="time">{{ formatTime(message.created_at) }}</span>
    </div>
    <div class="card-content">
      {{ message.content }}
    </div>
    <div class="card-footer">
      <button @click="handleEdit" class="btn-edit">编辑</button>
      <button @click="handleDelete" class="btn-delete">删除</button>
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  message: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['edit', 'delete'])

const formatTime = (time) => {
  return new Date(time).toLocaleString('zh-CN')
}

const handleEdit = () => {
  emit('edit', props.message)
}

const handleDelete = () => {
  if (confirm('确定要删除这条留言吗？')) {
    emit('delete', props.message.id)
  }
}
</script>

<style scoped>
.message-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.message-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.card-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
}

.author {
  font-weight: 600;
  color: #333;
}

.time {
  color: #999;
  font-size: 14px;
}

.card-content {
  color: #555;
  line-height: 1.6;
  margin-bottom: 16px;
}

.card-footer {
  display: flex;
  gap: 8px;
}

.btn-edit, .btn-delete {
  padding: 6px 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.2s;
}

.btn-edit {
  background: #e3f2fd;
  color: #1976d2;
}

.btn-edit:hover {
  background: #bbdefb;
}

.btn-delete {
  background: #ffebee;
  color: #d32f2f;
}

.btn-delete:hover {
  background: #ffcdd2;
}
</style>
```

### `src/components/MessageForm.vue`

```vue
<template>
  <div class="message-form">
    <h3>{{ isEdit ? '编辑留言' : '发表留言' }}</h3>
    <form @submit.prevent="handleSubmit">
      <div class="form-group">
        <label for="author">昵称</label>
        <input 
          id="author"
          v-model="form.author" 
          type="text" 
          placeholder="请输入昵称"
          required
        />
      </div>
      <div class="form-group">
        <label for="email">邮箱 (可选)</label>
        <input 
          id="email"
          v-model="form.email" 
          type="email" 
          placeholder="请输入邮箱"
        />
      </div>
      <div class="form-group">
        <label for="content">留言内容</label>
        <textarea 
          id="content"
          v-model="form.content" 
          placeholder="请输入留言内容..."
          rows="4"
          required
        ></textarea>
      </div>
      <div class="form-actions">
        <button type="submit" class="btn-submit">
          {{ isEdit ? '保存修改' : '发表留言' }}
        </button>
        <button v-if="isEdit" type="button" @click="handleCancel" class="btn-cancel">
          取消
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, watch, defineProps, defineEmits } from 'vue'

const props = defineProps({
  editMessage: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['submit', 'cancel'])

const form = ref({
  author: '',
  email: '',
  content: ''
})

const isEdit = ref(false)

watch(() => props.editMessage, (newVal) => {
  if (newVal) {
    form.value = { ...newVal }
    isEdit.value = true
  } else {
    form.value = { author: '', email: '', content: '' }
    isEdit.value = false
  }
}, { immediate: true })

const handleSubmit = () => {
  emit('submit', {
    ...form.value,
    id: props.editMessage?.id
  })
  
  if (!isEdit.value) {
    form.value = { author: '', email: '', content: '' }
  }
}

const handleCancel = () => {
  emit('cancel')
}
</script>

<style scoped>
.message-form {
  background: white;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h3 {
  margin-bottom: 20px;
  color: #333;
}

.form-group {
  margin-bottom: 16px;
}

label {
  display: block;
  margin-bottom: 6px;
  color: #555;
  font-weight: 500;
}

input, textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  transition: border-color 0.2s;
}

input:focus, textarea:focus {
  outline: none;
  border-color: #1976d2;
}

textarea {
  resize: vertical;
}

.form-actions {
  display: flex;
  gap: 12px;
}

.btn-submit {
  padding: 12px 24px;
  background: linear-gradient(135deg, #1976d2, #42a5f5);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.btn-submit:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(25, 118, 210, 0.4);
}

.btn-cancel {
  padding: 12px 24px;
  background: #eee;
  color: #666;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
}

.btn-cancel:hover {
  background: #ddd;
}
</style>
```

### `src/views/Home.vue`

```vue
<template>
  <div class="home">
    <header class="hero">
      <h1>💬 留言板</h1>
      <p>欢迎来到留言板，分享你的想法</p>
    </header>
    
    <main class="container">
      <!-- 留言表单 -->
      <MessageForm 
        :editMessage="editingMessage"
        @submit="handleSubmit"
        @cancel="editingMessage = null"
      />
      
      <!-- 搜索框 -->
      <div class="search-box">
        <input 
          v-model="searchKeyword" 
          type="text" 
          placeholder="搜索留言..."
          @input="handleSearch"
        />
      </div>
      
      <!-- 留言列表 -->
      <div class="message-list">
        <div v-if="loading" class="loading">加载中...</div>
        <div v-else-if="messages.length === 0" class="empty">
          暂无留言，快来发表第一条吧！
        </div>
        <MessageCard
          v-for="message in messages"
          :key="message.id"
          :message="message"
          @edit="handleEdit"
          @delete="handleDelete"
        />
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import MessageCard from '../components/MessageCard.vue'
import MessageForm from '../components/MessageForm.vue'
import { getMessages, createMessage, updateMessage, deleteMessage, searchMessages } from '../api/message'

const messages = ref([])
const loading = ref(false)
const editingMessage = ref(null)
const searchKeyword = ref('')

// 获取留言列表
const fetchMessages = async () => {
  loading.value = true
  try {
    const res = await getMessages()
    messages.value = res.data.data || []
  } catch (error) {
    console.error('获取留言失败:', error)
  } finally {
    loading.value = false
  }
}

// 提交留言
const handleSubmit = async (data) => {
  try {
    if (data.id) {
      // 更新
      await updateMessage(data.id, data)
      editingMessage.value = null
    } else {
      // 创建
      await createMessage(data)
    }
    await fetchMessages()
  } catch (error) {
    console.error('提交失败:', error)
    alert('操作失败，请重试')
  }
}

// 编辑留言
const handleEdit = (message) => {
  editingMessage.value = message
}

// 删除留言
const handleDelete = async (id) => {
  try {
    await deleteMessage(id)
    await fetchMessages()
  } catch (error) {
    console.error('删除失败:', error)
    alert('删除失败，请重试')
  }
}

// 搜索
let searchTimer = null
const handleSearch = () => {
  clearTimeout(searchTimer)
  searchTimer = setTimeout(async () => {
    if (searchKeyword.value) {
      const res = await searchMessages({ keyword: searchKeyword.value })
      messages.value = res.data.data || []
    } else {
      await fetchMessages()
    }
  }, 300)
}

onMounted(fetchMessages)
</script>

<style scoped>
.home {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.hero {
  text-align: center;
  padding: 60px 20px 40px;
  color: white;
}

.hero h1 {
  font-size: 2.5rem;
  margin-bottom: 12px;
}

.hero p {
  font-size: 1.1rem;
  opacity: 0.9;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 20px 40px;
}

.search-box {
  margin-bottom: 24px;
}

.search-box input {
  width: 100%;
  padding: 14px 20px;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.search-box input:focus {
  outline: none;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.loading, .empty {
  text-align: center;
  padding: 40px;
  color: white;
  font-size: 16px;
}
</style>
```

---

## 🚀 运行项目

```bash
# 开发模式
npm run dev

# 构建生产版本
npm run build
```

---

## 🔧 跨域配置

### Vite 代理配置 (`vite.config.js`)

```javascript
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
      }
    }
  }
}
```

### Go 后端 CORS 配置

```go
// main.go
import "github.com/gin-contrib/cors"

func main() {
    r := gin.Default()
    
    // CORS 配置
    r.Use(cors.New(cors.Config{
        AllowOrigins:     []string{"http://localhost:5173"},
        AllowMethods:     []string{"GET", "POST", "PUT", "DELETE"},
        AllowHeaders:     []string{"Content-Type", "Authorization"},
        AllowCredentials: true,
    }))
    
    // ... 其他配置
}
```

---

## 📚 相关笔记

- [[留言系统项目实战]] - 主项目 (后端)
- [[JWT用户认证]] - 用户认证

---

#Vue #Vite #前端开发 #留言系统
