<script setup>
const props = defineProps({
  field: { type: Object, required: true }
})

const emit = defineEmits(['update:value'])

function onInput(e) {
  emit('update:value', e.target.value)
}

function onCheck(e) {
  const current = Array.isArray(props.field.value) ? [...props.field.value] : []
  const val = e.target.value
  const idx = current.indexOf(val)
  if (idx === -1) current.push(val)
  else current.splice(idx, 1)
  emit('update:value', current)
}
</script>

<template>
  <div class="field-render">
    <label class="field-label">
      {{ field.label }}
      <span v-if="field.required" class="required">*</span>
    </label>

    <!-- 输入框 -->
    <input
      v-if="field.type === 'input'"
      class="form-input"
      type="text"
      :name="field.name"
      :placeholder="field.placeholder"
      :maxlength="field.maxLength > 0 ? field.maxLength : undefined"
      :value="field.value"
      @input="onInput"
    />
    <div v-if="field.type === 'input' && field.maxLength > 0" class="char-count">
      {{ (field.value || '').length }} / {{ field.maxLength }}
    </div>

    <!-- 文本域 -->
    <textarea
      v-else-if="field.type === 'textarea'"
      class="form-input form-textarea"
      :name="field.name"
      :rows="field.rows || 3"
      :placeholder="field.placeholder"
      :maxlength="field.maxLength > 0 ? field.maxLength : undefined"
      :value="field.value"
      @input="onInput"
    ></textarea>
    <div v-if="field.type === 'textarea' && field.maxLength > 0" class="char-count">
      {{ (field.value || '').length }} / {{ field.maxLength }}
    </div>

    <!-- 日期 -->
    <input
      v-else-if="field.type === 'date'"
      class="form-input"
      :type="field.dateOnly ? 'date' : 'datetime-local'"
      :name="field.name"
      :value="field.value"
      @input="onInput"
    />

    <!-- 单选 -->
    <div v-else-if="field.type === 'radio'" class="options-group">
      <label v-for="opt in field.options" :key="opt.value" class="option-item">
        <input
          type="radio"
          :name="field.name"
          :value="opt.value"
          :checked="field.value === opt.value"
          @change="onInput"
        />
        <span>{{ opt.label }}</span>
      </label>
    </div>

    <!-- 多选 -->
    <div v-else-if="field.type === 'checkbox'" class="options-group">
      <label v-for="opt in field.options" :key="opt.value" class="option-item">
        <input
          type="checkbox"
          :name="field.name"
          :value="opt.value"
          :checked="Array.isArray(field.value) && field.value.includes(opt.value)"
          @change="onCheck"
        />
        <span>{{ opt.label }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.field-render {
  margin-bottom: 18px;
}

.field-label {
  display: block;
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 8px;
  color: #374151;
}

.required {
  color: #ef4444;
  margin-left: 2px;
}

.form-input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  outline: none;
  transition: border-color 0.2s;
}

.form-input:focus {
  border-color: #6366f1;
}

.form-textarea {
  resize: vertical;
  font-family: inherit;
  line-height: 1.5;
}

.char-count {
  margin-top: 4px;
  font-size: 12px;
  color: #6b7280;
  text-align: right;
}

.options-group {
  display: flex;
  flex-wrap: wrap;
  gap: 12px 20px;
}

.option-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #374151;
  cursor: pointer;
}

.option-item input {
  cursor: pointer;
}
</style>
