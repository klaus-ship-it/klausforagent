<script setup lang="ts">
import { computed, ref } from 'vue'
import {
  NButton,
  NCard,
  NConfigProvider,
  NDataTable,
  NDivider,
  NForm,
  NFormItem,
  NIcon,
  NInput,
  NInputNumber,
  NLayout,
  NLayoutHeader,
  NLayoutSider,
  NModal,
  NSelect,
  NSpace,
  NStatistic,
  NTabPane,
  NTabs,
  NTag,
} from 'naive-ui'
import {
  BarChartOutline,
  CashOutline,
  ChevronDownOutline,
  CopyOutline,
  GitNetworkOutline,
  GridOutline,
  ListOutline,
  LogOutOutline,
  PeopleOutline,
  SearchOutline,
  ShareSocialOutline,
  WalletOutline,
} from '@vicons/ionicons5'

type NavKey = 'dashboard' | 'agents' | 'players' | 'codes' | 'commissionPlans' | 'commission' | 'withdrawal' | 'reports' | 'logs' | 'profile'
type Scope = 'direct' | 'all'
type Role = '運營商' | '總代理' | '一般代理'

interface AgentRow {
  id: string
  uid?: string
  account: string
  displayName?: string
  referralCode?: string
  phone?: string
  email?: string
  contactMethod?: string
  twoFactor?: '已啟用' | '未啟用'
  level: string
  path: string
  currency: string
  point: number
  children: number
  status: '啟用' | '停用'
  planId: string
  model?: 'CPA' | '佔成' | '返佣'
  cycle?: '即時' | '每日' | '每週' | '每月'
}

interface CommissionPlan {
  id: string
  name: string
  model: 'CPA' | '佔成' | '返佣'
  cycle: '即時' | '每日' | '每週' | '每月'
  allocationRate: number
  description: string
  assignedCount: number
  status: '啟用' | '停用'
}

interface PlayerRow {
  id: string
  account: string
  level: string
  path: string
  currency: string
  deposit: number
  bet: number
  status: '啟用' | '停用'
}

const themeOverrides = {
  common: { primaryColor: '#2f6fed', primaryColorHover: '#4c83ef', borderRadius: '10px' },
  Button: { borderRadiusMedium: '9px' },
}
const loggedIn = ref(false)
const loginAccount = ref('agent_demo')
const loginPassword = ref('demo123')
const activeKey = ref<NavKey>('dashboard')
const currentRole = ref<Role>('總代理')
const scope = ref<Scope>('all')
const search = ref('')
const agentLevelFilter = ref<string | null>(null)
const agentStatusFilter = ref<string | null>(null)
const agentCommissionFilter = ref<'CPA' | '佔成' | '返佣' | null>(null)
const showCreateAgent = ref(false)
const newAgentAccount = ref('')
const newAgentPoint = ref<number | null>(0)
const newAgentCurrency = ref('TWD')
const newAgentName = ref('')
const newAgentPhone = ref('')
const newAgentEmail = ref('')
const newAgentPassword = ref('')
const newAgentRole = ref<Role>('一般代理')
const showAgentDetail = ref(false)
const selectedAgent = ref<AgentRow | null>(null)
const detailTab = ref<'basic' | 'relationship' | 'commission' | 'logs'>('basic')
const draftPlanId = ref('PLAN-001')
const newAgentPlanId = ref('PLAN-001')
const showPlanEditor = ref(false)
const editingPlan = ref<CommissionPlan | null>(null)
const planName = ref('')
const planModel = ref<'CPA' | '佔成' | '返佣'>('返佣')
const planCycle = ref<'即時' | '每日' | '每週' | '每月'>('每週')
const planRate = ref<number | null>(4)
const planDescription = ref('')
const showWithdrawal = ref(false)
const showBankCard = ref(false)
const bankName = ref('台新銀行')
const bankAccount = ref('123456789012')
const bankHolder = ref('Klaus Lin')
const withdrawalAmount = ref<number | null>(null)
const selectedCycle = ref('每週')
const notice = ref<{ type: 'success' | 'warning'; title: string; content: string } | null>(null)

const agentRows = ref<AgentRow[]>([
  { id: 'A-1001', uid: 'AG-10001', account: 'agent_taipei', displayName: '台北總代理', referralCode: 'AGT-TW-8F4K', phone: '0912345678', email: 'taipei@example.com', contactMethod: 'Line: @agent_taipei', twoFactor: '已啟用', level: '總代理', path: 'agent_taipei', currency: 'TWD', point: 6, children: 3, status: '啟用', planId: 'PLAN-001', model: '返佣', cycle: '每週' },
  { id: 'A-1002', uid: 'AG-10024', account: 'north_team', displayName: '北區團隊', referralCode: 'AGT-TW-NORTH', phone: '0987654321', email: 'north@example.com', contactMethod: 'Line: @north_team', twoFactor: '未啟用', level: '一級代理', path: 'agent_taipei > north_team', currency: 'TWD', point: 4, children: 8, status: '啟用', planId: 'PLAN-001', model: '返佣', cycle: '每週' },
  { id: 'A-1003', uid: 'AG-10088', account: 'east_team', displayName: '東區團隊', referralCode: 'AGT-TW-EAST', phone: '0955123788', email: 'east@example.com', contactMethod: 'Email', twoFactor: '未啟用', level: '一級代理', path: 'agent_taipei > east_team', currency: 'TWD', point: 3, children: 5, status: '啟用', planId: 'PLAN-001', model: '返佣', cycle: '每週' },
  { id: 'A-1004', uid: 'AG-10102', account: 'sub_partner_01', displayName: '合作夥伴 01', referralCode: 'AGT-TW-SUB01', phone: '0933123456', email: 'partner01@example.com', contactMethod: 'Line: @sub_partner_01', twoFactor: '未啟用', level: '二級代理', path: 'agent_taipei > north_team > sub_partner_01', currency: 'TWD', point: 2, children: 2, status: '啟用', planId: 'PLAN-001', model: '返佣', cycle: '每週' },
])

const commissionPlans = ref<CommissionPlan[]>([
  { id: 'PLAN-001', name: '台灣返佣標準方案', model: '返佣', cycle: '每週', allocationRate: 6, description: '依有效投注額計算返佣；下級可用點數不得超過本方案額度。', assignedCount: 3, status: '啟用' },
  { id: 'PLAN-002', name: 'CPA 新客獎勵方案', model: 'CPA', cycle: '每月', allocationRate: 4, description: '註冊、首存、流水達標各自設定固定獎勵。', assignedCount: 0, status: '啟用' },
  { id: 'PLAN-003', name: '佔成合作方案', model: '佔成', cycle: '每月', allocationRate: 5, description: '依玩家總輸贏扣除行政成本後按比例分配，支援負數規則。', assignedCount: 0, status: '啟用' },
])

