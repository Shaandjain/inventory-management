<template>
  <div class="app">
    <!-- Sidebar -->
    <aside class="sidebar">
      <div class="sidebar-logo">
        <svg class="logo-mark" width="28" height="28" viewBox="0 0 24 24" fill="none" aria-hidden="true">
          <rect x="3" y="3" width="7" height="7" rx="1.5" fill="currentColor" />
          <rect x="14" y="3" width="7" height="7" rx="1.5" fill="currentColor" opacity="0.7" />
          <rect x="3" y="14" width="7" height="7" rx="1.5" fill="currentColor" opacity="0.7" />
          <rect x="14" y="14" width="7" height="7" rx="1.5" fill="currentColor" opacity="0.4" />
        </svg>
        <div class="logo-text">
          <div class="logo-name">{{ t('nav.companyName') }}</div>
          <div class="logo-sub">{{ t('nav.subtitle') }}</div>
        </div>
      </div>

      <nav class="sidebar-nav" aria-label="Main">
        <router-link to="/" :class="{ active: $route.path === '/' }">
          <NavIcon name="grid" />
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
          <NavIcon name="box" />
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
          <NavIcon name="receipt" />
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
          <NavIcon name="wallet" />
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
          <NavIcon name="trend" />
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
          <NavIcon name="doc" />
          <span class="nav-label">{{ t('nav.reports') }}</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <!-- Footer dropdowns open upward: room below the footer block is clipped by sidebar edge. -->
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <!-- Main column -->
    <main class="main">
      <FilterBar />
      <div class="main-content">
        <router-view />
      </div>
    </main>

    <!-- Modals (props/events preserved from original App.vue) -->
    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import NavIcon from './components/NavIcon.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher,
    NavIcon
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
/* Global resets — tokens.css already sets body font, color, bg */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* ── App shell ─────────────────────────────────────────────────────────────── */

.app {
  display: flex;
  min-height: 100vh;
  background: var(--bg);
}

/* ── Sidebar ───────────────────────────────────────────────────────────────── */

.sidebar {
  width: var(--sidebar-w);
  flex: 0 0 var(--sidebar-w);
  background: var(--surface);
  border-right: 1px solid var(--line);
  display: flex;
  flex-direction: column;
  position: sticky;
  top: 0;
  height: 100vh;
}

.sidebar-logo {
  padding: var(--s-6) var(--s-5);
  border-bottom: 1px solid var(--line);
  display: flex;
  align-items: center;
  gap: var(--s-3);
  color: var(--accent);
}

.logo-text {
  line-height: 1.2;
}

.logo-name {
  font-size: var(--fs-md);
  font-weight: 700;
  letter-spacing: -0.01em;
  color: var(--ink);
}

.logo-sub {
  font-family: var(--font-mono);
  font-size: var(--fs-eyebrow);
  color: var(--muted);
  margin-top: 2px;
}

.sidebar-nav {
  flex: 1;
  padding: var(--s-4) var(--s-3);
  display: flex;
  flex-direction: column;
  gap: var(--s-1);
  overflow-y: auto;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--s-3);
  padding: var(--s-3) var(--s-4);
  border-radius: var(--r-sm);
  font-size: var(--fs-body);
  font-weight: 500;
  color: var(--ink-2);
  text-decoration: none;
  position: relative;
  transition: background-color 0.15s ease, color 0.15s ease;
}

.sidebar-nav a:hover {
  background: var(--surface-2);
  color: var(--ink);
}

.sidebar-nav a.active {
  background: var(--accent-soft);
  color: var(--accent);
}

/* Left 3px accent bar — stronger directional cue than background tint alone. */
.sidebar-nav a.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 8px;
  bottom: 8px;
  width: 3px;
  background: var(--accent);
  border-radius: 0 var(--r-sm) var(--r-sm) 0;
}

.sidebar-footer {
  padding: var(--s-4) var(--s-3);
  border-top: 1px solid var(--line);
  display: flex;
  flex-direction: column;
  gap: var(--s-2);
}

/* Footer dropdowns open upward — viewport below sidebar footer is clipped. */
.sidebar-footer :deep(.dropdown-menu),
.sidebar-footer :deep(.menu) {
  bottom: 100%;
  top: auto;
  margin-bottom: var(--s-2);
}

/* ── Main column ───────────────────────────────────────────────────────────── */

.main {
  flex: 1;
  min-width: 0; /* prevents wide tables from pushing sidebar wider */
  display: flex;
  flex-direction: column;
}

/* FilterBar sticky to top of main scroll region, not viewport top */
.main > .filters-bar {
  position: sticky;
  top: 0;
  z-index: 30;
  background: var(--bg);
}

