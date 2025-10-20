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
    html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#071025);color:#dfe9f3}
    .app{display:grid;grid-template-columns:360px 1fr 360px;gap:18px;padding:22px;align-items:start}
    header{grid-column:1/-1;display:flex;justify-content:space-between;align-items:center;padding:12px 22px;background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);border-radius:var(--ui-radius);box-shadow:var(--shadow)}
    h1{font-size:18px;margin:0;color:var(--accent)}
    .controls{display:flex;gap:8px;align-items:center}
    .btn{background:var(--glass);border:1px solid rgba(255,255,255,0.03);padding:8px 12px;border-radius:9px;cursor:pointer;color:var(--muted)}
    .btn.primary{background:linear-gradient(90deg,#06344a,#0b6a6a);color:white;border:0}
    .panel{background:linear-gradient(180deg,var(--panel),rgba(255,255,255,0.02));padding:14px;border-radius:14px;box-shadow:var(--shadow);min-height:180px}
    .left{display:flex;flex-direction:column;gap:12px}

    /* Resources */
    .res{display:flex;flex-wrap:wrap;gap:10px}
    .res .tile{min-width:120px;padding:12px;border-radius:10px;background:var(--glass-2);border:1px solid rgba(255,255,255,0.03)}
    .big-num{font-weight:700;font-size:20px;color:white}
    .muted{color:var(--muted);font-size:12px}

    /* Tree area */
    .tree-wrap{position:relative;height:84vh;overflow:auto;padding:12px;border-radius:12px}
    svg.tree{width:100%;height:100%;display:block}

    .right{display:flex;flex-direction:column;gap:12px}
    .log{height:220px;overflow:auto;padding:8px;background:transparent;border-radius:10px}
    .log p{margin:6px 0;color:var(--muted);font-size:13px}

    /* Node styles */
    .node{fill:#0b1220;stroke:#12233a;stroke-width:2;filter:drop-shadow(0 4px 14px rgba(0,0,0,0.6))}
    .node.unlocked{fill:linear-gradient(180deg,#072a2a,#09323a)}

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
        <div style="display:flex;justify-content:space-between;align-items:center">
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
      <svg class="tree" viewBox="0 0 1400 1800" id="treeSvg" xmlns="http://www.w3.org/2000/svg"></svg>
      <div class="tooltip" id="tooltip" style="display:none"></div>
    </section>

    <section class="right">
      <div class="panel">
        <div style="display:flex;justify-content:space-between;align-items:center">
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
    /* ================================================================
       Empyreal Branches — a single-file upgrade-tree game
       Features:
        - SVG-based upgrade tree with branches and connections
        - Procedural nodes and branches
        - Persistent saves (localStorage) + import/export
        - Skill points, mana, prestige, quests, achievements
        - Auto-tick, offline progress estimation
        - Tooltips, previews, and visual unlock animations
       ================================================================ */

    // Utility helpers
    const qs = s => document.querySelector(s);
    const qsa = s => Array.from(document.querySelectorAll(s));
    const rand = (a,b)=>Math.floor(Math.random()*(b-a+1))+a;
    const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));

    // Game state skeleton
    let state = {
      mana: 250,
      skillPoints: 2,
      prestige: 0,
      nodes: {}, // keyed by id
      unlocked: new Set(),
      time: Date.now(),
      tick: 800,
      auto: false,
      quests: [],
      achievements: {},
      class: 'Novice',
      nodesData: [],
      lastSave: Date.now()
    };

    // Widget binds
    const manaEl = qs('#mana');
    const skillEl = qs('#skillPoints');
    const prestigeEl = qs('#prestige');
    const treeSvg = qs('#treeSvg');
    const tooltip = qs('#tooltip');
    const upgradeList = qs('#upgradeList');
    const questsEl = qs('#quests');
    const statsArea = qs('#statsArea');
    const logEl = qs('#log');
    const achGrid = qs('#achGrid');
    const classBadge = qs('#classBadge');

    // ---------------------- Node generation -------------------------
    // We'll procedurally generate a tree with layers. Each node has: id, x,y, title, description, costSP, costMana, effects, requirements
    function generateTree(){
      const layers = [ {y:220, count:1}, {y:460, count:3}, {y:760, count:4}, {y:1060,count:5}, {y:1400,count:6} ];
      const width = 1300;
      const nodes = [];

      layers.forEach((layer, li)=>{
        const gap = width/(layer.count+1);
        for(let i=0;i<layer.count;i++){
          const id = `n-${li}-${i}`;
          const title = generateNodeTitle(li,i);
          const desc = generateNodeDesc(li,i);
          const costSP = Math.max(1, Math.ceil((li+1)* (1 + Math.random()*1.2)));
          const costMana = Math.ceil((li+1)* (50 + Math.random()*150));
          const effects = generateEffects(li);
          const x = gap*(i+1);
          const y = layer.y + rand(-50,50);
          nodes.push({id,title,desc,costSP,costMana,x,y,layer:li,effects,unlocked:false,prereqs:[]});
        }
      });

      // link prereqs: nodes in layer n depend on one or two nodes in layer n-1
      nodes.forEach(n=>{
        if(n.layer===0) return;
        const prev = nodes.filter(p=>p.layer===n.layer-1);
        const choose = rand(1,Math.min(2,prev.length));
        const picks = [];
        while(picks.length<choose){
          const p = prev[rand(0,prev.length-1)];
          if(!picks.includes(p.id)) picks.push(p.id);
        }
        n.prereqs = picks;
      });

      // special boss/branch nodes
      const endNode = {id:'n-boss',title:'Empyreal Nexus',desc:'A shimmering core that amplifies all branches.',costSP:12,costMana:4000,x:width/2,y:1650,layer:5,effects:[{type:'prestige',value:1}],prereqs: nodes.filter(n=>n.layer===4).map(n=>n.id)};
      nodes.push(endNode);

      state.nodesData = nodes;
      state.nodes = Object.fromEntries(nodes.map(n=>[n.id, n]));
    }

    function generateNodeTitle(layer,i){
      const seeds = ['Lumen','Aegis','Glyph','Pulse','Echo','Veil','Shard','Spiral','Chrono','Frost','Flare','Harbor'];
      return seeds[(layer+i)%seeds.length] + ' ' + (['I','II','III','IV','V'][layer]||'Ω');
    }
    function generateNodeDesc(layer,i){
      const verbs = ['empowers','shields','weaves','accelerates','anchors','channels','refines'];
      return `This node ${verbs[layer%verbs.length]} nearby abilities and grants a passive bonus.`;
    }
    function generateEffects(layer){
      if(layer<1) return [{type:'manaPerSec',value:1+layer}];
      if(layer<3) return [{type:'manaPerSec',value:2+layer},{type:'skillGain',value:0}];
      return [{type:'manaPerSec',value:4+layer},{type:'critChance',value:0.02*layer}];
    }

    // ---------------------- Rendering -------------------------------
    function renderTree(){
      treeSvg.innerHTML = '';
      const defs = document.createElementNS('http://www.w3.org/2000/svg','defs');
      defs.innerHTML = `
        <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
          <feGaussianBlur stdDeviation="6" result="coloredBlur" />
          <feMerge>
            <feMergeNode in="coloredBlur"/>
            <feMergeNode in="SourceGraphic"/>
          </feMerge>
        </filter>
      `;
      treeSvg.appendChild(defs);

      // draw connections
      state.nodesData.forEach(n=>{
        n.prereqs.forEach(pid=>{
          const p = state.nodes[pid];
          if(!p) return;
          const path = document.createElementNS('http://www.w3.org/2000/svg','path');
          const dx = (n.x - p.x)/2;
          const d = `M ${p.x} ${p.y} C ${p.x+dx} ${p.y} ${n.x-dx} ${n.y} ${n.x} ${n.y}`;
          path.setAttribute('d',d);
          path.setAttribute('stroke', p.unlocked && n.unlocked ? '#5fffb0' : 'rgba(255,255,255,0.06)');
          path.setAttribute('stroke-width','3');
          path.setAttribute('fill','none');
          treeSvg.appendChild(path);
        })
      });

      // draw nodes
      state.nodesData.forEach(n=>{
        const g = document.createElementNS('http://www.w3.org/2000/svg','g');
        g.setAttribute('transform',`translate(${n.x-68},${n.y-34})`);
        g.setAttribute('data-id',n.id);

        const rect = document.createElementNS('http://www.w3.org/2000/svg','rect');
        rect.setAttribute('width','136');rect.setAttribute('height','68');rect.setAttribute('rx','12');rect.setAttribute('class','node');
        rect.setAttribute('fill', n.unlocked ? 'url(#grad-unlocked)' : '#06131f');
        rect.style.cursor='pointer';

        const title = document.createElementNS('http://www.w3.org/2000/svg','text');
        title.setAttribute('x','12');title.setAttribute('y','26');title.setAttribute('fill','#dff7ee');title.setAttribute('font-size','12');title.textContent=n.title;
        const subtitle = document.createElementNS('http://www.w3.org/2000/svg','text');
        subtitle.setAttribute('x','12');subtitle.setAttribute('y','46');subtitle.setAttribute('fill','#93a7b6');subtitle.setAttribute('font-size','11');subtitle.textContent=`SP:${n.costSP} M:${n.costMana}`;

        g.appendChild(rect);g.appendChild(title);g.appendChild(subtitle);

        // interactions
        g.addEventListener('mouseenter', e=>showTooltip(n,e));
        g.addEventListener('mouseleave', hideTooltip);
        g.addEventListener('click', e=>{ e.stopPropagation(); tryUnlock(n.id); });

        treeSvg.appendChild(g);
      });
    }

    // ---------------------- Tooltip & interactions ------------------
    function showTooltip(n,e){
      const unlocked = state.nodes[n.id].unlocked;
      const canAfford = state.skillPoints>=n.costSP && state.mana>=n.costMana && prereqsUnlocked(n);
      tooltip.style.display='block';
      tooltip.innerHTML = `<div style="font-weight:700;margin-bottom:6px">${n.title}</div>
        <div class="small">${n.desc}</div>
        <div style="margin-top:8px" class="small">Cost: <strong>${n.costSP} SP</strong> &middot; <strong>${n.costMana} Mana</strong></div>
        <div style="margin-top:6px" class="small">Prereqs: ${n.prereqs.length? n.prereqs.join(', '): 'None'}</div>
        <div style="margin-top:8px"><button class="btn" id="buyNode">${unlocked? 'Unlocked':'Unlock'}</button> <button class="btn" id="previewNode">Preview</button></div>
      `;
      positionTooltip(e.clientX,e.clientY);

      qs('#buyNode').addEventListener('click',()=>{tryUnlock(n.id);});
      qs('#previewNode').addEventListener('click',()=>{previewNode(n)});
    }
    function positionTooltip(x,y){
      const r = treeSvg.getBoundingClientRect();
      const tx = clamp(x - r.left + 20, 20, r.width - 300);
      const ty = clamp(y - r.top + 20, 20, r.height - 140);
      tooltip.style.left = tx+'px'; tooltip.style.top = ty+'px';
    }
    function hideTooltip(){ tooltip.style.display='none'; }

    // ---------------------- Unlock logic ----------------------------
    function prereqsUnlocked(n){
      return n.prereqs.every(pid=>state.nodes[pid] && state.nodes[pid].unlocked);
    }
    function tryUnlock(id){
      const n = state.nodes[id];
      if(!n) return;
      if(n.unlocked){ log(`Node ${n.title} already unlocked.`); return; }
      if(!prereqsUnlocked(n)){ log('Prerequisites not met.'); return; }
      if(state.skillPoints < n.costSP || state.mana < n.costMana){ log('Insufficient resources.'); return; }
      // spend
      state.skillPoints -= n.costSP; state.mana -= n.costMana; n.unlocked=true; state.unlocked.add(id);
      applyNodeEffects(n);
      log(`Unlocked ${n.title} — gained effects.`);
      checkAchievements(); renderAll(); save();
    }

    // Effects
