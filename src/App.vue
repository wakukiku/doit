<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from "vue";
import WeekStrip from "./components/WeekStrip.vue";
import TaskList from "./components/TaskList.vue";
import ProgressCard from "./components/ProgressCard.vue";
import UpcomingCard from "./components/UpcomingCard.vue";
import TaskModal from "./components/TaskModal.vue";

const STORAGE_KEY = "doit.tasks.v2";

function createId() {
  if (
    typeof globalThis !== "undefined" &&
    globalThis.crypto &&
    typeof globalThis.crypto.randomUUID === "function"
  ) {
    return globalThis.crypto.randomUUID();
  }

  return `${Date.now().toString(36)}-${Math.random().toString(36).slice(2, 10)}`;
}

function toISO(date) {
  const localDate = new Date(date);
  localDate.setMinutes(localDate.getMinutes() - localDate.getTimezoneOffset());
  return localDate.toISOString().slice(0, 10);
}

function parseISO(iso) {
  const [year, month, day] = iso.split("-").map(Number);
  return new Date(year, month - 1, day);
}

function addDays(date, amount) {
  const result = new Date(date);
  result.setDate(result.getDate() + amount);
  return result;
}

function startOfWeek(date) {
  const result = new Date(date);
  const day = result.getDay() || 7;
  result.setDate(result.getDate() - day + 1);
  result.setHours(0, 0, 0, 0);
  return result;
}

function safeLoadTasks() {
  try {
    const raw = window.localStorage.getItem(STORAGE_KEY);
    if (!raw) return null;
    const parsed = JSON.parse(raw);
    return Array.isArray(parsed) ? parsed : null;
  } catch {
    return null;
  }
}

function safeSaveTasks(value) {
  try {
    window.localStorage.setItem(STORAGE_KEY, JSON.stringify(value));
  } catch {
    // The app remains usable even when storage is unavailable.
  }
}

const todayDate = new Date();
const todayISO = toISO(todayDate);

const selectedDate = ref(todayISO);
const weekAnchor = ref(startOfWeek(todayDate));
const activeTab = ref("today");
const modalOpen = ref(false);
const editingTask = ref(null);

function seedTasks() {
  const base = parseISO(todayISO);

  return [
    {
      id: createId(),
      title: "Ответить на письма",
      date: toISO(base),
      time: "09:00",
      category: "work",
      completed: true,
    },
    {
      id: createId(),
      title: "Созвон с клиентом",
      date: toISO(base),
      time: "10:00",
      category: "work",
      completed: true,
    },
    {
      id: createId(),
      title: "Подготовить структуру проекта",
      date: toISO(base),
      time: "11:30",
      category: "work",
      completed: false,
    },
    {
      id: createId(),
      title: "Купить продукты",
      date: toISO(base),
      time: "15:00",
      category: "personal",
      completed: false,
    },
    {
      id: createId(),
      title: "Спортзал",
      date: toISO(base),
      time: "18:00",
      category: "personal",
      completed: false,
    },
    {
      id: createId(),
      title: "Почитать книгу",
      date: toISO(base),
      time: "21:00",
      category: "personal",
      completed: false,
    },
    {
      id: createId(),
      title: "Разобрать документы",
      date: toISO(addDays(base, 1)),
      time: "12:00",
      category: "personal",
      completed: false,
    },
    {
      id: createId(),
      title: "Запланировать встречу с командой",
      date: toISO(addDays(base, 1)),
      time: "16:00",
      category: "work",
      completed: false,
    },
    {
      id: createId(),
      title: "Оплатить подписку",
      date: toISO(addDays(base, 2)),
      time: "",
      category: "personal",
      completed: false,
    },
    {
      id: createId(),
      title: "Проверить макеты",
      date: toISO(addDays(base, 3)),
      time: "13:30",
      category: "work",
      completed: false,
    },
  ];
}

const tasks = ref(safeLoadTasks() ?? seedTasks());

watch(tasks, (value) => safeSaveTasks(value), { deep: true });

