<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#0f0f0f" media="(prefers-color-scheme: dark)">
<meta name="theme-color" content="#f0f0f0" media="(prefers-color-scheme: light)">
<title>Vault</title>
<style>
  :root {
    --bg: #0f0f0f;
    --surface: rgba(255,255,255,0.07);
    --surface-strong: rgba(255,255,255,0.12);
    --border: rgba(255,255,255,0.1);
    --text: #f0f0f0;
    --text-muted: rgba(240,240,240,0.5);
    --accent: #7c9fff;
    --accent-soft: rgba(124,159,255,0.15);
    --danger: #ff6b6b;
    --success: #6bffb8;
    --panel-bg: rgba(15,15,15,0.82);
    --blur: blur(18px);
    --radius: 16px;
    --radius-sm: 10px;
    --font: 'Segoe UI', system-ui, sans-serif;
  }
  @media (prefers-color-scheme: light) {
    :root {
      --bg: #f0f0f0;
      --surface: rgba(0,0,0,0.06);
      --surface-strong: rgba(0,0,0,0.1);
      --border: rgba(0,0,0,0.1);
      --text: #111;
      --text-muted: rgba(0,0,0,0.45);
      --accent: #3a6fff;
      --accent-soft: rgba(58,111,255,0.12);
      --danger: #d94f4f;
      --success: #1aaa66;
      --panel-bg: rgba(240,240,240,0.85);
    }
  }
  * { box-sizing: border-box; margin: 0; padding: 0; -webkit-tap-highlight-color: transparent; }
  html, body { height: 100%; overflow: hidden; background: var(--bg); color: var(--text); font-family: var(--font); }

  /* BG IMAGE */
  #bg-layer { position: fixed; inset: 0; z-index: 0; background-size: cover; background-position: center; }
  #bg-overlay { position: fixed; inset: 0; z-index: 1; background: rgba(0,0,0,0.38); pointer-events: none; }
  @media (prefers-color-scheme: light) { #bg-overlay { background: rgba(255,255,255,0.3); } }

  /* LAYOUT */
  #app { position: fixed; inset: 0; z-index: 2; display: flex; flex-direction: column; }

  /* TOP BAR */
  #topbar {
    display: flex; align-items: center; gap: 10px;
    padding: 12px 14px 10px;
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border-bottom: 1px solid var(--border); flex-shrink: 0;
  }
  #menu-btn {
    width: 36px; height: 36px; border-radius: 10px;
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 18px; cursor: pointer;
    display: flex; align-items: center; justify-content: center; flex-shrink: 0;
  }
  #search-bar {
    flex: 1; height: 36px; border-radius: 10px;
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 14px; padding: 0 12px; outline: none;
  }
  #search-bar::placeholder { color: var(--text-muted); }
  #new-journal-btn {
    height: 36px; padding: 0 14px; border-radius: 10px;
    background: var(--accent); border: none;
    color: #fff; font-size: 13px; font-weight: 600; cursor: pointer; flex-shrink: 0;
  }

  /* MAIN CONTENT */
  #main { flex: 1; overflow-y: auto; padding: 14px; display: flex; flex-direction: column; gap: 12px; }

  /* HOME CARDS */
  .journal-card {
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border: 1px solid var(--border); border-radius: var(--radius);
    padding: 16px; cursor: pointer;
    transition: transform 0.15s; position: relative; overflow: hidden;
  }
  .journal-card:active { transform: scale(0.98); }
  .journal-card .card-bg { position: absolute; inset: 0; background-size: cover; background-position: center; opacity: 0.18; }
  .journal-card .card-content { position: relative; z-index: 1; }
  .card-top { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 8px; }
  .card-month { font-size: 17px; font-weight: 700; }
  .card-status { font-size: 11px; padding: 3px 8px; border-radius: 20px; font-weight: 600; }
  .status-active { background: var(--accent-soft); color: var(--accent); }
  .status-locked { background: rgba(255,107,107,0.15); color: var(--danger); }
  .card-balance { font-size: 22px; font-weight: 800; margin-bottom: 4px; }
  .card-meta { font-size: 12px; color: var(--text-muted); }
  .card-progress { margin-top: 10px; height: 4px; background: var(--surface-strong); border-radius: 4px; overflow: hidden; }
  .card-progress-fill { height: 100%; background: var(--accent); border-radius: 4px; }

  /* EMPTY STATE */
  #empty-state {
    flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
    gap: 12px; text-align: center; padding: 40px 20px;
  }
  #empty-state .icon { font-size: 48px; opacity: 0.4; }
  #empty-state h2 { font-size: 18px; opacity: 0.7; }
  #empty-state p { font-size: 13px; color: var(--text-muted); max-width: 260px; line-height: 1.5; }

  /* SLIDE PANEL */
  #panel-overlay { position: fixed; inset: 0; z-index: 10; background: rgba(0,0,0,0.5); opacity: 0; pointer-events: none; transition: opacity 0.3s; }
  #panel-overlay.open { opacity: 1; pointer-events: all; }
  #side-panel {
    position: fixed; top: 0; left: 0; bottom: 0; width: 82%; max-width: 320px; z-index: 11;
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border-right: 1px solid var(--border);
    transform: translateX(-100%); transition: transform 0.3s cubic-bezier(.4,0,.2,1);
    display: flex; flex-direction: column; overflow: hidden;
  }
  #side-panel.open { transform: translateX(0); }
  #panel-header { padding: 20px 18px 14px; border-bottom: 1px solid var(--border); flex-shrink: 0; }
  #panel-header h1 { font-size: 22px; font-weight: 800; letter-spacing: -0.5px; }
  #panel-header p { font-size: 12px; color: var(--text-muted); margin-top: 2px; }
  #panel-tabs { display: flex; border-bottom: 1px solid var(--border); flex-shrink: 0; }
  .panel-tab {
    flex: 1; padding: 11px; text-align: center; font-size: 13px; font-weight: 600;
    cursor: pointer; color: var(--text-muted); border-bottom: 2px solid transparent; transition: color 0.2s;
  }
  .panel-tab.active { color: var(--accent); border-bottom-color: var(--accent); }
  #panel-body { flex: 1; overflow-y: auto; padding: 14px; }

  /* PANEL SECTIONS */
  .panel-section { margin-bottom: 20px; }
  .panel-section h3 { font-size: 11px; font-weight: 700; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.8px; margin-bottom: 10px; }
  .setting-row {
    display: flex; align-items: center; justify-content: space-between;
    padding: 11px 13px; background: var(--surface); border-radius: var(--radius-sm);
    margin-bottom: 6px; cursor: pointer; border: 1px solid var(--border);
  }
  .setting-row span { font-size: 14px; }
  .setting-row .val { font-size: 13px; color: var(--accent); }
  .btn-full {
    width: 100%; padding: 11px; border-radius: var(--radius-sm);
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 14px; cursor: pointer; text-align: left;
    margin-bottom: 6px; display: flex; align-items: center; gap: 10px;
  }
  .btn-full:active { background: var(--surface-strong); }
  .btn-danger { color: var(--danger); border-color: rgba(255,107,107,0.3); }

  /* HISTORY LIST */
  .history-item {
    padding: 12px; background: var(--surface); border-radius: var(--radius-sm);
    margin-bottom: 8px; cursor: pointer; border: 1px solid var(--border);
  }
  .history-item:active { background: var(--surface-strong); }
  .hi-top { display: flex; justify-content: space-between; align-items: center; margin-bottom: 4px; }
  .hi-name { font-size: 14px; font-weight: 600; }
  .hi-bal { font-size: 14px; font-weight: 700; color: var(--accent); }
  .hi-meta { font-size: 11px; color: var(--text-muted); }

  /* JOURNAL VIEW */
  #journal-view {
    position: fixed; inset: 0; z-index: 20; display: flex; flex-direction: column;
    transform: translateX(100%); transition: transform 0.3s cubic-bezier(.4,0,.2,1);
  }
  #journal-view.open { transform: translateX(0); }
  #journal-bg { position: absolute; inset: 0; background-size: cover; background-position: center; background-color: var(--bg); }
  #journal-bg-overlay { position: absolute; inset: 0; background: rgba(0,0,0,0.55); }
  @media (prefers-color-scheme: light) { #journal-bg-overlay { background: rgba(255,255,255,0.45); } }
  #journal-content { position: relative; z-index: 1; display: flex; flex-direction: column; height: 100%; }

  /* JOURNAL TOPBAR */
  #journal-topbar {
    display: flex; align-items: center; gap: 10px;
    padding: 12px 14px 10px;
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border-bottom: 1px solid var(--border); flex-shrink: 0;
  }
  #journal-back {
    width: 36px; height: 36px; border-radius: 10px;
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 18px; cursor: pointer;
    display: flex; align-items: center; justify-content: center;
  }
  #journal-title-area { flex: 1; min-width: 0; }
  #journal-title-area h2 { font-size: 16px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  #journal-title-area p { font-size: 11px; color: var(--text-muted); }
  .journal-topbar-btns { display: flex; gap: 6px; }
  .jt-btn {
    width: 36px; height: 36px; border-radius: 10px;
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 15px; cursor: pointer;
    display: flex; align-items: center; justify-content: center;
  }

  /* JOURNAL HEADER CARD */
  #journal-header-card {
    margin: 14px 14px 0;
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border: 1px solid var(--border); border-radius: var(--radius);
    padding: 16px; flex-shrink: 0;
  }
  #jh-day { font-size: 13px; color: var(--text-muted); margin-bottom: 2px; }
  #jh-date { font-size: 20px; font-weight: 800; margin-bottom: 10px; }
  .jh-money-row { display: flex; gap: 8px; }
  .jh-money-block {
    flex: 1; background: var(--surface); border-radius: var(--radius-sm);
    padding: 9px 10px; border: 1px solid var(--border); min-width: 0;
  }
  .jh-money-label { font-size: 9px; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.5px; margin-bottom: 3px; }
  .jh-money-val { font-size: 15px; font-weight: 800; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .jh-money-val.positive { color: var(--success); }
  .jh-money-val.warn { color: #ffb347; }
  .jh-money-val.danger { color: var(--danger); }
  #jh-days-left { margin-top: 10px; font-size: 12px; color: var(--text-muted); text-align: center; }

  /* TRANSACTIONS */
  #transactions-list { flex: 1; overflow-y: auto; padding: 10px 14px 14px; }
  .tx-date-header {
    font-size: 11px; font-weight: 700; color: var(--text-muted);
    text-transform: uppercase; letter-spacing: 0.6px; margin: 12px 0 6px;
  }
  .tx-item {
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border: 1px solid var(--border); border-radius: var(--radius-sm);
    padding: 11px 12px; margin-bottom: 6px;
    display: flex; align-items: center; gap: 10px;
  }
  .tx-item.income-item { border-color: rgba(107,255,184,0.25); }
  .tx-icon {
    width: 34px; height: 34px; border-radius: 9px;
    background: var(--accent-soft); display: flex; align-items: center;
    justify-content: center; font-size: 15px; flex-shrink: 0;
  }
  .tx-icon.income-icon { background: rgba(107,255,184,0.12); }
  .tx-info { flex: 1; min-width: 0; }
  .tx-label { font-size: 14px; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .tx-note { font-size: 11px; color: var(--text-muted); }
  .tx-right { display: flex; align-items: center; gap: 8px; flex-shrink: 0; }
  .tx-amount { font-size: 14px; font-weight: 700; color: var(--danger); }
  .tx-amount.income { color: var(--success); }
  .tx-del {
    width: 26px; height: 26px; border-radius: 7px;
    background: rgba(255,107,107,0.1); border: 1px solid rgba(255,107,107,0.25);
    color: var(--danger); font-size: 13px; cursor: pointer;
    display: flex; align-items: center; justify-content: center;
  }

  /* ADD TRANSACTION BAR */
  #add-tx-bar {
    padding: 10px 14px 14px;
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border-top: 1px solid var(--border); flex-shrink: 0;
  }
  #add-tx-bar.locked { display: none; }
  #locked-notice {
    padding: 12px 14px; background: var(--panel-bg); backdrop-filter: var(--blur);
    border-top: 1px solid var(--border); flex-shrink: 0;
    text-align: center; font-size: 13px; color: var(--text-muted); display: none;
  }
  #locked-notice.show { display: block; }
  .tx-type-row { display: flex; gap: 6px; margin-bottom: 8px; }
  .tx-type-btn {
    flex: 1; height: 32px; border-radius: 8px; font-size: 12px; font-weight: 600;
    cursor: pointer; border: 1px solid var(--border); background: var(--surface); color: var(--text-muted);
    transition: all 0.15s;
  }
  .tx-type-btn.active-expense { background: rgba(255,107,107,0.15); border-color: rgba(255,107,107,0.4); color: var(--danger); }
  .tx-type-btn.active-income  { background: rgba(107,255,184,0.12); border-color: rgba(107,255,184,0.35); color: var(--success); }
  .tx-input-row { display: flex; gap: 8px; margin-bottom: 8px; }
  #tx-amount-input, #tx-label-input {
    height: 40px; border-radius: var(--radius-sm);
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 14px; padding: 0 12px; outline: none;
  }
  #tx-amount-input:focus, #tx-label-input:focus { border-color: var(--accent); }
  #tx-amount-input { width: 110px; }
  #tx-label-input { flex: 1; }
  #tx-amount-input::placeholder, #tx-label-input::placeholder { color: var(--text-muted); }
  #tx-submit-btn {
    width: 100%; height: 40px; border-radius: var(--radius-sm);
    background: var(--accent); border: none;
    color: #fff; font-size: 14px; font-weight: 700; cursor: pointer;
  }
  #tx-shorthand-hints { display: flex; gap: 6px; flex-wrap: wrap; margin-bottom: 8px; }
  .sh-hint {
    padding: 4px 10px; background: var(--accent-soft);
    border-radius: 20px; font-size: 12px; color: var(--accent); cursor: pointer;
    border: 1px solid rgba(124,159,255,0.3);
  }

  /* MODALS */
  #modal-overlay {
    position: fixed; inset: 0; z-index: 50;
    background: rgba(0,0,0,0.6);
    display: none; align-items: flex-end; justify-content: center;
  }
  #modal-overlay.open { display: flex; }
  .modal-sheet {
    width: 100%; max-width: 500px;
    background: var(--panel-bg); backdrop-filter: var(--blur);
    border-top-left-radius: 24px; border-top-right-radius: 24px;
    border: 1px solid var(--border); border-bottom: none;
    padding: 20px 20px 36px; max-height: 90vh; overflow-y: auto;
  }
  .modal-handle { width: 40px; height: 4px; background: var(--border); border-radius: 4px; margin: 0 auto 18px; }
  .modal-title { font-size: 18px; font-weight: 800; margin-bottom: 16px; }
  .modal-label { font-size: 12px; color: var(--text-muted); margin-bottom: 6px; text-transform: uppercase; letter-spacing: 0.5px; }
  .modal-input {
    width: 100%; height: 44px; border-radius: var(--radius-sm);
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 15px; padding: 0 14px;
    outline: none; margin-bottom: 14px;
    -webkit-appearance: none; appearance: none;
  }
  .modal-input:focus { border-color: var(--accent); }
  .modal-btn {
    width: 100%; height: 46px; border-radius: var(--radius-sm);
    background: var(--accent); border: none;
    color: #fff; font-size: 15px; font-weight: 700; cursor: pointer; margin-top: 4px;
  }
  .modal-btn-sec {
    width: 100%; height: 46px; border-radius: var(--radius-sm);
    background: var(--surface); border: 1px solid var(--border);
    color: var(--text); font-size: 15px; cursor: pointer; margin-top: 8px;
  }
  .modal-row { display: flex; gap: 10px; }
  .modal-row .modal-input { flex: 1; }
  .modal-slider { width: 100%; margin-bottom: 14px; accent-color: var(--accent); }

  /* SHORTHAND TABLE */
  .sh-table { width: 100%; border-collapse: collapse; margin-bottom: 14px; }
  .sh-table th { font-size: 11px; color: var(--text-muted); text-align: left; padding: 6px 8px; text-transform: uppercase; }
  .sh-table td { padding: 8px; font-size: 14px; }
  .sh-table tr { border-bottom: 1px solid var(--border); }
  .sh-del { background: none; border: none; color: var(--danger); cursor: pointer; font-size: 16px; padding: 4px; }

  /* HIDDEN */
  #file-bg-input, #file-journal-img-input { display: none; }

  /* TOAST */
  #toast {
    position: fixed; bottom: 90px; left: 50%; transform: translateX(-50%) translateY(20px);
    background: var(--surface-strong); backdrop-filter: var(--blur);
    border: 1px solid var(--border); border-radius: 20px;
    padding: 10px 20px; font-size: 13px; z-index: 100;
    opacity: 0; transition: opacity 0.2s, transform 0.2s; white-space: nowrap;
  }
  #toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

  ::-webkit-scrollbar { width: 3px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
</style>
</head>
<body>

<div id="bg-layer"></div>
<div id="bg-overlay"></div>

<div id="app">
  <div id="topbar">
    <button id="menu-btn" onclick="togglePanel()">☰</button>
    <input id="search-bar" type="text" placeholder="Search journals…" oninput="filterJournals(this.value)">
    <button id="new-journal-btn" onclick="openModal('new-journal')">+ New</button>
  </div>
  <div id="main">
    <div id="empty-state">
      <div class="icon">📒</div>
      <h2>No journals yet</h2>
      <p>Tap <strong>+ New</strong> to create your first monthly tracker.</p>
    </div>
    <div id="journals-grid"></div>
  </div>
</div>

<!-- SIDE PANEL -->
<div id="panel-overlay" onclick="togglePanel()"></div>
<div id="side-panel">
  <div id="panel-header">
    <h1>Vault</h1>
    <p>Personal finance tracker</p>
  </div>
  <div id="panel-tabs">
    <div class="panel-tab active" onclick="switchPanelTab('history')">History</div>
    <div class="panel-tab" onclick="switchPanelTab('settings')">Settings</div>
    <div class="panel-tab" onclick="switchPanelTab('dict')">Dictionary</div>
  </div>
  <div id="panel-body">
    <div id="tab-history">
      <div id="panel-history-list"><p style="color:var(--text-muted);font-size:13px;">No journals yet.</p></div>
    </div>
    <div id="tab-settings" style="display:none">
      <div class="panel-section">
        <h3>Appearance</h3>
        <div class="setting-row" onclick="openModal('bg-image')">
          <span>🖼 Background image</span><span class="val">Change</span>
        </div>
        <div class="setting-row" onclick="openModal('text-size')">
          <span>🔤 Text size</span><span class="val" id="set-text-size">16px</span>
        </div>
        <div class="setting-row" onclick="openModal('panel-opacity')">
          <span>💧 Panel opacity</span><span class="val" id="set-opacity">82%</span>
        </div>
      </div>
      <div class="panel-section">
        <h3>Data</h3>
        <button class="btn-full" onclick="exportData()">📤 Export all data</button>
        <button class="btn-full" onclick="document.getElementById('import-input').click()">📥 Import data</button>
        <input type="file" id="import-input" accept=".json" style="display:none" onchange="importData(event)">
        <button class="btn-full btn-danger" onclick="confirmClearAll()">🗑 Clear all data</button>
      </div>
    </div>
    <div id="tab-dict" style="display:none">
      <div class="panel-section">
        <h3>Shorthand Dictionary</h3>
        <p style="font-size:12px;color:var(--text-muted);margin-bottom:12px;">Type a shorthand in transactions. e.g. "D" → "Data".</p>
        <table class="sh-table">
          <thead><tr><th>Short</th><th>Meaning</th><th></th></tr></thead>
          <tbody id="sh-table-body"></tbody>
        </table>
        <button class="btn-full" onclick="openModal('add-shorthand')">+ Add shorthand</button>
      </div>
    </div>
  </div>
</div>

<!-- JOURNAL VIEW -->
<div id="journal-view">
  <div id="journal-bg"></div>
  <div id="journal-bg-overlay"></div>
  <div id="journal-content">
    <div id="journal-topbar">
      <button id="journal-back" onclick="closeJournal()">←</button>
      <div id="journal-title-area">
        <h2 id="journal-title-text">Journal</h2>
        <p id="journal-subtitle-text"></p>
      </div>
      <div class="journal-topbar-btns">
        <button class="jt-btn" id="journal-img-btn" onclick="triggerJournalImg()" title="Change image">🖼</button>
        <button class="jt-btn" id="journal-delete-btn" onclick="deleteCurrentJournal()" title="Delete journal">🗑</button>
      </div>
    </div>
    <div id="journal-header-card">
      <div id="jh-day"></div>
      <div id="jh-date"></div>
      <div class="jh-money-row">
        <div class="jh-money-block">
          <div class="jh-money-label">Budget</div>
          <div class="jh-money-val" id="jh-total">—</div>
        </div>
        <div class="jh-money-block">
          <div class="jh-money-label">Spent</div>
          <div class="jh-money-val danger" id="jh-spent">—</div>
        </div>
        <div class="jh-money-block">
          <div class="jh-money-label">Left</div>
          <div class="jh-money-val" id="jh-remaining">—</div>
        </div>
      </div>
      <div id="jh-days-left"></div>
    </div>
    <div id="transactions-list"></div>
    <div id="add-tx-bar">
      <div class="tx-type-row">
        <button class="tx-type-btn active-expense" id="btn-expense" onclick="setTxType('expense')">− Expense</button>
        <button class="tx-type-btn" id="btn-income" onclick="setTxType('income')">+ Top-up</button>
      </div>
      <div id="tx-shorthand-hints"></div>
      <div class="tx-input-row">
        <input id="tx-amount-input" type="number" inputmode="decimal" placeholder="Amount" onkeydown="txEnter(event)">
        <input id="tx-label-input" type="text" placeholder="Label (D, food, rent…)" onkeydown="txEnter(event)">
      </div>
      <button id="tx-submit-btn" onclick="addTransaction()">Add Expense</button>
    </div>
    <div id="locked-notice">🔒 This journal is locked — month ended.</div>
  </div>
</div>

<!-- MODALS -->
<div id="modal-overlay" onclick="handleModalOverlayClick(event)">
  <div class="modal-sheet" id="modal-sheet">
    <div class="modal-handle"></div>
    <div id="modal-content"></div>
  </div>
</div>

<input type="file" id="file-bg-input" accept="image/*" onchange="handleBgImage(event)">
<input type="file" id="file-journal-img-input" accept="image/*" onchange="handleJournalImg(event)">
<div id="toast"></div>

<script>
// ─── STATE ───────────────────────────────────────────────────────────────────
let state = {
  journals: [],
  shorthands: [],
  settings: { bgImageb64: null, textSize: 16, panelOpacity: 0.82 }
};
let currentJournalId = null;
let currentTxType = 'expense';

// ─── PERSIST ─────────────────────────────────────────────────────────────────
function save() {
  try { localStorage.setItem('vault_state', JSON.stringify(state)); }
  catch(e) { toast('⚠️ Storage full — export your data!'); }
}
function load() {
  try { const r = localStorage.getItem('vault_state'); if (r) state = JSON.parse(r); } catch(e) {}
}

// ─── INIT ─────────────────────────────────────────────────────────────────────
function init() {
  load(); applySettings(); checkAutoLock();
  renderHome(); renderPanelHistory(); renderShorthandTable();
}

function applySettings() {
  const s = state.settings;
  document.getElementById('bg-layer').style.backgroundImage = s.bgImageb64 ? `url(${s.bgImageb64})` : 'none';
  document.getElementById('set-text-size').textContent = s.textSize + 'px';
  document.getElementById('set-opacity').textContent = Math.round(s.panelOpacity * 100) + '%';
  const dark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  document.documentElement.style.setProperty(
    '--panel-bg', `rgba(${dark ? '15,15,15' : '240,240,240'},${s.panelOpacity})`
  );
}

// ─── DATE HELPERS ─────────────────────────────────────────────────────────────
const MONTHS = ['January','February','March','April','May','June','July','August','September','October','November','December'];
const DAYS   = ['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'];
function daysInMonth(m, y) { return new Date(y, m + 1, 0).getDate(); }
function today() { return new Date(); }
function formatDate(d) {
  return String(d.getMonth()+1).padStart(2,'0') + '/' + String(d.getDate()).padStart(2,'0') + '/' + d.getFullYear();
}

// ─── AUTO LOCK ────────────────────────────────────────────────────────────────
function checkAutoLock() {
  const now = today();
  state.journals.forEach(j => {
    if (!j.locked && now > new Date(j.year, j.month + 1, 0)) j.locked = true;
  });
  save();
}

// ─── HOME ─────────────────────────────────────────────────────────────────────
function renderHome(filter = '') {
  const grid = document.getElementById('journals-grid');
  const empty = document.getElementById('empty-state');
  let list = state.journals.slice().reverse();
  if (filter) {
    const f = filter.toLowerCase();
    list = list.filter(j =>
      `${MONTHS[j.month]} ${j.year}`.toLowerCase().includes(f) ||
      j.transactions.some(t => t.label.toLowerCase().includes(f))
    );
  }
  if (!list.length) { empty.style.display = 'flex'; grid.innerHTML = ''; return; }
  empty.style.display = 'none';
  grid.innerHTML = list.map(j => {
    const spent = j.transactions.filter(t=>t.type!=='income').reduce((a,t)=>a+t.amount,0);
    const topups = j.transactions.filter(t=>t.type==='income').reduce((a,t)=>a+t.amount,0);
    const total = j.budget + topups;
    const rem = total - spent;
    const pct = total > 0 ? Math.min(100, (spent/total)*100) : 0;
    const col = rem < 0 ? 'danger' : rem < total * 0.2 ? 'warn' : 'success';
    const bgStyle = j.imageb64 ? `style="background-image:url(${j.imageb64})"` : '';
    return `<div class="journal-card" onclick="openJournal('${j.id}')">
      <div class="card-bg" ${bgStyle}></div>
      <div class="card-content">
        <div class="card-top">
          <span class="card-month">${MONTHS[j.month]} ${j.year}</span>
          <span class="card-status ${j.locked?'status-locked':'status-active'}">${j.locked?'🔒 Locked':'● Active'}</span>
        </div>
        <div class="card-balance" style="color:var(--${col})">${rem.toLocaleString()} ${j.currency}</div>
        <div class="card-meta">Budget: ${total.toLocaleString()} ${j.currency} · ${j.transactions.length} entries</div>
        <div class="card-progress"><div class="card-progress-fill" style="width:${pct}%"></div></div>
      </div>
    </div>`;
  }).join('');
}
function filterJournals(v) { renderHome(v); }

// ─── PANEL ────────────────────────────────────────────────────────────────────
function togglePanel() {
  document.getElementById('side-panel').classList.toggle('open');
  document.getElementById('panel-overlay').classList.toggle('open');
}
function switchPanelTab(tab) {
  document.querySelectorAll('.panel-tab').forEach((el,i) => {
    el.classList.toggle('active', ['history','settings','dict'][i] === tab);
  });
  document.getElementById('tab-history').style.display  = tab==='history'  ? '' : 'none';
  document.getElementById('tab-settings').style.display = tab==='settings' ? '' : 'none';
  document.getElementById('tab-dict').style.display     = tab==='dict'     ? '' : 'none';
}
function renderPanelHistory() {
  const el = document.getElementById('panel-history-list');
  if (!state.journals.length) { el.innerHTML = '<p style="color:var(--text-muted);font-size:13px;">No journals yet.</p>'; return; }
  el.innerHTML = state.journals.slice().reverse().map(j => {
    const spent = j.transactions.filter(t=>t.type!=='income').reduce((a,t)=>a+t.amount,0);
    const topups = j.transactions.filter(t=>t.type==='income').reduce((a,t)=>a+t.amount,0);
    const rem = j.budget + topups - spent;
    return `<div class="history-item" onclick="openJournal('${j.id}');togglePanel();">
      <div class="hi-top">
        <span class="hi-name">${MONTHS[j.month]} ${j.year}</span>
        <span class="hi-bal">${rem.toLocaleString()} ${j.currency}</span>
      </div>
      <div class="hi-meta">${j.transactions.length} entries · ${j.locked?'Locked':'Active'}</div>
    </div>`;
  }).join('');
}

// ─── SHORTHAND ────────────────────────────────────────────────────────────────
function renderShorthandTable() {
  const tbody = document.getElementById('sh-table-body');
  if (!state.shorthands.length) {
    tbody.innerHTML = '<tr><td colspan="3" style="color:var(--text-muted);font-size:13px;padding:8px;">None yet.</td></tr>';
    return;
  }
  tbody.innerHTML = state.shorthands.map((s,i) =>
    `<tr><td><strong>${s.key}</strong></td><td>${s.meaning}</td>
     <td><button class="sh-del" onclick="deleteShorthand(${i})">✕</button></td></tr>`
  ).join('');
}
function deleteShorthand(i) {
  state.shorthands.splice(i,1); save(); renderShorthandTable();
  if (currentJournalId) renderShorthandHints();
}
function expandShorthand(label) {
  const sh = state.shorthands.find(s => s.key.toLowerCase()===label.toLowerCase());
  return sh ? sh.meaning : label;
}
function renderShorthandHints() {
  const el = document.getElementById('tx-shorthand-hints');
  el.innerHTML = state.shorthands.map(s =>
    `<span class="sh-hint" onclick="document.getElementById('tx-label-input').value='${s.key}'">${s.key}=${s.meaning}</span>`
  ).join('');
}

// ─── TX TYPE ──────────────────────────────────────────────────────────────────
function setTxType(type) {
  currentTxType = type;
  document.getElementById('btn-expense').className = 'tx-type-btn' + (type==='expense' ? ' active-expense' : '');
  document.getElementById('btn-income').className  = 'tx-type-btn' + (type==='income'  ? ' active-income'  : '');
  document.getElementById('tx-submit-btn').textContent = type==='expense' ? 'Add Expense' : 'Add Top-up';
}
function txEnter(e) { if (e.key==='Enter') addTransaction(); }

// ─── JOURNAL OPEN/CLOSE ───────────────────────────────────────────────────────
function openJournal(id) {
  currentJournalId = id;
  const j = state.journals.find(x => x.id===id);
  if (!j) return;

  // BG
  const jBg = document.getElementById('journal-bg');
  if (j.imageb64) { jBg.style.backgroundImage = `url(${j.imageb64})`; jBg.style.backgroundColor = ''; }
  else { jBg.style.backgroundImage = 'none'; jBg.style.backgroundColor = 'var(--bg)'; }

  // TITLE
  document.getElementById('journal-title-text').textContent = `${MONTHS[j.month]} ${j.year}`;
  document.getElementById('journal-subtitle-text').textContent = j.locked ? '🔒 Read-only' : 'Active';

  // HEADER
  const now = today();
  const isCurrent = now.getMonth()===j.month && now.getFullYear()===j.year;
  const dayNum = isCurrent ? now.getDate() : daysInMonth(j.month, j.year);
  const dateObj = isCurrent ? now : new Date(j.year, j.month, daysInMonth(j.month, j.year));
  document.getElementById('jh-day').textContent  = `Day ${dayNum} — ${DAYS[dateObj.getDay()]}`;
  document.getElementById('jh-date').textContent = formatDate(dateObj);

  const spent  = j.transactions.filter(t=>t.type!=='income').reduce((a,t)=>a+t.amount,0);
  const topups = j.transactions.filter(t=>t.type==='income').reduce((a,t)=>a+t.amount,0);
  const total  = j.budget + topups;
  const rem    = total - spent;

  document.getElementById('jh-total').textContent = `${total.toLocaleString()} ${j.currency}`;
  document.getElementById('jh-spent').textContent = `${spent.toLocaleString()} ${j.currency}`;
  const remEl = document.getElementById('jh-remaining');
  remEl.textContent = `${rem.toLocaleString()} ${j.currency}`;
  remEl.className = 'jh-money-val ' + (rem<0 ? 'danger' : rem<total*0.2 ? 'warn' : 'positive');

  const totalDays = daysInMonth(j.month, j.year);
  const daysLeft  = j.locked ? 0 : (totalDays - (isCurrent ? now.getDate() : totalDays));
  const startDay  = j.startDay || 1;
  document.getElementById('jh-days-left').textContent = j.locked
    ? `Month ended · ${totalDays - startDay + 1} days tracked`
    : `${daysLeft} day${daysLeft!==1?'s':''} left this month`;

  renderTransactions(j);

  // LOCK STATE
  const locked = j.locked;
  document.getElementById('add-tx-bar').classList.toggle('locked', locked);
  document.getElementById('locked-notice').classList.toggle('show', locked);
  document.getElementById('journal-img-btn').style.display    = locked ? 'none' : '';
  document.getElementById('journal-delete-btn').style.display = locked ? 'none' : '';
  if (!locked) { setTxType('expense'); renderShorthandHints(); }

  document.getElementById('journal-view').classList.add('open');
}

function closeJournal() {
  document.getElementById('journal-view').classList.remove('open');
  currentJournalId = null;
  renderHome(); renderPanelHistory();
}

function deleteCurrentJournal() {
  if (!currentJournalId) return;
  const j = state.journals.find(x=>x.id===currentJournalId);
  if (!j) return;
  if (!confirm(`Delete "${MONTHS[j.month]} ${j.year}" journal? This cannot be undone.`)) return;
  state.journals = state.journals.filter(x=>x.id!==currentJournalId);
  save(); closeJournal(); toast('Journal deleted');
}

function renderTransactions(j) {
  const list = document.getElementById('transactions-list');
  if (!j.transactions.length) {
    list.innerHTML = '<p style="text-align:center;color:var(--text-muted);font-size:13px;padding:30px 0;">No entries yet.</p>';
    return;
  }
  const groups = {};
  j.transactions.forEach(t => { (groups[t.date] = groups[t.date]||[]).push(t); });
  const isUnlocked = !j.locked;
  list.innerHTML = Object.entries(groups).map(([date, txs]) =>
    `<div class="tx-date-header">${date}</div>` +
    txs.map(t => {
      const isIncome = t.type === 'income';
      const expanded = expandShorthand(t.label);
      const delBtn = isUnlocked
        ? `<button class="tx-del" onclick="deleteTransaction('${j.id}','${t.id}')">✕</button>` : '';
      return `<div class="tx-item${isIncome?' income-item':''}">
        <div class="tx-icon${isIncome?' income-icon':''}">${isIncome?'💰':'💸'}</div>
        <div class="tx-info">
          <div class="tx-label">${expanded}</div>
          ${expanded!==t.label ? `<div class="tx-note">(${t.label})</div>` : ''}
        </div>
        <div class="tx-right">
          <div class="tx-amount${isIncome?' income':''}">${isIncome?'+':'−'}${t.amount.toLocaleString()} ${j.currency}</div>
          ${delBtn}
        </div>
      </div>`;
    }).join('')
  ).join('');
}

function deleteTransaction(journalId, txId) {
  const j = state.journals.find(x=>x.id===journalId);
  if (!j || j.locked) return;
  j.transactions = j.transactions.filter(t=>t.id!==txId);
  save(); openJournal(journalId);
}

function addTransaction() {
  if (!currentJournalId) return;
  const j = state.journals.find(x=>x.id===currentJournalId);
  if (!j || j.locked) return;
  const amountRaw = document.getElementById('tx-amount-input').value.trim();
  const label     = document.getElementById('tx-label-input').value.trim();
  if (!amountRaw || !label) { toast('Enter amount and label'); return; }
  const amount = parseFloat(amountRaw);
  if (isNaN(amount) || amount <= 0) { toast('Invalid amount'); return; }
  j.transactions.push({ id: Date.now().toString(), amount, label, type: currentTxType, date: formatDate(today()) });
  save();
  document.getElementById('tx-amount-input').value = '';
  document.getElementById('tx-label-input').value  = '';
  openJournal(currentJournalId);
}

// ─── NEW JOURNAL ──────────────────────────────────────────────────────────────
function createJournal(month, year, budget, currency) {
  const now = today();
  const startDay = (now.getMonth()===month && now.getFullYear()===year) ? now.getDate() : 1;
  const id = `j_${Date.now()}`;
  state.journals.push({ id, month, year, startDay, budget, currency, transactions: [], imageb64: null, locked: false });
  save(); renderHome(); renderPanelHistory(); closeModal();
  toast(`${MONTHS[month]} ${year} created`);
  openJournal(id);
}

// ─── IMAGES ───────────────────────────────────────────────────────────────────
function handleBgImage(e) {
  const file = e.target.files[0]; if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => {
    state.settings.bgImageb64 = ev.target.result; save(); applySettings();
    document.getElementById('bg-layer').style.backgroundImage = `url(${ev.target.result})`;
    toast('Background updated');
  };
  reader.readAsDataURL(file); e.target.value = '';
}
function handleJournalImg(e) {
  const file = e.target.files[0]; if (!file || !currentJournalId) return;
  const reader = new FileReader();
  reader.onload = ev => {
    const j = state.journals.find(x=>x.id===currentJournalId);
    if (j) { j.imageb64 = ev.target.result; save(); openJournal(currentJournalId); toast('Image updated'); }
  };
  reader.readAsDataURL(file); e.target.value = '';
}
function triggerJournalImg() { document.getElementById('file-journal-img-input').click(); }

// ─── MODALS ───────────────────────────────────────────────────────────────────
function openModal(type) {
  const content = document.getElementById('modal-content');
  if (type === 'new-journal') {
    const now = today();
    content.innerHTML = `
      <div class="modal-title">New Journal</div>
      <div class="modal-label">Month & Year</div>
      <div class="modal-row">
        <select class="modal-input" id="nj-month" style="color:var(--text);background:var(--surface)">
          ${MONTHS.map((m,i)=>`<option value="${i}" ${i===now.getMonth()?'selected':''}>${m}</option>`).join('')}
        </select>
        <input class="modal-input" id="nj-year" type="number" value="${now.getFullYear()}" style="width:100px">
      </div>
      <div class="modal-label">Budget</div>
      <input class="modal-input" id="nj-budget" type="number" inputmode="decimal" placeholder="e.g. 5000">
      <div class="modal-label">Currency (e.g. USD, ZAR, NGN)</div>
      <input class="modal-input" id="nj-currency" type="text" placeholder="USD" maxlength="10">
      <button class="modal-btn" onclick="submitNewJournal()">Create Journal</button>
      <button class="modal-btn-sec" onclick="closeModal()">Cancel</button>`;
  } else if (type === 'add-shorthand') {
    content.innerHTML = `
      <div class="modal-title">Add Shorthand</div>
      <div class="modal-label">Key (e.g. D)</div>
      <input class="modal-input" id="sh-key" type="text" placeholder="D" maxlength="10">
      <div class="modal-label">Meaning (e.g. Data)</div>
      <input class="modal-input" id="sh-meaning" type="text" placeholder="Data">
      <button class="modal-btn" onclick="submitShorthand()">Save</button>
      <button class="modal-btn-sec" onclick="closeModal()">Cancel</button>`;
  } else if (type === 'bg-image') {
    content.innerHTML = `
      <div class="modal-title">Background Image</div>
      <p style="font-size:13px;color:var(--text-muted);margin-bottom:16px;">Pick an image from your device for the app background.</p>
      <button class="modal-btn" onclick="document.getElementById('file-bg-input').click();closeModal();">Choose from device</button>
      ${state.settings.bgImageb64 ? `<button class="modal-btn-sec" onclick="removeBg()">Remove background</button>` : ''}
      <button class="modal-btn-sec" onclick="closeModal()">Cancel</button>`;
  } else if (type === 'text-size') {
    content.innerHTML = `
      <div class="modal-title">Text Size</div>
      <div class="modal-label">Size: <span id="ts-val">${state.settings.textSize}px</span></div>
      <input class="modal-slider" type="range" min="12" max="22" value="${state.settings.textSize}"
        oninput="document.getElementById('ts-val').textContent=this.value+'px'" id="ts-slider">
      <button class="modal-btn" onclick="saveTextSize()">Save</button>
      <button class="modal-btn-sec" onclick="closeModal()">Cancel</button>`;
  } else if (type === 'panel-opacity') {
    const pct = Math.round(state.settings.panelOpacity * 100);
    content.innerHTML = `
      <div class="modal-title">Panel Opacity</div>
      <div class="modal-label">Opacity: <span id="op-val">${pct}%</span></div>
      <input class="modal-slider" type="range" min="40" max="98" value="${pct}"
        oninput="document.getElementById('op-val').textContent=this.value+'%'" id="op-slider">
      <button class="modal-btn" onclick="savePanelOpacity()">Save</button>
      <button class="modal-btn-sec" onclick="closeModal()">Cancel</button>`;
  }
  document.getElementById('modal-overlay').classList.add('open');
}
function closeModal() { document.getElementById('modal-overlay').classList.remove('open'); }
function handleModalOverlayClick(e) { if (e.target===document.getElementById('modal-overlay')) closeModal(); }

function submitNewJournal() {
  const month    = parseInt(document.getElementById('nj-month').value);
  const year     = parseInt(document.getElementById('nj-year').value);
  const budget   = parseFloat(document.getElementById('nj-budget').value);
  const currency = (document.getElementById('nj-currency').value.trim().toUpperCase()) || 'USD';
  if (isNaN(budget)||budget<=0) { toast('Enter a valid budget'); return; }
  if (isNaN(year)||year<2000)   { toast('Enter a valid year'); return; }
  if (state.journals.find(j=>j.month===month&&j.year===year)) { toast('Journal for this month already exists'); return; }
  createJournal(month, year, budget, currency);
}
function submitShorthand() {
  const key     = document.getElementById('sh-key').value.trim();
  const meaning = document.getElementById('sh-meaning').value.trim();
  if (!key||!meaning) { toast('Fill both fields'); return; }
  const i = state.shorthands.findIndex(s=>s.key.toLowerCase()===key.toLowerCase());
  if (i>=0) state.shorthands[i].meaning = meaning; else state.shorthands.push({key,meaning});
  save(); renderShorthandTable(); closeModal();
  if (currentJournalId) renderShorthandHints();
  toast(`Saved: ${key} → ${meaning}`);
}
function saveTextSize() {
  state.settings.textSize = parseInt(document.getElementById('ts-slider').value);
  save(); applySettings(); closeModal();
}
function savePanelOpacity() {
  state.settings.panelOpacity = parseInt(document.getElementById('op-slider').value)/100;
  save(); applySettings(); closeModal();
}
function removeBg() {
  state.settings.bgImageb64 = null; save();
  document.getElementById('bg-layer').style.backgroundImage = 'none';
  closeModal(); toast('Background removed');
}

// ─── DATA ─────────────────────────────────────────────────────────────────────
function exportData() {
  const blob = new Blob([JSON.stringify(state,null,2)], {type:'application/json'});
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement('a');
  a.href = url; a.download = `vault-backup-${formatDate(today()).replace(/\//g,'-')}.json`;
  a.click(); URL.revokeObjectURL(url); toast('Data exported');
}
function importData(e) {
  const file = e.target.files[0]; if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => {
    try {
      const imp = JSON.parse(ev.target.result);
      if (!imp.journals) throw new Error();
      state = imp; save(); applySettings(); checkAutoLock();
      renderHome(); renderPanelHistory(); renderShorthandTable();
      toast('Data imported');
    } catch { toast('Invalid backup file'); }
  };
  reader.readAsText(file); e.target.value = '';
}
function confirmClearAll() {
  if (confirm('Delete ALL journals and settings? Cannot be undone.')) {
    state = {journals:[],shorthands:[],settings:{bgImageb64:null,textSize:16,panelOpacity:0.82}};
    save(); applySettings(); renderHome(); renderPanelHistory(); renderShorthandTable();
    toast('All data cleared');
  }
}

// ─── TOAST ────────────────────────────────────────────────────────────────────
let toastTimer;
function toast(msg) {
  const el = document.getElementById('toast');
  el.textContent = msg; el.classList.add('show');
  clearTimeout(toastTimer);
  toastTimer = setTimeout(()=>el.classList.remove('show'), 2500);
}

init();
</script>
</body>
</html>
