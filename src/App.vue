<script setup lang="ts">
import { computed, ref } from 'vue'
import {
  NButton,
  NBadge,
  NCard,
  NConfigProvider,
  NDataTable,
  NDatePicker,
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
  NRadio,
  NRadioGroup,
  NSpace,
  NStatistic,
  NSwitch,
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
// 代理後台對外統一稱為「代理」；層級只用一級代理、二級代理……N級代理表示。
// 舊角色字面值保留僅供既有資料相容，介面不再顯示或提供角色切換。
type Role = '運營商' | '總代理' | '一般代理' | '代理'

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
  twoFactorBoundAt?: string
  twoFactorLastResetAt?: string
  twoFactorRequired?: boolean
  level: string
  path: string
  currency: string
  point: number
  children: number
  walletBalance: number
  withdrawalEnabled?: boolean
  registerAt?: string
  registerIp?: string
  lastLoginAt?: string
  lastLoginIp?: string
  directAgentCount?: number
  directPlayerCount?: number
  totalAgentCount?: number
  totalPlayerCount?: number
  totalDeposit?: number
  totalEffectiveBet?: number
  pendingTransferTargetPath?: string
  pendingTransferEffectiveAt?: string
  lastTransferAt?: string
  status: '啟用' | '停用'
  planId: string
  model?: '儲值' | '有效投注額' | '輸贏' | '佔成' | '返佣'
  cycle?: '即時' | '每日' | '每週' | '每月'
}

interface CommissionPlan {
  id: string
  name: string
  createdBy?: string
  parentAccount: string
  model: '儲值' | '有效投注額' | '輸贏'
  cycle: '即時' | '每日' | '每週' | '每月'
  allocationRate: number
  description: string
  assignedCount: number
  status: '啟用' | '停用'
  config?: CommissionConfig
}

interface CommissionConfig {
  share: { shareRate: number; adminCostRate: number; negativeMode: '負數沖銷' | '負數累計'; offsetType: '全額沖銷' | '週期固定沖銷額度'; offsetLimit: number }
  rebate: { rebateRate: number }
}

interface PlayerRow {
  id: string
  uid: string
  account: string
  displayName: string
  tags: string[]
  rtp: number
  vipLevel: string
  agentLevel: string
  referralCode?: string
  status: '啟用' | '鎖定' | '凍結' | '暫停'
  isOnline: boolean
  registerAt: string
  phone: string
  email: string
  gender: string
  birthday: string
  registerIp: string
  lastLoginAt: string
  lastLoginIp: string
  registerSource: string
  inviteCode: string
  consecutiveCheckInDays: number
  path: string
  currency: string
  deposit: number
  bet: number
}

interface OperationLog {
  time: string
  type: string
  actor: string
  detail: string
  ip: string
  target?: string
  before?: string
  after?: string
  effectiveAt?: string
}

interface CommissionWithdrawalRecord {
  id: string
  recordType: '傭金發放' | '傭金提領'
  applyAt: string
  settlePeriod: string
  amount: number
  fee: number
  netAmount: number
  balanceBefore: number
  balanceAfter: number
  status: '待處理' | '處理中' | '成功' | '失敗'
  bank: string
  reviewAt: string
  reviewer: string
  remark: string
  bankAccount?: string
  bankHolder?: string
  plan?: string
  settlementBase?: string
  systemResult?: string
}

type CommissionOrderStatus = '待處理' | '處理中' | '成功' | '失敗'

interface CommissionWithdrawalOrder {
  id: string
  account: string
  currency: string
  amount: number
  status: CommissionOrderStatus
  createdAt: string
  updatedAt: string
  bankName: string
  bankAccount: string
  bankHolder: string
  failureReason?: string
  processor?: string
  remark?: string
}

interface CommissionPayoutRecord {
  id: string
  account: string
  currency: string
  amount: number
  status: CommissionOrderStatus
  createdAt: string
  updatedAt: string
  plan: string
  cycle: string
  settlementBase: string
  systemResult: string
  remark: string
}

const themeOverrides = {
  common: { primaryColor: '#2b63bf', primaryColorHover: '#2455a3', borderRadius: '9px' },
  Button: { borderRadiusMedium: '9px' },
}
const loggedIn = ref(false)
const loginAccount = ref('agent_demo')
const loginPassword = ref('demo123')
const loginRole = ref<Role>('代理')
const activeKey = ref<NavKey>('dashboard')
const currentRole = ref<Role>('代理')
const scope = ref<Scope>('all')
const search = ref('')
const agentAccountSearch = ref('')
const agentAccountDraft = ref('')
const agentRegisterIp = ref('')
const agentRegisterDateRange = ref<[number, number] | null>(null)
const agentParentFilter = ref<string | null>(null)
const agentScope = ref<Scope>('all')
const agentScopeDraft = ref<Scope>('all')
const agentLevelFilter = ref<string | null>(null)
const agentStatusFilter = ref<string | null>(null)
const agentCommissionFilter = ref<'儲值' | '有效投注額' | '輸贏' | null>(null)
const agentLevelDraft = ref<string | null>(null)
const agentStatusDraft = ref<string | null>(null)
const agentCommissionDraft = ref<'儲值' | '有效投注額' | '輸贏' | null>(null)
const agentParentDraft = ref<string | null>(null)
const agentRegisterIpDraft = ref('')
const agentRegisterDateRangeDraft = ref<[number, number] | null>(null)
const showCreateAgent = ref(false)
const newAgentAccount = ref('')
const newAgentPoint = ref<number | null>(0)
const newAgentCurrency = ref('TWD')
const newAgentName = ref('')
const newAgentPhone = ref('')
const newAgentEmail = ref('')
const newAgentPassword = ref('')
const showAgentDetail = ref(false)
const showAgentDetailModal = ref(false)
const selectedAgent = ref<AgentRow | null>(null)
const showDeactivateAgentConfirm = ref(false)
const pendingDeactivateAgent = ref<AgentRow | null>(null)
const showAgentEdit = ref(false)
const agentNameDraft = ref('')
const agentPhoneDraft = ref('')
const agentEmailDraft = ref('')
const agentPasswordDraft = ref('')
const detailTab = ref<'basic' | 'wallet' | 'auth' | 'relationship' | 'commission' | 'withdrawals' | 'logs'>('basic')
const relationshipExpanded = ref<Record<string, boolean>>({})
const showTwoFactorAdmin = ref(false)
const selectedTwoFactorAgent = ref<AgentRow | null>(null)
const twoFactorGlobalEnabled = ref(true)
const showTwoFactorQr = ref(false)
const showTwoFactorResetConfirm = ref(false)
const showTwoFactorToggleConfirm = ref(false)
const pendingTwoFactorRequired = ref<boolean | null>(null)
const showLoginTwoFactorSetup = ref(false)
const showLoginTwoFactorVerify = ref(false)
const loginTwoFactorAccount = ref('')
const loginTwoFactorCode = ref('')
const showPlayerDetail = ref(false)
const showPlayerDetailModal = ref(false)
const selectedPlayer = ref<PlayerRow | null>(null)
const playerSearch = ref('')
const playerSearchType = ref<'id' | 'account' | 'phone'>('id')
const playerAffiliationType = ref<'invite' | 'promo'>('invite')
const playerAffiliationQuery = ref('')
const playerAdvancedSearch = ref(false)
const playerTagsFilter = ref<string[]>([])
const playerRegisterIp = ref('')
const playerRegisterDateRange = ref<[number, number] | null>(null)
const playerParentFilter = ref<string | null>(null)
const playerScope = ref<Scope>('all')
const playerSearchDraft = ref('')
const playerRegisterIpDraft = ref('')
const playerRegisterDateRangeDraft = ref<[number, number] | null>(null)
const playerParentDraft = ref<string | null>(null)
const playerScopeDraft = ref<Scope>('all')
const playerStatusFilter = ref<PlayerRow['status'] | null>(null)
const playerStatusFilterDraft = ref<PlayerRow['status'] | null>(null)
const playerDetailTab = ref<'basic' | 'wallet' | 'vip' | 'promotion' | 'audit' | 'asset' | 'game' | 'transfer' | 'invite' | 'agent'>('basic')
const showPlayerEdit = ref(false)
const showPlayerStatus = ref(false)
const showPlayerTransfer = ref(false)
const playerDisplayNameDraft = ref('')
const playerStatusDraft = ref<PlayerRow['status']>('啟用')
const playerTransferTarget = ref('')
const draftPlanId = ref('PLAN-001')
const newAgentPlanId = ref('PLAN-001')
const showPlanEditor = ref(false)
const editingPlan = ref<CommissionPlan | null>(null)
const planName = ref('')
const planParentAccount = ref('')
const planModel = ref<'儲值' | '有效投注額' | '輸贏'>('有效投注額')
const planCycle = ref<'即時' | '每日' | '每週' | '每月'>('每週')
const planRate = ref<number | null>(4)
const planDescription = ref('')
const shareRate = ref<number | null>(5)
const adminCostRate = ref<number | null>(0)
const negativeMode = ref<'負數沖銷' | '負數累計'>('負數累計')
const offsetType = ref<'全額沖銷' | '週期固定沖銷額度'>('全額沖銷')
const offsetLimit = ref<number | null>(0)
const rebateRate = ref<number | null>(4)
const showWithdrawal = ref(false)
const showBankCard = ref(false)
const bankName = ref('台新銀行')
const bankAccount = ref('123456789012')
const bankHolder = ref('Klaus Lin')
const withdrawalAmount = ref<number | null>(null)
const withdrawalMin = ref(1000)
const withdrawalDailyLimit = ref(3)
const withdrawalTodayCount = ref(1)
const withdrawableBalance = ref(28460)
const showWithdrawalStatusConfirm = ref(false)
const pendingWithdrawalAgent = ref<AgentRow | null>(null)
const pendingWithdrawalEnabled = ref<boolean | null>(null)
const showWalletAdjustment = ref(false)
const showWalletAdjustmentConfirm = ref(false)
const walletAdjustmentType = ref<'加款' | '扣款'>('加款')
const walletAdjustmentAmount = ref<number | null>(null)
const walletAdjustmentReason = ref('')
const showAgentTransfer = ref(false)
const showAgentTransferConfirm = ref(false)
const agentTransferTarget = ref('')
const pendingAgentTransfer = ref<AgentRow | null>(null)
const pendingAgentTransferTarget = ref<AgentRow | null>(null)
const logTypeFilter = ref<string | null>(null)
const logSearch = ref('')
const logDateRange = ref<[number, number] | null>(null)
const agentDetailLogType = ref<string | null>(null)
const agentDetailLogSearch = ref('')
const agentDetailLogDateRange = ref<[number, number] | null>(null)
const agentWithdrawalKeyword = ref('')
const agentWithdrawalStatus = ref<CommissionWithdrawalRecord['status'] | null>(null)
const agentWithdrawalType = ref<CommissionWithdrawalRecord['recordType'] | null>(null)
const agentWithdrawalDateRange = ref<[number, number] | null>(null)
const showCommissionRecordDetail = ref(false)
const selectedCommissionRecord = ref<CommissionWithdrawalRecord | null>(null)
const withdrawalOrders = ref<CommissionWithdrawalOrder[]>([
  { id: 'WD-20260902-001', account: 'agent_taipei', currency: 'TWD', amount: 12000, status: '待處理', createdAt: '2026-09-02 11:26:08', updatedAt: '2026-09-02 11:26:08', bankName: '台新銀行', bankAccount: '****9012', bankHolder: 'Klaus Lin' },
  { id: 'WD-20260901-003', account: 'north_team', currency: 'TWD', amount: 6800, status: '處理中', createdAt: '2026-09-01 15:09:22', updatedAt: '2026-09-01 15:18:40', bankName: '國泰世華', bankAccount: '****6621', bankHolder: 'North Chen', processor: 'operator_demo' },
  { id: 'WD-20260830-009', account: 'north_l2', currency: 'TWD', amount: 3600, status: '處理中', createdAt: '2026-08-30 16:42:18', updatedAt: '2026-08-30 16:50:02', bankName: '玉山銀行', bankAccount: '****7733', bankHolder: 'North L2', processor: 'operator_finance' },
  { id: 'WD-20260825-004', account: 'sub_partner_01', currency: 'TWD', amount: 8600, status: '成功', createdAt: '2026-08-25 09:14:32', updatedAt: '2026-08-25 14:08:21', bankName: '台新銀行', bankAccount: '****9012', bankHolder: 'Partner Lin', processor: 'operator_demo' },
  { id: 'WD-20260728-006', account: 'east_team', currency: 'TWD', amount: 4200, status: '失敗', createdAt: '2026-07-28 13:08:55', updatedAt: '2026-07-29 09:18:06', bankName: '台新銀行', bankAccount: '****4488', bankHolder: 'East Wang', processor: 'operator_demo', failureReason: '收款帳戶驗證未完成' },
])
const payoutRecords = ref<CommissionPayoutRecord[]>([
  { id: 'COM-20260901-001', account: 'agent_taipei', currency: 'TWD', amount: 28460, status: '成功', createdAt: '2026-09-01 00:00:00', updatedAt: '2026-09-01 00:00:03', plan: '台灣返佣標準方案', cycle: '2026-W35（08/25–08/31）', settlementBase: '有效投注總額 TWD 474,333', systemResult: '成功：已發放至傭金錢包', remark: '系統自動結算完成' },
  { id: 'COM-20260825-004', account: 'north_team', currency: 'TWD', amount: 18600, status: '處理中', createdAt: '2026-08-25 00:00:00', updatedAt: '2026-08-25 00:00:05', plan: '台灣返佣標準方案', cycle: '2026-W34（08/18–08/24）', settlementBase: '有效投注總額 TWD 310,000', systemResult: '待處理：代理停用，系統未能自動發放', remark: '待運營商手動確認' },
  { id: 'COM-20260811-002', account: 'sub_partner_01', currency: 'TWD', amount: 15400, status: '成功', createdAt: '2026-08-11 00:00:00', updatedAt: '2026-08-11 00:00:04', plan: '台灣返佣標準方案', cycle: '2026-W32（08/04–08/10）', settlementBase: '有效投注總額 TWD 256,667', systemResult: '成功：已發放至傭金錢包', remark: '系統自動結算完成' },
  { id: 'COM-20260728-006', account: 'east_team', currency: 'TWD', amount: 9200, status: '失敗', createdAt: '2026-07-28 00:00:00', updatedAt: '2026-07-28 00:00:06', plan: '佔成合作方案', cycle: '2026-07（07/01–07/27）', settlementBase: '輸贏總額 TWD -184,000', systemResult: '失敗：結算資料異常', remark: '運營商人工確認後維持失敗' },
])
const withdrawalReportKeyword = ref('')
const withdrawalReportStatus = ref<CommissionOrderStatus | null>(null)
const withdrawalReportDateRange = ref<[number, number] | null>(null)
const payoutReportKeyword = ref('')
const payoutReportStatus = ref<CommissionOrderStatus | null>(null)
const payoutReportDateRange = ref<[number, number] | null>(null)
const showWithdrawalProcess = ref(false)
const selectedWithdrawalOrder = ref<CommissionWithdrawalOrder | null>(null)
const withdrawalProcessAction = ref<'成功' | '失敗'>('成功')
const withdrawalFailureReason = ref('')
const showPayoutProcess = ref(false)
const selectedPayoutRecord = ref<CommissionPayoutRecord | null>(null)
const payoutProcessAction = ref<'成功' | '失敗'>('成功')
const payoutFailureReason = ref('')
const showCommissionActionConfirm = ref(false)
const pendingCommissionAction = ref<'withdrawal' | 'payout' | 'withdrawal-start' | null>(null)
const pendingCommissionResult = ref<'成功' | '失敗' | '處理中'>('成功')
const commissionActionRemark = ref('')
const playerPromotionRecords = computed(() => [
  { time: '2026-08-29 13:42:10', name: '首存回饋', amount: 800, status: '已完成', detail: '流水要求 8 倍，已完成 6.4 倍' },
  { time: '2026-08-18 20:16:44', name: 'VIP3 升級禮', amount: 1200, status: '已完成', detail: 'VIP3 升級獎勵' },
  { time: '2026-08-10 09:08:22', name: '週末返水', amount: 360, status: '待領取', detail: '可於 2026-09-05 前領取' },
])
const playerAuditRecords = computed(() => [
  { time: '2026-08-31 10:22:04', type: '登入', actor: 'player_klaus', detail: '登入成功', ip: '10.20.8.45' },
  { time: '2026-08-30 00:00:12', type: 'VIP 等級更新', actor: 'system', detail: 'VIP2 → VIP3', ip: '-' },
  { time: '2026-08-29 13:42:10', type: '領取優惠', actor: 'player_klaus', detail: '首存回饋 TWD 800', ip: '10.20.8.45' },
  { time: '2026-08-27 09:11:08', type: '安全設定', actor: 'player_klaus', detail: 'Google Auth 綁定成功', ip: '10.20.8.45' },
])
const playerAssetRecords = computed(() => [
  { time: '2026-08-31 10:22:04', type: '儲值', amount: 500, balance: 17840, status: '成功', detail: '信用卡儲值' },
  { time: '2026-08-28 16:18:42', type: '提領', amount: -4200, balance: 17340, status: '審核中', detail: '台新銀行 · ****1234' },
  { time: '2026-08-25 09:20:18', type: '儲值', amount: 3000, balance: 21540, status: '成功', detail: '超商代碼' },
])
const playerGameRecords = computed(() => [
  { time: '2026-08-31 10:18:42', game: 'Sport · 足球', rounds: 24, bet: 4200, result: '+680', status: '結算' },
  { time: '2026-08-30 22:04:16', game: 'Live Casino · 百家樂', rounds: 18, bet: 3600, result: '-420', status: '結算' },
  { time: '2026-08-29 19:40:33', game: 'Slot · Fortune Tiger', rounds: 42, bet: 2800, result: '+1250', status: '結算' },
])
const playerTransferRecords = computed(() => [
  { time: '2026-08-01 00:00:00', from: '平台 > agent_taipei > north_team', to: '平台 > agent_taipei > north_team > sub_partner_01', actor: 'operator_demo', status: '已生效' },
  { time: '2026-07-18 14:22:08', from: '平台 > agent_taipei', to: '平台 > agent_taipei > north_team', actor: 'operator_demo', status: '已生效' },
])
const selectedCycle = ref('每週')
const notice = ref<{ type: 'success' | 'warning'; title: string; content: string } | null>(null)

function toggleRelationshipSection(key: string) {
  const groups: Record<string, string[]> = {
    totalAgents: ['totalAgents', 'totalAgentMoney', 'totalAgentBet'],
    directAgents: ['directAgents', 'directAgentMoney', 'directAgentBet'],
    totalPlayers: ['totalPlayers', 'totalPlayerMoney', 'totalPlayerBet'],
    directPlayers: ['directPlayers', 'directPlayerMoney', 'directPlayerBet'],
  }
  const groupKey = Object.keys(groups).find((name) => groups[name].includes(key)) || key
  const next = !relationshipExpanded.value[groupKey]
  ;(groups[groupKey] || [key]).forEach((item) => { relationshipExpanded.value[item] = next })
}

function relationshipRows(type: 'agents' | 'players', scope: 'all' | 'direct') {
  const agent = selectedAgent.value
  if (!agent) return [] as Array<AgentRow | PlayerRow>
  const prefix = `${agent.path} >`
  if (type === 'agents') {
    return agentRows.value.filter((row) => scope === 'all' ? row.path.startsWith(prefix) : row.path === `${agent.path} > ${row.account}`)
  }
  const depth = agent.path.split(' > ').length
  return playerRows.value.filter((row) => scope === 'all' ? row.path.startsWith(prefix) : row.path.split(' > ').length === depth + 1)
}

function relationshipFilteredRows(type: 'agents' | 'players', scope: 'all' | 'direct') {
  const rows = [...relationshipRows(type, scope)]
  return relationshipFilterScope.value === 'cycle' ? rows.filter((row) => relationshipInCycle(relationshipFirstDeposit(row)?.at) || relationshipInCycle(relationshipRegisterAt(row))) : rows
}

function relationshipSum(type: 'agents' | 'players', scope: 'all' | 'direct', field: 'deposit' | 'bet') {
  return relationshipFilteredRows(type, scope).reduce((sum, row) => sum + (type === 'agents' ? ((field === 'deposit' ? (row as AgentRow).totalDeposit : (row as AgentRow).totalEffectiveBet) ?? 0) : (field === 'deposit' ? (row as PlayerRow).deposit : (row as PlayerRow).bet)), 0)
}

type RelationshipMetric = 'deposit' | 'bet' | 'firstDepositCount' | 'firstDepositAmount' | 'registeredCount'
const relationshipSortMode = ref<Record<string, string>>({})
const relationshipFilterScope = ref<'history' | 'cycle'>('history')
const relationshipFirstDepositDemo: Record<string, { at: string; amount: number }> = {
  agent_taipei: { at: '2026-08-03 10:21:00', amount: 68000 }, north_team: { at: '2026-08-06 15:40:00', amount: 32000 }, east_team: { at: '2026-08-12 09:15:00', amount: 18500 }, sub_partner_01: { at: '2026-08-18 13:22:00', amount: 9600 }, north_l2: { at: '2026-08-21 11:08:00', amount: 12800 }, player_klaus: { at: '2026-08-05 08:30:00', amount: 5000 }, member_888: { at: '2026-07-14 17:20:00', amount: 3000 }, user_lucky: { at: '2026-08-24 20:10:00', amount: 1800 }, north_l3: { at: '2026-08-26 14:12:00', amount: 6800 }, north_l4: { at: '2026-08-28 12:05:00', amount: 4200 },
}
const settlementCycleStart = '2026-08-01'
const settlementCycleEnd = '2026-08-31 23:59:59'

function relationshipMetricKey(type: 'agents' | 'players', scope: 'all' | 'direct', metric: RelationshipMetric) {
  return `${type}-${scope}-${metric}`
}

function relationshipFirstDeposit(row: AgentRow | PlayerRow) {
  return relationshipFirstDepositDemo[row.account]
}

function relationshipFirstDepositAt(row: AgentRow | PlayerRow) {
  return relationshipFirstDeposit(row)?.at ?? '-'
}

function relationshipFirstDepositAmount(row: AgentRow | PlayerRow) {
  const record = relationshipFirstDeposit(row)
  return record ? `${row.currency} ${record.amount.toLocaleString()}` : '-'
}

function relationshipRegisterAt(row: AgentRow | PlayerRow) {
  return 'registerAt' in row ? row.registerAt : undefined
}

function relationshipRegisterLabel(row: AgentRow | PlayerRow) {
  return relationshipRegisterAt(row) ?? '-'
}

function relationshipInCycle(value?: string) {
  return Boolean(value && value >= settlementCycleStart && value <= settlementCycleEnd)
}

function relationshipCycleCount(type: 'agents' | 'players', scope: 'all' | 'direct') {
  return relationshipRows(type, scope).filter((row) => relationshipInCycle(relationshipFirstDeposit(row)?.at)).length
}

function relationshipCycleAmount(type: 'agents' | 'players', scope: 'all' | 'direct') {
  return relationshipRows(type, scope).reduce((sum, row) => sum + (relationshipInCycle(relationshipFirstDeposit(row)?.at) ? relationshipFirstDeposit(row)?.amount ?? 0 : 0), 0)
}

function relationshipCycleRegistered(type: 'agents' | 'players', scope: 'all' | 'direct') {
  return relationshipRows(type, scope).filter((row) => relationshipInCycle(relationshipRegisterAt(row))).length
}

function relationshipDetailRows(type: 'agents' | 'players', scope: 'all' | 'direct') {
  let rows = relationshipFilteredRows(type, scope)
  const sort = relationshipSortMode.value[relationshipMetricKey(type, scope, 'deposit')] || relationshipSortMode.value[relationshipMetricKey(type, scope, 'bet')] || relationshipSortMode.value[relationshipMetricKey(type, scope, 'firstDepositCount')] || relationshipSortMode.value[relationshipMetricKey(type, scope, 'firstDepositAmount')] || relationshipSortMode.value[relationshipMetricKey(type, scope, 'registeredCount')]
  const rowAmount = (row: AgentRow | PlayerRow, field: 'deposit' | 'bet') => type === 'agents' ? (field === 'deposit' ? (row as AgentRow).totalDeposit ?? 0 : (row as AgentRow).totalEffectiveBet ?? 0) : (field === 'deposit' ? (row as PlayerRow).deposit : (row as PlayerRow).bet)
  if (sort === 'depositDesc') rows.sort((a, b) => rowAmount(b, 'deposit') - rowAmount(a, 'deposit'))
  if (sort === 'betDesc') rows.sort((a, b) => rowAmount(b, 'bet') - rowAmount(a, 'bet'))
  if (sort === 'firstDepositDesc') rows.sort((a, b) => (relationshipFirstDeposit(b)?.at ?? '').localeCompare(relationshipFirstDeposit(a)?.at ?? ''))
  if (sort === 'firstDepositAmountDesc') rows.sort((a, b) => (relationshipFirstDeposit(b)?.amount ?? 0) - (relationshipFirstDeposit(a)?.amount ?? 0))
  if (sort === 'registeredDesc') rows.sort((a, b) => (relationshipRegisterAt(b) ?? '').localeCompare(relationshipRegisterAt(a) ?? ''))
  return rows
}

function setRelationshipSort(key: string, event: Event) {
  const value = (event.target as HTMLSelectElement | null)?.value ?? ''
  const groupPrefix = key.slice(0, key.lastIndexOf('-'))
  Object.keys(relationshipSortMode.value).filter((item) => item.startsWith(`${groupPrefix}-`) && item !== key).forEach((item) => delete relationshipSortMode.value[item])
  if (!value) {
    delete relationshipSortMode.value[key]
    return
  }
  relationshipSortMode.value[key] = value
}


function defaultCommissionConfig(): CommissionConfig {
  return {
    share: { shareRate: 5, adminCostRate: 0, negativeMode: '負數累計', offsetType: '全額沖銷', offsetLimit: 0 },
    rebate: { rebateRate: 4 },
  }
}

function planConfig(plan: CommissionPlan): CommissionConfig {
  const fallback = defaultCommissionConfig()
  return {
    share: { ...fallback.share, ...plan.config?.share },
    rebate: { ...fallback.rebate, ...plan.config?.rebate },
  }
}

function planConfigSummary(plan: CommissionPlan) {
  const config = planConfig(plan)
  if (plan.model === '輸贏') return `${config.share.shareRate}% · 行政費 ${config.share.adminCostRate}% · ${config.share.negativeMode}`
  return `${config.rebate.rebateRate}% · 固定比例`
}

function planForAgent(row: AgentRow) {
  return commissionPlans.value.find((plan) => plan.id === row.planId)
}

function agentPlanSummary(row: AgentRow) {
  const plan = planForAgent(row)
  return plan ? planConfigSummary(plan) : '-'
}

