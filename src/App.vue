<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import * as echarts from 'echarts'

export interface QuickLink {
  label: string
  icon: string
  color: string
  badge?: number
  details?: QuickLinkDetail[]
}

export interface QuickLinkDetail {
  type: string
  count: number
}

export interface Notice {
  title: string
  date: string
}

export interface CityHSRData {
  city: string
  opened: number
  unopened: number
}

export interface TaskTypeData {
  type: string
  value: number
}

const quickLinks: QuickLink[] = [
  { label: '办公平台', icon: 'fa-desktop', color: 'linear-gradient(135deg, #1890ff, #096dd9)', badge: 23, details: [
    { type: '任务分配', count: 14 },
    { type: '个人审批', count: 5 },
    { type: '邮件查看', count: 4 }
  ]},
  { label: 'EHR平台', icon: 'fa-users', color: 'linear-gradient(135deg, #722ed1, #531dab)' },
  { label: '网上教育', icon: 'fa-graduation-cap', color: 'linear-gradient(135deg, #52c41a, #389e0d)' },
  { label: '财务系统', icon: 'fa-yen-sign', color: 'linear-gradient(135deg, #9254de, #722ed1)' },
  { label: '代办阅读', icon: 'fa-book-open', color: 'linear-gradient(135deg, #faad14, #d48806)' },
  { label: '通知公告', icon: 'fa-bullhorn', color: 'linear-gradient(135deg, #13c2c2, #08979c)' },
  { label: '员工通道', icon: 'fa-id-badge', color: 'linear-gradient(135deg, #f5222d, #cf1322)' },
  { label: '快捷通道', icon: 'fa-th', color: 'linear-gradient(135deg, #2f54eb, #1d39c4)' },
]

// 办公平台详细信息弹窗状态
const showOfficeDetail = ref(false)

const notices: Notice[] = [
  { title: '关于2020年端午节放假安排的通知', date: '05.09' },
  { title: '关于开展安全生产月活动的通知', date: '05.08' },
  { title: '关于调整夏季作息时间的通知', date: '05.07' },
  { title: '关于组织员工体检的通知', date: '05.06' },
  { title: '关于举办企业文化节的通知', date: '05.05' },
]

// 任务统计数据
// 高铁线路通车数据（江苏省各城市）
const cityHSRData: CityHSRData[] = [
  { city: '南京', opened: 12, unopened: 3 },
  { city: '苏州', opened: 15, unopened: 1 },
  { city: '无锡', opened: 10, unopened: 2 },
  { city: '常州', opened: 8, unopened: 2 },
  { city: '徐州', opened: 9, unopened: 2 },
  { city: '盐城', opened: 4, unopened: 5 },
]

const totalOpened = cityHSRData.reduce((sum, d) => sum + d.opened, 0)
const totalUnopened = cityHSRData.reduce((sum, d) => sum + d.unopened, 0)

// 当前月任务类型数据（雷达图）
const currentMonthTaskTypes: TaskTypeData[] = [
  { type: '审批任务', value: 35 },
  { type: '会议安排', value: 25 },
  { type: '项目跟进', value: 30 },
  { type: '文档处理', value: 20 },
  { type: '客户沟通', value: 28 },
  { type: '团队协作', value: 22 }
]

const isTripTooltipVisible = ref(false)

// 日历相关数据
const currentMonth = ref(new Date())
const selectedDate = ref(new Date())
const todoTasks = ref([
  { date: '2026-07-03', title: '项目进度汇报', completed: false },
  { date: '2026-07-03', title: '客户会议准备', completed: true },
  { date: '2026-07-04', title: '设计评审会', completed: false },
  { date: '2026-07-05', title: '月度总结报告', completed: false },
  { date: '2026-07-10', title: '团建活动', completed: false }
])

// ECharts 图表引用
const barChartRef = ref<HTMLElement | null>(null)
const radarChartRef = ref<HTMLElement | null>(null)

// 初始化图表
onMounted(() => {
  initHSRBarChart()
  initRadarChart()
})

const initHSRBarChart = () => {
  if (barChartRef.value) {
    const chart = echarts.init(barChartRef.value)
    const option = {
      tooltip: {
        trigger: 'axis',
        backgroundColor: 'rgba(255, 255, 255, 0.95)',
        borderColor: '#e8e8e8',
        textStyle: {
          color: '#262626'
        },
        formatter: '{b}<br/>已通车: {c0} 段<br/>未通车: {c1} 段'
      },
      legend: {
        data: ['已通车', '未通车'],
        bottom: 0,
        textStyle: {
          color: '#595959'
        }
      },
      grid: {
        left: '3%',
        right: '4%',
        bottom: '15%',
        containLabel: true
      },
      xAxis: {
        type: 'category',
        data: cityHSRData.map(item => item.city),
        axisLine: {
          lineStyle: { color: '#e8e8e8' }
        },
        axisLabel: {
          color: '#8c8c8c',
          fontSize: 12,
          interval: 0
        }
      },
      yAxis: {
        type: 'value',
        axisLine: {
          lineStyle: { color: '#e8e8e8' }
        },
        axisLabel: {
          color: '#8c8c8c'
        },
        splitLine: {
          lineStyle: { color: '#f0f0f0' }
        }
      },
      series: [
        {
          name: '已通车',
          type: 'bar',
          barWidth: '30%',
          barGap: '10%',
          data: cityHSRData.map(item => item.opened),
          itemStyle: {
            color: {
              type: 'linear',
              x: 0, y: 0, x2: 0, y2: 1,
              colorStops: [
                { offset: 0, color: '#52c41a' },
                { offset: 1, color: '#389e0d' }
              ]
            },
            borderRadius: [4, 4, 0, 0]
          }
        },
        {
          name: '未通车',
          type: 'bar',
          barWidth: '30%',
          data: cityHSRData.map(item => item.unopened),
          itemStyle: {
            color: {
              type: 'linear',
              x: 0, y: 0, x2: 0, y2: 1,
              colorStops: [
                { offset: 0, color: '#ff7a45' },
                { offset: 1, color: '#d4380d' }
              ]
            },
            borderRadius: [4, 4, 0, 0]
          }
        }
      ]
    }
    chart.setOption(option)
    
    window.addEventListener('resize', () => {
      chart.resize()
    })
  }
}

