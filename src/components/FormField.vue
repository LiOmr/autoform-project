<template>
  <div class="row">
    <label v-if="field.type !== 'checkbox' && field.label" :for="field.model" class="label">
      {{ field.label }}<span v-if="field.required" class="req" aria-hidden="true">*</span>
    </label>
    <input
      v-if="isTextLike"
      :id="field.model"
      :type="field.type"
      class="control"
      :class="{ 'is-error': showError }"
      :placeholder="field.placeholder || ''"
      v-model="value"
      :required="!!field.required"
      :minlength="field.minLength"
      :pattern="field.pattern"
      :aria-invalid="showError ? 'true' : 'false'"
      :aria-describedby="showError ? ('err-' + field.model) : undefined"
      @blur="touch"
    />
    <select
      v-else-if="field.type === 'select'"
      :id="field.model"
      class="control"
      :class="{ 'is-error': showError }"
      v-model="value"
      :required="!!field.required"
      :aria-invalid="showError ? 'true' : 'false'"
      :aria-describedby="showError ? ('err-' + field.model) : undefined"
      @blur="touch"
    >
      <option value="" disabled v-if="field.required && !value">
        {{ field.placeholder || 'Выберите…' }}
      </option>
      <option v-for="(opt, idx) in field.options || []" :key="idx" :value="opt">
        {{ opt }}
      </option>
    </select>
    <label v-else-if="field.type === 'checkbox'" class="checkbox" :for="field.model">
      <input
        type="checkbox"
        :id="field.model"
        v-model="value"
        :required="!!field.required"
        :aria-invalid="showError ? 'true' : 'false'"
        :aria-describedby="showError ? ('err-' + field.model) : undefined"
        @blur="touch"
      />
      <span class="checkbox__label">
        {{ field.label }}<span v-if="field.required" class="req" aria-hidden="true">*</span>
      </span>
    </label>
    <p v-if="(touched || showErrors) && error" class="error" :id="'err-' + field.model" aria-live="polite">
      {{ error }}
    </p>
  </div>
</template>

<script>
import { defineComponent, computed, ref } from 'vue';

export default defineComponent({
  name: 'FormField',
  props: {
    field: { type: Object, required: true },
    modelValue: { type: [String, Boolean, Number, Object], default: '' },
    showErrors: { type: Boolean, default: false }
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    const value = computed({
      get: () => props.modelValue,
      set: v => emit('update:modelValue', v)
    });
    const isTextLike = computed(() => ['text', 'email', 'password'].includes(props.field.type));
    const touched = ref(false);
    const touch = () => (touched.value = true);
    const error = computed(() => {
      const f = props.field;
      const v = value.value;
      if (f.type === 'checkbox') {
        if (f.required && v !== true) return 'Нужно согласиться с условиями';
        return '';
      }
      const str = (v ?? '').toString();
      if (f.required && str.trim().length === 0) return 'Поле обязательно';
      if (f.minLength && str.length < f.minLength) return `Минимальная длина: ${f.minLength}`;
      if (f.pattern) {
        try {
          const re = new RegExp(f.pattern);
          if (!re.test(str)) return 'Неверный формат';
        } catch (e) { void e; }
      }
      if (f.type === 'email' && str) {
        const emailRE = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRE.test(str)) return 'Введите корректный email';
      }
      return '';
    });
    const showError = computed(() => (touched.value || props.showErrors) && !!error.value);
    return { value, isTextLike, error, showErrors: props.showErrors, touched, touch, showError };
  }
});
</script>

<style scoped>
.row { display: grid; gap: 8px; }
.label { font-weight: 600; font-size: 14px; color: #e5e7eb; letter-spacing: .2px; }
.req { color: #94a3b8; margin-left: 4px; }
.control {
  width: 100%;
  padding: 12px 14px;
  font-size: 14px;
  color: #e5e7eb;
  background: rgba(255,255,255,.02);
  border: 1px solid #1f2937;
  border-radius: 12px;
  outline: none;
  transition: border-color .15s ease, box-shadow .15s ease, background-color .15s ease, transform .06s ease;
  backdrop-filter: blur(6px);
}
.control:hover { border-color: #334155; }
.control:focus-visible { border-color: #60a5fa; box-shadow: 0 0 0 4px rgba(96,165,250,.25); }
.checkbox { display: inline-flex; align-items: center; gap: 10px; user-select: none; }
.checkbox input[type="checkbox"] { inline-size: 18px; block-size: 18px; border-radius: 6px; accent-color: #60a5fa; }
.checkbox__label { font-size: 14px; }
.control.is-error { border-color: #ef4444; box-shadow: 0 0 0 4px rgba(239,68,68,.2); }
.error { margin: -2px 2px 0; color: #fca5a5; font-size: 12px; line-height: 1.3; }
@media (prefers-reduced-motion: reduce) { .control { transition: none; } }
</style>
