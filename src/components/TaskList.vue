<script setup>
defineProps({
  tasks: { type: Array, required: true },
  title: { type: String, required: true },
  subtitle: { type: String, default: "" },
  emptyText: { type: String, default: "Задач нет" },
});

const emit = defineEmits(["toggle", "edit", "add"]);
</script>

<template>
  <section class="task-panel card">
    <header class="task-panel-header">
      <div>
        <h1>{{ title }}</h1>
        <p v-if="subtitle">{{ subtitle }}</p>
      </div>

      <button class="primary-button" type="button" @click="emit('add')">
        <span class="plus" aria-hidden="true">+</span>
        <span class="button-label">Новая задача</span>
      </button>
    </header>

    <div v-if="tasks.length" class="task-list">
      <article
        v-for="task in tasks"
        :key="task.id"
        class="task-row"
        :class="{ completed: task.completed }"
      >
        <button
          class="checkbox"
          :class="{ checked: task.completed }"
          type="button"
          :aria-label="task.completed ? 'Вернуть задачу' : 'Выполнить задачу'"
          @click="emit('toggle', task.id)"
        >
          <svg v-if="task.completed" viewBox="0 0 20 20" aria-hidden="true">
            <path d="m5 10.5 3 3 7-7" />
          </svg>
        </button>

        <button class="task-main" type="button" @click="emit('edit', task)">
          <span class="task-title">{{ task.title }}</span>
          <span class="task-meta">
            <span v-if="task.time">{{ task.time }}</span>
            <span class="category-pill" :class="task.category">
              {{ task.category === "work" ? "Работа" : "Личное" }}
            </span>
          </span>
        </button>

        <button
          class="more-button"
          type="button"
          aria-label="Редактировать задачу"
          @click="emit('edit', task)"
        >
          <span></span>
          <span></span>
          <span></span>
        </button>
      </article>
    </div>

    <div v-else class="empty-state">
      <span>{{ emptyText }}</span>
      <button type="button" @click="emit('add')">Добавить задачу</button>
    </div>
  </section>
</template>
