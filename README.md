<!DOCTYPE html>
<html lang="pt">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>MSFEUP</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=JetBrains+Mono:wght@400;600&display=swap">
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/plotly.js/2.27.1/plotly.min.js"></script>
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
  --bg:#0D0F14; --bg2:#131720; --bg3:#1A1F2E; --bg4:#0F1119;
  --border:#1E2330; --border2:#252B3B;
  --text:#C8CDD8; --muted:#5A6070; --accent:#009A00; --danger:#FF4560;
  --font:'Inter',sans-serif; --mono:'JetBrains Mono',monospace;
}
html,body { height:100%; background:var(--bg); color:var(--text); font-family:var(--font); font-size:13px; overflow:hidden; }

/* ── LAYOUT ── */
.shell { display:grid; grid-template-columns:270px 1fr; grid-template-rows:44px 1fr; height:100vh; overflow:hidden; }

header {
  grid-column:1/-1; display:flex; align-items:center; gap:14px;
  padding:0 18px; background:var(--bg2); border-bottom:1px solid var(--border);
}
.logo { font-family:var(--mono); font-weight:600; font-size:13px; color:var(--accent); letter-spacing:.04em; white-space:nowrap; }
.sep { width:1px; height:18px; background:var(--border); flex-shrink:0; }
.run-badges { display:flex; gap:6px; flex-wrap:wrap; flex:1; min-width:0; }
.run-badge { display:flex; align-items:center; gap:6px; padding:3px 8px; border-radius:3px; background:var(--bg3); border:1px solid var(--border); font-family:var(--mono); font-size:11px; white-space:nowrap; }
.run-badge .dot { width:8px; height:8px; border-radius:50%; flex-shrink:0; }
.run-badge .rm { cursor:pointer; color:var(--muted); margin-left:4px; }
.run-badge .rm:hover { color:var(--danger); }

/* header right controls */
.hdr-right { display:flex; gap:8px; align-items:center; flex-shrink:0; }
.hdr-btn { display:flex; align-items:center; gap:5px; padding:4px 10px; border-radius:3px; border:1px solid var(--border2); background:var(--bg3); color:var(--muted); font-size:11px; font-family:var(--mono); cursor:pointer; transition:border-color .15s,color .15s; white-space:nowrap; }
.hdr-btn:hover { border-color:var(--accent); color:var(--accent); }
.hdr-btn.active { border-color:var(--accent); color:var(--accent); background:rgba(232,255,0,.06); }

/* ── SIDEBAR ── */
aside { background:var(--bg2); border-right:1px solid var(--border); display:flex; flex-direction:column; overflow:hidden; }
.sidebar-section { padding:12px 14px; border-bottom:1px solid var(--border); flex-shrink:0; }
.sidebar-label { font-size:10px; color:var(--muted); letter-spacing:.06em; text-transform:uppercase; margin-bottom:8px; }

.dropzone {
  border:1px dashed var(--border); border-radius:4px; padding:14px 12px;
  text-align:center; cursor:pointer; transition:border-color .2s,background .2s;
  color:var(--muted); font-size:12px; line-height:1.5; display:block;
}
.dropzone:hover,.dropzone.drag { border-color:var(--accent); background:rgba(232,255,0,.04); color:var(--text); }
.dropzone input { display:none; }
.dropzone .icon { font-size:18px; margin-bottom:4px; }

/* overlay mode toggle */
.mode-row { display:flex; gap:6px; margin-bottom:2px; }
.mode-btn { flex:1; text-align:center; padding:5px 0; border-radius:3px; border:1px solid var(--border2); background:var(--bg3); color:var(--muted); font-size:11px; font-family:var(--mono); cursor:pointer; transition:all .15s; }
.mode-btn.active { border-color:var(--accent); color:var(--accent); background:rgba(232,255,0,.06); }

/* signal list */
.signal-list { flex:1; overflow-y:auto; padding:4px 0; min-height:0; }
.signal-list::-webkit-scrollbar { width:3px; }
.signal-list::-webkit-scrollbar-thumb { background:var(--border2); border-radius:2px; }

