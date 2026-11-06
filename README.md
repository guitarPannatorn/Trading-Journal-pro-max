<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>สมุดบันทึกการเทรด</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans+Thai:wght@400;500;600&family=IBM+Plex+Sans:wght@400;500;600&display=swap" rel="stylesheet">
<style>
:root{
  --bg:#EDF0F3; --card:#FFFFFF; --ink:#1B2733; --muted:#66727F; --line:#D5DBE1;
  --win:#0F766E; --loss:#B4332E; --brass:#A67C1E; --focus:#1D4ED8;
  --font:"IBM Plex Sans Thai","IBM Plex Sans",system-ui,sans-serif;
}
@media (prefers-color-scheme:dark){
  :root{--bg:#12181E; --card:#1A222A; --ink:#E6EBF0; --muted:#8C99A6; --line:#2C3743; --win:#34B3A5; --loss:#E5726C; --brass:#D4A93E; --focus:#7AA2F7;}
}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font-family:var(--font);line-height:1.55}
.wrap{max-width:1100px;margin:0 auto;padding:20px 16px 60px}
h1{font-size:22px;margin:0 0 4px;font-weight:600}
.sub{color:var(--muted);font-size:14px;margin:0 0 20px}
.card{background:var(--card);border:1px solid var(--line);border-radius:8px;padding:16px;margin-bottom:16px}
.card h2{font-size:16px;margin:0 0 12px;font-weight:600}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(130px,1fr));gap:10px;margin-bottom:16px}
.stat{background:var(--card);border:1px solid var(--line);border-radius:6px;padding:10px 12px}
.stat b{display:block;font-size:20px;font-weight:600;font-variant-numeric:tabular-nums}
.stat span{font-size:13px;color:var(--muted)}
.pos{color:var(--win)} .neg{color:var(--loss)}
form{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:10px}
label{display:flex;flex-direction:column;font-size:13px;color:var(--muted);gap:4px}
input,select,textarea,button{font:inherit;color:var(--ink)}
input,select,textarea{background:var(--bg);border:1px solid var(--line);border-radius:6px;padding:8px;width:100%}
.full{grid-column:1/-1}
textarea{min-height:60px;resize:vertical}
button{border:1px solid var(--line);background:var(--card);padding:8px 14px;border-radius:6px;cursor:pointer}
button.primary{background:var(--ink);color:var(--card);border-color:var(--ink);font-weight:500}
button:hover{border-color:var(--muted)}
:focus-visible{outline:2px solid var(--focus);outline-offset:2px}
.row{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
.hint{font-size:12px;color:var(--muted)}
.tbl{overflow-x:auto}
table{border-collapse:collapse;width:100%;font-size:14px;font-variant-numeric:tabular-nums}
th,td{padding:8px 10px;border-bottom:1px solid var(--line);text-align:left;white-space:nowrap}
th{font-weight:500;color:var(--muted);font-size:13px}
td.note{white-space:normal;min-width:160px;color:var(--muted)}
td.num,th.num{text-align:right}
.tag{display:inline-block;padding:1px 8px;border-radius:99px;font-size:12px;border:1px solid var(--line)}
.buy{color:var(--win);border-color:var(--win)} .sell{color:var(--loss);border-color:var(--loss)}
.empty{padding:24px;text-align:center;color:var(--muted)}
svg{width:100%;height:auto;display:block}
.filters{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:10px}
.filters label{min-width:130px}
</style>
</head>
<body>
<div class="wrap">
  <h1>สมุดบันทึกการเทรด</h1>
  <p class="sub">ข้อมูลเก็บในเบราว์เซอร์เครื่องนี้ ใช้ปุ่มสำรอง CSV เพื่อเก็บไว้หรือย้ายเครื่อง</p>

  <div class="stats" id="stats"></div>

  <div class="card">
    <h2>กราฟกำไรสะสม</h2>
    <div id="chart"></div>
  </div>

  <div class="card">
    <h2 id="formTitle">บันทึกไม้ใหม่</h2>
    <form id="f">
      <label>วันที่<input type="date" name="date" required></label>
      <label>สัญลักษณ์<input name="symbol" value="XAUUSD" required></label>
      <label>ฝั่ง
        <select name="side"><option value="buy">Buy</option><option value="sell">Sell</option></select>
      </label>
      <label>ราคาเข้า<input type="number" step="any" name="entry"></label>
      <label>ราคาออก<input type="number" step="any" name="exit"></label>
      <label>Lot<input type="number" step="any" name="lot" value="0.01"></label>
      <label>Contract size<input type="number" step="any" name="cs" value="100"></label>
      <label>กำไร/ขาดทุน ($)<input type="number" step="any" name="pnl" placeholder="เว้นว่าง = คำนวณให้"></label>
      <label>ความเสี่ยงต่อไม้ ($)<input type="number" step="any" name="risk" placeholder="ใช้คำนวณ R"></label>
      <label>กลยุทธ์<input name="strategy" list="strats" placeholder="เช่น Breakout H1"></label>
      <datalist id="strats"></datalist>
      <label class="full">บันทึก (เหตุผลเข้า / ความรู้สึก / บทเรียน)<textarea name="note"></textarea></label>
      <div class="row full">
        <button class="primary" type="submit" id="saveBtn">บันทึกไม้</button>
        <button type="button" id="cancelBtn" hidden>ยกเลิกการแก้ไข</button>
        <span class="hint">ถ้าเว้นช่องกำไร/ขาดทุนไว้ ระบบคำนวณจาก (ออก − เข้า) × Lot × Contract size</span>
      </div>
    </form>
  </div>

  <div class="card">
    <h2>ประวัติการเทรด</h2>
    <div class="filters">
      <label>กลยุทธ์<select id="fStrat"><option value="">ทั้งหมด</option></select></label>
      <label>ผลลัพธ์<select id="fRes"><option value="">ทั้งหมด</option><option value="w">ชนะ</option><option value="l">แพ้</option></select></label>
    </div>
    <div class="tbl" id="tbl"></div>
    <div class="row" style="margin-top:12px">
      <button id="exp">สำรองเป็น CSV</button>
      <button id="impBtn">นำเข้า CSV</button>
      <input type="file" id="imp" accept=".csv" hidden>
    </div>
  </div>
</div>

<script>
const KEY = 'trading-journal-v1';
let trades = [];
let editId = null;

function load(){
  try { trades = JSON.parse(localStorage.getItem(KEY)) || []; } catch(e){ trades = []; }
}
function save(){
  try { localStorage.setItem(KEY, JSON.stringify(trades)); }
  catch(e){ alert('บันทึกลงเบราว์เซอร์ไม่ได้ กรุณากดสำรองเป็น CSV ไว้ก่อน'); }
}
const $ = s => document.querySelector(s);
const esc = s => String(s ?? '').replace(/[&<>"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
const fmt = n => (n >= 0 ? '+' : '−') + Math.abs(n).toLocaleString('en-US',{minimumFractionDigits:2,maximumFractionDigits:2});
const cls = n => n > 0 ? 'pos' : n < 0 ? 'neg' : '';

function sorted(list){ return [...list].sort((a,b) => a.date.localeCompare(b.date) || a.id - b.id); }

function renderStats(){
  const list = sorted(trades);
  const n = list.length;
  const wins = list.filter(t => t.pnl > 0), losses = list.filter(t => t.pnl < 0);
  const total = list.reduce((s,t) => s + t.pnl, 0);
  const gp = wins.reduce((s,t) => s + t.pnl, 0), gl = Math.abs(losses.reduce((s,t) => s + t.pnl, 0));
  const pf = gl ? (gp / gl).toFixed(2) : (gp ? '∞' : '–');
  const wr = n ? (wins.length / n * 100).toFixed(1) + '%' : '–';
  const avgW = wins.length ? gp / wins.length : 0, avgL = losses.length ? gl / losses.length : 0;
  const rs = list.filter(t => t.risk > 0).map(t => t.pnl / t.risk);
  const avgR = rs.length ? (rs.reduce((a,b) => a + b, 0) / rs.length).toFixed(2) : '–';
  let peak = 0, eq = 0, dd = 0;
  list.forEach(t => { eq += t.pnl; peak = Math.max(peak, eq); dd = Math.max(dd, peak - eq); });
  const items = [
    ['กำไรรวม ($)', fmt(total), cls(total)],
    ['จำนวนไม้', n, ''],
    ['Win rate', wr, ''],
    ['Profit factor', pf, ''],
    ['กำไรเฉลี่ย/ไม้ชนะ', n ? fmt(avgW) : '–', 'pos'],
    ['ขาดทุนเฉลี่ย/ไม้แพ้', n ? '−' + avgL.toFixed(2) : '–', 'neg'],
    ['R เฉลี่ย', avgR, ''],
    ['Drawdown สูงสุด ($)', n ? dd.toFixed(2) : '–', dd ? 'neg' : '']
  ];
  $('#stats').innerHTML = items.map(([l,v,c]) => `<div class="stat"><b class="${c}">${v}</b><span>${l}</span></div>`).join('');
}

function renderChart(){
  const list = sorted(trades);
  if (list.length < 2){ $('#chart').innerHTML = '<div class="empty">บันทึกอย่างน้อย 2 ไม้ เพื่อดูกราฟ</div>'; return; }
  let eq = 0; const pts = [0, ...list.map(t => eq += t.pnl)];
  const W = 800, H = 240, p = 36;
  const min = Math.min(...pts), max = Math.max(...pts), span = (max - min) || 1;
  const x = i => p + i * (W - 2*p) / (pts.length - 1);
  const y = v => H - p - (v - min) * (H - 2*p) / span;
  const path = pts.map((v,i) => (i ? 'L' : 'M') + x(i).toFixed(1) + ' ' + y(v).toFixed(1)).join(' ');
  const zero = (min < 0 && max > 0) ? `<line x1="${p}" x2="${W-p}" y1="${y(0)}" y2="${y(0)}" stroke="var(--line)" stroke-dasharray="4 4"/>` : '';
  const last = pts[pts.length - 1];
  $('#chart').innerHTML = `<svg viewBox="0 0 ${W} ${H}" role="img" aria-label="กราฟกำไรสะสม">
    ${zero}
    <path d="${path}" fill="none" stroke="${last >= 0 ? 'var(--win)' : 'var(--loss)'}" stroke-width="2"/>
    <text x="${p}" y="16" fill="var(--muted)" font-size="12">สูงสุด ${max.toFixed(2)}</text>
    <text x="${p}" y="${H-8}" fill="var(--muted)" font-size="12">ต่ำสุด ${min.toFixed(2)}</text>
    <text x="${W-p}" y="16" fill="var(--muted)" font-size="12" text-anchor="end">ล่าสุด ${last.toFixed(2)}</text>
  </svg>`;
}

function renderFilters(){
  const strats = [...new Set(trades.map(t => t.strategy).filter(Boolean))];
  const cur = $('#fStrat').value;
  $('#fStrat').innerHTML = '<option value="">ทั้งหมด</option>' + strats.map(s => `<option ${s===cur?'selected':''}>${esc(s)}</option>`).join('');
  $('#strats').innerHTML = strats.map(s => `<option value="${esc(s)}">`).join('');
}

function renderTable(){
  const fs = $('#fStrat').value, fr = $('#fRes').value;
  let list = sorted(trades).reverse();
  if (fs) list = list.filter(t => t.strategy === fs);
  if (fr === 'w') list = list.filter(t => t.pnl > 0);
  if (fr === 'l') list = list.filter(t => t.pnl < 0);
  if (!list.length){ $('#tbl').innerHTML = '<div class="empty">ยังไม่มีรายการ กรอกแบบฟอร์มด้านบนเพื่อบันทึกไม้แรก</div>'; return; }
  $('#tbl').innerHTML = `<table><thead><tr>
    <th>วันที่</th><th>สัญลักษณ์</th><th>ฝั่ง</th><th class="num">เข้า</th><th class="num">ออก</th><th class="num">Lot</th>
    <th class="num">P/L ($)</th><th class="num">R</th><th>กลยุทธ์</th><th>บันทึก</th><th></th></tr></thead><tbody>` +
    list.map(t => `<tr>
      <td>${esc(t.date)}</td><td>${esc(t.symbol)}</td>
      <td><span class="tag ${t.side}">${t.side === 'buy' ? 'Buy' : 'Sell'}</span></td>
      <td class="num">${t.entry ?? ''}</td><td class="num">${t.exit ?? ''}</td><td class="num">${t.lot ?? ''}</td>
      <td class="num ${cls(t.pnl)}">${fmt(t.pnl)}</td>
      <td class="num">${t.risk > 0 ? (t.pnl / t.risk).toFixed(2) : ''}</td>
      <td>${esc(t.strategy)}</td><td class="note">${esc(t.note)}</td>
      <td><button data-e="${t.id}">แก้ไข</button> <button data-d="${t.id}">ลบ</button></td></tr>`).join('') +
    '</tbody></table>';
}

function renderAll(){ renderStats(); renderChart(); renderFilters(); renderTable(); }

const f = $('#f');
function numOrNull(v){ return v === '' ? null : Number(v); }

f.addEventListener('submit', e => {
  e.preventDefault();
  const d = Object.fromEntries(new FormData(f));
  const entry = numOrNull(d.entry), exit = numOrNull(d.exit), lot = numOrNull(d.lot), cs = numOrNull(d.cs) ?? 100;
  let pnl = numOrNull(d.pnl);
  if (pnl === null){
    if (entry === null || exit === null || lot === null){ alert('กรอกกำไร/ขาดทุน หรือกรอกราคาเข้า ราคาออก และ Lot ให้ครบ'); return; }
    pnl = (exit - entry) * (d.side === 'buy' ? 1 : -1) * lot * cs;
    pnl = Math.round(pnl * 100) / 100;
  }
  const rec = { id: editId ?? Date.now(), date: d.date, symbol: d.symbol.trim().toUpperCase(), side: d.side,
    entry, exit, lot, pnl, risk: numOrNull(d.risk), strategy: d.strategy.trim(), note: d.note.trim() };
  if (editId !== null) trades = trades.map(t => t.id === editId ? rec : t); else trades.push(rec);
  save(); resetForm(); renderAll();
});

function resetForm(){
  editId = null; f.reset();
  f.date.valueAsDate = new Date();
  $('#formTitle').textContent = 'บันทึกไม้ใหม่';
  $('#saveBtn').textContent = 'บันทึกไม้';
  $('#cancelBtn').hidden = true;
}
$('#cancelBtn').onclick = resetForm;

$('#tbl').addEventListener('click', e => {
  const de = e.target.dataset.d, ee = e.target.dataset.e;
  if (de){
    if (confirm('ลบรายการนี้ใช่ไหม? การลบย้อนกลับไม่ได้')){ trades = trades.filter(t => t.id !== Number(de)); save(); renderAll(); }
  }
  if (ee){
    const t = trades.find(t => t.id === Number(ee)); if (!t) return;
    editId = t.id;
    f.date.value = t.date; f.symbol.value = t.symbol; f.side.value = t.side;
    f.entry.value = t.entry ?? ''; f.exit.value = t.exit ?? ''; f.lot.value = t.lot ?? '';
    f.pnl.value = t.pnl; f.risk.value = t.risk ?? ''; f.strategy.value = t.strategy; f.note.value = t.note;
    $('#formTitle').textContent = 'แก้ไขรายการ';
    $('#saveBtn').textContent = 'บันทึกการแก้ไข';
    $('#cancelBtn').hidden = false;
    f.scrollIntoView({behavior:'smooth'});
  }
});
$('#fStrat').onchange = renderTable; $('#fRes').onchange = renderTable;

/* CSV */
const COLS = ['date','symbol','side','entry','exit','lot','pnl','risk','strategy','note'];
function csvCell(v){ v = v ?? ''; v = String(v); return /[",\n]/.test(v) ? '"' + v.replace(/"/g,'""') + '"' : v; }
$('#exp').onclick = () => {
  if (!trades.length){ alert('ยังไม่มีข้อมูลให้สำรอง'); return; }
  const csv = '\uFEFF' + [COLS.join(','), ...sorted(trades).map(t => COLS.map(c => csvCell(t[c])).join(','))].join('\n');
  const a = document.createElement('a');
  a.href = URL.createObjectURL(new Blob([csv], {type:'text/csv;charset=utf-8'}));
  a.download = 'trading-journal-' + new Date().toISOString().slice(0,10) + '.csv';
  a.click(); URL.revokeObjectURL(a.href);
};
function parseCSV(text){
  const rows = []; let row = [], cur = '', q = false;
  text = text.replace(/^\uFEFF/, '');
  for (let i = 0; i < text.length; i++){
    const c = text[i];
    if (q){ if (c === '"'){ if (text[i+1] === '"'){ cur += '"'; i++; } else q = false; } else cur += c; }
    else if (c === '"') q = true;
    else if (c === ','){ row.push(cur); cur = ''; }
    else if (c === '\n' || c === '\r'){ if (c === '\r' && text[i+1] === '\n') i++; row.push(cur); rows.push(row); row = []; cur = ''; }
    else cur += c;
  }
  if (cur !== '' || row.length){ row.push(cur); rows.push(row); }
  return rows;
}
$('#impBtn').onclick = () => $('#imp').click();
$('#imp').onchange = async e => {
  const file = e.target.files[0]; if (!file) return;
  const rows = parseCSV(await file.text());
  const head = rows.shift() || [];
  if (!head.includes('date') || !head.includes('pnl')){ alert('ไฟล์ CSV ไม่ตรงรูปแบบ ต้องมีคอลัมน์ date และ pnl'); e.target.value = ''; return; }
  const added = rows.filter(r => r.length > 1).map((r, i) => {
    const o = {}; head.forEach((h, j) => o[h] = r[j]);
    return { id: Date.now() + i, date: o.date, symbol: (o.symbol || '').toUpperCase(), side: o.side === 'sell' ? 'sell' : 'buy',
      entry: numOrNull(o.entry ?? ''), exit: numOrNull(o.exit ?? ''), lot: numOrNull(o.lot ?? ''),
      pnl: Number(o.pnl) || 0, risk: numOrNull(o.risk ?? ''), strategy: o.strategy || '', note: o.note || '' };
  });
  if (confirm(`พบ ${added.length} รายการ ต้องการเพิ่มเข้าไปต่อท้ายข้อมูลเดิมใช่ไหม? (ข้อมูลเดิมจะไม่ถูกลบ)`)){
    trades = trades.concat(added); save(); renderAll();
  }
  e.target.value = '';
};

load(); resetForm(); renderAll();
</script>
</body>
</html>
