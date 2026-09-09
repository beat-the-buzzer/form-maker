<script setup>
import { reactive } from 'vue'

const props = defineProps({
  fields: { type: Array, required: true }
})

const emit = defineEmits(['add', 'update', 'remove', 'move'])

const FIELD_TYPES = [
  { type: 'input', label: '输入框' },
  { type: 'textarea', label: '文本域' },
  { type: 'date', label: '日期' },
  { type: 'radio', label: '单选' },
  { type: 'checkbox', label: '多选' }
]

// 无选项的字段类型
const NO_OPTIONS_TYPES = ['input', 'textarea', 'date']

// 添加新字段时的默认标签
const newLabel = reactive({ value: '' })

function addField(type) {
  const id = 'f_' + Date.now() + '_' + Math.random().toString(36).slice(2, 7)
  const field = {
    id,
    type,
    label: newLabel.value || defaultLabel(type),
    name: 'field_' + id.slice(2),
    required: false,
    placeholder: '',
    rows: type === 'textarea' ? 3 : undefined,
    dateOnly: type === 'date' ? true : undefined,
    maxLength: (type === 'input' || type === 'textarea') ? 0 : undefined,
    options: NO_OPTIONS_TYPES.includes(type) ? [] : [{ label: '选项1', value: 'opt_1' }, { label: '选项2', value: 'opt_2' }],
    value: type === 'checkbox' ? [] : ''
  }
  emit('add', field)
  newLabel.value = ''
}

function defaultLabel(type) {
  const map = { input: '输入框', textarea: '文本域', date: '日期', radio: '单选题', checkbox: '多选题' }
  return map[type] || '字段'
}

function typeLabel(type) {
  const map = { input: '输入框', textarea: '文本域', date: '日期', radio: '单选', checkbox: '多选' }
  return map[type] || type
}

function updateField(index, patch) {
  emit('update', index, patch)
}

function addOption(field) {
  const idx = field.options.length + 1
  field.options.push({ label: `选项${idx}`, value: `opt_${Date.now()}_${idx}` })
}

function removeOption(field, idx) {
  field.options.splice(idx, 1)
}
</script>

<template>
  <div class="config-panel">
    <h2 class="panel-title">字段配置</h2>

    <!-- 添加字段区 -->
    <div class="add-section">
      <input
        v-model="newLabel.value"
        class="label-input"
        placeholder="字段名称（可选，留空使用默认）"
      />
      <div class="add-btns">
        <button
          v-for="t in FIELD_TYPES"
          :key="t.type"
          class="add-btn"
          :class="'add-btn--' + t.type"
          @click="addField(t.type)"
        >
          + {{ t.label }}
        </button>
      </div>
    </div>

    <!-- 字段列表 -->
    <div v-if="fields.length === 0" class="empty-tip">
      还没有字段，点击上方按钮添加。
    </div>

    <div v-else class="field-list">
      <div v-for="(field, index) in fields" :key="field.id" class="field-card">
        <div class="card-header">
          <span class="card-type-tag" :class="'tag--' + field.type">
            {{ typeLabel(field.type) }}
          </span>
          <span class="card-title">{{ field.label }}</span>
          <div class="card-actions">
            <button class="icon-btn" :disabled="index === 0" title="上移" @click="emit('move', index, -1)">↑</button>
            <button class="icon-btn" :disabled="index === fields.length - 1" title="下移" @click="emit('move', index, 1)">↓</button>
            <button class="icon-btn icon-btn--danger" title="删除" @click="emit('remove', index)">✕</button>
          </div>
        </div>

        <div class="card-body">
          <div class="form-row">
            <label>字段名称</label>
            <input
              :value="field.label"
              class="text-input"
              @input="updateField(index, { label: $event.target.value })"
            />
          </div>

          <div class="form-row">
            <label>参数名 (name)</label>
            <input
              :value="field.name"
              class="text-input"
              placeholder="提交时使用的参数名"
              @input="updateField(index, { name: $event.target.value })"
            />
          </div>

          <div v-if="field.type === 'input' || field.type === 'textarea' || field.type === 'date'" class="form-row">
            <label>占位提示</label>
            <input
              :value="field.placeholder"
              class="text-input"
              @input="updateField(index, { placeholder: $event.target.value })"
            />
          </div>

          <div v-if="field.type === 'input' || field.type === 'textarea'" class="form-row">
            <label>最大字数（0 表示不限制）</label>
            <input
              :value="field.maxLength"
              type="number"
              min="0"
              class="text-input"
              @input="updateField(index, { maxLength: Math.max(0, Number($event.target.value) || 0) })"
            />
          </div>

          <div v-if="field.type === 'textarea'" class="form-row">
            <label>行数</label>
            <input
              :value="field.rows"
              type="number"
              min="1"
              max="20"
              class="text-input"
              @input="updateField(index, { rows: Number($event.target.value) || 3 })"
            />
          </div>

          <div v-if="field.type === 'date'" class="form-row form-row--inline">
            <label class="checkbox-label">
              <input
                type="checkbox"
                :checked="field.dateOnly"
                @change="updateField(index, { dateOnly: $event.target.checked })"
              />
              <span>仅日期（不含时间）</span>
            </label>
          </div>

          <div v-if="field.type === 'radio' || field.type === 'checkbox'" class="form-row">
            <label>选项配置</label>
            <div class="options-editor">
              <div v-for="(opt, i) in field.options" :key="i" class="option-edit-row">
                <input
                  :value="opt.label"
                  class="text-input"
                  placeholder="选项标签"
                  @input="updateField(index, { options: field.options.map((o, oi) => oi === i ? { ...o, label: $event.target.value } : o) })"
                />
                <input
                  :value="opt.value"
                  class="text-input opt-value"
                  placeholder="选项值"
                  @input="updateField(index, { options: field.options.map((o, oi) => oi === i ? { ...o, value: $event.target.value } : o) })"
                />
                <button class="icon-btn icon-btn--danger" @click="removeOption(field, i)">✕</button>
              </div>
              <button class="add-option-btn" @click="addOption(field)">+ 添加选项</button>
            </div>
          </div>

          <div class="form-row form-row--inline">
            <label class="checkbox-label">
              <input
                type="checkbox"
                :checked="field.required"
                @change="updateField(index, { required: $event.target.checked })"
              />
              <span>必填</span>
            </label>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.config-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  min-height: 0;
  overflow: hidden;
}