.sig-group { padding:6px 14px 2px; font-size:9px; color:var(--muted); letter-spacing:.08em; text-transform:uppercase; }
.sig-item { display:flex; align-items:center; gap:8px; padding:5px 14px; cursor:pointer; transition:background .12s; user-select:none; }
.sig-item:hover { background:var(--bg3); }
.sig-item.active { background:rgba(232,255,0,.05); }
.sig-swatch { width:12px; height:12px; border-radius:2px; flex-shrink:0; border:1px solid rgba(255,255,255,.1); transition:opacity .15s; }
.sig-item:not(.active) .sig-swatch { opacity:.3; }
.sig-name { font-size:11px; flex:1; }
.sig-unit { font-family:var(--mono); font-size:10px; color:var(--muted); }

/* ── MAIN ── */
main { overflow-y:auto; overflow-x:hidden; padding:12px; display:flex; flex-direction:column; gap:10px; background:var(--bg); min-height:0; }
main::-webkit-scrollbar { width:3px; }
main::-webkit-scrollbar-thumb { background:var(--border2); border-radius:2px; }

/* stats */
.stats-bar { display:grid; grid-template-columns:repeat(auto-fill,minmax(150px,1fr)); gap:8px; }
.stat-card { background:var(--bg2); border:1px solid var(--border); border-radius:4px; padding:10px 12px; }
.stat-label { font-size:10px; color:var(--muted); margin-bottom:5px; }
.stat-val { font-family:var(--mono); font-size:15px; font-weight:600; display:flex; align-items:baseline; gap:4px; }
.stat-val .unit { font-size:10px; color:var(--muted); font-weight:400; }
.stat-val .run-dot { width:6px; height:6px; border-radius:50%; flex-shrink:0; margin-right:1px; }

/* chart panels */
.chart-panel { background:var(--bg2); border:1px solid var(--border); border-radius:4px; overflow:hidden; }
.chart-header { display:flex; align-items:center; gap:8px; padding:7px 12px; border-bottom:1px solid var(--border); }
.chart-title { font-size:11px; font-weight:500; flex:1; }
.chart-units { display:flex; gap:6px; flex-wrap:wrap; }
.chart-unit-tag { font-family:var(--mono); font-size:10px; padding:1px 5px; border-radius:2px; }

/* zoom toolbar */
.chart-toolbar { display:flex; gap:4px; }
.tb-btn { width:22px; height:22px; display:flex; align-items:center; justify-content:center; border-radius:3px; border:1px solid var(--border2); background:var(--bg3); color:var(--muted); font-size:13px; cursor:pointer; transition:border-color .15s,color .15s; line-height:1; }
.tb-btn:hover { border-color:var(--accent); color:var(--accent); }

.chart-body { position:relative; }
.chart-body.h-sm { height:180px; }
.chart-body.h-md { height:240px; }
.chart-body.h-lg { height:320px; }

/* overlay chart gets taller */
.overlay-panel .chart-body { height:320px; }

/* empty */
.empty { display:flex; flex-direction:column; align-items:center; justify-content:center; height:100%; gap:10px; color:var(--muted); text-align:center; padding:40px; }
.empty .big { font-size:30px; }

/* loading */
.loading { display:none; position:fixed; inset:0; background:rgba(13,15,20,.88); align-items:center; justify-content:center; z-index:200; flex-direction:column; gap:12px; font-family:var(--mono); color:var(--accent); font-size:12px; }
.loading.show { display:flex; }
.spinner { width:26px; height:26px; border:2px solid var(--border); border-top-color:var(--accent); border-radius:50%; animation:spin .7s linear infinite; }
@keyframes spin { to { transform:rotate(360deg); } }