const agentRows = ref<AgentRow[]>([
  { id: 'A-1001', uid: 'AG-10001', account: 'agent_taipei', displayName: '台北代理', referralCode: 'PLY-TW-8F4K', phone: '0912345678', email: 'taipei@example.com', contactMethod: 'Line: @agent_taipei', twoFactor: '已啟用', twoFactorBoundAt: '2026-07-18 14:22:08', twoFactorRequired: true, level: '一級代理', path: 'agent_taipei', currency: 'TWD', point: 6, children: 3, directAgentCount: 2, directPlayerCount: 2, totalAgentCount: 4, totalPlayerCount: 186, totalDeposit: 342800, totalEffectiveBet: 1284600, walletBalance: 28460, withdrawalEnabled: true, registerAt: '2026-07-18 14:22:08', registerIp: '10.20.8.15', lastLoginAt: '2026-09-02 11:00:00', lastLoginIp: '10.20.8.15', status: '啟用', planId: 'PLAN-001', model: '有效投注額', cycle: '每週' },
  { id: 'A-1006', uid: 'AG-10121', account: 'agent_pnl', displayName: '輸贏代理', referralCode: 'PLY-TWD-PNL3', phone: '0922333444', email: 'pnl@example.com', contactMethod: 'Line: @agent_pnl', twoFactor: '未啟用', twoFactorBoundAt: '尚未綁定', twoFactorRequired: true, level: '一級代理', path: 'agent_pnl', currency: 'TWD', point: 5, children: 6, walletBalance: 9740, withdrawalEnabled: true, registerAt: '2026-07-21 09:12:44', registerIp: '10.20.8.66', lastLoginAt: '2026-09-01 17:42:10', lastLoginIp: '10.20.8.66', status: '啟用', planId: 'PLAN-003', model: '輸贏', cycle: '每月' },
  { id: 'A-1002', uid: 'AG-10024', account: 'north_team', displayName: '北區團隊', referralCode: 'PLY-TW-NORTH', phone: '0987654321', email: 'north@example.com', contactMethod: 'Line: @north_team', twoFactor: '未啟用', twoFactorRequired: true, level: '二級代理', path: 'agent_taipei > north_team', currency: 'TWD', point: 4, children: 8, directAgentCount: 1, directPlayerCount: 1, totalAgentCount: 2, totalPlayerCount: 74, totalDeposit: 158600, totalEffectiveBet: 628400, walletBalance: 12680, withdrawalEnabled: true, registerAt: '2026-07-22 10:18:21', registerIp: '10.20.8.42', lastLoginAt: '2026-09-02 09:58:12', lastLoginIp: '10.20.8.42', status: '啟用', planId: 'PLAN-002', model: '有效投注額', cycle: '每週' },
  { id: 'A-1003', uid: 'AG-10088', account: 'east_team', displayName: '東區團隊', referralCode: 'PLY-TW-EAST', phone: '0955123788', email: 'east@example.com', contactMethod: 'Email', twoFactor: '未啟用', twoFactorRequired: true, level: '二級代理', path: 'agent_taipei > east_team', currency: 'TWD', point: 3, children: 5, directAgentCount: 0, directPlayerCount: 1, totalAgentCount: 0, totalPlayerCount: 48, totalDeposit: 93600, totalEffectiveBet: 328400, walletBalance: 8340, withdrawalEnabled: true, registerAt: '2026-07-23 13:06:50', registerIp: '10.20.8.78', lastLoginAt: '2026-09-01 21:12:09', lastLoginIp: '10.20.8.78', status: '啟用', planId: 'PLAN-002', model: '儲值', cycle: '每週' },
  { id: 'A-1004', uid: 'AG-10102', account: 'sub_partner_01', displayName: '合作夥伴 01', referralCode: 'PLY-TW-SUB01', phone: '0933123456', email: 'partner01@example.com', contactMethod: 'Line: @sub_partner_01', twoFactor: '未啟用', twoFactorRequired: true, level: '三級代理', path: 'agent_taipei > north_team > sub_partner_01', currency: 'TWD', point: 2, children: 2, directAgentCount: 0, directPlayerCount: 1, totalAgentCount: 0, totalPlayerCount: 21, totalDeposit: 46200, totalEffectiveBet: 164800, walletBalance: 4260, withdrawalEnabled: true, registerAt: '2026-07-25 16:32:04', registerIp: '10.20.8.91', lastLoginAt: '2026-08-31 19:04:33', lastLoginIp: '10.20.8.91', status: '啟用', planId: 'PLAN-004', model: '有效投注額', cycle: '每週' },
  { id: 'A-1007', uid: 'AG-10131', account: 'north_l2', displayName: '北區二級代理', referralCode: 'PLY-TW-NL2', phone: '0911222333', email: 'north.l2@example.com', contactMethod: 'Line: @north_l2', twoFactor: '未啟用', twoFactorRequired: true, level: '三級代理', path: 'agent_taipei > north_team > north_l2', currency: 'TWD', point: 2, children: 1, directAgentCount: 1, directPlayerCount: 2, totalAgentCount: 2, totalPlayerCount: 32, totalDeposit: 58200, totalEffectiveBet: 214800, walletBalance: 3560, withdrawalEnabled: true, registerAt: '2026-08-01 09:10:00', registerIp: '10.20.8.101', lastLoginAt: '2026-09-02 10:12:30', lastLoginIp: '10.20.8.101', status: '啟用', planId: 'PLAN-005', model: '儲值', cycle: '每週' },
  { id: 'A-1008', uid: 'AG-10132', account: 'north_l3', displayName: '北區三級代理', referralCode: 'PLY-TW-NL3', phone: '0922444555', email: 'north.l3@example.com', contactMethod: 'Email', twoFactor: '未啟用', twoFactorRequired: true, level: '四級代理', path: 'agent_taipei > north_team > north_l2 > north_l3', currency: 'TWD', point: 1, children: 1, directAgentCount: 1, directPlayerCount: 1, totalAgentCount: 1, totalPlayerCount: 18, totalDeposit: 32400, totalEffectiveBet: 118600, walletBalance: 2180, withdrawalEnabled: true, registerAt: '2026-08-08 14:24:10', registerIp: '10.20.8.102', lastLoginAt: '2026-09-01 16:40:18', lastLoginIp: '10.20.8.102', status: '啟用', planId: 'PLAN-001', model: '有效投注額', cycle: '每週' },
  { id: 'A-1009', uid: 'AG-10133', account: 'north_l4', displayName: '北區四級代理', referralCode: 'PLY-TW-NL4', phone: '0933666777', email: 'north.l4@example.com', contactMethod: 'Line: @north_l4', twoFactor: '未啟用', twoFactorRequired: true, level: '五級代理', path: 'agent_taipei > north_team > north_l2 > north_l3 > north_l4', currency: 'TWD', point: 0.5, children: 0, directAgentCount: 0, directPlayerCount: 4, totalAgentCount: 0, totalPlayerCount: 4, totalDeposit: 9800, totalEffectiveBet: 42600, walletBalance: 960, withdrawalEnabled: true, registerAt: '2026-08-15 18:08:42', registerIp: '10.20.8.103', lastLoginAt: '2026-09-02 08:15:47', lastLoginIp: '10.20.8.103', status: '啟用', planId: 'PLAN-001', model: '有效投注額', cycle: '每週' },
])

const commissionPlans = ref<CommissionPlan[]>([
  { id: 'PLAN-001', name: '台灣有效投注標準方案', createdBy: '平台運營商', parentAccount: 'agent_taipei（代理）', model: '有效投注額', cycle: '每週', allocationRate: 6, description: '以有效投注總額計算傭金；下級可用點數不得超過本方案額度。', assignedCount: 3, status: '啟用', config: { ...defaultCommissionConfig(), rebate: { rebateRate: 4 } } },
  { id: 'PLAN-002', name: '台北代理儲值方案', createdBy: 'agent_taipei（代理）', parentAccount: 'agent_taipei（代理）', model: '儲值', cycle: '每週', allocationRate: 5, description: '以成功儲值金額計算傭金；下級可依額度建立自己的方案。', assignedCount: 2, status: '啟用', config: { ...defaultCommissionConfig(), rebate: { rebateRate: 3.5 } } },
  { id: 'PLAN-003', name: '輸贏合作方案', createdBy: '平台運營商', parentAccount: 'agent_taipei（代理）', model: '輸贏', cycle: '每月', allocationRate: 5, description: '依玩家總輸贏扣除行政成本後按比例分配，支援負數沖銷或負數累計。', assignedCount: 0, status: '啟用', config: { ...defaultCommissionConfig(), share: { shareRate: 5, adminCostRate: 1, negativeMode: '負數沖銷', offsetType: '週期固定沖銷額度', offsetLimit: 10000 } } },
  { id: 'PLAN-004', name: '北區下級有效投注方案', createdBy: 'north_team（代理）', parentAccount: 'north_team（代理）', model: '有效投注額', cycle: '每週', allocationRate: 4, description: '北區代理線下級方案；點數不得高於上級提供額度。', assignedCount: 1, status: '啟用', config: { ...defaultCommissionConfig(), rebate: { rebateRate: 3 } } },
  { id: 'PLAN-005', name: '北區儲值方案', createdBy: 'north_team（代理）', parentAccount: 'north_team（代理）', model: '儲值', cycle: '每週', allocationRate: 3, description: '北區代理使用的成功儲值傭金方案，修改於下個結算週期生效。', assignedCount: 1, status: '啟用', config: { ...defaultCommissionConfig(), rebate: { rebateRate: 2.5 } } },
])

const playerRows = ref<PlayerRow[]>([
  { id: 'P-20481', uid: 'P20481', account: 'player_klaus', displayName: 'Klaus 玩家', tags: ['VIP會員'], rtp: 96.8, vipLevel: 'VIP3', agentLevel: '一級玩家', referralCode: 'PLY-TW-3M7X-62ZD', status: '啟用', isOnline: true, registerAt: '2026-07-18 14:22:08', phone: '0912345678', email: 'player.klaus@example.com', gender: '男', birthday: '1992-04-18', registerIp: '10.20.8.31', lastLoginAt: '2026-08-31 09:58:12', lastLoginIp: '10.20.8.45', registerSource: '玩家推廣碼', inviteCode: 'INV-KL8S', consecutiveCheckInDays: 12, path: 'agent_taipei > north_team > player_klaus', currency: 'TWD', deposit: 12800, bet: 95600 },
  { id: 'P-20482', uid: 'P20482', account: 'member_888', displayName: 'Member 888', tags: ['一般'], rtp: 101.2, vipLevel: 'VIP1', agentLevel: '二級玩家', referralCode: 'PLY-TW-3M7X-62ZD', status: '啟用', isOnline: false, registerAt: '2026-06-03 09:41:22', phone: '0987654321', email: 'member888@example.com', gender: '女', birthday: '1996-10-03', registerIp: '10.20.8.52', lastLoginAt: '2026-08-30 20:15:44', lastLoginIp: '10.20.8.62', registerSource: '玩家推廣碼', inviteCode: 'INV-M888', consecutiveCheckInDays: 4, path: 'agent_taipei > north_team > sub_partner_01 > member_888', currency: 'TWD', deposit: 8600, bet: 43200 },
  { id: 'P-20483', uid: 'P20483', account: 'user_lucky', displayName: 'Lucky 玩家', tags: ['風控關注'], rtp: 108.6, vipLevel: 'VIP0', agentLevel: '一級玩家', referralCode: 'PLY-TW-3M7X-62ZD', status: '凍結', isOnline: false, registerAt: '2026-05-26 20:15:44', phone: '0955123788', email: 'lucky@example.com', gender: '男', birthday: '1988-07-21', registerIp: '10.20.8.70', lastLoginAt: '2026-08-21 11:04:02', lastLoginIp: '10.20.8.71', registerSource: '玩家推廣碼', inviteCode: 'INV-LUCKY', consecutiveCheckInDays: 0, path: 'agent_taipei > east_team > user_lucky', currency: 'TWD', deposit: 3200, bet: 18400 },
])

const logs = ref<OperationLog[]>([
  { time: '2026-08-31 10:22:04', type: '登入', actor: 'agent_demo', target: '代理後台', detail: '代理後台登入成功', before: '-', after: '登入成功', ip: '10.20.8.15' },
  { time: '2026-08-30 18:04:12', type: '設定傭金方案', actor: 'agent_demo', target: 'north_team', detail: '有效投注額傭金比例調整', before: '3%', after: '4%', ip: '10.20.8.15' },
  { time: '2026-08-29 14:11:38', type: '開設代理', actor: 'agent_demo', target: 'sub_partner_01', detail: '建立代理，幣別 TWD', before: '-', after: '啟用／TWD', ip: '10.20.8.15' },
  { time: '2026-08-27 09:43:51', type: '提領申請', actor: 'agent_demo', target: 'agent_taipei', detail: '申請提領 TWD 12,000', before: '可提領', after: '待平台審核', ip: '10.20.8.15' },
])

const navGroups = computed(() => [
  { title: '工作台', items: [{ key: 'dashboard', label: '營運概覽', icon: GridOutline }] },
  { title: '下級管理', items: [{ key: 'agents', label: '代理管理', icon: GitNetworkOutline }, { key: 'players', label: '玩家管理', icon: PeopleOutline }, { key: 'codes', label: '傭金方案設定', icon: ShareSocialOutline }] },
  { title: '傭金與報表', items: [{ key: 'commissionPlans', label: '傭金方案', icon: CashOutline }, { key: 'commission', label: '傭金發放紀錄', icon: CashOutline }, { key: 'withdrawal', label: '傭金提領紀錄', icon: WalletOutline }, { key: 'reports', label: '下級報表', icon: BarChartOutline }] },
  { title: '系統', items: [{ key: 'logs', label: '操作日誌', icon: ListOutline }, { key: 'profile', label: '帳戶設定', icon: PeopleOutline }] },
] as const)

const identity = computed(() => currentRole.value === '運營商' ? { account: 'operator_demo', label: '運營商', currency: '多幣別' } : { account: 'agent_taipei', label: '代理', currency: 'TWD' })
const identityAgentPath = computed(() => agentRows.value.find((row) => row.account === identity.value.account)?.path ?? identity.value.account)
function displayAgentPath(path?: string) {
  if (!path) return '-'
  return path.startsWith('平台 >') ? path : `平台 > ${path}`
}
function agentLevelFromDepth(depth: number) {
  const labels = ['', '一級代理', '二級代理', '三級代理', '四級代理', '五級代理']
  return labels[depth] ?? `${depth}級代理`
}
const canCreateAgent = computed(() => currentRole.value !== '運營商' || currentRole.value === '運營商')
const createTitle = computed(() => '開設下級代理')
const pageTitle = computed(() => navGroups.value.flatMap((group) => group.items).find((item) => item.key === activeKey.value)?.label ?? '營運概覽')
function inDateRange(value: string, range: [number, number] | null) {
  if (!range) return true
  const timestamp = new Date(value.replace(' ', 'T')).getTime()
  return timestamp >= range[0] && timestamp <= range[1] + 86400000 - 1
}

const visibleAgentFilterRows = computed(() => agentRows.value.filter((row) => canViewAgent(row)))
const agentParentOptions = computed(() => visibleAgentFilterRows.value
  .filter((row) => agentRows.value.some((child) => canViewAgent(child) && child.path.startsWith(`${row.path} >`)))
  .map((row) => ({ label: `${row.account}（${row.level}）`, value: row.account })))
const playerParentOptions = computed(() => visibleAgentFilterRows.value
  .map((row) => ({ label: `${row.account}（${row.level}）`, value: row.account })))

const filteredAgents = computed(() => agentRows.value.filter((row) => {
  const selectedParent = agentParentFilter.value ? agentRows.value.find((agent) => agent.account === agentParentFilter.value) : null
  const basePath = selectedParent?.path || identityAgentPath.value
  const directAgent = currentRole.value === '運營商'
    ? (selectedParent ? row.path === `${basePath} > ${row.account}` : row.path.split(' > ').length === 1)
    : (selectedParent ? row.path === `${basePath} > ${row.account}` : row.account === identity.value.account || row.path === `${basePath} > ${row.account}`)
  const underParent = selectedParent ? row.path.startsWith(`${basePath} >`) : canViewAgent(row)
  const inScope = underParent && (agentScope.value === 'all' || directAgent)
  const inLevel = !agentLevelFilter.value || row.level === agentLevelFilter.value
  const inStatus = !agentStatusFilter.value || row.status === agentStatusFilter.value
  const inCommission = !agentCommissionFilter.value || row.model === agentCommissionFilter.value
  const query = agentAccountSearch.value.trim().toLowerCase()
  const ipQuery = agentRegisterIp.value.trim().toLowerCase()
  return inScope && inLevel && inStatus && inCommission
    && (!query || `${row.account}${row.path}`.toLowerCase().includes(query))
    && (!ipQuery || (row.registerIp ?? '').toLowerCase().includes(ipQuery))
    && inDateRange(row.registerAt ?? '', agentRegisterDateRange.value)
}))
const filteredPlayers = computed(() => playerRows.value.filter((row) => {
  const selectedParent = playerParentFilter.value ? agentRows.value.find((agent) => agent.account === playerParentFilter.value) : null
  const basePath = selectedParent?.path || identityAgentPath.value
  const visibleToRole = currentRole.value === '運營商' || row.path.startsWith(`${identityAgentPath.value} >`)
  const directPlayer = selectedParent
    ? row.path === `${basePath} > ${row.account}`
    : (currentRole.value === '運營商' ? row.path.split(' > ').length === 2 : row.path.startsWith(`${basePath} >`) && row.path.split(' > ').length === basePath.split(' > ').length + 1)
  const inScope = (selectedParent ? row.path.startsWith(`${basePath} >`) : visibleToRole) && (playerScope.value === 'all' || directPlayer)
  const query = playerSearch.value.trim().toLowerCase()
  const ipQuery = playerRegisterIp.value.trim().toLowerCase()
  const searchValue = playerSearchType.value === 'id' ? row.id : playerSearchType.value === 'phone' ? row.phone : row.account
  const affiliationValue = playerAffiliationType.value === 'invite' ? (row.inviteCode || '') : (row.referralCode || '')
  return inScope
    && (!query || `${searchValue}${row.displayName}${row.path}`.toLowerCase().includes(query))
    && (!playerAffiliationQuery.value.trim() || affiliationValue.toLowerCase().includes(playerAffiliationQuery.value.trim().toLowerCase()))
    && (!ipQuery || row.registerIp.toLowerCase().includes(ipQuery))
    && (!playerStatusFilter.value || row.status === playerStatusFilter.value)
    && (!playerTagsFilter.value.length || playerTagsFilter.value.some((tag) => row.tags.includes(tag)))
    && inDateRange(row.registerAt, playerRegisterDateRange.value)
}))
const planOptions = computed(() => commissionPlans.value.filter((plan) => plan.status === '啟用' && (currentRole.value === '運營商' || plan.model === lineModel.value)).map((plan) => ({ label: `${plan.name}（${plan.model}／${plan.cycle}）`, value: plan.id })))
const selectedPlan = computed(() => commissionPlans.value.find((plan) => plan.id === draftPlanId.value) ?? commissionPlans.value[0])
const selectedPlanConfig = computed(() => planConfig(selectedPlan.value || commissionPlans.value[0]))
function normalizeCommissionMode(mode?: AgentRow['model']): CommissionPlan['model'] {
  if (mode === '佔成' || mode === '輸贏') return '輸贏'
  if (mode === '返佣' || mode === '有效投注額') return '有效投注額'
  return '有效投注額'
}
const lineModel = computed(() => currentRole.value === '運營商' ? null : normalizeCommissionMode(agentRows.value.find((row) => row.account === identity.value.account)?.model))
const selectedAgentPlan = computed(() => selectedAgent.value ? planForAgent(selectedAgent.value) : undefined)
const selectedAgentCreatedPlans = computed(() => selectedAgent.value ? plansCreatedBy(selectedAgent.value.account) : [])
const selectedAgentCommissionPlans = computed(() => {
  const plans = selectedAgentPlan.value ? [selectedAgentPlan.value] : []
  selectedAgentCreatedPlans.value.forEach((plan) => {
    if (!plans.some((item) => item.id === plan.id)) plans.push(plan)
  })
  return plans
})
const currentAgentForPlan = computed(() => agentRows.value.find((row) => row.account === identity.value.account))
const currentUpstreamPlan = computed(() => currentAgentForPlan.value ? planForAgent(currentAgentForPlan.value) : undefined)
const currentCreatedPlans = computed(() => plansCreatedBy(identity.value.account))
const visiblePlans = computed(() => currentRole.value === '運營商'
  ? commissionPlans.value
  : commissionPlans.value.filter((plan) => plan.createdBy?.startsWith(identity.value.account) || currentAgentForPlan.value?.planId === plan.id))
const availablePlanOptions = computed(() => visiblePlans.value.filter((plan) => plan.status === '啟用').map((plan) => ({ label: `${plan.name}（${plan.model}／${plan.cycle}）`, value: plan.id })))
const lineMaxRate = computed(() => currentRole.value === '運營商' ? 100 : (agentRows.value.find((row) => row.account === identity.value.account)?.point ?? 0))
const canManagePlans = computed(() => true)
const expandedPlanIds = ref<string[]>([])
const codesPlanModelFilter = ref<CommissionPlan['model'] | null>(null)
const codesPlanStatusFilter = ref<CommissionPlan['status'] | null>(null)
const detailPlanModelFilter = ref<CommissionPlan['model'] | null>(null)
const detailPlanStatusFilter = ref<CommissionPlan['status'] | null>(null)
const planReportCreatedBy = ref('')
const planReportAppliedAccount = ref('')
const planReportModel = ref<CommissionPlan['model'] | null>(null)
const planReportCycle = ref<CommissionPlan['cycle'] | null>(null)
const planReportStatus = ref<CommissionPlan['status'] | null>(null)
const filteredCurrentCreatedPlans = computed(() => currentCreatedPlans.value.filter((plan) => (!codesPlanModelFilter.value || plan.model === codesPlanModelFilter.value) && (!codesPlanStatusFilter.value || plan.status === codesPlanStatusFilter.value)))
const filteredSelectedAgentCreatedPlans = computed(() => selectedAgentCreatedPlans.value.filter((plan) => (!detailPlanModelFilter.value || plan.model === detailPlanModelFilter.value) && (!detailPlanStatusFilter.value || plan.status === detailPlanStatusFilter.value)))
const filteredVisiblePlans = computed(() => visiblePlans.value.filter((plan) => {
  const creatorQuery = planReportCreatedBy.value.trim().toLowerCase()
  const accountQuery = planReportAppliedAccount.value.trim().toLowerCase()
  const creator = `${plan.createdBy || ''} ${plan.parentAccount || ''}`.toLowerCase()
  const hasAppliedAccount = !accountQuery || planAppliedAgents(plan).some((agent) => agent.account.toLowerCase().includes(accountQuery))
  return (!creatorQuery || creator.includes(creatorQuery)) && hasAppliedAccount && (!planReportModel.value || plan.model === planReportModel.value) && (!planReportCycle.value || plan.cycle === planReportCycle.value) && (!planReportStatus.value || plan.status === planReportStatus.value)
}))
function canEditPlan(plan: CommissionPlan) {
  return currentRole.value === '運營商' || Boolean(plan.createdBy?.startsWith(identity.value.account))
}
function planAppliedAgents(plan: CommissionPlan) {
  return agentRows.value.filter((row) => row.planId === plan.id)
}
function plansCreatedBy(account: string) {
  return commissionPlans.value.filter((plan) => plan.createdBy?.startsWith(account))
}
function togglePlanExpanded(planId: string) {
  expandedPlanIds.value = expandedPlanIds.value.includes(planId)
    ? expandedPlanIds.value.filter((id) => id !== planId)
    : [...expandedPlanIds.value, planId]
}
const canOperatePlayers = computed(() => currentRole.value === '運營商')
const withdrawalEligible = computed(() => withdrawableBalance.value >= withdrawalMin.value && withdrawalTodayCount.value < withdrawalDailyLimit.value)
const planParentOptions = computed(() => [
  { label: 'agent_taipei（代理）', value: 'agent_taipei（代理）' },
  { label: 'north_team（代理）', value: 'north_team（代理）' },
  { label: 'east_team（代理）', value: 'east_team（代理）' },
])
const playerStatusOptions = [
  { label: '啟用', value: '啟用' },
  { label: '鎖定', value: '鎖定' },
  { label: '凍結', value: '凍結' },
  { label: '暫停', value: '暫停' },
]
const playerTransferOptions = computed(() => agentRows.value.filter((row) => row.status === '啟用').map((row) => ({ label: `${row.account} · ${row.path}`, value: row.path })))
const agentTransferOptions = computed(() => {
  if (!selectedAgent.value) return []
  return agentRows.value
    .filter((row) => row.status === '啟用' && row.path.split(' > ').length > 1 && row.currency === selectedAgent.value?.currency && row.id !== selectedAgent.value?.id)
    .filter((row) => !row.path.startsWith(`${selectedAgent.value?.path} >`))
    .map((row) => ({ label: `${row.account}（${row.level}）`, value: row.id }))
})
const selectedTransferTarget = computed(() => agentRows.value.find((row) => row.id === agentTransferTarget.value) ?? null)
const projectedTransferPath = computed(() => selectedAgent.value && selectedTransferTarget.value
  ? displayAgentPath(`${selectedTransferTarget.value.path} > ${selectedAgent.value.account}`)
  : '')
const filteredLogs = computed(() => logs.value.filter((log) => {
  const typeMatch = !logTypeFilter.value || log.type === logTypeFilter.value
  const query = logSearch.value.trim().toLowerCase()
  const textMatch = !query || `${log.actor}${log.target ?? ''}${log.detail}${log.ip}${log.before ?? ''}${log.after ?? ''}`.toLowerCase().includes(query)
  const timestamp = new Date(log.time.replace(' ', 'T')).getTime()
  const dateMatch = !logDateRange.value || (timestamp >= logDateRange.value[0] && timestamp <= logDateRange.value[1] + 86400000 - 1)
  return typeMatch && textMatch && dateMatch
}))
const agentDetailDemoLogs = computed<OperationLog[]>(() => {
  const account = selectedAgent.value?.account
  if (!account) return []
  return [
    { time: '2026-09-02 11:00:00', type: '登入', actor: account, target: account, detail: '代理後台登入成功', before: '-', after: '登入成功', ip: selectedAgent.value?.lastLoginIp || '10.20.8.15' },
    { time: '2026-09-01 16:20:14', type: '代理 2FA 設定', actor: 'operator_demo', target: account, detail: '更新 Google Auth 要求', before: '停用', after: '啟用', ip: '10.20.8.15' },
    { time: '2026-08-30 18:04:12', type: '設定傭金方案', actor: 'agent_demo', target: account, detail: '套用有效投注額方案', before: '未套用', after: '台灣有效投注標準方案', effectiveAt: '2026-09-01 00:00:00', ip: '10.20.8.15' },
    { time: '2026-08-29 14:11:38', type: '代理轉線', actor: 'operator_demo', target: account, detail: '代理線調整預約', before: selectedAgent.value?.path || '-', after: `${selectedAgent.value?.path || '-'}（新代理線）`, effectiveAt: '本次結算週期結束後', ip: '10.20.8.15' },
    { time: '2026-08-27 09:43:51', type: '修改提領狀態', actor: 'operator_demo', target: account, detail: '調整可提領狀態', before: '不可提領', after: '可提領', ip: '10.20.8.15' },
  ]
})
const filteredAgentDetailLogs = computed(() => {
  const query = agentDetailLogSearch.value.trim().toLowerCase()
  return agentDetailDemoLogs.value.filter((log) => {
    const typeMatch = !agentDetailLogType.value || log.type === agentDetailLogType.value
    const textMatch = !query || `${log.actor}${log.target ?? ''}${log.detail}${log.ip}${log.before ?? ''}${log.after ?? ''}`.toLowerCase().includes(query)
    const timestamp = new Date(log.time.replace(' ', 'T')).getTime()
    const dateMatch = !agentDetailLogDateRange.value || (timestamp >= agentDetailLogDateRange.value[0] && timestamp <= agentDetailLogDateRange.value[1] + 86400000 - 1)
    return typeMatch && textMatch && dateMatch
  })
})
const agentWithdrawalRecords = computed<CommissionWithdrawalRecord[]>(() => {
  const currency = selectedAgent.value?.currency || 'TWD'
  return [
    { id: 'COM-20260901-001', recordType: '傭金發放', applyAt: '2026-09-01 00:00:00', settlePeriod: '2026-W35', amount: 28460, fee: 0, netAmount: 28460, balanceBefore: 0, balanceAfter: 28460, status: '成功', bank: '返佣方案 · 每週結算', reviewAt: '系統結算', reviewer: 'system', remark: '依台灣返佣標準方案結算本期有效投注' },
    { id: 'WD-20260902-001', recordType: '傭金提領', applyAt: '2026-09-02 11:26:08', settlePeriod: '2026-W35', amount: 12000, fee: 0, netAmount: 12000, balanceBefore: 28460, balanceAfter: 16460, status: '處理中', bank: '台新銀行 · ****9012', reviewAt: '-', reviewer: '待分派', remark: '代理提領傭金轉現金' },
    { id: 'COM-20260825-004', recordType: '傭金發放', applyAt: '2026-08-25 00:00:00', settlePeriod: '2026-W34', amount: 18600, fee: 0, netAmount: 18600, balanceBefore: 12460, balanceAfter: 31060, status: '成功', bank: '返佣方案 · 每週結算', reviewAt: '系統結算', reviewer: 'system', remark: '依方案結算有效投注傭金' },
    { id: 'WD-20260825-004', recordType: '傭金提領', applyAt: '2026-08-25 09:14:32', settlePeriod: '2026-W34', amount: 8600, fee: 0, netAmount: 8600, balanceBefore: 31060, balanceAfter: 22460, status: '成功', bank: '台新銀行 · ****9012', reviewAt: '2026-08-25 14:08:21', reviewer: 'operator_demo', remark: `${currency} 傭金已匯入` },
    { id: 'COM-20260811-002', recordType: '傭金發放', applyAt: '2026-08-11 00:00:00', settlePeriod: '2026-W32', amount: 15400, fee: 0, netAmount: 15400, balanceBefore: 7060, balanceAfter: 22460, status: '成功', bank: '返佣方案 · 每週結算', reviewAt: '系統結算', reviewer: 'system', remark: '依方案結算有效投注傭金' },
    { id: 'WD-20260728-006', recordType: '傭金提領', applyAt: '2026-07-28 13:08:55', settlePeriod: '2026-W30', amount: 4200, fee: 0, netAmount: 4200, balanceBefore: 11260, balanceAfter: 7060, status: '失敗', bank: '台新銀行 · ****9012', reviewAt: '2026-07-29 09:18:06', reviewer: 'operator_demo', remark: '收款帳戶驗證未完成' },
  ]
})
const filteredAgentWithdrawalRecords = computed(() => {
  const query = agentWithdrawalKeyword.value.trim().toLowerCase()
  return agentWithdrawalRecords.value.filter((record) => {
    const textMatch = !query || `${record.id}${record.bank}${record.reviewer}${record.remark}`.toLowerCase().includes(query)
    const statusMatch = !agentWithdrawalStatus.value || record.status === agentWithdrawalStatus.value
    const typeMatch = !agentWithdrawalType.value || record.recordType === agentWithdrawalType.value
    const dateMatch = inDateRange(record.applyAt, agentWithdrawalDateRange.value)
    return textMatch && statusMatch && typeMatch && dateMatch
  })
})
const visibleWithdrawalOrders = computed(() => withdrawalOrders.value.filter((order) => currentRole.value === '運營商' || canViewAgent(agentRows.value.find((row) => row.account === order.account) || agentRows.value[0])))
const filteredWithdrawalOrders = computed(() => visibleWithdrawalOrders.value.filter((order) => {
  const query = withdrawalReportKeyword.value.trim().toLowerCase()
  return (!query || `${order.id}${order.account}${order.bankName}${order.bankAccount}${order.bankHolder}`.toLowerCase().includes(query))
    && (!withdrawalReportStatus.value || order.status === withdrawalReportStatus.value)
    && inDateRange(order.createdAt, withdrawalReportDateRange.value)
}))
const visiblePayoutRecords = computed(() => payoutRecords.value.filter((record) => currentRole.value === '運營商' || canViewAgent(agentRows.value.find((row) => row.account === record.account) || agentRows.value[0])))
const filteredPayoutRecords = computed(() => visiblePayoutRecords.value.filter((record) => {
  const query = payoutReportKeyword.value.trim().toLowerCase()
  return (!query || `${record.id}${record.account}${record.plan}${record.cycle}${record.remark}`.toLowerCase().includes(query))
    && (!payoutReportStatus.value || record.status === payoutReportStatus.value)
    && inDateRange(record.createdAt, payoutReportDateRange.value)
}))
const payoutSummary = computed(() => ({
  pending: filteredPayoutRecords.value.filter((row) => row.status === '處理中').length,
  success: filteredPayoutRecords.value.filter((row) => row.status === '成功').length,
  failed: filteredPayoutRecords.value.filter((row) => row.status === '失敗').length,
  amount: filteredPayoutRecords.value.reduce((sum, row) => sum + row.amount, 0),
}))
const withdrawalSummary = computed(() => ({
  pending: filteredWithdrawalOrders.value.filter((row) => row.status === '待處理').length,
  processing: filteredWithdrawalOrders.value.filter((row) => row.status === '處理中').length,
  success: filteredWithdrawalOrders.value.filter((row) => row.status === '成功').length,
  failed: filteredWithdrawalOrders.value.filter((row) => row.status === '失敗').length,
  amount: filteredWithdrawalOrders.value.reduce((sum, row) => sum + row.amount, 0),
}))
function openCommissionRecordDetail(record: CommissionWithdrawalRecord) {
  selectedCommissionRecord.value = record
  showCommissionRecordDetail.value = true
}

