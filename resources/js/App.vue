<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue';

const mainTabs = [
    { key: 'teams', label: 'Teams', badge: 29 },
    { key: 'bills', label: 'Invoices' },
    { key: 'servers', label: 'Servers' },
    { key: 'reports', label: 'Reports' },
];

const statusChips = [
    { key: 'all', label: 'All', dot: 'bg-slate-400', tint: 'bg-white' },
    { key: 'unpaid', label: 'Unpaid', dot: 'bg-rose-500', tint: 'bg-rose-50 text-rose-700' },
    { key: 'paid', label: 'Paid', dot: 'bg-emerald-500', tint: 'bg-emerald-50 text-emerald-700' },
    { key: 'suspended', label: 'Suspended', dot: 'bg-orange-500', tint: 'bg-orange-50 text-orange-700' },
    { key: 'terminated', label: 'Terminated', dot: 'bg-slate-600', tint: 'bg-slate-100 text-slate-700' },
    { key: 'grace', label: 'Grace Period', dot: 'bg-amber-500', tint: 'bg-amber-50 text-amber-700' },
    { key: 'removed', label: 'Deferred Next Month', dot: 'bg-slate-400', tint: 'bg-slate-100 text-slate-600' },
];

const summaryCards = [
    {
        title: 'Teams with Unpaid Bills',
        value: '12',
        subvalue: '/ 48 teams',
        helper: 'Teams that need attention',
        trend: '-2',
        trendTone: 'text-rose-600 bg-rose-50',
        iconTone: 'from-violet-500 to-violet-400',
        icon: '⚠',
    },
    {
        title: 'Total Unpaid Amount',
        value: '$572.00',
        helper: 'Across 29 invoices',
        trend: '-18%',
        trendTone: 'text-rose-600 bg-rose-50',
        iconTone: 'from-blue-500 to-sky-400',
        icon: '$',
    },
    {
        title: 'Invoices This Month',
        value: '86',
        helper: 'All generated invoices',
        trend: '12%',
        trendTone: 'text-emerald-600 bg-emerald-50',
        iconTone: 'from-emerald-500 to-emerald-300',
        icon: '✓',
    },
    {
        title: 'Deferred Next Month',
        value: '6',
        helper: 'Deferred until next billing cycle',
        iconTone: 'from-orange-500 to-orange-300',
        icon: '↺',
    },
];

const serviceTypeOptions = ['All Services', 'Managed', 'Self Managed', 'Addons', 'Others'];
const rowsPerPageOptions = [10, 20, 50, 100];
const FILTER_STORAGE_KEY = 'xcloud-billing-dashboard-filters';
const FILTER_STORAGE_TTL_MS = 24 * 60 * 60 * 1000;

const teams = [
    {
        id: '#129496',
        name: 'xC Infinity',
        email: 'team@xcinfinity.com',
        invoices: 3,
        status: 'Unpaid',
        statusKey: 'unpaid',
        agent: 'Dipu',
        serviceType: 'Managed',
        totalUnpaid: '$572.00',
        unpaidSince: 'Aug 12, 2025',
        overdueDays: 18,
        note: '-',
        initials: 'XC',
        managedBy: 'Dipu',
        lastInvoice: 'XC-INV-20260818-IVBZ0PFZEGFJ',
        linkedServers: 2,
        invoicesList: [
            { id: 'XC-INV-20260818-IVBZ0PFZEGFJ', period: 'Aug 2025', created: 'Aug 1, 2025', due: 'Aug 12, 2025', overdue: 18, amount: '$220.00', status: 'Unpaid', statusKey: 'unpaid', service: 'Primary Cloud Server', serviceType: 'Managed' },
            { id: 'XC-INV-20260716-L9QX2NBR7TMA', period: 'Jul 2025', created: 'Jul 1, 2025', due: 'Jul 12, 2025', overdue: 49, amount: '$180.00', status: 'Unpaid', statusKey: 'unpaid', service: 'Backup Storage Node', serviceType: 'Managed' },
            { id: 'XC-INV-20260611-Z3KDP8WY4HNC', period: 'Jun 2025', created: 'Jun 1, 2025', due: 'Jun 12, 2025', overdue: 79, amount: '$172.00', status: 'Unpaid', statusKey: 'unpaid', service: 'Managed DNS Cluster', serviceType: 'Managed' },
        ],
        notes: [
            { author: 'Dipu', role: 'Admin', time: 'Aug 18, 2025 · 10:24 AM', message: 'Customer requested a few more days to complete the payment.', type: 'agent', initials: 'D' },
            { author: 'Mahbub', role: 'Billing Agent', time: 'Aug 19, 2025 · 2:40 PM', message: 'Reminder email sent to the account owner.', type: 'agent', initials: 'M' },
            { author: 'System', role: 'Automation', time: 'Aug 19, 2025 · 2:41 PM', message: 'Status changed from Unpaid to Grace Period.', type: 'system', statusLabel: 'Status changed' },
            { author: 'Dipu', role: 'Admin', time: 'Aug 20, 2025 · 9:05 AM', message: 'Customer did not respond. Follow up again on Aug 30.', type: 'agent', initials: 'D', edited: true },
        ],
        billingMonth: 'Aug 2025',
    },
    {
        id: '#129405',
        name: 'Galaxy Bay',
        email: 'admin@galaxybay.com',
        invoices: 1,
        status: 'Suspended',
        statusKey: 'suspended',
        agent: 'Mahbub',
        serviceType: 'Self Managed',
        totalUnpaid: '$96.00',
        unpaidSince: 'Jul 22, 2025',
        overdueDays: 39,
        note: 'Server inactive',
        initials: 'GB',
        managedBy: 'Mahbub',
        lastInvoice: 'XC-INV-20260722-A9MT6QPL2XKD',
        linkedServers: 1,
        invoicesList: [
            { id: 'XC-INV-20260722-A9MT6QPL2XKD', period: 'Jul 2025', created: 'Jul 1, 2025', due: 'Jul 22, 2025', overdue: 39, amount: '$96.00', status: 'Suspended', statusKey: 'suspended', service: 'Shared VM Node', serviceType: 'Self Managed' },
        ],
        notes: [
            { author: 'System', role: 'Automation', time: 'Aug 14, 2025 · 8:20 AM', message: 'Status changed from Grace Period to Suspended.', type: 'system', statusLabel: 'Suspended' },
            { author: 'Mahbub', role: 'Billing Agent', time: 'Aug 14, 2025 · 9:14 AM', message: 'Server inactive', type: 'agent', initials: 'M' },
        ],
        billingMonth: 'Jul 2025',
    },
    {
        id: '#129398',
        name: 'Creative Studio',
        email: 'hello@creativestudio.io',
        invoices: 2,
        status: 'Grace Period',
        statusKey: 'grace',
        agent: 'Nirob',
        serviceType: 'Addons',
        totalUnpaid: '$124.00',
        unpaidSince: 'Aug 1, 2025',
        overdueDays: 29,
        note: 'Grace period until Aug 31, 2025',
        initials: 'CS',
        managedBy: 'Nirob',
        lastInvoice: 'XC-INV-20260801-C7PV4MXL8QRT',
        linkedServers: 3,
        invoicesList: [
            { id: 'XC-INV-20260801-C7PV4MXL8QRT', period: 'Aug 2025', created: 'Aug 1, 2025', due: 'Aug 1, 2025', overdue: 29, amount: '$64.00', status: 'Grace Period', statusKey: 'grace', service: 'Media Rendering Node', serviceType: 'Addons' },
            { id: 'XC-INV-20260701-N2WR5DZJ6KLF', period: 'Jul 2025', created: 'Jul 1, 2025', due: 'Jul 1, 2025', overdue: 60, amount: '$60.00', status: 'Grace Period', statusKey: 'grace', service: 'Archive Storage', serviceType: 'Addons' },
        ],
        notes: [{ author: 'Nirob', role: 'Billing Agent', time: 'Aug 10, 2025 · 11:12 AM', message: 'Grace period until Aug 31, 2025', type: 'agent', initials: 'N' }],
        billingMonth: 'Aug 2025',
    },
    {
        id: '#129372',
        name: 'Pixel Works',
        email: 'support@pixelworks.co',
        invoices: 1,
        status: 'Paid',
        statusKey: 'paid',
        agent: 'Sadat',
        serviceType: 'Managed',
        totalUnpaid: '$0.00',
        unpaidSince: 'Aug 10, 2025',
        overdueDays: 0,
        note: '-',
        initials: 'PW',
        managedBy: 'Sadat',
        lastInvoice: 'XC-INV-20260803-R4YM8TBQ1VHS',
        linkedServers: 2,
        invoicesList: [
            { id: 'XC-INV-20260803-R4YM8TBQ1VHS', period: 'Aug 2025', created: 'Aug 1, 2025', due: 'Aug 3, 2025', overdue: 0, amount: '$0.00', status: 'Paid', statusKey: 'paid', service: 'Core API Cluster', serviceType: 'Managed' },
        ],
        notes: [{ author: 'System', role: 'Automation', time: 'Aug 3, 2025 · 1:10 PM', message: 'Invoice paid successfully.', type: 'system', statusLabel: 'Paid' }],
        billingMonth: 'Aug 2025',
    },
    {
        id: '#129355',
        name: 'Alpha Traders',
        email: 'contact@alphatraders.com',
        invoices: 4,
        status: 'Terminated',
        statusKey: 'terminated',
        agent: 'Nayeem',
        serviceType: 'Others',
        totalUnpaid: '$0.00',
        unpaidSince: 'Jul 15, 2025',
        overdueDays: 0,
        note: 'Account terminated',
        initials: 'AT',
        managedBy: 'Nayeem',
        lastInvoice: 'XC-INV-20260519-H6NZ3WQP9YTR',
        linkedServers: 0,
        invoicesList: [
            { id: 'XC-INV-20260519-H6NZ3WQP9YTR', period: 'May 2025', created: 'May 1, 2025', due: 'May 19, 2025', overdue: 0, amount: '$0.00', status: 'Terminated', statusKey: 'terminated', service: 'Former Dedicated Node', serviceType: 'Others' },
        ],
        notes: [{ author: 'System', role: 'Automation', time: 'May 20, 2025 · 5:44 PM', message: 'Account terminated', type: 'system', statusLabel: 'Terminated' }],
        billingMonth: 'May 2025',
    },
    {
        id: '#129330',
        name: 'Demo Team',
        email: 'demo@example.com',
        invoices: 2,
        status: 'Deferred Next Month',
        statusKey: 'removed',
        agent: 'Jan-Jan',
        serviceType: 'Addons',
        totalUnpaid: '$206.00',
        unpaidSince: 'Aug 5, 2025',
        overdueDays: 25,
        note: 'Deferred next month',
        initials: 'DT',
        managedBy: 'Jan-Jan',
        lastInvoice: 'XC-INV-20260805-P8LX4QMW2KVD',
        linkedServers: 1,
        invoicesList: [
            { id: 'XC-INV-20260805-P8LX4QMW2KVD', period: 'Aug 2025', created: 'Aug 1, 2025', due: 'Aug 5, 2025', overdue: 25, amount: '$106.00', status: 'Deferred Next Month', statusKey: 'removed', service: 'Trial App Server', serviceType: 'Addons' },
            { id: 'XC-INV-20260705-U3VB7NQK5MTE', period: 'Jul 2025', created: 'Jul 1, 2025', due: 'Jul 5, 2025', overdue: 56, amount: '$100.00', status: 'Deferred Next Month', statusKey: 'removed', service: 'Demo Storage Node', serviceType: 'Addons' },
        ],
        notes: [{ author: 'Dipu', role: 'Admin', time: 'Aug 11, 2025 · 10:55 AM', message: 'Account deferred until next billing cycle after support escalation.', type: 'agent', initials: 'D' }],
        billingMonth: 'Aug 2025',
    },
    {
        id: '#129312',
        name: 'Old Project Team',
        email: 'old@sample.com',
        invoices: 1,
        status: 'Deferred Next Month',
        statusKey: 'removed',
        agent: 'System',
        serviceType: 'Others',
        totalUnpaid: '$0.00',
        unpaidSince: 'Jun 30, 2025',
        overdueDays: 0,
        note: 'Deferred next month',
        initials: 'OP',
        managedBy: 'System',
        lastInvoice: 'XC-INV-20260630-J5RX9PLD4WQS',
        linkedServers: 0,
        invoicesList: [
            { id: 'XC-INV-20260630-J5RX9PLD4WQS', period: 'Jun 2025', created: 'Jun 1, 2025', due: 'Jun 30, 2025', overdue: 0, amount: '$0.00', status: 'Deferred Next Month', statusKey: 'removed', service: 'Legacy Sandbox', serviceType: 'Others' },
        ],
        notes: [{ author: 'System', role: 'Automation', time: 'Jul 30, 2025 · 7:32 PM', message: 'Deferred next month', type: 'system', statusLabel: 'Deferred Next Month' }],
        billingMonth: 'Jun 2025',
    },
];

