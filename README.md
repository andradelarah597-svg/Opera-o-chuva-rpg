<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
<title>Operação CHUVA — RPG da Emergência</title>
<style>
  :root{
    --bg1:#02031a; --bg2:#051033;
    --accent:#00f6ff; --accent2:#7cffb2;
    --panel:rgba(255,255,255,0.04); --muted:#97c6d9; --danger:#ff6b6b;
    --glass: rgba(255,255,255,0.03);
  }
  *{box-sizing:border-box}
  body{
    margin:0; font-family:Inter,Segoe UI,Helvetica,Arial,sans-serif;
    background: linear-gradient(180deg,var(--bg1),var(--bg2));
    color:#e6fbff; -webkit-font-smoothing:antialiased;
  }
  #wrap{max-width:980px;margin:10px auto;padding:12px;}
  header{display:flex;justify-content:space-between;align-items:center;gap:12px}
  h1{margin:0;color:var(--accent);font-size:20px}
  .subtitle{color:var(--muted);font-size:12px}
  main{margin-top:12px;display:flex;gap:12px;flex-wrap:wrap}
  #view{flex:1 1 620px;background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(0,0,0,0.06));padding:12px;border-radius:12px;min-height:520px}
  #sidebar{flex:0 0 320px;display:flex;flex-direction:column;gap:12px}
  .panel{background:var(--panel);padding:10px;border-radius:10px;box-shadow:0 6px 18px rgba(0,0,0,0.6)}
  #canvas{width:100%;height:340px;border-radius:8px;background:linear-gradient(180deg,#021b2b,#00111a)}
  .row{display:flex;gap:8px;align-items:center}
  .bar{height:18px;background:rgba(255,255,255,0.06);border-radius:8px;overflow:hidden}
  .barInner{height:100%;background:linear-gradient(90deg,var(--accent2),var(--accent));width:100%}
  .big{font-weight:700;color:var(--accent2);font-size:18px}
  .muted{color:var(--muted);font-size:13px}
  .epi-grid{display:grid;grid-template-columns:repeat(2,1fr);gap:8px}
  .epi-btn{padding:8px;border-radius:8px;background:rgba(255,255,255,0.02);text-align:center;cursor:pointer;border:1px solid rgba(255,255,255,0.03);user-select:none}
  .epi-btn.selected{outline:2px solid rgba(124,255,178,0.14);box-shadow:0 8px 20px rgba(0,255,220,0.03);transform:translateY(-2px)}
  button.btn{padding:8px 12px;border-radius:8px;border:none;cursor:pointer;background:linear-gradient(90deg,var(--accent),#6ef6ff);color:#00110f;font-weight:700}
  button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.04);color:var(--muted)}
  .log{font-size:13px;color:var(--muted);max-height:120px;overflow:auto;padding:6px;border-radius:6px;background:rgba(0,0,0,0.06)}
  footer{margin-top:12px;color:var(--muted);font-size:13px;text-align:center}
  @media(max-width:980px){
    main{flex-direction:column}
    #sidebar{order:2;flex:unset;width:100%}
    #view{order:1}
    .epi-grid{grid-template-columns:repeat(3,1fr)}
  }
