<script setup>
defineProps({
  days: { type: Array, required: true },
  selectedDate: { type: String, required: true },
});

const emit = defineEmits(["select", "previous", "next"]);
</script>

<template>
  <section class="week-strip panel-soft" aria-label="Неделя">
    <div class="week-days">
      <button
        v-for="day in days"
        :key="day.iso"
        class="day-button"
        :class="{ active: day.iso === selectedDate, today: day.isToday }"
        type="button"
        @click="emit('select', day.iso)"
      >
        <span class="day-weekday">{{ day.weekday }}</span>
        <span class="day-number">{{ day.day }}</span>
        <span class="day-dot" aria-hidden="true"></span>
      </button>
    </div>

    <div class="week-arrows">
      <button
        class="icon-button"
        type="button"
        aria-label="Предыдущая неделя"
        @click="emit('previous')"
      >
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="m15 18-6-6 6-6" />
        </svg>
      </button>

      <button
        class="icon-button"
        type="button"
        aria-label="Следующая неделя"
        @click="emit('next')"
      >
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="m9 18 6-6-6-6" />
        </svg>
      </button>
    </div>
  </section>
</template>
