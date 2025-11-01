<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Daily Expenses Tracker — Multi-page</title>

  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <!-- CryptoJS -->
  <script src="https://cdn.jsdelivr.net/npm/crypto-js@4.1.1/crypto-js.min.js"></script>

  <style>
    :root{
      --bg1:#021026; --bg2:#0b1020; --accent1:#ffb86b; --accent2:#7c5cff; --glass: rgba(255,255,255,0.06);
      --muted: rgba(255,255,255,0.78);
      font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    html,body{height:100%;margin:0;background:
      radial-gradient(circle at 10% 10%, rgba(255,255,255,0.02), transparent 12%),
      linear-gradient(180deg,var(--bg1),var(--bg2));
      color:white;}
    .container{max-width:1150px;margin:18px auto;padding:18px;}

    /* Top nav */
    .topbar{display:flex;justify-content:space-between;align-items:center;gap:12px}
    .brand{font-weight:800;font-size:20px}
    .top-controls{display:flex;gap:8px;align-items:center}

    /* login - classic with 3D image */
    .login-wrap{display:grid;grid-template-columns:540px 1fr;gap:18px;margin-top:18px}
    .login-card{padding:26px;border-radius:16px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(0,0,0,0.12));border:1px solid rgba(255,255,255,0.04);box-shadow:0 18px 50px rgba(0,0,0,0.6)}
    .login-card h1{margin:0 0 8px;font-size:28px}
    .muted{color:var(--muted);font-size:13px}
    input,select,button,textarea{font-size:14px;padding:10px;border-radius:10px;border:none;background:var(--glass);color:white}
    button.primary{background:linear-gradient(90deg,var(--accent1),var(--accent2));color:#071124;font-weight:700;cursor:pointer}
    .field{margin:10px 0}
    /* 3D image block */
    .image-block{position:relative;padding:12px;border-radius:16px;overflow:visible}
    .layer{position:absolute;border-radius:18px;box-shadow:0 22px 60px rgba(0,0,0,0.6)}
    .layer.a{width:94%;height:260px;background:linear-gradient(90deg,#ffb199,#ffdfba);left:-8%;transform:rotate(-7deg);top:6%}
    .layer.b{width:88%;height:260px;background:linear-gradient(90deg,#8ec5ff,#e0c3fc);left:6%;transform:rotate(7deg);top:10%}
    .layer.c{width:82%;height:260px;background-position:center;background-size:cover;z-index:6;border:1px solid rgba(255,255,255,0.04);left:14%;top:16%}
    .caption{position:relative;z-index:8;margin-top:220px;color:var(--muted)}

    /* main app layout */
    .main-grid{display:grid;grid-template-columns:320px 1fr;gap:18px;margin-top:18px}
    .panel{padding:14px;border-radius:12px;background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);border:1px solid rgba(255,255,255,0.03)}
    .flex-row{display:flex;gap:8px;align-items:center}
    .chips{display:flex;gap:8px;flex-wrap:wrap}
    .chip{padding:8px 10px;border-radius:999px;background:rgba(255,255,255,0.03);cursor:pointer}
    table{width:100%;border-collapse:collapse}
    th,td{padding:8px;text-align:left;border-bottom:1px dashed rgba(255,255,255,0.03)}
    .big{font-weight:800;font-size:20px}

    /* pages */
    .page{display:none}
    .page.active{display:block}

    /* 3rd page - big linear graph container */
    .trend-graph{height:380px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.02));border-radius:12px;padding:12px}

    /* responsive */
    @media(max-width:980px){
      .login-wrap{grid-template-columns:1fr}
      .main-grid{grid-template-columns:1fr}
      .layer.a,.layer.b,.layer.c{left:6%;transform:none}
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="topbar">
      <div class="brand">ExpenseTracker</div>
      <div class="top-controls">
        <select id="lang" title="Language"><option value="en">English</option><option value="mr">मराठी</option></select>
        <select id="topCurrency"></select>
        <input id="convAmount" placeholder="100" style="width:90px" />
        <select id="convTo"></select>
        <button id="btnConv">Convert</button>
        <button id="navDashboard">Dashboard</button>
        <button id="navTrends">Trends</button>
        <button id="navLogin" style="display:none">Login</button>
      </div>
    </div>

    <!-- LOGIN PAGE -->
    <div id="loginBlock" class="login-wrap">
      <div class="login-card">
        <h1 id="welcomeTitle">Welcome — Expense Tracker</h1>
        <p class="muted" id="welcomeSub">Track, split & share — for roommates, trips & students.</p>

        <div class="field">
          <label class="muted" id="lblUser">Username</label>
          <input id="username" placeholder="e.g. Samruddhi" />
        </div>
        <div class="field">
          <label class="muted" id="lblPass">Password</label>
          <input id="password" type="password" placeholder="strong password" />
        </div>

        <div class="flex-row">
          <button id="btnLoginNow" class="primary">Login / Sign up</button>
          <div style="width:10px"></div>
          <button id="btnHelp">?</button>
        </div>

        <p class="muted" style="margin-top:12px">Password hashed locally (SHA256). Expenses encrypted (AES) in your browser. Use a strong password to protect data.</p>
      </div>

      <div class="image-block">
        <div class="layer a"></div>
        <div class="layer b"></div>
        <div class="layer c" id="mainImage" style="background-image:url('https://images.unsplash.com/photo-1524758631624-e2822e304c36?w=900&q=60')"></div>
        <div class="caption muted">A colorful 3D-style welcome — click login to start.</div>
      </div>
    </div>

    <!-- APP PAGES -->
    <div id="appPages" style="display:none">
      <!-- DASHBOARD PAGE -->
      <div id="pageDashboard" class="page active">
        <div class="main-grid">
          <!-- Left column -->
          <div class="panel">
            <div class="flex-row" style="justify-content:space-between">
              <div>
                <div class="muted">Budget</div>
                <input id="budgetInput" value="30000" />
              </div>
              <div style="text-align:right">
                <div class="muted">Remaining</div>
                <div class="big" id="remaining">0</div>
              </div>
            </div>

            <hr style="margin:10px 0;border:none;border-top:1px dashed rgba(255,255,255,0.04)">

            <div class="muted">Add Expense</div>
            <div style="margin-top:8px">
              <select id="catSelect"></select>
              <input id="amount" placeholder="Amount" style="margin-top:8px" />
              <input id="note" placeholder="Note (optional)" style="margin-top:8px" />
              <input id="expDate" type="date" style="margin-top:8px;width:100%" />
              <div style="display:flex;gap:8px;margin-top:8px">
                <button id="addBtn" class="primary">Add Expense</button>
                <button id="splitBtn">Split Bill</button>
              </div>
            </div>

            <hr style="margin:10px 0;border:none;border-top:1px dashed rgba(255,255,255,0.04)">

            <div class="muted">Share & Import</div>
            <div style="margin-top:8px;display:flex;gap:8px">
              <input type="file" id="fileImport" accept=".json" />
              <button id="importBtn">Import JSON</button>
            </div>
            <div style="margin-top:8px">
              <label class="muted">Due date reminder</label>
              <input id="dueDate" type="date" />
              <button id="setDue">Set Reminder</button>
            </div>
            <div style="margin-top:10px;display:flex;gap:6px">
              <button id="exportCSV">Export CSV</button>
              <button id="exportJSON">Export JSON</button>
              <button id="shareBtn">Share (roommates)</button>
            </div>
          </div>

          <!-- Right column -->
          <div>
            <div class="panel" style="margin-bottom:12px">
              <div style="display:flex;justify-content:space-between;align-items:center">
                <div style="display:flex;gap:8px;align-items:center">
                  <button class="chip" id="btnMonthly">Monthly</button>
                  <button class="chip" id="btnYearly">Yearly</button>
                </div>
                <div class="muted" id="recordCount">0 records</div>
              </div>

              <div style="display:flex;gap:12px;margin-top:12px;flex-wrap:wrap">
                <canvas id="mainLine" style="flex:1;min-width:300px;height:260px;background:linear-gradient(180deg, rgba(255,255,255,0.01), transparent);border-radius:10px"></canvas>
                <canvas id="mainPie" style="width:320px;height:260px;background:linear-gradient(180deg, rgba(255,255,255,0.01), transparent);border-radius:10px"></canvas>
              </div>
            </div>

            <div class="panel">
              <div style="display:flex;justify-content:space-between;align-items:center">
                <div class="muted">Transactions</div>
                <div><button id="clearAll">Clear All</button></div>
              </div>
              <div style="max-height:260px;overflow:auto;margin-top:8px">
                <table><thead><tr><th>Date</th><th>Category</th><th>Note</th><th>Amount</th></tr></thead><tbody id="txList"></tbody></table>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- TRENDS PAGE (3rd) -->
      <div id="pageTrends" class="page">
        <div class="panel">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>Trends</strong><div class="muted">Red = Expense, Green = Budget/Balance</div></div>
            <div>
              <button id="trendBack">Back</button>
            </div>
          </div>
          <div class="trend-graph" style="margin-top:12px">
            <canvas id="trendChart" style="width:100%;height:100%"></canvas>
          </div>
        </div>
      </div>
    </div>

    <footer style="margin-top:16px" class="muted">Built locally • Local AES encryption • SHA256 password hash • Host with HTTPS for full features.</footer>
  </div>

<script>
/* -------------------------
   Translations and config
   ------------------------- */
const TRANSLATIONS = {
  en:{welcome:'Welcome — Expense Tracker', subtitle:'Track, split & share — for roommates, trips & students.', username:'Username', password:'Password'},
  mr:{welcome:'स्वागत आहे — खर्च ट्रॅकर', subtitle:'खर्च ट्रॅक करा, विभाजित करा आणि शेअर करा — रूममेट्स, ट्रिप्स आणि विद्यार्थी.', username:'वापरकर्तानाव', password:'पासवर्ड'}
};
const CATEGORIES = [
  {key:'food', label:'Food', emoji:'🍔'},
  {key:'transport', label:'Transport', emoji:'🚗'},
  {key:'rent', label:'Rent', emoji:'🏠'},
  {key:'bills', label:'Utilities', emoji:'💡'},
  {key:'fun', label:'Entertainment', emoji:'🎮'},
  {key:'other', label:'Other', emoji:'✨'}
];
const DEFAULT_RATES = { INR:1, USD:0.012, EUR:0.011 };

/* -------------------------
   Crypto helpers & storage
   ------------------------- */
const AES = CryptoJS.AES, SHA256 = CryptoJS.SHA256;
function hash(s){ return SHA256(s).toString(); }
function encrypt(obj, pass){ return AES.encrypt(JSON.stringify(obj), pass).toString(); }
function decrypt(str, pass){ try{ const b = AES.decrypt(str, pass); const txt = b.toString(CryptoJS.enc.Utf8); return txt ? JSON.parse(txt) : null; }catch(e){ return null; } }

/* -------------------------
   App state
   ------------------------- */
let expenses = []; // {id,date,cat,amt,note}
let currencyRates = JSON.parse(localStorage.getItem('rates')||JSON.stringify(DEFAULT_RATES));
let currency = localStorage.getItem('currency') || 'INR';
let encPass = localStorage.getItem('enc_pass') || null;

/* -------------------------
   DOM refs
   ------------------------- */
const langSel = document.getElementById('lang');
const loginBlock = document.getElementById('loginBlock');
const appPages = document.getElementById('appPages');
const btnLoginNow = document.getElementById('btnLoginNow');
const usernameInput = document.getElementById('username');
const passwordInput = document.getElementById('password');
const welcomeTitle = document.getElementById('welcomeTitle');
const welcomeSub = document.getElementById('welcomeSub');
const topCurrency = document.getElementById('topCurrency');
const convTo = document.getElementById('convTo');
const convAmount = document.getElementById('convAmount');
const btnConv = document.getElementById('btnConv');
const navDashboard = document.getElementById('navDashboard');
const navTrends = document.getElementById('navTrends');
const navLogin = document.getElementById('navLogin');

const catSelect = document.getElementById('catSelect');
const addBtn = document.getElementById('addBtn');
const amountInput = document.getElementById('amount');
const noteInput = document.getElementById('note');
const expDate = document.getElementById('expDate');
const budgetInput = document.getElementById('budgetInput');
const remainingEl = document.getElementById('remaining');
const txList = document.getElementById('txList');
const recordCount = document.getElementById('recordCount');
const btnMonthly = document.getElementById('btnMonthly');
const btnYearly = document.getElementById('btnYearly');
const mainLine = document.getElementById('mainLine').getContext('2d');
const mainPie = document.getElementById('mainPie').getContext('2d');
const exportCSVBtn = document.getElementById('exportCSV');
const exportJSONBtn = document.getElementById('exportJSON');
const importBtn = document.getElementById('importBtn');
const fileImport = document.getElementById('fileImport');
const shareBtn = document.getElementById('shareBtn');
const splitBtn = document.getElementById('splitBtn');
const setDue = document.getElementById('setDue');
const dueDate = document.getElementById('dueDate');
const clearAllBtn = document.getElementById('clearAll');

const pageDashboard = document.getElementById('pageDashboard');
const pageTrends = document.getElementById('pageTrends');
const navTrendBtn = document.getElementById('navTrends');
const trendChartCtx = document.getElementById('trendChart').getContext('2d');
const trendBack = document.getElementById('trendBack');

/* -------------------------
   Init UI
   ------------------------- */
function init(){
  // translations
  langSel.addEventListener('change', applyLang);
  applyLang();

  // categories
  CATEGORIES.forEach(c=>{
    const opt = document.createElement('option'); opt.value=c.key; opt.textContent = ${c.emoji} ${c.label}; catSelect.appendChild(opt);
  });

  // currency selectors fill
  function fillCurr(){
    topCurrency.innerHTML=''; convTo.innerHTML='';
    Object.keys(currencyRates).forEach(k=>{
      const o1=document.createElement('option'); o1.value=k; o1.textContent=k; topCurrency.appendChild(o1);
      const o2=document.createElement('option'); o2.value=k; o2.textContent=k; convTo.appendChild(o2);
    });
    topCurrency.value = currency;
    convTo.value = 'USD';
  }
  fillCurr();

  // date default
  expDate.valueAsDate = new Date();
  // login prefill
  if(localStorage.getItem('saved_user')) usernameInput.value = localStorage.getItem('saved_user');

  // events
  btnLoginNow.onclick = loginAction;
  addBtn.onclick = addExpense;
  btnMonthly.onclick = ()=>{ btnMonthly.classList.add('active'); btnYearly.classList.remove('active'); updateCharts(); }
  btnYearly.onclick = ()=>{ btnYearly.classList.add('active'); btnMonthly.classList.remove('active'); updateCharts(); }
  navDashboard.onclick = ()=>showPage('dashboard');
  navTrends.onclick = ()=>showPage('trends');
  navLogin.onclick = ()=>{ location.reload(); };
  btnConv.onclick = ()=>{ const v=Number(convAmount.value); const to=convTo.value; if(!v) return alert('enter'); alert(${v} ${topCurrency.value} ≈ ${convertCurrency(v,to)} ${to}); }
  exportCSVBtn.onclick = exportCSV;
  exportJSONBtn.onclick = exportJSON;
  importBtn.onclick = ()=>{ if(fileImport.files[0]) importJSON(fileImport.files[0]); else alert('choose file'); }
  shareBtn.onclick = shareData;
  splitBtn.onclick = splitBill;
  setDue.onclick = setDueReminder;
  clearAllBtn.onclick = ()=>{ if(confirm('Clear all?')){ expenses=[]; persist(); refresh(); }};

  trendBack.onclick = ()=>showPage('dashboard');

  // charts setup
  setupCharts();

  // load encrypted data if exists
  loadEncrypted();
  refresh();
}
init();

/* -------------------------
   Language
   ------------------------- */
function applyLang(){
  const L = langSel.value;
  welcomeTitle.textContent = TRANSLATIONS[L].welcome;
  welcomeSub.textContent = TRANSLATIONS[L].subtitle;
  document.getElementById('lblUser').textContent = TRANSLATIONS[L].username;
  document.getElementById('lblPass').textContent = TRANSLATIONS[L].password;
}

/* -------------------------
   Auth
   ------------------------- */
function loginAction(){
  const u = usernameInput.value.trim(); const p = passwordInput.value;
  if(!u || !p) return alert('Enter username & password');
  const stored = localStorage.getItem('user_hash');
  const my = hash(u + '|' + p);
  if(!stored){
    localStorage.setItem('user_hash', my);
    localStorage.setItem('saved_user', u);
    localStorage.setItem('enc_pass', p);
    alert('Account created — data will be encrypted with your password.');
  } else {
    if(stored !== my) return alert('Incorrect credentials');
  }
  encPass = p;
  localStorage.setItem('enc_pass', p);
  localStorage.setItem('saved_user', u);
  // show app
  loginBlock.style.display = 'none';
  appPages.style.display = 'block';
}

/* -------------------------
   Save / Load encrypted
   ------------------------- */
function persist(){
  try{
    const enc = encrypt(expenses, localStorage.getItem('enc_pass') || encPass || 'default-pass');
    localStorage.setItem('expenses_enc', enc);
    localStorage.setItem('rates', JSON.stringify(currencyRates));
    localStorage.setItem('currency', topCurrency.value || currency);
  }catch(e){ console.error('save fail', e); }
}

function loadEncrypted(){
  const enc = localStorage.getItem('expenses_enc');
  if(!enc) {
    expenses = JSON.parse(localStorage.getItem('expenses_plain') || '[]');
    return;
  }
  const pass = localStorage.getItem('enc_pass') || encPass || prompt('Enter passphrase to decrypt (cancel to skip)');
  if(!pass) return;
  const dec = decrypt(enc, pass);
  if(Array.isArray(dec)) expenses = dec;
  else console.warn('Decryption failed');
}

/* -------------------------
   CRUD actions
   ------------------------- */
function addExpense(){
  const a = Number(amountInput.value);
  if(!a || a<=0) return alert('Enter valid amount');
  const item = {
    id: Date.now(),
    date: expDate.value || new Date().toISOString().slice(0,10),
    cat: catSelect.value,
    amt: Number(a.toFixed(2)),
    note: noteInput.value || ''
  };
  expenses.push(item);
  persist(); refresh();
  amountInput.value=''; noteInput.value='';
}

function refresh(){
  // update remaining
  const budget = Number(budgetInput.value) || 0;
  const spent = expenses.reduce((s,e)=> s + Number(e.amt), 0);
  remainingEl.textContent = (budget - spent).toFixed(2) + ' ' + (topCurrency.value || currency);

  // list
  txList.innerHTML = '';
  expenses.slice().reverse().forEach(e=>{
    const cat = CATEGORIES.find(c=>c.key===e.cat) || {emoji:'',label:e.cat};
    const tr = document.createElement('tr');
    tr.innerHTML = <td>${e.date}</td><td>${cat.emoji} ${cat.label}</td><td>${e.note||''}</td><td>${e.amt.toFixed(2)} ${topCurrency.value||currency}</td>;
    txList.appendChild(tr);
  });
  recordCount.textContent = ${expenses.length} records;
  updateCharts();
}

/* -------------------------
   Export / Import / Share
   ------------------------- */
function exportCSV(){
  if(!expenses.length) return alert('No data');
  let csv = 'id,date,category,amount,note\n' + expenses.map(e=>${e.id},${e.date},${e.cat},${e.amt},"${(e.note||'').replace(/"/g,'""')}").join('\n');
  const blob = new Blob([csv], {type:'text/csv'}); const url=URL.createObjectURL(blob);
  const a=document.createElement('a'); a.href=url; a.download=expenses_${new Date().toISOString().slice(0,10)}.csv; a.click(); URL.revokeObjectURL(url);
}
function exportJSON(){
  if(!expenses.length) return alert('No data');
  const blob = new Blob([JSON.stringify(expenses,null,2)], {type:'application/json'}); const url=URL.createObjectURL(blob);
  const a=document.createElement('a'); a.href=url; a.download=expenses_${new Date().toISOString().slice(0,10)}.json; a.click(); URL.revokeObjectURL(url);
}
function importJSON(file){
  const r=new FileReader();
  r.onload = e => {
    try{ const j=JSON.parse(e.target.result); if(Array.isArray(j)){ expenses = expenses.concat(j); persist(); refresh(); alert('Imported'); } else alert('Invalid file'); }
    catch(err){ alert('Parse error'); }
  };
  r.readAsText(file);
}
importBtn.addEventListener('click', ()=>{ if(fileImport.files[0]) importJSON(fileImport.files[0]); else alert('Choose a file'); });

function shareData(){
  if(!expenses.length) return alert('No data to share');
  const pass = prompt('Enter a temporary passphrase to encrypt the shared snapshot (share this passphrase with recipients):');
  if(!pass) return;
  const enc = encrypt(expenses, pass);
  const payload = btoa(enc);
  const shareText = Shared snapshot: #share:${payload}\nPassphrase: ${pass};
  navigator.clipboard.writeText(shareText).then(()=> alert('Share payload copied to clipboard — send it to roommates. Paste #share:... link in URL to import.'));
}

/* auto-import if URL has #share:... */
if(location.hash && location.hash.startsWith('#share:')){
  try{
    const payload = location.hash.replace('#share:','');
    const enc = atob(payload);
    const pass = prompt('Enter passphrase to decrypt shared snapshot:');
    if(pass){
      const dec = decrypt(enc, pass);
      if(Array.isArray(dec)){ expenses = expenses.concat(dec); persist(); refresh(); alert('Imported shared expenses'); location.hash=''; }
      else alert('Bad passphrase or payload');
    }
  }catch(e){}
}

/* -------------------------
   Split bill
   ------------------------- */
function splitBill(){
  const total = prompt('Enter total amount to split:');
  if(!total) return;
  const ppl = prompt('Enter names separated by comma (Alice,Bob,Charlie):');
  if(!ppl) return;
  const names = ppl.split(',').map(s=>s.trim()).filter(Boolean);
  const t = Number(total);
  if(!t || !names.length) return alert('Invalid');
  const per = Number((t / names.length).toFixed(2));
  const dateStr = new Date().toISOString().slice(0,10);
  names.forEach(n=>{
    expenses.push({ id: Date.now()+Math.floor(Math.random()*1000), date: dateStr, cat:'other', amt: per, note:Split (${names.join(',')}) - ${n} });
  });
  persist(); refresh();
  alert(Created ${names.length} entries of ${per} each.);
}

/* -------------------------
   Due date reminder
   ------------------------- */
function setDueReminder(){
  const d = dueDate.value;
  if(!d) return alert('Pick date');
  const when = new Date(d).getTime() - Date.now();
  if(when <= 0) return alert('Pick future date');
  if(Notification && Notification.permission !== 'granted') Notification.requestPermission();
  setTimeout(()=> {
    if(Notification && Notification.permission === 'granted') new Notification('Expense Due', { body:You set a due date for ${d} });
    else alert(Due date reached: ${d});
  }, when);
  alert('Reminder set (keep site open or host a service to ensure notification when closed).');
}

/* -------------------------
   Currency convert (basic)
   ------------------------- */
function convertCurrency(val,to){
  const baseINR = val * (currencyRates['INR'] || 1) / (currencyRates[topCurrency.value] || 1);
  const conv = baseINR * ((currencyRates[to]||1) / (currencyRates['INR']||1));
  return Number(conv.toFixed(2));
}
btnConv.addEventListener('click', ()=>{ const v=Number(convAmount.value); const to=convTo.value; if(!v) return alert('Enter value'); alert(${v} ${topCurrency.value} ≈ ${convertCurrency(v,to)} ${to}); });

/* -------------------------
   Charts
   ------------------------- */
let lineChart, pieChart, trendChart;
function setupCharts(){
  lineChart = new Chart(mainLine, {
    type:'line',
    data:{ labels:[], datasets:[ { label:'Expenses', data:[], fill:true, backgroundColor:'rgba(124,92,255,0.12)', borderColor:'#7c5cff', tension:0.3, borderWidth:2 } ]},
    options:{ responsive:true, maintainAspectRatio:false, plugins:{legend:{display:false}} }
  });
  pieChart = new Chart(mainPie, {
    type:'pie',
    data:{ labels:[], datasets:[ { data:[], backgroundColor:['#8884d8','#82ca9d','#ffc658','#ff7f7f','#a28df0','#f8b195'] } ]},
    options:{ responsive:true, plugins:{legend:{position:'bottom', labels:{color:'white'}}}}
  });

  trendChart = new Chart(trendChartCtx, {
    type:'line',
    data:{ labels:[], datasets:[
      { label:'Expenses', data:[], borderColor:'rgba(220,53,69,0.95)', backgroundColor:'rgba(220,53,69,0.12)', tension:0.3, borderWidth:2 },
      { label:'Budget Balance', data:[], borderColor:'rgba(40,167,69,0.95)', backgroundColor:'rgba(40,167,69,0.12)', tension:0.3, borderWidth:2 }
    ]},
    options:{ responsive:true, maintainAspectRatio:false, scales:{y:{beginAtZero:true}} }
  });
  updateCharts();
}

function updateCharts(){
  // monthly aggregation
  const mMap = {};
  expenses.forEach(e=>{ const m = e.date.slice(0,7); mMap[m] = (mMap[m]||0) + Number(e.amt); });
  const months = Object.keys(mMap).sort();
  const mData = months.map(k=>Number(mMap[k].toFixed(2)));

  // yearly
  const yMap = {};
  expenses.forEach(e=>{ const y = e.date.slice(0,4); yMap[y] = (yMap[y]||0) + Number(e.amt); });
  const years = Object.keys(yMap).sort();
  const yData = years.map(k=>Number(yMap[k].toFixed(2)));

  // choose view (monthly default)
  const labels = (btnMonthly.classList.contains('active') || !btnYearly.classList.contains('active')) ? (months.length ? months : [new Date().toISOString().slice(0,7)]) : (years.length ? years : [new Date().getFullYear()]);
  const dataToUse = (btnMonthly.classList.contains('active') || !btnYearly.classList.contains('active')) ? (mData.length ? mData : [0]) : (yData.length ? yData : [0]);

  lineChart.data.labels = labels;
  lineChart.data.datasets[0].data = dataToUse;
  lineChart.update();

  // pie by category
  const catMap = {};
  expenses.forEach(e=> catMap[e.cat] = (catMap[e.cat]||0) + Number(e.amt));
  const pieLabels = CATEGORIES.map(c=>${c.emoji} ${c.label});
  const pieValues = CATEGORIES.map(c=> Number((catMap[c.key]||0).toFixed(2)));
  pieChart.data.labels = pieLabels;
  pieChart.data.datasets[0].data = pieValues;
  pieChart.update();

  // trend chart: red = expenses history by month, green = budget balance (budget - cumulative spent)
  const budget = Number(budgetInput.value) || 0;
  const cum = [];
  let sum = 0;
  const trendLabels = labels.slice();
  labels.forEach((lab, idx)=>{
    const val = dataToUse[idx] || 0;
    sum += val;
    cum.push(sum);
  });
  const expensesLine = dataToUse.slice();
  const balanceLine = labels.map((l,i)=> Math.max(0, (budget - (cum[i] || 0))).toFixed(2));

  trendChart.data.labels = trendLabels;
  trendChart.data.datasets[0].data = expensesLine;
  trendChart.data.datasets[1].data = balanceLine;
  trendChart.update();
}

/* -------------------------
   Pages navigation
   ------------------------- */
function showPage(name){
  document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
  document.getElementById('pageDashboard').classList.remove('active');
  document.getElementById('pageTrends').classList.remove('active');
  if(name==='dashboard'){ document.getElementById('pageDashboard').classList.add('active'); }
  if(name==='trends'){ document.getElementById('pageTrends').classList.add('active'); updateCharts(); }
}

/* -------------------------
   Persistence on unload
   ------------------------- */
window.addEventListener('beforeunload', ()=> persist() );

/* -------------------------
   Load existing settings and refresh
   ------------------------- */
function loadInitial(){
  try{ currencyRates = JSON.parse(localStorage.getItem('rates')||JSON.stringify(DEFAULT_RATES)); }catch(e){}
  if(localStorage.getItem('currency')) topCurrency.value = localStorage.getItem('currency');
  // fill topCurrency/convTo options
  topCurrency.innerHTML=''; convTo.innerHTML='';
  Object.keys(currencyRates).forEach(k=>{
    const o1=document.createElement('option'); o1.value=k; o1.textContent=k; topCurrency.appendChild(o1);
    const o2=document.createElement('option'); o2.value=k; o2.textContent=k; convTo.appendChild(o2);
  });
  topCurrency.value = localStorage.getItem('currency') || 'INR';
  convTo.value = 'USD';
}
loadInitial();
persist(); // ensure data saved
// initial chart update
updateCharts();

</script>
</body>
</html># sam.in1