/* info pill in chart legend */
.legend-pill { display:inline-flex; align-items:center; gap:4px; padding:2px 6px; border-radius:2px; font-family:var(--mono); font-size:10px; background:rgba(0,0,0,.3); margin-top:2px; }
</style>
</head>
<body>
<div class="shell">

  <!-- HEADER -->
  <header>
    <span class="logo">MOTO<span style="color:var(--text);font-weight:300">STUDENT</span> ·telemetria</span>
    <div class="sep"></div>
    <div class="run-badges" id="runBadges"><span style="color:var(--muted);font-size:11px">nenhuma run carregada</span></div>
    <div class="hdr-right">
      <button class="hdr-btn" id="btnResetZoom" onclick="resetAllZoom()" title="Reset zoom de todos os gráficos">⤢ Reset Zoom</button>
      <button class="hdr-btn" id="btnSyncZoom" onclick="toggleSyncZoom()" title="Sincronizar zoom entre gráficos">⇄ Sync</button>
    </div>
  </header>

  <!-- SIDEBAR -->
  <aside>
    <div class="sidebar-section">
      <div class="sidebar-label">Ficheiros</div>
      <label class="dropzone" id="dropzone">
        <input type="file" id="fileInput" accept=".csv" multiple>
        <div class="icon">⊕</div>
        Arrasta CSVs aqui<br>ou clica para escolher
      </label>
    </div>

    <div class="sidebar-section">
      <div class="sidebar-label">Modo de visualização</div>
      <div class="mode-row">
        <div class="mode-btn active" id="modeSeparate" onclick="setMode('separate')">Separado</div>
        <div class="mode-btn" id="modeOverlay" onclick="setMode('overlay')">Overlay</div>
      </div>
    </div>

    <div class="sidebar-section" style="padding-bottom:6px">
      <div class="sidebar-label">Sinais</div>
    </div>
    <div class="signal-list" id="signalList"></div>
  </aside>

  <!-- MAIN -->
  <main id="main">
    <div class="empty" id="emptyState">
      <div class="big">◈</div>
      <p>Carrega um ou mais ficheiros CSV exportados pelo Kvaser.<br>Seleciona os sinais na lista à esquerda.<br>Em modo <strong>Overlay</strong> todos os sinais ficam num único gráfico.</p>
    </div>
  </main>
</div>

<div class="loading" id="loading">
  <div class="spinner"></div>
  <span id="loadingText">A processar...</span>
</div>

<script>
// ── CONFIG ──────────────────────────────────────────────────────────────────
const THROTTLE_V_MAX = 2.0;
const RUN_COLORS = ['#E8FF00','#00D4FF','#FF4560','#00E396','#FF9800','#AB47BC'];

const SIGNAL_GROUPS = [
  { group:'Velocidade & Acelerador', signals:[
    { key:'Actual_Velocity_RPM', label:'Velocidade Motor', unit:'RPM', vi:13, color:'#E8FF00' },
    { key:'Max_Velocity_RPM',    label:'Max Velocity',     unit:'RPM', vi:7,  color:'#60A5FA' },
    { key:'Throttle_Perc',       label:'Acelerador',       unit:'%',   vi:40, color:'#FF00FF', isThrottle:true },
  ]},
  { group:'Correntes', signals:[
    { key:'Iq_A',          label:'Iq (Torque)',   unit:'A', vi:52, color:'#00E396' },
    { key:'Target_Iq_A',   label:'Target Iq',     unit:'A', vi:46, color:'#C084FC' },
    { key:'Id_A',          label:'Id (Fluxo)',     unit:'A', vi:4,  color:'#A78BFA' },
    { key:'Target_Id_A',   label:'Target Id',     unit:'A', vi:1,  color:'#7B61FF' },
    { key:'Battery_Current_A', label:'Corrente Bateria', unit:'A', vi:10, color:'#F472B6' },
  ]},
  { group:'Torque', signals:[
    { key:'Torque_Demand', label:'Torque Demand', unit:'Nm', vi:37, color:'#FBBF24' },
    { key:'Torque_Actual', label:'Torque Actual', unit:'Nm', vi:16, color:'#FB923C' },
    { key:'Max_Power_Torque', label:'Max Power Torque', unit:'Nm', vi:22, color:'#F97316' },
  ]},
  { group:'Tensões', signals:[
    { key:'Bus_Voltage_V',       label:'Tensão Barramento', unit:'V', vi:43, color:'#00D4FF' },
    { key:'Capacitor_Voltage_V', label:'Tensão Lookup',     unit:'V', vi:25, color:'#06B6D4' },
    { key:'Uq_V',                label:'Uq',                unit:'V', vi:31, color:'#34D399' },
    { key:'Ud_V',                label:'Ud',                unit:'V', vi:49, color:'#4ADE80' },
    { key:'Voltage_Modulation',  label:'Modulação Tensão',  unit:'%', vi:34, color:'#A3E635' },
  ]},
  { group:'Temperatura', signals:[
    { key:'Motor_Temp_C',    label:'Temp. Motor',    unit:'°C', vi:19, color:'#FF4560' },
    { key:'Heatsink_Temp_C', label:'Temp. Heatsink', unit:'°C', vi:28, color:'#EF4444' },
  ]},
];

