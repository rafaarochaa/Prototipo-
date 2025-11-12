<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Lotérica Caixas — App</title>
  <style>
    :root{
      --bg:#fff0f6; --card:#fff; --accent:#6fd6ff; --muted:#6b6b6b; --glass:rgba(255,255,255,0.8); --shadow:0 8px 28px rgba(0,0,0,0.08);
      --radius:14px; font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, Arial;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#fff);color:#222}
    .app{max-width:1100px;margin:26px auto;padding:20px}
    header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:18px}
    header h1{font-size:20px;margin:0}
    .top-actions{display:flex;gap:8px;align-items:center}
    .btn{background:var(--accent);color:#fff;padding:8px 12px;border-radius:10px;border:none;cursor:pointer;box-shadow:var(--shadow);font-weight:600}
    .btn.ghost{background:transparent;color:var(--accent);border:1px solid rgba(255,111,163,0.18)}
    .layout{display:grid;grid-template-columns:260px 1fr;gap:18px}
    nav{background:var(--card);padding:14px;border-radius:var(--radius);box-shadow:var(--shadow)}
    nav button{display:block;width:100%;text-align:left;padding:10px;border-radius:10px;border:none;background:transparent;cursor:pointer;margin-bottom:6px}
    nav button.active{background:linear-gradient(90deg,rgba(255,111,163,0.12),rgba(255,111,163,0.06));font-weight:700}
    main{background:var(--card);padding:18px;border-radius:var(--radius);box-shadow:var(--shadow);min-height:560px}
    .card{background:var(--glass);padding:14px;border-radius:12px;margin-bottom:12px}
    label{display:block;font-size:13px;margin-bottom:6px;color:var(--muted)}
    input[type=text], input[type=number], select, textarea{width:100%;padding:10px;border-radius:8px;border:1px solid #f6d7e3;background:transparent}
    table{width:100%;border-collapse:collapse}
    th,td{padding:8px;border-bottom:1px solid #f4dfe8;font-size:13px}
    .muted{color:var(--muted);font-size:13px}
    .pill{display:inline-block;padding:6px 8px;border-radius:999px;background:rgba(255,111,163,0.12);color:var(--accent);font-weight:600}
    .row{display:flex;gap:10px}
    .col{flex:1}
    footer{margin-top:14px;text-align:center;color:var(--muted);font-size:13px}
    @media (max-width:900px){.layout{grid-template-columns:1fr}.app{padding:12px}}

    /* Modal PIN */
    .overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:linear-gradient(0deg,rgba(0,0,0,0.35),rgba(0,0,0,0.2));}
    .modal{background:var(--card);padding:20px;border-radius:12px;width:360px;box-shadow:0 10px 40px rgba(0,0,0,0.25)}
    .pin-inputs{display:flex;gap:10px;justify-content:center;margin:12px 0}
    .pin-inputs input{width:56px;height:56px;font-size:24px;text-align:center;border-radius:10px;border:1px solid #f0d6e1}

    /* admin badge */
    .admin-badge{background:#ffe6f0;color:var(--accent);padding:6px 8px;border-radius:8px;font-weight:700}
  </style>
</head>
<body>
  <div class="app">
    <header>
      <h1>Lotérica Caixas</h1>
      <div class="top-actions">
        <div class="muted small">Usuário: <span id="user-name">—</span></div>
        <button class="btn" id="open-profile">Perfil</button>
      </div>
    </header>

    <div class="layout">
      <nav>
        <button class="active" data-tab="dashboard">Dashboard</button>
        <button data-tab="cliente">Cliente (Perfil)</button>
        <button data-tab="jogos">Jogos da Sorte</button>
        <button data-tab="transacoes">Histórico / Pagamentos</button>
        <button data-tab="investir">Investir / Descontos</button>
        <button data-tab="admin">Painel Admin <span class="admin-badge">ADM</span></button>
        <button data-tab="config">Configurações</button>
      </nav>

      <main id="main-content"></main>
    </div>

    <footer>Design moderno • Fundo rosa claro • Demo — dados locais / opcional sync API</footer>
  </div>

  <!-- PIN modal -->
  <div class="overlay" id="pin-overlay" aria-hidden="false">
    <div class="modal" role="dialog" aria-modal="true">
      <h2>Entre com sua senha (4 dígitos)</h2>
      <p class="muted">Insira o PIN para acessar o app.</p>
      <div class="pin-inputs">
        <input maxlength="1" inputmode="numeric" pattern="[0-9]*" class="pin-digit" />
        <input maxlength="1" inputmode="numeric" pattern="[0-9]*" class="pin-digit" />
        <input maxlength="1" inputmode="numeric" pattern="[0-9]*" class="pin-digit" />
        <input maxlength="1" inputmode="numeric" pattern="[0-9]*" class="pin-digit" />
      </div>
      <div style="display:flex;gap:8px;justify-content:center">
        <button class="btn" id="btn-enter">Entrar</button>
        <button class="btn ghost" id="btn-reset">Resetar PIN</button>
      </div>
      <p class="muted small" style="text-align:center;margin-top:8px">PIN padrão: <strong>1234</strong> (troque nas configurações)</p>
    </div>
  </div>

<script>
/********************* Estado e persistência *********************/
const STORAGE_KEY = 'lot_caixas_v2';
let state = {
  user: { name: 'Cliente Demo', cpf: '', phone: '', email: '', private:true },
  pin: '1234',
  balance: 200.00,
  bets: [], // {id,type,numbers,amount,date,status,prize}
  payments: [], // {id,type,method,amount,date,details}
  investments: [],
  discounts: { loyalty: 0.05 },
  winningNumbers: {}, // game -> array
  admins: [{user:'admin', pass:'admin'}],
};

function load(){
  const raw = localStorage.getItem(STORAGE_KEY);
  if(raw){ try{ const parsed = JSON.parse(raw); state = Object.assign(state, parsed); } catch(e){console.error(e)} }
  document.getElementById('user-name').innerText = state.user.name || '—';
}
function save(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); }

/********************* Navegação / Router *********************/
const main = document.getElementById('main-content');
const navButtons = document.querySelectorAll('nav button');
navButtons.forEach(b=> b.addEventListener('click', ()=>{ navButtons.forEach(x=>x.classList.remove('active')); b.classList.add('active'); showTab(b.dataset.tab); }));
function showTab(tab){
  if(tab==='dashboard') return renderDashboard();
  if(tab==='cliente') return renderCliente();
  if(tab==='jogos') return renderJogos();
  if(tab==='transacoes') return renderTransacoes();
  if(tab==='investir') return renderInvestir();
  if(tab==='admin') return renderAdmin();
  if(tab==='config') return renderConfig();
}

/********************* Dashboard *********************/
function renderDashboard(){
  main.innerHTML = `
    <div class="card"><div style="display:flex;justify-content:space-between;align-items:center"><div><h3>Saldo disponível</h3><p class="muted">Seu saldo para apostas e saque</p></div><div class="pill">R$ ${state.balance.toFixed(2)}</div></div></div>
    <div class="card"><h3>Últimas apostas</h3><div id="last-bets"></div></div>
    <div class="card"><h3>Últimos pagamentos</h3><div id="last-payments"></div></div>
  `;
  renderLastBets(); renderLastPayments();
}
function renderLastBets(){ const el=document.getElementById('last-bets'); const last = state.bets.slice(-5).reverse(); if(!last.length) return el.innerHTML='<p class="muted">Nenhuma aposta.</p>'; el.innerHTML=`<table><thead><tr><th>Data</th><th>Jogo</th><th>Números</th><th>Valor</th><th>Status</th></tr></thead><tbody>${last.map(b=>`<tr><td>${new Date(b.date).toLocaleString()}</td><td>${b.type}</td><td>${b.numbers.join(', ')}</td><td>R$ ${b.amount.toFixed(2)}</td><td>${b.status}</td></tr>`).join('')}</tbody></table>`; }
function renderLastPayments(){ const el=document.getElementById('last-payments'); const last = state.payments.slice(-5).reverse(); if(!last.length) return el.innerHTML='<p class="muted">Nenhum pagamento.</p>'; el.innerHTML=`<table><thead><tr><th>Data</th><th>Tipo</th><th>Método</th><th>Valor</th></tr></thead><tbody>${last.map(p=>`<tr><td>${new Date(p.date).toLocaleString()}</td><td>${p.type}</td><td>${p.method||'-'}</td><td>R$ ${p.amount.toFixed(2)}</td></tr>`).join('')}</tbody></table>`; }

/********************* Cliente / Perfil *********************/
function renderCliente(){
  main.innerHTML = `
    <div class="card"><h3>Perfil</h3>
      <label>Nome</label><input id="cfg-name" value="${escapeHtml(state.user.name)}">
      <label>CPF</label><input id="cfg-cpf" value="${escapeHtml(state.user.cpf)}">
      <label>Telefone</label><input id="cfg-phone" value="${escapeHtml(state.user.phone)}">
      <label>Email</label><input id="cfg-email" value="${escapeHtml(state.user.email)}">
      <div style="margin-top:10px"><button class="btn" id="save-profile">Salvar perfil</button></div>
      <p class="muted small">As informações pessoais são privadas e ficam no seu navegador (local).</p>
    </div>
    <div class="card"><h3>Transações consolidadas</h3><div id="mini-trans"></div></div>
  `;
  document.getElementById('save-profile').addEventListener('click', ()=>{ state.user.name=document.getElementById('cfg-name').value||state.user.name; state.user.cpf=document.getElementById('cfg-cpf').value||state.user.cpf; state.user.phone=document.getElementById('cfg-phone').value||state.user.phone; state.user.email=document.getElementById('cfg-email').value||state.user.email; document.getElementById('user-name').innerText=state.user.name; save(); alert('Perfil salvo (local).'); });
  renderMiniTrans();
}
function renderMiniTrans(){ const el=document.getElementById('mini-trans'); const bets = state.bets.map(b=>({date:b.date,type:'Aposta',desc:`${b.type} — ${b.numbers.join(', ')}`,amount:-b.amount})); const pays = state.payments.map(p=>({date:p.date,type:p.type,desc:p.details||'-',amount:p.amount})); const all=bets.concat(pays).sort((a,b)=>new Date(b.date)-new Date(a.date)); if(!all.length) return el.innerHTML='<p class="muted">Nenhuma transação.</p>'; el.innerHTML=`<table><thead><tr><th>Data</th><th>Tipo</th><th>Descrição</th><th>Valor</th></tr></thead><tbody>${all.map(t=>`<tr><td>${new Date(t.date).toLocaleString()}</td><td>${t.type}</td><td>${t.desc}</td><td>${t.amount<0?`- R$ ${Math.abs(t.amount).toFixed(2)}`:`R$ ${t.amount.toFixed(2)}`}</td></tr>`).join('')}</tbody></table>`; }

/********************* Jogos da Sorte *********************/
const GAMES = {
  'Mega-Sena': { picks:6, max:60 },
  'Quina': { picks:5, max:80 },
  'Lotofácil': { picks:15, max:25 },
  'Lotomania': { picks:20, max:100 }
};

function renderJogos(){
  main.innerHTML = `
    <div class="card">
      <h3>Registrar Aposta</h3>
      <label>Tipo de Jogo</label>
      <select id="bet-type">${Object.keys(GAMES).map(g=>`<option>${g}</option>`).join('')}</select>
      <label style="margin-top:8px">Números (vírgula)</label>
      <input id="bet-numbers" placeholder="ex: 1,2,3...">
      <div class="row" style="margin-top:10px"><div class="col"><label>Valor (R$)</label><input id="bet-amount" type="number" value="5" min="1" step="0.5"></div><div class="col"><label>&nbsp;</label><button class="btn" id="place-bet">Registrar aposta</button></div></div>
      <p class="muted small">Valor será debitado do saldo.</p>
    </div>

    <div class="card">
      <h3>Inserir/Atualizar Números vencedores</h3>
      <label>Jogo</label>
      <select id="winning-game">${Object.keys(GAMES).map(g=>`<option>${g}</option>`).join('')}</select>
      <label style="margin-top:8px">Números vencedores (vírgula)</label>
      <input id="winning-nums" placeholder="ex: 1,2,3...">
      <div style="margin-top:10px"><button class="btn" id="save-winning">Salvar e validar apostas</button></div>
      <p class="muted small">Ao salvar, o app atualiza status das apostas e gera pagamentos para prêmios.</p>
    </div>

    <div class="card"><h3>Minhas apostas</h3><div id="bets-list"></div></div>
  `;
  document.getElementById('place-bet').addEventListener('click', placeBet);
  document.getElementById('save-winning').addEventListener('click', saveWinning);
  renderBetsList();
}

function placeBet(){
  const type = document.getElementById('bet-type').value;
  const raw = document.getElementById('bet-numbers').value;
  const nums = raw.split(',').map(s=>parseInt(s.trim())).filter(n=>Number.isFinite(n));
  const amount = parseFloat(document.getElementById('bet-amount').value)||0;
  const spec = GAMES[type];
  if(nums.length !== spec.picks){ alert(`O jogo ${type} requer ${spec.picks} números.`); return; }
  if(nums.some(n=>n<1 || n>spec.max)){ alert(`Números inválidos (1-${spec.max}).`); return; }
  if(amount<=0){ alert('Valor inválido'); return; }
  if(amount>state.balance){ alert('Saldo insuficiente'); return; }
  const bet = { id:'B'+Date.now(), type, numbers: nums, amount, date:new Date().toISOString(), status:'Em aberto', prize:0 };
  state.bets.push(bet);
  state.balance -= amount;
  state.payments.push({ id:'P'+Date.now(), type:'Compra Aposta', method:'Saldo', amount:-amount, date:new Date().toISOString(), details:`Aposta ${type}` });
  save(); renderBetsList(); renderDashboard(); alert('Aposta registrada.');
}

function renderBetsList(){ const el=document.getElementById('bets-list'); if(!state.bets.length) return el.innerHTML='<p class="muted">Sem apostas.</p>'; el.innerHTML=`<table><thead><tr><th>Data</th><th>Jogo</th><th>Números</th><th>Valor</th><th>Status</th><th>Ação</th></tr></thead><tbody>${state.bets.map(b=>`<tr><td>${new Date(b.date).toLocaleString()}</td><td>${b.type}</td><td>${b.numbers.join(', ')}</td><td>R$ ${b.amount.toFixed(2)}</td><td>${b.status}${b.prize?` — R$ ${b.prize.toFixed(2)}`:''}</td><td>${b.status==='Em aberto'?`<button class="btn ghost" onclick="cancelBet('${b.id}')">Cancelar</button>`:'-'}</td></tr>`).join('')}</tbody></table>`; }

window.cancelBet = function(id){ const idx=state.bets.findIndex(b=>b.id===id); if(idx===-1) return; if(!confirm('Cancelar aposta e estornar valor?')) return; const b=state.bets[idx]; state.bets.splice(idx,1); state.balance += b.amount; state.payments.push({ id:'P'+Date.now(), type:'Estorno', method:'Saldo', amount:b.amount, date:new Date().toISOString(), details:`Estorno ${b.type}` }); save(); renderBetsList(); renderDashboard(); }

/********************* Validar resultados e cálculo de prêmios *********************/
// Prize logic: more realistic fixed multipliers/tables per game
function prizeFor(type, stake, matched){
  // values are illustrative; in a real app should use official rules / pools
  const table = {
    'Mega-Sena': {6: 1000000,5: 50000,4: 1000}, // fixed prize values per R$1 stake approximation
    'Quina': {5: 500000,4: 10000,3: 200},
    'Lotofácil': {15: 300000,14: 2500,13: 200,12:50,11:10},
    'Lotomania': {20: 400000,19:5000,18:500,17:50,16:10,15:5}
  };
  const t = table[type] || {};
  const base = t[matched] || 0;
  // base represents prize for 1 unit stake; scale by stake
  return base * stake;
}

function saveWinning(){
  const game = document.getElementById('winning-game').value;
  const raw = document.getElementById('winning-nums').value;
  const arr = raw.split(',').map(s=>parseInt(s.trim())).filter(n=>Number.isFinite(n));
  const spec = GAMES[game];
  if(arr.length !== spec.picks){ if(!(game==='Lotofácil' && arr.length>=11 && arr.length<=15)) { alert(`Os números para ${game} devem ter ${spec.picks} (ou, para Lotofácil, 11-15).`); return; } }
  state.winningNumbers[game]=arr;
  let winners=0;
  state.bets.forEach(b=>{
    if(b.type!==game) return;
    if(b.status!=='Em aberto') return;
    const matched = b.numbers.filter(n=>arr.includes(n)).length;
    const prize = prizeFor(b.type, b.amount, matched);
    b.prize = prize;
    b.status = prize>0?`Vencedora (${matched} acertos)`:'Perdida';
    if(prize>0){ winners++; state.payments.push({ id:'W'+Date.now()+Math.floor(Math.random()*99), type:'Prêmio', method:'PIX', amount:prize, date:new Date().toISOString(), details:`Prêmio ${b.type} — ${matched} acertos` }); state.balance += prize; }
  });
  save(); renderBetsList(); renderDashboard(); alert(`Resultados processados. ${winners} apostas premiadas.`);
}

/********************* Transações / Pagamentos *********************/
function renderTransacoes(){
  main.innerHTML = `
    <div class="card"><h3>Histórico consolidado</h3><div id="tx-list"></div></div>
    <div class="card"><h3>Realizar depósito / pagamento</h3>
      <div class="row"><div class="col"><label>Método</label><select id="pay-method"><option>PIX</option><option>Cartão</option><option>Boleto</option><option>Saldo</option></select></div>
      <div class="col"><label>Valor</label><input id="pay-amount" type="number" min="0.5" step="0.5" value="10"></div>
      <div class="col"><label>&nbsp;</label><button class="btn" id="make-payment">Pagar / Adicionar</button></div></div>
      <p class="muted small">Cartão e Boleto aplicam pequenas taxas simuladas.</p>
    </div>
  `;
  document.getElementById('make-payment').addEventListener('click', makePayment);
  renderTxList();
}
function renderTxList(){ const el=document.getElementById('tx-list'); const bets = state.bets.map(b=>({date:b.date,type:'Aposta',desc:`${b.type} — ${b.numbers.join(', ')}`,amount:-b.amount})); const pays = state.payments.map(p=>({date:p.date,type:p.type,desc:p.details||'-',amount:p.amount,method:p.method})); const all=bets.concat(pays).sort((a,b)=>new Date(b.date)-new Date(a.date)); if(!all.length) return el.innerHTML='<p class="muted">Nenhuma transação.</p>'; el.innerHTML=`<table><thead><tr><th>Data</th><th>Tipo</th><th>Descrição</th><th>Valor</th></tr></thead><tbody>${all.map(t=>`<tr><td>${new Date(t.date).toLocaleString()}</td><td>${t.type}${t.method?` <span class="small">(${t.method})</span>`:''}</td><td>${t.desc}</td><td>${t.amount<0?`- R$ ${Math.abs(t.amount).toFixed(2)}`:`R$ ${t.amount.toFixed(2)}`}</td></tr>`).join('')}</tbody></table>`; }
function makePayment(){ const method=document.getElementById('pay-method').value; const amount=parseFloat(document.getElementById('pay-amount').value)||0; if(amount<=0){alert('Valor inválido');return;} let net=amount; if(method==='Cartão') net = amount*0.98; if(method==='Boleto') net = amount*0.995; state.balance += net; state.payments.push({ id:'PM'+Date.now(), type:'Depósito', method, amount: net, date:new Date().toISOString(), details:`Depósito via ${method}` }); save(); renderTransacoes(); renderDashboard(); alert('Depósito realizado.'); }

/********************* Investir / Descontos *********************/
function renderInvestir(){ main.innerHTML = `
  <div class="card"><h3>Investir saldo</h3>
    <div class="row"><div class="col"><label>Valor a investir (R$)</label><input id="invest-amount" type="number" min="1" step="0.5"></div><div class="col"><label>Prazo (meses)</label><input id="invest-months" type="number" min="1" value="6"></div><div class="col"><label>&nbsp;</label><button class="btn" id="do-invest">Investir</button></div></div>
    <p class="muted small">Taxa simulada: 0.8% ao mês</p>
  </div>
  <div class="card"><h3>Descontos</h3><p class="muted">Desconto fidelidade: <strong>${(state.discounts.loyalty*100).toFixed(1)}%</strong>. Ao investir você pode aumentar este desconto.</p><div id="invest-list"></div></div>
`; document.getElementById('do-invest').addEventListener('click', doInvest); renderInvestList(); }
function doInvest(){ const amount=parseFloat(document.getElementById('invest-amount').value)||0; const months=parseInt(document.getElementById('invest-months').value)||1; if(amount<=0||amount>state.balance){alert('Valor inválido ou saldo insuficiente');return;} const rate=0.008; const expected=amount*Math.pow(1+rate,months); const rec={id:'I'+Date.now(),amount,months,investedAt:new Date().toISOString(),expected}; state.investments.push(rec); state.balance -= amount; state.payments.push({id:'INV'+Date.now(),type:'Investimento',method:'Saldo',amount:-amount,date:new Date().toISOString(),details:`Investimento ${months} meses`}); // bonus: raise loyalty proportionally
state.discounts.loyalty = Math.min(0.2, state.discounts.loyalty + Math.min(0.01, amount/1000)); save(); renderInvestList(); renderDashboard(); alert('Investimento realizado.'); }
function renderInvestList(){ const el=document.getElementById('invest-list'); if(!state.investments.length) return el.innerHTML='<p class="muted">Nenhum investimento.</p>'; el.innerHTML=`<table><thead><tr><th>Data</th><th>Valor</th><th>Meses</th><th>Expectativa</th></tr></thead><tbody>${state.investments.map(i=>`<tr><td>${new Date(i.investedAt).toLocaleString()}</td><td>R$ ${i.amount.toFixed(2)}</td><td>${i.months}</td><td>R$ ${i.expected.toFixed(2)}</td></tr>`).join('')}</tbody></table>`; }

/********************* Painel Admin *********************/
function renderAdmin(){
  main.innerHTML = `
    <div class="card"><h3>Painel de Administração</h3>
      <p class="muted">Visualize e gerencie apostas, pagamentos e resultados.</p>
      <div style="margin-top:8px"><button class="btn" id="btn-refresh">Atualizar lista</button></div>
    </div>
    <div class="card"><h4>Apostas</h4><div id="admin-bets"></div></div>
    <div class="card"><h4>Pagamentos</h4><div id="admin-payments"></div></div>
  `;
  document.getElementById('btn-refresh').addEventListener('click', ()=>{ renderAdminLists(); });
  renderAdminLists();
}
function renderAdminLists(){ const eb=document.getElementById('admin-bets'); const ep=document.getElementById('admin-payments'); if(state.bets.length===0) eb.innerHTML='<p class="muted">Sem apostas.</p>'; else eb.innerHTML=`<table><thead><tr><th>Data</th><th>Jogo</th><th>Números</th><th>Valor</th><th>Status</th><th>Ações</th></tr></thead><tbody>${state.bets.map(b=>`<tr><td>${new Date(b.date).toLocaleString()}</td><td>${b.type}</td><td>${b.numbers.join(', ')}</td><td>R$ ${b.amount.toFixed(2)}</td><td>${b.status}</td><td><button class="btn ghost" onclick="adminMarkPaid('${b.id}')">Marcar Pago</button></td></tr>`).join('')}</tbody></table>`; if(state.payments.length===0) ep.innerHTML='<p class="muted">Sem pagamentos.</p>'; else ep.innerHTML=`<table><thead><tr><th>Data</th><th>Tipo</th><th>Método</th><th>Valor</th></tr></thead><tbody>${state.payments.map(p=>`<tr><td>${new Date(p.date).toLocaleString()}</td><td>${p.type}</td><td>${p.method||'-'}</td><td>R$ ${p.amount.toFixed(2)}</td></tr>`).join('')}</tbody></table>`; }
window.adminMarkPaid = function(bid){ const bet = state.bets.find(b=>b.id===bid); if(!bet) return; if(!confirm('Marcar prêmio desta aposta como pago?')) return; // create payment record if prize exists
  if(bet.prize && bet.prize>0){ state.payments.push({id:'ADM'+Date.now(), type:'Prêmio (Pago)', method:'PIX', amount: -bet.prize, date:new Date().toISOString(), details:`Pago ${bet.id}`}); /* note: negative indicates outflow */ }
  bet.status = bet.status + ' — Pago'; save(); renderAdminLists(); alert('Atualizado.'); }

/********************* Configurações *********************/
function renderConfig(){ main.innerHTML = `
  <div class="card"><h3>PIN e Segurança</h3><label>Novo PIN (4 dígitos)</label><input id="new-pin" maxlength="4" placeholder="ex:4321"><div style="margin-top:8px"><button class="btn" id="change-pin">Alterar PIN</button></div>
  <p class="muted small">PIN padrão: 1234 — altere por segurança.</p></div>
  <div class="card"><h3>Exportar / Reset</h3><div style="display:flex;gap:8px"><button class="btn" id="export-data">Exportar JSON</button><button class="btn ghost" id="reset-data">Resetar app</button></div></div>
`;
  document.getElementById('change-pin').addEventListener('click', ()=>{ const v=document.getElementById('new-pin').value.trim(); if(!/^[0-9]{4}$/.test(v)){alert('PIN precisa de 4 dígitos');return;} state.pin=v; save(); alert('PIN atualizado.'); });
  document.getElementById('export-data').addEventListener('click', ()=>{ const blob=new Blob([JSON.stringify(state,null,2)],{type:'application/json'}); const url=URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='lot_caixas_export.json'; a.click(); });
  document.getElementById('reset-data').addEventListener('click', ()=>{ if(!confirm('Resetar todos os dados locais?')) return; localStorage.removeItem(STORAGE_KEY); location.reload(); });
}

/********************* PIN Modal Behavior *********************/
const overlay = document.getElementById('pin-overlay'); const pinInputs = Array.from(document.querySelectorAll('.pin-digit'));
pinInputs.forEach((inp,i)=>{ inp.addEventListener('input', ()=>{ if(inp.value.length) pinInputs[Math.min(i+1,pinInputs.length-1)].focus(); }); inp.addEventListener('keydown', (e)=>{ if(e.key==='Backspace' && !inp.value && i>0) pinInputs[i-1].focus(); }); });
document.getElementById('btn-enter').addEventListener('click', ()=>{ const val = pinInputs.map(i=>i.value||'').join(''); if(val===state.pin){ overlay.style.display='none'; overlay.setAttribute('aria-hidden','true'); showTab('dashboard'); } else{ alert('PIN inválido'); pinInputs.forEach(i=>i.value=''); pinInputs[0].focus(); } });
document.getElementById('btn-reset').addEventListener('click', ()=>{ if(confirm('Restaurar PIN para 1234?')){ state.pin='1234'; save(); alert('PIN restaurado.'); } });

/********************* Utilitários *********************/
function escapeHtml(s){ return String(s||'').replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":"&#39;"})[c]); }

/********************* Inicialização *********************/
load(); overlay.style.display='flex'; pinInputs[0].focus(); // expose console helpers
window.APP = { state, save };

/********************* Sincronização com backend (exemplo) *********************/
async function syncToServer(){
  // exemplo - não funciona sem backend real
  try{
    const resp = await fetch('https://api.exemplo-loterica.com/sync',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(state)});
    const json = await resp.json(); console.log('sync ok',json);
  }catch(e){console.warn('sync falhou',e)}
}

</script>
</body>
</html>



