const playerRows = ref<PlayerRow[]>([
  { id: 'P-20481', account: 'player_k***', level: '一級玩家', path: 'agent_taipei > north_team > player_k***', currency: 'TWD', deposit: 12800, bet: 95600, status: '啟用' },
  { id: 'P-20482', account: 'member_8***', level: '二級玩家', path: 'agent_taipei > north_team > sub_partner_01 > member_8***', currency: 'TWD', deposit: 8600, bet: 43200, status: '啟用' },
  { id: 'P-20483', account: 'user_l***', level: '一級玩家', path: 'agent_taipei > east_team > user_l***', currency: 'TWD', deposit: 3200, bet: 18400, status: '停用' },
])

const logs = ref([
  { time: '2026-08-31 10:22:04', type: '登入', actor: 'agent_demo', detail: '代理後台登入成功', ip: '10.20.8.15' },
  { time: '2026-08-30 18:04:12', type: '設定反傭', actor: 'agent_demo', detail: 'north_team 返佣比例 3% → 4%', ip: '10.20.8.15' },
  { time: '2026-08-29 14:11:38', type: '開設代理', actor: 'agent_demo', detail: '建立 sub_partner_01，幣別 TWD', ip: '10.20.8.15' },
  { time: '2026-08-27 09:43:51', type: '提領申請', actor: 'agent_demo', detail: '申請提領 TWD 12,000，待平台審核', ip: '10.20.8.15' },
])

const navGroups = computed(() => [
  { title: '工作台', items: [{ key: 'dashboard', label: '營運概覽', icon: GridOutline }] },
  { title: '下級管理', items: [{ key: 'agents', label: '代理管理', icon: GitNetworkOutline }, { key: 'players', label: '玩家管理', icon: PeopleOutline }, { key: 'codes', label: '推廣碼', icon: ShareSocialOutline }] },
  { title: '傭金與報表', items: [{ key: 'commissionPlans', label: '傭金方案', icon: CashOutline }, { key: 'commission', label: '傭金中心', icon: CashOutline }, { key: 'withdrawal', label: '提領傭金', icon: WalletOutline }, { key: 'reports', label: '下級報表', icon: BarChartOutline }] },
  { title: '系統', items: [{ key: 'logs', label: '操作日誌', icon: ListOutline }, { key: 'profile', label: '帳戶設定', icon: PeopleOutline }] },
] as const)

const identity = computed(() => currentRole.value === '運營商' ? { account: 'operator_demo', label: '運營商', currency: '多幣別' } : currentRole.value === '總代理' ? { account: 'agent_taipei', label: '總代理', currency: 'TWD' } : { account: 'north_team', label: '一般代理', currency: 'TWD' })
const canCreateAgent = computed(() => currentRole.value !== '運營商' || currentRole.value === '運營商')
const createTitle = computed(() => currentRole.value === '運營商' ? '開設總代理' : '開設下級代理')
const pageTitle = computed(() => navGroups.value.flatMap((group) => group.items).find((item) => item.key === activeKey.value)?.label ?? '營運概覽')
const filteredAgents = computed(() => agentRows.value.filter((row) => {
  const inScope = scope.value === 'all' || row.level === '一級代理'
  const inLevel = !agentLevelFilter.value || row.level === agentLevelFilter.value
  const inStatus = !agentStatusFilter.value || row.status === agentStatusFilter.value
  const inCommission = !agentCommissionFilter.value || row.model === agentCommissionFilter.value
  return inScope && inLevel && inStatus && inCommission && `${row.account}${row.path}`.toLowerCase().includes(search.value.toLowerCase())
}))
const filteredPlayers = computed(() => playerRows.value.filter((row) => {
  const inScope = scope.value === 'all' || row.level === '一級玩家'
  return inScope && `${row.account}${row.path}`.toLowerCase().includes(search.value.toLowerCase())
}))
const planOptions = computed(() => commissionPlans.value.filter((plan) => plan.status === '啟用' && (currentRole.value === '運營商' || plan.model === lineModel.value)).map((plan) => ({ label: `${plan.name}（${plan.model}／${plan.cycle}）`, value: plan.id })))
const selectedPlan = computed(() => commissionPlans.value.find((plan) => plan.id === draftPlanId.value) ?? commissionPlans.value[0])
const lineModel = computed(() => currentRole.value === '運營商' ? null : (agentRows.value.find((row) => row.account === identity.value.account)?.model ?? '返佣'))
const visiblePlans = computed(() => commissionPlans.value.filter((plan) => currentRole.value === '運營商' || plan.model === lineModel.value))
const availablePlanOptions = computed(() => visiblePlans.value.filter((plan) => plan.status === '啟用').map((plan) => ({ label: `${plan.name}（${plan.model}／${plan.cycle}）`, value: plan.id })))
const lineMaxRate = computed(() => currentRole.value === '運營商' ? 100 : (agentRows.value.find((row) => row.account === identity.value.account)?.point ?? 0))
const canManagePlans = computed(() => true)

function login() {
  if (!loginAccount.value || !loginPassword.value) {
    showNotice('warning', '請輸入登入資訊', '此為原型示範，輸入任意非空白帳密即可登入。')
    return
  }
  loggedIn.value = true
  logs.value.unshift({ time: '2026-08-31 10:28:00', type: '登入', actor: loginAccount.value, detail: `${identity.value.label}登入代理後台成功`, ip: '10.20.8.15' })
}

function showNotice(type: 'success' | 'warning', title: string, content: string) {
  notice.value = { type, title, content }
  window.setTimeout(() => { notice.value = null }, 3500)
}

function logout() {
  loggedIn.value = false
  activeKey.value = 'dashboard'
}

function selectNav(key: string) {
  activeKey.value = key as NavKey
  search.value = ''
  agentLevelFilter.value = null
  agentStatusFilter.value = null
  agentCommissionFilter.value = null
}

function deactivateAgent(row: AgentRow) {
  row.status = row.status === '啟用' ? '停用' : '啟用'
  logs.value.unshift({ time: '2026-08-31 10:30:00', type: row.status === '啟用' ? '啟用代理' : '停用代理', actor: loginAccount.value, detail: `${row.account} 狀態變更為${row.status}`, ip: '10.20.8.15' })
  showNotice('success', '狀態已更新', `${row.account} 現在為${row.status}`)
}