</style>
</head>
<body>
<div id="wrap">
  <header>
    <div>
      <h1>Operação C.H.U.V.A. — RPG da Emergência</h1>
      <div class="subtitle">Futurista — Você é o técnico de enfermagem: proteja pacientes e equipe</div>
    </div>
    <div style="text-align:right">
      <div class="muted">Compartilhe o arquivo .html com a turma para jogar no celular</div>
    </div>
  </header>

  <main>
    <section id="view" class="panel">
      <canvas id="canvas"></canvas>
      <div style="margin-top:10px" id="sceneText" class="muted"></div>
    </section>

    <aside id="sidebar">
      <div class="panel">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div><strong>Energia (Hidratação)</strong></div>
          <div id="energyVal" class="big">100</div>
        </div>
        <div style="margin-top:8px" class="bar"><div id="energyBar" class="barInner" style="width:100%"></div></div>
        <div style="display:flex;gap:8px;justify-content:space-between;margin-top:8px">
          <div><small>Pontuação</small><div id="score" class="big">0</div></div>
          <div><small>Rodada</small><div id="round" class="big">1</div></div>
        </div>
      </div>

      <div class="panel">
        <div style="font-weight:700;margin-bottom:6px">Missão / Objetivo</div>
        <div id="missionText" class="muted">Elimine os agentes e leve os pacientes ao Leito de Alta. Escolha EPIs corretamente.</div>
        <div style="margin-top:8px;display:flex;gap:8px">
          <button id="startBtn" class="btn">Iniciar Missão</button>
          <button id="restartBtn" class="btn ghost">Reiniciar</button>
        </div>
      </div>

      <div class="panel">
        <div style="font-weight:700;margin-bottom:6px">Escolha até 2 EPIs (Armaduras)</div>
        <div class="epi-grid" id="epiGrid">
          <div class="epi-btn" data-epi="Luvas">🧤 Luvas</div>
          <div class="epi-btn" data-epi="Avental">🦺 Avental</div>
          <div class="epi-btn" data-epi="Máscara">😷 Máscara</div>
          <div class="epi-btn" data-epi="Óculos">🥽 Óculos</div>
          <div class="epi-btn" data-epi="FaceShield">🛡️ FaceShield</div>
          <div class="epi-btn" data-epi="Sabao">🧴 Sabão</div>
        </div>
        <div style="margin-top:8px;display:flex;gap:8px">
          <button id="applyBtn" class="btn">Aplicar EPIs</button>
          <button id="useItemBtn" class="btn ghost">Usar Item</button>
        </div>
      </div>

      <div class="panel">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div style="font-weight:700">Inventário</div>
          <div class="muted">(toque para usar)</div>
        </div>
        <div style="margin-top:8px;display:flex;gap:8px;flex-wrap:wrap">
          <button id="itemSoro" class="btn ghost">💧 Soro × 2</button>
          <button id="itemSabao" class="btn ghost">🧴 Sabão × 1</button>
        </div>
        <div style="margin-top:8px" class="log" id="log">Bem-vindo Técnico. Clique em Iniciar Missão.</div>
      </div>
    </aside>
  </main>

  <footer>Jogo educativo — adapte para aula. Criado para uso didático.</footer>
</div>

<script>
/* Operação CHUVA — RPG da Emergência
   Versão narrativa: fases, batalhas, EPIs como armadura, itens e HUD.
   Para compartilhar: salvar este arquivo .html e enviar aos alunos (Drive/Netlify/Replit).
*/

/* ---------- Estado do jogo ---------- */
const state = {
  energia: 100,
  score: 0,
  round: 1,
  phase: 0, // 0 intro, 1 triagem, 2 isolamento, 3 vigilância (final)
  enemiesToSpawn: 6,
  enemyQueue: [],
  current: null,
  epiSelected: [],
  inventory: { soro:2, sabao:1 },
  log: []
};

/* ---------- Definição de inimigos e requisitos de EPIs ---------- */
const enemies = [
  { id:'virus', name:'Vírus Entérico', color:'#7fdbff', req:['Máscara','FaceShield'], difficulty:1,
    hint:'Vírus -> gotículas / gotículas+contato (máscara/face shield)'},
  { id:'bacteria', name:'Bactéria Salmonella', color:'#ffd166', req:['Luvas','Avental'], difficulty:1,
    hint:'Bactéria -> contato com fezes/alimentos (luvas + avental)'},
  { id:'parasite', name:'Parasita Giardia', color:'#9b5de5', req:['Luvas','Sabao'], difficulty:1,
    hint:'Parasita -> higiene intensiva (luvas + sabão)'}
];

/* ---------- Elementos UI ---------- */
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
canvas.width = canvas.clientWidth;
canvas.height = canvas.clientHeight;

const ui = {
  energyVal: document.getElementById('energyVal'),
  energyBar: document.getElementById('energyBar'),
  score: document.getElementById('score'),
  round: document.getElementById('round'),
  log: document.getElementById('log'),
  sceneText: document.getElementById('sceneText'),
  missionText: document.getElementById('missionText'),
  startBtn: document.getElementById('startBtn'),
  restartBtn: document.getElementById('restartBtn'),
  epiGrid: document.getElementById('epiGrid'),
  applyBtn: document.getElementById('applyBtn'),
  useItemBtn: document.getElementById('useItemBtn'),
  itemSoro: document.getElementById('itemSoro'),
  itemSabao: document.getElementById('itemSabao')
};

