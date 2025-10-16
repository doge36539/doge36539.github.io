<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Pixel Deep — Full Fishing (30+ Fish)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser.min.js"></script>
  <style>
    html,body { margin:0; height:100%; background: #04121a; font-family: monospace; color:#e6eef6; }
    #ui { position: fixed; left:8px; top:8px; display:flex; gap:8px; align-items:center; z-index:9999; }
    .pill { background: rgba(10,14,25,0.9); border:1px solid rgba(80,110,140,0.18); padding:6px 10px; border-radius:10px; box-shadow: 0 4px 14px rgba(0,0,0,0.4);}
    #sidebar { position: fixed; right:8px; top:8px; width:360px; z-index:9999; }
    #tabs { display:flex; gap:6px; margin-bottom:6px; }
    .tabbtn { flex:1; padding:8px; text-align:center; background: rgba(9,12,20,0.85); border-radius:10px; cursor:pointer; border:1px solid rgba(80,110,140,0.12);}
    .panel { background: rgba(6,10,18,0.95); padding:12px; border-radius:10px; height:430px; overflow:auto; box-shadow: 0 6px 20px rgba(0,0,0,0.55);}
    .urow { display:flex; justify-content:space-between; gap:8px; align-items:center; margin:8px 0;}
    .ubtn { background: rgba(30,40,60,0.9); padding:6px 8px; border-radius:8px; cursor:pointer; border:1px solid rgba(120,160,200,0.06);}
    #controls { position: fixed; left:8px; bottom:8px; z-index:9999; display:flex; gap:8px; align-items:center;}
    button { font-family: monospace; color:inherit; }
    #msg { position: fixed; left:50%; transform:translateX(-50%); bottom:18px; background: rgba(0,0,0,0.6); padding:6px 12px; border-radius:8px; display:none; z-index:9999;}
    .small { font-size:12px; opacity:.86; }
    .price { color:#9ce38f; }
    .disabled { opacity:0.5; pointer-events:none; }
    a { color:#9ecbff; }
  </style>
</head>
<body>
  <div id="ui">
    <div class="pill">💰 <b>Cash</b>: $<span id="cash">0</span></div>
    <div class="pill">🎣 <b>Caught</b>: <span id="caught">0</span></div>
    <div class="pill">📏 <b>Depth</b>: <span id="depth">0</span> m</div>
    <div class="pill">🔧 <b>Rod</b>: <span id="dur">100</span>%</div>
  </div>

  <div id="sidebar">
    <div id="tabs">
      <div class="tabbtn" data-tab="upg">Upgrades</div>
      <div class="tabbtn" data-tab="index">Fish Index</div>
      <div class="tabbtn" data-tab="settings">Settings</div>
    </div>
    <div id="panel" class="panel">
      <div id="panel-inner">Dock near the shop to access Upgrades / Index / Settings.</div>
    </div>
  </div>

  <div id="controls">
    <button class="ubtn" id="dropBtn">Drop Hook (E)</button>
    <button class="ubtn" id="reelBtn">Reel (Space)</button>
    <button class="ubtn" id="autoReelBtn">Auto-Reel: OFF</button>
    <button class="ubtn" id="dockBtn">Dock Info</button>
  </div>

  <div id="msg"></div>

<script>
/* ---------------- CONFIG & DATA ---------------- */
const WORLD = { width: 8000, seaHeight: 120, depth: 6200, tile: 32, seed: Math.random()*999999 };
const START = {
  cash: 150,
  upgrades: {
    lineLength:1, hookSpeed:1, reelSpeed:1, strength:1, boatSpeed:1,
    lure:1, sonar:1, durability:1, spawnRate:1, rarityBoost:1
  },
  rodDur: 100, fishdex: {}, caught:0
};

/* 30+ fish: name, minDepth, maxDepth, basePrice, weightKg, rarity(1-5), biome */
const FISH = [
  ["Minnow", 40, 200, 5, 0.2, 1, "surface"],
  ["Anchovy", 60, 260, 6, 0.25, 1, "surface"],
  ["Sardine", 80, 300, 7, 0.3, 1, "surface"],
  ["Perch", 120, 500, 10, 0.6, 1, "kelp"],
  ["Trout", 160, 700, 14, 1.0, 1, "kelp"],
  ["Bass", 160, 900, 18, 1.4, 2, "kelp"],
  ["Mullet", 220, 1100,22, 1.6, 2, "kelp"],
  ["Herring", 240, 1100,20, 0.9,2,"kelp"],
  ["Snapper", 320, 1300,28, 2.0,2,"reef"],
  ["Catfish", 400, 1400,30, 2.4,2,"river"],
  ["Pike", 500, 1500,33, 2.6,2,"river"],
  ["Salmon", 600, 1600,36, 3.0,2,"open"],
  ["Carp", 700, 1700,32, 2.2,2,"mud"],
  ["Eel", 800, 1900,44, 1.8,3,"cave"],
  ["Grouper",900,2200,55, 4.2,3,"reef"],
  ["Swordtail",900,2100,48, 2.8,3,"reef"],
  ["Tuna",1100,2600,70, 6.0,3,"open"],
  ["Barracuda",1200,2700,76,5.2,3,"open"],
  ["Mahi-Mahi",1300,2800,80,4.8,3,"open"],
  ["Stingray",1600,3200,95,7.0,4,"sandy"],
  ["Halibut",1700,3400,100,8.0,4,"sandy"],
  ["Marlin",1900,3600,120,9.0,4,"deep"],
  ["Anglerfish",2400,4200,150,6.0,4,"abyss"],
  ["Black Cod",2600,4400,165,7.2,4,"abyss"],
  ["Coelacanth",2800,4800,210,8.5,5,"abyss"],
  ["Giant Squid",3200,5200,260,10.0,5,"abyss"],
  ["Oarfish",3400,5400,280,9.5,5,"abyss"],
  ["Opah",3600,5600,260,8.2,5,"abyss"],
  ["Gulper Eel",3800,5800,300,7.5,5,"abyss"],
  ["Blobfish",4000,5900,320,6.8,5,"abyss"],
  ["Abyssal Shark",4300,6000,380,12.0,5,"abyss"],
  ["Leviathan",4700,6200,600,20.0,5,"mythic"]
];

const UPG_META = {
  lineLength:  { label:"Line Length", base: 150, step:1.18, price:60 },
  hookSpeed:   { label:"Hook Move Speed", base: 70, step:1.12, price:50 },
  reelSpeed:   { label:"Reel Speed", base: 60, step:1.12, price:60 },
  strength:    { label:"Strength", base:2.0, step:1.22, price:80 },
  boatSpeed:   { label:"Boat Speed", base:80, step:1.10, price:45 },
  lure:        { label:"Lure Power", base:80, step:1.12, price:70 },
  sonar:       { label:"Sonar Range", base:90, step:1.15, price:60 },
  durability:  { label:"Rod Durability", base:1.0, step:1.16, price:95 },
  spawnRate:   { label:"Spawn Rate", base:1.0, step:1.15, price:90 },
  rarityBoost: { label:"Rarity Bias", base:1.0, step:1.12, price:110 }
};

/* ---------------- SAVE/LOAD ---------------- */
function save() { localStorage.setItem('pixeldeep_full', JSON.stringify(state)); }
function loadSave(){ try{ return JSON.parse(localStorage.getItem('pixeldeep_full')) || null; } catch(e){ return null; } }
const state = loadSave() || JSON.parse(JSON.stringify(START));

/* ----------------- UTILS & UI ---------------- */
const cashSpan = ()=>document.getElementById('cash');
const caughtSpan = ()=>document.getElementById('caught');
const depthSpan = ()=>document.getElementById('depth');
const durSpan = ()=>document.getElementById('dur');
const panelInner = ()=>document.getElementById('panel-inner');
const msgBox = document.getElementById('msg');

function showMsg(t,ms=1600){ msgBox.style.display='block'; msgBox.textContent=t; clearTimeout(showMsg._t); showMsg._t = setTimeout(()=>msgBox.style.display='none', ms); }
function format(n){ return Math.floor(n); }

/* Toggle / Tabs (shop-only) */
let docked = false;
document.querySelectorAll('.tabbtn').forEach(b=>{
  b.addEventListener('click', ()=>{
    if (!docked) { panelInner().innerHTML = '<div class="small">Dock at the shop to access this panel.</div>'; return; }
    const t = b.dataset.tab;
    if (t==='upg') renderUpgrades();
    if (t==='index') renderIndex();
    if (t==='settings') renderSettings();
  });
});

/* Control buttons */
let autoReel = false;
document.getElementById('dropBtn').onclick = ()=> window.gameDropHook && window.gameDropHook();
document.getElementById('reelBtn').onclick = ()=> window.gameStartReel && window.gameStartReel();
document.getElementById('autoReelBtn').onclick = ()=> { autoReel = !autoReel; document.getElementById('autoReelBtn').textContent = 'Auto-Reel: ' + (autoReel? 'ON':'OFF'); };
document.getElementById('dockBtn').onclick = ()=> { if (docked) showMsg('Docked: open Upgrades or Index.'); else showMsg('Not docked. Sail to the shop (left area).'); };

/* ---------------- PHASER SETUP ---------------- */
const config = {
  type: Phaser.WEBGL, width: 1280, height: 720,
  pixelArt: true, backgroundColor: 0x02121a,
  physics:{ default:'arcade', arcade:{ gravity:{y:0}, debug:false } },
  scene: { preload, create, update },
  render: { roundPixels:true }
};
const game = new Phaser.Game(config);

/* seeded noise helpers */
function rndSeeded(x,y){ const s = Math.sin((x*127.1 + y*311.7 + WORLD.seed)|0) * 43758.5453123; return s - Math.floor(s); }
function smoothNoise(x,y){ const xi=Math.floor(x), yi=Math.floor(y); const xf=x-xi, yf=y-yi;
  const n00=rndSeeded(xi,yi), n10=rndSeeded(xi+1,yi), n01=rndSeeded(xi,yi+1), n11=rndSeeded(xi+1,yi+1);
  const u=xf*xf*(3-2*xf), v=yf*yf*(3-2*yf);
  return (1-u)*(1-v)*n00 + u*(1-v)*n10 + (1-u)*v*n01 + u*v*n11;
}
function caveNoise(wx,wy){ let n=0, amp=1, freq=0.007; for (let o=0;o<4;o++){ n += amp * smoothNoise(wx*freq, wy*freq); amp*=0.5; freq*=2; } return n; }

/* shared vars */
let boat, hook, lineGraphics, fishGroup, wallsGroup, shopBuilding, camera, sonarCircle;
let carrying=null, hooked=false, hookDropped=false, hookBroken=false;
let spawnTimer=0, keys;

/* inlined tiny pixel textures */
function generateTextures(scene){
  // boat base
  scene.textures.generate('boat', { data:[
    '....333333....',
    '...33333333...',
    '..3333333333..',
    '.333339993333.',
    '33333999333333',
    '33333333333333',
    '...4444444....'
  ], pixelWidth:2, palette:{3:'#6e3f28',9:'#ffe4b3',4:'#2e7bbf'}});
  // shop (reuse boat atlas tinted later)
  // hook
  scene.textures.generate('hook', { data:[
    '....5....',
    '...555...',
    '..55555..',
    '...5.5...',
    '....5....'
  ], pixelWidth:2, palette:{5:'#d1d1d1'}});
  // fishes: multiple palettes
  scene.textures.generate('fA',{ data:['.666.','66666','66666','.666.'], pixelWidth:2, palette:{6:'#8fe37b'}});
  scene.textures.generate('fB',{ data:['.777.','77777','77777','.777.'], pixelWidth:2, palette:{7:'#e59b6f'}});
  scene.textures.generate('fC',{ data:['.888.','88888','88888','.888.'], pixelWidth:2, palette:{8:'#7fb7ff'}});
  scene.textures.generate('kelp',{ data:['.9.','999','.99'], pixelWidth:2, palette:{9:'#1d7a3a'}});
  scene.textures.generate('vent',{ data:['..a..','..aaa..','.aaaaa.','..aaa..','..a..'], pixelWidth:2, palette:{a:'#ff944d'}});
  scene.textures.generate('rock',{ data:['9999','9999','9999','9999'], pixelWidth:2, palette:{9:'#242634'}});
  scene.textures.generate('bubble',{ data:['b'], pixelWidth:2, palette:{b:'#bfe9ff'}});
}

/* --------------- SCENE LIFECYCLE --------------- */
function preload(){ }
function create(){
  const scene = this;
  generateTextures(scene);

  // world bounds
  scene.cameras.main.setBounds(0, 0, WORLD.width, WORLD.seaHeight + WORLD.depth);
  scene.physics.world.setBounds(0, 0, WORLD.width, WORLD.seaHeight + WORLD.depth);

  // background rectangles to simulate gradient water & surface
  scene.add.rectangle(WORLD.width/2, WORLD.seaHeight/2, WORLD.width, WORLD.seaHeight, 0x08324a).setDepth(-10);
  scene.add.rectangle(WORLD.width/2, (WORLD.seaHeight + WORLD.depth)/2, WORLD.width, WORLD.depth, 0x001826).setDepth(-11);

  // subtle caustics overlay
  const caust = scene.add.tileSprite(0,0, WORLD.width, WORLD.seaHeight + WORLD.depth, null).setOrigin(0).setAlpha(0.04).setDepth(-9);

  // shop building (left-side)
  shopBuilding = scene.physics.add.staticImage(220, WORLD.seaHeight - 58, 'boat').setScale(1.2);
  shopBuilding.setTint(0x997b58);
  shopBuilding.setData('isShop', true);

  // player boat
  boat = scene.physics.add.sprite(WORLD.width/2, WORLD.seaHeight - 20, 'boat').setScale(1.0);
  boat.body.allowGravity = false; boat.setImmovable(true);

  // hook attached to boat initially
  hook = scene.physics.add.sprite(boat.x, boat.y + 32, 'hook');
  hook.setDepth(6); hook.body.setCircle(6);
  hook.setCollideWorldBounds(true);

  lineGraphics = scene.add.graphics({ lineStyle: { width: 2, color: 0xcfeffb } });

  sonarCircle = scene.add.circle(hook.x, hook.y, UPG_META.sonar.base, 0x66ffff, 0.06).setVisible(false);

  // terrain & walls
  wallsGroup = scene.physics.add.staticGroup();
  generateTerrain(scene);

  // fish group
  fishGroup = scene.physics.add.group();
  for (let i=0;i<60;i++) spawnFish(scene);

  // collisions + overlaps
  scene.physics.add.collider(hook, wallsGroup);
  scene.physics.add.collider(fishGroup, wallsGroup);
  scene.physics.add.overlap(hook, fishGroup, (h,f)=>{
    if (!carrying && hookDropped && !hookBroken) {
      if (Math.random() < 0.92) { carrying = f; hooked = true; f.setTint(0xffe066); f.body.setVelocity(0,0); f.setDepth(7); showMsg('Fish hooked: ' + f.getData('name')); }
    }
  });

  // input
  keys = scene.input.keyboard.addKeys('W,A,S,D,E,SPACE,LEFT,RIGHT');
  scene.input.keyboard.on('keydown-E', ()=> dropHook());
  scene.input.keyboard.on('keydown-SPACE', ()=> startReel());

  // camera follow hook
  camera = scene.cameras.main; camera.startFollow(hook, true, 0.06, 0.06); camera.setZoom(1.35);

  // bubble ambience
  scene.time.addEvent({ delay: 650, loop:true, callback: ()=> spawnBubble(scene) });

  updateUI();
}

/* ---------------- UPDATE ---------------- */
function update(time, delta){
  const dt = delta/1000;
  const scene = game.scene.scenes[0];

  // boat movement (arrow keys)
  const curs = scene.input.keyboard.createCursorKeys();
  const bSpeed = UPG_META.boatSpeed.base * (state.upgrades ? state.upgrades.boatSpeed : 1);
  if (curs.left.isDown) { boat.x = Math.max(40, boat.x - bSpeed * dt); boat.body.updateFromGameObject(); }
  if (curs.right.isDown) { boat.x = Math.min(WORLD.width - 40, boat.x + bSpeed * dt); boat.body.updateFromGameObject(); }

  // hook behavior
  if (hookDropped && !hookBroken) {
    let mx=0,my=0;
    if (keys.A.isDown) mx -= 1;
    if (keys.D.isDown) mx += 1;
    if (keys.W.isDown) my -= 1;
    if (keys.S.isDown) my += 1;
    const hSpeed = UPG_META.hookSpeed.base * (state.upgrades ? state.upgrades.hookSpeed : 1);
    hook.x += mx * hSpeed * dt;
    hook.y += my * hSpeed * dt;

    // clamp to line length
    const baseLen = UPG_META.lineLength.base * (state.upgrades ? state.upgrades.lineLength : 1);
    const dx = hook.x - boat.x, dy = hook.y - boat.y; const dist = Math.hypot(dx,dy);
    if (dist > baseLen) { const k = baseLen / dist; hook.x = boat.x + dx * k; hook.y = boat.y + dy * k; }
    if (hook.y < WORLD.seaHeight + 6) hook.y = WORLD.seaHeight + 6;
  } else {
    // attached to boat
    hook.x = boat.x; hook.y = boat.y + 32;
  }

  // auto-reel behavior: if enabled & dropped & no fish hooked
  if (autoReel && hookDropped && !hookBroken && !hooked) {
    const r = UPG_META.reelSpeed.base * (state.upgrades ? state.upgrades.reelSpeed : 1);
    hook.y = Math.max(hook.y - r * dt * 12, WORLD.seaHeight + 8);
    if (hook.y <= WORLD.seaHeight + 12) { hookDropped=false; showMsg('Hook retracted (auto)'); }
  }

  // carrying fish follow hook and chance to snap
  if (carrying) {
    const wt = carrying.getData('wt');
    const str = UPG_META.strength.base * (state.upgrades ? state.upgrades.strength : 1);
    const over = Math.max(0, wt - str);
    if (over > 0 && Math.random() < 0.0007 * over * (1 / (state.upgrades ? state.upgrades.durability : 1))) {
      state.rodDur = Math.max(0, (state.rodDur || 100) - Math.ceil(over * 9));
      carrying.clearTint(); carrying = null; hooked = false;
      showMsg('Rod strained! Durability decreased.');
      if ((state.rodDur || 100) <= 0) { hookBroken = true; showMsg('Rod snapped! Repair at shop.'); }
      save(); updateUI();
    } else {
      carrying.x += (hook.x - carrying.x) * 0.38;
      carrying.y += (hook.y - carrying.y) * 0.38;
    }
  }

  // auto-sell at surface
  if (carrying && hook.y <= WORLD.seaHeight + 12) {
    const val = carrying.getData('price');
    state.cash = (state.cash || 0) + val;
    state.caught = (state.caught || 0) + 1;
    const nm = carrying.getData('name'); state.fishdex[nm] = true;
    carrying.destroy(); carrying = null; hooked = false;
    showMsg('Sold catch for $' + val);
    save(); updateUI();
  }

  // fish behavior + spawning
  spawnTimer += delta/1000;
  const interval = Math.max(0.5, 1.8 / (state.upgrades ? state.upgrades.spawnRate : 1));
  if (spawnTimer > interval && fishGroup.countActive(true) < 180) { spawnFish(game.scene.scenes[0]); spawnTimer = 0; }

  fishGroup.children.iterate(f=>{
    if (!f) return;
    const lure = UPG_META.lure.base * (state.upgrades ? state.upgrades.lure : 1);
    const d = Phaser.Math.Distance.Between(f.x,f.y, hook.x, hook.y);
    if (d < Math.min(320, lure) && !hookBroken) {
      const ang = Phaser.Math.Angle.Between(f.x,f.y, hook.x, hook.y);
      f.body.velocity.x += Math.cos(ang) * 10; f.body.velocity.y += Math.sin(ang) * 10;
    } else {
      f.body.velocity.x += Math.sin((f.x+f.getData('seed'))*0.01) * 2;
    }
    f.body.velocity.x = Phaser.Math.Clamp(f.body.velocity.x, -60, 60);
    f.body.velocity.y = Phaser.Math.Clamp(f.body.velocity.y, -40, 40);
    const minD = f.getData('minD') + WORLD.seaHeight; const maxD = f.getData('maxD') + WORLD.seaHeight;
    if (f.y < minD) f.body.velocity.y += 15;
    if (f.y > maxD) f.body.velocity.y -= 15;
    if (f.x < -120 || f.x > WORLD.width + 120) f.destroy();
  });

  // draw line
  lineGraphics.clear();
  lineGraphics.lineStyle(2, 0xcfeffb, 0.95);
  lineGraphics.beginPath(); lineGraphics.moveTo(boat.x, boat.y + 6); lineGraphics.lineTo(hook.x, hook.y); lineGraphics.strokePath();

  // sonar circle
  sonarCircle.setPosition(hook.x, hook.y);
  sonarCircle.setRadius(UPG_META.sonar.base * (state.upgrades ? state.upgrades.sonar : 1));
  sonarCircle.setVisible((state.upgrades && state.upgrades.sonar>1));

  // dock detection
  const dockDist = Phaser.Math.Distance.Between(boat.x, boat.y, shopBuilding.x, shopBuilding.y);
  const wasDocked = docked;
  docked = dockDist < 110;
  if (docked && !wasDocked) { showMsg('Docked at shop — open Upgrades / Index / Settings'); }
  if (!docked && wasDocked) { showMsg('Left shop'); }

  updateUI();
}

/* ---------------- ACTIONS: Drop/Reel ---------------- */
function dropHook(){
  if (hookBroken) { showMsg('Rod is broken — repair at shop.'); return; }
  if (!hookDropped) { hookDropped = true; hook.x = boat.x; hook.y = boat.y + 36; showMsg('Hook dropped.'); }
  else showMsg('Hook already dropped.');
}
function startReel(){
  if (hookBroken) { showMsg('Rod is broken — repair at shop.'); return; }
  const r = UPG_META.reelSpeed.base * (state.upgrades ? state.upgrades.reelSpeed : 1);
  const interval = setInterval(()=>{
    if (!hookDropped) { clearInterval(interval); return; }
    hook.y = Math.max(hook.y - r * 0.05, WORLD.seaHeight + 8);
    hook.x += (boat.x - hook.x) * 0.03;
    if (hook.y <= WORLD.seaHeight + 12) { hookDropped=false; clearInterval(interval); showMsg('Hook retracted.'); }
  }, 28);
}

/* expose for UI buttons & keyboard */
window.gameDropHook = dropHook;
window.gameStartReel = startReel;

/* keyboard binding for E (drop) already in create; used as redundancy too */
window.addEventListener('keydown', (ev)=> { if (ev.key==='e' || ev.key==='E') dropHook(); });

/* ------------- SPAWN FISH & TERRAIN ------------- */
function spawnFish(scene){
  // pick depth and candidate species
  const y = WORLD.seaHeight + Phaser.Math.Between(100, WORLD.depth - 60);
  const depthM = y - WORLD.seaHeight;
  const candidates = FISH.filter(f => depthM >= f[1] && depthM <= f[2]);
  if (candidates.length === 0) return;
  const rb = state.upgrades ? state.upgrades.rarityBoost : 1;
  const weights = candidates.map(c => Math.pow(c[5], rb));
  const idx = weightedChoice(weights); const spec = candidates[idx];
  const x = Phaser.Math.Between(40, WORLD.width - 40);
  const tex = Math.random() < 0.45 ? 'fA' : Math.random() < 0.75 ? 'fB': 'fC';
  const f = scene.physics.add.sprite(x, y, tex);
  f.setData('name', spec[0]); f.setData('minD', spec[1]); f.setData('maxD', spec[2]);
  f.setData('price', Math.max(5, Math.floor(spec[3] * (1 + Math.random()*0.6)))); f.setData('wt', spec[4]);
  f.setData('seed', Math.random()*1000); f.setScale(1.0);
  fishGroup.add(f);
}
function weightedChoice(weights){ const sum = weights.reduce((a,b)=>a+b,0); let r = Math.random()*sum; for (let i=0;i<weights.length;i++){ r -= weights[i]; if (r<=0) return i; } return weights.length-1; }

function generateTerrain(scene){
  const t = WORLD.tile, cols = Math.ceil(WORLD.width / t), rows = Math.ceil(WORLD.depth / t);
  for (let cx=0; cx<cols; cx++){
    for (let cy=0; cy<rows; cy++){
      const wx = cx * t, wy = WORLD.seaHeight + cy * t;
      const n = caveNoise(wx*0.6, wy*0.6);
      const threshold = 0.58 - Math.min(0.18, (wy - WORLD.seaHeight)/7500 * 0.18);
      if (n > threshold && wy > WORLD.seaHeight + 140) {
        const rock = scene.add.image(wx + t/2, wy + t/2, 'rock').setScale(0.8);
        wallsGroup.add(rock); rock.setDepth(2);
      } else {
        // kelp layer
        if (wy - WORLD.seaHeight > 160 && wy - WORLD.seaHeight < 900 && Math.random() < 0.02) {
          const k = scene.add.image(wx + t/2, wy + t/2 + 8, 'kelp').setScale(1.0); k.setDepth(1);
        }
        // vents deep
        if (wy - WORLD.seaHeight > 1800 && Math.random() < 0.006) {
          const v = scene.add.image(wx + t/2, wy + t/2, 'vent').setScale(0.9); v.setDepth(1);
        }
      }
    }
  }
}

/* bubbles */
function spawnBubble(scene){
  const x = Phaser.Math.Between(20, WORLD.width - 20);
  const y = WORLD.seaHeight + Phaser.Math.Between(30, WORLD.depth - 200);
  const b = scene.add.sprite(x,y,'bubble').setScale(0.6);
  scene.tweens.add({ targets:b, y: y - Phaser.Math.Between(70,160), alpha:0, duration:2500, onComplete:()=>b.destroy() });
}

/* ---------------- UI PANELS (Shop only) ---------------- */
function updateUI(){
  cashSpan().textContent = format(state.cash || 0);
  caughtSpan().textContent = format(state.caught || 0);
  depthSpan().textContent = Math.max(0, Math.floor(hook.y - WORLD.seaHeight));
  durSpan().textContent = Math.max(0, Math.floor(state.rodDur || 0));
  save();
}

function renderUpgrades(){
  panelInner().innerHTML = '';
  if (!docked) { panelInner().innerHTML = '<div class="small">Dock at the shop to access upgrades and repairs.</div>'; return; }
  panelInner().innerHTML = '<h3>Shop — Upgrades & Repairs</h3><div class="small">Repair rod or buy upgrades. Repairs cost scale with missing durability.</div>';
  const repairCost = Math.max(20, Math.floor((100 - (state.rodDur || 100)) * 1.6));
  const rrow = document.createElement('div'); rrow.className='urow';
  rrow.innerHTML = `<div><b>Repair Rod</b> <span class="small">Durability: ${state.rodDur||0}%</span></div>
    <div><span class="price">$${repairCost}</span> <button class="ubtn" id="repairBtn">Repair</button></div>`;
  panelInner().appendChild(rrow);
  document.getElementById('repairBtn').onclick = ()=> {
    if (state.cash >= repairCost) { state.cash -= repairCost; state.rodDur = 100; hookBroken = false; save(); updateUI(); showMsg('Rod repaired.'); renderUpgrades(); }
    else showMsg('Not enough cash.');
  };

  Object.keys(UPG_META).forEach(k=>{
    const meta = UPG_META[k]; const lvl = (state.upgrades && state.upgrades[k]) || 1;
    const price = Math.floor(meta.price * Math.pow(meta.step, lvl-1));
    const div = document.createElement('div'); div.className='urow';
    div.innerHTML = `<div><b>${meta.label}</b> <span class="small">Lv ${lvl}</span></div>
      <div><span class="price">$${price}</span> <button class="ubtn upBtn" data-k="${k}">Buy</button></div>`;
    panelInner().appendChild(div);
  });

  panelInner().querySelectorAll('.upBtn').forEach(btn=>{
    btn.onclick = ()=> {
      const k = btn.dataset.k; const meta = UPG_META[k]; const lvl = (state.upgrades && state.upgrades[k]) || 1;
      const price = Math.floor(meta.price * Math.pow(meta.step, lvl-1));
      if (state.cash >= price) { state.cash -= price; state.upgrades[k] = lvl + 1; save(); showMsg(`${meta.label} upgraded.`); renderUpgrades(); updateUI(); } else showMsg('Not enough cash'); 
    };
  });
}

function renderIndex(){
  panelInner().innerHTML = '<h3>Fish Index</h3><div class="small">Discovered fish:</div>';
  if (!docked) { panelInner().innerHTML = '<div class="small">Dock at shop to view index.</div>'; return; }
  FISH.forEach(f=>{
    const name = f[0]; const discovered = !!state.fishdex[name];
    const el = document.createElement('div'); el.style.margin = '6px 0';
    el.innerHTML = discovered ? `✅ <b>${name}</b> — $${f[3]} • depth ${f[1]}-${f[2]}m • ${f[6]}` : `❔ <b>Unknown</b> — discover at ${f[1]}m+`;
    panelInner().appendChild(el);
  });
}

function renderSettings(){
  panelInner().innerHTML = '<h3>Settings</h3><div class="small">Controls: WASD to move hook, Space to reel, ← → to move boat. Drop Hook (E).</div>';
  if (!docked) { panelInner().innerHTML += '<div class="small">Dock to access save/reset options.</div>'; return; }
  const reset = document.createElement('div'); reset.className='urow';
  reset.innerHTML = `<div>Reset Save</div><div><button class='ubtn' id='resetSave'>Reset</button></div>`;
  panelInner().appendChild(reset);
  panelInner().querySelector('#resetSave').onclick = ()=> { if (confirm('Reset all progress?')) { localStorage.removeItem('pixeldeep_full'); location.reload(); } };
}

/* ---------------- Save & Start ---------------- */
updateUI();
save();

/* ---------------- Expose for debugging in console ---------------- */
window._pixeldeep = { state, FISH };

/* ---- End of file ---- */
</script>
</body>
</html>