function createAgent() {
  if (!newAgentAccount.value.trim()) {
    showNotice('warning', '請輸入代理帳號', '代理帳號為必填欄位。')
    return
  }
  if (!newAgentPassword.value.trim()) {
    showNotice('warning', '請設定登入密碼', '建立代理時必須設定初始登入密碼。')
    return
  }
  const next = agentRows.value.length + 1
  const account = newAgentAccount.value.trim()
  const selected = commissionPlans.value.find((plan) => plan.id === newAgentPlanId.value) ?? commissionPlans.value[0]
  const savedPoint = selected.allocationRate
  const level = currentRole.value === '運營商' ? '總代理' : currentRole.value === '總代理' ? '一級代理' : '二級代理'
  const basePath = currentRole.value === '運營商' ? account : `${identity.value.account} > ${account}`
  agentRows.value.push({ id: `A-${1000 + next}`, uid: `AG-${10000 + next}`, account, displayName: newAgentName.value.trim() || account, referralCode: `AGT-${newAgentCurrency.value}-${String(next).padStart(4, '0')}`, phone: newAgentPhone.value.trim(), email: newAgentEmail.value.trim(), contactMethod: '尚未設定', twoFactor: '未啟用', level, path: basePath, currency: newAgentCurrency.value, point: savedPoint, children: 0, status: '啟用', planId: selected.id, model: selected.model, cycle: selected.cycle })
  selected.assignedCount += 1
  showCreateAgent.value = false
  newAgentAccount.value = ''
  newAgentName.value = ''
  newAgentPhone.value = ''
  newAgentEmail.value = ''
  newAgentPassword.value = ''
  newAgentPlanId.value = 'PLAN-001'
  logs.value.unshift({ time: '2026-08-31 10:31:00', type: '開設代理', actor: loginAccount.value, detail: `建立${level} ${account}（立即啟用）`, ip: '10.20.8.15' })
  showNotice('success', '代理已建立', `${account} 可立即登入使用；已套用「${selected.name}」。`)
}

function submitWithdrawal() {
  if (!withdrawalAmount.value || withdrawalAmount.value < 1000) {
    showNotice('warning', '未符合提領條件', '最低提領金額為 TWD 1,000。')
    return
  }
  showWithdrawal.value = false
  logs.value.unshift({ time: '2026-08-31 10:32:00', type: '提領申請', actor: loginAccount.value, detail: `申請提領 TWD ${withdrawalAmount.value.toLocaleString()}，待平台審核`, ip: '10.20.8.15' })
  showNotice('success', '提領申請已送出', '平台審核完成前，金額會維持在待審核餘額。')
  withdrawalAmount.value = null
}

function saveBankCard() {
  if (!bankName.value || !bankAccount.value || !bankHolder.value) {
    showNotice('warning', '資料未完成', '銀行名稱、帳號與戶名皆為必填。')
    return
  }
  showBankCard.value = false
  logs.value.unshift({ time: '2026-08-31 10:36:00', type: '更新帳戶', actor: loginAccount.value, detail: '新增／更新傭金收款銀行卡', ip: '10.20.8.15' })
  showNotice('success', '銀行卡已更新', '此銀行卡將作為後續傭金提領的收款帳戶。')
}

function maskPhone(phone?: string) {
  if (!phone) return '未填寫'
  return phone.length > 5 ? `${phone.slice(0, 2)}******${phone.slice(-3)}` : '******'
}

function maskEmail(email?: string) {
  if (!email) return '未填寫'
  const [name, domain] = email.split('@')
  if (!domain) return `${name.slice(0, 2)}******`
  return `${name.slice(0, 2)}******@${domain}`
}

function openAgent(row: AgentRow) {
  selectedAgent.value = row
  detailTab.value = 'basic'
  draftPlanId.value = row.planId
  showAgentDetail.value = true
}

function saveAgentPlan() {
  if (!selectedAgent.value) return
  const plan = commissionPlans.value.find((item) => item.id === draftPlanId.value)
  if (!plan) return
  const oldPlan = commissionPlans.value.find((item) => item.id === selectedAgent.value?.planId)
  selectedAgent.value.planId = plan.id
  selectedAgent.value.model = plan.model
  selectedAgent.value.cycle = plan.cycle
  selectedAgent.value.point = plan.allocationRate
  if (oldPlan && oldPlan.id !== plan.id) oldPlan.assignedCount = Math.max(0, oldPlan.assignedCount - 1)
  if (!oldPlan || oldPlan.id !== plan.id) plan.assignedCount += 1
  logs.value.unshift({ time: '2026-08-31 10:34:00', type: '套用傭金方案', actor: loginAccount.value, detail: `${selectedAgent.value.account} 套用「${plan.name}」${oldPlan ? `（原方案：${oldPlan.name}）` : ''}`, ip: '10.20.8.15' })
  showAgentDetail.value = false
  showNotice('success', '傭金方案已更新', `${selectedAgent.value.account} 已套用「${plan.name}」。`)
}

function openPlanEditor(plan?: CommissionPlan) {
  editingPlan.value = plan ?? null
  planName.value = plan?.name ?? ''
  planModel.value = plan?.model ?? '返佣'
  planCycle.value = plan?.cycle ?? '每週'
  planRate.value = plan?.allocationRate ?? 4
  planDescription.value = plan?.description ?? ''
  showPlanEditor.value = true
}

function savePlan() {
  if (!planName.value.trim() || planRate.value === null) {
    showNotice('warning', '方案資料未完成', '請填寫方案名稱與可分配點數。')
    return
  }
  if (currentRole.value !== '運營商' && planModel.value !== lineModel.value) {
    showNotice('warning', '傭金模式不符', `目前代理線為${lineModel.value}模式，只能建立或編輯${lineModel.value}方案。`)
    return
  }
  if (planRate.value > lineMaxRate.value) {
    showNotice('warning', '超過上級可分配點數', `本代理最多只能設定 ${lineMaxRate.value}% 的可分配點數。`)
    return
  }
  if (editingPlan.value) {
    editingPlan.value.name = planName.value.trim()
    editingPlan.value.model = planModel.value
    editingPlan.value.cycle = planCycle.value
    editingPlan.value.allocationRate = planRate.value
    editingPlan.value.description = planDescription.value.trim()
    showNotice('success', '傭金方案已更新', `「${editingPlan.value.name}」設定已儲存。`)
  } else {
    const id = `PLAN-${String(commissionPlans.value.length + 1).padStart(3, '0')}`
    commissionPlans.value.push({ id, name: planName.value.trim(), model: planModel.value, cycle: planCycle.value, allocationRate: planRate.value, description: planDescription.value.trim() || '依此方案規則產生代理傭金。', assignedCount: 0, status: '啟用' })
    showNotice('success', '傭金方案已建立', `「${planName.value.trim()}」可供代理套用。`)
  }
  showPlanEditor.value = false
  editingPlan.value = null
}

