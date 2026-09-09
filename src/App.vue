<script setup>
import { ref, computed, reactive } from 'vue'
import ConfigPanel from './components/ConfigPanel.vue'
import FieldRender from './components/FieldRender.vue'

const fields = ref([])

// 预览表单数据
const formData = reactive({})

// 错误信息
const errors = ref({})

function addField(field) {
  fields.value.push(field)
  syncValue(field)
}

function updateField(index, patch) {
  const old = fields.value[index]
  const updated = { ...old, ...patch }
  // 切换类型时重置 value
  if (patch.type && patch.type !== old.type) {
    updated.value = patch.type === 'checkbox' ? [] : ''
  }
  fields.value[index] = updated
  syncValue(updated)
}

function removeField(index) {
  const f = fields.value[index]
  delete formData[f.id]
  fields.value.splice(index, 1)
}

function moveField(index, dir) {
  const target = index + dir
  if (target < 0 || target >= fields.value.length) return
  const list = fields.value
  ;[list[index], list[target]] = [list[target], list[index]]
}

function syncValue(field) {
  if (field.type === 'checkbox') {
    formData[field.id] = Array.isArray(field.value) ? [...field.value] : []
  } else {
    formData[field.id] = field.value ?? ''
  }
}

function onFieldValue(field, value) {
  formData[field.id] = value
  field.value = value
  if (errors.value[field.id]) {
    delete errors.value[field.id]
  }
}

function validate() {
  errors.value = {}
  let valid = true
  for (const f of fields.value) {
    const val = formData[f.id]
    if (f.required) {
      const empty = f.type === 'checkbox'
        ? !Array.isArray(val) || val.length === 0
        : val === '' || val === null || val === undefined
      if (empty) {
        errors.value[f.id] = `${f.label}为必填项`
        valid = false
      }
    }
  }
  return valid
}

const showResult = ref(false)
const submitResult = ref('')

function handleSubmit() {
  if (!validate()) {
    showResult.value = false
    return
  }
  // 收集数据，使用 name 作为参数名
  const result = {}
  for (const f of fields.value) {
    const key = f.name || f.label
    result[key] = formData[f.id]
  }
  submitResult.value = JSON.stringify(result, null, 2)
  showResult.value = true
}

function handleReset() {
  for (const f of fields.value) {
    if (f.type === 'checkbox') {
      formData[f.id] = []
      f.value = []
    } else {
      formData[f.id] = ''
      f.value = ''
    }
  }
  errors.value = {}
  showResult.value = false
}

// 导出表单配置 JSON
const exportJson = computed(() =>
  JSON.stringify(
    fields.value.map(f => ({
      type: f.type,
      label: f.label,
      name: f.name,
      required: f.required,
      placeholder: f.placeholder,
      rows: f.rows,
      dateOnly: f.dateOnly,
      maxLength: f.maxLength,
      options: f.options
    })),
    null,
    2
  )
)

const showExport = ref(false)

function copyExport() {
  navigator.clipboard.writeText(exportJson.value)
}
</script>

<template>
  <div class="app">
    <header class="app-header">
      <h1>表单生成器</h1>
      <p>配置字段 → 实时预览 → 收集数据</p>
    </header>

    <main class="app-main">
      <!-- 左：配置区 -->
      <section class="pane pane--config">
        <ConfigPanel
          :fields="fields"
          @add="addField"
          @update="updateField"
          @remove="removeField"
          @move="moveField"
        />
      </section>

      <!-- 右：预览区 -->
      <section class="pane pane--preview">
        <div class="preview-inner">
          <h2 class="panel-title">表单预览</h2>

          <div v-if="fields.length === 0" class="empty-tip">
            请在左侧添加字段来生成表单
          </div>

          <form v-else class="preview-form" @submit.prevent="handleSubmit">
            <div v-for="f in fields" :key="f.id" class="field-wrap">
              <FieldRender
                :field="f"
                @update:value="(v) => onFieldValue(f, v)"
              />
              <p v-if="errors[f.id]" class="error-text">⚠ {{ errors[f.id] }}</p>
            </div>

            <div class="form-actions">
              <button type="submit" class="btn btn--primary">提交</button>
              <button type="button" class="btn btn--ghost" @click="handleReset">重置</button>
            </div>
          </form>

          <div v-if="showResult" class="result-box">
            <div class="result-header">
              <span>提交结果</span>
              <button class="btn btn--small" @click="showResult = false">关闭</button>
            </div>
            <pre>{{ submitResult }}</pre>
          </div>

          <div class="export-section">
            <button class="btn btn--ghost btn--block" @click="showExport = !showExport">
              {{ showExport ? '隐藏' : '查看' }}表单配置 JSON
            </button>
            <div v-if="showExport" class="export-box">
              <div class="result-header">
                <span>配置 JSON</span>
                <button class="btn btn--small" @click="copyExport">复制</button>
            </div>
              <pre>{{ exportJson }}</pre>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background: #fff;
  padding: 18px 28px;
  border-bottom: 1px solid #e5e7eb;
}

.app-header h1 {
  font-size: 20px;
  font-weight: 700;
  color: #111827;
}

.app-header p {
  font-size: 13px;
  color: #6b7280;
  margin-top: 4px;
}

.app-main {
  flex: 1;
  display: grid;
  grid-template-columns: 380px 1fr;
  gap: 20px;
  padding: 20px 28px;
  min-height: 0;
}

.pane {
  background: #fff;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  padding: 18px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.pane--config {
  height: calc(100vh - 140px);
}

.pane--preview {
  height: calc(100vh - 140px);
}

.preview-inner {
  flex: 1;
  overflow-y: auto;
  max-width: 640px;
  margin: 0 auto;
  width: 100%;
}

.panel-title {
  font-size: 16px;
  font-weight: 700;
  margin-bottom: 14px;
  color: #111827;
}

.empty-tip {
  text-align: center;
  color: #9ca3af;
  font-size: 14px;
  padding: 60px 0;
}

.preview-form {
  display: flex;
  flex-direction: column;
}

.error-text {
  color: #dc2626;
  font-size: 12px;
  margin-top: -8px;
  margin-bottom: 12px;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.btn {
  padding: 8px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.15s;
}

.btn--primary {
  background: #6366f1;
  color: #fff;
}

.btn--primary:hover {
  background: #4f46e5;
}

.btn--ghost {
  background: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;
}

.btn--ghost:hover {
  background: #e5e7eb;
}

.btn--block {
  width: 100%;
}

.btn--small {
  padding: 4px 10px;
  font-size: 12px;
  background: #f3f4f6;
  border: 1px solid #d1d5db;
  color: #374151;
}

.result-box,
.export-box {
  margin-top: 20px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
}

.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #f3f4f6;
  border-bottom: 1px solid #e5e7eb;
  font-size: 13px;
  font-weight: 600;
  color: #374151;
}

.result-box pre,
.export-box pre {
  padding: 12px;
  font-size: 12px;
  line-height: 1.6;
  overflow-x: auto;
  color: #065f46;
  font-family: 'SF Mono', Menlo, Consolas, monospace;
}

.export-section {
  margin-top: 24px;
}
</style>