const servers = [
    {
        id: 'SRV-88421',
        novaLink: 'Nova',
        vultrLink: 'Vultr',
        monthsUnpaid: 3,
        currentPlan: 'Elite',
        upgradeHistory: 'Personal > Business > Elite',
        currentStatus: 'Active',
        serviceType: 'Managed',
        note: 'Awaiting owner response',
        billingStatus: 'Suspended',
        billingStatusKey: 'suspended',
        team: 'xC Infinity',
        owner: 'Dipu',
        region: 'Singapore',
        ip: '149.28.122.44',
        monthlyCost: '$220.00',
        unpaidAmount: '$572.00',
        lastInvoice: '#INV-129496-03',
        overdueSince: 'Aug 12, 2025',
        billingMonth: 'Aug 2025',
        services: 'Web, DB, Cache',
        timeline: [
            { author: 'System', role: 'Automation', time: 'Aug 19, 2025 · 2:41 PM', message: 'Billing state moved to Suspended after third unpaid cycle.', type: 'system', statusLabel: 'Suspended' },
            { author: 'Dipu', role: 'Admin', time: 'Aug 20, 2025 · 9:12 AM', message: 'Waiting on customer before manual suspension inside provider panel.', type: 'agent', initials: 'D' },
        ],
    },
    {
        id: 'SRV-77214',
        novaLink: 'Nova',
        vultrLink: 'Vultr',
        monthsUnpaid: 2,
        currentPlan: 'Pro',
        upgradeHistory: 'Starter > Pro',
        currentStatus: 'Degraded',
        serviceType: 'Addons',
        note: 'Queue for suspension',
        billingStatus: 'Grace Period',
        billingStatusKey: 'grace',
        team: 'Creative Studio',
        owner: 'Nirob',
        region: 'Frankfurt',
        ip: '45.76.89.12',
        monthlyCost: '$64.00',
        unpaidAmount: '$124.00',
        lastInvoice: '#INV-129398-02',
        overdueSince: 'Aug 1, 2025',
        billingMonth: 'Aug 2025',
        services: 'Media Render Node',
        timeline: [
            { author: 'Nirob', role: 'Billing Agent', time: 'Aug 10, 2025 · 11:12 AM', message: 'Grace period extended while studio clears outstanding dues.', type: 'agent', initials: 'N' },
        ],
    },
    {
        id: 'SRV-66103',
        novaLink: 'Nova',
        vultrLink: 'Vultr',
        monthsUnpaid: 1,
        currentPlan: 'Scale',
        upgradeHistory: 'Starter > Scale',
        currentStatus: 'Warning',
        serviceType: 'Self Managed',
        note: 'Single unpaid month',
        billingStatus: 'Unpaid',
        billingStatusKey: 'unpaid',
        team: 'Galaxy Bay',
        owner: 'Mahbub',
        region: 'Tokyo',
        ip: '66.135.21.87',
        monthlyCost: '$96.00',
        unpaidAmount: '$96.00',
        lastInvoice: '#INV-129405-01',
        overdueSince: 'Jul 22, 2025',
        billingMonth: 'Jul 2025',
        services: 'Shared VM Node',
        timeline: [
            { author: 'Mahbub', role: 'Billing Agent', time: 'Aug 14, 2025 · 9:14 AM', message: 'Mark for suspend if unpaid by next cycle.', type: 'agent', initials: 'M' },
        ],
    },
];

const mainTab = ref('teams');
const drawerTab = ref('overview');
const selectedStatus = ref('all');
const selectedTeamId = ref(teams[0].id);
const drawerOpen = ref(true);
const teamSearch = ref('');
const openMenuId = ref(null);
const invoiceMenuId = ref(null);
const serverSearch = ref('');
const selectedServerId = ref(servers[0].id);
const serverDrawerOpen = ref(true);
const serverMenuId = ref(null);
const serverDrawerTab = ref('overview');
const selectedServerStatus = ref('all');
const selectedTeamFromDate = ref('');
const selectedTeamToDate = ref('');
const selectedInvoiceStatus = ref('all');
const selectedInvoiceFromDate = ref('');
const selectedInvoiceToDate = ref('');
const selectedServerFromDate = ref('');
const selectedServerToDate = ref('');
const selectedServiceType = ref('All Services');
const selectedInvoiceServiceType = ref('All Services');
const teamsPage = ref(1);
const teamsRowsPerPage = ref(10);
const invoicesPage = ref(1);
const invoicesRowsPerPage = ref(10);
const serversPage = ref(1);
const serversRowsPerPage = ref(10);
const teamAmountMin = ref('');
const invoiceAmountMin = ref('');
const serverAmountMin = ref('');
const ignoredItems = reactive({
    team: {},
    invoice: {},
    server: {},
});
const activeMenu = reactive({
    type: '',
    id: '',
    top: 0,
    left: 0,
    width: 0,
    openUp: false,
});
const modalState = reactive({
    type: '',
    title: '',
    targetId: '',
    targetLabel: '',
    filterScope: 'teams',
    duration: '7 days',
    emailUser: true,
    notice: '',
});
const copiedText = ref('');
let copyResetTimeout;
const persistedFilterRefs = {
    mainTab,
    selectedStatus,
    selectedServerStatus,
    selectedInvoiceStatus,
    teamSearch,
    serverSearch,
    selectedTeamFromDate,
    selectedTeamToDate,
    selectedInvoiceFromDate,
    selectedInvoiceToDate,
    selectedServerFromDate,
    selectedServerToDate,
    selectedServiceType,
    selectedInvoiceServiceType,
    teamAmountMin,
    invoiceAmountMin,
    serverAmountMin,
    teamsRowsPerPage,
    invoicesRowsPerPage,
    serversRowsPerPage,
};

const statusMap = Object.fromEntries(statusChips.map((chip) => [chip.key, chip]));

function monthLabelToValue(label) {
    const months = {
        Jan: '01',
        Feb: '02',
        Mar: '03',
        Apr: '04',
        May: '05',
        Jun: '06',
        Jul: '07',
        Aug: '08',
        Sep: '09',
        Oct: '10',
        Nov: '11',
        Dec: '12',
    };

    if (!label) return '';
    const [month, year] = label.split(' ');
    return month && year && months[month] ? `${year}-${months[month]}` : '';
}

function formatMonthYear(dateLabel) {
    const date = new Date(dateLabel);
    if (Number.isNaN(date.getTime())) return dateLabel;
    return date.toLocaleString('en-US', { month: 'short', year: 'numeric' });
}

function monthsOverdueLabel(dateLabel) {
    const date = new Date(dateLabel);
    if (Number.isNaN(date.getTime())) return 'No overdue balance';
    const today = new Date('2026-08-20');
    if (date > today) return 'No overdue balance';
    let months = (today.getFullYear() - date.getFullYear()) * 12 + (today.getMonth() - date.getMonth());
    if (today.getDate() >= date.getDate()) {
        months += 1;
    }
    months = Math.max(0, months);
    if (months === 0) return 'No overdue balance';
    return `${months} month${months === 1 ? '' : 's'} overdue`;
}

function dateFallsInRange(dateLabel, fromDate, toDate) {
    const normalized = monthLabelToValue(dateLabel);
    if (!normalized) return true;

    if (fromDate) {
        const fromMonth = fromDate.slice(0, 7);
        if (normalized < fromMonth) return false;
    }

    if (toDate) {
        const toMonth = toDate.slice(0, 7);
        if (normalized > toMonth) return false;
    }

    return true;
}

function parseCurrency(value) {
    return Number.parseFloat(String(value).replace(/[^0-9.]/g, '')) || 0;
}

const filteredTeams = computed(() =>
    teams.filter((team) => {
        const matchStatus = selectedStatus.value === 'all' || team.statusKey === selectedStatus.value;
        const searchable = [team.id, team.name, team.email, team.agent, team.lastInvoice].join(' ').toLowerCase();
        const matchRange = team.invoicesList.some((invoice) => dateFallsInRange(invoice.period, selectedTeamFromDate.value, selectedTeamToDate.value));
        const matchAmount = teamAmountMin.value === '' || parseCurrency(team.totalUnpaid) >= Number(teamAmountMin.value);
        return matchStatus && matchRange && matchAmount && searchable.includes(teamSearch.value.toLowerCase()) && !isIgnored('team', team.id);
    }),
);

const selectedTeam = computed(() => filteredTeams.value.find((team) => team.id === selectedTeamId.value) || filteredTeams.value[0] || teams[0]);
const teamsTotalPages = computed(() => Math.max(1, Math.ceil(filteredTeams.value.length / teamsRowsPerPage.value)));
const paginatedTeams = computed(() => {
    const start = (teamsPage.value - 1) * teamsRowsPerPage.value;
    return filteredTeams.value.slice(start, start + teamsRowsPerPage.value);
});
const teamsRangeStart = computed(() => (filteredTeams.value.length ? (teamsPage.value - 1) * teamsRowsPerPage.value + 1 : 0));
const teamsRangeEnd = computed(() => Math.min(teamsPage.value * teamsRowsPerPage.value, filteredTeams.value.length));
const teamPageNumbers = computed(() => Array.from({ length: teamsTotalPages.value }, (_, index) => index + 1));

const outstandingInvoices = computed(() =>
    (selectedTeam.value?.invoicesList || []).filter((invoice) => ['unpaid', 'suspended', 'grace'].includes(invoice.statusKey)),
);