const initRadarChart = () => {
  if (radarChartRef.value) {
    const chart = echarts.init(radarChartRef.value)
    const option = {
      tooltip: {
        backgroundColor: 'rgba(255, 255, 255, 0.95)',
        borderColor: '#e8e8e8',
        textStyle: {
          color: '#262626'
        },
        formatter: '{b}: {c}个'
      },
      radar: {
        indicator: currentMonthTaskTypes.map(item => ({
          name: item.type,
          max: 40
        })),
        splitArea: {
          areaStyle: {
            color: ['rgba(24, 144, 255, 0.02)', 'rgba(24, 144, 255, 0.04)', 
                    'rgba(24, 144, 255, 0.06)', 'rgba(24, 144, 255, 0.08)']
          }
        },
        axisLine: {
          lineStyle: {
            color: '#e8e8e8'
          }
        },
        splitLine: {
          lineStyle: {
            color: '#e8e8e8'
          }
        },
        name: {
          textStyle: {
            color: '#262626',
            fontSize: 12
          }
        }
      },
      series: [{
        name: '任务类型',
        type: 'radar',
        data: [{
          value: currentMonthTaskTypes.map(item => item.value),
          name: '任务数量',
          areaStyle: {
            color: 'rgba(82, 196, 26, 0.3)'
          },
          lineStyle: {
            color: '#52c41a',
            width: 2
          },
          itemStyle: {
            color: '#52c41a'
          }
        }]
      }]
    }
    chart.setOption(option)

    window.addEventListener('resize', () => {
      chart.resize()
    })
  }
}

// 日历计算函数
const getMonthName = (date: Date) => {
  return date.toLocaleDateString('zh-CN', { year: 'numeric', month: 'long' })
}

const isSameDay = (date1: Date, date2: Date) => {
  return date1.getFullYear() === date2.getFullYear() &&
    date1.getMonth() === date2.getMonth() &&
    date1.getDate() === date2.getDate()
}

const formatDate = (date: Date) => {
  return date.toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    weekday: 'short'
  })
}

const previousMonth = () => {
  currentMonth.value = new Date(currentMonth.value.getFullYear(), currentMonth.value.getMonth() - 1, 1)
}

const nextMonth = () => {
  currentMonth.value = new Date(currentMonth.value.getFullYear(), currentMonth.value.getMonth() + 1, 1)
}

const goToToday = () => {
  const today = new Date()
  currentMonth.value = today
  selectedDate.value = today
}

const selectDate = (date: Date) => {
  selectedDate.value = date
}

const hasTask = (date: Date) => {
  const dateString = date.toISOString().split('T')[0]
  return todoTasks.value.some(task => task.date === dateString)
}

const getTasksForDate = (date: Date) => {
  const dateString = date.toISOString().split('T')[0]
  return todoTasks.value.filter(task => task.date === dateString)
}

const toggleTask = (task: any) => {
  task.completed = !task.completed
}

const calendarDays = computed(() => {
  const year = currentMonth.value.getFullYear()
  const month = currentMonth.value.getMonth()

  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const startDay = firstDay.getDay()

  const days = []

  // 上个月的日期
  const prevMonthLastDay = new Date(year, month, 0).getDate()
  for (let i = startDay - 1; i >= 0; i--) {
    const date = new Date(year, month - 1, prevMonthLastDay - i)
    days.push({ date, day: date.getDate(), currentMonth: false })
  }

  // 当前月的日期
  for (let i = 1; i <= lastDay.getDate(); i++) {
    const date = new Date(year, month, i)
    days.push({ date, day: i, currentMonth: true })
  }

  // 下个月的日期
  const remainingDays = 42 - days.length
  for (let i = 1; i <= remainingDays; i++) {
    const date = new Date(year, month + 1, i)
    days.push({ date, day: i, currentMonth: false })
  }

  return days
})
</script>