const tabs = [
  { id: "today", label: "Сегодня", icon: "calendar" },
  { id: "week", label: "Неделя", icon: "week" },
  { id: "work", label: "Работа", icon: "briefcase" },
  { id: "personal", label: "Личное", icon: "person" },
];

const weekDays = computed(() => {
  const labels = ["Пн", "Вт", "Ср", "Чт", "Пт", "Сб", "Вс"];

  return labels.map((weekday, index) => {
    const date = addDays(weekAnchor.value, index);
    const iso = toISO(date);

    return {
      weekday,
      day: date.getDate(),
      iso,
      isToday: iso === todayISO,
    };
  });
});

const currentWeekRange = computed(() => ({
  start: toISO(weekAnchor.value),
  end: toISO(addDays(weekAnchor.value, 6)),
}));

function sortTasks(list) {
  return [...list].sort((a, b) => {
    const dateDifference = a.date.localeCompare(b.date);
    if (dateDifference !== 0) return dateDifference;
    return (a.time || "99:99").localeCompare(b.time || "99:99");
  });
}

const visibleTasks = computed(() => {
  let result = tasks.value;

  if (activeTab.value === "today") {
    result = result.filter((task) => task.date === selectedDate.value);
  } else if (activeTab.value === "week") {
    const { start, end } = currentWeekRange.value;
    result = result.filter((task) => task.date >= start && task.date <= end);
  } else {
    const { start, end } = currentWeekRange.value;
    result = result.filter(
      (task) =>
        task.category === activeTab.value &&
        task.date >= start &&
        task.date <= end,
    );
  }

  return sortTasks(result);
});

const selectedDayTasks = computed(() =>
  tasks.value.filter((task) => task.date === selectedDate.value),
);

const completedSelectedDay = computed(
  () => selectedDayTasks.value.filter((task) => task.completed).length,
);

function dateLabel(iso) {
  if (iso === todayISO) return "Сегодня";

  const tomorrowISO = toISO(addDays(parseISO(todayISO), 1));
  if (iso === tomorrowISO) return "Завтра";

  return new Intl.DateTimeFormat("ru-RU", {
    day: "numeric",
    month: "short",
  })
    .format(parseISO(iso))
    .replace(".", "");
}

const upcomingTasks = computed(() =>
  sortTasks(
    tasks.value.filter((task) => !task.completed && task.date >= todayISO),
  )
    .slice(0, 4)
    .map((task) => ({
      ...task,
      displayDate: dateLabel(task.date),
    })),
);

const headerTitle = computed(() => {
  if (activeTab.value === "week") return "Неделя";
  if (activeTab.value === "work") return "Работа";
  if (activeTab.value === "personal") return "Личное";
  if (selectedDate.value === todayISO) return "Сегодня";

  const weekday = new Intl.DateTimeFormat("ru-RU", {
    weekday: "long",
  }).format(parseISO(selectedDate.value));

  return weekday.charAt(0).toUpperCase() + weekday.slice(1);
});

const headerSubtitle = computed(() => {
  if (activeTab.value === "week") {
    const formatter = new Intl.DateTimeFormat("ru-RU", {
      day: "numeric",
      month: "long",
    });

    return `${formatter.format(weekAnchor.value)} — ${formatter.format(addDays(weekAnchor.value, 6))}`;
  }

  if (activeTab.value === "work") return "Задачи по работе на этой неделе";
  if (activeTab.value === "personal") return "Личные задачи на этой неделе";

  return new Intl.DateTimeFormat("ru-RU", {
    weekday: "long",
    day: "numeric",
    month: "long",
  }).format(parseISO(selectedDate.value));
});

function selectTab(tabId) {
  activeTab.value = tabId;

  if (tabId === "today") {
    selectedDate.value = todayISO;
    weekAnchor.value = startOfWeek(todayDate);
  }
}

function selectDay(iso) {
  selectedDate.value = iso;
  activeTab.value = "today";
}

function moveWeek(direction) {
  weekAnchor.value = addDays(weekAnchor.value, direction * 7);

  if (activeTab.value === "today") {
    selectedDate.value = toISO(weekAnchor.value);
  }
}

