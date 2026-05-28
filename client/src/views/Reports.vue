<template>
  <div class="reports">
    <div class="page-header">
      <h2>{{ t('reports.title') }}</h2>
      <p>{{ t('reports.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('reports.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>

    <!-- Empty state when filters exclude all data -->
    <div v-else-if="isEmpty" class="card no-data">
      <p>{{ t('reports.noData') }}</p>
    </div>

    <template v-else>
      <!-- Summary Stats -->
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-label">{{ t('reports.summary.totalRevenueYTD') }}</div>
          <div class="stat-value">{{ formatCurrency(totalRevenue) }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">{{ t('reports.summary.avgMonthlyRevenue') }}</div>
          <div class="stat-value">{{ formatCurrency(avgMonthlyRevenue) }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">{{ t('reports.summary.totalOrdersYTD') }}</div>
          <div class="stat-value">{{ totalOrders }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">{{ t('reports.summary.bestQuarter') }}</div>
          <div class="stat-value">{{ bestQuarter || '—' }}</div>
        </div>
      </div>

      <!-- Quarterly Performance -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('reports.quarterly.title') }}</h3>
        </div>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('reports.quarterly.quarter') }}</th>
                <th>{{ t('reports.quarterly.totalOrders') }}</th>
                <th>{{ t('reports.quarterly.totalRevenue') }}</th>
                <th>{{ t('reports.quarterly.avgOrderValue') }}</th>
                <th>{{ t('reports.quarterly.fulfillmentRate') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="q in quarterlyData" :key="q.quarter">
                <td><strong>{{ q.quarter }}</strong></td>
                <td>{{ q.total_orders }}</td>
                <td>{{ formatCurrency(q.total_revenue) }}</td>
                <td>{{ formatCurrency(q.avg_order_value) }}</td>
                <td>
                  <span :class="getFulfillmentClass(q.fulfillment_rate)">
                    {{ q.fulfillment_rate }}%
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Monthly Revenue Trend Chart -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('reports.monthlyTrend.title') }}</h3>
        </div>
        <div class="chart-container">
          <div class="bar-chart" role="img" :aria-label="t('reports.monthlyTrend.title')">
            <div
              v-for="month in monthlyData"
              :key="month.month"
              class="bar-wrapper"
            >
              <div class="bar-area">
                <div
                  class="bar"
                  :style="{ height: getBarHeightPct(month.revenue) + '%' }"
                  :title="formatCurrency(month.revenue)"
                ></div>
              </div>
              <!-- Horizontal label below bar — no rotate to avoid clipping -->
              <div class="bar-label">{{ formatMonthLabel(month.month) }}</div>
            </div>
          </div>
        </div>
      </div>

      <!-- Month-over-Month Analysis -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('reports.monthOverMonth.title') }}</h3>
        </div>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('reports.monthOverMonth.month') }}</th>
                <th>{{ t('reports.monthOverMonth.orders') }}</th>
                <th>{{ t('reports.monthOverMonth.revenue') }}</th>
                <th>{{ t('reports.monthOverMonth.change') }}</th>
                <th>{{ t('reports.monthOverMonth.growthRate') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(month, index) in monthlyData" :key="month.month">
                <td><strong>{{ formatMonthLabel(month.month) }}</strong></td>
                <td>{{ month.order_count }}</td>
                <td>{{ formatCurrency(month.revenue) }}</td>
                <td>
                  <span
                    v-if="index > 0"
                    :class="getChangeClass(month.revenue, monthlyData[index - 1].revenue)"
                  >
                    {{ formatChangeValue(month.revenue, monthlyData[index - 1].revenue) }}
                  </span>
                  <span v-else class="muted-dash">—</span>
                </td>
                <td>
                  <span
                    v-if="index > 0"
                    :class="getChangeClass(month.revenue, monthlyData[index - 1].revenue)"
                  >
                    {{ getGrowthRate(month.revenue, monthlyData[index - 1].revenue) }}
                  </span>
                  <span v-else class="muted-dash">—</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Reports',
  setup() {
    const { t, currentLocale, currentCurrency } = useI18n()
    const { selectedPeriod, selectedLocation, selectedCategory, selectedStatus, getCurrentFilters } = useFilters()

    const loading = ref(false)
    const error = ref(null)
    const quarterlyData = ref([])
    const monthlyData = ref([])

    // ── Derived / summary ────────────────────────────────────────────────────

    const isEmpty = computed(
      () => quarterlyData.value.length === 0 && monthlyData.value.length === 0
    )

    const totalRevenue = computed(() =>
      monthlyData.value.reduce((sum, m) => sum + m.revenue, 0)
    )

    const avgMonthlyRevenue = computed(() =>
      monthlyData.value.length > 0 ? totalRevenue.value / monthlyData.value.length : 0
    )

    const totalOrders = computed(() =>
      monthlyData.value.reduce((sum, m) => sum + m.order_count, 0)
    )

    const bestQuarter = computed(() => {
      if (quarterlyData.value.length === 0) return ''
      return quarterlyData.value.reduce(
        (best, q) => (q.total_revenue > best.total_revenue ? q : best),
        quarterlyData.value[0]
      ).quarter
    })

    // ── Chart helpers ────────────────────────────────────────────────────────

    const maxRevenue = computed(() =>
      Math.max(...monthlyData.value.map((m) => m.revenue), 1)
    )

    // Returns percentage height (0–100) so CSS can scale the bar container freely
    const getBarHeightPct = (revenue) => (revenue / maxRevenue.value) * 100

    // ── Formatting ───────────────────────────────────────────────────────────

    const currencyFormatter = computed(
      () =>
        new Intl.NumberFormat(
          currentLocale.value === 'ja' ? 'ja-JP' : 'en-US',
          { style: 'currency', currency: currentCurrency.value, maximumFractionDigits: 2 }
        )
    )

    const formatCurrency = (value) => currencyFormatter.value.format(value ?? 0)

    // Month abbreviation keys indexed 0–11, matching JS Date.getMonth()
    const MONTH_KEYS = ['jan', 'feb', 'mar', 'apr', 'may', 'jun', 'jul', 'aug', 'sep', 'oct', 'nov', 'dec']

    // Converts "YYYY-MM" → translated abbrev + year, e.g. "Jan 2024" / "1月 2024"
    const formatMonthLabel = (monthStr) => {
      const parts = monthStr.split('-')
      const year = parts[0]
      const monthIndex = parseInt(parts[1], 10) - 1
      if (isNaN(monthIndex) || monthIndex < 0 || monthIndex > 11) return monthStr
      return `${t(`months.${MONTH_KEYS[monthIndex]}`)} ${year}`
    }

    // ── Badge / class helpers ────────────────────────────────────────────────

    const getFulfillmentClass = (rate) => {
      if (rate >= 90) return 'badge success'
      if (rate >= 75) return 'badge warning'
      return 'badge danger'
    }

    const getChangeClass = (current, previous) => {
      const delta = current - previous
      if (delta > 0) return 'change positive'
      if (delta < 0) return 'change negative'
      return 'change'
    }

    const formatChangeValue = (current, previous) => {
      const delta = current - previous
      const abs = formatCurrency(Math.abs(delta))
      if (delta > 0) return `+${abs}`
      if (delta < 0) return `-${abs}`
      return abs
    }

    // Returns t('reports.monthOverMonth.notAvailable') when previous === 0
    const getGrowthRate = (current, previous) => {
      if (previous === 0) return t('reports.monthOverMonth.notAvailable')
      const rate = ((current - previous) / previous) * 100
      const sign = rate > 0 ? '+' : ''
      return `${sign}${rate.toFixed(1)}%`
    }

    // ── Data loading ─────────────────────────────────────────────────────────

    const loadData = async () => {
      loading.value = true
      error.value = null
      try {
        const filters = getCurrentFilters()
        const [quarterly, monthly] = await Promise.all([
          api.getQuarterlyReports(filters),
          api.getMonthlyTrends(filters)
        ])
        quarterlyData.value = quarterly
        monthlyData.value = monthly
      } catch (err) {
        console.error('Failed to load reports:', err)
        error.value = 'Failed to load reports: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Watch all four filter refs so any change triggers a re-fetch
    watch(
      [selectedPeriod, selectedLocation, selectedCategory, selectedStatus],
      loadData
    )

    onMounted(loadData)

    return {
      t,
      loading,
      error,
      quarterlyData,
      monthlyData,
      isEmpty,
      totalRevenue,
      avgMonthlyRevenue,
      totalOrders,
      bestQuarter,
      formatCurrency,
      formatMonthLabel,
      getBarHeightPct,
      getFulfillmentClass,
      getChangeClass,
      formatChangeValue,
      getGrowthRate
    }
  }
}
</script>

<style scoped>
.reports {
  padding: 0;
}

/* Empty-state message centered in card */
.no-data {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 160px;
  color: var(--muted);
  font-size: var(--fs-body);
}

/* ── Bar chart ──────────────────────────────────────────────────────────────── */

.chart-container {
  padding: var(--s-4) 0 var(--s-2);
}

.bar-chart {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  height: 220px;
  gap: var(--s-2);
}

.bar-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
}

/* Allocates the 200px drawing area for the bar itself */
.bar-area {
  width: 100%;
  height: 200px;
  display: flex;
  align-items: flex-end;
}

.bar {
  width: 100%;
  background: var(--accent);
  border-radius: var(--r-sm) var(--r-sm) 0 0;
  transition: height 0.3s ease, opacity 0.15s ease;
  cursor: default;
  min-height: 2px;
}

.bar:hover {
  opacity: 0.8;
}

/* Plain horizontal label — no rotate, no clipping */
.bar-label {
  margin-top: var(--s-2);
  font-family: var(--font-mono);
  font-size: var(--fs-caption);
  color: var(--muted);
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
}

/* ── Month-over-month change cells ──────────────────────────────────────────── */

.change {
  font-weight: 600;
  font-size: var(--fs-small);
}

.change.positive {
  color: var(--success);
}

.change.negative {
  color: var(--danger);
}

.muted-dash {
  color: var(--muted-2);
}
</style>
