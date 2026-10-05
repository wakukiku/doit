<script setup>
defineProps({
  tasks: { type: Array, required: true },
});

const emit = defineEmits(["toggle", "edit"]);
</script>

<template>
  <section class="side-card card upcoming-card">
    <h2>Ближайшие задачи</h2>

    <div v-if="tasks.length" class="upcoming-list">
      <article v-for="task in tasks" :key="task.id" class="upcoming-row">
        <button
          class="checkbox mini"
          type="button"
          :aria-label="`Выполнить: ${task.title}`"
          @click="emit('toggle', task.id)"
        ></button>

        <button class="upcoming-copy" type="button" @click="emit('edit', task)">
          <span>{{ task.title }}</span>
          <small>
            {{ task.displayDate
            }}<template v-if="task.time"> · {{ task.time }}</template>
          </small>
        </button>
      </article>
    </div>

    <p v-else class="side-empty">Ближайших задач нет</p>
  </section>
</template>
