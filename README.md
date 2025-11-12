<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Lotérica Caixas — App</title>
  <style>
    :root{
      --bg:#ffeaf0; /* light pink */
      --card:#fff;
      --muted:#7a7a7a;
      --accent:#ff6fa3;
      --glass: rgba(255,255,255,0.7);
      --shadow: 0 6px 18px rgba(0,0,0,0.08);
      font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
    }
    html,body{height:100%;}
    body{margin:0;background:linear-gradient(180deg,var(--bg),#fff);color:#222}
    .app{max-width:1100px;margin:28px auto;padding:22px;border-radius:18px;background:linear-gradient(180deg,rgba(255,255,255,0.9),rgba(255,255,255,0.95));box-shadow:var(--shadow)}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
    .brand{display:flex;align-items:center;gap:12px}
    .logo{width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--accent),#ff9cc2);display:flex;align-items:center;justify-content:center;color:white;font-weight:700;font-size:18px}
    h1{margin:0;font-size:20px}
    nav{display:flex;gap:8px}
    .tab{padding:8px 12px;border-radius:10px;background:transparent;border:1px solid transparent;cursor:pointer}
    .tab.active{background:var(--accent);color:white}
    main{display:grid;grid-template-columns:1fr 360px;gap:18px}
    .card{background:var(--card);padding:16px;border-radius:12px;box-shadow:var(--shadow)}
    .profile .row{display:flex;justify-content:space-between;align-items:center;padding:8px 0;border-bottom:1px dashed #eee}
    .muted{color:var(--muted);font-size:13px}
    .actions{display:flex;gap:8px;margin-top:10px}
    button{background:var(--accent);border:none;color:white;padding:8px 12px;border-radius:10px;cursor:pointer}
    button.ghost{background:transparent;color:var(--accent);border:1px solid var(--accent)}
    .transactions table{width:100%;border-collapse:collapse}
    .transactions th,.transactions td{padding:8px;border-bottom:1px solid #f3f3f3;text-align:left;font-size:13px}
    .status.win{color:green;font-weight:600}
    .status.lose{color:#d00}
    .grid-2{display:grid;grid-template-columns:1fr 1fr;gap:10px}
    label{display:block;font-size:13px;margin-bottom:6px}
    input[type=text], input[type=number], select{width:100%;padding:8px;border-radius:8px;border:1px solid #eee}
    .small{font-size:12px}
    .admin-list{max-height:320px;overflow:auto}
    footer{margin-top:14px;text-align:center;color:var(--muted);font-size:13px}

    /* Login overlay */
    .overlay{position:fixed;inset:0;background:rgba(0,0,0,0.35);display:flex;align-items:center;justify-content:center}
    .login{width:360px;background:linear-gradient(180deg,#fff,#fff7fb);padding:20px;border-radius:12px;box-shadow:0 12px 40px rgba(0,0,0,0.2);text-align:center}
    .pin-inputs{display:flex;gap:8px;justify-content:center;margin:14px 0}
    .pin-inputs input{width:56px;height:56px;font-size:24px;text-align:center;border-radius:10px;border:1px solid #eee}
    .help{font-size:13px;color:var(--muted)}

    /* responsive */
    @media (max-width:880px){main{grid-template-columns:1fr;}}
  </style>
</head>
<body>
  <div class="app" id="app">
    <header>
      <div class="brand">
        <div class="logo">LC</div>
        <div>
          <h1>Lotérica Caixas</h1>
          <div class="muted">Gestão de apostas, pagamentos e histórico</div>
        </div>
      </div>
      <nav>
        <button class="tab active" data-tab="cliente">Cliente</button>
        <button class="tab" data-tab="jogos">Jogos da Sorte</button>
        <button class="tab" data-tab="pagamentos">Pagamentos</button>
        <button class="tab" data-tab="admin">Admin</button>
      </nav>
    </header>

    <main>
      <section id="cliente" class="card cliente">
        <h2>Perfil do Cliente</h2>
        <div class="profile">
          <div class="row"><div><strong id="clientName">—</strong><div class="muted">Nome</div></div><div><button id="editProfile" class="ghost">Editar</button></div></div>
          <div class="row"><div><span id="clientCPF">***.***.***-**</span><div class="muted">CPF (privado)</div></div></div>
          <div class="row"><div><strong>Saldo: R$ <span id="clientSaldo">0.00</span></strong><div class="muted">Disponível</div></div>
          <div>
            <button id="recarregar">Recarregar</button>
          </div></div>
          <div class="actions">
            <button id="novaAposta">Nova Aposta</button>
            <button id="verHistorico" class="ghost">Ver Histórico</button>
          </div>
        </div>
      </section>

      <aside>
        <div class="card transactions">
          <h3>Histórico Consolidado</h3>
          <div class="muted small">Inclui Jogos da Sorte e Pagamentos</div>
          <div style="margin-top:10px;">
            <table id="txTable"><thead><tr><th>Data</th><th>Tipo</th><th>Detalhes</th><th>Valor</th></tr></thead><tbody></tbody></table>
          </div>
        </div>

        <div class="card" style="margin-top:12px;">
          <h3>Atalhos</h3>
          <div style="display:flex;flex-direction:column;gap:8px;margin-top:8px;">
            <button id="toJogos" class="ghost">Ir para Jogos</button>
            <button id="toPagamentos" class="ghost">Ir para Pagamentos</button>
          </div>
        </div>
      </aside>

      <section id="jogos" class="card" style="display:none;">
        <h2>Jogos da Sorte</h2>
        <div class="muted small">Adicione apostas e valide resultados. Sistema suporta: Mega-Sena, Quina, Lotofácil, Lotomania.</div>

        <div style="margin-top:12px;" id="apostaForm">
          <label>Tipo de Jogo</label>
          <select id="tipoJogo"><option value="megasena">Mega-Sena</option><option value="quina">Quina</option><option value="lotofacil">Lotofácil</option><option value="lotomania">Lotomania</option></select>
          <div style="margin-top:8px;" id="numerosWrap">
            <label>Números (separados por vírgula)</label>
            <input type="text" id="numeros" placeholder="ex: 05,12,23,34,45,56" />
          </div>
          <div class="grid-2" style="margin-top:8px;">
            <div>
              <label>Valor da Aposta (R$)</label>
              <input type="number" id="valorAposta" min="0" step="0.01" value="2.00" />
            </div>
            <div>
              <label>Data da Aposta</label>
              <input type="date" id="dataAposta" />
            </div>
          </div>
          <div class="actions"><button id="salvarAposta">Salvar Aposta</button><button id="limparAposta" class="ghost">Limpar</button></div>
        </div>

        <hr style="margin:14px 0;" />
        <div>
          <h3>Configurar Resultado Oficial</h3>
          <div class="muted small">Escolha o tipo de jogo e insira os números sorteados — o sistema atualizará automaticamente apostas e valores de prêmio.</div>
          <div style="margin-top:8px;">
            <select id="resTipo"><option value="megasena">Mega-Sena</option><option value="quina">Quina</option><option value="lotofacil">Lotofácil</option><option value="lotomania">Lotomania</option></select>
            <label style="margin-top:8px;">Números sorteados</label>
            <input type="text" id="resNumeros" placeholder="ex: 05,12,23,34,45,56" />
            <div class="actions" style="margin-top:10px;"><button id="confirmResultado">Confirmar Resultado</button></div>
          </div>
        </div>

        <hr style="margin:14px 0;" />
        <div>
          <h3>Apostas Recentes</h3>
          <div id="apostasList" class="admin-list" style="margin-top:8px;"></div>
        </div>
      </section>

      <section id="pagamentos" class="card" style="display:none;">
        <h2>Pagamentos & Investimentos</h2>
        <div class="muted small">Gerencie pagamentos, formas de pagamento e invista saldo do cliente.</div>
        <div style="margin-top:8px;">
          <label>Forma de Pagamento</label>
          <select id="formaPagamento"><option>PIX</option><option>Cartão</option><option>Boleto</option><option>Transferência</option></select>
          <label style="margin-top:8px">Valor (R$)</label>
          <input type="number" id="valorPagamento" min="0" step="0.01" />
          <div class="actions" style="margin-top:8px;"><button id="registrarPagamento">Registrar Pagamento</button></div>
        </div>

        <hr style="margin:12px 0" />
        <div>
          <h3>Investir Saldo</h3>
          <label>Opções</label>
          <select id="investOption"><option value="poupanca">Poupança (simulação)</option><option value="tesouro">Tesouro Direto (simulação)</option><option value="fundos">Fundos (simulação)</option></select>
          <label style="margin-top:8px">Valor a Investir</label>
          <input type="number" id="valorInvest" min="0" step="0.01" />
          <div class="small muted">Ao investir via app, o cliente recebe desconto de taxa em apostas (simulado).</div>
          <div class="actions" style="margin-top:8px;"><button id="investirSaldo">Investir</button></div>
        </div>

        <hr style="margin:12px 0" />
        <div>
          <h3>Histórico de Pagamentos</h3>
          <div id="pagosList" style="margin-top:8px;"></div>
        </div>
      </section>

      <section id="admin" class="card" style="display:none;">
        <h2>Painel do Administrador</h2>
        <div class="muted small">Visualize e gerencie apostas, resultados e pagamentos.</div>
        <div style="margin-top:12px;">
          <h4>Apostas</h4>
          <div id="adminApostas" class="admin-list"></div>
        </div>
        <div style="margin-top:12px;">
          <h4>Formulário Rápido — Inserir Resultado</h4>
          <label>Tipo</label>
          <select id="adminResTipo"><option value="megasena">Mega-Sena</option><option value="quina">Quina</option><option value="lotofacil">Lotofácil</option><option value="lotomania">Lotomania</option></select>
          <label style="margin-top:8px">Números</label>
          <input type="text" id="adminResNums" placeholder="05,12,23,34,45,56" />
          <div class="actions" style="margin-top:8px;"><button id="adminSetResultado">Aplicar Resultado</button></div>
        </div>
      </section>

    </main>

    <footer>Lotérica Caixas — protótipo. Dados salvos localmente (localStorage) para demonstração.</footer>
  </div>

  <!-- Login overlay (4-digit pin) -->
  <div id="loginOverlay" class="overlay">
    <div class="login">
      <h2>Entrar — Lotérica Caixas</h2>
      <div class="help">Digite sua senha de 4 dígitos</div>
      <div class="pin-inputs" id="pinInputs">
        <input maxlength="1" type="text" inputmode="numeric" pattern="[0-9]*" />
        <input maxlength="1" type="text" inputmode="numeric" pattern="[0-9]*" />
        <input maxlength="1" type="text" inputmode="numeric" pattern="[0-9]*" />
        <input maxlength="1" type="text" inputmode="numeric" pattern="[0-9]*" />
      </div>
      <div class="muted small">Senha padrão para teste: <strong>1234</strong></div>
    </div>
  </div>

  <script>
    // ---------- Utilities & Persistence ----------
    const STORAGE_KEY = 'loterica_caixas_v1';
    const defaultState = {
      client: {name:'Ana Cliente', cpf:'000.000.000-00', saldo:100.00, pin:'1234'},
      apostas: [], // {id, tipo, numeros:[], valor, data, status:'pendente'|'ganhou'|'perdeu', acertos:0, valor_premio:0}
      pagamentos: [], // {id, data, tipo, valor, descricao}
      settings:{discountActive:false}
    };
    function load(){
      try{const raw=localStorage.getItem(STORAGE_KEY);return raw?JSON.parse(raw):structuredClone(defaultState);}catch(e){console.error(e);return structuredClone(defaultState)}
    }
    function save(state){localStorage.setItem(STORAGE_KEY,JSON.stringify(state));}
    let state = load();

    // ---------- UI wiring ----------
    const tabs = document.querySelectorAll('.tab');
    const sections = {cliente:document.getElementById('cliente'),jogos:document.getElementById('jogos'),pagamentos:document.getElementById('pagamentos'),admin:document.getElementById('admin')};
    tabs.forEach(t=>t.addEventListener('click',()=>{tabs.forEach(x=>x.classList.remove('active'));t.classList.add('active');const tab=t.dataset.tab;Object.values(sections).forEach(s=>s.style.display='none');sections[tab].style.display='block'}));

    // Login
    const loginOverlay = document.getElementById('loginOverlay');
    const pinInputs = Array.from(document.querySelectorAll('.pin-inputs input'));
    pinInputs.forEach((inp,i)=>{
      inp.addEventListener('input',()=>{if(inp.value.length) pinInputs[Math.min(i+1,pinInputs.length-1)].focus();});
      inp.addEventListener('keydown',e=>{if(e.key==='Backspace' && !inp.value && i>0) pinInputs[i-1].focus();});
    });
    function checkPin(){const pin=pinInputs.map(i=>i.value||'').join('');if(pin.length===4){if(pin===state.client.pin){loginOverlay.style.display='none';renderAll();}else{alert('PIN incorreto. Use 1234 para teste.');pinInputs.forEach(i=>i.value='');pinInputs[0].focus();}}}
    pinInputs.forEach(i=>i.addEventListener('input',checkPin));
    pinInputs[0].focus();

    // Render functions
    function renderProfile(){document.getElementById('clientName').textContent=state.client.name;document.getElementById('clientCPF').textContent='***.***.***-**';document.getElementById('clientSaldo').textContent=state.client.saldo.toFixed(2);}    
    function renderTx(){const tb=document.querySelector('#txTable tbody');tb.innerHTML='';
      const all = [...state.apostas.map(a=>({date:a.data,type:'Aposta',details:`${a.tipo} — ${a.numeros.join(', ')}`,valor:(a.valor_premio? a.valor_premio : -a.valor)})), ...state.pagamentos.map(p=>({date:p.data,type:'Pagamento',details:p.tipo + (p.descricao? ' — '+p.descricao : ''),valor:p.valor}))];
      all.sort((a,b)=> new Date(b.date) - new Date(a.date));
      all.forEach(r=>{const tr=document.createElement('tr');tr.innerHTML=`<td>${new Date(r.date).toLocaleString()}</td><td>${r.type}</td><td>${r.details}</td><td>R$ ${Number(r.valor).toFixed(2)}</td>`;tb.appendChild(tr);});
    }
    function renderApostasList(){const wrap=document.getElementById('apostasList');wrap.innerHTML='';state.apostas.slice().reverse().forEach(a=>{const div=document.createElement('div');div.style.padding='10px';div.style.border='1px solid #faf0f5';div.style.borderRadius='10px';div.style.marginBottom='8px';div.innerHTML=`<div style="display:flex;justify-content:space-between"><div><strong>${a.tipo}</strong><div class="muted small">${a.numeros.join(', ')}</div></div><div><div class="small">R$ ${a.valor.toFixed(2)}</div><div class="muted small">${a.status}</div></div></div>`;wrap.appendChild(div);});
    }
    function renderAdminApostas(){const wrap=document.getElementById('adminApostas');wrap.innerHTML='';state.apostas.slice().reverse().forEach(a=>{const container=document.createElement('div');container.style.padding='8px';container.style.borderBottom='1px solid #f5f5f5';container.innerHTML=`<div style="display:flex;justify-content:space-between"><div><strong>${a.tipo}</strong><div class="muted small">${a.numeros.join(', ')} — ${new Date(a.data).toLocaleString()}</div></div><div style="text-align:right"><div class="small">Valor: R$ ${a.valor.toFixed(2)}</div><div class="small">Status: ${a.status}</div></div></div>`;wrap.appendChild(container);});}
    function renderPagamentos(){const wrap=document.getElementById('pagosList');wrap.innerHTML='';state.pagamentos.slice().reverse().forEach(p=>{const d=document.createElement('div');d.style.padding='8px';d.style.borderBottom='1px solid #f5f5f5';d.innerHTML=`<div style="display:flex;justify-content:space-between"><div><strong>${p.tipo}</strong><div class="muted small">${p.descricao||''}</div></div><div>R$ ${p.valor.toFixed(2)}</div></div>`;wrap.appendChild(d);});}

    function renderAll(){renderProfile();renderTx();renderApostasList();renderAdminApostas();renderPagamentos();}

    // ---------- Apostas: create, validate, prize calc ----------
    document.getElementById('salvarAposta').addEventListener('click',()=>{
      const tipo=document.getElementById('tipoJogo').value;
      const numeros = document.getElementById('numeros').value.split(',').map(s=>s.trim()).filter(Boolean);
      const valor = Number(document.getElementById('valorAposta').value) || 0;
      const data = document.getElementById('dataAposta').value || new Date().toISOString();
      if(!numeros.length){return alert('Insira números válidos.');}
      // basic validation per jogo
      const valid = validateNumbersForGame(tipo,numeros);
      if(!valid.ok) return alert(valid.msg);
      const aposta = {id:cryptoId(),tipo,numeros,valor,data,status:'pendente',acertos:0,valor_premio:0};
      state.apostas.push(aposta); state.client.saldo -= valor; // debit
      state.pagamentos.push({id:cryptoId(),data:new Date().toISOString(),tipo:'Pagamento',valor:-valor,descricao:'Compra de aposta'});
      save(state); renderAll(); alert('Aposta registrada.');
    });
    document.getElementById('limparAposta').addEventListener('click',()=>{document.getElementById('numeros').value='';document.getElementById('valorAposta').value='2.00';});

    function validateNumbersForGame(tipo,arr){
      // basic checks: quantity & numeric
      const nums = arr.map(n=>Number(n));
      if(nums.some(isNaN)) return {ok:false,msg:'Números devem ser numéricos.'};
      const rules = {megasena:{min:6,max:6,range:1},quina:{min:5,max:5,range:1},lotofacil:{min:15,max:15,range:1},lotomania:{min:50,max:50,range:1}};
      const r = rules[tipo]; if(!r) return {ok:false,msg:'Tipo inválido.'};
      if(nums.length<r.min || nums.length>r.max) return {ok:false,msg:`${tipo}: precisa de ${r.min} números.`};
      return {ok:true};
    }

    // admin/app: set official result
    document.getElementById('confirmResultado').addEventListener('click',()=>{const tipo=document.getElementById('resTipo').value;const res=document.getElementById('resNumeros').value.split(',').map(s=>s.trim()).filter(Boolean);applyResult(tipo,res);});
    document.getElementById('adminSetResultado').addEventListener('click',()=>{const tipo=document.getElementById('adminResTipo').value;const res=document.getElementById('adminResNums').value.split(',').map(s=>s.trim()).filter(Boolean);applyResult(tipo,res);});

    function applyResult(tipo,resNumeros){
      if(!resNumeros.length) return alert('Insira os números sorteados.');
      // update each pending aposta of that type
      let winners=0;let losers=0;
      state.apostas.forEach(a=>{if(a.tipo===tipo && a.status==='pendente'){
        const acert = a.numeros.filter(n=> resNumeros.includes(n)).length;
        a.acertos = acert;
        // determine prize value (simplified rules for demo)
        const prize = calculatePrize(tipo,acert,a.valor);
        if(prize>0){a.status='ganhou';a.valor_premio=prize; winners++;
          // create payment record for prize
          state.pagamentos.push({id:cryptoId(),data:new Date().toISOString(),tipo:'Prêmio',valor:prize,descricao:`Prêmio ${a.tipo} — acertos: ${acert}`});
          state.client.saldo += prize;
        } else {a.status='perdeu'; a.valor_premio=0; losers++;}
      }});
      save(state); renderAll(); alert(`Resultado aplicado. Vencedores: ${winners}, perdedores atualizados: ${losers}`);
    }

    function calculatePrize(tipo,acertos,valorAposta){
      // NOTE: Esto é uma simulação simplificada dos prêmios.
      // Regras artificiais: para demonstração, premia acertos mínimos.
      const table = {
        megasena: {6:5000000,5:50000,4:1000},
        quina: {5:1000000,4:2000,3:50},
        lotofacil: {15:100000,14:500,13:30},
        lotomania: {20:300000,19:1000,18:30}
      };
      const tier = table[tipo]||{};
      const base = tier[acertos] || 0;
      // Apply proportion to bet value to keep relation
      return base? Math.max( (base * (valorAposta/2)), 1 ) : 0;
    }

    // ---------- Pagamentos & Investir ----------
    document.getElementById('registrarPagamento').addEventListener('click',()=>{
      const tipo = document.getElementById('formaPagamento').value;
      const valor = Number(document.getElementById('valorPagamento').value)||0;
      if(valor<=0) return alert('Insira um valor maior que 0.');
      state.pagamentos.push({id:cryptoId(),data:new Date().toISOString(),tipo:tipo,valor:valor,descricao:'Depósito'});
      state.client.saldo += valor; save(state); renderAll(); alert('Pagamento registrado.');
    });

    document.getElementById('investirSaldo').addEventListener('click',()=>{
      const op = document.getElementById('investOption').value; const val = Number(document.getElementById('valorInvest').value)||0;
      if(val<=0) return alert('Insira valor válido.'); if(val>state.client.saldo) return alert('Saldo insuficiente.');
      // simulate investment: remove from balance, create record and apply immediate small bonus (simulated)
      state.client.saldo -= val;
      const bonus = val * 0.02; // 2% immediate simulated bonus
      state.pagamentos.push({id:cryptoId(),data:new Date().toISOString(),tipo:'Investimento',valor:-val,descricao:`${op}`});
      state.pagamentos.push({id:cryptoId(),data:new Date().toISOString(),tipo:'Bônus Invest',valor:bonus,descricao:'Bônus por investir via app'});
      state.client.saldo += bonus;
      // enable discount state if investing
      state.settings.discountActive = true;
      save(state); renderAll(); alert(`Investimento simulado realizado. Bônus R$ ${bonus.toFixed(2)} concedido. Desconto em apostas ativado.`);
    });

    // ---------- Admin actions ----------
    // simple admin can delete aposta
    function adminDeleteAposta(id){state.apostas = state.apostas.filter(a=>a.id!==id); save(state); renderAll();}

    // ---------- Helpers ----------
    function cryptoId(){return 'id-'+Math.random().toString(36).slice(2,9);}    

    // initial render
    renderAll();

    // expose some shortcuts
    document.getElementById('toJogos').addEventListener('click',()=>{document.querySelector('[data-tab="jogos"]').click();});
    document.getElementById('toPagamentos').addEventListener('click',()=>{document.querySelector('[data-tab="pagamentos"]').click();});
    document.getElementById('novaAposta').addEventListener('click',()=>{document.querySelector('[data-tab="jogos"]').click();document.getElementById('numeros').focus();});
    document.getElementById('verHistorico').addEventListener('click',()=>{document.querySelector('[data-tab="cliente"]').click();document.getElementById('txTable').scrollIntoView();});
    document.getElementById('recarregar').addEventListener('click',()=>{const v=Number(prompt('Valor a recarregar (R$):')||0); if(v>0){state.client.saldo+=v;state.pagamentos.push({id:cryptoId(),data:new Date().toISOString(),tipo:'Recarga',valor:v});save(state);renderAll();}});

    // set default date today for forms
    document.getElementById('dataAposta').value = new Date().toISOString().slice(0,10);

    // small note: the app uses localStorage for demo. In a real app, personal data must be stored on a secure server and encrypted.
  </script>
</body>
</html>


