/* ---------- Helpers ---------- */
function log(msg){
  const time = new Date().toLocaleTimeString();
  state.log.unshift(`[${time}] ${msg}`);
  if(state.log.length>60) state.log.pop();
  ui.log.innerHTML = state.log.slice(0,10).join('<br>');
}

function updateHUD(){
  ui.energyVal.textContent = Math.max(0, Math.round(state.energia));
  ui.energyBar.style.width = Math.max(0, state.energia) + '%';
  ui.score.textContent = state.score;
  ui.round.textContent = state.round;
  ui.itemSoro.textContent = `💧 Soro × ${state.inventory.soro}`;
  ui.itemSabao.textContent = `🧴 Sabão × ${state.inventory.sabao}`;
}

/* ---------- Game flow ---------- */
function resetGame(){
  state.energia = 100; state.score = 0; state.round = 1; state.phase = 0;
  state.enemiesToSpawn = 6; state.enemyQueue = []; state.current = null;
  state.epiSelected = []; state.inventory = { soro:2, sabao:1 }; state.log = [];
  ui.sceneText.textContent = '';
  prepareEnemyQueue();
  updateHUD();
  drawScene();
  log('Jogo reiniciado. Clique Iniciar Missão para começar.');
}
ui.restartBtn.addEventListener('click', resetGame);

function prepareEnemyQueue(){
  // sequence: triagem (2 enemies), isolamento (2 enemies), vigilância (2 stronger)
  state.enemyQueue = [];
  for(let i=0;i<2;i++) state.enemyQueue.push(randomEnemy());
  for(let i=0;i<2;i++) state.enemyQueue.push(randomEnemy());
  for(let i=0;i<2;i++) state.enemyQueue.push(randomEnemy());
  state.enemiesToSpawn = state.enemyQueue.length;
}

function randomEnemy(){
  // pick random but weight similar
  return JSON.parse(JSON.stringify(enemies[Math.floor(Math.random()*enemies.length)]));
}

function startMission(){
  state.phase = 1;
  ui.missionText.textContent = 'Fase 1 — Triagem: identifique e proteja. Escolha EPIs corretos.';
  log('Missão iniciada — Triagem em andamento.');
  spawnNextEnemy();
}
ui.startBtn.addEventListener('click', ()=> {
  if(state.phase===0) startMission();
});

function spawnNextEnemy(){
  if(state.enemyQueue.length===0){
    // proceed phases or finish
    if(state.phase===1){ state.phase=2; ui.missionText.textContent='Fase 2 — Isolamento: contenção e hidratação'; log('Começa Isolamento.'); prepareEnemyQueuePhase2(); spawnNextEnemy(); return; }
    if(state.phase===2){ state.phase=3; ui.missionText.textContent='Fase 3 — Vigilância: notificar e controlar o surto'; log('Começa Vigilância.'); prepareEnemyQueuePhase3(); spawnNextEnemy(); return; }
    if(state.phase===3){ victory(); return; }
  } else {
    state.current = state.enemyQueue.shift();
    state.enemiesToSpawn = state.enemyQueue.length + 1;
    ui.sceneText.textContent = `Agente detectado: ${state.current.name} — ${state.current.hint}`;
    log(`Agente ${state.current.name} entrou na emergência.`);
    drawScene();
  }
  updateHUD();
}
function prepareEnemyQueuePhase2(){
  state.enemyQueue = [];
  for(let i=0;i<3;i++) state.enemyQueue.push(randomEnemy());
}
function prepareEnemyQueuePhase3(){
  state.enemyQueue = [];
  for(let i=0;i<4;i++){ state.enemyQueue.push(randomEnemy()); }
}

/* ---------- EPI UI ---------- */
document.querySelectorAll('.epi-btn').forEach(btn=>{
  btn.addEventListener('click', ()=>{
    const epi = btn.dataset.epi;
    const idx = state.epiSelected.indexOf(epi);
    if(idx>-1){
      state.epiSelected.splice(idx,1); btn.classList.remove('selected');
    } else {
      if(state.epiSelected.length>=2){
        const removed = state.epiSelected.shift();
        const prev = document.querySelector(`.epi-btn[data-epi="${removed}"]`);
        if(prev) prev.classList.remove('selected');
      }
      state.epiSelected.push(epi); btn.classList.add('selected');
    }
    log(`EPIs selecionados: ${state.epiSelected.join(', ') || 'nenhum'}`);
  });
});