const SIGNAL_MAP = SIGNAL_GROUPS.flatMap(g => g.signals);
const SIG_BY_KEY = Object.fromEntries(SIGNAL_MAP.map(s => [s.key, s]));

const DEFAULT_ACTIVE = new Set(['Actual_Velocity_RPM','Throttle_Perc','Iq_A','Motor_Temp_C','Torque_Actual','Bus_Voltage_V']);

let runs = [];
let activeSignals = new Set(DEFAULT_ACTIVE);
let viewMode = 'separate'; // 'separate' | 'overlay'
let syncZoom = true;
let zoomLock = false; // prevent re-entrant relayout events

// ── PARSE ────────────────────────────────────────────────────────────────────
function parseKvaserCSV(text, fileName) {
  const result = Papa.parse(text, { skipEmptyLines: false });
  const rows = result.data;
  if (rows.length < 2) return null;

  const data = {};
  for (const sig of SIGNAL_MAP) {
    const ti = sig.vi - 1;
    const vi = sig.vi;
    const xArr = [], yArr = [];
    for (let r = 1; r < rows.length; r++) {
      const row = rows[r];
      const t = parseFloat(row[ti]);
      let v = parseFloat(row[vi]);
      if (isNaN(t) || isNaN(v)) continue;
      if (sig.isThrottle) v = Math.min(100, Math.max(0, (v / THROTTLE_V_MAX) * 100));
      if (sig.key === 'Iq_A' && Math.abs(v) < 4) v = 0;
      xArr.push(t); yArr.push(v);
    }
    data[sig.key] = { x: xArr, y: yArr };
  }

  // Stats
  const vel  = data['Actual_Velocity_RPM'].y;
  const temp = data['Motor_Temp_C'].y;
  const iq   = data['Iq_A'].y;
  const ibat = data['Battery_Current_A'].y;
  const tArr = data['Actual_Velocity_RPM'].x;
  const tMin = tArr.length ? Math.min(...tArr) : 0;
  const tMax = tArr.length ? Math.max(...tArr) : 0;
  const duracao = tMax - tMin;

  const thrX = data['Throttle_Perc'].x, thrY = data['Throttle_Perc'].y;
  let throttle80 = 0;
  for (let i = 1; i < thrX.length; i++)
    if (thrY[i-1] > 80) throttle80 += thrX[i] - thrX[i-1];

  const vcapX = data['Bus_Voltage_V'].x, vcapY = data['Bus_Voltage_V'].y;
  const ibatX = data['Battery_Current_A'].x, ibatY = data['Battery_Current_A'].y;
  let energiaWh = 0;
  for (let i = 1; i < vcapX.length; i++) {
    const nearest = ibatX.reduce((best, tx, j) => Math.abs(tx - vcapX[i]) < Math.abs(ibatX[best] - vcapX[i]) ? j : best, 0);
    energiaWh += vcapY[i] * ibatY[nearest] * (vcapX[i] - vcapX[i-1]);
  }
  energiaWh = Math.abs(energiaWh) / 3600;

  return {
    name: fileName.replace(/\.csv$/i,''),
    data,
    stats: {
      duracao,
      velMax:   vel.length  ? Math.max(...vel)  : 0,
      tempMax:  temp.length ? Math.max(...temp) : 0,
      iqPico:   iq.length   ? Math.max(...iq.map(Math.abs))   : 0,
      ibatPico: ibat.length ? Math.max(...ibat.map(Math.abs)) : 0,
      energiaWh,
      throttle80,
      throttle80Perc: duracao > 0 ? throttle80/duracao*100 : 0,
    }
  };
}

