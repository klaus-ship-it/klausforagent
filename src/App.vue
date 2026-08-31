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
  NThing,
} from 'naive-ui'
import {
  BarChartOutline,
  CashOutline,
  ChevronDownOutline,
  CopyOutline,
  EyeOutline,
  GitNetworkOutline,
  GridOutline,
  ListOutline,
  LogOutOutline,
  PeopleOutline,
  SearchOutline,
  ShareSocialOutline,
  WalletOutline,
} from '@vicons/ionicons5'

type NavKey = 'dashboard' | 'agents' | 'players' | 'codes' | 'commission' | 'withdrawal' | 'reports' | 'logs'
type Scope = 'direct' | 'all'

interface AgentRow {
  id: string
  account: string
  level: string
  path: string
  currency: string
  point: number
  children: number
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
const scope = ref<Scope>('all')
const search = ref('')
const showCreateAgent = ref(false)
const newAgentAccount = ref('')
const newAgentPoint = ref<number | null>(0)
const newAgentCurrency = ref('TWD')
const showAgentDetail = ref(false)
const selectedAgent = ref<AgentRow | null>(null)
const draftPoint = ref<number | null>(null)
const showWithdrawal = ref(false)
const withdrawalAmount = ref<number | null>(null)
const selectedCycle = ref('每週')
const notice = ref<{ type: 'success' | 'warning'; title: string; content: string } | null>(null)

const agentRows = ref<AgentRow[]>([
  { id: 'A-1001', account: 'agent_taipei', level: '總代理', path: 'agent_taipei', currency: 'TWD', point: 6, children: 3, status: '啟用' },
  { id: 'A-1002', account: 'north_team', level: '一級代理', path: 'agent_taipei > north_team', currency: 'TWD', point: 4, children: 8, status: '啟用' },
  { id: 'A-1003', account: 'east_team', level: '一級代理', path: 'agent_taipei > east_team', currency: 'TWD', point: 3, children: 5, status: '啟用' },
  { id: 'A-1004', account: 'sub_partner_01', level: '二級代理', path: 'agent_taipei > north_team > sub_partner_01', currency: 'TWD', point: 2, children: 2, status: '啟用' },
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

const navGroups = [
  { title: '工作台', items: [{ key: 'dashboard', label: '營運概覽', icon: GridOutline }] },
  { title: '下級管理', items: [{ key: 'agents', label: '代理管理', icon: GitNetworkOutline }, { key: 'players', label: '玩家管理', icon: PeopleOutline }, { key: 'codes', label: '推廣碼', icon: ShareSocialOutline }] },
  { title: '傭金與報表', items: [{ key: 'commission', label: '傭金中心', icon: CashOutline }, { key: 'withdrawal', label: '提領傭金', icon: WalletOutline }, { key: 'reports', label: '下級報表', icon: BarChartOutline }] },
  { title: '系統', items: [{ key: 'logs', label: '操作日誌', icon: ListOutline }] },
] as const

const pageTitle = computed(() => navGroups.flatMap((group) => group.items).find((item) => item.key === activeKey.value)?.label ?? '營運概覽')
const filteredAgents = computed(() => agentRows.value.filter((row) => {
  const inScope = scope.value === 'all' || row.level === '一級代理'
  return inScope && `${row.account}${row.path}`.toLowerCase().includes(search.value.toLowerCase())
}))
const filteredPlayers = computed(() => playerRows.value.filter((row) => {
  const inScope = scope.value === 'all' || row.level === '一級玩家'
  return inScope && `${row.account}${row.path}`.toLowerCase().includes(search.value.toLowerCase())
}))

function login() {
  if (!loginAccount.value || !loginPassword.value) {
    showNotice('warning', '請輸入登入資訊', '此為原型示範，輸入任意非空白帳密即可登入。')
    return
  }
  loggedIn.value = true
  logs.value.unshift({ time: '2026-08-31 10:28:00', type: '登入', actor: loginAccount.value, detail: '代理後台登入成功', ip: '10.20.8.15' })
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
  const next = agentRows.value.length + 1
  const account = newAgentAccount.value.trim()
  agentRows.value.push({ id: `A-${1000 + next}`, account, level: '一級代理', path: `agent_taipei > ${account}`, currency: newAgentCurrency.value, point: newAgentPoint.value ?? 0, children: 0, status: '啟用' })
  showCreateAgent.value = false
  newAgentAccount.value = ''
  newAgentPoint.value = 0
  logs.value.unshift({ time: '2026-08-31 10:31:00', type: '開設代理', actor: loginAccount.value, detail: '建立新的一級代理（立即啟用）', ip: '10.20.8.15' })
  showNotice('success', '代理已建立', '新代理可立即登入使用；反傭比例預設為 0%。')
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

function openAgent(row: AgentRow) {
  selectedAgent.value = row
  draftPoint.value = row.point
  showAgentDetail.value = true
}

function agentRowProps(row: AgentRow) {
  return { style: 'cursor: pointer', onClick: () => openAgent(row) }
}

function saveAgentPoint() {
  if (!selectedAgent.value || draftPoint.value === null) return
  const oldPoint = selectedAgent.value.point
  selectedAgent.value.point = draftPoint.value
  logs.value.unshift({ time: '2026-08-31 10:34:00', type: '設定反傭', actor: loginAccount.value, detail: `${selectedAgent.value.account} 反傭點數 ${oldPoint}% → ${draftPoint.value}%`, ip: '10.20.8.15' })
  showAgentDetail.value = false
  showNotice('success', '反傭點數已更新', `${selectedAgent.value.account} 的可分配反傭上限為 ${draftPoint.value}%。`)
}

async function copyReferral(code: string, label: string) {
  await navigator.clipboard?.writeText(code)
  showNotice('success', `${label}已複製`, code)
}

function exportMessage(label: string) {
  showNotice('success', `${label}已準備`, '原型示範不會下載真實資料；正式版將依目前篩選條件匯出。')
}

const agentColumns = [
  { title: '代理帳號', key: 'account' },
  { title: '層級', key: 'level' },
  { title: '樹狀路徑', key: 'path' },
  { title: '幣別', key: 'currency' },
  { title: '可分配反傭點數', key: 'point', render: (row: AgentRow) => `${row.point}%` },
  { title: '下級數', key: 'children' },
  { title: '狀態', key: 'status', render: (row: AgentRow) => row.status },
]
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
          <div class="agent-chip"><div class="avatar">A</div><div><strong>agent_demo</strong><span>總代理 · TWD</span></div><ChevronDownOutline class="chip-chevron" /></div>
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
            <div class="top-actions"><NTag type="success" size="small" round>系統運作中</NTag><div class="top-avatar">A</div></div>
          </NLayoutHeader>
          <main class="content-area">
            <section v-if="activeKey === 'dashboard'">
              <div class="welcome-row"><div><p class="eyebrow blue">OVERVIEW</p><h1>早安，agent_demo</h1><p class="muted">這是你目前可查看的代理線營運摘要。</p></div><NTag type="info" round>資料更新：剛剛</NTag></div>
              <div class="stat-grid">
                <NCard><NStatistic label="本期可提領傭金" :value="28460" prefix="TWD " /></NCard>
                <NCard><NStatistic label="待結算傭金" :value="12680" prefix="TWD " /></NCard>
                <NCard><NStatistic label="下級代理" :value="16" suffix=" 位" /></NCard>
                <NCard><NStatistic label="下級玩家" :value="428" suffix=" 位" /></NCard>
              </div>
              <div class="dashboard-grid">
                <NCard title="傭金趨勢（近 7 期）"><div class="sparkline"><span v-for="(height, index) in [36, 48, 43, 70, 58, 82, 76]" :key="index" :style="{ height: `${height}%` }" /></div><div class="chart-labels"><span>週一</span><span>週二</span><span>週三</span><span>週四</span><span>週五</span><span>週六</span><span>今日</span></div></NCard>
                <NCard title="代理線摘要"><div class="summary-list"><div><span>總代直屬代理</span><strong>3 位</strong></div><div><span>全部代理層級</span><strong>4 層</strong></div><div><span>本期有效投注</span><strong>TWD 1,284,600</strong></div><div><span>本期產生傭金</span><strong class="positive">TWD 28,460</strong></div></div></NCard>
              </div>
              <NCard title="最近操作" class="recent-card"><div v-for="log in logs.slice(0, 3)" :key="`${log.time}-${log.detail}`" class="recent-row"><div class="log-dot" /><div><strong>{{ log.type }}</strong><span>{{ log.detail }}</span></div><time>{{ log.time }}</time></div></NCard>
            </section>

            <section v-else-if="activeKey === 'agents'">
              <div class="section-head"><div><h1>代理管理</h1><p class="muted">以代理為主體查看層級、代理線、幣別與可分配反傭點數；代理可立即開設下級。</p></div><NButton type="primary" @click="showCreateAgent = true">＋ 開設下級代理</NButton></div>
              <div class="agent-focus-grid"><NCard class="focus-card"><div class="focus-icon">A</div><div><span>目前登入代理</span><strong>agent_demo</strong><small>總代理 · TWD · 可管理 4 層代理線</small></div></NCard><NCard class="focus-card"><div class="focus-icon blue">%</div><div><span>反傭點數說明</span><strong>可分配反傭點數</strong><small>代表此代理可分配給下級的比例上限；不可超過上級可分配額度。</small></div></NCard><NCard class="focus-card"><div class="focus-icon green">↳</div><div><span>代理線狀態</span><strong>3 條 · 幣別一致</strong><small>同一條代理線內代理與玩家必須使用相同幣別。</small></div></NCard></div>
              <NCard class="filter-card"><div class="filter-row"><div class="scope-toggle"><button :class="{ active: scope === 'direct' }" @click="scope = 'direct'">只看直屬</button><button :class="{ active: scope === 'all' }" @click="scope = 'all'">查看全部下級</button></div><NInput v-model:value="search" clearable placeholder="搜尋帳號或樹狀路徑" class="search-input"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div></NCard>
              <NCard :bordered="false" class="table-card"><NDataTable :columns="agentColumns" :data="filteredAgents" :pagination="{ pageSize: 8 }" :row-props="agentRowProps" /><NDivider /><div class="table-hint">點擊任一代理列可查看完整代理資訊與調整反傭點數</div><div v-for="row in filteredAgents" :key="row.id" class="action-line"><NThing :title="row.account" :description="`${row.level} · ${row.path}`"><template #avatar><div class="row-avatar agent">A</div></template></NThing><NTag :type="row.status === '啟用' ? 'success' : 'default'" round>{{ row.status }}</NTag><NSpace><NButton quaternary size="small" @click="openAgent(row)"><NIcon><EyeOutline /></NIcon> 查看代理</NButton><NButton quaternary size="small" @click="deactivateAgent(row)">{{ row.status === '啟用' ? '停用' : '啟用' }}</NButton></NSpace></div></NCard>
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
          </main>
        </NLayout>
      </NLayout>

      <NModal v-model:show="showAgentDetail" preset="card" :title="selectedAgent ? `代理資訊 · ${selectedAgent.account}` : '代理資訊'" class="modal-card"><template v-if="selectedAgent"><div class="agent-detail-grid"><div><span>代理帳號</span><strong>{{ selectedAgent.account }}</strong></div><div><span>代理層級</span><strong>{{ selectedAgent.level }}</strong></div><div><span>幣別</span><strong>{{ selectedAgent.currency }}</strong></div><div><span>目前狀態</span><NTag :type="selectedAgent.status === '啟用' ? 'success' : 'default'" round>{{ selectedAgent.status }}</NTag></div><div class="full"><span>樹狀路徑</span><strong>{{ selectedAgent.path }}</strong></div><div class="full"><span>可分配反傭點數</span><p class="modal-help">此值是本代理可設定給下一層的比例上限，不能高於上級可分配額度。</p><NInputNumber v-model:value="draftPoint" :min="0" :max="6" :step="0.5" style="width: 180px"><template #suffix>%</template></NInputNumber></div></div></template><template #footer><NSpace justify="space-between" style="width: 100%"><NButton secondary @click="selectedAgent && deactivateAgent(selectedAgent)">{{ selectedAgent?.status === '啟用' ? '停用代理' : '啟用代理' }}</NButton><NSpace><NButton @click="showAgentDetail = false">取消</NButton><NButton type="primary" @click="saveAgentPoint">儲存反傭點數</NButton></NSpace></NSpace></template></NModal>
      <NModal v-model:show="showCreateAgent" preset="card" title="開設下級代理" class="modal-card"><p class="modal-intro">新代理建立後立即啟用；帳號與其他不可變更欄位請由平台客服處理。</p><NForm label-placement="top"><NFormItem label="代理帳號"><NInput v-model:value="newAgentAccount" placeholder="輸入新代理帳號" /></NFormItem><NFormItem label="代理反傭比例"><NInputNumber v-model:value="newAgentPoint" :min="0" :max="6" :step="0.5" style="width: 100%"><template #suffix>%</template></NInputNumber></NFormItem><NFormItem label="幣別"><NSelect v-model:value="newAgentCurrency" :options="[{label:'TWD 新台幣',value:'TWD'}]" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showCreateAgent = false">取消</NButton><NButton type="primary" @click="createAgent">建立並啟用</NButton></NSpace></template></NModal>
      <NModal v-model:show="showWithdrawal" preset="card" title="申請提領傭金" class="modal-card"><p class="modal-intro">申請將進入平台審核；目前可提領餘額 TWD 28,460。</p><NForm label-placement="top"><NFormItem label="提領金額"><NInputNumber v-model:value="withdrawalAmount" :min="1000" :max="28460" style="width: 100%"><template #prefix>TWD</template></NInputNumber></NFormItem><NFormItem label="收款帳戶"><NSelect value="bank-001" :options="[{label:'台新銀行 · ****1234',value:'bank-001'}]" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showWithdrawal = false">取消</NButton><NButton type="primary" @click="submitWithdrawal">送出申請</NButton></NSpace></template></NModal>
    </div>
  </NConfigProvider>
</template>