function togglePlan(plan: CommissionPlan) {
  plan.status = plan.status === '啟用' ? '停用' : '啟用'
  logs.value.unshift({ time: '2026-08-31 10:35:00', type: plan.status === '啟用' ? '啟用傭金方案' : '停用傭金方案', actor: loginAccount.value, detail: `${plan.name} 狀態變更為${plan.status}`, ip: '10.20.8.15' })
  showNotice('success', '方案狀態已更新', `${plan.name} 現在為${plan.status}`)
}

async function copyReferral(code: string, label: string) {
  await navigator.clipboard?.writeText(code)
  showNotice('success', `${label}已複製`, code)
}

function exportMessage(label: string) {
  showNotice('success', `${label}已準備`, '原型示範不會下載真實資料；正式版將依目前篩選條件匯出。')
}

const playerColumns = [
  { title: '玩家帳號', key: 'account' },
  { title: '層級', key: 'level' },
  { title: '樹狀路徑', key: 'path' },
  { title: '幣別', key: 'currency' },
  { title: '期間儲值', key: 'deposit', render: (row: PlayerRow) => row.deposit.toLocaleString() },
  { title: '期間有效投注', key: 'bet', render: (row: PlayerRow) => row.bet.toLocaleString() },
  { title: '狀態', key: 'status' },
]
</script>

