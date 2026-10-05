<script setup>
import { reactive, watch } from "vue";

const props = defineProps({
  open: { type: Boolean, required: true },
  task: { type: Object, default: null },
  defaultDate: { type: String, required: true },
});

const emit = defineEmits(["close", "save", "delete"]);

const form = reactive({
  title: "",
  date: props.defaultDate,
  time: "",
  category: "work",
});

watch(
  () => [props.open, props.task, props.defaultDate],
  () => {
    if (!props.open) return;

    form.title = props.task?.title ?? "";
    form.date = props.task?.date ?? props.defaultDate;
    form.time = props.task?.time ?? "";
    form.category = props.task?.category ?? "work";
  },
  { immediate: true },
);

function submit() {
  const title = form.title.trim();
  if (!title || !form.date) return;

  emit("save", {
    title,
    date: form.date,
    time: form.time,
    category: form.category,
  });
}
</script>

<template>
  <Teleport to="body">
    <Transition name="modal-fade">
      <div v-if="open" class="modal-backdrop" @mousedown.self="emit('close')">
        <section
          class="modal-card"
          role="dialog"
          aria-modal="true"
          :aria-label="task ? 'Редактировать задачу' : 'Новая задача'"
        >
          <header class="modal-header">
            <h2>{{ task ? "Редактировать задачу" : "Новая задача" }}</h2>
            <button
              class="modal-close"
              type="button"
              aria-label="Закрыть"
              @click="emit('close')"
            >
              ×
            </button>
          </header>

          <form @submit.prevent="submit">
            <label class="field wide">
              <span>Задача</span>
              <input
                v-model="form.title"
                autofocus
                maxlength="90"
                placeholder="Что нужно сделать?"
              />
            </label>

            <div class="form-grid">
              <label class="field">
                <span>Дата</span>
                <input v-model="form.date" type="date" required />
              </label>

              <label class="field">
                <span>Время</span>
                <input v-model="form.time" type="time" />
              </label>
            </div>

            <fieldset class="category-choice">
              <legend>Категория</legend>

              <label :class="{ selected: form.category === 'work' }">
                <input v-model="form.category" type="radio" value="work" />
                Работа
              </label>

              <label :class="{ selected: form.category === 'personal' }">
                <input v-model="form.category" type="radio" value="personal" />
                Личное
              </label>
            </fieldset>

            <footer class="modal-actions">
              <button
                v-if="task"
                class="danger-button"
                type="button"
                @click="emit('delete', task.id)"
              >
                Удалить
              </button>
              <span v-else></span>

              <div>
                <button
                  class="secondary-button"
                  type="button"
                  @click="emit('close')"
                >
                  Отмена
                </button>
                <button class="primary-button modal-save" type="submit">
                  Сохранить
                </button>
              </div>
            </footer>
          </form>
        </section>
      </div>
    </Transition>
  </Teleport>
</template>