function openWithdrawalOrder(order: CommissionWithdrawalOrder) {
  selectedWithdrawalOrder.value = order
  withdrawalProcessAction.value = '成功'
  withdrawalFailureReason.value = order.failureReason || ''
  showWithdrawalProcess.value = true
}
function startWithdrawalProcessing(order: CommissionWithdrawalOrder) {
  if (currentRole.value !== '運營商' || order.status !== '待處理') return
  openWithdrawalOrder(order)
  prepareCommissionResult('withdrawal-start', '處理中')
}
function prepareCommissionResult(kind: 'withdrawal' | 'payout' | 'withdrawal-start', result: '成功' | '失敗' | '處理中') {
  pendingCommissionAction.value = kind
  pendingCommissionResult.value = result
  commissionActionRemark.value = ''
  showCommissionActionConfirm.value = true
}
function confirmCommissionResult() {
  if (pendingCommissionAction.value !== 'withdrawal-start' && !commissionActionRemark.value.trim()) {
    showNotice('warning', '請填寫原因備註', '所有人工操作都必須留下原因備註，才能完成二次確認。')
    return
  }
  if (pendingCommissionAction.value === 'withdrawal-start') {
    const order = selectedWithdrawalOrder.value
    if (order && order.status === '待處理') {
      order.status = '處理中'
      order.updatedAt = operationTimestamp()
      order.processor = identity.value.account
      order.remark = commissionActionRemark.value.trim() || undefined
      showCommissionActionConfirm.value = false
      showWithdrawalProcess.value = true
      showNotice('success', '已進入出款處理', '此訂單已鎖定為處理中，其他人無法重複操作。')
    }
  } else if (pendingCommissionAction.value === 'withdrawal') {
    withdrawalProcessAction.value = pendingCommissionResult.value
    withdrawalFailureReason.value = commissionActionRemark.value.trim()
    completeWithdrawalOrder()
  } else if (pendingCommissionAction.value === 'payout') {
    payoutProcessAction.value = pendingCommissionResult.value
    payoutFailureReason.value = commissionActionRemark.value.trim()
    completePayoutRecord()
  }
  showCommissionActionConfirm.value = false
  pendingCommissionAction.value = null
}
function completeWithdrawalOrder() {
  const order = selectedWithdrawalOrder.value
  if (!order || currentRole.value !== '運營商' || order.status !== '處理中') return
  if (!withdrawalFailureReason.value.trim()) {
    showNotice('warning', '請填寫原因備註', '完成出款處理前必須記錄原因備註。')
    return
  }
  order.status = withdrawalProcessAction.value
  order.updatedAt = operationTimestamp()
  order.failureReason = withdrawalFailureReason.value.trim()
  order.remark = withdrawalFailureReason.value.trim()
  showWithdrawalProcess.value = false
  showNotice('success', `出款${order.status}`, `訂單 ${order.id} 已更新為${order.status}。`)
}
function openPayoutProcess(record: CommissionPayoutRecord) {
  selectedPayoutRecord.value = record
  payoutProcessAction.value = '成功'
  payoutFailureReason.value = record.status === '失敗' ? record.remark : ''
  showPayoutProcess.value = true
}
function completePayoutRecord() {
  const record = selectedPayoutRecord.value
  if (!record || currentRole.value !== '運營商' || record.status !== '處理中') return
  if (!payoutFailureReason.value.trim()) {
    showNotice('warning', '請填寫原因備註', '完成傭金發放處理前必須記錄原因備註。')
    return
  }
  record.status = payoutProcessAction.value
  record.updatedAt = operationTimestamp()
  record.systemResult = payoutProcessAction.value === '成功' ? '成功：已發放至傭金錢包' : '失敗：運營商手動判定未通過'
  record.remark = payoutFailureReason.value.trim()
  showPayoutProcess.value = false
  showNotice('success', `傭金發放${record.status}`, `紀錄 ${record.id} 已更新為${record.status}。`)
}

function canViewAgent(row: AgentRow) {
  if (currentRole.value === '運營商') return true
  return row.account === identity.value.account || row.path.startsWith(`${identityAgentPath.value} >`)
}

function canManageAgent(row: AgentRow) {
  if (currentRole.value === '運營商') return true
  return row.path.startsWith(`${identityAgentPath.value} >`)
}

function login() {
  if (!loginAccount.value || !loginPassword.value) {
  showNotice('warning', '請輸入登入資訊', '帳號與密碼皆為必填。')
    return
  }
  const account = loginAccount.value.trim()
  const agent = agentRows.value.find((row) => row.account === account)
  if (agent?.twoFactorRequired) {
    loginTwoFactorAccount.value = account
    loginTwoFactorCode.value = ''
    if (agent.twoFactor !== '已啟用') {
      showLoginTwoFactorSetup.value = true
    } else {
      showLoginTwoFactorVerify.value = true
    }
    return
  }
  completeLogin()
}

function completeLogin() {
  currentRole.value = loginRole.value
  loggedIn.value = true
  logs.value.unshift({ time: operationTimestamp(), type: '登入', actor: loginAccount.value.trim(), target: identity.value.account, detail: `${identity.value.label}登入代理後台成功`, before: '-', after: '登入成功', ip: '10.20.8.15' })
}

function validateTwoFactorCode() {
  if (!/^\d{6}$/.test(loginTwoFactorCode.value.trim())) {
    showNotice('warning', '驗證碼格式錯誤', '請輸入 Google Authenticator 顯示的 6 位數驗證碼。')
    return false
  }
  return true
}

function verifyLoginTwoFactor() {
  if (!validateTwoFactorCode()) return
  showLoginTwoFactorVerify.value = false
  completeLogin()
}

function cancelLoginTwoFactor() {
  showLoginTwoFactorSetup.value = false
  showLoginTwoFactorVerify.value = false
  loginTwoFactorCode.value = ''
}

function showNotice(type: 'success' | 'warning', title: string, content: string) {
  notice.value = { type, title, content }
  window.setTimeout(() => { notice.value = null }, 3500)
}

function logout() {
  loggedIn.value = false
  activeKey.value = 'dashboard'
}

function applyAgentFilters() {
  agentAccountSearch.value = agentAccountDraft.value
  agentRegisterIp.value = agentRegisterIpDraft.value
  agentRegisterDateRange.value = agentRegisterDateRangeDraft.value
  agentParentFilter.value = agentParentDraft.value
  agentScope.value = agentScopeDraft.value
  agentLevelFilter.value = agentLevelDraft.value
  agentStatusFilter.value = agentStatusDraft.value
  agentCommissionFilter.value = agentCommissionDraft.value
}

function applyPlayerFilters() {
  playerSearch.value = playerSearchDraft.value
  playerRegisterIp.value = playerRegisterIpDraft.value
  playerRegisterDateRange.value = playerRegisterDateRangeDraft.value
  playerParentFilter.value = playerParentDraft.value
  playerScope.value = playerScopeDraft.value
  playerStatusFilter.value = playerStatusFilterDraft.value
}

function selectNav(key: string) {
  activeKey.value = key as NavKey
  // 代理詳情報表是代理管理頁內的 overlay；離開主介面時清除詳情狀態，避免覆蓋其他頁面。
  showAgentDetail.value = false
  showAgentDetailModal.value = false
  if (key === 'players') showPlayerDetail.value = false
  search.value = ''
  playerSearch.value = ''
  playerSearchDraft.value = ''
  playerRegisterIp.value = ''
  playerRegisterIpDraft.value = ''
  playerRegisterDateRange.value = null
  playerRegisterDateRangeDraft.value = null
  playerParentFilter.value = null
  playerParentDraft.value = null
  playerScope.value = 'all'
  playerScopeDraft.value = 'all'
  playerStatusFilter.value = null
  playerStatusFilterDraft.value = null
  agentAccountSearch.value = ''
  agentAccountDraft.value = ''
  agentRegisterIp.value = ''
  agentRegisterIpDraft.value = ''
  agentRegisterDateRange.value = null
  agentRegisterDateRangeDraft.value = null
  agentParentFilter.value = null
  agentParentDraft.value = null
  agentScope.value = 'all'
  agentScopeDraft.value = 'all'
  agentLevelFilter.value = null
  agentLevelDraft.value = null
  agentStatusFilter.value = null
  agentStatusDraft.value = null
  agentCommissionFilter.value = null
  agentCommissionDraft.value = null
}

function deactivateAgent(row: AgentRow) {
  if (!canManageAgent(row)) {
    showNotice('warning', '無操作權限', '只有運營商或具備管理權限的上級代理可以停用或啟用代理。')
    return
  }
  if (row.status === '啟用') {
    pendingDeactivateAgent.value = row
    showDeactivateAgentConfirm.value = true
    return
  }
  applyAgentStatus(row)
}

function applyAgentStatus(row: AgentRow) {
  row.status = row.status === '啟用' ? '停用' : '啟用'
  logs.value.unshift({ time: operationTimestamp(), type: row.status === '啟用' ? '啟用代理' : '停用代理', actor: loginAccount.value, target: row.account, detail: '代理狀態變更', before: row.status === '啟用' ? '停用' : '啟用', after: row.status, ip: '10.20.8.15' })
  showNotice('success', '狀態已更新', `${row.account} 現在為${row.status}`)
}

function confirmDeactivateAgent() {
  if (!pendingDeactivateAgent.value) return
  applyAgentStatus(pendingDeactivateAgent.value)
  pendingDeactivateAgent.value = null
  showDeactivateAgentConfirm.value = false
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
  const basePath = currentRole.value === '運營商' ? account : `${identity.value.account} > ${account}`
  const level = agentLevelFromDepth(basePath.split(' > ').length)
  agentRows.value.push({ id: `A-${1000 + next}`, uid: `AG-${10000 + next}`, account, displayName: newAgentName.value.trim() || account, referralCode: `PLY-${newAgentCurrency.value}-${String(next).padStart(4, '0')}`, phone: newAgentPhone.value.trim(), email: newAgentEmail.value.trim(), contactMethod: '尚未設定', twoFactor: '未啟用', twoFactorBoundAt: twoFactorGlobalEnabled.value ? '待首次登入綁定' : '不要求綁定', twoFactorRequired: twoFactorGlobalEnabled.value, level, path: basePath, currency: newAgentCurrency.value, point: savedPoint, children: 0, walletBalance: 0, withdrawalEnabled: true, registerAt: '2026-09-02 11:20:00', lastLoginAt: '尚未登入', lastLoginIp: '尚未登入', status: '啟用', planId: selected.id, model: selected.model, cycle: selected.cycle })
  selected.assignedCount += 1
  showCreateAgent.value = false
  newAgentAccount.value = ''
  newAgentName.value = ''
  newAgentPhone.value = ''
  newAgentEmail.value = ''
  newAgentPassword.value = ''
  newAgentPlanId.value = 'PLAN-001'
  logs.value.unshift({ time: operationTimestamp(), type: '開設代理', actor: loginAccount.value, target: account, detail: `建立${level}，幣別 ${newAgentCurrency.value}`, before: '-', after: '啟用', ip: '10.20.8.15' })
  showNotice('success', '代理已建立', `${account} 可立即登入使用；已套用「${selected.name}」。`)
}

function submitWithdrawal() {
  if (!withdrawalEligible.value) {
    showNotice('warning', '目前不可提領', `可提領條件：餘額至少 TWD ${withdrawalMin.value.toLocaleString()}，且每日未達 ${withdrawalDailyLimit.value} 次上限。`)
    return
  }
  if (!withdrawalAmount.value || withdrawalAmount.value < withdrawalMin.value) {
    showNotice('warning', '未符合提領條件', `最低提領金額為 TWD ${withdrawalMin.value.toLocaleString()}。`)
    return
  }
  if (withdrawalAmount.value > withdrawableBalance.value) {
    showNotice('warning', '提領金額超過餘額', `目前最多可提領 TWD ${withdrawableBalance.value.toLocaleString()}。`)
    return
  }
  showWithdrawal.value = false
  const amount = withdrawalAmount.value
  withdrawalOrders.value.unshift({ id: `WD-${operationTimestamp().replace(/[-: ]/g, '').slice(0, 12)}-${String(withdrawalOrders.value.length + 1).padStart(3, '0')}`, account: identity.value.account, currency: identity.value.currency, amount, status: '待處理', createdAt: operationTimestamp(), updatedAt: operationTimestamp(), bankName: bankName.value, bankAccount: `****${bankAccount.value.slice(-4)}`, bankHolder: bankHolder.value })
  logs.value.unshift({ time: operationTimestamp(), type: '提領申請', actor: loginAccount.value, target: identity.value.account, detail: `申請提領 TWD ${amount.toLocaleString()}`, before: '可提領', after: '待處理', ip: '10.20.8.15' })
  showNotice('success', '提領申請已送出', '訂單目前為待處理，等待運營商出款處理。')
  withdrawalAmount.value = null
}

function saveBankCard() {
  if (!bankName.value || !bankAccount.value || !bankHolder.value) {
    showNotice('warning', '資料未完成', '銀行名稱、帳號與戶名皆為必填。')
    return
  }
  showBankCard.value = false
  logs.value.unshift({ time: operationTimestamp(), type: '更新帳戶', actor: loginAccount.value, target: identity.value.account, detail: '新增／更新傭金收款銀行卡', before: '舊收款帳戶', after: '新收款帳戶待驗證', ip: '10.20.8.15' })
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
  detailTab.value = 'wallet'
  draftPlanId.value = row.planId
  showAgentDetail.value = true
}

function editAgent(row: AgentRow) {
  if (!canManageAgent(row)) {
    showNotice('warning', '無編輯權限', '只有運營商或具備管理權限的上級代理可編輯自己權限範圍內的代理基本資料。')
    return
  }
  selectedAgent.value = row
  agentNameDraft.value = row.displayName
  agentPhoneDraft.value = row.phone ?? ''
  agentEmailDraft.value = row.email ?? ''
  agentPasswordDraft.value = ''
  showAgentEdit.value = true
}

function saveAgentProfile() {
  if (!selectedAgent.value || !agentNameDraft.value.trim()) {
    showNotice('warning', '資料未完成', '代理名稱不可為空白。')
    return
  }
  selectedAgent.value.displayName = agentNameDraft.value.trim()
  selectedAgent.value.phone = agentPhoneDraft.value.trim()
  selectedAgent.value.email = agentEmailDraft.value.trim()
  const passwordChanged = Boolean(agentPasswordDraft.value.trim())
  logs.value.unshift({ time: operationTimestamp(), type: '編輯代理資料', actor: loginAccount.value, target: selectedAgent.value.account, detail: passwordChanged ? '基本資料與登入密碼已更新' : '基本資料已更新（登入密碼未變更）', before: passwordChanged ? '原密碼' : '密碼維持不變', after: passwordChanged ? '新密碼已設定' : '密碼維持不變', ip: '10.20.8.15' })
  showAgentEdit.value = false
  agentPasswordDraft.value = ''
  showNotice('success', '代理資料已更新', `${selectedAgent.value.account} 的基本資料已儲存。`)
}

const nextSettlementEffectiveAt = computed(() => selectedAgent.value ? `${selectedAgent.value.cycle === '每月' ? '2026-10-01' : '2026-09-09'} 00:00:00` : '')

function openAgentTransfer(row: AgentRow) {
  if (currentRole.value !== '運營商') {
    showNotice('warning', '無操作權限', '代理轉線已移至運營後台，由運營商統一操作。')
    return
  }
  if (row.path.split(' > ').length === 1) {
    showNotice('warning', '不可轉線', '一級代理不可轉移代理線。')
    return
  }
  selectedAgent.value = row
  agentTransferTarget.value = ''
  showAgentTransfer.value = true
}

function submitAgentTransfer() {
  if (!selectedAgent.value || !agentTransferTarget.value) {
    showNotice('warning', '請選擇新代理線', '請選擇同幣別且不在目前下級樹狀路徑內的代理。')
    return
  }
  const target = agentRows.value.find((row) => row.id === agentTransferTarget.value)
  if (!target || target.path.split(' > ').length === 1 || target.currency !== selectedAgent.value.currency || target.path.startsWith(`${selectedAgent.value.path} >`)) {
    showNotice('warning', '代理線不可用', '新代理線必須與目前代理同幣別，且不可選擇自己或自己的下級。')
    return
  }
  pendingAgentTransfer.value = selectedAgent.value
  pendingAgentTransferTarget.value = target
  showAgentTransferConfirm.value = true
}

function confirmAgentTransfer() {
  if (!pendingAgentTransfer.value || !pendingAgentTransferTarget.value) return
  const source = pendingAgentTransfer.value
  const target = pendingAgentTransferTarget.value
  const effectiveAt = nextSettlementEffectiveAt.value
  source.pendingTransferTargetPath = `${target.path} > ${source.account}`
  source.pendingTransferEffectiveAt = effectiveAt
  source.lastTransferAt = operationTimestamp()
  logs.value.unshift({ time: operationTimestamp(), type: '代理轉線', actor: loginAccount.value, target: source.account, detail: `預約轉至 ${target.account}；本次結算週期結束後生效`, before: source.path, after: source.pendingTransferTargetPath, effectiveAt, ip: '10.20.8.15' })
  showAgentTransfer.value = false
  showAgentTransferConfirm.value = false
  showNotice('success', '代理轉線已預約', `將於本次結算週期結束後（${effectiveAt}）生效；生效前訂單仍歸原代理線。`)
  pendingAgentTransfer.value = null
  pendingAgentTransferTarget.value = null
}

function openTwoFactorAdmin(row: AgentRow) {
  if (!canManageTwoFactor(row)) {
    showNotice('warning', '無管理權限', 'Google Auth 只能管理自己代理線下的代理。')
    return
  }
  selectedTwoFactorAgent.value = row
  showTwoFactorAdmin.value = true
}

function openTwoFactorAdminPage(row: AgentRow) {
  selectedTwoFactorAgent.value = row
  detailTab.value = 'auth'
}

function canManageTwoFactor(row: AgentRow) {
  if (currentRole.value === '運營商') return true
  return row.account === identity.value.account || row.path.startsWith(`${identityAgentPath.value} >`)
}

function viewTwoFactorQr() {
  if (!selectedTwoFactorAgent.value || selectedTwoFactorAgent.value.twoFactorRequired === false) return
  showTwoFactorQr.value = true
  logs.value.unshift({ time: operationTimestamp(), type: '查看 Google Auth', actor: loginAccount.value, target: selectedTwoFactorAgent.value.account, detail: `查看${selectedTwoFactorAgent.value.twoFactor === '已啟用' ? '目前' : '待綁定'} QR Code`, before: selectedTwoFactorAgent.value.twoFactor === '已啟用' ? '已綁定' : '未綁定', after: 'QR Code 已顯示', ip: '10.20.8.15' })
}

function operationTimestamp() {
  return new Date().toLocaleString('zh-TW', { hour12: false }).replaceAll('/', '-')
}

function requestToggleAgentTwoFactor(value: boolean) {
  if (!selectedTwoFactorAgent.value || !canManageTwoFactor(selectedTwoFactorAgent.value)) return
  pendingTwoFactorRequired.value = value
  showTwoFactorToggleConfirm.value = true
}

function confirmToggleAgentTwoFactor() {
  if (!selectedTwoFactorAgent.value || pendingTwoFactorRequired.value === null) return
  const value = pendingTwoFactorRequired.value
  selectedTwoFactorAgent.value.twoFactorRequired = value
  logs.value.unshift({ time: operationTimestamp(), type: '代理 2FA 設定', actor: loginAccount.value, target: selectedTwoFactorAgent.value.account, detail: `個別 2FA 要求變更`, before: value ? '停用' : '啟用', after: value ? '啟用' : '停用', ip: '10.20.8.15' })
  showNotice('success', '代理 2FA 設定已更新', `${selectedTwoFactorAgent.value.account} 的個別 2FA 已${value ? '啟用' : '停用'}。`)
  pendingTwoFactorRequired.value = null
  showTwoFactorToggleConfirm.value = false
}

function prepareTwoFactorReset() {
  if (!selectedTwoFactorAgent.value || selectedTwoFactorAgent.value.twoFactorRequired === false) return
  showTwoFactorResetConfirm.value = true
}

function completeTwoFactorReset() {
  if (!selectedTwoFactorAgent.value) return
  selectedTwoFactorAgent.value.twoFactor = '未啟用'
  selectedTwoFactorAgent.value.twoFactorBoundAt = '尚未綁定（下次登入顯示 QR Code）'
  selectedTwoFactorAgent.value.twoFactorLastResetAt = operationTimestamp()
  logs.value.unshift({ time: operationTimestamp(), type: '重設 Google Auth', actor: loginAccount.value, target: selectedTwoFactorAgent.value.account, detail: 'QR Code 已重設，等待重新綁定', before: '已綁定', after: '未綁定', effectiveAt: '下次登入綁定流程', ip: '10.20.8.15' })
  showTwoFactorResetConfirm.value = false
  showTwoFactorAdmin.value = false
  showNotice('success', 'QR Code 已重設', selectedTwoFactorAgent.value.twoFactorRequired === false ? '目前此代理不要求 2FA；重新啟用後，代理登入時會進入首次綁定流程。' : '此代理已恢復為尚未綁定狀態，下次登入時會顯示新的綁定 QR Code。')
}

function requestToggleWithdrawalStatus(row: AgentRow) {
  if (currentRole.value !== '運營商') {
    showNotice('warning', '無操作權限', '提領狀態只能由運營商修改。')
    return
  }
  pendingWithdrawalAgent.value = row
  pendingWithdrawalEnabled.value = row.withdrawalEnabled === false
  showWithdrawalStatusConfirm.value = true
}

function confirmToggleWithdrawalStatus() {
  if (!pendingWithdrawalAgent.value || pendingWithdrawalEnabled.value === null) return
  const enabled = pendingWithdrawalEnabled.value
  pendingWithdrawalAgent.value.withdrawalEnabled = enabled
  logs.value.unshift({ time: operationTimestamp(), type: '修改提領狀態', actor: loginAccount.value, target: pendingWithdrawalAgent.value.account, detail: '提領狀態變更', before: enabled ? '不可提領' : '可提領', after: enabled ? '可提領' : '不可提領', ip: '10.20.8.15' })
  showNotice('success', '提領狀態已更新', `${pendingWithdrawalAgent.value.account} 現在為${enabled ? '可提領' : '不可提領'}。`)
  pendingWithdrawalAgent.value = null
  pendingWithdrawalEnabled.value = null
  showWithdrawalStatusConfirm.value = false
}

function openWalletAdjustment(row: AgentRow) {
  if (currentRole.value !== '運營商') {
    showNotice('warning', '無操作權限', '人工加扣款只能由運營商操作。')
    return
  }
  selectedAgent.value = row
  walletAdjustmentType.value = '加款'
  walletAdjustmentAmount.value = null
  walletAdjustmentReason.value = ''
  showWalletAdjustment.value = true
}

function submitWalletAdjustment() {
  if (!selectedAgent.value || !walletAdjustmentAmount.value || walletAdjustmentAmount.value <= 0) {
    showNotice('warning', '資料未完成', '請輸入大於 0 的調整金額。')
    return
  }
  if (!walletAdjustmentReason.value.trim()) {
    showNotice('warning', '資料未完成', '人工加扣款必須填寫調整原因。')
    return
  }
  const amount = walletAdjustmentAmount.value
  const delta = walletAdjustmentType.value === '加款' ? amount : -amount
  if (selectedAgent.value.walletBalance + delta < 0) {
    showNotice('warning', '扣款金額超過餘額', '人工扣款後代理傭金錢包不可低於 0。')
    return
  }
  showWalletAdjustmentConfirm.value = true
}

function confirmWalletAdjustment() {
  if (!selectedAgent.value || !walletAdjustmentAmount.value) return
  const amount = walletAdjustmentAmount.value
  const delta = walletAdjustmentType.value === '加款' ? amount : -amount
  const before = selectedAgent.value.walletBalance
  selectedAgent.value.walletBalance += delta
  logs.value.unshift({ time: operationTimestamp(), type: '人工加扣款', actor: loginAccount.value, target: selectedAgent.value.account, detail: `${walletAdjustmentType.value} ${selectedAgent.value.currency} ${amount.toLocaleString()}：${walletAdjustmentReason.value.trim()}`, before: `${selectedAgent.value.currency} ${before.toLocaleString()}`, after: `${selectedAgent.value.currency} ${selectedAgent.value.walletBalance.toLocaleString()}`, ip: '10.20.8.15' })
  showWalletAdjustment.value = false
  showWalletAdjustmentConfirm.value = false
  showNotice('success', '人工加扣款已完成', `${selectedAgent.value.account} 的傭金錢包已${walletAdjustmentType.value}${selectedAgent.value.currency} ${amount.toLocaleString()}。`)
}

function completeLoginTwoFactorBind() {
  if (!validateTwoFactorCode()) return
  const row = agentRows.value.find((item) => item.account === loginTwoFactorAccount.value)
  if (row) {
    row.twoFactor = '已啟用'
    row.twoFactorBoundAt = '2026-09-02 11:00:00'
  }
  showLoginTwoFactorSetup.value = false
  logs.value.unshift({ time: '2026-09-02 11:00:00', type: '完成 Google Auth 綁定', actor: loginTwoFactorAccount.value, detail: `${loginTwoFactorAccount.value} 完成首次／重設後的 Google Auth 綁定`, ip: '10.20.8.15' })
  completeLogin()
  showNotice('success', 'Google Auth 綁定完成', '驗證碼已確認，現在可以進入代理後台。')
}

function toggleTwoFactorGlobal(value: boolean) {
  if (currentRole.value !== '運營商') return
  twoFactorGlobalEnabled.value = value
  logs.value.unshift({ time: '2026-08-31 10:47:00', type: value ? '啟用 2FA 全域設定' : '停用 2FA 全域設定', actor: loginAccount.value, detail: `新建立代理的 Google Auth 預設已${value ? '啟用' : '停用'}`, ip: '10.20.8.15' })
  showNotice('success', '2FA 全域設定已更新', value ? '之後新建立的代理預設需要 Google Auth；既有代理設定不變。' : '之後新建立的代理預設不要求 Google Auth；既有代理設定不變。')
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
  const config = plan ? planConfig(plan) : defaultCommissionConfig()
  editingPlan.value = plan ?? null
  planName.value = plan?.name ?? ''
  planParentAccount.value = plan?.parentAccount ?? `${identity.value.account}（${identity.value.label}）`
  planModel.value = plan?.model ?? '有效投注額'
  planCycle.value = plan?.cycle ?? '每週'
  planRate.value = plan?.allocationRate ?? 4
  planDescription.value = plan?.description ?? ''
  shareRate.value = config.share.shareRate
  adminCostRate.value = config.share.adminCostRate
  negativeMode.value = config.share.negativeMode
  offsetType.value = config.share.offsetType
  offsetLimit.value = config.share.offsetLimit
  rebateRate.value = config.rebate.rebateRate
  showPlanEditor.value = true
}

