<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Dashboard Hiệu Suất</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap');

  :root{
    --bg:#0F1117;--surface:#161B27;--surface2:#1E2535;--surface3:#252D3D;
    --accent:#6C63FF;--accent2:#10B981;--accent3:#F59E0B;--accent4:#EF4444;
    --text:#E8EAF0;--muted:#8B93A8;--border:rgba(255,255,255,.07);
    --w1:#3B82F6;--w2:#10B981;--w3:#F59E0B;--w4:#EF4444;--w5:#8B5CF6;
    --r:10px;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  html,body{height:100%;font-family:'DM Sans',sans-serif;background:var(--bg);color:var(--text);font-size:13px}
  ::-webkit-scrollbar{width:4px;height:4px}
  ::-webkit-scrollbar-track{background:transparent}
  ::-webkit-scrollbar-thumb{background:var(--surface3);border-radius:4px}

  /* ── Layout ── */
  .layout{display:grid;grid-template-columns:200px 1fr;height:100vh;overflow:hidden}

  /* ── Sidebar ── */
  .sidebar{background:var(--surface);border-right:1px solid var(--border);display:flex;flex-direction:column;padding:20px 0;overflow:hidden}
  .brand{padding:0 20px 20px;border-bottom:1px solid var(--border)}
  .brand-icon{width:32px;height:32px;background:var(--accent);border-radius:8px;display:flex;align-items:center;justify-content:center;font-size:16px;margin-bottom:10px}
  .brand h2{font-size:13px;font-weight:600;color:var(--text);line-height:1.3}
  .brand p{font-size:11px;color:var(--muted);margin-top:2px}
  .nav{padding:16px 12px;flex:1}
  .nav-label{font-size:10px;font-weight:600;color:var(--muted);letter-spacing:.08em;text-transform:uppercase;padding:0 8px;margin-bottom:6px}
  .nav-item{display:flex;align-items:center;gap:10px;padding:9px 10px;border-radius:8px;cursor:pointer;color:var(--muted);font-size:12px;font-weight:500;transition:all .15s;margin-bottom:2px}
  .nav-item:hover{background:var(--surface2);color:var(--text)}
  .nav-item.active{background:var(--accent);color:#fff}
  .nav-item .icon{font-size:15px;width:18px;text-align:center}
  .week-section{padding:16px 12px 0;border-top:1px solid var(--border);margin-top:auto}
  .week-btn{display:block;width:100%;padding:7px 10px;border-radius:6px;border:1px solid var(--border);background:transparent;color:var(--muted);font-size:11px;font-family:inherit;cursor:pointer;text-align:left;margin-bottom:4px;transition:all .15s;font-weight:500}
  .week-btn:hover{background:var(--surface2);color:var(--text)}
  .week-btn.active{color:#fff;border-color:transparent;font-weight:600}
  .w1.active{background:var(--w1)}.w2.active{background:var(--w2)}.w3.active{background:var(--w3)}.w4.active{background:var(--w4)}.w5.active{background:var(--w5)}

  /* ── Main ── */
  .main{overflow-y:auto;display:flex;flex-direction:column}
  .topbar{padding:18px 24px;border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;background:var(--surface);position:sticky;top:0;z-index:10}
  .topbar-left h1{font-size:16px;font-weight:600;color:var(--text)}
  .topbar-left p{font-size:11px;color:var(--muted);margin-top:2px}
  .topbar-right{display:flex;gap:8px;align-items:center}
  .badge{padding:4px 10px;border-radius:20px;font-size:11px;font-weight:600;background:rgba(108,99,255,.15);color:var(--accent)}
  .btn-refresh{padding:6px 14px;border-radius:6px;border:1px solid var(--border);background:var(--surface2);color:var(--text);font-size:12px;cursor:pointer;font-family:inherit;transition:all .15s}
  .btn-refresh:hover{background:var(--surface3)}

  /* ── Content ── */
  .content{padding:20px 24px;flex:1}
  .section{display:none}
  .section.active{display:block}

  /* ── Metrics row ── */
  .metrics{display:grid;grid-template-columns:repeat(4,1fr);gap:12px;margin-bottom:20px}
  .metric{background:var(--surface);border:1px solid var(--border);border-radius:var(--r);padding:14px 16px}
  .metric-top{display:flex;align-items:center;justify-content:space-between;margin-bottom:10px}
  .metric-label{font-size:11px;color:var(--muted);font-weight:500}
  .metric-icon{width:28px;height:28px;border-radius:7px;display:flex;align-items:center;justify-content:center;font-size:14px}
  .metric-val{font-size:24px;font-weight:600;color:var(--text);font-family:'DM Mono',monospace;letter-spacing:-.02em}
  .metric-sub{font-size:11px;color:var(--muted);margin-top:4px}
  .metric-trend{font-size:10px;font-weight:600;margin-top:6px}
  .up{color:var(--accent2)}.down{color:var(--accent4)}

  /* ── Grids ── */
  .grid-2{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-bottom:14px}
  .grid-3{display:grid;grid-template-columns:1fr 1fr 1fr;gap:14px;margin-bottom:14px}
  .card{background:var(--surface);border:1px solid var(--border);border-radius:var(--r);padding:16px}
  .card-head{display:flex;align-items:center;justify-content:space-between;margin-bottom:14px}
  .card-title{font-size:13px;font-weight:600;color:var(--text)}
  .card-badge{font-size:10px;padding:3px 8px;border-radius:20px;font-weight:600}

  /* ── Tables ── */
  .tbl{width:100%;border-collapse:collapse}
  .tbl th{text-align:left;padding:6px 8px;font-size:10px;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.05em;border-bottom:1px solid var(--border)}
  .tbl td{padding:8px 8px;border-bottom:1px solid rgba(255,255,255,.03);font-size:12px;color:var(--text)}
  .tbl tr:last-child td{border-bottom:none}
  .tbl .total-row td{color:var(--accent);font-weight:600;background:rgba(108,99,255,.06);border-radius:4px}

  /* ── Pills ── */
  .pill{display:inline-block;padding:2px 8px;border-radius:20px;font-size:10px;font-weight:600}
  .pill-green{background:rgba(16,185,129,.15);color:#34D399}
  .pill-yellow{background:rgba(245,158,11,.15);color:#FBBF24}
  .pill-red{background:rgba(239,68,68,.15);color:#F87171}
  .pill-blue{background:rgba(59,130,246,.15);color:#60A5FA}

  /* ── Progress ── */
  .prog-wrap{display:flex;align-items:center;gap:8px}
  .prog-bg{flex:1;height:5px;background:var(--surface3);border-radius:4px;overflow:hidden}
  .prog-fill{height:5px;border-radius:4px;transition:width .5s ease}
  .prog-label{font-size:10px;color:var(--muted);min-width:28px;text-align:right;font-family:'DM Mono',monospace}

  /* ── Chart containers ── */
  .chart-box{position:relative;width:100%}
  .h200{height:200px}
  .h240{height:240px}
  .h180{height:180px}

  /* ── Overview section ── */
  .kpi-strip{display:grid;grid-template-columns:repeat(5,1fr);gap:10px;margin-bottom:16px}
  .kpi{background:var(--surface);border:1px solid var(--border);border-radius:8px;padding:12px;text-align:center}
  .kpi-w{display:block;font-size:10px;color:var(--muted);margin-bottom:4px;font-weight:600}
  .kpi-n{display:block;font-size:18px;font-weight:600;color:var(--text);font-family:'DM Mono',monospace}
  .kpi-s{display:block;font-size:10px;color:var(--muted);margin-top:2px}

  /* ── Loading ── */
  .loading{display:flex;flex-direction:column;align-items:center;justify-content:center;height:300px;gap:12px;color:var(--muted)}
  .spinner{width:32px;height:32px;border:2px solid var(--surface3);border-top-color:var(--accent);border-radius:50%;animation:spin .7s linear infinite}
  @keyframes spin{to{transform:rotate(360deg)}}
  .error-box{background:rgba(239,68,68,.1);border:1px solid rgba(239,68,68,.3);border-radius:8px;padding:16px;color:#F87171;font-size:12px}

  /* ── Avatar ── */
  .avatar{width:24px;height:24px;border-radius:50%;background:var(--accent);display:inline-flex;align-items:center;justify-content:center;font-size:9px;font-weight:700;color:#fff;margin-right:6px;vertical-align:middle;flex-shrink:0}

  /* ── B-Roll donut wrapper ── */
  .donut-wrap{display:flex;align-items:center;gap:16px}
  .donut-chart{position:relative;width:120px;height:120px;flex-shrink:0}
  .donut-list{flex:1}
  .donut-item{display:flex;align-items:center;justify-content:space-between;padding:4px 0;font-size:11px;border-bottom:1px solid var(--border)}
  .donut-item:last-child{border-bottom:none}
  .dot{width:8px;height:8px;border-radius:50%;display:inline-block;margin-right:6px;flex-shrink:0}
</style>
</head>
<body>

<div class="layout">
  <!-- ── SIDEBAR ── -->
  <aside class="sidebar">
    <div class="brand">
      <div class="brand-icon">📊</div>
      <h2>Hiệu Suất<br>Nhân Sự</h2>
      <p id="metaMonth">Đang tải...</p>
    </div>
    <nav class="nav">
      <div class="nav-label">Chuyên mục</div>
      <div class="nav-item active" data-sec="overview">
        <span class="icon">⬛</span> Tổng quan
      </div>
      <div class="nav-item" data-sec="casestudy">
        <span class="icon">📋</span> Casestudy
      </div>
      <div class="nav-item" data-sec="videoview">
        <span class="icon">🎬</span> Video & View
      </div>
      <div class="nav-item" data-sec="kichban">
        <span class="icon">📝</span> Kịch Bản
      </div>
      <div class="nav-item" data-sec="broll">
        <span class="icon">🎥</span> B-Roll
      </div>
    </nav>
    <div class="week-section">
      <div class="nav-label" style="padding:0 8px;margin-bottom:8px">Chọn tuần</div>
      <button class="week-btn w1 active" data-w="0">● Tuần 1</button>
      <button class="week-btn w2" data-w="1">● Tuần 2</button>
      <button class="week-btn w3" data-w="2">● Tuần 3</button>
      <button class="week-btn w4" data-w="3">● Tuần 4</button>
      <button class="week-btn w5" data-w="4">● Tuần 5</button>
    </div>
  </aside>

  <!-- ── MAIN ── -->
  <div class="main">
    <div class="topbar">
      <div class="topbar-left">
        <h1 id="sectionTitle">Tổng Quan</h1>
        <p id="updatedAt">Đang tải dữ liệu...</p>
      </div>
      <div class="topbar-right">
        <span class="badge" id="weekBadge">Tuần 1</span>
        <button class="btn-refresh" onclick="loadData()">↺ Làm mới</button>
      </div>
    </div>

    <div class="content">
      <!-- LOADING -->
      <div id="loadingState" class="loading">
        <div class="spinner"></div>
        <span>Đang đọc dữ liệu từ Sheets...</span>
      </div>

      <!-- ERROR -->
      <div id="errorState" style="display:none">
        <div class="error-box" id="errorMsg"></div>
      </div>

      <!-- ── OVERVIEW ── -->
      <div class="section" id="sec-overview">
        <div class="kpi-strip" id="overviewKpi"></div>
        <div class="grid-2">
          <div class="card">
            <div class="card-head"><span class="card-title">Casestudy & Video theo tuần</span></div>
            <div class="chart-box h240"><canvas id="chartOverview" role="img" aria-label="Biểu đồ casestudy và video theo tuần"></canvas></div>
          </div>
          <div class="card">
            <div class="card-head"><span class="card-title">Tổng View theo tuần</span></div>
            <div class="chart-box h240"><canvas id="chartViews" role="img" aria-label="Biểu đồ tổng view theo tuần"></canvas></div>
          </div>
        </div>
        <div class="card">
          <div class="card-head"><span class="card-title">Tiến độ Kịch Bản theo tuần</span></div>
          <div id="overviewKB"></div>
        </div>
      </div>

      <!-- ── CASESTUDY ── -->
      <div class="section" id="sec-casestudy">
        <div class="metrics" id="csMetrics"></div>
        <div class="grid-2">
          <div class="card">
            <div class="card-head">
              <span class="card-title">Chi tiết thành viên</span>
              <span class="card-badge pill pill-blue" id="csTotalBadge"></span>
            </div>
            <table class="tbl" id="csTable">
              <thead><tr><th>Thành Viên</th><th>Casestudy</th><th>Đóng góp</th><th>Trạng thái</th><th>Note</th></tr></thead>
              <tbody></tbody>
            </table>
          </div>
          <div class="card">
            <div class="card-head"><span class="card-title">Phân bổ casestudy</span></div>
            <div class="chart-box h200"><canvas id="chartCS" role="img" aria-label="Biểu đồ phân bổ casestudy"></canvas></div>
          </div>
        </div>
      </div>

      <!-- ── VIDEO & VIEW ── -->
      <div class="section" id="sec-videoview">
        <div class="metrics" id="vvMetrics"></div>
        <div class="grid-2">
          <div class="card">
            <div class="card-head"><span class="card-title">Chi tiết theo kênh</span></div>
            <table class="tbl" id="vvTable">
              <thead><tr><th>Kênh</th><th>Video</th><th>Tổng View</th><th>TB View/Video</th></tr></thead>
              <tbody></tbody>
            </table>
          </div>
          <div class="card">
            <div class="card-head"><span class="card-title">So sánh view theo kênh</span></div>
            <div class="chart-box h200"><canvas id="chartVV" role="img" aria-label="Biểu đồ so sánh view theo kênh"></canvas></div>
          </div>
        </div>
      </div>

      <!-- ── KỊCH BẢN ── -->
      <div class="section" id="sec-kichban">
        <div class="metrics" id="kbMetrics"></div>
        <div class="card" style="margin-bottom:14px">
          <div class="card-head"><span class="card-title">Tiến độ theo kênh</span></div>
          <table class="tbl" id="kbTable">
            <thead><tr><th>Kênh</th><th>Hoàn thành / Mục tiêu</th><th>% Đạt</th><th>Phân công</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
        <div class="card">
          <div class="card-head"><span class="card-title">Radar tiến độ</span></div>
          <div class="chart-box h240"><canvas id="chartKB" role="img" aria-label="Biểu đồ radar tiến độ kịch bản"></canvas></div>
        </div>
      </div>

      <!-- ── B-ROLL ── -->
      <div class="section" id="sec-broll">
        <div class="metrics" id="brMetrics"></div>
        <div class="grid-2">
          <div class="card">
            <div class="card-head"><span class="card-title">Chi tiết tài nguyên</span></div>
            <table class="tbl" id="brTable">
              <thead><tr><th>Loại tài nguyên</th><th>Số lượng</th><th>Tỉ lệ</th><th>Ghi chú</th></tr></thead>
              <tbody></tbody>
            </table>
          </div>
          <div class="card">
            <div class="card-head"><span class="card-title">Phân bổ tài nguyên</span></div>
            <div class="donut-wrap">
              <div class="donut-chart"><canvas id="chartBR" role="img" aria-label="Biểu đồ phân bổ b-roll"></canvas></div>
              <div class="donut-list" id="brLegend"></div>
            </div>
          </div>
        </div>
      </div>
    </div><!-- /content -->
  </div><!-- /main -->
</div><!-- /layout -->

<script>
// ── State ─────────────────────────────────────────────────────
let ALL = null;
let curWeek = 0;
let curSec  = 'overview';
const charts = {};

const WEEK_COLORS = ['#3B82F6','#10B981','#F59E0B','#EF4444','#8B5CF6'];
const BR_COLORS   = ['#6C63FF','#10B981','#F59E0B','#EF4444','#8B5CF6','#06B6D4'];
const SECTION_TITLES = {
  overview:'Tổng Quan', casestudy:'Casestudy (Form 4F)',
  videoview:'Video & View', kichban:'Kịch Bản', broll:'BRoll',
};

// ── Load data from Apps Script ─────────────────────────────────
function loadData() {
  showLoading(true);
  google.script.run
    .withSuccessHandler(onDataLoaded)
    .withFailureHandler(onError)
    .getAllData();
}

function onDataLoaded(data) {
  ALL = data;
  showLoading(false);
  document.getElementById('metaMonth').textContent = data.meta.month;
  document.getElementById('updatedAt').textContent  = 'Cập nhật: ' + data.meta.updatedAt;
  renderSection();
}

function onError(err) {
  showLoading(false);
  document.getElementById('errorState').style.display = 'block';
  document.getElementById('errorMsg').textContent = '⚠️ Lỗi: ' + (err.message || err);
}

function showLoading(v) {
  document.getElementById('loadingState').style.display = v ? 'flex' : 'none';
  document.getElementById('errorState').style.display   = 'none';
  document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
}

// ── Navigation ─────────────────────────────────────────────────
document.querySelectorAll('.nav-item').forEach(el => {
  el.addEventListener('click', () => {
    document.querySelectorAll('.nav-item').forEach(x => x.classList.remove('active'));
    el.classList.add('active');
    curSec = el.dataset.sec;
    document.getElementById('sectionTitle').textContent = SECTION_TITLES[curSec];
    if (ALL) renderSection();
  });
});

document.querySelectorAll('.week-btn').forEach(el => {
  el.addEventListener('click', () => {
    document.querySelectorAll('.week-btn').forEach(x => x.classList.remove('active'));
    el.classList.add('active');
    curWeek = parseInt(el.dataset.w);
    document.getElementById('weekBadge').textContent = `Tuần ${curWeek + 1}`;
    if (ALL) renderSection();
  });
});

// ── Render dispatcher ──────────────────────────────────────────
function renderSection() {
  document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
  const sec = document.getElementById('sec-' + curSec);
  if (sec) sec.classList.add('active');
  if (curSec === 'overview')   renderOverview();
  if (curSec === 'casestudy')  renderCasestudy();
  if (curSec === 'videoview')  renderVideoView();
  if (curSec === 'kichban')    renderKichBan();
  if (curSec === 'broll')      renderBRoll();
}

// ── Helpers ───────────────────────────────────────────────────
function fmtNum(n) { return n >= 1000 ? (n/1000).toFixed(1)+'K' : String(n); }
function pillStatus(pct) {
  if (pct >= 80) return 'pill-green';
  if (pct >= 60) return 'pill-yellow';
  return 'pill-red';
}
function statusLabel(pct) {
  if (pct >= 80) return 'Tốt';
  if (pct >= 60) return 'Đạt';
  return 'Cần cải thiện';
}
function destroyChart(id) { if (charts[id]) { charts[id].destroy(); delete charts[id]; } }
function initials(name) {
  return name.split(' ').slice(-2).map(w => w[0]).join('').toUpperCase();
}
function progBar(val, max, color) {
  const pct = max > 0 ? Math.min(100, Math.round(val/max*100)) : 0;
  return `<div class="prog-wrap">
    <div class="prog-bg"><div class="prog-fill" style="width:${pct}%;background:${color}"></div></div>
    <span class="prog-label">${pct}%</span>
  </div>`;
}

// ── OVERVIEW ──────────────────────────────────────────────────
function renderOverview() {
  const wColors = WEEK_COLORS;
  const labels  = ALL.casestudy.map(w => w.label);
  const csData  = ALL.casestudy.map(w => w.total);
  const vidData = ALL.videoview.map(w => w.totalVideos);
  const viewData= ALL.videoview.map(w => w.totalViews);
  const kbData  = ALL.kichban.map(w => w.totalPct);

  // KPI strip
  document.getElementById('overviewKpi').innerHTML = ALL.casestudy.map((w,i) => {
    const vv = ALL.videoview[i] || {};
    return `<div class="kpi">
      <span class="kpi-w" style="color:${wColors[i]}">${w.label}</span>
      <span class="kpi-n">${w.total}</span>
      <span class="kpi-s">${vv.totalVideos||0} video · ${fmtNum(vv.totalViews||0)} views</span>
    </div>`;
  }).join('');

  // Chart overview
  destroyChart('overview');
  charts['overview'] = new Chart(document.getElementById('chartOverview'), {
    type:'bar',
    data:{
      labels,
      datasets:[
        {label:'Casestudy',data:csData,backgroundColor:wColors.map(c=>c+'99'),borderColor:wColors,borderWidth:1.5,borderRadius:5,barPercentage:.45},
        {label:'Video',data:vidData,backgroundColor:'rgba(16,185,129,.3)',borderColor:'#10B981',borderWidth:1.5,borderRadius:5,barPercentage:.45},
      ]
    },
    options:{
      responsive:true,maintainAspectRatio:false,
      plugins:{legend:{display:false}},
      scales:{
        x:{grid:{display:false},ticks:{color:'#8B93A8',font:{size:10}}},
        y:{grid:{color:'rgba(255,255,255,.04)'},ticks:{color:'#8B93A8',font:{size:10},stepSize:2}},
      }
    }
  });

  // Chart views
  destroyChart('views');
  charts['views'] = new Chart(document.getElementById('chartViews'), {
    type:'line',
    data:{
      labels,
      datasets:[{
        label:'Tổng View',data:viewData,
        borderColor:'#6C63FF',backgroundColor:'rgba(108,99,255,.12)',
        borderWidth:2,pointBackgroundColor:'#6C63FF',pointRadius:4,fill:true,tension:.35,
      }]
    },
    options:{
      responsive:true,maintainAspectRatio:false,
      plugins:{legend:{display:false}},
      scales:{
        x:{grid:{display:false},ticks:{color:'#8B93A8',font:{size:10}}},
        y:{grid:{color:'rgba(255,255,255,.04)'},ticks:{color:'#8B93A8',font:{size:10},callback:v=>fmtNum(v)}},
      }
    }
  });

  // KB progress bars
  document.getElementById('overviewKB').innerHTML = ALL.kichban.map((w,i) => `
    <div style="display:flex;align-items:center;gap:12px;padding:10px 0;border-bottom:1px solid rgba(255,255,255,.04)">
      <span style="font-size:11px;color:${wColors[i]};font-weight:600;min-width:48px">${w.label}</span>
      <div style="flex:1">${progBar(w.totalDone,w.totalTarget,wColors[i])}</div>
      <span style="font-size:11px;color:#8B93A8;min-width:60px;text-align:right">${w.totalDone}/${w.totalTarget} KB</span>
    </div>
  `).join('');
}

// ── CASESTUDY ────────────────────────────────────────────────
function renderCasestudy() {
  const d = ALL.casestudy[curWeek] || { members:[], total:0 };
  const wc= WEEK_COLORS[curWeek];

  // Metrics
  const top = d.members.reduce((a,m) => m.count > a.count ? m : a, {name:'—',count:0});
  document.getElementById('csMetrics').innerHTML = `
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Tổng Casestudy</span><div class="metric-icon" style="background:${wc}22"><span style="color:${wc}">📋</span></div></div>
      <div class="metric-val">${d.total}</div><div class="metric-sub">case hoàn thành</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Thành Viên</span><div class="metric-icon" style="background:#6C63FF22"><span>👥</span></div></div>
      <div class="metric-val">${d.members.length}</div><div class="metric-sub">người tham gia</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Trung Bình / Người</span><div class="metric-icon" style="background:#10B98122"><span>📈</span></div></div>
      <div class="metric-val">${d.members.length ? (d.total/d.members.length).toFixed(1) : 0}</div><div class="metric-sub">case / thành viên</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Đóng góp cao nhất</span><div class="metric-icon" style="background:#F59E0B22"><span>🏆</span></div></div>
      <div class="metric-val">${top.count}</div><div class="metric-sub">${top.name}</div>
    </div>
  `;

  // Badge
  document.getElementById('csTotalBadge').textContent = `Tổng: ${d.total} case`;

  // Table
  const tbody = document.querySelector('#csTable tbody');
  tbody.innerHTML = d.members.map(m => {
    const pct = d.total > 0 ? Math.round(m.count/d.total*100) : 0;
    const st  = m.count >= 4 ? 'pill-green' : m.count >= 2 ? 'pill-yellow' : 'pill-red';
    return `<tr>
      <td><span class="avatar">${initials(m.name)}</span>${m.name}</td>
      <td style="font-family:'DM Mono',monospace;font-weight:500;color:${wc}">${m.count}</td>
      <td>${progBar(m.count,d.total,wc)}</td>
      <td><span class="pill ${st}">${statusLabel(m.count>=4?90:m.count>=2?70:30)}</span></td>
      <td style="color:#8B93A8;font-size:11px">${m.note||'—'}</td>
    </tr>`;
  }).join('');
  // Total row
  tbody.innerHTML += `<tr class="total-row"><td colspan="1">TỔNG</td><td>${d.total}</td><td colspan="3"></td></tr>`;

  // Donut chart
  destroyChart('cs');
  if (d.members.length) {
    charts['cs'] = new Chart(document.getElementById('chartCS'), {
      type:'doughnut',
      data:{
        labels:d.members.map(m=>m.name.split(' ').slice(-1)[0]),
        datasets:[{data:d.members.map(m=>m.count),backgroundColor:WEEK_COLORS,borderWidth:2,borderColor:'#161B27',hoverOffset:6}]
      },
      options:{responsive:true,maintainAspectRatio:false,cutout:'68%',plugins:{legend:{position:'right',labels:{color:'#8B93A8',font:{size:10},boxWidth:10}}}}
    });
  }
}

// ── VIDEO & VIEW ──────────────────────────────────────────────
function renderVideoView() {
  const d = ALL.videoview[curWeek] || { channels:[], totalVideos:0, totalViews:0 };
  const wc= WEEK_COLORS[curWeek];

  document.getElementById('vvMetrics').innerHTML = `
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Tổng Video</span><div class="metric-icon" style="background:${wc}22"><span>🎬</span></div></div>
      <div class="metric-val">${d.totalVideos}</div><div class="metric-sub">video đăng tải</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Tổng View</span><div class="metric-icon" style="background:#6C63FF22"><span>👁️</span></div></div>
      <div class="metric-val">${fmtNum(d.totalViews)}</div><div class="metric-sub">lượt xem</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">TB View / Video</span><div class="metric-icon" style="background:#10B98122"><span>📊</span></div></div>
      <div class="metric-val">${d.totalVideos ? fmtNum(Math.round(d.totalViews/d.totalVideos)) : 0}</div><div class="metric-sub">view trung bình</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Số kênh</span><div class="metric-icon" style="background:#F59E0B22"><span>📡</span></div></div>
      <div class="metric-val">${d.channels.length}</div><div class="metric-sub">kênh hoạt động</div>
    </div>
  `;

  const tbody = document.querySelector('#vvTable tbody');
  tbody.innerHTML = d.channels.map((c,i) => `<tr>
    <td><span class="dot" style="background:${WEEK_COLORS[i%5]}"></span>${c.name}</td>
    <td style="font-family:'DM Mono',monospace">${c.videos}</td>
    <td style="font-family:'DM Mono',monospace;color:${wc}">${c.views.toLocaleString('vi-VN')}</td>
    <td style="font-family:'DM Mono',monospace;color:#8B93A8">${c.videos ? fmtNum(Math.round(c.views/c.videos)) : '—'}</td>
  </tr>`).join('');
  tbody.innerHTML += `<tr class="total-row"><td>TỔNG</td><td>${d.totalVideos}</td><td>${d.totalViews.toLocaleString('vi-VN')}</td><td>—</td></tr>`;

  destroyChart('vv');
  charts['vv'] = new Chart(document.getElementById('chartVV'), {
    type:'bar',
    data:{
      labels:d.channels.map(c=>c.name),
      datasets:[{
        label:'View',data:d.channels.map(c=>c.views),
        backgroundColor:WEEK_COLORS.slice(0,d.channels.length).map(c=>c+'88'),
        borderColor:WEEK_COLORS,borderWidth:1.5,borderRadius:6,barPercentage:.55,
      }]
    },
    options:{
      responsive:true,maintainAspectRatio:false,
      plugins:{legend:{display:false}},
      scales:{
        x:{grid:{display:false},ticks:{color:'#8B93A8',font:{size:10}}},
        y:{grid:{color:'rgba(255,255,255,.04)'},ticks:{color:'#8B93A8',font:{size:10},callback:v=>fmtNum(v)}},
      }
    }
  });
}

// ── KỊCH BẢN ─────────────────────────────────────────────────
function renderKichBan() {
  const d = ALL.kichban[curWeek] || { channels:[], totalDone:0, totalTarget:0, totalPct:0 };
  const wc= WEEK_COLORS[curWeek];

  document.getElementById('kbMetrics').innerHTML = `
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Hoàn thành</span><div class="metric-icon" style="background:${wc}22"><span>✅</span></div></div>
      <div class="metric-val">${d.totalDone}</div><div class="metric-sub">kịch bản</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Mục tiêu</span><div class="metric-icon" style="background:#6C63FF22"><span>🎯</span></div></div>
      <div class="metric-val">${d.totalTarget}</div><div class="metric-sub">kịch bản</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">% Hoàn thành</span><div class="metric-icon" style="background:#10B98122"><span>📈</span></div></div>
      <div class="metric-val">${d.totalPct}%</div>
      <div class="metric-trend ${d.totalPct>=80?'up':'down'}">${d.totalPct>=80?'▲ Đạt mục tiêu':'▼ Chưa đạt'}</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Còn thiếu</span><div class="metric-icon" style="background:#EF444422"><span>⚠️</span></div></div>
      <div class="metric-val">${Math.max(0,d.totalTarget-d.totalDone)}</div><div class="metric-sub">kịch bản</div>
    </div>
  `;

  const tbody = document.querySelector('#kbTable tbody');
  tbody.innerHTML = d.channels.map((c,i) => `<tr>
    <td><span class="dot" style="background:${WEEK_COLORS[i%5]}"></span>${c.name}</td>
    <td>${progBar(c.done,c.target,WEEK_COLORS[i%5])}</td>
    <td><span class="pill ${pillStatus(c.pct)}">${c.pct}%</span></td>
    <td style="color:#8B93A8;font-size:11px">${c.assignee||'—'}</td>
  </tr>`).join('');
  tbody.innerHTML += `<tr class="total-row"><td>TỔNG</td><td>${progBar(d.totalDone,d.totalTarget,wc)}</td><td>${d.totalPct}%</td><td>—</td></tr>`;

  destroyChart('kb');
  if (d.channels.length) {
    charts['kb'] = new Chart(document.getElementById('chartKB'), {
      type:'radar',
      data:{
        labels:d.channels.map(c=>c.name),
        datasets:[
          {label:'Hoàn thành',data:d.channels.map(c=>c.done),borderColor:wc,backgroundColor:wc+'22',pointBackgroundColor:wc,borderWidth:2,pointRadius:4},
          {label:'Mục tiêu',data:d.channels.map(c=>c.target),borderColor:'#8B93A8',backgroundColor:'rgba(139,147,168,.08)',borderDash:[4,4],pointRadius:3,borderWidth:1.5},
        ]
      },
      options:{
        responsive:true,maintainAspectRatio:false,
        scales:{r:{angleLines:{color:'rgba(255,255,255,.06)'},grid:{color:'rgba(255,255,255,.06)'},pointLabels:{color:'#8B93A8',font:{size:10}},ticks:{color:'#8B93A8',font:{size:9},backdropColor:'transparent'}}},
        plugins:{legend:{labels:{color:'#8B93A8',font:{size:10},boxWidth:10}}},
      }
    });
  }
}

// ── B-ROLL ───────────────────────────────────────────────────
function renderBRoll() {
  const d  = ALL.broll[curWeek] || { items:[], total:0 };
  const wc = WEEK_COLORS[curWeek];

  document.getElementById('brMetrics').innerHTML = `
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Tổng tài nguyên</span><div class="metric-icon" style="background:${wc}22"><span>🎥</span></div></div>
      <div class="metric-val">${d.total}</div><div class="metric-sub">files thu thập</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Loại tài nguyên</span><div class="metric-icon" style="background:#6C63FF22"><span>🗂️</span></div></div>
      <div class="metric-val">${d.items.length}</div><div class="metric-sub">loại khác nhau</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">Nhiều nhất</span><div class="metric-icon" style="background:#10B98122"><span>⭐</span></div></div>
      <div class="metric-val">${(d.items.reduce((a,i)=>i.qty>a.qty?i:a,{qty:0,type:'—'})||{}).qty||0}</div>
      <div class="metric-sub">${(d.items.reduce((a,i)=>i.qty>a.qty?i:a,{qty:0,type:'—'})||{}).type||'—'}</div>
    </div>
    <div class="metric">
      <div class="metric-top"><span class="metric-label">TB / loại</span><div class="metric-icon" style="background:#F59E0B22"><span>📦</span></div></div>
      <div class="metric-val">${d.items.length ? Math.round(d.total/d.items.length) : 0}</div><div class="metric-sub">files / loại</div>
    </div>
  `;

  const tbody = document.querySelector('#brTable tbody');
  tbody.innerHTML = d.items.map((it,i) => {
    const pct = d.total > 0 ? Math.round(it.qty/d.total*100) : 0;
    return `<tr>
      <td><span class="dot" style="background:${BR_COLORS[i%6]}"></span>${it.type}</td>
      <td style="font-family:'DM Mono',monospace;color:${BR_COLORS[i%6]}">${it.qty}</td>
      <td>${progBar(it.qty,d.total,BR_COLORS[i%6])}</td>
      <td style="color:#8B93A8;font-size:11px">${it.note||'—'}</td>
    </tr>`;
  }).join('');
  tbody.innerHTML += `<tr class="total-row"><td>TỔNG</td><td>${d.total}</td><td colspan="2"></td></tr>`;

  destroyChart('br');
  if (d.items.length) {
    charts['br'] = new Chart(document.getElementById('chartBR'), {
      type:'doughnut',
      data:{
        labels:d.items.map(i=>i.type),
        datasets:[{data:d.items.map(i=>i.qty),backgroundColor:BR_COLORS,borderWidth:2,borderColor:'#161B27',hoverOffset:4}]
      },
      options:{responsive:true,maintainAspectRatio:false,cutout:'60%',plugins:{legend:{display:false}}}
    });
    document.getElementById('brLegend').innerHTML = d.items.map((it,i) => `
      <div class="donut-item">
        <span><span class="dot" style="background:${BR_COLORS[i%6]}"></span>${it.type}</span>
        <span style="font-family:'DM Mono',monospace;font-size:11px;color:#8B93A8">${it.qty}</span>
      </div>
    `).join('');
  }
}

// ── Boot ─────────────────────────────────────────────────────
loadData();
</script>
</body>
</html>