const allBills = computed(() =>
    teams
        .flatMap((team) => team.invoicesList.map((invoice) => ({ ...invoice, teamId: team.id, teamName: team.name, teamEmail: team.email, serviceType: invoice.serviceType ?? team.serviceType })))
        .filter((invoice) => selectedInvoiceStatus.value === 'all' || invoice.statusKey === selectedInvoiceStatus.value)
        .filter((invoice) => selectedInvoiceServiceType.value === 'All Services' || invoice.serviceType === selectedInvoiceServiceType.value)
        .filter((invoice) => dateFallsInRange(invoice.period, selectedInvoiceFromDate.value, selectedInvoiceToDate.value))
        .filter((invoice) => {
            const amount = parseCurrency(invoice.amount);
            const min = invoiceAmountMin.value === '' ? null : Number(invoiceAmountMin.value);
            if (min !== null && amount < min) return false;
            return true;
        })
        .filter((invoice) => !isIgnored('invoice', invoice.id)),
);
const invoicesTotalPages = computed(() => Math.max(1, Math.ceil(allBills.value.length / invoicesRowsPerPage.value)));
const paginatedBills = computed(() => {
    const start = (invoicesPage.value - 1) * invoicesRowsPerPage.value;
    return allBills.value.slice(start, start + invoicesRowsPerPage.value);
});
const invoicesRangeStart = computed(() => (allBills.value.length ? (invoicesPage.value - 1) * invoicesRowsPerPage.value + 1 : 0));
const invoicesRangeEnd = computed(() => Math.min(invoicesPage.value * invoicesRowsPerPage.value, allBills.value.length));
const invoicePageNumbers = computed(() => Array.from({ length: invoicesTotalPages.value }, (_, index) => index + 1));

const filteredServers = computed(() =>
    servers.filter((server) => {
        const matchStatus = selectedServerStatus.value === 'all' || server.billingStatusKey === selectedServerStatus.value;
        const searchable = [
            server.id,
            server.team,
            server.owner,
            server.currentPlan,
            server.billingStatus,
            server.note,
            server.region,
        ]
            .join(' ')
            .toLowerCase();
        const matchRange = dateFallsInRange(server.billingMonth, selectedServerFromDate.value, selectedServerToDate.value);
        const matchAmount = serverAmountMin.value === '' || parseCurrency(server.unpaidAmount) >= Number(serverAmountMin.value);
        return matchStatus && matchRange && matchAmount && searchable.includes(serverSearch.value.toLowerCase()) && !isIgnored('server', server.id);
    }),
);
const serversTotalPages = computed(() => Math.max(1, Math.ceil(filteredServers.value.length / serversRowsPerPage.value)));
const paginatedServers = computed(() => {
    const start = (serversPage.value - 1) * serversRowsPerPage.value;
    return filteredServers.value.slice(start, start + serversRowsPerPage.value);
});
const serversRangeStart = computed(() => (filteredServers.value.length ? (serversPage.value - 1) * serversRowsPerPage.value + 1 : 0));
const serversRangeEnd = computed(() => Math.min(serversPage.value * serversRowsPerPage.value, filteredServers.value.length));
const serverPageNumbers = computed(() => Array.from({ length: serversTotalPages.value }, (_, index) => index + 1));

const selectedServer = computed(() => filteredServers.value.find((server) => server.id === selectedServerId.value) || filteredServers.value[0] || servers[0]);

const selectedStatusLabel = computed(() => statusMap[selectedStatus.value]?.label || 'All Statuses');
const selectedStatusMeta = computed(() => statusMap[selectedStatus.value] || statusMap.all);
const selectedInvoiceStatusMeta = computed(() => statusMap[selectedInvoiceStatus.value] || statusMap.all);
const selectedServerStatusMeta = computed(() => statusMap[selectedServerStatus.value] || statusMap.all);

function badgeClass(statusKey) {
    return {
        unpaid: 'bg-rose-50 text-rose-700 ring-1 ring-rose-100',
        paid: 'bg-emerald-50 text-emerald-700 ring-1 ring-emerald-100',
        suspended: 'bg-orange-50 text-orange-700 ring-1 ring-orange-100',
        terminated: 'bg-slate-100 text-slate-700 ring-1 ring-slate-200',
        grace: 'bg-amber-50 text-amber-700 ring-1 ring-amber-100',
        removed: 'bg-slate-100 text-slate-600 ring-1 ring-slate-200',
    }[statusKey];
}

function statusFilterClass(statusKey) {
    return {
        all: 'bg-white text-slate-700 ring-1 ring-slate-200',
        unpaid: 'bg-rose-50 text-rose-700 ring-1 ring-rose-100',
        paid: 'bg-emerald-50 text-emerald-700 ring-1 ring-emerald-100',
        suspended: 'bg-orange-50 text-orange-700 ring-1 ring-orange-100',
        terminated: 'bg-slate-100 text-slate-700 ring-1 ring-slate-200',
        grace: 'bg-amber-50 text-amber-700 ring-1 ring-amber-100',
        removed: 'bg-slate-100 text-slate-600 ring-1 ring-slate-200',
    }[statusKey];
}

function selectTeam(teamId) {
    selectedTeamId.value = teamId;
    drawerOpen.value = true;
}

function setStatusFilter(key) {
    selectedStatus.value = key;
    openMenuId.value = null;
}

function setActiveMenuPosition(button, width) {
    const rect = button.getBoundingClientRect();
    const estimatedHeight = 260;
    const openUp = window.innerHeight - rect.bottom < estimatedHeight;
    activeMenu.top = openUp ? rect.top - 8 : rect.bottom + 8;
    activeMenu.left = Math.max(12, rect.right - width);
    activeMenu.width = width;
    activeMenu.openUp = openUp;
}

function toggleMenu(event, teamId) {
    if (openMenuId.value === teamId) {
        closeAllMenus();
        return;
    }
    openMenuId.value = teamId;
    invoiceMenuId.value = null;
    serverMenuId.value = null;
    activeMenu.type = 'team';
    activeMenu.id = teamId;
    setActiveMenuPosition(event.currentTarget, 192);
}

function closeAllMenus() {
    openMenuId.value = null;
    invoiceMenuId.value = null;
    serverMenuId.value = null;
    activeMenu.type = '';
    activeMenu.id = '';
}

function closeDrawer() {
    drawerOpen.value = false;
}

function resetFilters() {
    selectedStatus.value = 'all';
    teamSearch.value = '';
    selectedTeamFromDate.value = '';
    selectedTeamToDate.value = '';
    teamAmountMin.value = '';
    teamsPage.value = 1;
}

function confirmAction(label, teamName) {
    window.alert(`${label} for ${teamName}`);
    openMenuId.value = null;
}

function selectServer(serverId) {
    selectedServerId.value = serverId;
    serverDrawerOpen.value = true;
}

function toggleServerMenu(event, serverId) {
    if (serverMenuId.value === serverId) {
        closeAllMenus();
        return;
    }
    serverMenuId.value = serverId;
    openMenuId.value = null;
    invoiceMenuId.value = null;
    activeMenu.type = 'server';
    activeMenu.id = serverId;
    setActiveMenuPosition(event.currentTarget, 160);
}

function setServerStatusFilter(key) {
    selectedServerStatus.value = key;
    closeAllMenus();
    serversPage.value = 1;
}

function suspendServer(serverId) {
    const server = servers.find((item) => item.id === serverId);
    if (!server) return;
    modalState.type = 'suspend-server';
    modalState.title = 'Suspend Server';
    modalState.targetId = server.id;
    modalState.targetLabel = server.id;
    modalState.notice = '';
    serverMenuId.value = null;
}

function toggleInvoiceMenu(event, invoiceId) {
    if (invoiceMenuId.value === invoiceId) {
        closeAllMenus();
        return;
    }
    invoiceMenuId.value = invoiceId;
    openMenuId.value = null;
    serverMenuId.value = null;
    activeMenu.type = 'invoice';
    activeMenu.id = invoiceId;
    setActiveMenuPosition(event.currentTarget, 176);
}

function activateTeam(teamId) {
    const team = teams.find((item) => item.id === teamId);
    if (!team) return;
    modalState.type = 'activate-team';
    modalState.title = 'Activate Team';
    modalState.targetId = team.id;
    modalState.targetLabel = team.name;
    modalState.duration = '7 days';
    modalState.emailUser = true;
    openMenuId.value = null;
}

function openMoreFilters(scope = 'teams') {
    modalState.type = 'more-filters';
    modalState.title = 'More Filters';
    modalState.targetId = '';
    modalState.targetLabel = 'Add more filters now, and keep this modal ready for future billing filters.';
    modalState.filterScope = scope;
}

function takePayment(invoiceId, forced = false) {
    const invoice = allBills.value.find((item) => item.id === invoiceId);
    if (!invoice) return;
    window.alert(`${forced ? 'Force Take Payment' : 'Take Payment'} for ${invoice.id}`);
    invoiceMenuId.value = null;
}

function terminateServer(serverId) {
    const server = servers.find((item) => item.id === serverId);
    if (!server) return;
    window.alert(`Terminate ${server.id}`);
    serverMenuId.value = null;
}

function activeMonthForType(type) {
    if (type === 'team') {
        return selectedTeamFromDate.value ? selectedTeamFromDate.value.slice(0, 7) : '2026-08';
    }
    if (type === 'invoice') {
        return selectedInvoiceFromDate.value ? selectedInvoiceFromDate.value.slice(0, 7) : '2026-08';
    }
    return selectedServerFromDate.value ? selectedServerFromDate.value.slice(0, 7) : '2026-08';
}

async function copyText(value) {
    if (!value) return;

    try {
        await navigator.clipboard.writeText(value);
        copiedText.value = value;
        window.clearTimeout(copyResetTimeout);
        copyResetTimeout = window.setTimeout(() => {
            copiedText.value = '';
        }, 1600);
    } catch {
        window.alert(`Copy this value:\n${value}`);
    }
}

function ignoreForOneMonth(type, id, label) {
    ignoredItems[type][id] = activeMonthForType(type);
    if (type === 'team') openMenuId.value = null;
    if (type === 'invoice') invoiceMenuId.value = null;
    if (type === 'server') serverMenuId.value = null;
    window.alert(`${label} hidden for ${ignoredItems[type][id]}. It will reappear next month.`);
}

function isIgnored(type, id) {
    const ignoredMonth = ignoredItems[type][id];
    if (!ignoredMonth) return false;
    return ignoredMonth === activeMonthForType(type);
}

function closeModal() {
    modalState.type = '';
    modalState.title = '';
    modalState.targetId = '';
    modalState.targetLabel = '';
    modalState.filterScope = 'teams';
    modalState.notice = '';
}

function submitModalAction() {
    if (modalState.type === 'suspend-server') {
        window.alert(`Suspend ${modalState.targetLabel} with notice: ${modalState.notice || 'No notice provided'}`);
    }
    if (modalState.type === 'activate-team') {
        window.alert(`Activate ${modalState.targetLabel} for ${modalState.duration}${modalState.emailUser ? ' and email the user' : ''}.`);
    }
    if (modalState.type === 'more-filters') {
        window.alert('Filters updated for this view.');
    }
    closeModal();
}

function setTeamsPage(page) {
    teamsPage.value = Math.min(Math.max(1, page), teamsTotalPages.value);
}

function setInvoicesPage(page) {
    invoicesPage.value = Math.min(Math.max(1, page), invoicesTotalPages.value);
}