function savePlan() {
  if (!planName.value.trim() || !planParentAccount.value.trim() || planRate.value === null) {
    showNotice('warning', '方案資料未完成', '請填寫方案名稱、建立代理與默認可分配點數。')
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
    const childMinimum = Math.max(...planAppliedAgents(editingPlan.value).map((row) => row.point ?? 0), 0)
    if (planRate.value < childMinimum) {
      showNotice('warning', '低於下級方案點數', `目前已有下級使用此方案，點數不可低於 ${childMinimum}%。`)
      return
    }
  }
  if (planModel.value === '輸贏' && [shareRate.value, adminCostRate.value, offsetLimit.value].some((value) => value === null || value < 0)) {
    showNotice('warning', '佔成設定未完成', '請完成佔成比例、行政費率與負數沖銷額度。')
    return
  }
  if ((planModel.value === '有效投注額' || planModel.value === '儲值') && (rebateRate.value === null || rebateRate.value < 0)) {
    showNotice('warning', '傭金比例未完成', `請填寫${planModel.value === '儲值' ? '成功儲值' : '有效投注額'}比例。`)
    return
  }
  const config: CommissionConfig = {
    share: { shareRate: shareRate.value ?? 0, adminCostRate: adminCostRate.value ?? 0, negativeMode: negativeMode.value, offsetType: offsetType.value, offsetLimit: offsetLimit.value ?? 0 },
    rebate: { rebateRate: rebateRate.value ?? 0 },
  }
  if (editingPlan.value) {
    editingPlan.value.name = planName.value.trim()
    editingPlan.value.parentAccount = planParentAccount.value.trim()
    editingPlan.value.model = planModel.value
    editingPlan.value.cycle = planCycle.value
    editingPlan.value.allocationRate = planRate.value
    editingPlan.value.description = planDescription.value.trim()
    editingPlan.value.config = config
    showNotice('success', '傭金方案已更新', `「${editingPlan.value.name}」設定已儲存。`)
  } else {
    const id = `PLAN-${String(commissionPlans.value.length + 1).padStart(3, '0')}`
    commissionPlans.value.push({ id, name: planName.value.trim(), createdBy: `${identity.value.account}（${identity.value.label}）`, parentAccount: planParentAccount.value.trim(), model: planModel.value, cycle: planCycle.value, allocationRate: planRate.value, description: planDescription.value.trim() || '依此方案規則產生代理傭金。', assignedCount: 0, status: '啟用', config })
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

function openPlayer(row: PlayerRow) {
  selectedPlayer.value = row
  playerDetailTab.value = 'wallet'
  showPlayerDetail.value = true
}

function requireOperator(action: string) {
  if (canOperatePlayers.value) return true
  showNotice('warning', '目前代理僅可查看', `代理不可${action}玩家，需由運營商後台操作。`)
  return false
}

function editPlayer(row: PlayerRow) {
  if (!requireOperator('編輯資料')) return
  selectedPlayer.value = row
  playerDisplayNameDraft.value = row.displayName
  showPlayerEdit.value = true
}

function savePlayerEdit() {
  if (!selectedPlayer.value || !playerDisplayNameDraft.value.trim()) {
    showNotice('warning', '資料未完成', '顯示名稱不可為空白。')
    return
  }
  selectedPlayer.value.displayName = playerDisplayNameDraft.value.trim()
  logs.value.unshift({ time: '2026-08-31 10:42:00', type: '編輯玩家資料', actor: loginAccount.value, detail: `${selectedPlayer.value.account} 顯示名稱已更新`, ip: '10.20.8.15' })
  showPlayerEdit.value = false
  showNotice('success', '玩家資料已更新', `${selectedPlayer.value.account} 的顯示名稱已儲存。`)
}

function managePlayerStatus(row: PlayerRow) {
  if (!requireOperator('修改狀態')) return
  selectedPlayer.value = row
  playerStatusDraft.value = row.status
  showPlayerStatus.value = true
}

function savePlayerStatus() {
  if (!selectedPlayer.value) return
  const oldStatus = selectedPlayer.value.status
  selectedPlayer.value.status = playerStatusDraft.value
  logs.value.unshift({ time: '2026-08-31 10:43:00', type: '玩家狀態管理', actor: loginAccount.value, detail: `${selectedPlayer.value.account} 狀態 ${oldStatus} → ${playerStatusDraft.value}`, ip: '10.20.8.15' })
  showPlayerStatus.value = false
  showNotice('success', '玩家狀態已更新', `${selectedPlayer.value.account} 現在為${playerStatusDraft.value}。`)
}

function transferPlayer(row: PlayerRow) {
  if (!requireOperator('轉線')) return
  selectedPlayer.value = row
  playerTransferTarget.value = ''
  showPlayerTransfer.value = true
}

function savePlayerTransfer() {
  if (!selectedPlayer.value || !playerTransferTarget.value) {
    showNotice('warning', '尚未選擇代理線', '請選擇新的代理線後再送出。')
    return
  }
  const oldPath = selectedPlayer.value.path
  const account = selectedPlayer.value.account
  selectedPlayer.value.path = `${playerTransferTarget.value} > ${account}`
  logs.value.unshift({ time: '2026-08-31 10:44:00', type: '玩家轉線', actor: loginAccount.value, detail: `${account} 已由 ${oldPath} 轉入 ${selectedPlayer.value.path}`, ip: '10.20.8.15' })
  showPlayerTransfer.value = false
  showNotice('success', '玩家轉線已建立', '轉線生效時間依平台排程規則處理。')
}

function playerStatusType(status: PlayerRow['status']) {
  const map: Record<PlayerRow['status'], 'success' | 'warning' | 'error' | 'info'> = { 啟用: 'success', 鎖定: 'warning', 凍結: 'info', 暫停: 'error' }
  return map[status]
}

function exportMessage(label: string) {
  showNotice('success', `${label}已準備`, '已套用目前篩選條件，資料已準備完成。')
}

const playerColumns = computed(() => [
  { title: '玩家 ID', key: 'id', width: 110 },
  { title: '玩家帳號', key: 'account', width: 150, render: (row: PlayerRow) => h(NButton, { text: true, type: 'primary', size: 'small', class: 'player-account-link', onClick: () => openPlayer(row) }, { default: () => row.account }) },
  { title: '顯示名稱', key: 'displayName', width: 140 },
  { title: '標籤', key: 'tags', width: 150, render: (row: PlayerRow) => h('div', { class: 'table-tags' }, row.tags.map((tag) => h(NTag, { size: 'small', round: true, type: tag === '風控關注' ? 'warning' : 'default' }, { default: () => tag }))) },
  { title: 'RTP', key: 'rtp', width: 80, render: (row: PlayerRow) => h('span', { class: row.rtp < 100 ? 'positive' : 'negative' }, `${row.rtp}%`) },
  { title: 'VIP 等級', key: 'vipLevel', width: 100 },
  { title: '帳號狀態', key: 'status', width: 100, render: (row: PlayerRow) => h(NTag, { type: playerStatusType(row.status), bordered: false, round: true }, { default: () => row.status }) },
  { title: '在線狀態', key: 'isOnline', width: 100, render: (row: PlayerRow) => h(NBadge, { dot: true, type: row.isOnline ? 'success' : 'default' }, () => row.isOnline ? '在線' : '離線') },
  { title: '註冊時間', key: 'registerAt', width: 170 },
  { title: '操作', key: 'actions', width: canOperatePlayers.value ? 190 : 120, fixed: 'right' as const, render: (row: PlayerRow) => h('div', { class: 'table-actions' }, [
    h(NButton, { size: 'small', secondary: true, type: 'primary', onClick: () => openPlayer(row) }, { default: () => '查看詳情' }),
    canOperatePlayers.value ? h(NButton, { size: 'small', quaternary: true, onClick: () => managePlayerStatus(row) }, { default: () => '狀態管理' }) : null,
  ]) },
])
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
            <NFormItem label="登入角色">
              <NInput value="代理" readonly />
            </NFormItem>
            <NButton type="primary" block size="large" @click="login">登入代理後台</NButton>
          </NForm>
        </NCard>
      </div>

      <NLayout v-else has-sider class="app-shell">
        <NLayoutSider bordered :width="250" class="sidebar">
          <div class="side-brand"><span class="brand-mark small">Y</span><div><strong>YOTA</strong><span>AGENT CONSOLE</span></div></div>
          <div class="agent-chip"><div class="avatar">{{ identity.label.slice(0, 1) }}</div><div><strong>{{ identity.account }}</strong><span>{{ identity.label }} · {{ identity.currency }}</span></div><ChevronDownOutline class="chip-chevron" /></div>
          <div class="role-switcher"><span>登入角色</span><div class="role-readonly">代理</div></div>
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
                <NCard><NStatistic :label="currentRole === '運營商' ? '平台一級代理' : '下級代理'" :value="currentRole === '運營商' ? 8 : 16" suffix=" 位" /></NCard>
                <NCard><NStatistic label="下級玩家" :value="428" suffix=" 位" /></NCard>
              </div>
              <div class="dashboard-grid">
                <NCard title="傭金趨勢（近 7 期）"><div class="sparkline"><span v-for="(height, index) in [36, 48, 43, 70, 58, 82, 76]" :key="index" :style="{ height: `${height}%` }" /></div><div class="chart-labels"><span>週一</span><span>週二</span><span>週三</span><span>週四</span><span>週五</span><span>週六</span><span>今日</span></div></NCard>
                <NCard title="代理線摘要"><div class="summary-list"><div><span>{{ currentRole === '運營商' ? '啟用中的代理線' : '直屬代理' }}</span><strong>{{ currentRole === '運營商' ? '12 條' : '3 位' }}</strong></div><div><span>目前可見層級</span><strong>{{ currentRole === '運營商' ? '無限層' : '4 層' }}</strong></div><div><span>本期有效投注</span><strong>TWD 1,284,600</strong></div><div><span>本期產生傭金</span><strong class="positive">TWD 28,460</strong></div></div></NCard>
              </div>
              <NCard title="最近操作" class="recent-card"><div v-for="log in logs.slice(0, 3)" :key="`${log.time}-${log.detail}`" class="recent-row"><div class="log-dot" /><div><strong>{{ log.type }}</strong><span>{{ log.detail }}</span></div><time>{{ log.time }}</time></div></NCard>
            </section>

            <section v-else-if="activeKey === 'agents'">
              <template v-if="!showAgentDetail">
              <div class="section-head agent-section-head"><div><h1>代理管理</h1><p class="muted">以代理為主體查看個人摘要、代理層級、代理線、幣別、傭金模式與套用方案；代理操作權限一致。</p></div><div class="agent-head-side"><NButton type="primary" @click="showCreateAgent = true">＋ {{ createTitle }}</NButton><div class="agent-focus-grid"><NCard class="focus-card"><div><span>目前登入角色</span><strong>{{ identity.account }}</strong><small>{{ identity.label }} · {{ identity.currency }} · 可查看自身代理線</small></div></NCard><NCard class="focus-card"><div><span>傭金方案</span><strong>三種模式可配置</strong><small>儲值／有效投注額／輸贏；同一代理線固定使用一種模式。</small></div></NCard><NCard class="focus-card"><div><span>代理線狀態</span><strong>3 條 · 幣別一致</strong><small>同一條代理線內代理與玩家必須使用相同幣別。</small></div></NCard></div></div></div>
              <div class="role-banner"><strong>{{ identity.label }}可執行範圍</strong><span>{{ currentRole === '運營商' ? '建立一級代理、設定傭金模式與結算週期，查看全平台代理網絡。' : currentRole === '總代理' ? '建立直屬代理、套用同一代理線傭金模式，查看全部下級與各代理線報表。' : '建立直屬下級代理、套用上級提供的傭金模式，僅查看自身代理線資料。' }}</span></div>
              <NCard v-if="currentRole === '運營商'" title="2FA 全域設定" class="security-card agent-security-card"><div class="security-setting"><div><strong>新代理預設啟用 Google Auth</strong><p class="modal-help">設定新代理首次登入時是否需要綁定 Google Auth。僅運營商可修改。</p></div><NSwitch :value="twoFactorGlobalEnabled" @update:value="toggleTwoFactorGlobal" /></div></NCard>
              <div class="table-section-label"><strong>篩選欄位</strong><span>設定條件後點擊搜尋；上級代理僅能選擇目前角色可查看的代理線</span></div><NCard class="filter-card filter-card-emphasis"><div class="filter-row filter-row-advanced"><div class="filter-field-group filter-scope-group"><label>查詢範圍</label><div class="scope-toggle"><button :class="{ active: agentScopeDraft === 'direct' }" @click="agentScopeDraft = 'direct'">只看直屬</button><button :class="{ active: agentScopeDraft === 'all' }" @click="agentScopeDraft = 'all'">查看全下級</button></div></div><div class="filter-field-group filter-account-group"><label>代理帳號</label><NInput v-model:value="agentAccountDraft" clearable placeholder="搜尋代理帳號" class="filter-field filter-account"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div><div class="filter-field-group"><label>註冊 IP</label><NInput v-model:value="agentRegisterIpDraft" clearable placeholder="輸入註冊 IP" class="filter-field"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div><div class="filter-field-group filter-date-group"><label>註冊時間</label><NDatePicker v-model:value="agentRegisterDateRangeDraft" type="daterange" clearable format="yyyy-MM-dd" start-placeholder="註冊起日" end-placeholder="註冊迄日" placeholder="選擇時間區段" class="filter-date" /></div><div class="filter-field-group filter-parent-group"><label>上級代理</label><NSelect v-model:value="agentParentDraft" clearable placeholder="選擇上級代理" :options="agentParentOptions" class="filter-parent" /></div><div class="filter-field-group"><label>代理層級</label><NSelect v-model:value="agentLevelDraft" clearable placeholder="全部層級" :options="[{label:'一級代理',value:'一級代理'},{label:'二級代理',value:'二級代理'},{label:'三級代理',value:'三級代理'},{label:'四級代理',value:'四級代理'},{label:'五級代理',value:'五級代理'}]" class="filter-select" /></div><div class="filter-field-group"><label>傭金模式</label><NSelect v-model:value="agentCommissionDraft" clearable placeholder="全部模式" :options="[{label:'儲值',value:'儲值'},{label:'有效投注額',value:'有效投注額'},{label:'輸贏',value:'輸贏'}]" class="filter-select" /></div><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="agentStatusDraft" clearable placeholder="全部狀態" :options="[{label:'啟用',value:'啟用'},{label:'停用',value:'停用'}]" class="filter-select" /></div><NButton type="primary" class="filter-search-button" @click="applyAgentFilters"><template #icon><NIcon><SearchOutline /></NIcon></template>搜尋</NButton></div></NCard>
               <div class="table-section-label"><strong>代理資料</strong><span>共 {{ filteredAgents.length }} 筆；帳號查詢僅搜尋目前角色可見的下級代理線</span></div><NCard :bordered="false" class="table-card table-card-emphasis"><div class="agent-table-wrap"><div class="agent-table"><div class="agent-table-head"><span>代理帳號</span><span>層級</span><span>幣別</span><span>傭金模式</span><span>結算週期</span><span>下級數</span><span>傭金錢包餘額</span><span>狀態</span><span>傭金方案</span><span>模式設定</span><span>操作</span></div><div v-for="row in filteredAgents" :key="row.id" class="agent-table-row"><button class="account-link" @click="openAgent(row)">{{ row.account }}</button><span>{{ row.level }}</span><span>{{ row.currency }}</span><NTag size="small" :type="row.model === '輸贏' ? 'warning' : 'success'" round>{{ normalizeCommissionMode(row.model) }}</NTag><span>{{ row.cycle ?? '沿用上級' }}</span><span>{{ row.children }}</span><strong class="wallet-value">{{ row.currency }} {{ row.walletBalance.toLocaleString() }}</strong><NTag size="small" :type="row.status === '啟用' ? 'success' : 'error'" round>{{ row.status }}</NTag><div><strong>{{ planForAgent(row)?.name ?? '未套用' }}</strong><small class="mode-summary">{{ agentPlanSummary(row) }}</small></div><span class="mode-summary">{{ normalizeCommissionMode(row.model) }}</span><div class="table-actions"><NButton quaternary size="small" @click="openAgent(row)">查看詳情</NButton><NButton quaternary size="small" @click="deactivateAgent(row)">狀態管理</NButton></div></div></div></div><div class="table-hint">傭金錢包餘額為目前可提領金額；傭金方案不使用推廣碼，代理玩家推廣碼可於代理詳情複製。</div></NCard>
              </template>
              <template v-else-if="selectedAgent">
                  <div class="detail-layout"><aside class="profile-panel"><div class="profile-panel-head"><h3>基本資料</h3><NSpace><NButton v-if="canManageAgent(selectedAgent)" size="small" secondary @click="editAgent(selectedAgent)">編輯資料</NButton><NButton v-if="canManageAgent(selectedAgent)" size="small" :type="selectedAgent.status === '啟用' ? 'warning' : 'primary'" secondary @click="deactivateAgent(selectedAgent)">{{ selectedAgent.status === '啟用' ? '停用代理' : '啟用代理' }}</NButton></NSpace></div><div class="profile-avatar">{{ selectedAgent.displayName?.slice(0, 1) || selectedAgent.account.slice(0, 1).toUpperCase() }}</div><h2>{{ selectedAgent.displayName || selectedAgent.account }}</h2><p class="profile-id">UID：{{ selectedAgent.uid || selectedAgent.id }}</p><div class="profile-status"><NTag :type="selectedAgent.status === '啟用' ? 'success' : 'error'" size="small" round>{{ selectedAgent.status }}</NTag><NTag type="info" size="small" round>{{ selectedAgent.level }}</NTag></div><div class="profile-fields"><div><span>登入帳號</span><strong>{{ selectedAgent.account }}</strong></div><div><span>登入密碼</span><strong>••••••••</strong></div><div><span>帳號類型</span><strong>{{ selectedAgent.level }}</strong></div><div><span>代理 UID</span><strong>{{ selectedAgent.uid || selectedAgent.id }}</strong></div><div><span>真實姓名</span><strong>{{ selectedAgent.displayName || '未填寫' }}</strong></div><div><span>手機號碼</span><strong>{{ maskPhone(selectedAgent.phone) }}</strong></div><div><span>Email</span><strong>{{ maskEmail(selectedAgent.email) }}</strong></div><div><span>代理傭金錢包餘額</span><strong class="positive">{{ selectedAgent.currency }} {{ selectedAgent.walletBalance.toLocaleString() }}</strong></div><div><span>幣別</span><strong>{{ selectedAgent.currency }}</strong></div><div><span>註冊時間</span><strong>{{ selectedAgent.registerAt || '尚未記錄' }}</strong></div></div></aside><div class="detail-workspace"><div class="detail-page-head"><div><button class="back-link" @click="showAgentDetail = false">← 返回代理管理</button><h1>代理詳情 · {{ selectedAgent.account }}</h1><p class="muted">查看代理帳號、代理線、傭金錢包與安全設定。</p></div><NTag type="info" round>{{ selectedAgent.level }}</NTag></div>
                  <div class="detail-tabs"><button :class="{ active: detailTab === 'wallet' }" @click="detailTab = 'wallet'">即時資料</button><button :class="{ active: detailTab === 'auth' }" @click="openTwoFactorAdminPage(selectedAgent)">Google Auth</button><button :class="{ active: detailTab === 'relationship' }" @click="detailTab = 'relationship'">代理報表</button><button :class="{ active: detailTab === 'commission' }" @click="detailTab = 'commission'">傭金方案</button><button :class="{ active: detailTab === 'withdrawals' }" @click="detailTab = 'withdrawals'">傭金帳變紀錄</button><button :class="{ active: detailTab === 'logs' }" @click="detailTab = 'logs'">操作紀錄</button></div>
                <div v-if="detailTab === 'basic'" class="agent-detail-grid detail-page-card"><div><span>登入帳號</span><strong>{{ selectedAgent.account }}</strong></div><div><span>代理傭金錢包餘額</span><strong class="positive">{{ selectedAgent.currency }} {{ selectedAgent.walletBalance.toLocaleString() }}</strong></div><div><span>登入密碼</span><strong>••••••••</strong><p class="modal-help">只有直屬上級或運營商可更改。</p></div><div><span>帳號類型</span><strong>{{ selectedAgent.level }}</strong></div><div><span>代理 UID（系統生成）</span><strong>{{ selectedAgent.uid || selectedAgent.id }}</strong></div><div><span>真實姓名</span><strong>{{ selectedAgent.displayName || '未填寫' }}</strong></div><div><span>手機號碼</span><strong>{{ maskPhone(selectedAgent.phone) }}</strong></div><div><span>Email</span><strong>{{ maskEmail(selectedAgent.email) }}</strong></div><div><span>幣別</span><strong>{{ selectedAgent.currency }}</strong></div><div><span>目前狀態</span><NTag :type="selectedAgent.status === '啟用' ? 'success' : 'error'" round>{{ selectedAgent.status }}</NTag></div></div>
                  <div v-else-if="detailTab === 'auth'" class="auth-page-card"><div class="auth-page-heading"><div><h3>Google Auth 雙重驗證</h3><p>在本分頁管理此代理是否需要 2FA、查看目前 QR Code，或將綁定狀態重設。</p></div></div><div class="auth-toggle-row"><div><strong>此代理要求 2FA</strong><span>{{ selectedAgent.twoFactorRequired === false ? '登入時不需要 Google Auth' : '登入時需要 Google Auth' }}</span></div><NSwitch :value="selectedAgent.twoFactorRequired !== false" :disabled="!canManageTwoFactor(selectedAgent)" size="large" @update:value="requestToggleAgentTwoFactor"><template #checked>啟用</template><template #unchecked>不啟用</template></NSwitch></div><div class="auth-page-grid"><div><span>個別 2FA 要求</span><strong>{{ selectedAgent.twoFactorRequired === false ? '停用' : '啟用' }}</strong></div><div><span>綁定狀態</span><strong>{{ selectedAgent.twoFactor === '已啟用' ? '已綁定' : '未綁定' }}</strong></div><div><span>綁定時間</span><strong>{{ selectedAgent.twoFactorBoundAt || '尚未綁定' }}</strong></div></div><div class="auth-page-actions"><NButton v-if="canManageTwoFactor(selectedAgent)" secondary :disabled="selectedAgent.twoFactorRequired === false" @click="viewTwoFactorQr">查看目前 QR Code</NButton><NButton v-if="canManageTwoFactor(selectedAgent)" type="warning" :disabled="selectedAgent.twoFactorRequired === false" @click="prepareTwoFactorReset">重設 QR Code</NButton><span>{{ selectedAgent.twoFactorRequired === false ? '目前未啟用 2FA，相關 QR Code 操作不可用' : selectedAgent.twoFactor === '已啟用' ? '目前已綁定，可查看或重設 QR Code' : '目前未綁定，可查看下一次登入使用的 QR Code，也可再次重設' }}</span></div><p v-if="selectedAgent.twoFactorLastResetAt" class="auth-last-reset">上次重設時間：{{ selectedAgent.twoFactorLastResetAt }}</p></div>
                 <div v-else-if="detailTab === 'wallet'" class="agent-detail-grid detail-page-card"><div v-if="currentRole === '運營商'" class="full wallet-action-row"><div><span>人工資金調整</span><strong>人工加扣款</strong><small>資金帳變不可轉帳；可由運營商人工加扣款，所有調整都會留下操作紀錄。</small></div><NButton type="primary" @click="openWalletAdjustment(selectedAgent)">人工加扣款</NButton></div><div><span>代理傭金錢包餘額</span><strong class="positive">{{ selectedAgent.currency }} {{ selectedAgent.walletBalance.toLocaleString() }}</strong></div><div><span>待結算傭金</span><strong>{{ selectedAgent.currency }} 12,680</strong></div><div><span>本期產生傭金</span><strong>{{ selectedAgent.currency }} 28,460</strong></div><div class="wallet-status-row"><span>可提領狀態</span><div><NTag :type="selectedAgent.withdrawalEnabled === false ? 'error' : 'success'" round>{{ selectedAgent.withdrawalEnabled === false ? '不可提領' : '可提領' }}</NTag><NButton v-if="currentRole === '運營商'" size="small" secondary @click="requestToggleWithdrawalStatus(selectedAgent)">{{ selectedAgent.withdrawalEnabled === false ? '設為可提領' : '設為不可提領' }}</NButton></div></div><div><span>註冊時間</span><strong>{{ selectedAgent.registerAt || '尚未記錄' }}</strong></div><div><span>最後登入時間</span><strong>{{ selectedAgent.lastLoginAt || '尚未登入' }}</strong></div><div><span>最後登入 IP</span><strong>{{ selectedAgent.lastLoginIp || '尚未記錄' }}</strong></div><div class="full wallet-review-note"><span>提領審核</span><strong>提領申請需由運營商審核</strong></div></div>
                  <div v-else-if="detailTab === 'relationship'" class="agent-info-panel detail-page-card"><div class="agent-info-header"><div><span>完整樹狀路徑</span><strong>{{ displayAgentPath(selectedAgent.path) }}</strong><p v-if="selectedAgent.pendingTransferTargetPath" class="pending-transfer">預約新代理線：{{ displayAgentPath(selectedAgent.pendingTransferTargetPath) }}（{{ selectedAgent.pendingTransferEffectiveAt }} 生效）</p></div><NButton v-if="currentRole === '運營商' || currentRole === '總代理'" class="agent-transfer-button" type="primary" :disabled="selectedAgent.path.split(' > ').length === 1" @click="openAgentTransfer(selectedAgent)">更換代理線</NButton></div><div class="relationship-report-rows"><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('totalAgents')"><span>總下級（含直屬）代理人數</span><strong>{{ selectedAgent.totalAgentCount ?? selectedAgent.children }} 位</strong><em>{{ relationshipExpanded.totalAgents ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.totalAgents" class="relationship-drawer"><div v-for="row in agentRows.filter((item) => item.path.startsWith(`${selectedAgent.path} >`))" :key="row.id"><span>{{ row.account }}</span><small>{{ row.level }} · {{ row.path }}</small></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('totalAgentMoney')"><span>總下級（含直屬）代理總儲值金額</span><strong>{{ selectedAgent.currency }} {{ (selectedAgent.totalDeposit ?? 0).toLocaleString() }}</strong><em>{{ relationshipExpanded.totalAgentMoney ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.totalAgentMoney" class="relationship-drawer"><div v-for="row in agentRows.filter((item) => item.path.startsWith(`${selectedAgent.path} >`))" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ (row.totalDeposit ?? 0).toLocaleString() }}</strong></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('totalAgentBet')"><span>總下級（含直屬）代理總有效投注</span><strong>{{ selectedAgent.currency }} {{ (selectedAgent.totalEffectiveBet ?? 0).toLocaleString() }}</strong><em>{{ relationshipExpanded.totalAgentBet ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.totalAgentBet" class="relationship-drawer"><div v-for="row in agentRows.filter((item) => item.path.startsWith(`${selectedAgent.path} >`))" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ (row.totalEffectiveBet ?? 0).toLocaleString() }}</strong></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('directAgents')"><span>直屬下級代理人數</span><strong>{{ selectedAgent.directAgentCount ?? selectedAgent.children }} 位</strong><em>{{ relationshipExpanded.directAgents ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.directAgents" class="relationship-drawer"><div v-for="row in agentRows.filter((item) => item.path === `${selectedAgent.path} > ${item.account}`)" :key="row.id"><span>{{ row.account }}</span><small>{{ row.level }} · {{ row.status }}</small></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('directAgentMoney')"><span>直屬下級代理總儲值金額</span><strong>{{ selectedAgent.currency }} {{ (selectedAgent.directAgentCount ? Math.round((selectedAgent.totalDeposit ?? 0) / selectedAgent.directAgentCount) : 0).toLocaleString() }}</strong><em>{{ relationshipExpanded.directAgentMoney ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.directAgentMoney" class="relationship-drawer"><div v-for="row in agentRows.filter((item) => item.path === `${selectedAgent.path} > ${item.account}`)" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ (row.totalDeposit ?? 0).toLocaleString() }}</strong></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('directAgentBet')"><span>直屬下級代理總有效投注</span><strong>{{ selectedAgent.currency }} {{ (selectedAgent.directAgentCount ? Math.round((selectedAgent.totalEffectiveBet ?? 0) / selectedAgent.directAgentCount) : 0).toLocaleString() }}</strong><em>{{ relationshipExpanded.directAgentBet ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.directAgentBet" class="relationship-drawer"><div v-for="row in agentRows.filter((item) => item.path === `${selectedAgent.path} > ${item.account}`)" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ (row.totalEffectiveBet ?? 0).toLocaleString() }}</strong></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('totalPlayers')"><span>總下級（含直屬）玩家人數</span><strong>{{ selectedAgent.totalPlayerCount ?? 0 }} 位</strong><em>{{ relationshipExpanded.totalPlayers ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.totalPlayers" class="relationship-drawer"><div v-for="row in playerRows.filter((item) => item.path.startsWith(`${selectedAgent.path} >`))" :key="row.id"><span>{{ row.account }}</span><small>{{ row.path }}</small></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('totalPlayerMoney')"><span>總下級（含直屬）玩家總儲值金額</span><strong>{{ selectedAgent.currency }} {{ (selectedAgent.totalDeposit ?? 0).toLocaleString() }}</strong><em>{{ relationshipExpanded.totalPlayerMoney ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.totalPlayerMoney" class="relationship-drawer"><div v-for="row in playerRows.filter((item) => item.path.startsWith(`${selectedAgent.path} >`))" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ row.deposit.toLocaleString() }}</strong></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('totalPlayerBet')"><span>總下級（含直屬）玩家總有效投注</span><strong>{{ selectedAgent.currency }} {{ (selectedAgent.totalEffectiveBet ?? 0).toLocaleString() }}</strong><em>{{ relationshipExpanded.totalPlayerBet ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.totalPlayerBet" class="relationship-drawer"><div v-for="row in playerRows.filter((item) => item.path.startsWith(`${selectedAgent.path} >`))" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ row.bet.toLocaleString() }}</strong></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('directPlayers')"><span>直屬下級玩家人數</span><strong>{{ selectedAgent.directPlayerCount ?? 0 }} 位</strong><em>{{ relationshipExpanded.directPlayers ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.directPlayers" class="relationship-drawer"><div v-for="row in playerRows.filter((item) => item.path.split(' > ').length === selectedAgent.path.split(' > ').length + 1)" :key="row.id"><span>{{ row.account }}</span><small>{{ row.path }}</small></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('directPlayerMoney')"><span>直屬下級玩家總儲值金額</span><strong>{{ selectedAgent.currency }} {{ selectedAgent.directPlayerCount ? Math.round((selectedAgent.totalDeposit ?? 0) / selectedAgent.directPlayerCount).toLocaleString() : '0' }}</strong><em>{{ relationshipExpanded.directPlayerMoney ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.directPlayerMoney" class="relationship-drawer"><div v-for="row in playerRows.filter((item) => item.path.split(' > ').length === selectedAgent.path.split(' > ').length + 1)" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ row.deposit.toLocaleString() }}</strong></div></div></div><div class="relationship-report-row"><button class="relationship-row-trigger" @click="toggleRelationshipSection('directPlayerBet')"><span>直屬下級玩家總有效投注</span><strong>{{ selectedAgent.currency }} {{ selectedAgent.directPlayerCount ? Math.round((selectedAgent.totalEffectiveBet ?? 0) / selectedAgent.directPlayerCount).toLocaleString() : '0' }}</strong><em>{{ relationshipExpanded.directPlayerBet ? '收闔' : '展開' }}⌄</em></button><div v-if="relationshipExpanded.directPlayerBet" class="relationship-drawer"><div v-for="row in playerRows.filter((item) => item.path.split(' > ').length === selectedAgent.path.split(' > ').length + 1)" :key="row.id"><span>{{ row.account }}</span><strong>{{ row.currency }} {{ row.bet.toLocaleString() }}</strong></div></div></div></div><div class="agent-info-rule"><strong>代理轉線規則</strong><p>僅限同幣別代理線；一級代理不可轉線。轉線於本次結算週期結束後生效，生效前訂單歸原代理線，生效後新訂單歸新代理線，不回溯重算歷史傭金。</p></div></div>
                <div v-else-if="detailTab === 'commission'" class="agent-plan-overview detail-page-card"><div class="agent-plan-overview-head"><div><span>上級給予此代理的方案</span><strong>{{ selectedAgentPlan?.name || '未套用方案' }}</strong></div><NTag type="info" round>唯讀查看</NTag></div><div v-if="selectedAgentPlan" class="agent-plan-overview-grid"><div><span>方案名稱</span><strong>{{ selectedAgentPlan.name }}</strong></div><div><span>建立代理</span><strong>{{ selectedAgentPlan.createdBy || selectedAgentPlan.parentAccount }}</strong></div><div><span>傭金模式</span><strong>{{ selectedAgentPlan.model }}</strong></div><div><span>結算週期</span><strong>{{ selectedAgentPlan.cycle }}</strong></div><div><span>默認可分配點數</span><strong>{{ selectedAgentPlan.allocationRate }}%</strong></div><div><span>方案規則</span><strong>{{ selectedAgentPlan.description }}</strong></div></div><div class="agent-plan-overview-section"><div class="table-section-label"><strong>此代理建立給直屬下級的方案</strong><span>僅顯示方案與套用代理，新增／修改請至傭金方案設定</span></div><div v-for="plan in selectedAgentCreatedPlans" :key="plan.id" class="agent-downline-plan-row"><div><strong>{{ plan.name }}</strong><small>{{ plan.model }} · {{ plan.cycle }}</small></div><strong>{{ plan.allocationRate }}%</strong><span>套用 {{ planAppliedAgents(plan).length }} 位</span><div class="agent-downline-plan-agents"><NTag v-for="agent in planAppliedAgents(plan)" :key="agent.id" size="small" round :type="agent.status === '啟用' ? 'success' : 'error'">{{ agent.account }}</NTag><span v-if="!planAppliedAgents(plan).length" class="muted">尚無套用代理</span></div></div><p v-if="!selectedAgentCreatedPlans.length" class="modal-help">此代理尚未建立給直屬下級的方案。</p></div></div>
                 <div v-else-if="detailTab === 'withdrawals'" class="agent-withdrawal-panel detail-page-card"><div class="table-section-label"><strong>傭金帳變紀錄</strong><span>整合傭金提領與傭金發放報表；本頁僅供查看詳情</span></div><NCard class="filter-card filter-card-emphasis agent-withdrawal-filter"><div class="filter-row filter-row-advanced"><div class="filter-field-group"><label>帳變類型</label><NSelect v-model:value="agentWithdrawalType" clearable placeholder="全部類型" :options="[{label:'傭金發放',value:'傭金發放'},{label:'傭金提領',value:'傭金提領'}]" class="filter-select" /></div><div class="filter-field-group filter-date-group"><label>發生時間</label><NDatePicker v-model:value="agentWithdrawalDateRange" type="daterange" clearable format="yyyy-MM-dd" start-placeholder="起始日期" end-placeholder="結束日期" placeholder="選擇時間區段" class="filter-date" /></div><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="agentWithdrawalStatus" clearable placeholder="全部狀態" :options="[{label:'待處理',value:'待處理'},{label:'處理中',value:'處理中'},{label:'成功',value:'成功'},{label:'失敗',value:'失敗'}]" class="filter-select" /></div><div class="filter-field-group filter-account-group"><label>關鍵字／單號</label><NInput v-model:value="agentWithdrawalKeyword" clearable placeholder="搜尋單號、銀行、方案或備註" class="filter-field filter-log-search"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div></div></NCard><NCard :bordered="false" class="table-card withdrawal-record-card"><div class="withdrawal-record-head"><span>發生時間</span><span>帳變類型</span><span>帳變單號</span><span>結算週期</span><span>帳變金額</span><span>餘額</span><span>狀態</span><span>操作</span></div><div v-for="record in filteredAgentWithdrawalRecords" :key="record.id" class="withdrawal-record-row account-record-row"><time>{{ record.applyAt }}</time><NTag size="small" round :type="record.recordType === '傭金發放' ? 'info' : 'warning'">{{ record.recordType }}</NTag><code>{{ record.id }}</code><span>{{ record.settlePeriod }}</span><strong :class="record.recordType === '傭金發放' ? 'positive' : 'negative'">{{ record.recordType === '傭金發放' ? '+' : '-' }}{{ selectedAgent.currency }} {{ record.netAmount.toLocaleString() }}</strong><strong>{{ selectedAgent.currency }} {{ record.balanceAfter.toLocaleString() }}</strong><NTag size="small" round :type="record.status === '成功' ? 'success' : record.status === '失敗' ? 'error' : 'warning'">{{ record.status }}</NTag><NButton size="small" quaternary @click="openCommissionRecordDetail(record)">查看詳情</NButton></div><p v-if="!filteredAgentWithdrawalRecords.length" class="modal-help">目前沒有符合條件的傭金帳變紀錄。</p></NCard></div>
                <div v-else class="agent-log-panel detail-page-card"><div class="table-section-label"><strong>操作紀錄</strong><span>顯示此代理的登入、設定、轉線與提領狀態異動</span></div><NCard class="filter-card filter-card-emphasis agent-detail-log-filter"><div class="filter-row filter-row-advanced"><div class="filter-field-group"><label>操作類型</label><NSelect v-model:value="agentDetailLogType" clearable placeholder="全部類型" :options="[{label:'登入',value:'登入'},{label:'代理 2FA 設定',value:'代理 2FA 設定'},{label:'設定反傭',value:'設定反傭'},{label:'代理轉線',value:'代理轉線'},{label:'修改提領狀態',value:'修改提領狀態'}]" class="filter-select" /></div><div class="filter-field-group filter-date-group"><label>操作時間</label><NDatePicker v-model:value="agentDetailLogDateRange" type="daterange" clearable format="yyyy-MM-dd" start-placeholder="起始日期" end-placeholder="結束日期" placeholder="選擇時間區段" class="filter-date" /></div><div class="filter-field-group filter-account-group"><label>關鍵字</label><NInput v-model:value="agentDetailLogSearch" clearable placeholder="搜尋內容、操作者或 IP" class="filter-field filter-log-search"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div></div></NCard><NCard :bordered="false" class="table-card log-detail-card"><div class="log-table-head log-table-head-detailed"><span>時間</span><span>操作類型</span><span>操作人</span><span>詳細資訊</span><span>生效時間</span><span>IP</span></div><div v-for="log in filteredAgentDetailLogs" :key="`${log.time}-${log.detail}`" class="log-table-row log-table-row-detailed"><time>{{ log.time }}</time><NTag size="small" round :type="log.type === '登入' ? 'info' : log.type.includes('提領') ? 'warning' : 'default'">{{ log.type }}</NTag><span>{{ log.actor }}</span><div><strong>{{ log.detail }}</strong><small>{{ log.before || '-' }} → {{ log.after || '-' }}</small></div><span>{{ log.effectiveAt || '-' }}</span><code>{{ log.ip }}</code></div><p v-if="!filteredAgentDetailLogs.length" class="modal-help">目前沒有符合條件的操作紀錄。</p></NCard></div>
                </div></div>
              </template>
            </section>

            <section v-else-if="activeKey === 'players'">
              <template v-if="!showPlayerDetail">
              <NCard class="player-source-filter" size="small"><div class="player-source-filter-row"><div class="player-source-search"><label>搜尋</label><NRadioGroup v-model:value="playerSearchType" size="small"><NRadio value="id">ID</NRadio><NRadio value="account">帳號</NRadio><NRadio value="phone">手機</NRadio></NRadioGroup><NInput v-model:value="playerSearchDraft" clearable placeholder="請輸入關鍵字" /></div><div class="player-source-search"><label>所屬</label><NRadioGroup v-model:value="playerAffiliationType" size="small"><NRadio value="invite">邀請碼</NRadio><NRadio value="promo">推薦碼</NRadio></NRadioGroup><NInput v-model:value="playerAffiliationQuery" clearable placeholder="請輸入" /></div><NButton type="primary" @click="applyPlayerFilters"><template #icon><NIcon><SearchOutline /></NIcon></template>搜尋</NButton><NButton text @click="playerAdvancedSearch = !playerAdvancedSearch">{{ playerAdvancedSearch ? '收起搜尋' : '進階搜尋' }}</NButton></div><div v-if="playerAdvancedSearch" class="player-source-advanced"><div class="filter-field-group"><label>標籤</label><NSelect v-model:value="playerTagsFilter" multiple clearable placeholder="全部" :options="[{label:'VIP會員',value:'VIP會員'},{label:'一般',value:'一般'},{label:'風控關注',value:'風控關注'}]" /></div><div class="filter-field-group"><label>註冊 IP</label><NInput v-model:value="playerRegisterIpDraft" clearable placeholder="請輸入關鍵字" /></div><div class="filter-field-group"><label>註冊時間</label><NDatePicker v-model:value="playerRegisterDateRangeDraft" type="daterange" clearable format="yyyy-MM-dd" placeholder="開始日期 → 結束日期" /></div><div class="filter-field-group"><label>上級代理</label><NSelect v-model:value="playerParentDraft" clearable placeholder="選擇上級代理" :options="playerParentOptions" /></div><div class="filter-field-group"><label>查詢範圍</label><div class="scope-toggle"><button :class="{ active: playerScopeDraft === 'direct' }" @click="playerScopeDraft = 'direct'">只看直屬</button><button :class="{ active: playerScopeDraft === 'all' }" @click="playerScopeDraft = 'all'">查看全下級</button></div></div></div></NCard>
              <div class="section-head"><div><h1>玩家管理</h1><p class="muted">完整呈現運營後台玩家列表；代理相關路徑於玩家詳情的「代理關係」分頁查看。</p></div><NTag :type="canOperatePlayers ? 'warning' : 'info'" round>{{ canOperatePlayers ? '運營商可操作' : '代理唯讀' }}</NTag></div>
              <div class="table-section-label"><strong>篩選欄位</strong><span>設定條件後點擊搜尋；上級代理僅能查詢目前角色可查看的代理線</span></div><NCard class="filter-card filter-card-emphasis"><div class="filter-row filter-row-advanced"><div class="filter-field-group filter-scope-group"><label>查詢範圍</label><div class="scope-toggle"><button :class="{ active: playerScopeDraft === 'direct' }" @click="playerScopeDraft = 'direct'">只看直屬</button><button :class="{ active: playerScopeDraft === 'all' }" @click="playerScopeDraft = 'all'">查看全下級</button></div></div><div class="filter-field-group filter-account-group"><label>玩家帳號</label><NInput v-model:value="playerSearchDraft" clearable placeholder="搜尋玩家帳號或路徑" class="filter-field filter-account"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div><div class="filter-field-group"><label>註冊 IP</label><NInput v-model:value="playerRegisterIpDraft" clearable placeholder="輸入註冊 IP" class="filter-field"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div><div class="filter-field-group filter-date-group"><label>註冊時間</label><NDatePicker v-model:value="playerRegisterDateRangeDraft" type="daterange" clearable format="yyyy-MM-dd" start-placeholder="註冊起日" end-placeholder="註冊迄日" placeholder="選擇時間區段" class="filter-date" /></div><div class="filter-field-group filter-parent-group"><label>上級代理</label><NSelect v-model:value="playerParentDraft" clearable placeholder="選擇上級代理" :options="playerParentOptions" class="filter-parent" /></div><div class="filter-field-group"><label>玩家狀態</label><NSelect v-model:value="playerStatusFilterDraft" clearable placeholder="全部狀態" :options="playerStatusOptions" class="filter-select" /></div><NButton type="primary" class="filter-search-button" @click="applyPlayerFilters"><template #icon><NIcon><SearchOutline /></NIcon></template>搜尋</NButton></div></NCard>
              <div class="table-section-label"><strong>資料顯示欄位</strong><span>點擊玩家帳號或右側「查看詳情」即可開啟完整玩家詳情</span></div><NCard :bordered="false" class="table-card"><div class="player-table-wrap"><div class="player-table"><div class="player-table-head"><span>玩家 ID</span><span>玩家帳號</span><span>顯示名稱</span><span>標籤</span><span>RTP</span><span>VIP 等級</span><span>帳號狀態</span><span>在線狀態</span><span>註冊時間</span><span>操作</span></div><div v-for="row in filteredPlayers" :key="row.id" class="player-table-row"><span>{{ row.id }}</span><button class="account-link" @click="openPlayer(row)">{{ row.account }}</button><span>{{ row.displayName }}</span><div class="table-tags"><NTag v-for="tag in row.tags" :key="tag" size="small" round :type="tag === '風控關注' ? 'warning' : 'default'">{{ tag }}</NTag></div><span :class="row.rtp < 100 ? 'positive' : 'negative'">{{ row.rtp }}%</span><span>{{ row.vipLevel }}</span><NTag size="small" round :type="playerStatusType(row.status)">{{ row.status }}</NTag><NTag size="small" round :type="row.isOnline ? 'success' : 'default'">{{ row.isOnline ? '在線' : '離線' }}</NTag><span>{{ row.registerAt }}</span><div class="table-actions"><NButton size="small" secondary type="primary" @click="openPlayer(row)">查看詳情</NButton></div></div></div></div><div class="table-hint">目前顯示 {{ filteredPlayers.length }} 位玩家；點擊「查看詳情」可查看基本資料、VIP 資訊、資金帳變與代理關係。</div><div class="privacy-note">權限提示：代理只能查看玩家資料，不能停用、編輯或轉線；狀態及資金操作由運營商後台處理。隱私欄位依運營後台規則遮罩。</div></NCard>
              </template>
              <template v-else-if="selectedPlayer">
                 <div class="detail-layout"><aside class="profile-panel"><div class="profile-panel-head"><h3>基本資料</h3><NSpace><NButton v-if="canOperatePlayers" size="small" secondary @click="editPlayer(selectedPlayer)">編輯資料</NButton><NButton v-if="canOperatePlayers" size="small" type="warning" secondary @click="managePlayerStatus(selectedPlayer)">狀態管理</NButton></NSpace></div><div class="profile-avatar">{{ selectedPlayer.displayName?.slice(0, 1) || selectedPlayer.account.slice(0, 1).toUpperCase() }}</div><h2>{{ selectedPlayer.displayName }}</h2><p class="profile-id">ID：{{ selectedPlayer.id }}</p><div class="profile-status"><NTag :type="playerStatusType(selectedPlayer.status)" size="small" round>{{ selectedPlayer.status }}</NTag><NTag type="info" size="small" round>{{ selectedPlayer.vipLevel }}</NTag></div><div class="profile-fields"><div><span>玩家 ID</span><strong>{{ selectedPlayer.id }}</strong></div><div><span>玩家帳號</span><strong>{{ selectedPlayer.account }}</strong></div><div><span>顯示名稱</span><strong>{{ selectedPlayer.displayName }}</strong></div><div><span>手機號碼</span><strong>{{ selectedPlayer.phone || '-' }}</strong></div><div><span>Email</span><strong>{{ selectedPlayer.email || '-' }}</strong></div><div><span>VIP 等級</span><strong>{{ selectedPlayer.vipLevel }}</strong></div><div><span>註冊來源</span><strong>{{ selectedPlayer.registerSource }}</strong></div><div><span>註冊時間</span><strong>{{ selectedPlayer.registerAt }}</strong></div><div><span>帳號狀態</span><strong>{{ selectedPlayer.status }}</strong></div><div><span>在線狀態</span><strong>{{ selectedPlayer.isOnline ? '在線' : '離線' }}</strong></div><div><span>所屬代理</span><strong>{{ selectedPlayer.agentLevel }}</strong></div><div><span>幣別</span><strong>{{ selectedPlayer.currency }}</strong></div></div></aside><div class="detail-workspace"><div class="detail-page-head"><div><button class="back-link" @click="showPlayerDetail = false">← 返回玩家管理</button><h1>玩家詳情 · {{ selectedPlayer.account }}</h1><p class="muted">查看玩家基本資料、VIP、資金、遊戲與代理關係。</p></div><NTag :type="playerStatusType(selectedPlayer.status)" round>{{ selectedPlayer.status }}</NTag></div>
                 <div class="detail-tabs"><button :class="{ active: playerDetailTab === 'wallet' }" @click="playerDetailTab = 'wallet'">即時資料</button><button :class="{ active: playerDetailTab === 'vip' }" @click="playerDetailTab = 'vip'">VIP 資訊</button><button :class="{ active: playerDetailTab === 'promotion' }" @click="playerDetailTab = 'promotion'">優惠紀錄</button><button :class="{ active: playerDetailTab === 'audit' }" @click="playerDetailTab = 'audit'">操作稽核</button><button :class="{ active: playerDetailTab === 'asset' }" @click="playerDetailTab = 'asset'">資金帳變</button><button :class="{ active: playerDetailTab === 'game' }" @click="playerDetailTab = 'game'">遊戲紀錄</button><button :class="{ active: playerDetailTab === 'transfer' }" @click="playerDetailTab = 'transfer'">轉線紀錄</button><button :class="{ active: playerDetailTab === 'invite' }" @click="playerDetailTab = 'invite'">邀請明細</button><button :class="{ active: playerDetailTab === 'agent' }" @click="playerDetailTab = 'agent'">代理關係</button></div>
                 <div v-if="playerDetailTab === 'basic'" class="agent-detail-grid detail-page-card"><div><span>玩家 ID</span><strong>{{ selectedPlayer.id }}</strong></div><div><span>玩家帳號</span><strong>{{ selectedPlayer.account }}</strong></div><div><span>顯示名稱</span><strong>{{ selectedPlayer.displayName }}</strong></div><div><span>手機號碼</span><strong>{{ selectedPlayer.phone || '-' }}</strong></div><div><span>Email</span><strong>{{ selectedPlayer.email || '-' }}</strong></div><div><span>VIP 等級</span><strong>{{ selectedPlayer.vipLevel }}</strong></div><div><span>註冊來源</span><strong>{{ selectedPlayer.registerSource }}</strong></div><div><span>註冊時間</span><strong>{{ selectedPlayer.registerAt }}</strong></div><div><span>帳號狀態</span><NTag :type="playerStatusType(selectedPlayer.status)" round>{{ selectedPlayer.status }}</NTag></div><div><span>在線狀態</span><NTag :type="selectedPlayer.isOnline ? 'success' : 'default'" round>{{ selectedPlayer.isOnline ? '在線' : '離線' }}</NTag></div></div>
                <div v-else-if="playerDetailTab === 'wallet'" class="agent-detail-grid detail-page-card"><div><span>現金錢包</span><strong>{{ selectedPlayer.currency }} 12,800</strong></div><div><span>活動錢包</span><strong>{{ selectedPlayer.currency }} 3,260</strong></div><div><span>遊戲錢包</span><strong>{{ selectedPlayer.currency }} 1,840</strong></div><div><span>安全錢包</span><strong>{{ selectedPlayer.currency }} 0</strong></div><div class="full"><span>帳號權限</span><p class="modal-help">資金資料僅供代理查看，不可進行轉帳、加扣款或撤銷。</p></div></div>
                <div v-else-if="playerDetailTab === 'vip'" class="agent-detail-grid detail-page-card"><div><span>VIP 等級</span><strong>{{ selectedPlayer.vipLevel }}</strong></div><div><span>本月儲值</span><strong>{{ selectedPlayer.currency }} {{ selectedPlayer.deposit.toLocaleString() }}</strong></div><div><span>本月投注</span><strong>{{ selectedPlayer.currency }} {{ selectedPlayer.bet.toLocaleString() }}</strong></div><div><span>VIP 權限</span><strong>依運營後台規則享有</strong></div><div class="full"><span>VIP 說明</span><p class="modal-help">VIP 升降級與獎勵由運營後台管理；代理後台僅提供查詢。</p></div></div>
                <div v-else-if="playerDetailTab === 'promotion'" class="player-report-panel detail-page-card"><div class="report-summary-grid"><div><span>紀錄總數</span><strong>{{ playerPromotionRecords.length }} 筆</strong></div><div><span>已完成</span><strong>2 筆</strong></div><div><span>待領取金額</span><strong>{{ selectedPlayer.currency }} 360</strong></div></div><div class="player-report-table"><div class="player-report-head"><span>發生時間</span><span>優惠活動</span><span>優惠金額</span><span>狀態</span><span>明細</span></div><div v-for="record in playerPromotionRecords" :key="record.time" class="player-report-row"><time>{{ record.time }}</time><span>{{ record.name }}</span><strong>{{ selectedPlayer.currency }} {{ record.amount.toLocaleString() }}</strong><NTag size="small" round :type="record.status === '已完成' ? 'success' : 'warning'">{{ record.status }}</NTag><span>{{ record.detail }}</span></div></div></div>
                <div v-else-if="playerDetailTab === 'asset'" class="player-report-panel detail-page-card"><div class="report-summary-grid"><div><span>異動總筆數</span><strong>18 筆</strong></div><div><span>本月儲值</span><strong>{{ selectedPlayer.currency }} {{ selectedPlayer.deposit.toLocaleString() }}</strong></div><div><span>本月提領</span><strong>{{ selectedPlayer.currency }} 4,200</strong></div></div><div class="player-report-table"><div class="player-report-head"><span>發生時間</span><span>帳變類型</span><span>帳變金額</span><span>餘額</span><span>狀態／明細</span></div><div v-for="record in playerAssetRecords" :key="record.time" class="player-report-row"><time>{{ record.time }}</time><span>{{ record.type }}</span><strong :class="record.amount > 0 ? 'positive' : 'negative'">{{ record.amount > 0 ? '+' : '' }}{{ selectedPlayer.currency }} {{ Math.abs(record.amount).toLocaleString() }}</strong><strong>{{ selectedPlayer.currency }} {{ record.balance.toLocaleString() }}</strong><span><NTag size="small" round :type="record.status === '成功' ? 'success' : 'warning'">{{ record.status }}</NTag><small class="mode-summary">{{ record.detail }}</small></span></div></div></div>
                 <div v-else-if="playerDetailTab === 'game'" class="player-report-panel detail-page-card"><div class="report-summary-grid"><div><span>紀錄總筆數</span><strong>{{ playerGameRecords.length }} 筆</strong></div><div><span>本期有效投注</span><strong>{{ selectedPlayer.currency }} 86,420</strong></div><div><span>RTP</span><strong :class="selectedPlayer.rtp < 100 ? 'positive' : 'negative'">{{ selectedPlayer.rtp }}%</strong></div></div><div class="player-report-table"><div class="player-report-head"><span>遊戲時間</span><span>遊戲／場次</span><span>投注額</span><span>輸贏</span><span>狀態</span></div><div v-for="record in playerGameRecords" :key="record.time" class="player-report-row"><time>{{ record.time }}</time><span>{{ record.game }}<small class="mode-summary">{{ record.rounds }} 局</small></span><strong>{{ selectedPlayer.currency }} {{ record.bet.toLocaleString() }}</strong><strong :class="record.result.startsWith('-') ? 'negative' : 'positive'">{{ selectedPlayer.currency }} {{ record.result }}</strong><NTag size="small" round type="success">{{ record.status }}</NTag></div></div></div>
                 <div v-else-if="playerDetailTab === 'audit'" class="player-report-panel detail-page-card"><div class="report-summary-grid"><div><span>稽核總筆數</span><strong>{{ playerAuditRecords.length }} 筆</strong></div><div><span>最近登入</span><strong>{{ selectedPlayer.lastLoginAt }}</strong></div><div><span>最近登入 IP</span><strong>{{ selectedPlayer.lastLoginIp }}</strong></div></div><div class="player-report-table"><div class="player-report-head"><span>操作時間</span><span>操作類型</span><span>操作人</span><span>詳細資訊</span><span>IP</span></div><div v-for="record in playerAuditRecords" :key="record.time" class="player-report-row"><time>{{ record.time }}</time><span>{{ record.type }}</span><span>{{ record.actor }}</span><strong>{{ record.detail }}</strong><code>{{ record.ip }}</code></div></div></div>
                 <div v-else-if="playerDetailTab === 'transfer'" class="player-report-panel detail-page-card"><div class="report-summary-grid"><div><span>轉線總筆數</span><strong>{{ playerTransferRecords.length }} 筆</strong></div><div><span>目前代理線</span><strong>{{ displayAgentPath(selectedPlayer.path) }}</strong></div><div><span>最近生效時間</span><strong>2026-08-01 00:00:00</strong></div></div><div class="player-report-table"><div class="player-report-head"><span>生效時間</span><span>原代理線</span><span>新代理線</span><span>執行人</span><span>狀態</span></div><div v-for="record in playerTransferRecords" :key="record.time" class="player-report-row"><time>{{ record.time }}</time><span>{{ record.from }}</span><span>{{ record.to }}</span><span>{{ record.actor }}</span><NTag size="small" round type="success">{{ record.status }}</NTag></div></div></div>
                 <div v-else-if="playerDetailTab === 'invite'" class="detail-list detail-page-card"><div class="detail-list-row"><span>註冊使用玩家推廣碼</span><strong class="code-text">{{ selectedPlayer.referralCode || '未記錄' }}</strong></div><div class="detail-list-row"><span>自身玩家邀請碼</span><strong class="code-text">{{ selectedPlayer.inviteCode }}</strong></div><div class="detail-list-row"><span>邀請總人數</span><strong>3 人</strong></div><div class="detail-list-row"><span>有效邀請</span><strong>2 人</strong></div><div class="detail-list-row"><span>待生效邀請</span><strong>1 人</strong></div><p class="modal-help">玩家邀請玩家不產生反水，反水只回饋代理線上的代理身分。</p></div>
                 <div v-else class="agent-detail-grid detail-page-card"><div class="full"><span>完整樹狀路徑</span><strong>{{ displayAgentPath(selectedPlayer.path) }}</strong></div><div><span>玩家層級</span><strong>{{ selectedPlayer.agentLevel }}</strong></div><div><span>所屬幣別</span><strong>{{ selectedPlayer.currency }}</strong></div><div><span>直屬代理</span><strong>north_team</strong></div><div><span>歸屬方式</span><strong>玩家推廣碼</strong></div><div><span>資料可見範圍</span><strong>所屬代理線</strong></div><div class="full"><span>代理後台權限</span><p class="modal-help">代理可以查看所屬下級玩家，但不能操作停用、編輯或轉線；玩家狀態與資金操作由運營商後台處理。</p></div></div></div></div>
              </template>
            </section>

            <section v-else-if="activeKey === 'codes'">
              <div class="section-head"><div><h1>傭金方案設定</h1><p class="muted">查看上級開給我的方案，並管理我開給直屬下級的方案。</p></div><NButton v-if="canManagePlans" type="primary" @click="openPlanEditor()">＋ 新增傭金方案</NButton></div>
              <template v-if="currentRole !== '運營商'"><div class="table-section-label"><strong>上級開給我的方案</strong><span>上級提供的方案僅供查看，不可修改</span></div><NCard :bordered="false" class="table-card upstream-plan-card"><div v-if="currentUpstreamPlan" class="upstream-plan-row"><div><span>方案名稱</span><strong>{{ currentUpstreamPlan.name }}</strong></div><div><span>建立代理</span><strong>{{ currentUpstreamPlan.createdBy || currentUpstreamPlan.parentAccount }}</strong></div><div><span>傭金模式</span><NTag size="small" round :type="currentUpstreamPlan.model === '輸贏' ? 'warning' : 'success'">{{ currentUpstreamPlan.model }}</NTag></div><div><span>結算週期</span><strong>{{ currentUpstreamPlan.cycle }}</strong></div><div><span>默認點數</span><strong>{{ currentUpstreamPlan.allocationRate }}%</strong></div><div><span>套用代理</span><strong>{{ planAppliedAgents(currentUpstreamPlan).length }} 位</strong></div><div><span>狀態</span><NTag size="small" round type="success">{{ currentUpstreamPlan.status }}</NTag></div></div><p v-else class="modal-help">目前尚未有上級方案。</p></NCard></template>
              <div class="table-section-label"><strong>我開給直屬下級的方案</strong><span>此區塊只顯示自己建立的方案；套用代理可展開查看明細</span></div><NCard class="filter-card filter-card-emphasis plan-filter-card"><div class="filter-row plan-filter-row"><div class="filter-field-group"><label>傭金模式</label><NSelect v-model:value="codesPlanModelFilter" clearable placeholder="全部模式" :options="[{label:'儲值',value:'儲值'},{label:'有效投注額',value:'有效投注額'},{label:'輸贏',value:'輸贏'}]" /></div><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="codesPlanStatusFilter" clearable placeholder="全部狀態" :options="[{label:'啟用',value:'啟用'},{label:'停用',value:'停用'}]" /></div></div></NCard>
              <NCard :bordered="false" class="table-card"><div class="plan-table-wrap"><div class="plan-table plan-table-own"><div class="plan-table-head"><span>方案名稱</span><span>建立代理</span><span>傭金模式</span><span>結算週期</span><span>默認點數</span><span>套用代理</span><span>狀態</span><span>操作</span></div><div v-for="plan in filteredCurrentCreatedPlans" :key="plan.id" class="plan-table-row"><div><strong>{{ plan.name }}</strong><small>{{ plan.description }}</small></div><span>{{ plan.createdBy || plan.parentAccount }}</span><div><NTag size="small" :type="plan.model === '輸贏' ? 'warning' : 'success'" round>{{ plan.model }}</NTag><small class="mode-summary">{{ planConfigSummary(plan) }}</small></div><span>{{ plan.cycle }}</span><span>{{ plan.allocationRate }}%</span><span class="plan-assignee-count">{{ planAppliedAgents(plan).length }} 位<NButton quaternary size="tiny" class="plan-expand-inline" @click="togglePlanExpanded(plan.id)">{{ expandedPlanIds.includes(plan.id) ? '收闔' : '展開' }}</NButton></span><NTag size="small" :type="plan.status === '啟用' ? 'success' : 'error'" round>{{ plan.status }}</NTag><div class="table-actions"><NButton v-if="canEditPlan(plan)" quaternary size="small" @click="openPlanEditor(plan)">編輯</NButton><NButton v-if="canEditPlan(plan)" quaternary size="small" @click="togglePlan(plan)">{{ plan.status === '啟用' ? '停用' : '啟用' }}</NButton></div><div v-if="expandedPlanIds.includes(plan.id)" class="plan-assigned-detail"><div class="plan-assigned-head"><span>套用代理帳號</span><span>狀態</span><span>註冊時間</span><span>最後登入時間</span></div><div v-for="agent in planAppliedAgents(plan)" :key="agent.id" class="plan-assigned-row"><strong>{{ agent.account }}</strong><NTag size="small" round :type="agent.status === '啟用' ? 'success' : 'error'">{{ agent.status }}</NTag><time>{{ agent.registerAt || '-' }}</time><time>{{ agent.lastLoginAt || '-' }}</time></div><p v-if="!planAppliedAgents(plan).length" class="modal-help">目前沒有代理套用此方案。</p></div></div><p v-if="!filteredCurrentCreatedPlans.length" class="modal-help">目前尚未建立給直屬下級的方案。</p></div></div></NCard>
            </section>

            <section v-else-if="activeKey === 'commissionPlans'">
              <div class="section-head"><div><h1>傭金方案</h1><p class="muted">以報表方式查看目前代理線可見的傭金方案，並展開查看套用代理。</p><p class="line-rule">目前顯示上級提供及本代理線可見方案；方案調整請至傭金方案設定。</p></div></div>
               <div class="plan-mode-grid"><NCard v-for="mode in [{name:'儲值',desc:'依成功儲值金額按方案比例計算。'},{name:'有效投注額',desc:'依有效投注總額按方案比例計算。'},{name:'輸贏',desc:'依輸贏總額扣除行政成本後按比例分配。'}]" :key="mode.name" class="plan-mode-card"><NTag :type="mode.name === '輸贏' ? 'warning' : 'success'" round>{{ mode.name }}</NTag><span>{{ mode.desc }}</span></NCard></div>
