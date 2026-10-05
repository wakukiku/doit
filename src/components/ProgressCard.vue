<script setup>
import { computed, onMounted, ref, watch } from "vue";

const props = defineProps({
  total: { type: Number, required: true },
  completed: { type: Number, required: true },
});

const RADIUS = 82;
const CIRCUMFERENCE = 2 * Math.PI * RADIUS;

const percent = computed(() => {
  if (props.total <= 0) return 0;
  return Math.min(100, Math.max(0, (props.completed / props.total) * 100));
});

// Keep the displayed progress separate so the ring has a real value to animate
// from. It also gives the first render a soft fill-in rather than appearing full.
const animatedPercent = ref(0);

onMounted(() => {
  requestAnimationFrame(() => {
    animatedPercent.value = percent.value;
  });
});

watch(percent, (value) => {
  animatedPercent.value = value;
});

const dashOffset = computed(
  () => CIRCUMFERENCE * (1 - animatedPercent.value / 100),
);
</script>

<template>
  <section class="side-card card progress-card">
    <h2>Прогресс дня</h2>

    <div
      class="progress-ring"
      role="img"
      :aria-label="`Выполнено ${completed} из ${total}`"
    >
      <svg class="progress-ring-svg" viewBox="0 0 200 200" aria-hidden="true">
        <circle class="progress-ring-track" cx="100" cy="100" :r="RADIUS" />
        <circle
          class="progress-ring-value"
          cx="100"
          cy="100"
          :r="RADIUS"
          :stroke-dasharray="CIRCUMFERENCE"
          :stroke-dashoffset="dashOffset"
        />
      </svg>

      <div class="progress-ring-inner">
        <div class="progress-number">
          <Transition name="progress-count" mode="out-in">
            <strong :key="`${completed}-${total}`">
              {{ completed }} из {{ total }}
            </strong>
          </Transition>
        </div>
        <span>задач</span>
      </div>
    </div>
  </section>
</template>