function setServersPage(page) {
    serversPage.value = Math.min(Math.max(1, page), serversTotalPages.value);
}

watch([filteredTeams, teamsRowsPerPage], () => {
    if (teamsPage.value > teamsTotalPages.value) {
        teamsPage.value = teamsTotalPages.value;
    }
});

watch([allBills, invoicesRowsPerPage], () => {
    if (invoicesPage.value > invoicesTotalPages.value) {
        invoicesPage.value = invoicesTotalPages.value;
    }
});

watch([filteredServers, serversRowsPerPage], () => {
    if (serversPage.value > serversTotalPages.value) {
        serversPage.value = serversTotalPages.value;
    }
});

function handleDocumentClick(event) {
    if (!event.target.closest('[data-action-menu]')) {
        closeAllMenus();
    }
}

function saveFilterState() {
    const values = Object.fromEntries(
        Object.entries(persistedFilterRefs).map(([key, filterRef]) => [key, filterRef.value]),
    );

    localStorage.setItem(
        FILTER_STORAGE_KEY,
        JSON.stringify({
            expiresAt: Date.now() + FILTER_STORAGE_TTL_MS,
            values,
        }),
    );
}

function restoreFilterState() {
    const raw = localStorage.getItem(FILTER_STORAGE_KEY);
    if (!raw) return;

    try {
        const parsed = JSON.parse(raw);
        if (!parsed?.expiresAt || parsed.expiresAt < Date.now()) {
            localStorage.removeItem(FILTER_STORAGE_KEY);
            return;
        }

        Object.entries(persistedFilterRefs).forEach(([key, filterRef]) => {
            if (Object.prototype.hasOwnProperty.call(parsed.values ?? {}, key)) {
                const restoredValue = parsed.values[key];
                if (['teamsRowsPerPage', 'invoicesRowsPerPage', 'serversRowsPerPage'].includes(key)) {
                    const numericValue = Number(restoredValue);
                    filterRef.value = rowsPerPageOptions.includes(numericValue) ? numericValue : filterRef.value;
                    return;
                }
                filterRef.value = restoredValue ?? filterRef.value;
            }
        });
    } catch {
        localStorage.removeItem(FILTER_STORAGE_KEY);
    }
}

function menuPositionClass(index, total) {
    return index >= total - 2 ? 'bottom-9' : 'top-9';
}

onMounted(() => {
    restoreFilterState();
    document.addEventListener('click', handleDocumentClick);
    window.addEventListener('resize', closeAllMenus);
    window.addEventListener('scroll', closeAllMenus, true);
});

onBeforeUnmount(() => {
    document.removeEventListener('click', handleDocumentClick);
    window.removeEventListener('resize', closeAllMenus);
    window.removeEventListener('scroll', closeAllMenus, true);
    window.clearTimeout(copyResetTimeout);
});

watch(
    Object.values(persistedFilterRefs),
    () => {
        saveFilterState();
    },
);
</script>