/* ---------- Apply EPIs (combate) ---------- */
ui.applyBtn.addEventListener('click', ()=>{
  if(!state.current){ log('Nenhum agente para enfrentar.'); return; }
  resolveCombat();
});

/* ---------- Items ---------- */
ui.itemSoro.addEventListener('click', ()=>{
  if(state.inventory.soro>0){
    state.inventory.soro--; state.energia = Math.min(100, state.energia+30);
    log('Usou Soro — Hidratação recuperada.');
    updateHUD();
  } else log('Sem Soro disponível.');
});
ui.itemSabao.addEventListener('click', ()=>{
  if(state.inventory.sabao>0){
    state.inventory.sabao--; log('Usou Sabão (efeito contra Parasitas).'); updateHUD();
  } else log('Sem Sabão disponível.');
});
ui.useItemBtn.addEventListener('click', ()=> {
  // quick-use: if current is parasite try apply sabao automatically
  if(!state.current){ log('Nenhum agente no momento.'); return; }
  if(state.current.id==='parasite' && state.inventory.sabao>0){
    state.inventory.sabao--;
    log('Sabão aplicado diretamente — vantagem contra Parasita.');
    // auto success if at least Luvas selected, otherwise partial
    if(state.epiSelected.includes('Luvas')){
      log('Combinação Sabão + Luvas -> sucesso automático.');
      successCombat();
    } else {
      // small damage mitigated
      damagePlayer(6); log('Sem Luvas: dano leve apesar do sabão.');
      spawnAfterDelay();
    }
    updateHUD();
  } else if(state.inventory.soro>0){
    ui.itemSoro.click();
  } else log('Nenhum item aplicável disponível.');
});

/* ---------- Combat resolution logic ---------- */
function resolveCombat(){
  const enemy = state.current;
  const required = enemy.req.slice();
  const chosen = state.epiSelected.slice();

  // allow FaceShield to substitute Máscara etc.
  const normalize = (s)=>s.replace(/\s+/g,'').toLowerCase();

  function matches(reqItem){
    // special mapping: 'Sabao' can be chosen, or consumed from inventory
    const reqNorm = normalize(reqItem);
    if(reqNorm==='sabao'){
      if(chosen.some(c=>normalize(c)==='sabao')) return true;
      if(state.inventory.sabao>0) { state.inventory.sabao--; log('Sabão automático consumido do inventário.'); return true; }
      return false;
    }
    // FaceShield can satisfy máscara requirement if chosen
    if(reqNorm==='máscara' || reqNorm==='mascara') {
      if(chosen.some(c=>normalize(c)==='máscara' || normalize(c)==='faceshield')) return true;
      return false;
    }
    // default: item must be among chosen
    return chosen.some(c=>normalize(c)===reqNorm);
  }

  // success if all requirements matched
  const success = required.every(r => matches(r));

  if(success){
    successCombat();
  } else {
    failCombat();
  }
  // reset episelection
  state.epiSelected = [];
  document.querySelectorAll('.epi-btn').forEach(b=>b.classList.remove('selected'));
  updateHUD();
}

function successCombat(){
  log(`✅ Sucesso! ${state.current.name} neutralizado sem dano.`);
  state.score += 12;
  state.current = null;
  state.round++;
  state.energia = Math.min(100, state.energia + 5); // small heal for correct action
  spawnAfterDelay();
}

function failCombat(){
  const dmg = Math.min(40, 8 + Math.floor(state.round*2));
  damagePlayer(dmg);
  log(`❌ EPI incorreto! Recebeu dano de ${dmg}.`);
  state.score = Math.max(0, state.score - 4);
  state.current = null;
  state.round++;
  spawnAfterDelay();
}

function spawnAfterDelay(){
  updateHUD();
  setTimeout(()=> {
    spawnNextEnemy();
    drawScene();
  }, 800);
}

/* ---------- Player damage & victory ---------- */
function damagePlayer(amount){
  state.energia = Math.max(0, state.energia - amount);
  updateHUD();
  if(state.energia<=0){
    setTimeout(()=> {
      alert(`Hidratação zerada! A emergência entrou em colapso. Pontuação: ${state.score}`);
      resetGame();
    }, 200);
  }
}