<style scoped lang="less">
:global(*) {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:global(:root) {
  --primary: #1890ff;
  --primary-dark: #096dd9;
  --sidebar-bg: #0050b3;
  --sidebar-dark: #003a8c;
  --bg: #f0f2f5;
  --card-bg: #ffffff;
  --text-primary: #262626;
  --text-secondary: #8c8c8c;
  --text-light: rgba(255, 255, 255, 0.85);
  --border: #e8e8e8;
  --shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  --radius: 8px;
  --sidebar-width: 72px;
}

:global(body) {
  min-width: 320px;
  min-height: 100vh;
  overflow-x: hidden;
  color: var(--text-primary);
  background: var(--bg);
  font-family: "PingFang SC", "Microsoft YaHei", "Helvetica Neue", Arial, sans-serif;
}

a {
  color: inherit;
  text-decoration: none;
}

button,
input,
select {
  font: inherit;
}

.app {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  position: fixed;
  z-index: 100;
  top: 0;
  bottom: 0;
  left: 0;
  display: flex;
  width: var(--sidebar-width);
  padding: 20px 0;
  flex-direction: column;
  align-items: center;
  background: linear-gradient(180deg, var(--sidebar-bg) 0%, var(--sidebar-dark) 100%);

  &-top {
    display: flex;
    flex: 1;
    flex-direction: column;
    align-items: center;
  }

  &-menu {
    display: flex;
    width: 100%;
    padding: 0 8px;
    flex-direction: column;
    gap: 12px;
  }

  &-item {
    display: flex;
    padding: 6px 4px;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    border-radius: 6px;
    color: var(--text-light);
    font-size: 11px;
    transition: background 0.2s;

    &:hover {
      background: rgba(255, 255, 255, 0.1);
    }
  }

  &-nav {
    display: flex;
    padding-bottom: 8px;
    flex-direction: column;
    gap: 4px;
  }
}

.avatar {
  width: 44px;
  height: 44px;
  margin-bottom: 8px;
  overflow: hidden;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}

.user-name {
  margin-bottom: 16px;
  color: #fff;
  font-size: 12px;
  font-weight: 500;
}

.badge {
  display: inline-flex;
  min-width: 18px;
  height: 18px;
  padding: 0 5px;
  align-items: center;
  justify-content: center;
  border-radius: 9px;
  color: #fff;
  font-size: 11px;
  font-weight: 600;

  &-orange {
    background: #fa8c16;
  }

  &-blue {
    background: #40a9ff;
  }

  &-white {
    min-width: 16px;
    height: 16px;
    background: rgba(255, 255, 255, 0.25);
    font-size: 10px;
  }
}

.nav-icon {
  display: flex;
  width: 44px;
  height: 44px;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.7);
  font-size: 18px;
  transition: all 0.2s;

  &:hover {
    color: #fff;
    background: rgba(255, 255, 255, 0.1);
  }

  &.active {
    color: var(--primary);
    background: #fff;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  }
}

.main {
  min-width: 0;
  margin-left: var(--sidebar-width);
  padding: 0 24px 24px;
  flex: 1;
}

.header {
  display: flex;
  padding: 16px 0;
  align-items: center;
  justify-content: space-between;
  gap: 24px;

  &-search {
    display: flex;
    min-width: 320px;
    overflow: hidden;
    align-items: center;
    border-radius: 4px;
    background: #fff;
    box-shadow: var(--shadow);

    input {
      flex: 1;
      padding: 10px 16px;
      border: 0;
      outline: 0;
      color: var(--text-primary);
      font-size: 14px;

      &::placeholder {
        color: #bfbfbf;
      }
    }
  }
}

.logo {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo-icon {
  display: flex;
  width: 48px;
  height: 48px;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  color: #fff;
  background: linear-gradient(135deg, #1890ff, #0050b3);
  font-size: 11px;
  font-weight: 700;
}

.logo-text h1 {
  color: var(--text-primary);
  font-size: 18px;
  font-weight: 600;
  line-height: 1.3;
}

.logo-text p {
  margin-top: 2px;
  color: var(--text-secondary);
  font-size: 10px;
  letter-spacing: 0.3px;
}

.btn-search {
  padding: 10px 24px;
  border: 0;
  color: #fff;
  background: var(--primary);
  cursor: pointer;
  font-size: 14px;
  transition: background 0.2s;

  &:hover {
    background: var(--primary-dark);
  }
}

.quick-links {
  display: flex;
  margin-bottom: 16px;
  padding: 20px 28px;
  flex-wrap: wrap;
  gap: 20px 32px;
  border-radius: var(--radius);
  background: var(--card-bg);
  box-shadow: var(--shadow);
}

.quick-item {
  position: relative;
  display: flex;
  width: 72px;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  transition: transform 0.2s;

  &:hover {
    transform: translateY(-2px);
    z-index: 100;
  }

  span {
    color: var(--text-primary);
    font-size: 12px;
    text-align: center;
    white-space: nowrap;
  }
}

.quick-icon {
  display: flex;
  width: 52px;
  height: 52px;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  color: #fff;
  font-size: 20px;
  flex-shrink: 0;

  &.dashed {
    border: 2px dashed #d9d9d9;
    color: #bfbfbf;
    background: #fafafa;
  }
}

.add-module {
  opacity: 0.65;
  transition: opacity 0.2s;

  &:hover {
    opacity: 1;
  }
}

// 快捷入口徽章样式
.quick-badge {
  position: absolute;
  top: -6px;
  right: -6px;
  display: flex;
  min-width: 20px;
  height: 20px;
  padding: 0 6px;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  color: #fff;
  font-size: 11px;
  font-weight: 600;
  background: #ff4d4f;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  z-index: 1;

  &:hover {
    transform: scale(1.1);
    box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3);
  }
}

// 办公平台详细信息弹窗样式
.office-detail-popup {
  position: absolute;
  z-index: 10000;
  top: 50%;
  left: 100%;
  transform: translateY(-50%);
  margin-left: 12px;
  min-width: 200px;
  padding: 12px;
  border-radius: 8px;
  background: #fff;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);

  &::after {
    position: absolute;
    top: 50%;
    right: 100%;
    transform: translateY(-50%);
    border: 6px solid transparent;
    border-right-color: #fff;
    content: '';
  }

  &-title {
    padding-bottom: 8px;
    margin-bottom: 8px;
    color: var(--text-primary);
    font-size: 14px;
    font-weight: 600;
    border-bottom: 1px solid var(--border);
  }

  &-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 6px 0;

    &-type {
      color: var(--text-secondary);
      font-size: 12px;
    }

    &-count {
      color: var(--primary);
      font-size: 13px;
      font-weight: 600;
    }
  }
}

.approval-icons-section {
  display: flex;
  gap: 16px;
}

.personal-info-content {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.info-greeting {
  color: rgba(255, 255, 255, 0.9);
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 2px;
}

.info-name-phone {
  color: #fff;
  font-size: 18px;
  font-weight: 700;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
}

.info-slogan {
  color: rgba(255, 255, 255, 0.75);
  font-size: 13px;
  font-style: italic;
}

.info-position {
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  font-weight: 500;
}

.action-text {
  font-size: 12px;
}

@keyframes approvalScroll {
  0% {
    transform: translateY(0);
  }

  100% {
    transform: translateY(-50%);
  }
}

.action-item {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  color: #fff;
  font-size: 12px;
  transition: transform 0.2s;

  &:hover {
    transform: scale(1.05);
  }

  .badge {
    position: absolute;
    top: -4px;
    right: -4px;
    min-width: 18px;
    height: 18px;
    padding: 0 5px;
    border-radius: 9px;
    color: #fff;
    font-size: 11px;
    font-weight: 600;
    background: #ff4d4f;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  }

  &-highlight {
    .action-icon {
      border: 2px solid rgba(255, 255, 255, 0.85);
      background: rgba(255, 255, 255, 0.45);
      box-shadow: 0 0 14px rgba(255, 255, 255, 0.45);
    }

    >span:not(.action-tooltip) {
      font-weight: 600;
      text-shadow: 0 0 8px rgba(255, 255, 255, 0.35);
    }

    &:hover .action-tooltip,
    &:focus-visible .action-tooltip,
    &.is-tooltip-visible .action-tooltip {
      opacity: 1;
      transform: translateX(-50%) translateY(0);
      visibility: visible;
    }
  }
}

.action-icon {
  display: flex;
  width: 40px;
  height: 40px;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  font-size: 16px;
}

.action-tooltip {
  position: absolute;
  z-index: 2;
  bottom: calc(100% + 8px);
  left: 50%;
  padding: 6px 10px;
  border-radius: 4px;
  opacity: 0;
  color: #fff;
  background: rgba(0, 0, 0, 0.75);
  font-size: 11px;
  pointer-events: none;
  transform: translateX(-50%) translateY(4px);
  transition: opacity 0.2s, transform 0.2s, visibility 0.2s;
  visibility: hidden;
  white-space: nowrap;

  &::after {
    position: absolute;
    top: 100%;
    left: 50%;
    border: 5px solid transparent;
    border-top-color: rgba(0, 0, 0, 0.75);
    content: '';
    transform: translateX(-50%);
  }
}

// 状态卡片区域样式
.status-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 16px;
}

.status-card {
  border-radius: var(--radius);
  overflow: hidden;
  box-shadow: var(--shadow);
}

.approval-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
}

.approval-content {
  display: flex;
  padding: 20px 24px;
  gap: 32px;
  flex: 1;
}

.personal-info-section {
  flex: 1;
}

.approval-actions {
  display: flex;
  justify-content: space-around;
  padding: 12px 24px;
  gap: 24px;
  align-items: center;
  background: rgba(0, 0, 0, 0.1);
}

.todo-card {
  display: flex;
  padding: 20px 24px;
  flex-direction: column;
  gap: 16px;
  background: linear-gradient(135deg, #1890ff 0%, #096dd9 100%);
}

.todo-section {
  color: #fff;
}

.todo-header {
  display: flex;
  margin-bottom: 8px;
  align-items: baseline;
  justify-content: space-between;
}

.todo-label {
  opacity: 0.9;
  font-size: 14px;
}

.todo-count {
  font-size: 28px;
  font-weight: 700;

  small {
    opacity: 0.8;
    font-size: 14px;
    font-weight: 400;
  }
}

.progress {
  &-wrap {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  &-bar {
    height: 6px;
    flex: 1;
    overflow: hidden;
    border-radius: 3px;
    background: rgba(255, 255, 255, 0.3);
  }

  &-fill {
    height: 100%;
    border-radius: 3px;
    background: #fff;

    &-45 {
      width: 45%;
    }

    &-80 {
      width: 80%;
    }
  }

  &-text {
    opacity: 0.85;
    font-size: 12px;
    white-space: nowrap;
  }
}

.notice-card,
.weather-widget {
  background: var(--card-bg);
}

.notice-card {
  padding: 16px 20px;
}

.card-title {
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border);
  color: var(--text-primary);
  font-size: 15px;
  font-weight: 600;
}

.notice-list {
  list-style: none;

  li {
    display: flex;
    padding: 7px 0;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px dashed #f0f0f0;

    &:last-child {
      border-bottom: 0;
    }
  }

  a {
    flex: 1;
    margin-right: 12px;
    overflow: hidden;
    color: var(--text-primary);
    font-size: 13px;
    text-overflow: ellipsis;
    transition: color 0.2s;
    white-space: nowrap;

    &:hover {
      color: var(--primary);
    }
  }

  .date {
    flex-shrink: 0;
    color: var(--text-secondary);
    font-size: 12px;
  }
}

// 任务统计样式
.task-stats-section {
  display: grid;
  align-items: start;
  grid-template-columns: 1fr 280px;
  gap: 16px;
  margin-bottom: 16px;
}

.task-stats-main {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.task-stats-card {
  display: flex;
  flex-direction: column;
  padding: 20px 24px;
  border-radius: var(--radius);
  background: #fff;
  box-shadow: var(--shadow);
  transition: transform 0.2s, box-shadow 0.2s;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.12);
  }
}

.task-stats-title {
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border);
  color: var(--text-primary);
  font-size: 24px;
  font-weight: 600;
}