// ── PLOTLY BASE ──────────────────────────────────────────────────────────────
function baseLayout(extra = {}) {
  return {
    paper_bgcolor: 'transparent',
    plot_bgcolor:  '#0D0F14',
    margin: { t:6, r:16, b:36, l:52 },
    font:   { family:'JetBrains Mono', size:10, color:'#5A6070' },
    xaxis: {
      color:'#5A6070', gridcolor:'#1A1F2E', zerolinecolor:'#1A1F2E',
      tickfont:{ size:10 }, autorange:true, title:{ text:'s', font:{ size:9 } }
    },
    yaxis: {
      color:'#5A6070', gridcolor:'#1A1F2E', zerolinecolor:'#1A1F2E',
      tickfont:{ size:10 }, autorange:true
    },
    legend: { font:{ size:10 }, bgcolor:'rgba(13,15,20,.7)', bordercolor:'#1E2330', borderwidth:1, x:0, y:1 },
    hovermode: 'x unified',
    hoverlabel: { bgcolor:'#1A1F2E', bordercolor:'#252B3B', font:{ family:'JetBrains Mono', size:11, color:'#C8CDD8' } },
    dragmode: 'zoom',
    selectdirection: 'h',
    ...extra
  };
}

const PLOTLY_CONFIG = {
  displayModeBar: true,
  modeBarButtonsToRemove: ['toImage','sendDataToCloud','editInChartStudio','select2d','lasso2d'],
  modeBarButtonsToAdd: [],
  displaylogo: false,
  responsive: true,
  scrollZoom: true,
};

// ── ZOOM SYNC ────────────────────────────────────────────────────────────────
function getPlotIds() {
  return [...document.querySelectorAll('[id^="plot-"]')].map(el => el.id);
}

function syncZoomToAll(sourceId, xRange) {
  if (!syncZoom || zoomLock) return;
  zoomLock = true;
  for (const id of getPlotIds()) {
    if (id === sourceId) continue;
    const el = document.getElementById(id);
    if (!el || !el._fullLayout) continue;
    Plotly.relayout(el, { 'xaxis.range': xRange, 'xaxis.autorange': false });
  }
  zoomLock = false;
}

function attachZoomListener(divId) {
  const el = document.getElementById(divId);
  if (!el) return;
  el.on('plotly_relayout', (ev) => {
    if (zoomLock) return;
    const x0 = ev['xaxis.range[0]'], x1 = ev['xaxis.range[1]'];
    if (x0 !== undefined && x1 !== undefined) syncZoomToAll(divId, [x0, x1]);
    if (ev['xaxis.autorange']) syncZoomToAll(divId, null);
  });
}

function resetAllZoom() {
  for (const id of getPlotIds()) {
    const el = document.getElementById(id);
    if (el && el._fullLayout) Plotly.relayout(el, { 'xaxis.autorange': true, 'yaxis.autorange': true });
  }
}

function toggleSyncZoom() {
  syncZoom = !syncZoom;
  document.getElementById('btnSyncZoom').classList.toggle('active', syncZoom);
}
document.getElementById('btnSyncZoom').classList.add('active');

// ── MODE ─────────────────────────────────────────────────────────────────────
function setMode(m) {
  viewMode = m;
  document.getElementById('modeSeparate').classList.toggle('active', m === 'separate');
  document.getElementById('modeOverlay').classList.toggle('active', m === 'overlay');
  rebuildCharts();
}

// ── RENDER ───────────────────────────────────────────────────────────────────
function renderAll() {
  renderBadges();
  renderStats();
  rebuildCharts();
}

function renderBadges() {
  const el = document.getElementById('runBadges');
  if (!runs.length) {
    el.innerHTML = '<span style="color:var(--muted);font-size:11px">nenhuma run carregada</span>';
    return;
  }
  el.innerHTML = runs.map((r,i) => `
    <div class="run-badge">
      <div class="dot" style="background:${r.color}"></div>
      <span>${r.name}</span>
      <span class="rm" onclick="removeRun(${i})">✕</span>
    </div>`).join('');
}

