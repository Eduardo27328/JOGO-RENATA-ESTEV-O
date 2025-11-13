<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Pyramid Quest — Aventuras na Pirâmide</title>
  <style>
    :root{
      --bg:#07101a; --panel:#081426; --accent:#f2c94c; --accent-2:#d97706; --muted:#9fb3c8; --glass:rgba(255,255,255,0.03);
      font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, Arial;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0; background: radial-gradient(circle at 20% 10%, #0b2a36 0%, #051019 40%, #02060a 100%); color:#eaf6ff; display:flex; align-items:center; justify-content:center; padding:24px}

    .container{width:100%; max-width:1100px}
    header{display:flex;align-items:center;gap:14px;margin-bottom:18px}
    .logo{width:68px;height:68px;border-radius:12px;background:linear-gradient(180deg,#ffd27a,#d98b13);display:flex;align-items:center;justify-content:center;font-weight:900;color:#061018;font-size:22px;box-shadow:0 8px 30px rgba(217,135,19,0.14)}
    h1{margin:0;font-size:22px}
    p.lead{margin:4px 0 0;color:var(--muted);font-size:14px}

    .game{display:grid;grid-template-columns:420px 1fr;gap:18px}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent); border-radius:14px; padding:16px; box-shadow:0 10px 30px rgba(2,6,23,.6); border:1px solid rgba(255,255,255,0.03)}

    /* Left: Pyramid */
    .pyramid-area{display:flex;flex-direction:column;align-items:center;gap:10px}
    .pyramid-viewport{width:360px; height:360px; position:relative; overflow:hidden; border-radius:10px; background: linear-gradient(180deg,#0b1723,#08121b); padding:18px}
    .pyramid-graphic{width:100%;height:100%;display:flex;flex-direction:column;align-items:center;justify-content:flex-end;gap:10px}

    .row{display:flex;gap:10px;justify-content:center;transition:transform .35s ease}
    .cell{width:64px;height:64px;border-radius:10px;background:var(--glass);display:flex;align-items:center;justify-content:center;font-weight:800;font-size:18px;color:var(--accent);border:2px dashed rgba(242,201,84,0.12);position:relative;cursor:pointer}
    .cell.filled{background:linear-gradient(180deg, rgba(242,201,84,0.12), rgba(242,201,84,0.03)); color:#081018; border-style:solid; border-color: rgba(242,201,84,0.22);}
    .cell .small{position:absolute;right:6px;top:6px;font-size:11px;color:var(--muted)}

    .controls{display:flex;gap:10px;margin-top:8px}
    button{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 12px;border-radius:10px;color:var(--muted);cursor:pointer;backdrop-filter:blur(3px)}
    button.primary{background:linear-gradient(180deg,var(--accent),var(--accent-2));color:#081018;border:none;font-weight:700}

    /* Right: Info */
    .info{display:flex;flex-direction:column;gap:12px}
    .topbar{display:flex;justify-content:space-between;align-items:center}
    .score{font-size:18px;font-weight:800;color:var(--accent)}
    .level{font-size:14px;color:var(--muted)}

    .blocks{display:flex;flex-wrap:wrap;gap:10px;padding:8px}
    .block{width:70px;height:70px;border-radius:10px;display:flex;align-items:center;justify-content:center;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:2px solid rgba(255,255,255,0.03);cursor:pointer;font-weight:900;font-size:18px;transition:transform .12s}
    .block:hover{transform:translateY(-6px)}
    .block.used{opacity:0.45;transform:none;cursor:default}

    .hint-box{padding:10px;border-radius:8px;background:rgba(0,0,0,0.18);color:var(--muted)}

    .footer-actions{display:flex;justify-content:space-between;align-items:center;margin-top:8px;color:var(--muted);font-size:13px}

    /* Modal / overlay */
    .overlay{position:fixed;inset:0;background:linear-gradient(180deg,rgba(0,0,0,0.45),rgba(0,0,0,0.6));display:flex;align-items:center;justify-content:center;z-index:60}
    .modal{width:94%;max-width:720px;background:linear-gradient(180deg,#061927,#06202a);border-radius:16px;padding:20px;border:1px solid rgba(255,255,255,0.04)}
    .modal h2{margin:0 0 8px}
    .btn-row{display:flex;gap:8px;justify-content:flex-end;margin-top:12px}

    /* confetti canvas */
    #confetti{position:fixed;left:0;top:0;pointer-events:none;z-index:50}

    /* responsive */
    @media (max-width:980px){.game{grid-template-columns:1fr} .pyramid-viewport{height:300px}}
  </style>
</head>
<body>
  <canvas id="confetti"></canvas>
  <div class="container">
    <header>
      <div class="logo">PQ</div>
      <div>
        <h1>Pyramid Quest — Aventuras na Pirâmide</h1>
        <p class="lead">Agora com animações, sons, partículas, sistema de níveis e uma aventura temática no Egito — pronto para impressionar seu professor.</p>
      </div>
    </header>

    <div class="game">
      <div class="card pyramid-area">
        <div class="pyramid-viewport" id="viewport">
          <div class="pyramid-graphic" id="pyramidGraphic">
            <!-- rows generated by JS -->
          </div>
        </div>

        <div style="display:flex;gap:10px;align-items:center;justify-content:center;margin-top:8px">
          <div style="text-align:center;color:var(--muted)">Alvo do nível<br><div style="font-weight:900;font-size:22px;color:var(--accent)" id="target">--</div></div>
        </div>

        <div class="controls">
          <button id="hintBtn">💡 Dica</button>
          <button id="resetBtn">🔁 Reset</button>
          <button class="primary" id="nextBtn">▶ Iniciar</button>
        </div>
      </div>

      <div class="card info">
        <div class="topbar">
          <div>
            <div class="level">Nível <span id="levelNum">0</span> / <span id="levelTotal">0</span></div>
            <div style="font-size:13px;color:var(--muted)" id="modeLabel">Modo</div>
          </div>
          <div class="score">Pontos: <span id="score">0</span></div>
        </div>

        <div class="hint-box" id="message">Bem-vindo! Clique em Iniciar para entrar na pirâmide.</div>

        <h3 style="margin:8px 0 4px">Blocos disponíveis</h3>
        <div class="blocks" id="blocks"></div>

        <div class="footer-actions">
          <div style="font-size:13px;color:var(--muted)">Atalhos: H = dica • R = reset • N = próximo</div>
          <div style="font-size:13px;color:var(--muted)">Tema: Aventura Egípcia</div>
        </div>
      </div>
    </div>

    <!-- Modal vitória -->
    <div id="winModal" style="display:none;" class="overlay">
      <div class="modal">
        <h2>🏆 Parabéns, Explorador!</h2>
        <p id="winText">Você completou o nível.</p>
        <div style="display:flex;gap:10px;align-items:center;justify-content:space-between;margin-top:12px">
          <div style="font-size:16px">Pontuação total: <strong id="finalScore">0</strong></div>
          <div>
            <button id="shareBtn">Copiar resumo</button>
            <button id="continueBtn" class="primary">Continuar</button>
          </div>
        </div>
      </div>
    </div>

  </div>

  <script>
    /*************** Data ***************/
    const levels = [
      {id:1, rows:[3], target:12, mode:'SOMA', hint:'Tente somar 3 blocos que deem 12. Combine 4+4+4 ou 5+4+3.'},
      {id:2, rows:[3,2], target:18, mode:'SOMA', hint:'A base tem mais espaços — experimente usar blocos maiores na base.'},
      {id:3, rows:[4,3,2], target:30, mode:'ÁREA', hint:'Pense que cada bloco adiciona à soma total da pirâmide — visualize quantidade.'},
      {id:4, rows:[4,4], target:24, mode:'FRAÇÃO', hint:'Use números que completem frações facilmente (ex.: 12 + 6 + 6).'},
      {id:5, rows:[5,4,3,2,1], target:55, mode:'DESAFIO', hint:'Misture números grandes e pequenos para chegar em 55.'}
    ];

    let current = 0; let score = 0; let selectedBlock = null; let used = new Set();

    const pyramidGraphic = document.getElementById('pyramidGraphic');
    const blocksArea = document.getElementById('blocks');
    const targetEl = document.getElementById('target');
    const messageEl = document.getElementById('message');
    const levelNum = document.getElementById('levelNum');
    const levelTotal = document.getElementById('levelTotal');
    const scoreEl = document.getElementById('score');
    const modeLabel = document.getElementById('modeLabel');
    const winModal = document.getElementById('winModal');
    const winText = document.getElementById('winText');
    const finalScore = document.getElementById('finalScore');

    // confetti canvas
    const confettiCanvas = document.getElementById('confetti');
    const cctx = confettiCanvas.getContext('2d');
    let confettiPieces = [];

    function resizeConfetti(){ confettiCanvas.width = window.innerWidth; confettiCanvas.height = window.innerHeight; }
    window.addEventListener('resize', resizeConfetti); resizeConfetti();

    function spawnConfetti(){
      confettiPieces = [];
      for(let i=0;i<120;i++){
        confettiPieces.push({x:Math.random()*confettiCanvas.width, y:Math.random()*-confettiCanvas.height, vx:(Math.random()-0.5)*4, vy:2+Math.random()*6, size:4+Math.random()*8, rot:Math.random()*360, color: `hsl(${Math.random()*60+30}, 80%, 55%)`});
      }
      requestAnimationFrame(updateConfetti);
      setTimeout(()=>confettiPieces=[] , 4000);
    }
    function updateConfetti(){ cctx.clearRect(0,0,confettiCanvas.width,confettiCanvas.height); confettiPieces.forEach(p=>{ p.x+=p.vx; p.y+=p.vy; p.rot+=p.vx*2; cctx.save(); cctx.translate(p.x,p.y); cctx.rotate(p.rot*Math.PI/180); cctx.fillStyle=p.color; cctx.fillRect(-p.size/2,-p.size/2,p.size,p.size); cctx.restore(); }); if(confettiPieces.length) requestAnimationFrame(updateConfetti); }

    /*************** Sound: WebAudio tiny FX ***************/
    const audioCtx = new (window.AudioContext||window.webkitAudioContext)();
    function fx(freq, duration=0.08, type='sine', vol=0.08){ const o = audioCtx.createOscillator(); const g = audioCtx.createGain(); o.type = type; o.frequency.value = freq; g.gain.value = vol; o.connect(g); g.connect(audioCtx.destination); o.start(); o.stop(audioCtx.currentTime + duration); }
    function playClick(){ fx(880,0.04,'square',0.03); }
    function playPlace(){ fx(660,0.06,'sine',0.06); }
    function playError(){ fx(220,0.16,'sawtooth',0.08); }
    function playWin(){ fx(880,0.08,'sine',0.06); setTimeout(()=>fx(1320,0.12,'sine',0.06),90); }

    /*************** Utils ***************/
    function el(tag, cls, txt){const e=document.createElement(tag); if(cls) e.className=cls; if(txt!==undefined) e.textContent=txt; return e}

    /*************** Game Functions ***************/
    function startLevel(idx){
      current = idx; const lvl = levels[idx];
      // UI
      levelNum.textContent = idx+1; levelTotal.textContent = levels.length; modeLabel.textContent = lvl.mode; targetEl.textContent = lvl.target; messageEl.textContent = 'Coloque blocos até a soma total da pirâmide ser ' + lvl.target + '.';
      // generate pyramid
      pyramidGraphic.innerHTML = '';
      for(let r = lvl.rows.length-1; r>=0; r--){
        const row = el('div','row'); row.style.transform = 'translateY(0)';
        for(let i=0;i<lvl.rows[r];i++){
          const cell = el('div','cell','');
          cell.dataset.row = r; cell.dataset.index = i;
          cell.tabIndex = 0;
          const small = el('div','small','0'); small.style.opacity=0.0; cell.appendChild(small);
          cell.addEventListener('click', ()=>onCellClick(cell));
          pyramidGraphic.appendChild(row);
          row.appendChild(cell);
        }
      }
      // animate rows coming from bottom
      Array.from(document.querySelectorAll('.row')).forEach((row, i)=>{ row.style.transform='translateY(24px)'; row.style.opacity=0; setTimeout(()=>{ row.style.transform='translateY(0)'; row.style.opacity=1 }, i*80); });
      // generate blocks
      generateBlocks(lvl);
      used = new Set(); selectedBlock = null; scoreEl.textContent = score;
      document.getElementById('nextBtn').textContent = 'Reiniciar Nível';
    }

    function generateBlocks(lvl){ blocksArea.innerHTML=''; let pool=[]; const base = lvl.target; for(let i=1;i<=9;i++) pool.push(i); for(let i=10;i<=Math.min(24, Math.ceil(base/2)); i+=2) pool.push(i); pool = pool.sort(()=>Math.random()-0.5).slice(0,12);
      pool.forEach((n, id)=>{
        const b = el('div','block', n); b.dataset.value = n; b.dataset.id = id; b.title='Clique e depois clique em um espaço';
        b.addEventListener('click', ()=>onBlockClick(b));
        blocksArea.appendChild(b);
      });
    }

    function onBlockClick(b){ if(used.has(b.dataset.id)) return; document.querySelectorAll('.block').forEach(x=>x.classList.remove('selected')); b.classList.add('selected'); selectedBlock = b; playClick(); messageEl.textContent = 'Bloco ' + b.dataset.value + ' selecionado — clique em um espaço da pirâmide.'; }

    function onCellClick(cell){ if(!selectedBlock){ messageEl.textContent = 'Selecione um bloco antes de clicar em um espaço.'; playError(); return; }
      const bid = selectedBlock.dataset.id; if(used.has(bid)){ messageEl.textContent='Esse bloco já foi usado.'; playError(); return; }
      // animate placement
      cell.textContent = selectedBlock.dataset.value; cell.classList.add('filled'); cell.querySelector('.small').textContent = selectedBlock.dataset.value; cell.querySelector('.small').style.opacity=0.8;
      used.add(bid); selectedBlock.classList.add('used'); selectedBlock.classList.remove('selected'); selectedBlock = null; playPlace(); messageEl.textContent = 'Bloco colocado!'; checkProgress(); }

    function checkProgress(){ const lvl = levels[current]; const placedVals = Array.from(document.querySelectorAll('.cell')).map(c=>parseInt(c.textContent)||0); const sum = placedVals.reduce((a,b)=>a+b,0);
      if(sum === lvl.target){ // win
        score += 20 + (levels.length - current)*5; scoreEl.textContent = score; playWin(); spawnConfetti(); showWin(lvl, score);
      } else {
        const diff = lvl.target - sum; messageEl.textContent = 'Soma atual: ' + sum + ' • faltam ' + diff + ' para ' + lvl.target + '.';
      }
    }

    function showWin(lvl, sc){ winText.textContent = `Você completou o nível ${lvl.id} — objetivo ${lvl.target}!`; finalScore.textContent = sc; winModal.style.display='flex'; document.body.classList.add('modal-open'); }

    function resetLevel(){ startLevel(current); playClick(); }

    function nextLevelOrRestart(){ if(current < levels.length-1) startLevel(current+1); else { // game finished — show final modal with option to restart
        winText.textContent = '🎉 Você concluiu todos os níveis da Aventura!'; finalScore.textContent = score; winModal.style.display='flex'; }
    }

    // Buttons
    document.getElementById('hintBtn').addEventListener('click', ()=>{ messageEl.textContent = 'Dica: ' + levels[current].hint; playClick(); });
    document.getElementById('resetBtn').addEventListener('click', ()=>{ resetLevel(); });
    document.getElementById('nextBtn').addEventListener('click', ()=>{ if(document.getElementById('nextBtn').textContent==='Iniciar' || document.getElementById('nextBtn').textContent==='▶ Iniciar') startLevel(0); else nextLevelOrRestart(); });

    // Modal actions
    document.getElementById('continueBtn').addEventListener('click', ()=>{ winModal.style.display='none'; document.body.classList.remove('modal-open'); if(current < levels.length-1) startLevel(current+1); else startLevel(0); });
    document.getElementById('shareBtn').addEventListener('click', ()=>{
      const summary = `Pyramid Quest — resultado: pontuação ${score} após completar até o nível ${current+1}.`; navigator.clipboard.writeText(summary).then(()=>{ alert('Resumo copiado!'); });
    });

    // Keyboard
    document.addEventListener('keydown', (e)=>{ if(e.key==='h') document.getElementById('hintBtn').click(); if(e.key==='r') document.getElementById('resetBtn').click(); if(e.key==='n') document.getElementById('nextBtn').click(); if(e.key===' '){ // space = place random block for demo
        e.preventDefault(); const available = Array.from(document.querySelectorAll('.block')).find(b=>!used.has(b.dataset.id)); if(available) { available.click(); const empty = Array.from(document.querySelectorAll('.cell')).find(c=>!c.classList.contains('filled')); if(empty) empty.click(); }
      } });

    // Save/load
    window.addEventListener('beforeunload', ()=>{ localStorage.setItem('pyramidQuest_score', score); localStorage.setItem('pyramidQuest_level', current); });
    window.addEventListener('load', ()=>{ const s = parseInt(localStorage.getItem('pyramidQuest_score')||'0'); const l = parseInt(localStorage.getItem('pyramidQuest_level')||'0'); score = s||0; scoreEl.textContent = score; startLevel(0); });

    // Auto-start first level
    //startLevel(0);
  </script>
</body>
</html>