<template>
    <div class="min-h-screen bg-[radial-gradient(circle_at_top_left,_rgba(255,255,255,0.96),_rgba(241,245,249,1)_55%)]">
        <header class="sticky top-0 z-30 border-b border-slate-200/80 bg-white/90 backdrop-blur">
            <div class="mx-auto h-13 max-w-[1600px] px-3"></div>
        </header>

        <main class="mx-auto max-w-[1600px] px-3 py-3">
            <section class="mb-3 flex flex-col gap-3 xl:flex-row xl:items-center xl:justify-between">
                <div>
                    <h1 class="text-[20px] font-semibold tracking-tight text-slate-950">Billing Dashboard</h1>
                    <p class="mt-0.5 text-[12px] text-slate-500">Overview of teams, invoices and payments</p>
                </div>
            </section>

            <nav class="mb-3 flex flex-wrap gap-1.5">
                <button
                    v-for="tab in mainTabs"
                    :key="tab.key"
                    @click="mainTab = tab.key"
                    :class="mainTab === tab.key ? 'bg-white text-blue-700 shadow-[0_12px_35px_rgba(15,23,42,0.06)] ring-1 ring-blue-100' : 'text-slate-500'"
                    class="flex h-8.5 items-center gap-1.5 rounded-lg px-3 text-[12px] font-medium"
                >
                    <span>{{ tab.label }}</span>
                    <span v-if="tab.badge" class="rounded-full bg-rose-50 px-2 py-0.5 text-[11px] font-bold text-rose-600">{{ tab.badge }}</span>
                </button>
            </nav>

            <section v-if="mainTab === 'teams'" class="surface-card overflow-visible">
                <div class="border-b border-slate-100 bg-linear-to-b from-slate-50/80 to-white px-3 py-3">
                    <div class="flex flex-wrap items-center gap-1.5">
                        <label class="soft-ring flex h-9 min-w-[240px] flex-1 items-center gap-2 rounded-lg px-3">
                            <span class="text-slate-400">🔎</span>
                            <input
                                v-model="teamSearch"
                                type="text"
                                class="w-full border-none bg-transparent text-[12px] outline-none placeholder:text-slate-400"
                                placeholder="Search by team name, team ID, user, invoice ID, email..."
                            />
                        </label>
                        <label :class="[statusFilterClass(selectedStatus), 'flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium']">
                            <span>Status</span>
                            <span :class="[selectedStatusMeta.dot, 'h-2 w-2 rounded-full']"></span>
                            <select v-model="selectedStatus" class="border-none bg-transparent pr-5 text-[12px] outline-none">
                                <option v-for="chip in statusChips" :key="`team-status-${chip.key}`" :value="chip.key">{{ chip.label }}</option>
                            </select>
                        </label>
                        <div class="soft-ring flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">
                            <span>From</span>
                            <input v-model="selectedTeamFromDate" type="date" class="min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none" />
                            <span class="text-slate-300">-</span>
                            <span>To</span>
                            <input v-model="selectedTeamToDate" type="date" class="min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none" />
                        </div>
                        <div class="soft-ring flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">
                            <span>Amount</span>
                            <span class="text-slate-300">$</span>
                            <input v-model="teamAmountMin" type="number" min="0" step="1" class="w-20 min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none placeholder:text-slate-400" placeholder="Minimum" />
                        </div>
                        <button @click="openMoreFilters('teams')" class="soft-ring flex h-9 items-center gap-1.5 rounded-lg px-2.5 text-[12px] font-semibold text-slate-700">☰ <span>More Filters</span></button>
                        <button @click="resetFilters" class="h-9 px-1.5 text-[12px] font-semibold text-blue-600">Clear</button>
                    </div>
                </div>

                <div class="flex flex-col xl:flex-row">
                    <div class="min-w-0 flex-1" :class="drawerOpen ? 'border-r border-slate-100' : ''">
                        <div class="overflow-x-auto overflow-y-visible">
                            <table class="min-w-full text-left">
                                <thead class="bg-slate-50/70 text-[9px] font-bold uppercase tracking-[0.14em] text-slate-500">
                                    <tr>
                                        <th class="px-3 py-2.5"><span class="block h-3.5 w-3.5 rounded border border-slate-300 bg-white"></span></th>
                                        <th class="px-3 py-2.5">Team ID</th>
                                        <th class="px-3 py-2.5">Team</th>
                                        <th class="px-3 py-2.5">Invoices</th>
                                        <th class="px-3 py-2.5">Status</th>
                                        <th class="px-3 py-2.5">Agent</th>
                                        <th class="px-3 py-2.5">Total Unpaid</th>
                                        <th class="px-3 py-2.5">Unpaid Since</th>
                                        <th class="px-3 py-2.5">Note</th>
                                        <th class="px-3 py-2.5">Actions</th>
                                    </tr>
                                </thead>
                                <tbody v-if="filteredTeams.length">
                                    <tr
                                        v-for="(team, index) in paginatedTeams"
                                        :key="team.id"
                                        @click="selectTeam(team.id)"
                                        :class="selectedTeam?.id === team.id && drawerOpen ? 'bg-blue-50/80' : 'bg-white hover:bg-slate-50/80'"
                                        class="cursor-pointer border-t border-slate-100 align-top text-[12px] text-slate-700 transition-colors"
                                    >
                                        <td class="px-3 py-2.5"><span class="block h-3.5 w-3.5 rounded border border-slate-300 bg-white"></span></td>
                                        <td class="px-3 py-2.5 font-semibold text-slate-800">{{ team.id }}</td>
                                        <td class="px-3 py-2.5">
                                            <div class="font-semibold text-slate-900">{{ team.name }}</div>
                                            <div class="mt-0.5 text-[11px] leading-4 text-slate-500">{{ team.email }}</div>
                                        </td>
                                        <td class="px-3 py-2.5 font-semibold">{{ team.invoices }}</td>
                                        <td class="px-3 py-2.5">
                                            <span :class="[badgeClass(team.statusKey), 'inline-flex items-center gap-1 rounded-full px-2 py-0.5 text-[10px] font-bold']">
                                                <span class="h-1.5 w-1.5 rounded-full bg-current"></span>
                                                {{ team.status }}
                                            </span>
                                        </td>
                                        <td class="px-3 py-2.5 font-medium">{{ team.agent }}</td>
                                        <td class="px-3 py-2.5 font-semibold text-slate-900">{{ team.totalUnpaid }}</td>
                                        <td class="px-3 py-2.5">
                                            <div>{{ formatMonthYear(team.unpaidSince) }}</div>
                                            <div class="mt-0.5 text-[11px] leading-4 text-slate-500">{{ monthsOverdueLabel(team.unpaidSince) }}</div>
                                        </td>
                                        <td class="px-3 py-2.5 text-slate-600">{{ team.note }}</td>
                                        <td class="px-3 py-2.5">
                                            <div class="relative flex items-center gap-2" data-action-menu @click.stop>
                                                <button @click="toggleMenu($event, team.id)" class="soft-ring grid h-7 w-7 place-items-center rounded-md text-slate-500">⋯</button>
                                            </div>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>

                        <div v-if="!filteredTeams.length" class="px-6 py-16 text-center">
                            <h3 class="text-xl font-semibold text-slate-900">No matching teams</h3>
                            <p class="mt-2 text-slate-500">Try a different team name, Team ID, email, invoice ID, or status filter.</p>
                        </div>

                        <div class="flex flex-wrap items-center justify-between gap-3 border-t border-slate-100 px-3 py-3 text-[12px] text-slate-500">
                            <div>Showing {{ filteredTeams.length ? `${teamsRangeStart}-${teamsRangeEnd}` : '0' }} of {{ filteredTeams.length }} results</div>
                            <div class="flex flex-wrap items-center gap-2">
                                <button @click="setTeamsPage(teamsPage - 1)" :disabled="teamsPage === 1" class="soft-ring h-8 rounded-md px-2.5 disabled:cursor-not-allowed disabled:opacity-50">Previous</button>
                                <button
                                    v-for="page in teamPageNumbers"
                                    :key="page"
                                    @click="setTeamsPage(page)"
                                    :class="teamsPage === page ? 'border border-blue-200 bg-blue-600 font-semibold text-white' : 'soft-ring'"
                                    class="grid h-8 min-w-8 place-items-center rounded-md px-2"
                                >
                                    {{ page }}
                                </button>
                                <button @click="setTeamsPage(teamsPage + 1)" :disabled="teamsPage === teamsTotalPages" class="soft-ring h-8 rounded-md px-2.5 disabled:cursor-not-allowed disabled:opacity-50">Next</button>
                                <label class="soft-ring flex h-8 items-center rounded-md px-2.5 text-[12px] text-slate-600">
                                    <select v-model="teamsRowsPerPage" class="border-none bg-transparent text-[12px] outline-none">
                                        <option v-for="size in rowsPerPageOptions" :key="size" :value="size">{{ size }}</option>
                                    </select>
                                    <span class="ml-1">/ page</span>
                                </label>
                            </div>
                        </div>
                    </div>

                    <aside v-if="drawerOpen && selectedTeam" class="flex w-full shrink-0 flex-col xl:w-[288px]">
                        <div class="border-b border-slate-100 bg-slate-50/70 px-3 py-3">
                            <div class="mb-2.5 flex items-center justify-between">
                                <h2 class="text-lg font-semibold text-slate-900">Team Details</h2>
                                <button @click="closeDrawer" class="text-2xl text-slate-400">×</button>
                            </div>
                            <div class="rounded-xl border border-slate-200 bg-white p-3">
                                <div class="flex items-center gap-2.5">
                                    <div class="grid h-10 w-10 place-items-center rounded-lg bg-blue-100 text-sm font-bold text-blue-700">{{ selectedTeam.initials }}</div>
                                    <div class="min-w-0">
                                        <div class="flex flex-wrap items-center gap-2">
                                            <h3 class="text-sm font-semibold text-slate-900">{{ selectedTeam.name }}</h3>
                                            <span :class="[badgeClass(selectedTeam.statusKey), 'inline-flex items-center rounded-full px-2 py-0.5 text-[10px] font-bold']">{{ selectedTeam.status }}</span>
                                        </div>
                                        <p class="mt-1 truncate text-[12px] text-slate-500">{{ selectedTeam.email }}</p>
                                        <p class="mt-0.5 text-[11px] text-slate-500">Managed by: {{ selectedTeam.managedBy }}</p>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="flex border-b border-slate-100 px-3">
                            <button @click="drawerTab = 'overview'" :class="drawerTab === 'overview' ? 'border-blue-500 text-blue-700' : 'border-transparent text-slate-500'" class="border-b-2 px-1.5 py-2.5 text-[12px] font-semibold">Overview</button>
                            <button @click="drawerTab = 'invoices'" :class="drawerTab === 'invoices' ? 'border-blue-500 text-blue-700' : 'border-transparent text-slate-500'" class="border-b-2 px-1.5 py-2.5 text-[12px] font-semibold">Invoices ({{ outstandingInvoices.length }})</button>
                            <button @click="drawerTab = 'notes'" :class="drawerTab === 'notes' ? 'border-blue-500 text-blue-700' : 'border-transparent text-slate-500'" class="border-b-2 px-1.5 py-2.5 text-[12px] font-semibold">Notes</button>
                        </div>

                        <div class="flex-1 overflow-y-auto px-3 py-3">
                            <div v-if="drawerTab === 'overview'" class="space-y-2.5">
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="text-xs text-slate-500">Total Unpaid Amount</div>
                                    <div class="mt-1.5 text-[32px] leading-none font-semibold tracking-tight text-slate-950">{{ selectedTeam.totalUnpaid }}</div>
                                </section>
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="space-y-2.5">
                                        <div class="flex items-start gap-2.5">
                                            <div class="grid h-8 w-8 place-items-center rounded-lg bg-slate-100 text-slate-500">📅</div>
                                            <div>
                                                <div class="text-xs text-slate-500">Unpaid Since</div>
                                                <div class="text-[13px] font-semibold text-slate-900">{{ formatMonthYear(selectedTeam.unpaidSince) }}</div>
                                                <div class="text-[12px] text-slate-500">{{ monthsOverdueLabel(selectedTeam.unpaidSince) }}</div>
                                            </div>
                                        </div>
                                        <div class="flex items-start gap-2.5">
                                            <div class="grid h-8 w-8 place-items-center rounded-lg bg-slate-100 text-slate-500">🧾</div>
                                            <div>
                                                <div class="text-xs text-slate-500">Last Invoice</div>
                                                <button
                                                    @click="copyText(selectedTeam.lastInvoice)"
                                                    :title="selectedTeam.lastInvoice"
                                                    class="mt-1 inline-flex max-w-full items-center gap-1 rounded-lg bg-slate-100 px-2 py-1 text-left text-[10px] font-semibold text-slate-700 hover:bg-slate-200"
                                                >
                                                    <span class="truncate">{{ selectedTeam.lastInvoice }}</span>
                                                    <span class="shrink-0 text-[9px] text-slate-500">{{ copiedText === selectedTeam.lastInvoice ? 'Copied' : 'Copy' }}</span>
                                                </button>
                                            </div>
                                        </div>
                                        <div class="flex items-start gap-2.5">
                                            <div class="grid h-8 w-8 place-items-center rounded-lg bg-slate-100 text-slate-500">🖥</div>
                                            <div>
                                                <div class="text-xs text-slate-500">Linked Servers</div>
                                                <div class="text-[13px] font-semibold text-slate-900">{{ selectedTeam.linkedServers }} servers</div>
                                            </div>
                                        </div>
                                    </div>
                                </section>
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="mb-2.5 text-[11px] font-semibold text-slate-900">Actions</div>
                                    <div class="space-y-2">
                                        <button class="soft-ring flex h-9 w-full items-center justify-center rounded-lg text-[12px] font-semibold text-slate-700">Send Reminder</button>
                                        <button class="flex h-9 w-full items-center justify-center rounded-lg border border-rose-200 bg-rose-50 text-[12px] font-semibold text-rose-600">Remove Next Month</button>
                                    </div>
                                </section>
                            </div>

                            <div v-else-if="drawerTab === 'invoices'" class="space-y-2.5">
                                <section class="rounded-xl border border-slate-200 bg-slate-50/70 p-3">
                                    <div class="grid grid-cols-1 gap-3 text-[12px] sm:grid-cols-3">
                                        <div><div class="text-slate-500">Total unpaid amount</div><div class="mt-1 font-semibold text-slate-900">{{ selectedTeam.totalUnpaid }}</div></div>
                                        <div><div class="text-slate-500">Number of unpaid invoices</div><div class="mt-1 font-semibold text-slate-900">{{ outstandingInvoices.length }}</div></div>
                                        <div><div class="text-slate-500">Oldest unpaid date</div><div class="mt-1 font-semibold text-slate-900">{{ outstandingInvoices[0]?.due ?? 'N/A' }}</div></div>
                                    </div>
                                </section>
                                <article v-for="invoice in outstandingInvoices" :key="invoice.id" class="rounded-xl border border-slate-200 bg-white p-3 shadow-sm">
                                    <div class="flex flex-col gap-2">
                                        <div class="min-w-0">
                                            <button
                                                @click="copyText(invoice.id)"
                                                :title="invoice.id"
                                                class="flex max-w-full items-center gap-1 rounded-lg bg-slate-100 px-2 py-1 text-left text-[10px] font-semibold text-slate-800 hover:bg-slate-200"
                                            >
                                                <span class="truncate">{{ invoice.id }}</span>
                                                <span class="shrink-0 text-[9px] text-slate-500">{{ copiedText === invoice.id ? 'Copied' : 'Copy' }}</span>
                                            </button>
                                            <div class="mt-1 text-[12px] font-semibold text-slate-900"><span class="text-slate-500">({{ invoice.serviceType }})</span> - {{ invoice.amount }}</div>
                                            <div class="mt-0.5 text-[12px] text-slate-500">{{ invoice.period }}</div>
                                        </div>
                                        <span :class="[badgeClass(invoice.statusKey), 'inline-flex w-fit items-center rounded-full px-2 py-0.5 text-[10px] font-bold']">{{ invoice.status }}</span>
                                    </div>
                                    <div class="mt-2.5 grid gap-2.5 text-[11px] text-slate-600 sm:grid-cols-2">
                                        <div><div class="text-slate-500">Invoice creation date</div><div class="font-semibold text-slate-900">{{ invoice.created }}</div></div>
                                        <div><div class="text-slate-500">Due date</div><div class="font-semibold text-slate-900">{{ invoice.due }}</div></div>
                                        <div><div class="text-slate-500">Overdue days</div><div class="font-semibold text-slate-900">{{ invoice.overdue }} days</div></div>
                                        <div class="sm:col-span-2"><div class="text-slate-500">Related server or service</div><div class="font-semibold text-slate-900">{{ invoice.service }}</div></div>
                                    </div>
                                    <div class="mt-2.5 flex flex-wrap items-center justify-between gap-2">
                                        <button class="soft-ring flex h-8 items-center gap-1 rounded-md px-2.5 text-[11px] font-semibold text-blue-700">👁 <span>View Invoice</span></button>
                                        <div class="flex items-center gap-2">
                                            <button class="text-[11px] font-semibold text-slate-500">⤓ Download</button>
                                            <button class="soft-ring grid h-8 w-8 place-items-center rounded-md text-slate-500">⋯</button>
                                        </div>
                                    </div>
                                </article>
                            </div>

                            <div v-else class="space-y-2.5">
                                <div v-for="(note, index) in selectedTeam.notes" :key="index" class="flex items-start gap-2.5">
                                    <div class="grid h-8 w-8 shrink-0 place-items-center rounded-lg text-[12px]" :class="note.type === 'system' ? 'bg-slate-100 text-slate-600' : 'bg-blue-100 text-blue-700'">
                                        {{ note.type === 'system' ? 'S' : note.initials }}
                                    </div>
                                    <div class="w-full rounded-xl border p-3" :class="note.type === 'system' ? 'border-dashed border-slate-200 bg-slate-50' : 'border-slate-200 bg-white'">
                                        <div class="flex flex-wrap items-center justify-between gap-2">
                                            <div>
                                                <span class="text-[13px] font-semibold text-slate-900">{{ note.author }}</span>
                                                <span class="ml-1.5 text-[11px] text-slate-500">{{ note.role }}</span>
                                            </div>
                                            <div class="text-[11px] text-slate-400">{{ note.time }}</div>
                                        </div>
                                        <p class="mt-2 text-[12px] leading-5 text-slate-700">{{ note.message }}</p>
                                        <div v-if="note.statusLabel" class="mt-2 inline-flex rounded-full bg-blue-50 px-2 py-0.5 text-[10px] font-bold text-blue-700">{{ note.statusLabel }}</div>
                                        <div v-if="note.edited" class="mt-2 text-[11px] text-slate-400">Edited</div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div v-if="drawerTab === 'notes'" class="border-t border-slate-100 bg-white px-3 py-3">
                            <div class="mb-2 text-[11px] font-medium text-slate-500">Private note. Visible only to xCloud team members.</div>
                            <div class="rounded-xl border border-slate-200 bg-slate-50/70 p-3">
                                <textarea class="min-h-18 w-full resize-y border-none bg-transparent text-[12px] text-slate-700 outline-none placeholder:text-slate-400" placeholder="Write an internal note..."></textarea>
                                <div class="mt-2.5 flex flex-wrap items-center justify-between gap-3">
                                    <div class="flex flex-wrap items-center gap-3">
                                        <button class="text-[11px] font-semibold text-slate-500">📎 Attachment</button>
                                        <button class="text-[11px] font-semibold text-slate-500">@ Mention teammate</button>
                                    </div>
                                    <button class="rounded-lg bg-blue-600 px-3 py-2 text-[12px] font-semibold text-white shadow-lg shadow-blue-600/20">Send</button>
                                </div>
                            </div>
                        </div>
                    </aside>
                </div>
            </section>

            <section v-else-if="mainTab === 'bills'" class="surface-card overflow-visible">
                <div class="border-b border-slate-100 bg-linear-to-b from-slate-50/80 to-white px-3 py-3">
                    <div class="flex flex-wrap items-center gap-1.5">
                        <label class="soft-ring flex h-9 min-w-[240px] flex-1 items-center gap-2 rounded-lg px-3">
                            <span class="text-slate-400">🔎</span>
                            <input type="text" class="w-full border-none bg-transparent text-[12px] outline-none placeholder:text-slate-400" placeholder="Search by invoice ID, team ID, team name, or email..." />
                        </label>
                        <label :class="[statusFilterClass(selectedInvoiceStatus), 'flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium']">
                            <span>Status</span>
                            <span :class="[selectedInvoiceStatusMeta.dot, 'h-2 w-2 rounded-full']"></span>
                            <select v-model="selectedInvoiceStatus" class="border-none bg-transparent pr-5 text-[12px] outline-none">
                                <option v-for="chip in statusChips" :key="`invoice-status-${chip.key}`" :value="chip.key">{{ chip.label }}</option>
                            </select>
                        </label>
                        <div class="soft-ring flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">
                            <span>From</span>
                            <input v-model="selectedInvoiceFromDate" type="date" class="min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none" />
                            <span class="text-slate-300">-</span>
                            <span>To</span>
                            <input v-model="selectedInvoiceToDate" type="date" class="min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none" />
                        </div>
                        <label class="soft-ring flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">
                            <span>Service Type</span>
                            <select v-model="selectedInvoiceServiceType" class="border-none bg-transparent pr-5 text-[12px] text-slate-500 outline-none">
                                <option v-for="serviceType in serviceTypeOptions" :key="serviceType" :value="serviceType">{{ serviceType }}</option>
                            </select>
                        </label>
                        <div class="soft-ring flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">
                            <span>Amount</span>
                            <span class="text-slate-300">$</span>
                            <input v-model="invoiceAmountMin" type="number" min="0" step="1" class="w-20 min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none placeholder:text-slate-400" placeholder="Minimum" />
                        </div>
                        <button class="soft-ring flex h-9 items-center gap-1.5 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">Due Date <span class="text-slate-500">Any</span> <span class="text-slate-400">⌄</span></button>
                        <button class="soft-ring flex h-9 items-center gap-1.5 rounded-lg px-2.5 text-[12px] font-semibold text-slate-700">⤓ <span>Export</span></button>
                    </div>
                </div>
                <div class="overflow-x-auto overflow-y-visible">
                    <table class="min-w-full text-left">
                        <thead class="bg-slate-50/70 text-[9px] font-bold uppercase tracking-[0.14em] text-slate-500">
                            <tr>
                                <th class="px-3 py-2.5"><span class="block h-3.5 w-3.5 rounded border border-slate-300 bg-white"></span></th>
                                <th class="px-3 py-2.5">Invoice ID</th>
                                <th class="px-3 py-2.5">Team ID</th>
                                <th class="px-3 py-2.5">Team</th>
                                <th class="px-3 py-2.5">Billing Period</th>
                                <th class="px-3 py-2.5">Invoice Date</th>
                                <th class="px-3 py-2.5">Due Date</th>
                                <th class="px-3 py-2.5">Amount</th>
                                <th class="px-3 py-2.5">Status</th>
                                <th class="px-3 py-2.5">Service Type</th>
                                <th class="px-3 py-2.5">Actions</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(bill, index) in paginatedBills" :key="bill.id" class="border-t border-slate-100 text-[12px] text-slate-700 hover:bg-slate-50/80">
                                <td class="px-3 py-2.5"><span class="block h-3.5 w-3.5 rounded border border-slate-300 bg-white"></span></td>
                                <td class="px-3 py-2.5">
                                    <button
                                        @click.stop="copyText(bill.id)"
                                        :title="bill.id"
                                        class="inline-flex max-w-[210px] items-center gap-1 rounded-lg bg-slate-100 px-2 py-1 text-left text-[10px] font-semibold text-slate-800 hover:bg-slate-200"
                                    >
                                        <span class="truncate">{{ bill.id }}</span>
                                        <span class="shrink-0 text-[9px] text-slate-500">{{ copiedText === bill.id ? 'Copied' : 'Copy' }}</span>
                                    </button>
                                </td>
                                <td class="px-3 py-2.5 font-semibold">{{ bill.teamId }}</td>
                                <td class="px-3 py-2.5">
                                    <div class="font-semibold text-slate-900">{{ bill.teamName }}</div>
                                    <div class="mt-0.5 text-[11px] leading-4 text-slate-500">{{ bill.teamEmail }}</div>
                                </td>
                                <td class="px-3 py-2.5">{{ bill.period }}</td>
                                <td class="px-3 py-2.5">{{ bill.created }}</td>
                                <td class="px-3 py-2.5">{{ bill.due }}</td>
                                <td class="px-3 py-2.5 font-semibold text-slate-900">{{ bill.amount }}</td>
                                <td class="px-3 py-2.5"><span :class="[badgeClass(bill.statusKey), 'inline-flex items-center rounded-full px-2 py-0.5 text-[10px] font-bold']">{{ bill.status }}</span></td>
                                <td class="px-3 py-2.5">{{ bill.serviceType }}</td>
                                <td class="px-3 py-2.5">
                                    <div class="relative flex items-center gap-2" data-action-menu @click.stop>
                                        <button @click="toggleInvoiceMenu($event, bill.id)" class="soft-ring grid h-7 w-7 place-items-center rounded-md text-slate-500">⋯</button>
                                    </div>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
                <div class="flex flex-wrap items-center justify-between gap-3 border-t border-slate-100 px-3 py-3 text-[12px] text-slate-500">
                    <div>Showing {{ allBills.length ? `${invoicesRangeStart}-${invoicesRangeEnd}` : '0' }} of {{ allBills.length }} results</div>
                    <div class="flex flex-wrap items-center gap-2">
                        <button @click="setInvoicesPage(invoicesPage - 1)" :disabled="invoicesPage === 1" class="soft-ring h-8 rounded-md px-2.5 disabled:cursor-not-allowed disabled:opacity-50">Previous</button>
                        <button
                            v-for="page in invoicePageNumbers"
                            :key="page"
                            @click="setInvoicesPage(page)"
                            :class="invoicesPage === page ? 'border border-blue-200 bg-blue-600 font-semibold text-white' : 'soft-ring'"
                            class="grid h-8 min-w-8 place-items-center rounded-md px-2"
                        >
                            {{ page }}
                        </button>
                        <button @click="setInvoicesPage(invoicesPage + 1)" :disabled="invoicesPage === invoicesTotalPages" class="soft-ring h-8 rounded-md px-2.5 disabled:cursor-not-allowed disabled:opacity-50">Next</button>
                        <label class="soft-ring flex h-8 items-center rounded-md px-2.5 text-[12px] text-slate-600">
                            <select v-model="invoicesRowsPerPage" class="border-none bg-transparent text-[12px] outline-none">
                                <option v-for="size in rowsPerPageOptions" :key="size" :value="size">{{ size }}</option>
                            </select>
                            <span class="ml-1">/ page</span>
                        </label>
                    </div>
                </div>
            </section>

            <section v-else-if="mainTab === 'servers'" class="surface-card overflow-visible">
                <div class="border-b border-slate-100 bg-linear-to-b from-slate-50/80 to-white px-3 py-3">
                    <div class="flex flex-wrap items-center gap-1.5">
                        <label class="soft-ring flex h-9 min-w-[260px] flex-1 items-center gap-2 rounded-lg px-3">
                            <span class="text-slate-400">🔎</span>
                            <input v-model="serverSearch" type="text" class="w-full border-none bg-transparent text-[12px] outline-none placeholder:text-slate-400" placeholder="Search by server ID, team, owner, plan, or note..." />
                        </label>
                        <label :class="[statusFilterClass(selectedServerStatus), 'flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium']">
                            <span>Status</span>
                            <span :class="[selectedServerStatusMeta.dot, 'h-2 w-2 rounded-full']"></span>
                            <select v-model="selectedServerStatus" class="border-none bg-transparent pr-5 text-[12px] outline-none">
                                <option v-for="chip in statusChips" :key="`server-status-${chip.key}`" :value="chip.key">{{ chip.label }}</option>
                            </select>
                        </label>
                        <div class="soft-ring flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">
                            <span>From</span>
                            <input v-model="selectedServerFromDate" type="date" class="min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none" />
                            <span class="text-slate-300">-</span>
                            <span>To</span>
                            <input v-model="selectedServerToDate" type="date" class="min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none" />
                        </div>
                        <div class="soft-ring flex h-9 items-center gap-2 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">
                            <span>Amount</span>
                            <span class="text-slate-300">$</span>
                            <input v-model="serverAmountMin" type="number" min="0" step="1" class="w-20 min-w-0 border-none bg-transparent text-[12px] text-slate-500 outline-none placeholder:text-slate-400" placeholder="Minimum" />
                        </div>
                        <button class="soft-ring flex h-9 items-center gap-1.5 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">Region <span class="text-slate-500">All Regions</span> <span class="text-slate-400">⌄</span></button>
                        <button class="soft-ring flex h-9 items-center gap-1.5 rounded-lg px-2.5 text-[12px] font-medium text-slate-700">Months Unpaid <span class="text-slate-500">Any</span> <span class="text-slate-400">⌄</span></button>
                    </div>
                </div>

                <div class="flex flex-col xl:flex-row">
                    <div class="min-w-0 flex-1" :class="serverDrawerOpen ? 'border-r border-slate-100' : ''">
                        <div class="overflow-x-auto overflow-y-visible">
                            <table class="min-w-full text-left">
                                <thead class="bg-slate-50/70 text-[9px] font-bold uppercase tracking-[0.14em] text-slate-500">
                                    <tr>
                                        <th class="px-3 py-2.5">Server ID</th>
                                        <th class="px-3 py-2.5">Nova</th>
                                        <th class="px-3 py-2.5">Vultr</th>
                                        <th class="px-3 py-2.5">Months Unpaid</th>
                                        <th class="px-3 py-2.5">Upgrade History</th>
                                        <th class="px-3 py-2.5">Service Type</th>
                                        <th class="px-3 py-2.5">Current Status</th>
                                        <th class="px-3 py-2.5">Note</th>
                                        <th class="px-3 py-2.5">Status</th>
                                        <th class="px-3 py-2.5">Actions</th>
                                    </tr>
                                </thead>
                                <tbody v-if="filteredServers.length">
                                    <tr
                                        v-for="(server, index) in paginatedServers"
                                        :key="server.id"
                                        @click="selectServer(server.id)"
                                        :class="selectedServer?.id === server.id && serverDrawerOpen ? 'bg-blue-50/80' : 'bg-white hover:bg-slate-50/80'"
                                        class="cursor-pointer border-t border-slate-100 align-top text-[12px] text-slate-700 transition-colors"
                                    >
                                        <td class="px-3 py-2.5 font-semibold text-slate-900">{{ server.id }}</td>
                                        <td class="px-3 py-2.5">
                                            <a href="#" class="font-semibold text-blue-700 hover:underline" @click.stop>{{ server.novaLink }}</a>
                                        </td>
                                        <td class="px-3 py-2.5">
                                            <a href="#" class="font-semibold text-blue-700 hover:underline" @click.stop>{{ server.vultrLink }}</a>
                                        </td>
                                        <td class="px-3 py-2.5 font-semibold">{{ server.monthsUnpaid }}</td>
                                        <td class="px-3 py-2.5">
                                            <span class="cursor-help font-semibold text-slate-900 underline decoration-dotted underline-offset-3" :title="server.upgradeHistory">{{ server.currentPlan }}</span>
                                        </td>
                                        <td class="px-3 py-2.5 font-medium">{{ server.serviceType }}</td>
                                        <td class="px-3 py-2.5">
                                            <span class="inline-flex rounded-full bg-slate-100 px-2 py-0.5 text-[10px] font-bold text-slate-700">{{ server.currentStatus }}</span>
                                        </td>
                                        <td class="px-3 py-2.5 text-slate-600">{{ server.note }}</td>
                                        <td class="px-3 py-2.5">
                                            <span :class="[badgeClass(server.billingStatusKey), 'inline-flex items-center rounded-full px-2 py-0.5 text-[10px] font-bold']">{{ server.billingStatus }}</span>
                                        </td>
                                        <td class="px-3 py-2.5">
                                            <div class="relative flex items-center gap-2" data-action-menu @click.stop>
                                                <button @click="toggleServerMenu($event, server.id)" class="soft-ring grid h-7 w-7 place-items-center rounded-md text-slate-500">⋯</button>
                                            </div>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>

                        <div v-if="!filteredServers.length" class="px-6 py-16 text-center">
                            <h3 class="text-lg font-semibold text-slate-900">No unpaid servers found</h3>
                            <p class="mt-2 text-[12px] text-slate-500">Try a different server ID, team, plan, or owner search.</p>
                        </div>
                        <div class="flex flex-wrap items-center justify-between gap-3 border-t border-slate-100 px-3 py-3 text-[12px] text-slate-500">
                            <div>Showing {{ filteredServers.length ? `${serversRangeStart}-${serversRangeEnd}` : '0' }} of {{ filteredServers.length }} results</div>
                            <div class="flex flex-wrap items-center gap-2">
                                <button @click="setServersPage(serversPage - 1)" :disabled="serversPage === 1" class="soft-ring h-8 rounded-md px-2.5 disabled:cursor-not-allowed disabled:opacity-50">Previous</button>
                                <button
                                    v-for="page in serverPageNumbers"
                                    :key="page"
                                    @click="setServersPage(page)"
                                    :class="serversPage === page ? 'border border-blue-200 bg-blue-600 font-semibold text-white' : 'soft-ring'"
                                    class="grid h-8 min-w-8 place-items-center rounded-md px-2"
                                >
                                    {{ page }}
                                </button>
                                <button @click="setServersPage(serversPage + 1)" :disabled="serversPage === serversTotalPages" class="soft-ring h-8 rounded-md px-2.5 disabled:cursor-not-allowed disabled:opacity-50">Next</button>
                                <label class="soft-ring flex h-8 items-center rounded-md px-2.5 text-[12px] text-slate-600">
                                    <select v-model="serversRowsPerPage" class="border-none bg-transparent text-[12px] outline-none">
                                        <option v-for="size in rowsPerPageOptions" :key="size" :value="size">{{ size }}</option>
                                    </select>
                                    <span class="ml-1">/ page</span>
                                </label>
                            </div>
                        </div>
                    </div>

                    <aside v-if="serverDrawerOpen && selectedServer" class="flex w-full shrink-0 flex-col xl:w-[288px]">
                        <div class="border-b border-slate-100 bg-slate-50/70 px-3 py-3">
                            <div class="mb-2.5 flex items-center justify-between">
                                <h2 class="text-lg font-semibold text-slate-900">Server Details</h2>
                                <button @click="serverDrawerOpen = false" class="text-2xl text-slate-400">×</button>
                            </div>
                            <div class="rounded-xl border border-slate-200 bg-white p-3">
                                <div class="flex items-center gap-2.5">
                                    <div class="grid h-10 w-10 place-items-center rounded-lg bg-blue-100 text-sm font-bold text-blue-700">SV</div>
                                    <div class="min-w-0">
                                        <div class="flex flex-wrap items-center gap-2">
                                            <h3 class="text-sm font-semibold text-slate-900">{{ selectedServer.id }}</h3>
                                            <span :class="[badgeClass(selectedServer.billingStatusKey), 'inline-flex items-center rounded-full px-2 py-0.5 text-[10px] font-bold']">{{ selectedServer.billingStatus }}</span>
                                        </div>
                                        <p class="mt-1 text-[12px] text-slate-500">{{ selectedServer.team }}</p>
                                        <p class="mt-0.5 text-[11px] text-slate-500">Managed by: {{ selectedServer.owner }}</p>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="flex border-b border-slate-100 px-3">
                            <button @click="serverDrawerTab = 'overview'" :class="serverDrawerTab === 'overview' ? 'border-blue-500 text-blue-700' : 'border-transparent text-slate-500'" class="border-b-2 px-1.5 py-2.5 text-[12px] font-semibold">Overview</button>
                            <button @click="serverDrawerTab = 'history'" :class="serverDrawerTab === 'history' ? 'border-blue-500 text-blue-700' : 'border-transparent text-slate-500'" class="border-b-2 px-1.5 py-2.5 text-[12px] font-semibold">Upgrade History</button>
                            <button @click="serverDrawerTab = 'notes'" :class="serverDrawerTab === 'notes' ? 'border-blue-500 text-blue-700' : 'border-transparent text-slate-500'" class="border-b-2 px-1.5 py-2.5 text-[12px] font-semibold">Notes</button>
                        </div>

                        <div class="flex-1 overflow-y-auto px-3 py-3">
                            <div v-if="serverDrawerTab === 'overview'" class="space-y-2.5">
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="text-xs text-slate-500">Unpaid Amount</div>
                                    <div class="mt-1.5 text-[32px] leading-none font-semibold tracking-tight text-slate-950">{{ selectedServer.unpaidAmount }}</div>
                                </section>
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="space-y-2.5">
                                        <div class="flex items-start gap-2.5">
                                            <div class="grid h-8 w-8 place-items-center rounded-lg bg-slate-100 text-slate-500">🗓</div>
                                            <div>
                                                <div class="text-xs text-slate-500">Overdue Since</div>
                                                <div class="text-[13px] font-semibold text-slate-900">{{ selectedServer.overdueSince }}</div>
                                            </div>
                                        </div>
                                        <div class="flex items-start gap-2.5">
                                            <div class="grid h-8 w-8 place-items-center rounded-lg bg-slate-100 text-slate-500">📦</div>
                                            <div>
                                                <div class="text-xs text-slate-500">Current Plan</div>
                                                <div class="text-[13px] font-semibold text-slate-900">{{ selectedServer.currentPlan }}</div>
                                            </div>
                                        </div>
                                        <div class="flex items-start gap-2.5">
                                            <div class="grid h-8 w-8 place-items-center rounded-lg bg-slate-100 text-slate-500">🌍</div>
                                            <div>
                                                <div class="text-xs text-slate-500">Region / IP</div>
                                                <div class="text-[13px] font-semibold text-slate-900">{{ selectedServer.region }}</div>
                                                <div class="text-[11px] text-slate-500">{{ selectedServer.ip }}</div>
                                            </div>
                                        </div>
                                        <div class="flex items-start gap-2.5">
                                            <div class="grid h-8 w-8 place-items-center rounded-lg bg-slate-100 text-slate-500">🧾</div>
                                            <div>
                                                <div class="text-xs text-slate-500">Last Invoice</div>
                                                <button
                                                    @click="copyText(selectedServer.lastInvoice)"
                                                    :title="selectedServer.lastInvoice"
                                                    class="mt-1 inline-flex max-w-full items-center gap-1 rounded-lg bg-slate-100 px-2 py-1 text-left text-[10px] font-semibold text-slate-700 hover:bg-slate-200"
                                                >
                                                    <span class="truncate">{{ selectedServer.lastInvoice }}</span>
                                                    <span class="shrink-0 text-[9px] text-slate-500">{{ copiedText === selectedServer.lastInvoice ? 'Copied' : 'Copy' }}</span>
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                </section>
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="mb-2.5 text-[11px] font-semibold text-slate-900">Action</div>
                                    <button @click="suspendServer(selectedServer.id)" class="flex h-9 w-full items-center justify-center rounded-lg border border-rose-200 bg-rose-50 text-[12px] font-semibold text-rose-600">Suspend</button>
                                </section>
                            </div>

                            <div v-else-if="serverDrawerTab === 'history'" class="space-y-2.5">
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="text-xs text-slate-500">Current plan</div>
                                    <div class="mt-1 text-[13px] font-semibold text-slate-900">{{ selectedServer.currentPlan }}</div>
                                </section>
                                <section class="rounded-xl border border-slate-200 bg-white p-3">
                                    <div class="text-xs text-slate-500">Previous upgrades</div>
                                    <div class="mt-1 text-[13px] font-semibold text-slate-900">{{ selectedServer.upgradeHistory }}</div>
                                </section>
                            </div>

                            <div v-else class="space-y-2.5">
                                <div v-for="(note, index) in selectedServer.timeline" :key="index" class="flex items-start gap-2.5">
                                    <div class="grid h-8 w-8 shrink-0 place-items-center rounded-lg text-[12px]" :class="note.type === 'system' ? 'bg-slate-100 text-slate-600' : 'bg-blue-100 text-blue-700'">
                                        {{ note.type === 'system' ? 'S' : note.initials }}
                                    </div>
                                    <div class="w-full rounded-xl border p-3" :class="note.type === 'system' ? 'border-dashed border-slate-200 bg-slate-50' : 'border-slate-200 bg-white'">
                                        <div class="flex flex-wrap items-center justify-between gap-2">
                                            <div>
                                                <span class="text-[13px] font-semibold text-slate-900">{{ note.author }}</span>
                                                <span class="ml-1.5 text-[11px] text-slate-500">{{ note.role }}</span>
                                            </div>
                                            <div class="text-[11px] text-slate-400">{{ note.time }}</div>
                                        </div>
                                        <p class="mt-2 text-[12px] leading-5 text-slate-700">{{ note.message }}</p>
                                        <div v-if="note.statusLabel" class="mt-2 inline-flex rounded-full bg-blue-50 px-2 py-0.5 text-[10px] font-bold text-blue-700">{{ note.statusLabel }}</div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </aside>
                </div>
            </section>

            <section v-else class="space-y-3">
                <div class="flex flex-wrap items-center justify-between gap-2">
                    <div class="text-[13px] font-semibold text-slate-900">Reports & Stats</div>
                    <div class="flex flex-wrap items-center gap-2">
                        <button class="soft-ring flex h-9 items-center gap-2 rounded-lg px-3 text-[12px] font-medium text-slate-700">
                            <span>📅</span>
                            <span>Aug 1, 2025 - Aug 31, 2025</span>
                            <span class="text-slate-400">⌄</span>
                        </button>
                        <button class="soft-ring flex h-9 items-center gap-2 rounded-lg px-3 text-[12px] font-semibold text-blue-700">
                            <span>⤓</span>
                            <span>Export Report</span>
                        </button>
                    </div>
                </div>

                <section class="grid gap-2.5 xl:grid-cols-4">
                    <article v-for="card in summaryCards" :key="card.title" class="surface-card grid grid-cols-[42px_1fr] gap-2.5 p-3">
                        <div :class="['grid h-10 w-10 place-items-center rounded-lg bg-linear-to-br text-sm font-bold text-white shadow-lg', card.iconTone]">{{ card.icon }}</div>
                        <div>
                            <p class="text-[11px] leading-4 text-slate-500">{{ card.title }}</p>
                            <div class="mt-1.5 flex flex-wrap items-end gap-1.5">
                                <span class="text-[30px] leading-none font-semibold tracking-tight text-slate-950">{{ card.value }}</span>
                                <span v-if="card.subvalue" class="pb-0.5 text-[12px] font-medium text-slate-500">{{ card.subvalue }}</span>
                                <span v-if="card.trend" :class="['rounded-full px-2 py-0.5 text-[11px] font-bold', card.trendTone]">{{ card.trend }}</span>
                            </div>
                            <p class="mt-1.5 text-[11px] leading-4 text-slate-500">{{ card.helper }}</p>
                        </div>
                    </article>
                </section>

                <section class="grid gap-3 lg:grid-cols-3">
                    <article class="surface-card p-4" v-for="panel in 3" :key="panel">
                        <h3 class="text-base font-semibold text-slate-900">Reporting Workspace</h3>
                        <p class="mt-1.5 text-[12px] leading-5 text-slate-500">
                            Summaries, exports, agent workload, and downloadable operational reports would live here using the same xCloud card and table system.
                        </p>
                    </article>
                </section>
            </section>

            <div v-if="modalState.type" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-950/35 px-4">
                <div :class="modalState.type === 'more-filters' ? 'max-w-3xl' : 'max-w-md'" class="w-full rounded-2xl border border-slate-200 bg-white p-5 shadow-2xl shadow-slate-950/15">
                    <div class="flex items-start justify-between gap-3">
                        <div>
                            <h3 class="text-lg font-semibold text-slate-900">{{ modalState.title }}</h3>
                            <p class="mt-1 text-[12px] text-slate-500">{{ modalState.targetLabel }}</p>
                        </div>
                        <button @click="closeModal" class="text-xl text-slate-400">×</button>
                    </div>

                    <div v-if="modalState.type === 'suspend-server'" class="mt-4 space-y-3">
                        <label class="block">
                            <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Notice shown to the user</span>
                            <textarea v-model="modalState.notice" class="min-h-28 w-full rounded-xl border border-slate-200 bg-slate-50 px-3 py-2 text-[12px] text-slate-700 outline-none" placeholder="Write a suspension notice..."></textarea>
                        </label>
                    </div>

                    <div v-else-if="modalState.type === 'activate-team'" class="mt-4 space-y-3">
                        <label class="block">
                            <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Duration</span>
                            <select v-model="modalState.duration" class="w-full rounded-xl border border-slate-200 bg-slate-50 px-3 py-2 text-[12px] text-slate-700 outline-none">
                                <option>7 days</option>
                                <option>14 days</option>
                                <option>30 days</option>
                            </select>
                        </label>
                        <label class="flex items-center gap-2 text-[12px] text-slate-700">
                            <input v-model="modalState.emailUser" type="checkbox" class="h-4 w-4 rounded border-slate-300" />
                            <span>Email the user</span>
                        </label>
                    </div>

                    <div v-else-if="modalState.type === 'more-filters'" class="mt-4 space-y-4">
                        <div class="grid gap-3 md:grid-cols-2">
                            <label class="block">
                                <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Status</span>
                                <div :class="[statusFilterClass(selectedStatus), 'flex items-center gap-2 rounded-xl px-3 py-2 text-[12px] font-medium']">
                                    <span :class="[selectedStatusMeta.dot, 'h-2 w-2 rounded-full']"></span>
                                    <select v-model="selectedStatus" class="w-full border-none bg-transparent outline-none">
                                    <option v-for="chip in statusChips" :key="`modal-status-${chip.key}`" :value="chip.key">{{ chip.label }}</option>
                                    </select>
                                </div>
                            </label>
                            <label class="block">
                                <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Billing From</span>
                                <input v-model="selectedTeamFromDate" type="date" class="w-full rounded-xl border border-slate-200 bg-slate-50 px-3 py-2 text-[12px] text-slate-700 outline-none" />
                            </label>
                            <label class="block">
                                <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Billing To</span>
                                <input v-model="selectedTeamToDate" type="date" class="w-full rounded-xl border border-slate-200 bg-slate-50 px-3 py-2 text-[12px] text-slate-700 outline-none" />
                            </label>
                            <label class="block">
                                <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Minimum Amount</span>
                                <input v-model="teamAmountMin" type="number" min="0" step="1" class="w-full rounded-xl border border-slate-200 bg-slate-50 px-3 py-2 text-[12px] text-slate-700 outline-none" placeholder="Minimum unpaid amount" />
                            </label>
                            <label class="block">
                                <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Rows Per Page</span>
                                <select v-model="teamsRowsPerPage" class="w-full rounded-xl border border-slate-200 bg-slate-50 px-3 py-2 text-[12px] text-slate-700 outline-none">
                                    <option v-for="size in rowsPerPageOptions" :key="`modal-page-size-${size}`" :value="size">{{ size }} / page</option>
                                </select>
                            </label>
                        </div>

                        <div class="rounded-2xl border border-dashed border-slate-200 bg-slate-50/80 p-4">
                            <div class="mb-3 text-[12px] font-semibold text-slate-800">Placeholder filters for later</div>
                            <div class="grid gap-3 md:grid-cols-2">
                                <label class="block">
                                    <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Account Tier</span>
                                    <select class="w-full rounded-xl border border-slate-200 bg-white px-3 py-2 text-[12px] text-slate-500 outline-none">
                                        <option>Any tier</option>
                                        <option>Starter</option>
                                        <option>Growth</option>
                                        <option>Enterprise</option>
                                    </select>
                                </label>
                                <label class="block">
                                    <span class="mb-1.5 block text-[12px] font-medium text-slate-700">Payment Risk</span>
                                    <select class="w-full rounded-xl border border-slate-200 bg-white px-3 py-2 text-[12px] text-slate-500 outline-none">
                                        <option>Any risk level</option>
                                        <option>Low</option>
                                        <option>Medium</option>
                                        <option>High</option>
                                    </select>
                                </label>
                                <label class="flex items-center gap-2 rounded-xl border border-slate-200 bg-white px-3 py-2 text-[12px] text-slate-700">
                                    <input type="checkbox" class="h-4 w-4 rounded border-slate-300" />
                                    <span>Only pinned teams</span>
                                </label>
                                <label class="flex items-center gap-2 rounded-xl border border-slate-200 bg-white px-3 py-2 text-[12px] text-slate-700">
                                    <input type="checkbox" class="h-4 w-4 rounded border-slate-300" />
                                    <span>Only teams with notes</span>
                                </label>
                            </div>
                        </div>
                    </div>

                    <div class="mt-5 flex items-center justify-end gap-2">
                        <button @click="closeModal" class="rounded-xl border border-slate-200 px-3 py-2 text-[12px] font-semibold text-slate-600">Cancel</button>
                        <button @click="submitModalAction" class="rounded-xl bg-blue-600 px-3 py-2 text-[12px] font-semibold text-white">
                            {{ modalState.type === 'suspend-server' ? 'Suspend Server' : modalState.type === 'activate-team' ? 'Activate Team' : 'Apply Filters' }}
                        </button>
                    </div>
                </div>
            </div>

            <div
                v-if="activeMenu.type === 'team' && openMenuId"
                data-action-menu
                :class="['fixed z-[200] rounded-xl border border-slate-200 bg-white p-1.5 shadow-2xl shadow-slate-900/10', activeMenu.openUp ? '-translate-y-full' : '']"
                :style="{ top: `${activeMenu.top}px`, left: `${activeMenu.left}px`, width: `${activeMenu.width}px` }"
                @click.stop
            >
                <button class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">View in Nova</button>
                <button class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">View User in Nova</button>
                <button class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Add Internal Note</button>
                <button class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Send Payment Reminder</button>
                <button class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Copy Team ID</button>
                <button class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Copy Team Email</button>
                <button class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Pin Team</button>
                <button @click="activateTeam(openMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Activate Team</button>
                <button @click="confirmAction('Set Grace Period', teams.find((item) => item.id === openMenuId)?.name || openMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Set Grace Period</button>
                <button @click="confirmAction('Restore as Unpaid/Deactivate', teams.find((item) => item.id === openMenuId)?.name || openMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Restore as unpaid/Deactivate</button>
                <button @click="ignoreForOneMonth('team', openMenuId, teams.find((item) => item.id === openMenuId)?.name || openMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[13px] hover:bg-slate-50">Ignore for 1 month</button>
            </div>

            <div
                v-if="activeMenu.type === 'invoice' && invoiceMenuId"
                data-action-menu
                :class="['fixed z-[200] rounded-xl border border-slate-200 bg-white p-1.5 shadow-2xl shadow-slate-900/10', activeMenu.openUp ? '-translate-y-full' : '']"
                :style="{ top: `${activeMenu.top}px`, left: `${activeMenu.left}px`, width: `${activeMenu.width}px` }"
                @click.stop
            >
                <button @click="confirmAction('View in Nova', invoiceMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[12px] hover:bg-slate-50">View in Nova</button>
                <button @click="takePayment(invoiceMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[12px] hover:bg-slate-50">Take Payment</button>
                <button @click="takePayment(invoiceMenuId, true)" class="w-full rounded-lg px-3 py-2 text-left text-[12px] hover:bg-slate-50">Force Take Payment</button>
                <button @click="ignoreForOneMonth('invoice', invoiceMenuId, invoiceMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[12px] hover:bg-slate-50">Ignore for 1 month</button>
            </div>

            <div
                v-if="activeMenu.type === 'server' && serverMenuId"
                data-action-menu
                :class="['fixed z-[200] rounded-xl border border-slate-200 bg-white p-1.5 shadow-2xl shadow-slate-900/10', activeMenu.openUp ? '-translate-y-full' : '']"
                :style="{ top: `${activeMenu.top}px`, left: `${activeMenu.left}px`, width: `${activeMenu.width}px` }"
                @click.stop
            >
                <button @click="suspendServer(serverMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[12px] text-rose-600 hover:bg-rose-50">Suspend</button>
                <button @click="terminateServer(serverMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[12px] text-rose-600 hover:bg-rose-50">Terminate</button>
                <button @click="ignoreForOneMonth('server', serverMenuId, serverMenuId)" class="w-full rounded-lg px-3 py-2 text-left text-[12px] hover:bg-slate-50">Ignore for 1 month</button>
            </div>
        </main>
    </div>
</template>
