# autoform-project

Компонент автоформы на **Vue 3**, который строит форму по переданной JSON-схеме, поддерживает поля `text`, `email`, `password`, `select`, `checkbox`, выполняет минимальную валидацию (`required`, `minLength`, `pattern`) и синхронно пишет значения в объект через `v-model`.

## Реализация
- `src/components/FormGenerator.vue` — генератор формы по JSON-схеме, рендер нужных типов, валидация, `v-model`, событие `submit`.
- `src/components/FormField.vue` — атомарный компонент одного поля с валидацией.


## Установка и запуск
```
npm install
```

```
npm run serve
```

Перейдите по URL, указанному в выводе консоли после запуска сервера (по умолчанию — http://localhost:8080).