function renderStats() {
  const main = document.getElementById('main');
  document.getElementById('emptyState')?.remove();

  let bar = document.getElementById('statsBar');
  if (!runs.length) {
    bar?.remove();
    main.innerHTML = `<div class="empty" id="emptyState"><div class="big">◈</div><p>Carrega um ou mais ficheiros CSV exportados pelo Kvaser.<br>Depois selecciona os sinais que queres comparar.</p></div>`;
    return;
  }
  if (!bar) {
    bar = document.createElement('div');
    bar.id = 'statsBar'; bar.className = 'stats-bar';
    main.prepend(bar);
  }

  const defs = [
    { key:'duracao',         label:'Duração',            unit:'s',  fmt:v=>v.toFixed(1) },
    { key:'velMax',          label:'Vel. Máxima',         unit:'RPM',fmt:v=>Math.round(v) },
    { key:'tempMax',         label:'Temp. Motor Máx.',    unit:'°C', fmt:v=>v.toFixed(1) },
    { key:'iqPico',          label:'Iq Pico',             unit:'A',  fmt:v=>v.toFixed(1) },
    { key:'ibatPico',        label:'Corrente Bat. Pico',  unit:'A',  fmt:v=>v.toFixed(1) },
    { key:'energiaWh',       label:'Energia',             unit:'Wh', fmt:v=>v.toFixed(2) },
    { key:'throttle80Perc',  label:'Throttle >80%',       unit:'%',  fmt:v=>v.toFixed(1) },
  ];
  bar.innerHTML = defs.map(s => `
    <div class="stat-card">
      <div class="stat-label">${s.label}</div>
      ${runs.map(r => `
        <div class="stat-val">
          <div class="run-dot" style="background:${r.color}"></div>
          ${s.fmt(r.stats[s.key])}<span class="unit">${s.unit}</span>
        </div>`).join('')}
    </div>`).join('');
}

// ── CHART BUILDING ───────────────────────────────────────────────────────────
function purgeAllPlots() {
  for (const id of getPlotIds()) {
    const el = document.getElementById(id);
    if (el) Plotly.purge(el);
  }
  // Remove all chart panels
  document.querySelectorAll('.chart-panel').forEach(p => p.remove());
}

function rebuildCharts() {
  if (!runs.length) return;
  purgeAllPlots();

  if (viewMode === 'overlay') {
    buildOverlayChart();
  } else {
    buildSeparateCharts();
  }
}

/* ── SEPARATE MODE ── */
function buildSeparateCharts() {
  const main = document.getElementById('main');
  const activeSigs = SIGNAL_MAP.filter(s => activeSignals.has(s.key));

  for (const sig of activeSigs) {
    const panel = document.createElement('div');
    panel.className = 'chart-panel';
    panel.id = `panel-${sig.key}`;
    panel.innerHTML = `
      <div class="chart-header">
        <div class="sig-swatch" style="background:${sig.color};border:none;width:10px;height:10px;border-radius:2px;flex-shrink:0"></div>
        <span class="chart-title">${sig.label}</span>
        <div class="chart-units">
          <span class="chart-unit-tag" style="background:rgba(232,255,0,.08);color:var(--accent)">${sig.unit}</span>
        </div>
        <div class="chart-toolbar">
          <div class="tb-btn" onclick="zoomIn('plot-${sig.key}')" title="Zoom In">+</div>
          <div class="tb-btn" onclick="zoomOut('plot-${sig.key}')" title="Zoom Out">−</div>
          <div class="tb-btn" onclick="resetZoom('plot-${sig.key}')" title="Reset Zoom">⤢</div>
        </div>
      </div>
      <div class="chart-body h-sm" id="plot-${sig.key}"></div>`;
    main.appendChild(panel);

    const traces = runs.map(r => ({
      x: r.data[sig.key].x,
      y: r.data[sig.key].y,
      name: r.name,
      type: 'scatter', mode: 'lines',
      line: { color: runs.length === 1 ? sig.color : r.color, width: 1.6 },
      hovertemplate: `%{y:.3f} ${sig.unit}<extra>${r.name}</extra>`
    }));

    const layout = baseLayout({
      showlegend: runs.length > 1,
      yaxis: { ...baseLayout().yaxis, title:{ text: sig.unit, font:{ size:9 } } }
    });

    Plotly.newPlot(`plot-${sig.key}`, traces, layout, PLOTLY_CONFIG);
    attachZoomListener(`plot-${sig.key}`);
  }
}