.panel-title {
  font-size: 16px;
  font-weight: 700;
  margin-bottom: 14px;
  color: #111827;
  flex-shrink: 0;
}

.add-section {
  padding: 14px;
  background: #fff;
  border-radius: 8px;
  margin-bottom: 16px;
  border: 1px solid #e5e7eb;
  flex-shrink: 0;
}

.label-input,
.text-input {
  width: 100%;
  padding: 7px 10px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 13px;
  outline: none;
}

.label-input {
  margin-bottom: 10px;
}

.text-input:focus {
  border-color: #6366f1;
}

.add-btns {
  display: flex;
  gap: 8px;
}

.add-btn {
  flex: 1;
  padding: 8px 0;
  border: none;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 600;
  color: #fff;
  transition: opacity 0.2s;
}

.add-btn:hover {
  opacity: 0.85;
}

.add-btn--input {
  background: #6366f1;
}

.add-btn--textarea {
  background: #8b5cf6;
}

.add-btn--date {
  background: #0ea5e9;
}

.add-btn--radio {
  background: #10b981;
}

.add-btn--checkbox {
  background: #f59e0b;
}

.empty-tip {
  text-align: center;
  color: #9ca3af;
  font-size: 13px;
  padding: 30px 0;
  flex-shrink: 0;
}

.field-list {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.field-card {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
  flex-shrink: 0;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  background: #f9fafb;
  border-bottom: 1px solid #e5e7eb;
}

.card-type-tag {
  font-size: 11px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 4px;
  color: #fff;
}

.tag--input {
  background: #6366f1;
}

.tag--textarea {
  background: #8b5cf6;
}

.tag--date {
  background: #0ea5e9;
}

.tag--radio {
  background: #10b981;
}

.tag--checkbox {
  background: #f59e0b;
}

.card-title {
  flex: 1;
  font-size: 13px;
  font-weight: 600;
  color: #1f2937;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.card-actions {
  display: flex;
  gap: 4px;
}

.icon-btn {
  width: 26px;
  height: 26px;
  border: 1px solid #e5e7eb;
  background: #fff;
  border-radius: 4px;
  font-size: 13px;
  color: #6b7280;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.15s;
}

.icon-btn:hover:not(:disabled) {
  background: #f3f4f6;
  color: #111827;
}

.icon-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.icon-btn--danger:hover {
  background: #fef2f2;
  color: #dc2626;
  border-color: #fecaca;
}

.card-body {
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.form-row {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.form-row--inline {
  flex-direction: row;
  align-items: center;
}

.form-row label {
  font-size: 12px;
  color: #6b7280;
  font-weight: 600;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 6px;
  cursor: pointer;
  font-size: 13px !important;
  color: #374151 !important;
}

.options-editor {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.option-edit-row {
  display: flex;
  gap: 6px;
}

.option-edit-row .text-input {
  flex: 1;
}

.opt-value {
  flex: 0.8 !important;
}

.add-option-btn {
  align-self: flex-start;
  padding: 5px 12px;
  border: 1px dashed #d1d5db;
  background: #fff;
  border-radius: 6px;
  font-size: 12px;
  color: #6366f1;
}

.add-option-btn:hover {
  border-color: #6366f1;
  background: #eef2ff;
}
</style>
