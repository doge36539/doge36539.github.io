<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Empyreal Branches — Upgrade Tree Game</title>
  <style>
    :root{
      --bg:#0b1020;--panel:#0f1724;--accent:#6de6c1;--muted:#9aa7bf;--glass: rgba(255,255,255,0.03);
      --good:#68e07b;--bad:#ff7b7b;--glass-2: rgba(255,255,255,0.02);
      --ui-radius:12px;--shadow: 0 10px 30px rgba(2,6,23,0.7);
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
    }
    *{box-sizing:border-box}
    html,body{
      margin:0;
      min-height:100%;
      display:flex;
      justify-content:center;
      background:linear-gradient(180deg,var(--bg),#071025);
      color:#dfe9f3;
    }
    .app{
      display:grid;
      grid-template-columns:360px 1fr 360px;
      gap:18px;
      padding:22px;
      max-width:1400px;
      width:100%;
      align-items:start;
      justify-items:center;
    }
    header{
      grid-column:1/-1;
      display:flex;
      justify-content:space-between;
      align-items:center;
      padding:12px 22px;
      background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);
      border-radius:var(--ui-radius);
      box-shadow:var(--shadow);
    }
    h1{font-size:18px;margin:0;color:var(--accent);text-align:center}
    .controls{display:flex;gap:8px;align-items:center}
    .btn{background:var(--glass);border:1px solid rgba(255,255,255,0.03);padding:8px 12px;border-radius:9px;cursor:pointer;color:var(--muted)}
    .btn.primary{background:linear-gradient(90deg,#06344a,#0b6a6a);color:white;border:0}
    .panel{background:linear-gradient(180deg,var(--panel),rgba(255,255,255,0.02));padding:14px;border-radius:14px;box-shadow:var(--shadow);display:flex;flex-direction:column;align-items:center;text-align:center}
    .left{display:flex;flex-direction:column;gap:12px}

    /* Resources */
    .res{display:flex;flex-wrap:wrap;gap:10px;justify-content:center}
    .res .tile{min-width:120px;padding:12px;border-radius:10px;background:var(--glass-2);border:1px solid rgba(255,255,255,0.03)}
    .big-num{font-weight:700;font-size:20px;color:white}
    .muted{color:var(--muted);font-size:12px}

    /* Tree area */
    .tree-wrap{position:relative;height:84vh;overflow:hidden;padding:12px;border-radius:12px;width:100%}
    svg.tree{width:100%;height:100%;display:block;cursor:grab}

    .right{display:flex;flex-direction:column;gap:12px}
    .log{height:220px;overflow:auto;padding:8px;background:transparent;border-radius:10px}
    .log p{margin:6px 0;color:var(--muted);font-size:13px}

    /* Node styles */
    .node{fill:#0b1220;stroke:#12233a;stroke-width:2;filter:drop-shadow(0 4px 14px rgba(0,0,0,0.6));transition:all 0.2s;}
    .node.unlocked{fill:#0a2a2a;stroke:#6de6c1}

    /* Tooltip */
    .tooltip{position:absolute;pointer-events:none;background:rgba(7,12,20,0.96);padding:10px;border-radius:8px;min-width:220px;border:1px solid rgba(255,255,255,0.03);box-shadow:0 6px 20px rgba(5,10,20,0.7);z-index:40}

    /* Upgrade list */
    .upgrades{display:grid;grid-template-columns:1fr;gap:8px}
    .upgrade-row{display:flex;justify-content:space-between;align-items:center;padding:8px;border-radius:8px;background:var(--glass)}

    /* Achievements */
    .ach-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:8px}
    .ach{padding:8px;border-radius:8px;background:rgba(255,255,255,0.02);text-align:center}

    /* Footer small */
    footer.small{grid-column:1/-1;text-align:center;color:var(--muted);font-size:12px;padding-top:8px}

    /* Responsive */
    @media (max-width:1100px){.app{grid-template-columns:1fr;}
      header{flex-direction:column;gap:8px}
      .left,.right{order:0}
    }

    /* Decorative */
    .spark{position:absolute;width:10px;height:10px;border-radius:50%;background:radial-gradient(circle,#fff,#00ffd6);opacity:0.06}

    /* small helpers */
    .small{font-size:12px;color:var(--muted)}

    /* slider */
    .row{display:flex;gap:10px;align-items:center}
    input[type=range]{width:120px}

    /* class badges */
    .badge{padding:6px 10px;border-radius:999px;background:rgba(255,255,255,0.03);font-weight:600}

    /* cursor */
    #treeSvg:active { cursor:grabbing; }
  </style>
</head>
<body>
<header>
  <h1>Empyreal Branches <span class="small">— an upgrade tree odyssey</span></h1>
  <div class="controls">
    <div class="badge" id="classBadge">Class: Novice</div>
    <button class="btn" id="exportBtn">Export Save</button>
    <button class="btn" id="importBtn">Import Save</button>
    <button class="btn primary" id="resetBtn">Reset</button>
  </div>
</header>

<main class="app">
  <section class="left">
    <div class="panel">
      <div style="display:flex;justify-content:space-between;align-items:center;width:100%">
        <div>
          <div class="muted">Resources</div>
          <div class="res" id="resList">
            <div class="tile"><div class="muted">Mana</div><div class="big-num" id="mana">0</div></div>
            <div class="tile"><div class="muted">Skill Points</div><div class="big-num" id="skillPoints">0</div></div>
            <div class="tile"><div class="muted">Prestige</div><div class="big-num" id="prestige">0</div></div>
          </div>
        </div>
        <div style="text-align:right;min-width:140px">
          <div class="muted">Tick Speed</div>
          <div class="small">Auto: <span id="autoStatus">Off</span></div>
          <div class="row"><input type="range" id="tickRange" min="200" max="2000" step="50" value="800"><div class="small" id="tickLabel">800ms</div></div>
        </div>
      </div>
    </div>

    <div class="panel" style="min-height:220px">
      <div class="muted">Active Quests</div>
      <div id="quests"></div>
      <div style="margin-top:10px" class="small">Quests are procedural mini-goals that reward skill points or mana. Complete them to unlock tree branches faster.</div>
    </div>

    <div class="panel">
      <div class="muted">Upgrades</div>
      <div class="upgrades" id="upgradeList"></div>
    </div>
  </section>

  <section class="panel tree-wrap" id="treePanel">
    <svg class="tree" viewBox="0 0 1400 1800" id="treeSvg" xmlns="http://www.w3.org/2000/svg">
      <g id="treeGroup"></g>
    </svg>
    <div class="tooltip" id="tooltip" style="display:none"></div>
  </section>

  <section class="right">
    <div class="panel">
      <div style="display:flex;justify-content:space-between;align-items:center;width:100%">
        <div><div class="muted">Player Stats</div><div id="statsArea"></div></div>
        <div style="text-align:right"><div class="muted">Fast Actions</div><button class="btn" id="gainMana">Gain 100 Mana</button></div>
      </div>
    </div>

    <div class="panel">
      <div class="muted">Achievements</div>
      <div class="ach-grid" id="achGrid"></div>
    </div>

    <div class="panel">
      <div class="muted">Game Log</div>
      <div class="log" id="log"></div>
      <div style="margin-top:8px" class="small">Tip: hover nodes to preview cost, click to unlock (if you have skill points).</div>
    </div>
  </section>

  <footer class="small">Built with love — open-source style. Save stored locally.</footer>
</main>

<script>
  const qs = s => document.querySelector(s);
  const qsa = s => Array.from(document.querySelectorAll(s));
  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));

  // ---------------- State -------------------
  let state = {
    mana:250, skillPoints:2, prestige:0,
    nodes:{}, unlocked:new Set(), tick:800,
    auto:false, quests:[], achievements:{}, class:'Novice'
  };

  const manaEl = qs('#mana'), skillEl = qs('#skillPoints'), prestigeEl = qs('#prestige');
  const treeSvg = qs('#treeSvg'), tooltip = qs('#tooltip');

  // ---------------- Tree zoom/drag -------------------
  const treeGroup = qs('#treeGroup');
  let treeZoom=1, treeX=0, treeY=0, isDragging=false, dragStart={x:0,y:0};

  function updateTreeTransform(){ treeGroup.setAttribute('transform',`translate(${treeX},${treeY}) scale(${treeZoom})`); }

  treeSvg.addEventListener('wheel', e=>{
    e.preventDefault();
    const rect = treeSvg.getBoundingClientRect();
    const mx = e.clientX - rect.left;
    const my = e.clientY - rect.top;
    const oldZoom = treeZoom;
    treeZoom *= e.deltaY<0?1.1:0.9;
    treeZoom = clamp(treeZoom,0.5,3);
    treeX -= (mx - treeX)*(treeZoom/oldZoom-1);
    treeY -= (my - treeY)*(treeZoom/oldZoom-1);
    updateTreeTransform();
  });

  treeSvg.addEventListener('mousedown', e=>{ isDragging=true; dragStart.x=e.clientX-treeX; dragStart.y=e.clientY-treeY; });
  window.addEventListener('mousemove', e=>{ if(isDragging){ treeX=e.clientX-dragStart.x; treeY=e.clientY-dragStart.y; updateTreeTransform(); } });
  window.addEventListener('mouseup', ()=>{ isDragging=false; });

  // Initial update
  updateTreeTransform();
</script>
</body>
</html>