/* ── OVERLAY MODE ── */
function buildOverlayChart() {
  const main = document.getElementById('main');
  const activeSigs = SIGNAL_MAP.filter(s => activeSignals.has(s.key));
  if (!activeSigs.length) return;

  // Group signals by unit to assign shared y-axes
  const unitAxes = {}; // unit -> yaxis index (1-based)
  let axisCount = 0;
  for (const sig of activeSigs) {
    if (!(sig.unit in unitAxes)) {
      axisCount++;
      unitAxes[sig.unit] = axisCount;
    }
  }

  // Build unit tags for header
  const unitColors = {};
  for (const sig of activeSigs) unitColors[sig.unit] = unitColors[sig.unit] || sig.color;
  const unitTags = Object.entries(unitColors).map(([u, c]) =>
    `<span class="chart-unit-tag" style="background:rgba(0,0,0,.3);color:${c};border:1px solid ${c}40">${u}</span>`
  ).join('');

  const panel = document.createElement('div');
  panel.className = 'chart-panel overlay-panel';
  panel.id = 'panel-overlay';
  panel.innerHTML = `
    <div class="chart-header">
      <span class="chart-title">Overlay — ${activeSigs.length} sinais</span>
      <div class="chart-units">${unitTags}</div>
      <div class="chart-toolbar">
        <div class="tb-btn" onclick="zoomIn('plot-overlay')" title="Zoom In">+</div>
        <div class="tb-btn" onclick="zoomOut('plot-overlay')" title="Zoom Out">−</div>
        <div class="tb-btn" onclick="resetZoom('plot-overlay')" title="Reset Zoom">⤢</div>
      </div>
    </div>
    <div class="chart-body" id="plot-overlay"></div>`;
  main.appendChild(panel);

  const traces = [];
  for (const sig of activeSigs) {
    const axIdx = unitAxes[sig.unit];
    const yAxisKey = axIdx === 1 ? 'y' : `y${axIdx}`;
    for (const r of runs) {
      traces.push({
        x: r.data[sig.key].x,
        y: r.data[sig.key].y,
        name: runs.length > 1 ? `${sig.label} [${r.name}]` : sig.label,
        type: 'scatter', mode: 'lines',
        yaxis: yAxisKey,
        line: { color: sig.color, width: 1.6, dash: runs.length > 1 && runs.indexOf(r) > 0 ? 'dash' : 'solid' },
        hovertemplate: `%{y:.3f} ${sig.unit}<extra>${runs.length > 1 ? r.name + ' · ' : ''}${sig.label}</extra>`
      });
    }
  }

  // Build layout with multiple y-axes
  const layout = baseLayout({
    showlegend: true,
    legend: { ...baseLayout().legend, orientation: 'h', y: -0.15, x: 0 },
    margin: { t:6, r: Math.max(16, (axisCount - 1) * 52), b:80, l:52 },
  });

  // Assign y-axes
  for (const [unit, idx] of Object.entries(unitAxes)) {
    const sig = activeSigs.find(s => s.unit === unit);
    const axisName = idx === 1 ? 'yaxis' : `yaxis${idx}`;
    const side = (idx % 2 === 0) ? 'right' : 'left';
    const offset = idx > 2 ? (Math.floor((idx-1)/2)) * 50 : 0;
    layout[axisName] = {
      color: sig.color,
      gridcolor: idx === 1 ? '#1A1F2E' : 'transparent',
      zerolinecolor: '#1A1F2E',
      tickfont: { size:10, color: sig.color },
      title: { text: unit, font:{ size:9, color: sig.color } },
      autorange: true,
      overlaying: idx > 1 ? 'y' : undefined,
      side: side,
      position: side === 'right' ? 1 - (Math.floor((idx-2)/2)) * 0.08 : undefined,
      showgrid: idx === 1,
    };
    if (idx > 1) layout[axisName].anchor = 'free';
  }

  Plotly.newPlot('plot-overlay', traces, layout, PLOTLY_CONFIG);
  attachZoomListener('plot-overlay');
}