<template>
  <NConfigProvider :theme-overrides="themeOverrides">
    <div>
      <div v-if="notice" class="toast" :class="notice.type"><strong>{{ notice.title }}</strong><span>{{ notice.content }}</span></div>
      <div v-if="!loggedIn" class="login-page">
        <div class="login-glow glow-one" />
        <div class="login-glow glow-two" />
        <NCard class="login-card" :bordered="false">
          <div class="brand-mark">Y</div>
          <div class="eyebrow">YOTA PARTNER CONSOLE</div>
          <h1>代理後台</h1>
          <p class="muted">管理下級代理、玩家數據與傭金結算</p>
          <NForm class="login-form" @submit.prevent="login">
            <NFormItem label="帳號">
              <NInput v-model:value="loginAccount" placeholder="輸入代理帳號" @keyup.enter="login" />
            </NFormItem>
            <NFormItem label="密碼">
              <NInput v-model:value="loginPassword" type="password" show-password-on="click" placeholder="輸入密碼" @keyup.enter="login" />
            </NFormItem>
            <NButton type="primary" block size="large" @click="login">登入代理後台</NButton>
          </NForm>
          <div class="demo-hint">原型示範：任意非空白帳密即可登入</div>
        </NCard>
      </div>

      <NLayout v-else has-sider class="app-shell">
        <NLayoutSider bordered :width="250" class="sidebar">
          <div class="side-brand"><span class="brand-mark small">Y</span><div><strong>YOTA</strong><span>AGENT CONSOLE</span></div></div>
          <div class="agent-chip"><div class="avatar">{{ identity.label.slice(0, 1) }}</div><div><strong>{{ identity.account }}</strong><span>{{ identity.label }} · {{ identity.currency }}</span></div><ChevronDownOutline class="chip-chevron" /></div>
          <div class="role-switcher"><span>原型角色切換</span><NSelect v-model:value="currentRole" size="small" :options="[{label:'運營商',value:'運營商'},{label:'總代理',value:'總代理'},{label:'一般代理',value:'一般代理'}]" /></div>
          <div class="nav-scroll">
            <div v-for="group in navGroups" :key="group.title" class="nav-group">
              <div class="nav-title">{{ group.title }}</div>
              <button v-for="item in group.items" :key="item.key" class="nav-item" :class="{ active: activeKey === item.key }" @click="selectNav(item.key)">
                <NIcon :size="18"><component :is="item.icon" /></NIcon><span>{{ item.label }}</span>
              </button>
            </div>
          </div>
          <button class="logout-btn" @click="logout"><NIcon :size="18"><LogOutOutline /></NIcon>登出</button>
        </NLayoutSider>
        <NLayout>
          <NLayoutHeader bordered class="topbar">
            <div><div class="breadcrumb">代理後台 <span>/</span> {{ pageTitle }}</div><h2>{{ pageTitle }}</h2></div>
            <div class="top-actions"><NTag type="success" size="small" round>系統運作中</NTag><NTag type="info" size="small" round>{{ identity.label }}</NTag><div class="top-avatar">{{ identity.label.slice(0, 1) }}</div></div>
          </NLayoutHeader>
          <main class="content-area">
            <section v-if="activeKey === 'dashboard'">
              <div class="welcome-row"><div><p class="eyebrow blue">{{ identity.label.toUpperCase() }} OVERVIEW</p><h1>早安，{{ identity.account }}</h1><p class="muted">{{ currentRole === '運營商' ? '查看全平台代理線、結算週期與傭金狀態。' : currentRole === '總代理' ? '查看所有代理線與直屬／全部下級的營運摘要。' : '查看你的直屬下級、玩家資料與可提領傭金。' }}</p></div><NTag type="info" round>資料更新：剛剛</NTag></div>
              <div class="stat-grid">
                <NCard><NStatistic label="本期可提領傭金" :value="28460" prefix="TWD " /></NCard>
                <NCard><NStatistic label="待結算傭金" :value="12680" prefix="TWD " /></NCard>
                <NCard><NStatistic :label="currentRole === '運營商' ? '平台總代理' : '下級代理'" :value="currentRole === '運營商' ? 8 : 16" suffix=" 位" /></NCard>
                <NCard><NStatistic label="下級玩家" :value="428" suffix=" 位" /></NCard>
              </div>
              <div class="dashboard-grid">
                <NCard title="傭金趨勢（近 7 期）"><div class="sparkline"><span v-for="(height, index) in [36, 48, 43, 70, 58, 82, 76]" :key="index" :style="{ height: `${height}%` }" /></div><div class="chart-labels"><span>週一</span><span>週二</span><span>週三</span><span>週四</span><span>週五</span><span>週六</span><span>今日</span></div></NCard>
                <NCard title="代理線摘要"><div class="summary-list"><div><span>{{ currentRole === '運營商' ? '啟用中的代理線' : '直屬代理' }}</span><strong>{{ currentRole === '運營商' ? '12 條' : '3 位' }}</strong></div><div><span>目前可見層級</span><strong>{{ currentRole === '運營商' ? '無限層' : '4 層' }}</strong></div><div><span>本期有效投注</span><strong>TWD 1,284,600</strong></div><div><span>本期產生傭金</span><strong class="positive">TWD 28,460</strong></div></div></NCard>
              </div>
              <NCard title="最近操作" class="recent-card"><div v-for="log in logs.slice(0, 3)" :key="`${log.time}-${log.detail}`" class="recent-row"><div class="log-dot" /><div><strong>{{ log.type }}</strong><span>{{ log.detail }}</span></div><time>{{ log.time }}</time></div></NCard>
            </section>

            <section v-else-if="activeKey === 'agents'">
              <div class="section-head"><div><h1>代理管理</h1><p class="muted">以代理為主體查看個人摘要、代理層級、代理線、幣別、傭金模式與套用方案；依角色提供對應操作。</p></div><NButton type="primary" @click="showCreateAgent = true">＋ {{ createTitle }}</NButton></div>
              <div class="role-banner"><strong>{{ identity.label }}可執行範圍</strong><span>{{ currentRole === '運營商' ? '建立總代理、設定傭金模式與結算週期，查看全平台代理網絡。' : currentRole === '總代理' ? '建立直屬代理、套用同一代理線傭金模式，查看全部下級與各代理線報表。' : '建立直屬下級代理、套用上級提供的傭金模式，僅查看自身代理線資料。' }}</span></div>
              <div class="agent-focus-grid"><NCard class="focus-card"><div class="focus-icon">A</div><div><span>目前登入角色</span><strong>{{ identity.account }}</strong><small>{{ identity.label }} · {{ identity.currency }} · 可查看{{ currentRole === '運營商' ? '全平台' : '自身代理線' }}</small></div></NCard><NCard class="focus-card"><div class="focus-icon blue">%</div><div><span>傭金方案</span><strong>三種模式可配置</strong><small>CPA／佔成／返佣分開管理，代理僅套用方案。</small></div></NCard><NCard class="focus-card"><div class="focus-icon green">↳</div><div><span>代理線狀態</span><strong>{{ currentRole === '運營商' ? '12 條 · 多幣別' : '3 條 · 幣別一致' }}</strong><small>同一條代理線內代理與玩家必須使用相同幣別。</small></div></NCard></div>
              <div class="table-section-label"><strong>篩選欄位</strong><span>設定查看範圍、代理層級、傭金模式與狀態</span></div><NCard class="filter-card"><div class="filter-row"><div class="scope-toggle"><button :class="{ active: scope === 'direct' }" @click="scope = 'direct'">只看直屬</button><button :class="{ active: scope === 'all' }" @click="scope = 'all'">查看全部下級</button></div><NSelect v-model:value="agentLevelFilter" clearable placeholder="代理層級" :options="[{label:'總代理',value:'總代理'},{label:'一級代理',value:'一級代理'},{label:'二級代理',value:'二級代理'}]" style="width: 140px" /><NSelect v-model:value="agentCommissionFilter" clearable placeholder="傭金模式" :options="[{label:'CPA',value:'CPA'},{label:'佔成',value:'佔成'},{label:'返佣',value:'返佣'}]" style="width: 130px" /><NSelect v-model:value="agentStatusFilter" clearable placeholder="狀態" :options="[{label:'啟用',value:'啟用'},{label:'停用',value:'停用'}]" style="width: 120px" /><NInput v-model:value="search" clearable placeholder="搜尋代理帳號" class="search-input"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div></NCard>
              <div class="table-section-label"><strong>資料顯示欄位</strong><span>代理帳號與層級分開顯示；樹狀路徑請進入詳情查看</span></div><NCard :bordered="false" class="table-card"><div class="agent-table-wrap"><div class="agent-table"><div class="agent-table-head"><span>代理帳號</span><span>層級</span><span>幣別</span><span>傭金模式</span><span>結算週期</span><span>下級數</span><span>狀態</span><span>操作</span></div><div v-for="row in filteredAgents" :key="row.id" class="agent-table-row"><button class="account-link" @click="openAgent(row)">{{ row.account }}</button><span>{{ row.level }}</span><span>{{ row.currency }}</span><span>{{ row.model ?? '沿用上級' }}</span><span>{{ row.cycle ?? '沿用上級' }}</span><span>{{ row.children }}</span><NTag size="small" :type="row.status === '啟用' ? 'success' : 'error'" round>{{ row.status }}</NTag><div class="table-actions"><NButton quaternary size="small" @click="openAgent(row)">查看詳情</NButton><NButton quaternary size="small" @click="deactivateAgent(row)">{{ row.status === '啟用' ? '停用' : '啟用' }}</NButton></div></div></div></div><div class="table-hint">完整樹狀路徑集中於代理詳情的「代理關係」；傭金規則集中於「傭金方案」管理。</div></NCard>
            </section>

            <section v-else-if="activeKey === 'players'">
              <div class="section-head"><div><h1>玩家管理</h1><p class="muted">玩家資料僅限查看，手機與信箱依隱私規則遮罩。</p></div><NTag type="info" round>唯讀資料</NTag></div>
              <NCard class="filter-card"><div class="filter-row"><div class="scope-toggle"><button :class="{ active: scope === 'direct' }" @click="scope = 'direct'">直屬玩家</button><button :class="{ active: scope === 'all' }" @click="scope = 'all'">全部下級玩家</button></div><NInput v-model:value="search" clearable placeholder="搜尋玩家帳號或路徑" class="search-input"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div></NCard>
              <NCard :bordered="false" class="table-card"><NDataTable :columns="playerColumns" :data="filteredPlayers" :pagination="{ pageSize: 8 }" /><div class="privacy-note">隱私提示：手機僅顯示前 2 碼與後 3 碼；信箱僅顯示前 2 個字元，其餘以 * 遮罩。</div></NCard>
            </section>

            <section v-else-if="activeKey === 'codes'">
              <div class="section-head"><div><h1>推廣碼</h1><p class="muted">分開管理直屬代理與直屬玩家推廣碼，避免傭金歸屬混淆。</p></div></div>
              <div class="code-grid"><NCard title="直屬代理推廣碼" class="code-card"><p class="muted">下線註冊成為代理後，可依代理線規則產生傭金。</p><div class="code-box">AGT-TW-8F4K-29Q1<NButton quaternary circle aria-label="複製直屬代理推廣碼" @click="copyReferral('AGT-TW-8F4K-29Q1', '直屬代理推廣碼')"><NIcon><CopyOutline /></NIcon></NButton></div><NTag type="success" round>可產生代理傭金</NTag></NCard><NCard title="直屬玩家推廣碼" class="code-card"><p class="muted">僅用於玩家註冊，不產生下線抽傭；註冊後不再建立代理推廣碼。</p><div class="code-box">PLY-TW-3M7X-62ZD<NButton quaternary circle aria-label="複製直屬玩家推廣碼" @click="copyReferral('PLY-TW-3M7X-62ZD', '直屬玩家推廣碼')"><NIcon><CopyOutline /></NIcon></NButton></div><NTag round>不產生代理傭金</NTag></NCard></div>
              <NCard title="推廣碼使用規則" class="rules-card"><div class="rule-grid"><div><span class="rule-num">01</span><p>一個帳號只能歸屬一條代理線，且幣別必須一致。</p></div><div><span class="rule-num">02</span><p>直屬代理推廣碼可建立無限層代理；直屬玩家碼僅建立玩家。</p></div><div><span class="rule-num">03</span><p>代理線轉移不回溯重算，生效前後訂單依原／新代理線歸屬。</p></div></div></NCard>
            </section>

            <section v-else-if="activeKey === 'commissionPlans'">
              <div class="section-head"><div><h1>傭金方案</h1><p class="muted">獨立管理 CPA、佔成、返佣三種傭金模式；代理建立或調整時套用方案，不在代理列表直接修改點數。</p><p class="line-rule">{{ currentRole === '運營商' ? '運營商可建立與編輯三種模式方案。' : `目前代理線由總代設定為${lineModel}，本線只能建立或套用${lineModel}方案，點數上限為${lineMaxRate}%。` }}</p></div><NButton v-if="canManagePlans" type="primary" @click="openPlanEditor()">＋ 新增傭金方案</NButton></div>
              <div class="plan-mode-grid"><NCard v-for="mode in [{name:'CPA',desc:'註冊、首存、流水達標各自設定固定獎勵。'},{name:'佔成',desc:'依玩家總輸贏扣除行政成本後按比例分配。'},{name:'返佣',desc:'依有效投注額按返佣比例計算。'}]" :key="mode.name" class="plan-mode-card"><strong>{{ mode.name }}</strong><span>{{ mode.desc }}</span></NCard></div>
              <div class="table-section-label"><strong>方案資料</strong><span>代理套用方案後，依方案的模式、週期與可分配點數計算</span></div><NCard :bordered="false" class="table-card"><div class="plan-table-wrap"><div class="plan-table"><div class="plan-table-head"><span>方案名稱</span><span>傭金模式</span><span>結算週期</span><span>可分配點數</span><span>套用代理</span><span>狀態</span><span>操作</span></div><div v-for="plan in visiblePlans" :key="plan.id" class="plan-table-row"><div><strong>{{ plan.name }}</strong><small>{{ plan.description }}</small></div><span>{{ plan.model }}</span><span>{{ plan.cycle }}</span><span>{{ plan.allocationRate }}%</span><span>{{ plan.assignedCount }} 位</span><NTag size="small" :type="plan.status === '啟用' ? 'success' : 'error'" round>{{ plan.status }}</NTag><div class="table-actions"><NButton v-if="canManagePlans" quaternary size="small" @click="openPlanEditor(plan)">編輯</NButton><NButton v-if="canManagePlans" quaternary size="small" @click="togglePlan(plan)">{{ plan.status === '啟用' ? '停用' : '啟用' }}</NButton><span v-if="!canManagePlans" class="muted">僅可套用</span></div></div></div></div><div class="table-hint">可分配點數代表本方案代理可設定給下一級的最高額度；同一條代理線只能使用一種傭金模式，下級方案不得超過上級額度。</div></NCard>
            </section>

            <section v-else-if="activeKey === 'commission'">
              <div class="section-head"><div><h1>傭金中心</h1><p class="muted">查看目前代理線的傭金模式、結算週期與餘額。</p></div><NTag type="info" round>本期：2026/08/25–08/31</NTag></div>
              <div class="commission-cards"><NCard><div class="mini-label">目前模式</div><div class="big-value">CPA <NTag type="info" size="small">總代設定</NTag></div><p class="muted">註冊、首存、流水達標各自獨立計算</p></NCard><NCard><div class="mini-label">結算週期</div><div class="big-value">{{ selectedCycle }}</div><p class="muted">變更於下一個結算週期生效</p></NCard><NCard><div class="mini-label">本期可提領</div><div class="big-value positive">TWD 28,460</div><p class="muted">已扣除追回與手續費</p></NCard></div>
              <NCard title="傭金明細" class="table-card"><NTabs type="line"><NTabPane name="all" tab="全部"><div class="commission-row" v-for="row in [{date:'08/31',type:'返佣',source:'north_team · 有效投注',amount:'+ TWD 3,240'},{date:'08/30',type:'CPA',source:'首存達標 × 4',amount:'+ TWD 1,200'},{date:'08/28',type:'追回',source:'異常訂單 P-20318',amount:'- TWD 400'}]" :key="row.date + row.source"><div><strong>{{ row.type }}</strong><span>{{ row.date }} · {{ row.source }}</span></div><strong :class="row.amount.startsWith('-') ? 'negative' : 'positive'">{{ row.amount }}</strong></div></NTabPane></NTabs></NCard>
            </section>

            <section v-else-if="activeKey === 'withdrawal'">
              <div class="section-head"><div><h1>提領傭金</h1><p class="muted">提領申請送出後由運營商審核，審核期間不會刪除原紀錄。</p></div><NButton type="primary" @click="showWithdrawal = true">申請提領</NButton></div>
              <div class="withdraw-grid"><NCard><NStatistic label="可提領傭金" :value="28460" prefix="TWD " /><NDivider /><div class="summary-list"><div><span>最低提領</span><strong>TWD 1,000</strong></div><div><span>手續費</span><strong>每筆 TWD 30</strong></div><div><span>每日次數上限</span><strong>3 次</strong></div></div></NCard><NCard title="申請紀錄"><div class="commission-row"><div><strong>審核中</strong><span>2026/08/27 09:43 · TWD 12,000</span></div><NTag type="warning" round>待審核</NTag></div><div class="commission-row"><div><strong>已完成</strong><span>2026/08/20 15:10 · TWD 8,000</span></div><NTag type="success" round>已撥款</NTag></div></NCard></div>
            </section>

            <section v-else-if="activeKey === 'reports'">
              <div class="section-head"><div><h1>下級報表</h1><p class="muted">以樹狀路徑與層級欄位呈現，避免無限層代理造成數據重複。</p></div><NButton secondary @click="exportMessage('下級報表')">匯出報表</NButton></div>
              <NCard class="filter-card"><div class="filter-row"><NSelect v-model:value="selectedCycle" :options="[{label:'本週',value:'本週'},{label:'本月',value:'本月'},{label:'上月',value:'上月'}]" style="width: 140px" /><div class="scope-toggle"><button :class="{ active: scope === 'direct' }" @click="scope = 'direct'">只看直屬</button><button :class="{ active: scope === 'all' }" @click="scope = 'all'">查看全部下級</button></div></div></NCard>
              <div class="stat-grid"><NCard><NStatistic label="總有效投注" :value="1284600" prefix="TWD " /></NCard><NCard><NStatistic label="總儲值" :value="342800" prefix="TWD " /></NCard><NCard><NStatistic label="產生傭金" :value="28460" prefix="TWD " /></NCard><NCard><NStatistic label="活躍玩家" :value="186" suffix=" 位" /></NCard></div>
              <NCard :bordered="false" class="table-card"><div class="report-header"><span>代理／玩家</span><span>層級</span><span>路徑</span><span>有效投注</span><span>傭金</span></div><div v-for="row in agentRows.slice(0, 4)" :key="row.id" class="report-row"><strong>{{ row.account }}</strong><span>{{ row.level }}</span><span class="path-text">{{ row.path }}</span><span>TWD {{ (row.point * 128460).toLocaleString() }}</span><strong class="positive">TWD {{ (row.point * 2846).toLocaleString() }}</strong></div></NCard>
            </section>

            <section v-else-if="activeKey === 'logs'">
              <div class="section-head"><div><h1>操作日誌</h1><p class="muted">完整保留登入、開設代理、調整反傭、提領等操作。</p></div><NButton secondary @click="exportMessage('操作紀錄')">匯出紀錄</NButton></div>
              <NCard class="filter-card"><div class="filter-row"><NSelect placeholder="操作類型" clearable :options="[{label:'登入',value:'登入'},{label:'開設代理',value:'開設代理'},{label:'設定反傭',value:'設定反傭'},{label:'提領申請',value:'提領申請'}]" style="width: 180px" /><NInput placeholder="搜尋操作內容或 IP" class="search-input"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div></NCard>
              <NCard :bordered="false" class="table-card"><div class="log-table-head"><span>時間</span><span>操作類型</span><span>操作人</span><span>內容</span><span>IP</span></div><div v-for="log in logs" :key="`${log.time}-${log.detail}`" class="log-table-row"><time>{{ log.time }}</time><NTag size="small" round :type="log.type === '登入' ? 'info' : log.type === '提領申請' ? 'warning' : 'default'">{{ log.type }}</NTag><span>{{ log.actor }}</span><span>{{ log.detail }}</span><code>{{ log.ip }}</code></div></NCard>
            </section>

            <section v-else-if="activeKey === 'profile'">
              <div class="section-head"><div><h1>帳戶設定</h1><p class="muted">管理目前登入代理的個人資料、安全設定與傭金收款銀行卡。</p></div><NTag type="info" round>{{ identity.label }}</NTag></div>
              <div class="profile-grid"><NCard title="個人資料"><div class="profile-list"><div><span>代理帳號</span><strong>{{ identity.account }}</strong></div><div><span>角色</span><strong>{{ identity.label }}</strong></div><div><span>所屬幣別</span><strong>{{ identity.currency }}</strong></div><div><span>手機</span><strong>09******123</strong></div><div><span>Email</span><strong>ka********@example.com</strong></div><div><span>登入密碼</span><strong>••••••••</strong><NButton size="small" quaternary>修改</NButton></div></div></NCard><NCard title="傭金收款銀行卡"><div class="bank-card"><div class="bank-brand">台新銀行</div><strong>**** **** 9012</strong><span>戶名：Klaus Lin</span><NTag type="success" round>已驗證</NTag></div><NButton type="primary" secondary @click="showBankCard = true">管理銀行卡</NButton><p class="modal-help">銀行卡僅用於傭金提領，提領時需選擇已驗證的收款帳戶。</p></NCard></div>
              <NCard title="角色使用說明" class="role-guide"><div class="role-guide-grid"><div><NTag type="info" round>運營商</NTag><p>建立總代理、設定傭金模式與結算週期，查看全平台代理網絡。</p></div><div><NTag type="success" round>總代理</NTag><p>建立直屬代理、設定反傭比例，查看全部下級報表。</p></div><div><NTag round>一般代理</NTag><p>管理自己的直屬下級，僅查看所屬代理線資料。</p></div></div></NCard>
            </section>
          </main>
        </NLayout>
      </NLayout>

      <NModal v-model:show="showAgentDetail" preset="card" :title="selectedAgent ? `代理詳情 · ${selectedAgent.account}` : '代理詳情'" class="modal-card">
        <template v-if="selectedAgent">
          <div class="detail-tabs"><button :class="{ active: detailTab === 'basic' }" @click="detailTab = 'basic'">基本資料</button><button :class="{ active: detailTab === 'relationship' }" @click="detailTab = 'relationship'">代理關係</button><button :class="{ active: detailTab === 'commission' }" @click="detailTab = 'commission'">傭金設定</button><button :class="{ active: detailTab === 'logs' }" @click="detailTab = 'logs'">操作紀錄</button></div>
          <div v-if="detailTab === 'basic'" class="agent-detail-grid">
            <div><span>登入帳號</span><strong>{{ selectedAgent.account }}</strong></div><div><span>登入密碼</span><strong>••••••••</strong><p class="modal-help">如需重設請聯繫平台客服。</p></div>
            <div><span>帳號類型</span><strong>{{ selectedAgent.level }}</strong><NTag size="small" round>不可修改</NTag></div><div><span>代理 UID（系統生成）</span><strong>{{ selectedAgent.uid || selectedAgent.id }}</strong></div>
            <div><span>推廣碼</span><strong>{{ selectedAgent.referralCode || '未設定' }}</strong></div><div><span>真實姓名</span><strong>{{ selectedAgent.displayName || '未填寫' }}</strong></div>
            <div><span>手機號碼</span><strong>{{ maskPhone(selectedAgent.phone) }}</strong></div><div><span>聯絡方式</span><strong>{{ selectedAgent.contactMethod || '未設定' }}</strong></div>
            <div><span>Email</span><strong>{{ maskEmail(selectedAgent.email) }}</strong></div><div><span>2FA 雙重驗證</span><NTag size="small" :type="selectedAgent.twoFactor === '已啟用' ? 'success' : 'default'" round>{{ selectedAgent.twoFactor || '未啟用' }}</NTag></div>
            <div><span>幣別</span><strong>{{ selectedAgent.currency }}</strong></div><div><span>目前狀態</span><NTag :type="selectedAgent.status === '啟用' ? 'success' : 'error'" round>{{ selectedAgent.status }}</NTag></div>
          </div>
          <div v-else-if="detailTab === 'relationship'" class="agent-detail-grid"><div class="full"><span>完整樹狀路徑</span><strong>{{ selectedAgent.path }}</strong></div><div><span>代理層級</span><strong>{{ selectedAgent.level }}</strong></div><div><span>直屬下級數</span><strong>{{ selectedAgent.children }} 位</strong></div><div class="full"><span>關係說明</span><p class="modal-help">此代理只能隸屬一條代理線；轉移代理線時，生效前後訂單依原／新代理線歸屬，不回溯重算歷史傭金。</p></div></div>
          <div v-else-if="detailTab === 'commission'" class="agent-detail-grid"><div class="full"><span>套用傭金方案</span><NSelect v-model:value="draftPlanId" :options="planOptions" /></div><div><span>傭金模式</span><strong>{{ selectedPlan.model }}</strong></div><div><span>結算週期</span><strong>{{ selectedPlan.cycle }}</strong></div><div><span>可分配點數</span><strong>{{ selectedPlan.allocationRate }}%</strong></div><div class="full"><span>方案規則</span><p class="modal-help">{{ selectedPlan.description }} 下級代理可設定的點數不得超過上級可分配額度。</p></div></div>
          <div v-else class="detail-log-list"><div v-for="log in logs.filter((item) => item.detail.includes(selectedAgent?.account ?? '')).slice(0, 5)" :key="`${log.time}-${log.detail}`" class="recent-row"><div class="log-dot" /><div><strong>{{ log.type }}</strong><span>{{ log.detail }}</span></div><time>{{ log.time }}</time></div><p v-if="!logs.some((item) => item.detail.includes(selectedAgent?.account ?? ''))" class="modal-help">目前沒有此代理的操作紀錄。</p></div>
        </template>
        <template #footer><NSpace justify="space-between" style="width: 100%"><NButton secondary @click="selectedAgent && deactivateAgent(selectedAgent)">{{ selectedAgent?.status === '啟用' ? '停用代理' : '啟用代理' }}</NButton><NSpace><NButton @click="showAgentDetail = false">關閉</NButton><NButton v-if="detailTab === 'commission'" type="primary" @click="saveAgentPlan">儲存傭金方案</NButton></NSpace></NSpace></template>
      </NModal>
      <NModal v-model:show="showCreateAgent" preset="card" :title="createTitle" class="modal-card"><p class="modal-intro">{{ currentRole === '運營商' ? '建立總代理時指定幣別與傭金方案；建立後可立即登入。' : '建立下級代理後立即啟用；代理只能隸屬一條代理線，傭金規則請套用既有方案。' }}</p><NForm label-placement="top"><div class="form-two-col"><NFormItem label="代理帳號"><NInput v-model:value="newAgentAccount" placeholder="輸入新代理帳號" /></NFormItem><NFormItem label="代理名稱"><NInput v-model:value="newAgentName" placeholder="輸入顯示名稱" /></NFormItem></div><div class="form-two-col"><NFormItem label="登入密碼"><NInput v-model:value="newAgentPassword" type="password" placeholder="設定初始登入密碼" /></NFormItem><NFormItem label="聯絡手機"><NInput v-model:value="newAgentPhone" placeholder="選填" /></NFormItem></div><NFormItem label="聯絡 Email"><NInput v-model:value="newAgentEmail" placeholder="選填" /></NFormItem><div class="form-two-col"><NFormItem label="套用傭金方案"><NSelect v-model:value="newAgentPlanId" :options="planOptions" /></NFormItem><NFormItem label="幣別"><NSelect v-model:value="newAgentCurrency" :options="[{label:'TWD 新台幣',value:'TWD'},{label:'USD 美元',value:'USD'},{label:'JPY 日圓',value:'JPY'}]" /></NFormItem></div></NForm><template #footer><NSpace justify="end"><NButton @click="showCreateAgent = false">取消</NButton><NButton type="primary" @click="createAgent">建立並啟用</NButton></NSpace></template></NModal>
      <NModal v-model:show="showPlanEditor" preset="card" :title="editingPlan ? '編輯傭金方案' : '新增傭金方案'" class="modal-card"><p class="modal-intro">方案定義傭金模式、結算週期與代理可分配點數；儲存後可在代理詳情套用。</p><NForm label-placement="top"><NFormItem label="方案名稱"><NInput v-model:value="planName" placeholder="例如：台灣返佣標準方案" /></NFormItem><div class="form-two-col"><NFormItem label="傭金模式"><NSelect v-model:value="planModel" :disabled="currentRole !== '運營商'" :options="currentRole === '運營商' ? [{label:'CPA',value:'CPA'},{label:'佔成',value:'佔成'},{label:'返佣',value:'返佣'}] : [{label: `${lineModel}（沿用上級）`, value: lineModel}]" /></NFormItem><NFormItem label="結算週期"><NSelect v-model:value="planCycle" :disabled="currentRole !== '運營商'" :options="[{label:'即時',value:'即時'},{label:'每日',value:'每日'},{label:'每週',value:'每週'},{label:'每月',value:'每月'}]" /></NFormItem></div><NFormItem label="可分配點數"><NInputNumber v-model:value="planRate" :min="0" :max="lineMaxRate" :step="0.5" style="width: 100%"><template #suffix>%</template></NInputNumber><p class="modal-help">{{ currentRole === '運營商' ? '運營商可依方案設定點數。' : `本代理線最多可設定 ${lineMaxRate}%；不得超過總代提供的額度。` }}</p></NFormItem><NFormItem label="方案說明"><NInput v-model:value="planDescription" type="textarea" :rows="3" placeholder="說明此方案的計算方式與下級點數規則" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showPlanEditor = false">取消</NButton><NButton type="primary" @click="savePlan">儲存方案</NButton></NSpace></template></NModal>
      <NModal v-model:show="showWithdrawal" preset="card" title="申請提領傭金" class="modal-card"><p class="modal-intro">申請將進入平台審核；目前可提領餘額 TWD 28,460。</p><NForm label-placement="top"><NFormItem label="提領金額"><NInputNumber v-model:value="withdrawalAmount" :min="1000" :max="28460" style="width: 100%"><template #prefix>TWD</template></NInputNumber></NFormItem><NFormItem label="收款帳戶"><NSelect value="bank-001" :options="[{label:'台新銀行 · ****1234',value:'bank-001'}]" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showWithdrawal = false">取消</NButton><NButton type="primary" @click="submitWithdrawal">送出申請</NButton></NSpace></template></NModal>
      <NModal v-model:show="showBankCard" preset="card" title="管理傭金收款銀行卡" class="modal-card"><p class="modal-intro">銀行卡資料會經過驗證，僅可用於傭金提領。</p><NForm label-placement="top"><NFormItem label="銀行名稱"><NInput v-model:value="bankName" placeholder="輸入銀行名稱" /></NFormItem><NFormItem label="帳號"><NInput v-model:value="bankAccount" placeholder="輸入收款帳號" /></NFormItem><NFormItem label="戶名"><NInput v-model:value="bankHolder" placeholder="輸入戶名" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showBankCard = false">取消</NButton><NButton type="primary" @click="saveBankCard">儲存並送驗證</NButton></NSpace></template></NModal>
    </div>
  </NConfigProvider>
</template>