<NCard class="filter-card filter-card-emphasis plan-filter-card plan-report-filter-card"><div class="filter-row plan-filter-row"><div class="filter-field-group"><label>建立代理</label><NInput v-model:value="planReportCreatedBy" clearable placeholder="查詢建立代理" /></div><div class="filter-field-group"><label>套用帳號</label><NInput v-model:value="planReportAppliedAccount" clearable placeholder="查詢套用帳號" /></div><div class="filter-field-group"><label>傭金模式</label><NSelect v-model:value="planReportModel" clearable placeholder="全部模式" :options="[{label:'儲值',value:'儲值'},{label:'有效投注額',value:'有效投注額'},{label:'輸贏',value:'輸贏'}]" /></div><div class="filter-field-group"><label>結算週期</label><NSelect v-model:value="planReportCycle" clearable placeholder="全部週期" :options="[{label:'即時',value:'即時'},{label:'每日',value:'每日'},{label:'每週',value:'每週'},{label:'每月',value:'每月'}]" /></div><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="planReportStatus" clearable placeholder="全部狀態" :options="[{label:'啟用',value:'啟用'},{label:'停用',value:'停用'}]" /></div></div></NCard>
               <div class="table-section-label"><strong>方案資料</strong><span>建立代理代表方案建立者；展開後查看套用代理帳號、狀態、註冊時間與最後登入時間</span></div><NCard :bordered="false" class="table-card"><div class="plan-table-wrap"><div class="plan-table"><div class="plan-table-head"><span>方案名稱</span><span>建立代理</span><span>傭金模式</span><span>結算週期</span><span>默認點數</span><span>套用代理</span><span>狀態</span><span>操作</span></div><div v-for="plan in filteredVisiblePlans" :key="plan.id" class="plan-table-row"><div><strong>{{ plan.name }}</strong><small>{{ plan.description }}</small></div><span>{{ plan.createdBy || plan.parentAccount }}</span><div><NTag size="small" :type="plan.model === '輸贏' ? 'warning' : 'success'" round>{{ plan.model }}</NTag><small class="mode-summary">{{ planConfigSummary(plan) }}</small></div><span>{{ plan.cycle }}</span><span>{{ plan.allocationRate }}%</span><span class="plan-assignee-count">{{ planAppliedAgents(plan).length }} 位<NButton quaternary size="tiny" class="plan-expand-inline" @click="togglePlanExpanded(plan.id)">{{ expandedPlanIds.includes(plan.id) ? '收闔' : '展開' }}</NButton></span><NTag size="small" :type="plan.status === '啟用' ? 'success' : 'error'" round>{{ plan.status }}</NTag><div class="table-actions"><span class="muted">唯讀報表</span></div><div v-if="expandedPlanIds.includes(plan.id)" class="plan-assigned-detail"><div class="plan-assigned-head"><span>套用代理帳號</span><span>狀態</span><span>註冊時間</span><span>最後登入時間</span></div><div v-for="agent in planAppliedAgents(plan)" :key="agent.id" class="plan-assigned-row"><strong>{{ agent.account }}</strong><NTag size="small" round :type="agent.status === '啟用' ? 'success' : 'error'">{{ agent.status }}</NTag><time>{{ agent.registerAt || '-' }}</time><time>{{ agent.lastLoginAt || '-' }}</time></div><p v-if="!planAppliedAgents(plan).length" class="modal-help">目前沒有代理套用此方案。</p></div></div></div></div><div class="table-hint">傭金方案不使用推廣碼；同一條代理線只能使用一種傭金模式，下級方案不得超過上級額度。</div></NCard>
            </section>

            <section v-else-if="activeKey === 'commission'">
              <div class="section-head"><div><h1>傭金發放紀錄</h1><p class="muted">系統依結算週期產生傭金發放紀錄；處理中的異常紀錄由運營商手動判定成功或失敗。</p></div><NTag type="info" round>系統結算報表</NTag></div>
              <div class="commission-cards"><NCard><div class="mini-label">待手動處理</div><div class="big-value">{{ payoutSummary.pending }} 筆</div><p class="muted">依目前篩選結果</p></NCard><NCard><div class="mini-label">成功／失敗</div><div class="big-value positive">{{ payoutSummary.success }}／{{ payoutSummary.failed }}</div><p class="muted">依目前篩選結果</p></NCard><NCard><div class="mini-label">發放金額</div><div class="big-value positive">TWD {{ payoutSummary.amount.toLocaleString() }}</div><p class="muted">依目前篩選區間統計</p></NCard></div>
              <NCard class="filter-card filter-card-emphasis"><div class="filter-row filter-row-advanced"><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="payoutReportStatus" clearable placeholder="全部狀態" :options="[{label:'處理中',value:'處理中'},{label:'成功',value:'成功'},{label:'失敗',value:'失敗'}]" class="filter-select" /></div><div class="filter-field-group filter-date-group"><label>紀錄產生時間</label><NDatePicker v-model:value="payoutReportDateRange" type="daterange" clearable format="yyyy-MM-dd" placeholder="選擇時間區段" class="filter-date" /></div><div class="filter-field-group filter-account-group"><label>帳號／單號</label><NInput v-model:value="payoutReportKeyword" clearable placeholder="搜尋代理帳號、方案或單號" class="filter-field" /></div></div></NCard>
              <NCard :bordered="false" class="table-card commission-report-card"><div class="commission-report-head"><span>帳號</span><span>訂單編號</span><span>金額</span><span>狀態</span><span>紀錄產生時間</span><span>異動時間</span><span>操作</span></div><div v-for="record in filteredPayoutRecords" :key="record.id" class="commission-report-row"><strong>{{ record.account }}</strong><code>{{ record.id }}</code><strong class="positive">+ {{ record.currency }} {{ record.amount.toLocaleString() }}</strong><NTag size="small" round :type="record.status === '成功' ? 'success' : record.status === '失敗' ? 'error' : 'warning'">{{ record.status }}</NTag><time>{{ record.createdAt }}</time><time>{{ record.updatedAt }}</time><div class="report-action-group"><NButton size="small" quaternary @click="openPayoutProcess(record)">查看詳情</NButton><NButton v-if="currentRole === '運營商' && record.status === '處理中'" size="small" type="primary" @click="openPayoutProcess(record)">手動處理</NButton></div></div><p v-if="!filteredPayoutRecords.length" class="modal-help">目前沒有符合條件的傭金發放紀錄。</p></NCard>
            </section>

            <section v-else-if="activeKey === 'withdrawal'">
              <div class="section-head"><div><h1>傭金提領紀錄</h1><p class="muted">代理提交提領後先為待處理；運營商開始出款後鎖定為處理中，完成後更新成功或失敗。</p></div><NButton v-if="currentRole !== '運營商'" type="primary" @click="showWithdrawal = true">申請提領</NButton></div>
               <div class="commission-cards"><NCard><div class="mini-label">待處理</div><div class="big-value">{{ withdrawalSummary.pending }} 筆</div><p class="muted">依目前篩選結果</p></NCard><NCard><div class="mini-label">處理中</div><div class="big-value">{{ withdrawalSummary.processing }} 筆</div><p class="muted">已鎖定，不可重複處理</p></NCard><NCard><div class="mini-label">成功／失敗</div><div class="big-value positive">{{ withdrawalSummary.success }}／{{ withdrawalSummary.failed }}</div><p class="muted">依目前篩選結果</p></NCard><NCard><div class="mini-label">總提領金額</div><div class="big-value negative">TWD {{ withdrawalSummary.amount.toLocaleString() }}</div><p class="muted">依目前篩選區間統計</p></NCard></div>
              <NCard class="filter-card filter-card-emphasis"><div class="filter-row filter-row-advanced"><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="withdrawalReportStatus" clearable placeholder="全部狀態" :options="[{label:'待處理',value:'待處理'},{label:'處理中',value:'處理中'},{label:'成功',value:'成功'},{label:'失敗',value:'失敗'}]" class="filter-select" /></div><div class="filter-field-group filter-date-group"><label>紀錄產生時間</label><NDatePicker v-model:value="withdrawalReportDateRange" type="daterange" clearable format="yyyy-MM-dd" placeholder="選擇時間區段" class="filter-date" /></div><div class="filter-field-group filter-account-group"><label>帳號／單號</label><NInput v-model:value="withdrawalReportKeyword" clearable placeholder="搜尋代理帳號、銀行或單號" class="filter-field" /></div></div></NCard>
              <NCard :bordered="false" class="table-card commission-report-card"><div class="commission-report-head"><span>帳號</span><span>訂單編號</span><span>金額</span><span>狀態</span><span>紀錄產生時間</span><span>異動時間</span><span>操作</span></div><div v-for="order in filteredWithdrawalOrders" :key="order.id" class="commission-report-row"><strong>{{ order.account }}</strong><code>{{ order.id }}</code><strong class="negative">- {{ order.currency }} {{ order.amount.toLocaleString() }}</strong><NTag size="small" round :type="order.status === '成功' ? 'success' : order.status === '失敗' ? 'error' : 'warning'">{{ order.status }}</NTag><time>{{ order.createdAt }}</time><time>{{ order.updatedAt }}</time><div class="report-action-group"><NButton size="small" quaternary @click="openWithdrawalOrder(order)">查看詳情</NButton><NButton v-if="currentRole === '運營商' && order.status === '待處理'" size="small" type="primary" @click="startWithdrawalProcessing(order)">出款處理</NButton><NButton v-else-if="currentRole === '運營商' && order.status === '處理中' && order.processor === identity.account" size="small" type="primary" @click="openWithdrawalOrder(order)">繼續處理</NButton><NButton v-else-if="currentRole === '運營商' && order.status === '處理中'" size="small" disabled>他人處理中</NButton></div></div><p v-if="!filteredWithdrawalOrders.length" class="modal-help">目前沒有符合條件的傭金提領紀錄。</p></NCard>
            </section>

            <section v-else-if="activeKey === 'reports'">
              <div class="section-head"><div><h1>下級報表</h1><p class="muted">以樹狀路徑與層級欄位呈現，避免無限層代理造成數據重複。</p></div><NButton secondary @click="exportMessage('下級報表')">匯出報表</NButton></div>
              <NCard class="filter-card"><div class="filter-row"><NSelect v-model:value="selectedCycle" :options="[{label:'本週',value:'本週'},{label:'本月',value:'本月'},{label:'上月',value:'上月'}]" style="width: 140px" /><div class="scope-toggle"><button :class="{ active: scope === 'direct' }" @click="scope = 'direct'">只看直屬</button><button :class="{ active: scope === 'all' }" @click="scope = 'all'">查看全部下級</button></div></div></NCard>
              <div class="stat-grid"><NCard><NStatistic label="總有效投注" :value="1284600" prefix="TWD " /></NCard><NCard><NStatistic label="總儲值" :value="342800" prefix="TWD " /></NCard><NCard><NStatistic label="產生傭金" :value="28460" prefix="TWD " /></NCard><NCard><NStatistic label="活躍玩家" :value="186" suffix=" 位" /></NCard></div>
              <NCard :bordered="false" class="table-card"><div class="report-header"><span>代理／玩家</span><span>層級</span><span>路徑</span><span>有效投注</span><span>傭金</span></div><div v-for="row in agentRows.slice(0, 4)" :key="row.id" class="report-row"><strong>{{ row.account }}</strong><span>{{ row.level }}</span><span class="path-text">{{ row.path }}</span><span>TWD {{ (row.point * 128460).toLocaleString() }}</span><strong class="positive">TWD {{ (row.point * 2846).toLocaleString() }}</strong></div></NCard>
            </section>

            <section v-else-if="activeKey === 'logs'">
              <div class="section-head"><div><h1>操作日誌</h1><p class="muted">完整保留登入、開設代理、調整反傭、提領等操作。</p></div><NButton secondary @click="exportMessage('操作紀錄')">匯出紀錄</NButton></div>
              <NCard class="filter-card"><div class="filter-row"><NSelect v-model:value="logTypeFilter" placeholder="操作類型" clearable :options="[{label:'登入',value:'登入'},{label:'開設代理',value:'開設代理'},{label:'設定反傭',value:'設定反傭'},{label:'提領申請',value:'提領申請'},{label:'查看 Google Auth',value:'查看 Google Auth'},{label:'重設 Google Auth',value:'重設 Google Auth'},{label:'代理 2FA 設定',value:'代理 2FA 設定'},{label:'代理轉線',value:'代理轉線'},{label:'修改提領狀態',value:'修改提領狀態'},{label:'人工加扣款',value:'人工加扣款'},{label:'編輯代理資料',value:'編輯代理資料'}]" style="width: 220px" /><NDatePicker v-model:value="logDateRange" type="daterange" clearable format="yyyy-MM-dd" placeholder="選擇時間範圍" style="width: 250px" /><NInput v-model:value="logSearch" clearable placeholder="搜尋操作人、對象、內容或 IP" class="search-input"><template #prefix><NIcon><SearchOutline /></NIcon></template></NInput></div></NCard>
              <NCard :bordered="false" class="table-card"><div class="log-table-head log-table-head-detailed"><span>時間</span><span>操作類型</span><span>操作人</span><span>對象</span><span>詳細資訊</span><span>生效時間</span><span>IP</span></div><div v-for="log in filteredLogs" :key="`${log.time}-${log.detail}`" class="log-table-row log-table-row-detailed"><time>{{ log.time }}</time><NTag size="small" round :type="log.type === '登入' ? 'info' : log.type.includes('提領') ? 'warning' : 'default'">{{ log.type }}</NTag><span>{{ log.actor }}</span><span>{{ log.target || '-' }}</span><div><strong>{{ log.detail }}</strong><small v-if="log.before || log.after">{{ log.before || '-' }} → {{ log.after || '-' }}</small></div><span>{{ log.effectiveAt || '-' }}</span><code>{{ log.ip }}</code></div><p v-if="!filteredLogs.length" class="modal-help">目前沒有符合條件的操作紀錄。</p></NCard>
            </section>

            <section v-else-if="activeKey === 'profile'">
              <div class="section-head"><div><h1>帳戶設定</h1><p class="muted">管理目前登入代理的個人資料、安全設定與傭金收款銀行卡。</p></div><NTag type="info" round>{{ identity.label }}</NTag></div>
              <div class="profile-grid"><NCard title="個人資料"><div class="profile-list"><div><span>代理帳號</span><strong>{{ identity.account }}</strong></div><div><span>角色</span><strong>{{ identity.label }}</strong></div><div><span>所屬幣別</span><strong>{{ identity.currency }}</strong></div><div><span>手機</span><strong>09******123</strong></div><div><span>Email</span><strong>ka********@example.com</strong></div><div><span>登入密碼</span><strong>••••••••</strong><NButton size="small" quaternary>修改</NButton></div></div></NCard><NCard title="傭金收款銀行卡"><div class="bank-card"><div class="bank-brand">台新銀行</div><strong>**** **** 9012</strong><span>戶名：Klaus Lin</span><NTag type="success" round>已驗證</NTag></div><NButton type="primary" secondary @click="showBankCard = true">管理銀行卡</NButton><p class="modal-help">銀行卡僅用於傭金提領，提領時需選擇已驗證的收款帳戶。</p></NCard></div>
              <NCard title="代理後台使用說明" class="role-guide"><div class="role-guide-grid"><div><NTag type="info" round>代理</NTag><p>可建立、編輯及管理自己代理線下的代理與 2FA，查看下級玩家與傭金報表。更換代理線、玩家狀態及資金操作由運營商後台統一處理。</p></div></div></NCard>
            </section>
          <div v-if="activeKey === 'agents' && showAgentDetail && detailTab === 'commission' && selectedAgent" class="relationship-clean-panel commission-plan-overlay">
            <div class="commission-plan-overlay-head"><div><span>代理傭金方案</span><strong>{{ selectedAgent.account }}</strong><p class="muted">上方為上級開給此代理的方案，下方為此代理開給直屬下級的方案。</p></div><NTag type="info" round>唯讀查看</NTag></div>
            <div class="table-section-label"><strong>上級開給我的方案</strong><span>上級提供的方案僅供查看</span></div>
            <NCard :bordered="false" class="upstream-plan-card"><div v-if="selectedAgentPlan" class="upstream-plan-row"><div><span>方案名稱</span><strong>{{ selectedAgentPlan.name }}</strong></div><div><span>建立代理</span><strong>{{ selectedAgentPlan.createdBy || selectedAgentPlan.parentAccount }}</strong></div><div><span>傭金模式</span><NTag size="small" round :type="selectedAgentPlan.model === '輸贏' ? 'warning' : 'success'">{{ selectedAgentPlan.model }}</NTag></div><div><span>結算週期</span><strong>{{ selectedAgentPlan.cycle }}</strong></div><div><span>默認點數</span><strong>{{ selectedAgentPlan.allocationRate }}%</strong></div><div><span>套用代理</span><strong>{{ planAppliedAgents(selectedAgentPlan).length }} 位</strong></div><div><span>狀態</span><NTag size="small" round type="success">{{ selectedAgentPlan.status }}</NTag></div></div><p v-else class="modal-help">目前尚未有上級方案。</p></NCard>
            <div class="table-section-label"><strong>我開給直屬下級的方案</strong><span>此區塊不顯示建立代理與操作欄位；套用代理可展開查看明細</span></div><NCard class="filter-card filter-card-emphasis plan-filter-card"><div class="filter-row plan-filter-row"><div class="filter-field-group"><label>傭金模式</label><NSelect v-model:value="detailPlanModelFilter" clearable placeholder="全部模式" :options="[{label:'儲值',value:'儲值'},{label:'有效投注額',value:'有效投注額'},{label:'輸贏',value:'輸贏'}]" /></div><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="detailPlanStatusFilter" clearable placeholder="全部狀態" :options="[{label:'啟用',value:'啟用'},{label:'停用',value:'停用'}]" /></div></div></NCard>
            <NCard :bordered="false" class="table-card"><div class="plan-table-wrap"><div class="plan-table plan-table-own"><div class="plan-table-head"><span>方案名稱</span><span>建立代理</span><span>傭金模式</span><span>結算週期</span><span>默認點數</span><span>套用代理</span><span>狀態</span></div><div v-for="plan in filteredSelectedAgentCreatedPlans" :key="plan.id" class="plan-table-row"><div><strong>{{ plan.name }}</strong><small>{{ plan.description }}</small></div><span>{{ plan.createdBy || plan.parentAccount }}</span><div><NTag size="small" :type="plan.model === '輸贏' ? 'warning' : 'success'" round>{{ plan.model }}</NTag><small class="mode-summary">{{ planConfigSummary(plan) }}</small></div><span>{{ plan.cycle }}</span><span>{{ plan.allocationRate }}%</span><span class="plan-assignee-count">{{ planAppliedAgents(plan).length }} 位<NButton quaternary size="tiny" class="plan-expand-inline" @click="togglePlanExpanded(plan.id)">{{ expandedPlanIds.includes(plan.id) ? '收闔' : '展開' }}</NButton></span><NTag size="small" :type="plan.status === '啟用' ? 'success' : 'error'" round>{{ plan.status }}</NTag><div v-if="expandedPlanIds.includes(plan.id)" class="plan-assigned-detail"><div class="plan-assigned-head"><span>套用代理帳號</span><span>狀態</span><span>註冊時間</span><span>最後登入時間</span></div><div v-for="agent in planAppliedAgents(plan)" :key="agent.id" class="plan-assigned-row"><strong>{{ agent.account }}</strong><NTag size="small" round :type="agent.status === '啟用' ? 'success' : 'error'">{{ agent.status }}</NTag><time>{{ agent.registerAt || '-' }}</time><time>{{ agent.lastLoginAt || '-' }}</time></div><p v-if="!planAppliedAgents(plan).length" class="modal-help">目前沒有代理套用此方案。</p></div></div><p v-if="!filteredSelectedAgentCreatedPlans.length" class="modal-help">此代理尚未建立給直屬下級的方案。</p></div></div></NCard>
          </div>
          <div v-if="activeKey === 'agents' && showAgentDetail && detailTab === 'relationship' && selectedAgent" class="relationship-clean-panel relationship-clean-overlay">
            <div class="relationship-clean-header">
              <div class="relationship-clean-header-main"><div class="relationship-agent-total"><span>代理總人數</span><strong>{{ relationshipRows('agents', 'all').length }} 人</strong></div><div class="relationship-player-total"><span>玩家總人數</span><strong>{{ relationshipRows('players', 'all').length }} 人</strong></div><div><span>完整樹狀路徑</span><strong>{{ displayAgentPath(selectedAgent.path) }}</strong><p v-if="selectedAgent.pendingTransferTargetPath" class="pending-transfer">預約新代理線：{{ displayAgentPath(selectedAgent.pendingTransferTargetPath) }}（{{ selectedAgent.pendingTransferEffectiveAt }} 生效）</p></div></div>
              <div class="relationship-clean-header-tools"><label>資料範圍<select v-model="relationshipFilterScope"><option value="history">歷史</option><option value="cycle">結算週期</option></select></label><NButton v-if="currentRole === '運營商' || currentRole === '總代理'" class="agent-transfer-button" type="primary" :disabled="selectedAgent.path.split(' > ').length === 1" @click="openAgentTransfer(selectedAgent)">更換代理線</NButton></div>
            </div>
            <div class="relationship-clean-rows">
              <div class="relationship-clean-row">
                <div class="relationship-clean-trigger" role="button" tabindex="0" @click="toggleRelationshipSection('totalAgents')"><div class="relationship-clean-title-row"><span class="relationship-clean-title">總下級（含直屬）代理</span><em>{{ relationshipExpanded.totalAgents ? '收闔' : '展開' }}⌄</em></div><div class="relationship-clean-metrics"><div class="relationship-clean-metric"><small>代理人數</small><strong>{{ relationshipFilteredRows('agents', 'all').length }} 位</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總儲值金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'all', 'deposit')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'all', 'deposit'), $event)"><option value="">排序</option><option value="depositDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('agents', 'all', 'deposit').toLocaleString() }}</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總有效投注</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'all', 'bet')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'all', 'bet'), $event)"><option value="">排序</option><option value="betDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('agents', 'all', 'bet').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'all', 'firstDepositCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'all', 'firstDepositCount'), $event)"><option value="">排序</option><option value="firstDepositDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleCount('agents', 'all') }} 位</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存總金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'all', 'firstDepositAmount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'all', 'firstDepositAmount'), $event)"><option value="">排序</option><option value="firstDepositAmountDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipCycleAmount('agents', 'all').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期註冊人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'all', 'registeredCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'all', 'registeredCount'), $event)"><option value="">排序</option><option value="registeredDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleRegistered('agents', 'all') }} 位</strong></div></div></div>
                <div v-if="relationshipExpanded.totalAgents" class="relationship-clean-drawer"><div class="relationship-clean-drawer-head"><span>帳號</span><span>儲值總金額</span><span>有效投注</span><span>首存時間</span><span>首存金額</span><span>註冊時間</span></div><div v-for="row in relationshipDetailRows('agents', 'all')" :key="row.id" class="relationship-clean-drawer-row"><strong>{{ row.account }}</strong><span>{{ row.currency }} {{ (row.totalDeposit ?? row.deposit ?? 0).toLocaleString() }}</span><span>{{ row.currency }} {{ (row.totalEffectiveBet ?? row.bet ?? 0).toLocaleString() }}</span><span>{{ relationshipFirstDepositAt(row) }}</span><span>{{ relationshipFirstDepositAmount(row) }}</span><span>{{ relationshipRegisterLabel(row) }}</span></div></div>
              </div>
              <div class="relationship-clean-row">
                <div class="relationship-clean-trigger" role="button" tabindex="0" @click="toggleRelationshipSection('directAgents')"><div class="relationship-clean-title-row"><span class="relationship-clean-title">直屬下級代理</span><em>{{ relationshipExpanded.directAgents ? '收闔' : '展開' }}⌄</em></div><div class="relationship-clean-metrics"><div class="relationship-clean-metric"><small>代理人數</small><strong>{{ relationshipFilteredRows('agents', 'direct').length }} 位</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總儲值金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'direct', 'deposit')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'direct', 'deposit'), $event)"><option value="">排序</option><option value="depositDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('agents', 'direct', 'deposit').toLocaleString() }}</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總有效投注</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'direct', 'bet')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'direct', 'bet'), $event)"><option value="">排序</option><option value="betDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('agents', 'direct', 'bet').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'direct', 'firstDepositCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'direct', 'firstDepositCount'), $event)"><option value="">排序</option><option value="firstDepositDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleCount('agents', 'direct') }} 位</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存總金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'direct', 'firstDepositAmount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'direct', 'firstDepositAmount'), $event)"><option value="">排序</option><option value="firstDepositAmountDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipCycleAmount('agents', 'direct').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期註冊人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('agents', 'direct', 'registeredCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('agents', 'direct', 'registeredCount'), $event)"><option value="">排序</option><option value="registeredDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleRegistered('agents', 'direct') }} 位</strong></div></div></div>
                <div v-if="relationshipExpanded.directAgents" class="relationship-clean-drawer"><div class="relationship-clean-drawer-head"><span>帳號</span><span>儲值總金額</span><span>有效投注</span><span>首存時間</span><span>首存金額</span><span>註冊時間</span></div><div v-for="row in relationshipDetailRows('agents', 'direct')" :key="row.id" class="relationship-clean-drawer-row"><strong>{{ row.account }}</strong><span>{{ row.currency }} {{ (row.totalDeposit ?? row.deposit ?? 0).toLocaleString() }}</span><span>{{ row.currency }} {{ (row.totalEffectiveBet ?? row.bet ?? 0).toLocaleString() }}</span><span>{{ relationshipFirstDepositAt(row) }}</span><span>{{ relationshipFirstDepositAmount(row) }}</span><span>{{ relationshipRegisterLabel(row) }}</span></div></div>
              </div>
              <div class="relationship-clean-row">
                <div class="relationship-clean-trigger" role="button" tabindex="0" @click="toggleRelationshipSection('totalPlayers')"><div class="relationship-clean-title-row"><span class="relationship-clean-title">總下級（含直屬）玩家</span><em>{{ relationshipExpanded.totalPlayers ? '收闔' : '展開' }}⌄</em></div><div class="relationship-clean-metrics"><div class="relationship-clean-metric"><small>玩家人數</small><strong>{{ relationshipFilteredRows('players', 'all').length }} 位</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總儲值金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'all', 'deposit')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'all', 'deposit'), $event)"><option value="">排序</option><option value="depositDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('players', 'all', 'deposit').toLocaleString() }}</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總有效投注</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'all', 'bet')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'all', 'bet'), $event)"><option value="">排序</option><option value="betDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('players', 'all', 'bet').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'all', 'firstDepositCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'all', 'firstDepositCount'), $event)"><option value="">排序</option><option value="firstDepositDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleCount('players', 'all') }} 位</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存總金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'all', 'firstDepositAmount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'all', 'firstDepositAmount'), $event)"><option value="">排序</option><option value="firstDepositAmountDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipCycleAmount('players', 'all').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期註冊人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'all', 'registeredCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'all', 'registeredCount'), $event)"><option value="">排序</option><option value="registeredDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleRegistered('players', 'all') }} 位</strong></div></div></div>
                <div v-if="relationshipExpanded.totalPlayers" class="relationship-clean-drawer"><div class="relationship-clean-drawer-head"><span>帳號</span><span>儲值總金額</span><span>有效投注</span><span>首存時間</span><span>首存金額</span><span>註冊時間</span></div><div v-for="row in relationshipDetailRows('players', 'all')" :key="row.id" class="relationship-clean-drawer-row"><strong>{{ row.account }}</strong><span>{{ row.currency }} {{ (row.totalDeposit ?? row.deposit ?? 0).toLocaleString() }}</span><span>{{ row.currency }} {{ (row.totalEffectiveBet ?? row.bet ?? 0).toLocaleString() }}</span><span>{{ relationshipFirstDepositAt(row) }}</span><span>{{ relationshipFirstDepositAmount(row) }}</span><span>{{ relationshipRegisterLabel(row) }}</span></div></div>
              </div>
              <div class="relationship-clean-row">
                <div class="relationship-clean-trigger" role="button" tabindex="0" @click="toggleRelationshipSection('directPlayers')"><div class="relationship-clean-title-row"><span class="relationship-clean-title">直屬下級玩家</span><em>{{ relationshipExpanded.directPlayers ? '收闔' : '展開' }}⌄</em></div><div class="relationship-clean-metrics"><div class="relationship-clean-metric"><small>玩家人數</small><strong>{{ relationshipFilteredRows('players', 'direct').length }} 位</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總儲值金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'direct', 'deposit')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'direct', 'deposit'), $event)"><option value="">排序</option><option value="depositDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('players', 'direct', 'deposit').toLocaleString() }}</strong></div><div class="relationship-clean-metric"><div class="relationship-metric-label"><small>總有效投注</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'direct', 'bet')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'direct', 'bet'), $event)"><option value="">排序</option><option value="betDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipSum('players', 'direct', 'bet').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'direct', 'firstDepositCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'direct', 'firstDepositCount'), $event)"><option value="">排序</option><option value="firstDepositDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleCount('players', 'direct') }} 位</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期首存總金額</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'direct', 'firstDepositAmount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'direct', 'firstDepositAmount'), $event)"><option value="">排序</option><option value="firstDepositAmountDesc">金額由大到小</option></select></div><strong>{{ selectedAgent.currency }} {{ relationshipCycleAmount('players', 'direct').toLocaleString() }}</strong></div><div class="relationship-clean-metric relationship-cycle-metric"><div class="relationship-metric-label"><small>結算週期註冊人數</small><select class="relationship-metric-sort" :value="relationshipSortMode[relationshipMetricKey('players', 'direct', 'registeredCount')] ?? ''" @click.stop @change="setRelationshipSort(relationshipMetricKey('players', 'direct', 'registeredCount'), $event)"><option value="">排序</option><option value="registeredDesc">時間由近而遠</option></select></div><strong>{{ relationshipCycleRegistered('players', 'direct') }} 位</strong></div></div></div>
                <div v-if="relationshipExpanded.directPlayers" class="relationship-clean-drawer"><div class="relationship-clean-drawer-head"><span>帳號</span><span>儲值總金額</span><span>有效投注</span><span>首存時間</span><span>首存金額</span><span>註冊時間</span></div><div v-for="row in relationshipDetailRows('players', 'direct')" :key="row.id" class="relationship-clean-drawer-row"><strong>{{ row.account }}</strong><span>{{ row.currency }} {{ (row.totalDeposit ?? row.deposit ?? 0).toLocaleString() }}</span><span>{{ row.currency }} {{ (row.totalEffectiveBet ?? row.bet ?? 0).toLocaleString() }}</span><span>{{ relationshipFirstDepositAt(row) }}</span><span>{{ relationshipFirstDepositAmount(row) }}</span><span>{{ relationshipRegisterLabel(row) }}</span></div></div>
              </div>
            </div>
            <div class="agent-info-rule"><strong>代理轉線規則</strong><p>僅限同幣別代理線；一級代理不可轉線。轉線於本次結算週期結束後生效，生效前訂單歸原代理線，生效後新訂單歸新代理線，不回溯重算歷史傭金。</p></div>
          </div>
          </main>
        </NLayout>
      </NLayout>

      <NModal v-model:show="showDeactivateAgentConfirm" preset="card" title="停用代理" class="modal-card">
        <p class="modal-intro">{{ pendingDeactivateAgent ? `即將停用代理「${pendingDeactivateAgent.account}」。` : '' }}</p>
        <div class="auth-warning"><strong>停用影響說明</strong><p>此操作只會停用代理帳號，不會移除代理線，也不會影響既有下級、歷史傭金與當期結算。若要將代理及其下級轉移至其他代理線，請使用「更換代理線」。</p></div>
        <template #footer><NSpace justify="end"><NButton @click="showDeactivateAgentConfirm = false">取消</NButton><NButton type="warning" @click="confirmDeactivateAgent">確認停用</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showAgentEdit" preset="card" title="編輯代理基本資料" class="modal-card">
        <p class="modal-intro">可修改顯示名稱、手機號碼、Email 與登入密碼；帳號、UID、幣別及傭金模式不可在此修改。</p>
        <NForm label-placement="top"><NFormItem label="真實姓名／顯示名稱"><NInput v-model:value="agentNameDraft" /></NFormItem><div class="form-two-col"><NFormItem label="手機號碼"><NInput v-model:value="agentPhoneDraft" /></NFormItem><NFormItem label="Email"><NInput v-model:value="agentEmailDraft" /></NFormItem></div><NFormItem label="登入密碼"><NInput v-model:value="agentPasswordDraft" type="password" show-password-on="click" placeholder="預設為空白，留白代表不修改密碼" /><template #feedback></template></NFormItem><p class="modal-help">密碼欄位留白送出時，不會修改原登入密碼；只有填寫新密碼才會觸發修改。</p></NForm>
        <template #footer><NSpace justify="end"><NButton @click="showAgentEdit = false">取消</NButton><NButton type="primary" @click="saveAgentProfile">儲存</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showAgentTransfer" preset="card" title="更換代理線" class="modal-card">
        <p class="modal-intro">代理轉線會在本次結算週期結束後生效。請選擇同幣別的新上級代理線。</p>
        <NForm label-placement="top"><NFormItem label="目前代理線"><NInput :value="displayAgentPath(selectedAgent?.path)" readonly /></NFormItem><NFormItem label="新代理帳號"><NSelect v-model:value="agentTransferTarget" filterable :options="agentTransferOptions" placeholder="搜尋並選擇新代理帳號" /></NFormItem></NForm>
        <div v-if="selectedTransferTarget" class="transfer-preview"><div class="transfer-preview-title">更換後代理線預覽</div><strong>{{ projectedTransferPath }}</strong><span>新上級代理：{{ selectedTransferTarget.account }} · {{ selectedTransferTarget.level }}</span></div>
        <div class="auth-warning"><strong>生效時間：{{ nextSettlementEffectiveAt }}</strong><p>生效前訂單仍歸原代理線；生效後的新訂單歸新代理線，歷史傭金不回溯重算。</p></div>
        <template #footer><NSpace justify="end"><NButton @click="showAgentTransfer = false">取消</NButton><NButton type="primary" @click="submitAgentTransfer">下一步確認</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showAgentTransferConfirm" preset="card" title="確認代理轉線" class="modal-card">
        <p class="modal-intro">請確認將「{{ pendingAgentTransfer?.account }}」轉至「{{ pendingAgentTransferTarget?.account }}」代理線。</p>
        <div class="transfer-path-summary"><span>目前代理線</span><strong>{{ displayAgentPath(pendingAgentTransfer?.path) }}</strong><span>新代理線</span><strong>{{ pendingAgentTransferTarget ? displayAgentPath(`${pendingAgentTransferTarget.path} > ${pendingAgentTransfer?.account}`) : '-' }}</strong></div>
        <div class="auth-warning"><strong>本次結算週期結束後生效：{{ nextSettlementEffectiveAt }}</strong><p>只適用同幣別代理線；生效前後訂單分別歸屬原／新代理線，不回溯重算歷史傭金。</p></div>
        <template #footer><NSpace justify="end"><NButton @click="showAgentTransferConfirm = false">返回修改</NButton><NButton type="warning" @click="confirmAgentTransfer">確認轉線</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showLoginTwoFactorSetup" preset="card" title="首次登入：綁定 Google Auth" class="modal-card auth-admin-modal">
        <div class="auth-admin-panel">
          <div class="auth-admin-heading"><div><span>登入帳號</span><strong>{{ loginTwoFactorAccount }}</strong></div><NTag type="warning" round>尚未綁定</NTag></div>
          <p class="modal-help">系統已確認帳號密碼，且此帳號需要 Google Auth。請先掃描 QR Code，完成綁定後輸入目前的 6 位數驗證碼。</p>
          <div class="auth-qr-layout"><div class="auth-qr" aria-label="Google Auth QR Code 示意"><span>QR</span></div><div><strong>使用 Google Authenticator 掃描</strong><ol><li>開啟 Google Authenticator。</li><li>掃描 QR Code，或手動輸入密鑰。</li><li>在下方輸入綁定後顯示的驗證碼。</li></ol><p class="auth-secret">手動密鑰：JBSW Y3DP EHPK 3PXP</p></div></div>
          <NFormItem label="Google Auth 驗證碼"><NInput v-model:value="loginTwoFactorCode" inputmode="numeric" maxlength="6" placeholder="請輸入 6 位數驗證碼" @keyup.enter="completeLoginTwoFactorBind" /></NFormItem>
          <p class="modal-help">驗證成功後才會完成綁定並進入代理後台。</p>
        </div>
        <template #footer><NSpace justify="end"><NButton @click="cancelLoginTwoFactor">取消</NButton><NButton type="primary" @click="completeLoginTwoFactorBind">確認綁定</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showLoginTwoFactorVerify" preset="card" title="Google Auth 驗證" class="modal-card">
        <p class="modal-intro">帳號密碼已確認。請輸入 Google Authenticator 目前顯示的 6 位數驗證碼。</p>
        <NFormItem label="Google Auth 驗證碼"><NInput v-model:value="loginTwoFactorCode" inputmode="numeric" maxlength="6" placeholder="請輸入 6 位數驗證碼" @keyup.enter="verifyLoginTwoFactor" /></NFormItem>
        <template #footer><NSpace justify="end"><NButton @click="cancelLoginTwoFactor">取消</NButton><NButton type="primary" @click="verifyLoginTwoFactor">確認並登入</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showAgentDetailModal" preset="card" :title="selectedAgent ? `代理詳情 · ${selectedAgent.account}` : '代理詳情'" class="modal-card">
        <template v-if="selectedAgent">
           <div class="detail-tabs"><button :class="{ active: detailTab === 'basic' }" @click="detailTab = 'basic'">基本資料</button><button :class="{ active: detailTab === 'wallet' }" @click="detailTab = 'wallet'">即時資料</button><button :class="{ active: detailTab === 'relationship' }" @click="detailTab = 'relationship'">代理報表</button><button :class="{ active: detailTab === 'commission' }" @click="detailTab = 'commission'">傭金方案</button><button :class="{ active: detailTab === 'withdrawals' }" @click="detailTab = 'withdrawals'">傭金帳變紀錄</button><button :class="{ active: detailTab === 'logs' }" @click="detailTab = 'logs'">操作紀錄</button></div>
           <div v-if="detailTab === 'basic'" class="agent-detail-grid">
            <div><span>登入帳號</span><strong>{{ selectedAgent.account }}</strong></div><div><span>登入密碼</span><strong>••••••••</strong><p class="modal-help">如需重設請聯繫平台客服。</p></div>
            <div><span>帳號類型</span><strong>{{ selectedAgent.level }}</strong><NTag size="small" round>不可修改</NTag></div><div><span>代理 UID（系統生成）</span><strong>{{ selectedAgent.uid || selectedAgent.id }}</strong></div>
             <div><span>玩家推廣碼</span><div class="detail-inline"><strong>{{ selectedAgent.referralCode || '未設定' }}</strong><NButton v-if="selectedAgent.referralCode" size="small" quaternary @click="copyReferral(selectedAgent.referralCode, '玩家推廣碼')">複製</NButton></div></div><div><span>真實姓名</span><strong>{{ selectedAgent.displayName || '未填寫' }}</strong></div>
            <div><span>手機號碼</span><strong>{{ maskPhone(selectedAgent.phone) }}</strong></div><div><span>聯絡方式</span><strong>{{ selectedAgent.contactMethod || '未設定' }}</strong></div>
             <div><span>Email</span><strong>{{ maskEmail(selectedAgent.email) }}</strong></div><div><span>2FA 雙重驗證</span><div class="detail-inline"><NTag size="small" :type="selectedAgent.twoFactor === '已啟用' ? 'success' : 'default'" round>{{ selectedAgent.twoFactor || '未啟用' }}</NTag><NButton v-if="canManageTwoFactor(selectedAgent)" size="small" quaternary @click="openTwoFactorAdmin(selectedAgent)">管理 Google Auth</NButton></div><p class="modal-help">代理遺失驗證器時，由運營商重設後重新綁定。</p></div>
             <div><span>幣別</span><strong>{{ selectedAgent.currency }}</strong></div><div><span>代理傭金錢包餘額</span><strong class="positive">{{ selectedAgent.currency }} {{ selectedAgent.walletBalance.toLocaleString() }}</strong></div>
           </div>
           <div v-else-if="detailTab === 'wallet'" class="agent-detail-grid"><div><span>代理傭金錢包餘額</span><strong class="positive">{{ selectedAgent.currency }} {{ selectedAgent.walletBalance.toLocaleString() }}</strong></div><div><span>待結算傭金</span><strong>{{ selectedAgent.currency }} 12,680</strong></div><div><span>本期產生傭金</span><strong>{{ selectedAgent.currency }} 28,460</strong></div><div><span>可提領狀態</span><NTag type="success" round>可提領</NTag></div><div class="full"><span>錢包權限</span><p class="modal-help">代理傭金錢包僅可查看與申請提領；資金帳變不可轉帳、加扣款，提領需由運營商審核。</p></div></div>
           <div v-else-if="detailTab === 'relationship'" class="agent-detail-grid"><div class="full"><span>完整樹狀路徑</span><strong>{{ selectedAgent.path }}</strong></div><div><span>代理層級</span><strong>{{ selectedAgent.level }}</strong></div><div><span>直屬下級數</span><strong>{{ selectedAgent.children }} 位</strong></div><div class="full"><span>關係說明</span><p class="modal-help">此代理只能隸屬一條代理線；轉移代理線時，生效前後訂單依原／新代理線歸屬，不回溯重算歷史傭金。</p></div></div>
           <div v-else-if="detailTab === 'commission'" class="agent-plan-overview"><div class="agent-plan-overview-head"><div><span>上級給予此代理的方案</span><strong>{{ selectedAgentPlan?.name || '未套用方案' }}</strong></div><NTag type="info" round>唯讀查看</NTag></div><div v-if="selectedAgentPlan" class="agent-plan-overview-grid"><div><span>方案名稱</span><strong>{{ selectedAgentPlan.name }}</strong></div><div><span>建立代理</span><strong>{{ selectedAgentPlan.createdBy || selectedAgentPlan.parentAccount }}</strong></div><div><span>傭金模式</span><strong>{{ selectedAgentPlan.model }}</strong></div><div><span>結算週期</span><strong>{{ selectedAgentPlan.cycle }}</strong></div><div><span>默認可分配點數</span><strong>{{ selectedAgentPlan.allocationRate }}%</strong></div><div><span>方案規則</span><strong>{{ selectedAgentPlan.description }}</strong></div></div><div class="agent-plan-overview-section"><strong>此代理建立給直屬下級的方案</strong><div v-for="plan in selectedAgentCreatedPlans" :key="plan.id" class="agent-downline-plan-row"><div><strong>{{ plan.name }}</strong><small>{{ plan.model }} · {{ plan.cycle }}</small></div><strong>{{ plan.allocationRate }}%</strong><span>套用 {{ planAppliedAgents(plan).length }} 位</span></div><p v-if="!selectedAgentCreatedPlans.length" class="modal-help">此代理尚未建立給直屬下級的方案。</p></div></div>
           <div v-else-if="detailTab === 'withdrawals'" class="agent-withdrawal-panel"><div class="table-section-label"><strong>傭金帳變紀錄</strong><span>整合傭金提領與傭金發放報表；本頁僅供查看詳情</span></div><div class="filter-row filter-row-advanced agent-withdrawal-filter"><div class="filter-field-group"><label>帳變類型</label><NSelect v-model:value="agentWithdrawalType" clearable placeholder="全部類型" :options="[{label:'傭金發放',value:'傭金發放'},{label:'傭金提領',value:'傭金提領'}]" class="filter-select" /></div><div class="filter-field-group filter-date-group"><label>發生時間</label><NDatePicker v-model:value="agentWithdrawalDateRange" type="daterange" clearable format="yyyy-MM-dd" placeholder="選擇時間區段" /></div><div class="filter-field-group"><label>狀態</label><NSelect v-model:value="agentWithdrawalStatus" clearable placeholder="全部狀態" :options="[{label:'待處理',value:'待處理'},{label:'處理中',value:'處理中'},{label:'成功',value:'成功'},{label:'失敗',value:'失敗'}]" class="filter-select" /></div><div class="filter-field-group filter-account-group"><label>關鍵字／單號</label><NInput v-model:value="agentWithdrawalKeyword" clearable placeholder="搜尋單號、銀行、方案或備註" class="filter-field" /></div></div><div class="withdrawal-record-list"><div v-for="record in filteredAgentWithdrawalRecords" :key="record.id" class="withdrawal-record-item account-record-item"><div><NTag size="small" round :type="record.recordType === '傭金發放' ? 'info' : 'warning'">{{ record.recordType }}</NTag><strong>{{ record.id }}</strong><small>{{ record.applyAt }} · {{ record.settlePeriod }}</small></div><div><strong :class="record.recordType === '傭金發放' ? 'positive' : 'negative'">{{ record.recordType === '傭金發放' ? '+' : '-' }}{{ selectedAgent.currency }} {{ record.netAmount.toLocaleString() }}</strong><small>餘額 {{ selectedAgent.currency }} {{ record.balanceAfter.toLocaleString() }}</small></div><NTag size="small" round :type="record.status === '成功' ? 'success' : record.status === '失敗' ? 'error' : 'warning'">{{ record.status }}</NTag><NButton size="small" quaternary @click="openCommissionRecordDetail(record)">查看詳情</NButton></div><p v-if="!filteredAgentWithdrawalRecords.length" class="modal-help">目前沒有符合條件的傭金帳變紀錄。</p></div></div>
           <div v-else class="detail-log-list"><div v-for="log in logs.filter((item) => item.detail.includes(selectedAgent?.account ?? '')).slice(0, 5)" :key="`${log.time}-${log.detail}`" class="recent-row"><div class="log-dot" /><div><strong>{{ log.type }}</strong><span>{{ log.detail }}</span></div><time>{{ log.time }}</time></div><p v-if="!logs.some((item) => item.detail.includes(selectedAgent?.account ?? ''))" class="modal-help">目前沒有此代理的操作紀錄。</p></div>
       </template>
       <template #footer><NSpace justify="space-between" style="width: 100%"><NButton secondary @click="selectedAgent && deactivateAgent(selectedAgent)">狀態管理</NButton><NSpace><NButton @click="showAgentDetailModal = false">關閉</NButton></NSpace></NSpace></template>
      </NModal>
       <NModal v-model:show="showCommissionRecordDetail" preset="card" title="傭金帳變詳情" class="modal-card">
        <template v-if="selectedCommissionRecord">
          <div class="agent-detail-grid"><div><span>帳變類型</span><strong>{{ selectedCommissionRecord.recordType }}</strong></div><div><span>帳變單號</span><strong class="code-text">{{ selectedCommissionRecord.id }}</strong></div><div><span>紀錄產生時間</span><strong>{{ selectedCommissionRecord.applyAt }}</strong></div><div><span>異動時間</span><strong>{{ selectedCommissionRecord.reviewAt }}</strong></div><div><span>結算週期</span><strong>{{ selectedCommissionRecord.settlePeriod }}</strong></div><div><span>帳變金額</span><strong :class="selectedCommissionRecord.recordType === '傭金發放' ? 'positive' : 'negative'">{{ selectedCommissionRecord.recordType === '傭金發放' ? '+' : '-' }}{{ selectedAgent?.currency }} {{ selectedCommissionRecord.netAmount.toLocaleString() }}</strong></div><div><span>帳變前餘額</span><strong>{{ selectedAgent?.currency }} {{ selectedCommissionRecord.balanceBefore.toLocaleString() }}</strong></div><div><span>帳變後餘額</span><strong>{{ selectedAgent?.currency }} {{ selectedCommissionRecord.balanceAfter.toLocaleString() }}</strong></div><div><span>狀態</span><NTag round :type="selectedCommissionRecord.status === '成功' ? 'success' : selectedCommissionRecord.status === '失敗' ? 'error' : 'warning'">{{ selectedCommissionRecord.status }}</NTag></div><template v-if="selectedCommissionRecord.recordType === '傭金提領'"><div><span>綁定銀行</span><strong>{{ selectedCommissionRecord.bank }}</strong></div><div><span>銀行帳號</span><strong>{{ selectedCommissionRecord.bankAccount || '****9012' }}</strong></div><div><span>戶名</span><strong>{{ selectedCommissionRecord.bankHolder || 'Klaus Lin' }}</strong></div></template><template v-else><div><span>傭金方案</span><strong>{{ selectedCommissionRecord.plan || selectedCommissionRecord.bank }}</strong></div><div><span>結算基數</span><strong>{{ selectedCommissionRecord.settlementBase || '有效投注總額' }}</strong></div><div><span>系統處理結果</span><strong>{{ selectedCommissionRecord.systemResult || selectedCommissionRecord.remark }}</strong></div></template><div class="full"><span>備註</span><p class="modal-help">{{ selectedCommissionRecord.reviewer }} · {{ selectedCommissionRecord.remark }}</p></div><div v-if="selectedCommissionRecord.recordType === '傭金提領'" class="full"><span>費用</span><strong>{{ selectedAgent?.currency }} {{ selectedCommissionRecord.fee.toLocaleString() }}</strong></div></div>
        </template>
        <template #footer><NSpace justify="end"><NButton @click="showCommissionRecordDetail = false">關閉</NButton></NSpace></template>
       </NModal>
       <NModal v-model:show="showWithdrawalProcess" preset="card" title="傭金提領紀錄詳情／出款處理" class="modal-card">
         <template v-if="selectedWithdrawalOrder">
           <div class="agent-detail-grid"><div><span>帳號</span><strong>{{ selectedWithdrawalOrder.account }}</strong></div><div><span>訂單編號</span><strong class="code-text">{{ selectedWithdrawalOrder.id }}</strong></div><div><span>金額</span><strong class="negative">- {{ selectedWithdrawalOrder.currency }} {{ selectedWithdrawalOrder.amount.toLocaleString() }}</strong></div><div><span>狀態</span><NTag round :type="selectedWithdrawalOrder.status === '成功' ? 'success' : selectedWithdrawalOrder.status === '失敗' ? 'error' : 'warning'">{{ selectedWithdrawalOrder.status }}</NTag></div><div><span>紀錄產生時間</span><strong>{{ selectedWithdrawalOrder.createdAt }}</strong></div><div><span>異動時間</span><strong>{{ selectedWithdrawalOrder.updatedAt }}</strong></div><div><span>綁定銀行</span><strong>{{ selectedWithdrawalOrder.bankName }}</strong></div><div><span>銀行帳號</span><strong>{{ selectedWithdrawalOrder.bankAccount }}</strong></div><div><span>戶名</span><strong>{{ selectedWithdrawalOrder.bankHolder }}</strong></div><div v-if="selectedWithdrawalOrder.failureReason && selectedWithdrawalOrder.status === '失敗'" class="full"><span>失敗原因</span><strong class="negative">{{ selectedWithdrawalOrder.failureReason }}</strong></div><div v-if="selectedWithdrawalOrder.remark" class="full"><span>操作備註</span><p class="modal-help">{{ selectedWithdrawalOrder.remark }}</p></div></div>
            <p v-if="currentRole === '運營商' && selectedWithdrawalOrder.status === '處理中'" class="modal-help">請選擇出款結果；點擊後將進入二次確認並填寫原因備註。</p>
         </template>
          <template #footer><NSpace justify="end"><NButton @click="showWithdrawalProcess = false">關閉</NButton><template v-if="currentRole === '運營商' && selectedWithdrawalOrder?.status === '處理中'"><NButton type="error" secondary @click="prepareCommissionResult('withdrawal', '失敗')">失敗</NButton><NButton type="primary" @click="prepareCommissionResult('withdrawal', '成功')">成功</NButton></template></NSpace></template>
       </NModal>
       <NModal v-model:show="showPayoutProcess" preset="card" title="傭金發放紀錄詳情／手動處理" class="modal-card">
         <template v-if="selectedPayoutRecord">
           <div class="agent-detail-grid"><div><span>帳號</span><strong>{{ selectedPayoutRecord.account }}</strong></div><div><span>訂單編號</span><strong class="code-text">{{ selectedPayoutRecord.id }}</strong></div><div><span>金額</span><strong class="positive">+ {{ selectedPayoutRecord.currency }} {{ selectedPayoutRecord.amount.toLocaleString() }}</strong></div><div><span>狀態</span><NTag round :type="selectedPayoutRecord.status === '成功' ? 'success' : selectedPayoutRecord.status === '失敗' ? 'error' : 'warning'">{{ selectedPayoutRecord.status }}</NTag></div><div><span>紀錄產生時間</span><strong>{{ selectedPayoutRecord.createdAt }}</strong></div><div><span>異動時間</span><strong>{{ selectedPayoutRecord.updatedAt }}</strong></div><div><span>傭金方案</span><strong>{{ selectedPayoutRecord.plan }}</strong></div><div><span>結算週期</span><strong>{{ selectedPayoutRecord.cycle }}</strong></div><div class="full"><span>結算基數</span><strong>{{ selectedPayoutRecord.settlementBase }}</strong></div><div class="full"><span>系統處理結果</span><strong>{{ selectedPayoutRecord.systemResult }}</strong></div><div class="full"><span>備註</span><p class="modal-help">{{ selectedPayoutRecord.remark }}</p></div></div>
            <p v-if="currentRole === '運營商' && selectedPayoutRecord.status === '處理中'" class="modal-help">請選擇系統處理結果；點擊後將進入二次確認並填寫原因備註。</p>
         </template>
          <template #footer><NSpace justify="end"><NButton @click="showPayoutProcess = false">關閉</NButton><template v-if="currentRole === '運營商' && selectedPayoutRecord?.status === '處理中'"><NButton type="error" secondary @click="prepareCommissionResult('payout', '失敗')">失敗</NButton><NButton type="primary" @click="prepareCommissionResult('payout', '成功')">成功</NButton></template></NSpace></template>
        </NModal>
        <NModal v-model:show="showCommissionActionConfirm" preset="card" title="確認傭金帳務操作" class="modal-card">
           <p class="modal-intro">即將將此筆{{ pendingCommissionAction === 'payout' ? '傭金發放' : '傭金提領' }}紀錄更新為「{{ pendingCommissionResult }}」，請確認後送出。</p>
           <NForm v-if="pendingCommissionAction !== 'withdrawal-start'" label-placement="top"><NFormItem label="原因備註"><NInput v-model:value="commissionActionRemark" type="textarea" :rows="4" placeholder="請填寫本次操作原因，送出後會寫入紀錄。" /></NFormItem></NForm>
          <div class="auth-warning"><strong>二次確認</strong><p>此操作會更新報表狀態並保留操作人與異動時間，請確認資料無誤。</p></div>
          <template #footer><NSpace justify="end"><NButton @click="showCommissionActionConfirm = false">取消</NButton><NButton type="warning" @click="confirmCommissionResult">確認{{ pendingCommissionResult }}</NButton></NSpace></template>
        </NModal>
       <NModal v-model:show="showTwoFactorAdmin" preset="card" title="Google Auth 管理" class="modal-card auth-admin-modal">
        <template v-if="selectedTwoFactorAgent">
          <div class="auth-admin-panel">
            <div class="auth-admin-heading"><div><span>代理帳號</span><strong>{{ selectedTwoFactorAgent.account }}</strong></div><NTag :type="selectedTwoFactorAgent.twoFactor === '已啟用' ? 'success' : 'warning'" round>{{ selectedTwoFactorAgent.twoFactor || '未啟用' }}</NTag></div>
            <div class="auth-admin-grid"><div><span>綁定時間</span><strong>{{ selectedTwoFactorAgent.twoFactorBoundAt || '尚未綁定' }}</strong></div><div><span>本次操作權限</span><strong>{{ currentRole === '運營商' ? '全部代理' : '自己下線' }}</strong></div></div>
            <div class="auth-warning"><strong>代理遺失 Google Auth 驗證器？</strong><p>重設後，原驗證器立即失效，代理會恢復為尚未綁定狀態；新的 QR Code 可在此查看，也會在代理下次登入的綁定流程中使用。</p></div>
            <NSpace><NButton secondary :disabled="selectedTwoFactorAgent.twoFactorRequired === false" @click="viewTwoFactorQr">查看目前 QR Code</NButton><NButton type="warning" :disabled="selectedTwoFactorAgent.twoFactorRequired === false" @click="prepareTwoFactorReset">重設 QR Code</NButton></NSpace>
          </div>
        </template>
        <template #footer><NSpace justify="end"><NButton @click="showTwoFactorAdmin = false">關閉</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showTwoFactorQr" preset="card" title="查看目前 QR Code" class="modal-card auth-admin-modal">
        <template v-if="selectedTwoFactorAgent">
          <div class="auth-admin-heading"><div><span>代理帳號</span><strong>{{ selectedTwoFactorAgent.account }}</strong></div><NTag :type="selectedTwoFactorAgent.twoFactor === '已啟用' ? 'success' : 'warning'" round>{{ selectedTwoFactorAgent.twoFactor === '已啟用' ? '目前已綁定' : '重設後待綁定' }}</NTag></div>
          <div class="auth-qr-layout auth-qr-modal-content"><div class="auth-qr" aria-label="Google Auth QR Code 示意"><span>QR</span></div><div><strong>{{ selectedTwoFactorAgent.twoFactor === '已啟用' ? '目前綁定 QR Code' : '下一次登入使用的 QR Code' }}</strong><p class="modal-help">{{ selectedTwoFactorAgent.twoFactor === '已啟用' ? '此為目前綁定中的 QR Code。' : '此為重設後下一次登入綁定流程會提供的 QR Code，現在即可查看。' }}查看行為已寫入操作日誌。</p><p class="auth-secret">手動密鑰：JBSW Y3DP EHPK 3PXP</p></div></div>
        </template>
        <template #footer><NSpace justify="end"><NButton type="primary" @click="showTwoFactorQr = false">關閉</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showTwoFactorResetConfirm" preset="card" title="重設 QR Code" class="modal-card">
        <p class="modal-intro">{{ selectedTwoFactorAgent?.twoFactorLastResetAt ? `上次已於${selectedTwoFactorAgent.twoFactorLastResetAt}重置，目前狀態為${selectedTwoFactorAgent.twoFactor === '已啟用' ? '已綁定' : '未綁定'}，請問是否還要繼續操作？` : '將會重置代理登入時的驗證二維碼，請問是否確認。' }}</p>
        <div class="auth-warning"><strong>重設後的登入流程</strong><p>目前綁定會立即失效，代理將視為尚未綁定。若帳號需要 2FA，下次登入確認帳號密碼後才會顯示新的綁定 QR Code，並須輸入驗證碼完成綁定。</p></div>
        <template #footer><NSpace justify="end"><NButton @click="showTwoFactorResetConfirm = false">取消</NButton><NButton type="warning" @click="completeTwoFactorReset">確認重設</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showTwoFactorToggleConfirm" preset="card" title="修改 2FA 要求" class="modal-card">
        <p class="modal-intro">即將將此代理的登入 2FA 要求改為「{{ pendingTwoFactorRequired ? '啟用' : '停用' }}」，請問是否確認？</p>
        <div class="auth-warning"><strong>操作影響</strong><p>{{ pendingTwoFactorRequired ? '代理下次登入時需要 Google Auth；若尚未綁定，登入確認帳密後會顯示綁定 QR Code。' : '代理登入時不再要求 Google Auth；既有綁定資料會保留，之後重新啟用時可繼續使用。' }}</p></div>
        <template #footer><NSpace justify="end"><NButton @click="showTwoFactorToggleConfirm = false">取消</NButton><NButton type="primary" @click="confirmToggleAgentTwoFactor">確認修改</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showWithdrawalStatusConfirm" preset="card" title="修改提領狀態" class="modal-card">
        <p class="modal-intro">即將將代理「{{ pendingWithdrawalAgent?.account }}」的提領狀態改為「{{ pendingWithdrawalEnabled ? '可提領' : '不可提領' }}」，請問是否確認？</p>
        <div class="auth-warning"><strong>操作影響</strong><p>{{ pendingWithdrawalEnabled ? '代理符合其他提領條件時可提出提領申請。' : '代理仍可查看錢包與歷史紀錄，但不可提出新的提領申請。' }}</p></div>
        <template #footer><NSpace justify="end"><NButton @click="showWithdrawalStatusConfirm = false">取消</NButton><NButton type="primary" @click="confirmToggleWithdrawalStatus">確認修改</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showWalletAdjustment" preset="card" title="人工加扣款" class="modal-card">
        <p class="modal-intro">此操作會直接異動代理傭金錢包，僅限運營商操作，送出前會再次確認。</p>
        <NForm label-placement="top"><div class="form-two-col"><NFormItem label="調整類型"><NSelect v-model:value="walletAdjustmentType" :options="[{label:'加款',value:'加款'},{label:'扣款',value:'扣款'}]" /></NFormItem><NFormItem label="調整金額"><NInputNumber v-model:value="walletAdjustmentAmount" :min="0" style="width: 100%" /></NFormItem></div><NFormItem label="調整原因"><NInput v-model:value="walletAdjustmentReason" type="textarea" :rows="3" placeholder="請輸入人工加扣款原因" /></NFormItem></NForm>
        <template #footer><NSpace justify="end"><NButton @click="showWalletAdjustment = false">取消</NButton><NButton type="primary" @click="submitWalletAdjustment">下一步確認</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showWalletAdjustmentConfirm" preset="card" title="確認人工加扣款" class="modal-card">
        <p class="modal-intro">即將對代理「{{ selectedAgent?.account }}」執行{{ walletAdjustmentType }} {{ selectedAgent?.currency }} {{ walletAdjustmentAmount?.toLocaleString() }}，請問是否確認？</p>
        <div class="auth-warning"><strong>調整原因</strong><p>{{ walletAdjustmentReason }}</p></div>
        <template #footer><NSpace justify="end"><NButton @click="showWalletAdjustmentConfirm = false">返回修改</NButton><NButton type="warning" @click="confirmWalletAdjustment">確認加扣款</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showPlayerDetailModal" preset="card" :title="selectedPlayer ? `玩家詳情 · ${selectedPlayer.account}` : '玩家詳情'" class="modal-card">
        <template v-if="selectedPlayer">
          <div class="detail-tabs"><button :class="{ active: playerDetailTab === 'basic' }" @click="playerDetailTab = 'basic'">基本資料</button><button :class="{ active: playerDetailTab === 'wallet' }" @click="playerDetailTab = 'wallet'">即時資料</button><button :class="{ active: playerDetailTab === 'vip' }" @click="playerDetailTab = 'vip'">VIP 資訊</button><button :class="{ active: playerDetailTab === 'promotion' }" @click="playerDetailTab = 'promotion'">優惠紀錄</button><button :class="{ active: playerDetailTab === 'audit' }" @click="playerDetailTab = 'audit'">操作稽核</button><button :class="{ active: playerDetailTab === 'asset' }" @click="playerDetailTab = 'asset'">資金帳變</button><button :class="{ active: playerDetailTab === 'game' }" @click="playerDetailTab = 'game'">遊戲紀錄</button><button :class="{ active: playerDetailTab === 'transfer' }" @click="playerDetailTab = 'transfer'">轉線紀錄</button><button :class="{ active: playerDetailTab === 'invite' }" @click="playerDetailTab = 'invite'">邀請明細</button><button :class="{ active: playerDetailTab === 'agent' }" @click="playerDetailTab = 'agent'">代理關係</button></div>
          <div v-if="playerDetailTab === 'basic'" class="agent-detail-grid"><div><span>玩家 ID</span><strong>{{ selectedPlayer.id }}</strong></div><div><span>玩家帳號</span><strong>{{ selectedPlayer.account }}</strong></div><div><span>顯示名稱</span><strong>{{ selectedPlayer.displayName }}</strong></div><div><span>手機號碼</span><strong>{{ selectedPlayer.phone || '-' }}</strong></div><div><span>Email</span><strong>{{ selectedPlayer.email || '-' }}</strong></div><div><span>VIP 等級</span><strong>{{ selectedPlayer.vipLevel }}</strong></div><div><span>性別</span><strong>{{ selectedPlayer.gender || '-' }}</strong></div><div><span>生日</span><strong>{{ selectedPlayer.birthday || '-' }}</strong></div><div><span>註冊來源</span><strong>{{ selectedPlayer.registerSource }}</strong></div><div><span>自身邀請碼</span><strong class="code-text">{{ selectedPlayer.inviteCode }}</strong></div><div><span>註冊 IP</span><strong>{{ selectedPlayer.registerIp }}</strong></div><div><span>最近登入</span><strong>{{ selectedPlayer.lastLoginAt }}</strong></div><div><span>最近登入 IP</span><strong>{{ selectedPlayer.lastLoginIp }}</strong></div><div><span>連續簽到</span><strong>{{ selectedPlayer.consecutiveCheckInDays }} 天</strong></div><div><span>帳號狀態</span><NTag :type="playerStatusType(selectedPlayer.status)" round>{{ selectedPlayer.status }}</NTag></div><div><span>在線狀態</span><NTag :type="selectedPlayer.isOnline ? 'success' : 'default'" round>{{ selectedPlayer.isOnline ? '在線' : '離線' }}</NTag></div><div class="full"><span>標籤</span><div class="table-tags"><NTag v-for="tag in selectedPlayer.tags" :key="tag" size="small" round>{{ tag }}</NTag></div></div></div>
          <div v-else-if="playerDetailTab === 'wallet'" class="agent-detail-grid"><div><span>現金錢包</span><strong>{{ selectedPlayer.currency }} 12,800</strong></div><div><span>活動錢包</span><strong>{{ selectedPlayer.currency }} 3,260</strong></div><div><span>遊戲錢包</span><strong>{{ selectedPlayer.currency }} 1,840</strong></div><div><span>安全錢包</span><strong>{{ selectedPlayer.currency }} 0</strong></div><div><span>本月儲值</span><strong>{{ selectedPlayer.currency }} {{ selectedPlayer.deposit.toLocaleString() }}</strong></div><div><span>本月提領</span><strong>{{ selectedPlayer.currency }} 4,200</strong></div><div><span>最後儲值時間</span><strong>2026-08-31 10:22:04</strong></div><div><span>最後提領狀態</span><NTag type="warning" round>審核中</NTag></div><div class="full"><span>帳號權限</span><p class="modal-help">禁言：正常；優惠：開啟；儲值：開啟；遊戲：開啟。代理角色只能查看，不能修改權限。</p></div></div>
          <div v-else-if="playerDetailTab === 'vip'" class="agent-detail-grid"><div><span>VIP 等級</span><strong>{{ selectedPlayer.vipLevel }}</strong></div><div><span>本月 VIP 累計</span><strong>儲值 {{ selectedPlayer.deposit.toLocaleString() }}／投注 {{ selectedPlayer.bet.toLocaleString() }}</strong></div><div><span>下級門檻進度</span><strong>78%</strong></div><div><span>升級預估</span><strong>再投注 {{ selectedPlayer.currency }} 12,400</strong></div><div><span>本月獎勵</span><strong>{{ selectedPlayer.currency }} 1,200</strong></div><div><span>最後升級時間</span><strong>2026-08-01 00:00:12</strong></div><div class="full"><span>VIP 資訊</span><p class="modal-help">VIP 升降級與獎勵依運營後台規則執行；代理後台僅提供查詢。</p></div></div>
          <div v-else-if="playerDetailTab === 'promotion'" class="detail-list"><div class="detail-list-row"><span>優惠紀錄</span><strong>本期已領取 2 筆，待領取 1 筆</strong></div><div class="detail-list-row"><span>最近活動</span><strong>VIP3 升級禮 · 已完成</strong></div><div class="detail-list-row"><span>待領取優惠</span><strong>{{ selectedPlayer.currency }} 800 · 首存回饋</strong></div><div class="detail-list-row"><span>最後領取時間</span><strong>2026-08-29 13:42:10</strong></div><div class="detail-list-row"><span>流水要求</span><strong>8 倍（目前完成 6.4 倍）</strong></div><p class="modal-help">優惠領取與派發由運營後台管理；代理只能查看下級玩家的優惠狀況。</p></div>
          <div v-else-if="playerDetailTab === 'audit'" class="detail-list"><div v-for="item in [{type:'登入',time:'2026-08-31 10:22:04',detail:'登入成功 · 10.20.8.45'},{type:'VIP 等級更新',time:'2026-08-30 00:00:12',detail:'VIP2 → VIP3'},{type:'領取優惠',time:'2026-08-29 13:42:10',detail:'首存回饋 TWD 800'},{type:'修改安全設定',time:'2026-08-27 09:11:08',detail:'Google Auth 綁定成功'}]" :key="item.time" class="detail-list-row"><span>{{ item.type }}</span><strong>{{ item.time }}<small class="mode-summary">{{ item.detail }}</small></strong></div><p class="modal-help">此頁僅呈現該玩家的操作稽核摘要。</p></div>
          <div v-else-if="playerDetailTab === 'asset'" class="detail-list"><div class="detail-list-row"><span>最近一筆資金帳變</span><strong>{{ selectedPlayer.currency }} +500 儲值成功</strong></div><div class="detail-list-row"><span>本期資金異動筆數</span><strong>18 筆</strong></div><div class="detail-list-row"><span>本月儲值總額</span><strong>{{ selectedPlayer.currency }} {{ selectedPlayer.deposit.toLocaleString() }}</strong></div><div class="detail-list-row"><span>本月提領總額</span><strong>{{ selectedPlayer.currency }} 4,200</strong></div><div class="detail-list-row"><span>最後異動時間</span><strong>2026-08-31 10:22:04</strong></div><p class="modal-help">資金帳變僅供查看，不可在代理後台進行轉帳、加扣款或撤銷。</p></div>
          <div v-else-if="playerDetailTab === 'game'" class="detail-list"><div class="detail-list-row"><span>最近投注</span><strong>{{ selectedPlayer.currency }} {{ selectedPlayer.bet.toLocaleString() }}</strong></div><div class="detail-list-row"><span>本期有效投注</span><strong>{{ selectedPlayer.currency }} 86,420</strong></div><div class="detail-list-row"><span>最近遊戲</span><strong>Sport · Live Casino</strong></div><div class="detail-list-row"><span>熱門遊戲</span><strong>足球盤口／百家樂</strong></div><div class="detail-list-row"><span>RTP</span><strong :class="selectedPlayer.rtp < 100 ? 'positive' : 'negative'">{{ selectedPlayer.rtp }}%</strong></div><div class="detail-list-row"><span>最近遊戲時間</span><strong>2026-08-31 10:18:42</strong></div></div>
          <div v-else-if="playerDetailTab === 'transfer'" class="detail-list"><div class="detail-list-row"><span>目前代理線</span><strong>{{ displayAgentPath(selectedPlayer.path) }}</strong></div><div class="detail-list-row"><span>原代理線</span><strong>平台 > agent_taipei > north_team</strong></div><div class="detail-list-row"><span>最近轉線</span><strong>2026-08-01 00:00:00</strong></div><div class="detail-list-row"><span>轉線生效時間</span><strong>2026-08-01 00:00:00</strong></div><div class="detail-list-row"><span>執行人</span><strong>operator_demo</strong></div><p class="modal-help">玩家轉線由運營商操作；歷史訂單不回溯重算。</p></div>
          <div v-else-if="playerDetailTab === 'invite'" class="detail-list"><div class="detail-list-row"><span>註冊使用推廣碼</span><strong class="code-text">{{ selectedPlayer.referralCode || '未記錄' }}</strong></div><div class="detail-list-row"><span>自身邀請碼</span><strong class="code-text">{{ selectedPlayer.inviteCode }}</strong></div><div class="detail-list-row"><span>邀請總人數</span><strong>3 人</strong></div><div class="detail-list-row"><span>有效邀請</span><strong>2 人</strong></div><div class="detail-list-row"><span>待生效邀請</span><strong>1 人</strong></div><div class="detail-list-row"><span>最近邀請時間</span><strong>2026-08-27 18:08:42</strong></div></div>
          <div v-else-if="playerDetailTab === 'agent'" class="agent-detail-grid"><div class="full"><span>完整樹狀路徑</span><strong>{{ displayAgentPath(selectedPlayer.path) }}</strong></div><div><span>玩家層級</span><strong>{{ selectedPlayer.agentLevel }}</strong></div><div><span>所屬幣別</span><strong>{{ selectedPlayer.currency }}</strong></div><div><span>直屬代理</span><strong>north_team</strong></div><div><span>歸屬方式</span><strong>玩家推廣碼</strong></div><div><span>資料可見範圍</span><strong>所屬代理線</strong></div><div class="full"><span>代理後台權限</span><p class="modal-help">代理可以查看所屬下級玩家，但不能操作停用、編輯或轉線；玩家狀態與資金操作由運營商後台處理。</p></div></div>
        </template>
        <template #footer><NSpace justify="end"><NButton @click="showPlayerDetail = false">關閉</NButton></NSpace></template>
      </NModal>
      <NModal v-model:show="showPlayerEdit" preset="card" title="編輯玩家資料" class="modal-card"><p class="modal-intro">僅運營商可以編輯玩家資料；代理角色只能查看。</p><NForm label-placement="top"><NFormItem label="顯示名稱"><NInput v-model:value="playerDisplayNameDraft" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showPlayerEdit = false">取消</NButton><NButton type="primary" @click="savePlayerEdit">儲存</NButton></NSpace></template></NModal>
      <NModal v-model:show="showPlayerStatus" preset="card" title="玩家狀態管理" class="modal-card"><p class="modal-intro">僅運營商可以修改玩家帳號狀態。</p><NForm label-placement="top"><NFormItem label="帳號狀態"><NSelect v-model:value="playerStatusDraft" :options="playerStatusOptions" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showPlayerStatus = false">取消</NButton><NButton type="primary" @click="savePlayerStatus">儲存</NButton></NSpace></template></NModal>
      <NModal v-model:show="showPlayerTransfer" preset="card" title="玩家轉線" class="modal-card"><p class="modal-intro">僅運營商可以操作轉線；轉線後新產生的資料歸入新代理線，歷史資料不回溯重算。</p><NForm label-placement="top"><NFormItem label="新代理線"><NSelect v-model:value="playerTransferTarget" :options="playerTransferOptions" placeholder="選擇啟用中的代理線" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showPlayerTransfer = false">取消</NButton><NButton type="primary" @click="savePlayerTransfer">建立轉線</NButton></NSpace></template></NModal>
      <NModal v-model:show="showCreateAgent" preset="card" :title="createTitle" class="modal-card"><p class="modal-intro">{{ currentRole === '運營商' ? '建立一級代理時指定幣別與傭金方案；建立後可立即登入。' : '建立下級代理後立即啟用；代理只能隸屬一條代理線，傭金規則請套用既有方案。' }}</p><NForm label-placement="top"><div class="form-two-col"><NFormItem label="代理帳號"><NInput v-model:value="newAgentAccount" placeholder="輸入新代理帳號" /></NFormItem><NFormItem label="代理名稱"><NInput v-model:value="newAgentName" placeholder="輸入顯示名稱" /></NFormItem></div><div class="form-two-col"><NFormItem label="登入密碼"><NInput v-model:value="newAgentPassword" type="password" placeholder="設定初始登入密碼" /></NFormItem><NFormItem label="聯絡手機"><NInput v-model:value="newAgentPhone" placeholder="選填" /></NFormItem></div><NFormItem label="聯絡 Email"><NInput v-model:value="newAgentEmail" placeholder="選填" /></NFormItem><div class="form-two-col"><NFormItem label="套用傭金方案"><NSelect v-model:value="newAgentPlanId" :options="planOptions" /></NFormItem><NFormItem label="幣別"><NSelect v-model:value="newAgentCurrency" :options="[{label:'TWD 新台幣',value:'TWD'},{label:'USD 美元',value:'USD'},{label:'JPY 日圓',value:'JPY'}]" /></NFormItem></div></NForm><template #footer><NSpace justify="end"><NButton @click="showCreateAgent = false">取消</NButton><NButton type="primary" @click="createAgent">建立並啟用</NButton></NSpace></template></NModal>
              <NModal v-model:show="showPlanEditor" preset="card" :title="editingPlan ? '編輯傭金方案' : '新增傭金方案'" class="modal-card"><p class="modal-intro">傭金方案不使用推廣碼；玩家推廣碼由代理固定持有，方案依指定模式計算傭金。</p><NForm label-placement="top"><NFormItem label="方案名稱"><NInput v-model:value="planName" placeholder="例如：台灣有效投注標準方案" /></NFormItem><NFormItem label="建立代理"><NSelect v-model:value="planParentAccount" :options="planParentOptions" /></NFormItem><div class="form-two-col"><NFormItem label="傭金模式"><NSelect v-model:value="planModel" disabled :options="[{label: `${lineModel}（同一代理線固定模式）`, value: lineModel}]" /></NFormItem><NFormItem label="結算週期"><NSelect v-model:value="planCycle" :options="[{label:'即時',value:'即時'},{label:'每日',value:'每日'},{label:'每週',value:'每週'},{label:'每月',value:'每月'}]" /></NFormItem></div><div v-if="planModel === '輸贏'" class="mode-config-card"><strong>輸贏設定</strong><p class="modal-help">（玩家總輸贏 − 行政成本）× 輸贏佔成比例；負數依設定沖銷或累計。</p><div class="form-two-col"><NFormItem label="輸贏佔成比例"><NInputNumber v-model:value="shareRate" :min="0" :max="100" style="width: 100%"><template #suffix>%</template></NInputNumber></NFormItem><NFormItem label="行政費率"><NInputNumber v-model:value="adminCostRate" :min="0" :max="100" style="width: 100%"><template #suffix>%</template></NInputNumber></NFormItem><NFormItem label="負數處理"><NSelect v-model:value="negativeMode" :options="[{label:'負數沖銷',value:'負數沖銷'},{label:'負數累計',value:'負數累計'}]" /></NFormItem><NFormItem v-if="negativeMode === '負數沖銷'" label="沖銷方式"><NSelect v-model:value="offsetType" :options="[{label:'全額沖銷',value:'全額沖銷'},{label:'週期固定沖銷額度',value:'週期固定沖銷額度'}]" /></NFormItem><NFormItem v-if="negativeMode === '負數沖銷' && offsetType === '週期固定沖銷額度'" label="每期沖銷上限"><NInputNumber v-model:value="offsetLimit" :min="0" style="width: 100%" /></NFormItem></div></div><div v-else class="mode-config-card"><strong>{{ planModel === '儲值' ? '儲值設定' : '有效投注額設定' }}</strong><p class="modal-help">{{ planModel === '儲值' ? '成功儲值金額 × 傭金比例。' : '有效投注總額 × 傭金比例。' }}固定使用單一比例計算。</p><div class="form-two-col"><NFormItem :label="planModel === '儲值' ? '成功儲值比例' : '有效投注額比例'"><NInputNumber v-model:value="rebateRate" :min="0" :max="100" style="width: 100%"><template #suffix>%</template></NInputNumber></NFormItem></div></div><NFormItem label="默認可分配點數"><NInputNumber v-model:value="planRate" :min="0" :max="lineMaxRate" :step="0.5" style="width: 100%"><template #suffix>%</template></NInputNumber><p class="modal-help">本代理線最多可設定 {{ lineMaxRate }}%；不得超過上級提供的額度。</p></NFormItem><NFormItem label="方案說明"><NInput v-model:value="planDescription" type="textarea" :rows="3" placeholder="說明此方案的計算方式與下級點數規則" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showPlanEditor = false">取消</NButton><NButton type="primary" @click="savePlan">儲存方案</NButton></NSpace></template></NModal>
      <NModal v-model:show="showWithdrawal" preset="card" title="申請提領傭金" class="modal-card"><p class="modal-intro">申請將進入平台審核；目前可提領餘額 TWD 28,460。</p><NForm label-placement="top"><NFormItem label="提領金額"><NInputNumber v-model:value="withdrawalAmount" :min="1000" :max="28460" style="width: 100%"><template #prefix>TWD</template></NInputNumber></NFormItem><NFormItem label="收款帳戶"><NSelect value="bank-001" :options="[{label:'台新銀行 · ****1234',value:'bank-001'}]" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showWithdrawal = false">取消</NButton><NButton type="primary" @click="submitWithdrawal">送出申請</NButton></NSpace></template></NModal>
      <NModal v-model:show="showBankCard" preset="card" title="管理傭金收款銀行卡" class="modal-card"><p class="modal-intro">銀行卡資料會經過驗證，僅可用於傭金提領。</p><NForm label-placement="top"><NFormItem label="銀行名稱"><NInput v-model:value="bankName" placeholder="輸入銀行名稱" /></NFormItem><NFormItem label="帳號"><NInput v-model:value="bankAccount" placeholder="輸入收款帳號" /></NFormItem><NFormItem label="戶名"><NInput v-model:value="bankHolder" placeholder="輸入戶名" /></NFormItem></NForm><template #footer><NSpace justify="end"><NButton @click="showBankCard = false">取消</NButton><NButton type="primary" @click="saveBankCard">儲存並送驗證</NButton></NSpace></template></NModal>
    </div>
  </NConfigProvider>
</template>