// ── ZOOM HELPERS ─────────────────────────────────────────────────────────────
function zoomIn(divId) {
  const el = document.getElementById(divId);
  if (!el || !el._fullLayout) return;
  const xr = el._fullLayout.xaxis.range;
  const mid = (xr[0]+xr[1])/2, span = (xr[1]-xr[0]) * 0.35;
  Plotly.relayout(el, { 'xaxis.range': [mid-span, mid+span], 'xaxis.autorange': false });
}
function zoomOut(divId) {
  const el = document.getElementById(divId);
  if (!el || !el._fullLayout) return;
  const xr = el._fullLayout.xaxis.range;
  const mid = (xr[0]+xr[1])/2, span = (xr[1]-xr[0]) * 0.75;
  Plotly.relayout(el, { 'xaxis.range': [mid-span, mid+span], 'xaxis.autorange': false });
}
function resetZoom(divId) {
  const el = document.getElementById(divId);
  if (!el) return;
  Plotly.relayout(el, { 'xaxis.autorange': true, 'yaxis.autorange': true });
}

// ── SIGNALS ──────────────────────────────────────────────────────────────────
function toggleSignal(key) {
  if (activeSignals.has(key)) activeSignals.delete(key);
  else activeSignals.add(key);
  renderSignalList();
  if (runs.length) rebuildCharts();
}

function renderSignalList() {
  const list = document.getElementById('signalList');
  list.innerHTML = SIGNAL_GROUPS.map(g => `
    <div class="sig-group">${g.group}</div>
    ${g.signals.map(sig => `
      <div class="sig-item ${activeSignals.has(sig.key)?'active':''}" onclick="toggleSignal('${sig.key}')">
        <div class="sig-swatch" style="background:${sig.color}"></div>
        <span class="sig-name">${sig.label}</span>
        <span class="sig-unit">${sig.unit}</span>
      </div>`).join('')}
  `).join('');
}

// ── RUN MANAGEMENT ───────────────────────────────────────────────────────────
function removeRun(i) {
  runs.splice(i,1);
  runs.forEach((r,j) => r.color = RUN_COLORS[j % RUN_COLORS.length]);
  renderAll();
  renderSignalList();
}

async function loadFiles(files) {
  const loading = document.getElementById('loading');
  const loadingText = document.getElementById('loadingText');
  loading.classList.add('show');
  for (const file of files) {
    loadingText.textContent = `A processar ${file.name}…`;
    const text = await file.text();
    const run = parseKvaserCSV(text, file.name);
    if (run) { run.color = RUN_COLORS[runs.length % RUN_COLORS.length]; runs.push(run); }
    await new Promise(r => setTimeout(r, 0));
  }
  loading.classList.remove('show');
  renderSignalList();
  renderAll();
}

// ── FILE INPUT ────────────────────────────────────────────────────────────────
document.getElementById('fileInput').addEventListener('change', e => {
  if (e.target.files.length) loadFiles(Array.from(e.target.files));
  e.target.value = '';
});
const dz = document.getElementById('dropzone');
dz.addEventListener('dragover', e => { e.preventDefault(); dz.classList.add('drag'); });
dz.addEventListener('dragleave', () => dz.classList.remove('drag'));
dz.addEventListener('drop', e => {
  e.preventDefault(); dz.classList.remove('drag');
  const files = Array.from(e.dataTransfer.files).filter(f => f.name.endsWith('.csv'));
  if (files.length) loadFiles(files);
});

// ── INIT ──────────────────────────────────────────────────────────────────────
renderSignalList();
</script>
</body>
</html>