function toggleTask(id) {
  const task = tasks.value.find((item) => item.id === id);
  if (task) task.completed = !task.completed;
}

function openCreate() {
  editingTask.value = null;
  modalOpen.value = true;
}

function openEdit(task) {
  editingTask.value = { ...task };
  modalOpen.value = true;
}

function closeModal() {
  modalOpen.value = false;
  editingTask.value = null;
}

function saveTask(payload) {
  if (editingTask.value) {
    const index = tasks.value.findIndex(
      (task) => task.id === editingTask.value.id,
    );

    if (index !== -1) {
      tasks.value[index] = {
        ...tasks.value[index],
        ...payload,
      };
    }
  } else {
    tasks.value.push({
      id: createId(),
      ...payload,
      completed: false,
    });
  }

  closeModal();
}

function deleteTask(id) {
  tasks.value = tasks.value.filter((task) => task.id !== id);
  closeModal();
}

function handleKeydown(event) {
  if (event.key === "Escape" && modalOpen.value) {
    closeModal();
  }

  if ((event.metaKey || event.ctrlKey) && event.key.toLowerCase() === "k") {
    event.preventDefault();
    openCreate();
  }
}

onMounted(() => {
  window.addEventListener("keydown", handleKeydown);
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeydown);
});
</script>

<template>
  <main class="app-shell">
    <section class="app-window">
      <header class="topbar">
        <div class="window-dots" aria-hidden="true">
          <span></span>
          <span></span>
          <span></span>
        </div>
        <div class="brand">doit</div>
        <div class="topbar-spacer"></div>
      </header>

      <div class="workspace">
        <nav class="main-nav" aria-label="Разделы">
          <button
            v-for="tab in tabs"
            :key="tab.id"
            type="button"
            :class="{ active: activeTab === tab.id }"
            @click="selectTab(tab.id)"
          >
            <svg
              v-if="tab.icon === 'calendar'"
              viewBox="0 0 24 24"
              aria-hidden="true"
            >
              <path
                d="M7 3v3m10-3v3M4 9h16M5 5h14a1 1 0 0 1 1 1v14H4V6a1 1 0 0 1 1-1Z"
              />
            </svg>

            <svg
              v-else-if="tab.icon === 'week'"
              viewBox="0 0 24 24"
              aria-hidden="true"
            >
              <path
                d="M7 3v3m10-3v3M4 9h16M5 5h14a1 1 0 0 1 1 1v14H4V6a1 1 0 0 1 1-1Zm3 8h2m2 0h2m2 0h2"
              />
            </svg>

            <svg
              v-else-if="tab.icon === 'briefcase'"
              viewBox="0 0 24 24"
              aria-hidden="true"
            >
              <path
                d="M9 6V4h6v2m-11 4h16m-16-3h16v12H4V7Zm0 5c5 2 11 2 16 0"
              />
            </svg>

            <svg v-else viewBox="0 0 24 24" aria-hidden="true">
              <circle cx="12" cy="8" r="3" />
              <path d="M5 20c.5-4 3-6 7-6s6.5 2 7 6" />
            </svg>

            <span>{{ tab.label }}</span>
          </button>
        </nav>

        <WeekStrip
          :days="weekDays"
          :selected-date="selectedDate"
          @select="selectDay"
          @previous="moveWeek(-1)"
          @next="moveWeek(1)"
        />

        <div class="content-grid">
          <TaskList
            :tasks="visibleTasks"
            :title="headerTitle"
            :subtitle="headerSubtitle"
            @toggle="toggleTask"
            @edit="openEdit"
            @add="openCreate"
          />

          <aside class="sidebar">
            <ProgressCard
              :total="selectedDayTasks.length"
              :completed="completedSelectedDay"
            />
            <UpcomingCard
              :tasks="upcomingTasks"
              @toggle="toggleTask"
              @edit="openEdit"
            />
          </aside>
        </div>
      </div>
    </section>

    <TaskModal
      :open="modalOpen"
      :task="editingTask"
      :default-date="selectedDate"
      @close="closeModal"
      @save="saveTask"
      @delete="deleteTask"
    />
  </main>
</template>