.main-content {
  width: 100%;
  max-width: var(--content-max);
  margin: 0 auto;
  padding: var(--s-8) var(--s-10);
}

/* ── Page header ───────────────────────────────────────────────────────────── */

.page-header {
  margin-bottom: var(--s-8);
}

.page-header h2 {
  font-size: var(--fs-h1);
  font-weight: 700;
  letter-spacing: -0.02em;
  color: var(--ink);
  margin: 0 0 var(--s-2);
}

.page-header p {
  font-size: var(--fs-body);
  color: var(--muted);
  margin: 0;
}

/* ── Cards ─────────────────────────────────────────────────────────────────── */

.card {
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: var(--r-md);
  padding: var(--s-5);
  margin-bottom: var(--s-5);
  /* No shadow on default cards — hairline rule is sufficient. */
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--s-4);
  padding-bottom: var(--s-3);
  border-bottom: 1px solid var(--line);
}

.card-title {
  font-size: var(--fs-lg);
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--ink);
}

/* ── Stat grid ─────────────────────────────────────────────────────────────── */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--s-5);
  margin-bottom: var(--s-8);
}

.stat-card {
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: var(--r-md);
  padding: var(--s-5);
}

.stat-label {
  font-family: var(--font-mono);
  font-size: var(--fs-eyebrow);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--muted);
  margin-bottom: var(--s-2);
}

.stat-value {
  font-size: var(--fs-h2);
  font-weight: 700;
  letter-spacing: -0.02em;
  color: var(--ink);
  font-variant-numeric: tabular-nums;
}

.stat-card.warning .stat-value { color: var(--warn); }
.stat-card.success .stat-value { color: var(--success); }
.stat-card.danger  .stat-value { color: var(--danger); }
.stat-card.info    .stat-value { color: var(--info); }

/* ── Tables ────────────────────────────────────────────────────────────────── */

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--surface-2);
  border-top: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
}

th {
  text-align: left;
  padding: var(--s-3) var(--s-4);
  font-family: var(--font-mono);
  font-size: var(--fs-eyebrow);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--muted);
  background: var(--surface-2);
}

td {
  padding: var(--s-3) var(--s-4);
  font-size: var(--fs-small);
  color: var(--ink-2);
  border-bottom: 1px solid var(--line);
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--surface-2);
}

/* ── Badges ────────────────────────────────────────────────────────────────── */

.badge {
  display: inline-block;
  padding: var(--s-1) var(--s-3);
  border-radius: 999px;
  font-size: var(--fs-caption);
  font-weight: 600;
  /* Not uppercase — soft modernist aesthetic is restrained, badge text reads as-is. */
}

.badge.success    { background: var(--success-soft); color: var(--success); }
.badge.warning    { background: var(--warn-soft);    color: var(--warn); }
.badge.danger     { background: var(--danger-soft);  color: var(--danger); }
.badge.info       { background: var(--info-soft);    color: var(--info); }

/* Trend badges */
.badge.increasing { background: var(--success-soft); color: var(--success); }
.badge.decreasing { background: var(--danger-soft);  color: var(--danger); }
.badge.stable     { background: var(--surface-2);    color: var(--muted); }

/* Priority badges — mapped to severity tokens */
.badge.high       { background: var(--danger-soft);  color: var(--danger); }
.badge.medium     { background: var(--warn-soft);    color: var(--warn); }
.badge.low        { background: var(--info-soft);    color: var(--info); }

/* ── Loading & error states ────────────────────────────────────────────────── */

.loading {
  text-align: center;
  padding: var(--s-12);
  color: var(--muted);
  font-size: var(--fs-body);
}

.error {
  background: var(--danger-soft);
  border: 1px solid var(--danger);
  color: var(--danger);
  padding: var(--s-4);
  border-radius: var(--r-md);
  margin: var(--s-4) 0;
  font-size: var(--fs-body);
}

/* ── Responsive: collapse sidebar to icon-only below 1100px ────────────────── */

/* 1100px chosen: sidebar 232 + main padding 80 + readable content 720 ≈ 1032px minimum.
   1100px gives breathing room before collapse. */
@media (max-width: 1100px) {
  .sidebar {
    width: var(--sidebar-w-compact);
    flex-basis: var(--sidebar-w-compact);
  }

  .sidebar .nav-label,
  .sidebar-logo .logo-text {
    display: none;
  }

  .sidebar-nav a {
    justify-content: center;
    padding: var(--s-3);
  }

  .main-content {
    padding: var(--s-6) var(--s-5);
  }
}
</style>
