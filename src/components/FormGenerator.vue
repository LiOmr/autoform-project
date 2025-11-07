<template>
  <form class="autoform" @submit.prevent="onSubmitInternal">
    <div v-for="(field, idx) in schema.fields" :key="idx" class="row">
      <label v-if="field.type !== 'checkbox' && field.label" :for="field.model" class="label">
        {{ field.label }}<span v-if="field.required" class="req" aria-hidden="true">*</span>
      </label>
      <input
        v-if="isTextLike(field.type)"
        :id="field.model"
        class="control"
        :class="{ 'is-error': showErr(field) }"
        :type="field.type"
        v-model="internal[field.model]"
        :placeholder="field.placeholder || ''"
        :required="!!field.required"
        :minlength="field.minLength"
        :pattern="field.pattern"
        :aria-invalid="showErr(field) ? 'true' : 'false'"
        :aria-describedby="showErr(field) ? ('err-' + field.model) : undefined"
        @blur="touch(field.model)"
      />
      <select
        v-else-if="field.type === 'select'"
        :id="field.model"
        class="control"
        :class="{ 'is-error': showErr(field) }"
        v-model="internal[field.model]"
        :required="!!field.required"
        :aria-invalid="showErr(field) ? 'true' : 'false'"
        :aria-describedby="showErr(field) ? ('err-' + field.model) : undefined"
        @blur="touch(field.model)"
      >
        <option value="" disabled v-if="field.required && !internal[field.model]">
          {{ field.placeholder || 'Выберите…' }}
        </option>
        <option v-for="(opt, i) in (field.options || [])" :key="i" :value="opt">{{ opt }}</option>
      </select>
      <label v-else-if="field.type === 'checkbox'" class="checkbox" :for="field.model">
        <input
          type="checkbox"
          :id="field.model"
          v-model="internal[field.model]"
          :required="!!field.required"
          :aria-invalid="showErr(field) ? 'true' : 'false'"
          :aria-describedby="showErr(field) ? ('err-' + field.model) : undefined"
          @blur="touch(field.model)"
        />
        <span class="checkbox__label">
          {{ field.label }}<span v-if="field.required" class="req" aria-hidden="true">*</span>
        </span>
      </label>
      <p v-if="(touched[field.model] || submitAttempted) && fieldError(field)" class="error" :id="'err-' + field.model" aria-live="polite">
        {{ fieldError(field) }}
      </p>
    </div>
    <button type="submit" class="btn"><span class="btn__glow"></span>Отправить</button>
  </form>
</template>

<script>
import { defineComponent, reactive, watch, ref } from 'vue';

export default defineComponent({
  name: 'FormGenerator',
  props: {
    schema: { type: Object, required: true },
    modelValue: { type: Object, default: () => ({}) }
  },
  emits: ['update:modelValue', 'submit'],
  setup(props, { emit }) {
    const internal = reactive({});
    const touched = reactive({});
    const submitAttempted = ref(false);

    const initFromSchema = () => {
      const next = {};
      for (const f of props.schema?.fields || []) {
        const hasExternal = props.modelValue && Object.prototype.hasOwnProperty.call(props.modelValue, f.model);
        if (hasExternal) next[f.model] = props.modelValue[f.model];
        else if (f.type === 'checkbox') next[f.model] = false;
        else next[f.model] = '';
      }
      Object.keys(internal).forEach(k => delete internal[k]);
      Object.assign(internal, next);
      submitAttempted.value = false;
      Object.keys(touched).forEach(k => delete touched[k]);
    };

    watch(() => props.schema, initFromSchema, { deep: true, immediate: true });

    watch(internal, (val) => {
      emit('update:modelValue', { ...val });
    }, { deep: true, immediate: true });

    watch(() => props.modelValue, (nv) => {
      if (!nv) return;
      Object.keys(nv).forEach(k => (internal[k] = nv[k]));
    }, { deep: true });

    const fieldError = (f) => {
      const v = internal[f.model];
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
    };

    const isTextLike = (t) => t === 'text' || t === 'email' || t === 'password';
    const showErr = (f) => (touched[f.model] || submitAttempted.value) && !!fieldError(f);
    const touch = (key) => (touched[key] = true);
    const validateAll = () => (props.schema.fields || []).every(f => !fieldError(f));

    const onSubmitInternal = () => {
      submitAttempted.value = true;
      if (!validateAll()) {
        const first = (props.schema.fields || []).find(f => fieldError(f));
        if (first && first.model) {
          const el = document.getElementById(first.model);
          if (el && typeof el.focus === 'function') el.focus();
        }
        return;
      }
      emit('submit', { ...internal });
    };

    return { internal, touched, submitAttempted, fieldError, isTextLike, showErr, touch, onSubmitInternal };
  }
});
</script>

<style scoped>
.autoform {
  --text: #e5e7eb;
  --muted: #94a3b8;
  --border: #1f2937;
  --ring: #60a5fa;
  --ring-soft: rgba(96,165,250,.25);
  padding: 18px;
  display: grid;
  gap: 14px;
}
.row { display: grid; gap: 8px; }
.label { font-weight: 600; font-size: 14px; color: var(--text); letter-spacing: .2px; }
.req { color: var(--muted); margin-left: 4px; }
.control {
  width: 100%;
  padding: 12px 14px;
  font-size: 14px;
  color: var(--text);
  background: rgba(255,255,255,.02);
  border: 1px solid var(--border);
  border-radius: 12px;
  outline: none;
  transition: border-color .15s ease, box-shadow .15s ease, background-color .15s ease, transform .06s ease;
  backdrop-filter: blur(6px);
}
.control:hover { border-color: #334155; }
.control:focus-visible { border-color: var(--ring); box-shadow: 0 0 0 4px var(--ring-soft); }
.checkbox { display: inline-flex; align-items: center; gap: 10px; user-select: none; }
.checkbox input[type="checkbox"] { inline-size: 18px; block-size: 18px; border-radius: 6px; accent-color: var(--ring); }
.checkbox__label { font-size: 14px; }
.control.is-error { border-color: #ef4444; box-shadow: 0 0 0 4px rgba(239,68,68,.2); }
.error { margin: -2px 2px 0; color: #fca5a5; font-size: 12px; line-height: 1.3; }
.btn {
  position: relative;
  justify-self: start;
  margin-top: 6px;
  padding: 12px 18px;
  font-weight: 700;
  font-size: 14px;
  color: white;
  background: linear-gradient(180deg, #60a5fa 0%, #2563eb 100%);
  border: none;
  border-radius: 12px;
  cursor: pointer;
  box-shadow: 0 8px 22px rgba(37,99,235,.35);
  transition: transform .06s ease, filter .15s ease, box-shadow .15s ease;
}
.btn:hover { filter: brightness(1.03); }
.btn:active { transform: translateY(1px); box-shadow: 0 6px 18px rgba(37,99,235,.45); }
.btn__glow {
  position: absolute;
  inset: -1px;
  border-radius: 12px;
  background:
    radial-gradient(180px 60px at 10% 0%, rgba(255,255,255,.25), transparent 60%),
    radial-gradient(260px 60px at 80% 100%, rgba(59,130,246,.25), transparent 60%);
  mix-blend-mode: screen;
  pointer-events: none;
}
@media (prefers-reduced-motion: reduce) {
  .control, .btn { transition: none; }
}
</style>