.project-finance-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 20px;
}

.finance-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  border-radius: 8px;
}

.finance-icon {
  display: flex;
  width: 40px;
  height: 40px;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  font-size: 18px;
  flex-shrink: 0;
}

.finance-item:first-child .finance-icon {
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  color: #fff;
}

.finance-item:last-child .finance-icon {
  background: linear-gradient(135deg, #ffa502, #e67e22);
  color: #fff;
}

.finance-body {
  display: flex;
  flex-direction: column;
  gap: 4px;
  flex: 1;
}

.finance-label {
  color: var(--text-secondary);
  font-size: 13px;
  font-weight: 500;
}

.finance-value {
  color: #1890ff;
  font-size: 22px;
  font-weight: 700;
}

.project-stats-section {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 12px;
  padding-top: 16px;
  border-top: 1px solid var(--border);
}

.project-stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 12px 8px;
  border-radius: 8px;
  background: #fafafa;
  transition: background 0.2s;

  &:hover {
    background: #f0f5ff;
  }
}

.project-stat-label {
  color: var(--text-secondary);
  font-size: 12px;
  font-weight: 500;
}

.project-stat-count {
  color: var(--text-primary);
  font-size: 24px;
  font-weight: 700;
  line-height: 1;
}

// 图表区域样式
.charts-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.chart-card {
  padding: 20px;
  border-radius: var(--radius);
  background: #fff;
  box-shadow: var(--shadow);
}