function victory(){
  log('🎉 Missão cumprida: pacientes estabilizados. Leito de Alta alcançado.');
  setTimeout(()=> {
    alert(`Missão cumprida!\nPontuação final: ${state.score}\nRodadas: ${state.round}`);
    resetGame();
  }, 300);
}

/* ---------- Draw scene (simple canvas visuals) ---------- */
function drawScene(){
  // clear
  ctx.clearRect(0,0,canvas.width,canvas.height);
  // gradient background
  const g = ctx.createLinearGradient(0,0,0,canvas.height);
  g.addColorStop(0,'#041a2b'); g.addColorStop(1,'#001012');
  ctx.fillStyle = g; ctx.fillRect(0,0,canvas.width,canvas.height);

  // center UI block for enemy
  if(state.current){
    const e = state.current;
    const x = canvas.width/2; const y = canvas.height/2 - 20;
    // aura
    const rg = ctx.createRadialGradient(x,y,10,x,y,120);
    rg.addColorStop(0,e.color+'88'); rg.addColorStop(1,e.color+'00');
    ctx.fillStyle = rg; ctx.beginPath(); ctx.arc(x,y,120,0,Math.PI*2); ctx.fill();

    // enemy core
    ctx.beginPath(); ctx.fillStyle = e.color; ctx.arc(x,y,48,0,Math.PI*2); ctx.fill();
    // label
    ctx.fillStyle = '#001022'; ctx.font='700 18px Inter, Arial'; ctx.textAlign='center';
    ctx.fillText(e.name, x, y+6);

    // BOX with required EPIs
    ctx.fillStyle = 'rgba(0,0,0,0.45)'; ctx.fillRect(12,12,260,64);
    ctx.fillStyle = '#cfefff'; ctx.font='600 14px Inter, Arial'; ctx.textAlign='left';
    ctx.fillText(`Agente: ${e.name}`, 24, 36);
    ctx.fillStyle = '#9fc9ff'; ctx.font='600 12px Inter, Arial';
    ctx.fillText(`Requer: ${e.req.join(' + ')}`, 24, 56);
  } else {
    ctx.fillStyle = 'rgba(255,255,255,0.03)'; ctx.font='600 16px Inter, Arial';
    ctx.textAlign='center'; ctx.fillText('Aguardando agente... (clique Aplicar EPIs)', canvas.width/2, canvas.height/2);
  }

  // draw player platform
  ctx.fillStyle = 'rgba(0,255,220,0.06)'; ctx.beginPath();
  ctx.ellipse(canvas.width/2, canvas.height - 40, 220, 30, 0, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = '#00f6ff'; ctx.fillRect(canvas.width/2 - 28, canvas.height - 120, 56, 86);
  ctx.fillStyle = '#001011'; ctx.fillRect(canvas.width/2 - 22, canvas.height - 98, 44, 64);
  ctx.fillStyle = '#9ff'; ctx.font='600 12px Inter, Arial'; ctx.textAlign='center';
  ctx.fillText('Técnico', canvas.width/2, canvas.height - 40);
}

/* ---------- Initialization ---------- */
window.addEventListener('resize', ()=> {
  canvas.width = canvas.clientWidth; canvas.height = canvas.clientHeight; drawScene();
});
prepareEnemyQueue();
resetGame();
drawScene();

/* ---------- Controls: keyboard for desktop convenience ---------- */
document.addEventListener('keydown', e=>{
  if(e.key==='1') document.querySelector('.epi-btn[data-epi="Luvas"]').click();
  if(e.key==='2') document.querySelector('.epi-btn[data-epi="Avental"]').click();
  if(e.key==='3') document.querySelector('.epi-btn[data-epi="Máscara"]').click();
  if(e.key==='4') document.querySelector('.epi-btn[data-epi="Óculos"]').click();
  if(e.key==='5') document.querySelector('.epi-btn[data-epi="FaceShield"]').click();
  if(e.key==='6') document.querySelector('.epi-btn[data-epi="Sabao"]').click();
  if(e.key===' ') ui.applyBtn.click();
});
</script>
</body>
</html>
