<template>
  <main class="page">
    <section class="card">
      <FormGenerator :schema="formSchema" v-model="formData" @submit="onSubmit" />
    </section>
  </main>
</template>

<script>
import { defineComponent, reactive } from 'vue';
import FormGenerator from './components/FormGenerator.vue';

export default defineComponent({
  name: 'App',
  components: { FormGenerator },
  setup() {
    const formData = reactive({});
    const formSchema = {
      fields: [
        { type: 'text', label: 'Имя', model: 'name', required: true },
        { type: 'email', label: 'Email', model: 'email', required: true },
        { type: 'password', label: 'Пароль', model: 'password', required: true, minLength: 10 },
        { type: 'select', label: 'Роль', model: 'role', options: ['Админ', 'Пользователь'], required: true },
        { type: 'checkbox', label: 'Согласен с условиями', model: 'terms', required: true }
      ]
    };
    const onSubmit = (payload) => {
      console.log('SUBMIT:', payload);
      alert('Отправлено!');
    };
    return { formData, formSchema, onSubmit };
  }
});
</script>

<style>
:root {
  --bg: #0b1220;
  --panel: #0f172a;
  --panel-2: #111827;
  --text: #e5e7eb;
  --muted: #94a3b8;
  --border: #1f2937;
  --ring: #60a5fa;
  --ring-soft: rgba(96,165,250,.25);
  --accent: #2563eb;
  --shadow: 0 10px 24px rgba(0,0,0,.35), 0 2px 6px rgba(0,0,0,.25);
}

* { box-sizing: border-box; }
html, body, #app { height: 100%; }
body {
  margin: 0;
  font: 16px/1.5 ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
  color: var(--text);
  background:
    radial-gradient(1200px 600px at 10% -10%, rgba(59,130,246,.15), transparent 60%),
    radial-gradient(1000px 500px at 90% 10%, rgba(147,51,234,.12), transparent 60%),
    var(--bg);
}

.page {
  max-width: 920px;
  margin: 40px auto;
  padding: 0 20px 40px;
  display: grid;
  gap: 20px;
  align-content: start;
}
.card {
  background: linear-gradient(180deg, var(--panel) 0%, var(--panel-2) 100%);
  border: 1px solid var(--border);
  border-radius: 16px;
  box-shadow: var(--shadow);
  overflow: clip;
}
</style>