.chart-container {
  width: 100%;
  height: 228px;
}

// 图表卡片标题
.chart-card-title {
  margin: 0 0 16px;
  padding-bottom: 10px;
  border-bottom: 2px solid #d9d9d9;
  color: var(--text-primary);
  font-size: 16px;
  font-weight: 600;
}

// 任务类型文字说明
.task-type-text {
  margin: 0 0 12px;
  color: var(--text-secondary);
  font-size: 13px;
  line-height: 1.6;
  text-align: center;
}

// 高铁线路通车统计
.hSR-summary {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 24px;
  margin-bottom: 20px;
  padding: 16px 24px;
  background: #fafafa;
  border-radius: 8px;
}

.hSR-summary-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.hSR-summary-icon {
  display: flex;
  width: 44px;
  height: 44px;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  font-size: 20px;
  flex-shrink: 0;
}

.hSR-summary-opened .hSR-summary-icon {
  background: linear-gradient(135deg, #52c41a, #389e0d);
  color: #fff;
}

.hSR-summary-unopened .hSR-summary-icon {
  background: linear-gradient(135deg, #ff7a45, #d4380d);
  color: #fff;
}

.hSR-summary-body {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.hSR-summary-label {
  color: var(--text-secondary);
  font-size: 13px;
}

.hSR-summary-value {
  color: var(--text-primary);
  font-size: 28px;
  font-weight: 700;
  line-height: 1;
}

.hSR-summary-divider {
  width: 1px;
  height: 48px;
  background: #e8e8e8;
}

.task-stats-sidebar {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.weather-widget {
  padding: 20px;
  border-radius: var(--radius);
  background: var(--card-bg);
  box-shadow: var(--shadow);
  flex: 1;
}

.calendar-widget {
  padding: 20px;
  border-radius: var(--radius);
  background: var(--card-bg);
  box-shadow: var(--shadow);
  flex: 1;
}

.weather-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.weather-main {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.weather-city {
  color: var(--text-primary);
  font-size: 16px;
  font-weight: 600;
}

.weather-date {
  color: var(--text-secondary);
  font-size: 12px;
}

.weather-temp {
  flex: 1;
}

.temp-value {
  color: var(--primary);
  font-size: 48px;
  font-weight: 300;
  line-height: 1;
}

.weather-desc p {
  margin-top: 8px;
  color: var(--text-secondary);
  font-size: 12px;
  line-height: 1.6;
}

.air-quality {
  color: #52c41a !important;
}

.weather-icon {
  color: #faad14;
  font-size: 56px;
}

.weather-forecast {
  display: flex;
  padding-top: 16px;
  justify-content: space-between;
  border-top: 1px solid var(--border);
}

.forecast-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  color: var(--text-secondary);
  font-size: 12px;
}

.forecast-item i {
  color: #8c8c8c;
  font-size: 16px;
}

// 日历组件样式
.calendar-widget {
  padding: 20px;
  border-radius: var(--radius);
  background: var(--card-bg);
  box-shadow: var(--shadow);
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.calendar-month {
  color: var(--text-primary);
  font-size: 16px;
  font-weight: 600;
}

.calendar-nav {
  display: flex;
  gap: 8px;
}

.calendar-nav-btn {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 6px;
  background: #f0f2f5;
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s;
  font-size: 12px;

  &:hover {
    background: var(--primary);
    color: #fff;
  }
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 8px;
  margin-bottom: 20px;
}

.calendar-day-header {
  text-align: center;
  color: var(--text-secondary);
  font-size: 12px;
  font-weight: 500;
  padding: 8px 0;
}

.calendar-day {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
  position: relative;

  &:hover {
    background: #f0f2f5;
  }

  &.other-month {
    color: var(--text-secondary);
    opacity: 0.5;
  }

  &.selected {
    background: var(--primary);
    color: #fff;
  }

  &.today {
    border: 2px solid var(--primary);
    font-weight: 600;
  }

  .task-indicator {
    position: absolute;
    bottom: 2px;
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background: #ff4d4f;
  }
}

.tasks-section {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid var(--border);
}

.tasks-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.tasks-title {
  color: var(--text-primary);
  font-size: 15px;
  font-weight: 600;
}

.selected-date {
  color: var(--primary);
  font-size: 12px;
}

.task-list {
  list-style: none;
}

.task-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 0;
  border-bottom: 1px dashed #f0f0f0;

  &:last-child {
    border-bottom: none;
  }

  &.completed {
    opacity: 0.6;
  }

  .task-checkbox {
    width: 16px;
    height: 16px;
    border: 1px solid #d9d9d9;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.2s;

    &:checked {
      background: var(--primary);
      border-color: var(--primary);
    }
  }

  .task-text {
    flex: 1;
    color: var(--text-primary);
    font-size: 13px;
  }
}

.quick-link-btn {
  display: flex;
  width: 100%;
  padding: 14px 20px;
  align-items: center;
  justify-content: center;
  gap: 8px;
  border: 0;
  border-radius: var(--radius);
  color: #fff;
  background: var(--primary);
  box-shadow: var(--shadow);
  cursor: pointer;
  font-size: 14px;
  transition: background 0.2s;

  &:hover {
    background: var(--primary-dark);
  }

  .fa-chevron-down {
    margin-left: auto;
    font-size: 12px;
  }
}

@media (max-width: 1400px) {
  .charts-section {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 1200px) {
  .status-row {
    grid-template-columns: 1fr 1fr;
  }

  .approval-card {
    grid-column: span 2;
  }

  .task-stats-section {
    grid-template-columns: 1fr;
  }

  .task-stats-sidebar {
    flex-direction: row;
    flex-wrap: wrap;
  }

  .calendar-widget {
    min-width: 280px;
    flex: 1;
  }

  .quick-link-btn {
    width: auto;
    min-width: 200px;
    flex: 0 0 auto;
  }
}

@media (max-width: 768px) {
  :global(:root) {
    --sidebar-width: 56px;
  }

  .main {
    padding: 0 12px 12px;
  }

  .header {
    align-items: flex-start;
    flex-direction: column;

    &-search {
      width: 100%;
      min-width: unset;
    }
  }

  .logo-text {
    h1 {
      font-size: 14px;
    }

    p {
      display: none;
    }
  }

  .status-row {
    grid-template-columns: 1fr;
  }

  .hSR-summary {
    flex-direction: column;
    gap: 12px;
  }

  .hSR-summary-divider {
    width: 80%;
    height: 1px;
  }

  .approval-card {
    grid-column: span 1;
  }

  .approval-content {
    flex-direction: column;
    gap: 16px;
  }

  .personal-info-section,
  .approval-icons-section {
    width: 100%;
  }

  .quick-links {
    justify-content: center;
    gap: 16px;
  }

  .project-stats-section {
    grid-template-columns: repeat(3, 1fr);
  }

  .project-finance-section {
    grid-template-columns: 1fr;
  }
}
</style>

<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-top">
        <div class="avatar">
          <img src="https://api.dicebear.com/7.x/avataaars/svg?seed=ZhangWei&backgroundColor=b6e3f4" alt="张伟">
        </div>
        <div class="user-name">张伟大</div>
        <div class="sidebar-menu">
          <a href="#" class="sidebar-item">
            <span>待办</span>
            <span class="badge badge-orange">2</span>
          </a>
          <a href="#" class="sidebar-item">
            <span>未读</span>
            <span class="badge badge-blue">5</span>
          </a>
        </div>
      </div>
      <nav class="sidebar-nav" aria-label="主导航">
        <a href="#" class="nav-icon" title="搜索"><i class="fas fa-search"></i></a>
        <a href="#" class="nav-icon active" title="首页"><i class="fas fa-home"></i></a>
        <a href="#" class="nav-icon" title="通知"><i class="fas fa-bell"></i></a>
        <a href="#" class="nav-icon" title="设置"><i class="fas fa-cog"></i></a>
      </nav>
    </aside>

    <main class="main">
      <header class="header">
        <div class="logo">
          <div class="logo-icon">CRSGC</div>
          <div class="logo-text">
            <h1>中国铁路上海局集团有限公司</h1>
            <p>China State Railway Group Co.,Ltd.</p>
          </div>
        </div>
        <div class="header-search">
          <input type="text" placeholder="请输入搜索关键词">
          <button class="btn-search" type="button">搜索</button>
        </div>
      </header>

      <section class="quick-links" aria-label="快捷入口">
        <a v-for="item in quickLinks" :key="item.label" href="#" class="quick-item" 
           @mouseenter="item.label === '办公平台' && (showOfficeDetail = true)"
           @mouseleave="showOfficeDetail = false">
          <div class="quick-icon" :style="{ background: item.color }">
            <i class="fas" :class="item.icon"></i>
          </div>
          <span>{{ item.label }}</span>
          <div v-if="item.badge" class="quick-badge">{{ item.badge }}</div>
          <div v-if="item.details && showOfficeDetail" class="office-detail-popup">
            <div class="office-detail-popup-title">详细信息</div>
            <div v-for="detail in item.details" :key="detail.type" class="office-detail-popup-item">
              <span class="office-detail-popup-item-type">{{ detail.type }}：</span>
              <span class="office-detail-popup-item-count">{{ detail.count }}</span>
            </div>
          </div>
        </a>
        <a href="#" class="quick-item add-module">
          <div class="quick-icon dashed">
            <i class="fas fa-plus"></i>
          </div>
          <span>添加模块</span>
        </a>
      </section>

      <section class="status-row">
        <div class="status-card approval-card">
          <div class="approval-content">
            <div class="personal-info-section">
              <div class="personal-info-content">
                <div class="info-greeting">早上好 ~ </div>
                <div class="info-name-phone">
                  张伟（13866595678）
                </div>
                <div class="info-slogan">对未来的真正期望，就是把一切都奉献给现在</div>
                <div class="info-position">机务段三段段长</div>
              </div>
            </div>
            <div class="approval-icons-section">
              <a href="#" class="action-item">
                <div class="action-icon"><i class="fas fa-tasks"></i></div>
                <span class="action-text">我审批的</span>
                <span class="badge badge-white">3</span>
              </a>
              <a href="#" class="action-item">
                <div class="action-icon"><i class="fas fa-paper-plane"></i></div>
                <span class="action-text">我发起的</span>
                <span class="badge badge-white">7</span>
              </a>
            </div>
          </div>
          <div class="approval-actions">
            <a href="#" class="action-item">
              <div class="action-icon"><i class="fas fa-calendar-minus"></i></div>
              <span>在岗</span>
            </a>
            <a href="#" class="action-item action-item-highlight"
              :class="{ 'is-tooltip-visible': isTripTooltipVisible }" title="出差时间"
              @click.prevent="isTripTooltipVisible = !isTripTooltipVisible">
              <div class="action-icon"><i class="fas fa-plane"></i></div>
              <span>出差</span>
              <span class="action-tooltip">06.10 - 06.14 · 北京总部</span>
            </a>
            <a href="#" class="action-item">
              <div class="action-icon"><i class="fas fa-walking"></i></div>
              <span>请假</span>
            </a>
          </div>
        </div>
        <div class="status-card todo-card">
          <div class="todo-section">
            <div class="todo-header">
              <span class="todo-label">个人待办</span>
              <span class="todo-count">25 <small>项</small></span>
            </div>
            <div class="progress-wrap">
              <div class="progress-bar">
                <div class="progress-fill progress-fill-45"></div>
              </div>
              <span class="progress-text">完成度 45%</span>
            </div>
          </div>
          <div class="todo-section">
            <div class="todo-header">
              <span class="todo-label">个人待阅</span>
              <span class="todo-count">18 <small>项</small></span>
            </div>
            <div class="progress-wrap">
              <div class="progress-bar">
                <div class="progress-fill progress-fill-80"></div>
              </div>
              <span class="progress-text">完成量 80%</span>
            </div>
          </div>
        </div>

        <div class="status-card notice-card">
          <h3 class="card-title">通知公告</h3>
          <ul class="notice-list">
            <li v-for="notice in notices" :key="notice.title">
              <a href="#">{{ notice.title }}</a>
              <span class="date">{{ notice.date }}</span>
            </li>
          </ul>
        </div>
      </section>

      <!-- 任务统计模块 -->
      <section class="task-stats-section">
        <div class="task-stats-main">
          <!-- 任务统计卡片 -->
          <div class="task-stats-card">
            <h3 class="task-stats-title">XXXXX项目</h3>
            <div class="project-finance-section">
              <div class="finance-item">
                <i class="fas fa-file-invoice finance-icon"></i>
                <div class="finance-body">
                  <span class="finance-label">中标金额</span>
                  <span class="finance-value">¥12,500,000.00</span>
                </div>
              </div>
              <div class="finance-item">
                <i class="fas fa-wallet finance-icon"></i>
                <div class="finance-body">
                  <span class="finance-label">已支付金额</span>
                  <span class="finance-value">¥8,320,000.00</span>
                </div>
              </div>
            </div>
            <div class="project-stats-section">
              <div class="project-stat-item">
                <span class="project-stat-label">项目总数</span>
                <span class="project-stat-count">156</span>
              </div>
              <div class="project-stat-item">
                <span class="project-stat-label">投标书</span>
                <span class="project-stat-count">89</span>
              </div>
              <div class="project-stat-item">
                <span class="project-stat-label">在建数</span>
                <span class="project-stat-count">42</span>
              </div>
              <div class="project-stat-item">
                <span class="project-stat-label">初检数</span>
                <span class="project-stat-count">28</span>
              </div>
              <div class="project-stat-item">
                <span class="project-stat-label">验收数</span>
                <span class="project-stat-count">18</span>
              </div>
            </div>
          </div>

          <!-- 高铁线路通车情况 & 任务类型分布 -->
          <div class="charts-section">
            <div class="chart-card">
              <h4 class="chart-card-title">高铁线路通车情况</h4>
              <div class="hSR-summary">
                <div class="hSR-summary-item hSR-summary-opened">
                  <span class="hSR-summary-icon"><i class="fas fa-subway"></i></span>
                  <div class="hSR-summary-body">
                    <span class="hSR-summary-label">通车路段数量</span>
                    <span class="hSR-summary-value">{{ totalOpened }}</span>
                  </div>
                </div>
                <div class="hSR-summary-divider"></div>
                <div class="hSR-summary-item hSR-summary-unopened">
                  <span class="hSR-summary-icon"><i class="fas fa-road"></i></span>
                  <div class="hSR-summary-body">
                    <span class="hSR-summary-label">未通车路段数量</span>
                    <span class="hSR-summary-value">{{ totalUnopened }}</span>
                  </div>
                </div>
              </div>
              <div ref="barChartRef" class="chart-container" style="height: 200px;"></div>
            </div>
            <div class="chart-card">
              <h4 class="chart-card-title">当前月任务类型分布</h4>
              <p class="task-type-text">
                <span v-for="(item, index) in currentMonthTaskTypes" :key="item.type">
                  {{ item.type }} {{ item.value }}个<template v-if="index < currentMonthTaskTypes.length - 1"> · </template>
                </span>
              </p>
              <div ref="radarChartRef" class="chart-container"></div>
            </div>
          </div>
        </div>

        <aside class="task-stats-sidebar">
          <!-- 天气卡片 -->
          <div class="weather-widget">
            <div class="weather-header">
              <span class="weather-city">上海</span>
              <span class="weather-date">2026年7月3日</span>
            </div>
            <div class="weather-main">
              <div class="weather-temp">
                <span class="temp-value">26°</span>
                <div class="weather-desc">
                  <p>中雨 | 今晚台风预警，下班记得带伞</p>
                  <p class="air-quality">空气质量 优</p>
                </div>
              </div>
              <div class="weather-icon"><i class="fas fa-cloud-sun"></i></div>
            </div>
            <div class="weather-forecast">
              <div class="forecast-item"><span>14:00</span><i class="fas fa-cloud-rain"></i><span>24°</span></div>
              <div class="forecast-item"><span>15:00</span><i class="fas fa-cloud"></i><span>23°</span></div>
              <div class="forecast-item"><span>16:00</span><i class="fas fa-cloud-sun"></i><span>22°</span></div>
              <div class="forecast-item"><span>17:00</span><i class="fas fa-sun"></i><span>21°</span></div>
              <div class="forecast-item"><span>18:00</span><i class="fas fa-cloud-moon"></i><span>21°</span></div>
            </div>
          </div>

          <!-- 日历卡片 -->
          <div class="calendar-widget">
            <div class="calendar-header">
              <span class="calendar-month">{{ getMonthName(currentMonth) }}</span>
              <div class="calendar-nav">
                <button class="calendar-nav-btn" @click="previousMonth">
                  <i class="fas fa-chevron-left"></i>
                </button>
                <button class="calendar-nav-btn" @click="goToToday">今天</button>
                <button class="calendar-nav-btn" @click="nextMonth">
                  <i class="fas fa-chevron-right"></i>
                </button>
              </div>
            </div>

            <div class="calendar-grid">
              <div class="calendar-day-header">日</div>
              <div class="calendar-day-header">一</div>
              <div class="calendar-day-header">二</div>
              <div class="calendar-day-header">三</div>
              <div class="calendar-day-header">四</div>
              <div class="calendar-day-header">五</div>
              <div class="calendar-day-header">六</div>

              <div v-for="(day, index) in calendarDays" :key="index" class="calendar-day" :class="{
                'other-month': !day.currentMonth,
                'selected': isSameDay(day.date, selectedDate),
                'today': isSameDay(day.date, new Date())
              }" @click="selectDate(day.date)">
                {{ day.day }}
                <div v-if="hasTask(day.date)" class="task-indicator"></div>
              </div>
            </div>

            <div class="tasks-section">
              <div class="tasks-header">
                <span class="tasks-title">待办任务</span>
                <span class="selected-date">{{ formatDate(selectedDate) }}</span>
              </div>

              <ul class="task-list">
                <li v-for="task in getTasksForDate(selectedDate)" :key="task.title" class="task-item"
                  :class="{ 'completed': task.completed }">
                  <input type="checkbox" class="task-checkbox" :checked="task.completed" @change="toggleTask(task)">
                  <span class="task-text">{{ task.title }}</span>
                </li>
                <li v-if="getTasksForDate(selectedDate).length === 0" class="task-item">
                  <span class="task-text">该日期暂无待办任务</span>
                </li>
              </ul>
            </div>
          </div>

        </aside>
      </section>
    </main>
  </div>
</template>
