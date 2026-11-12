<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#0a0410">
<title>Vampiro vs Zombies - Boss Edition</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box;-webkit-tap-highlight-color:transparent}
  html,body{
    width:100%;height:100%;overflow:hidden;background:#05030a;
    touch-action:none;overscroll-behavior:none;
    -webkit-user-select:none;user-select:none;
    font-family:system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;
  }
  #wrap{
    --s:1;
    position:fixed;top:0;left:0;overflow:hidden;
    transform-origin:top left;
    background:#05030a;
  }
  canvas{display:block;position:absolute;top:0;left:0;}

  #score{
    position:absolute;top:calc(10px * var(--s));left:50%;
    transform:translateX(-50%);
    font-size:calc(30px * var(--s));font-weight:800;color:#fff;
    text-shadow:0 0 calc(14px * var(--s)) #a855f7, 0 calc(3px * var(--s)) calc(6px * var(--s)) #000;
    letter-spacing:calc(1px * var(--s));
    pointer-events:none;
  }
  #lives{
    position:absolute;top:calc(12px * var(--s));left:calc(16px * var(--s));
    font-size:calc(20px * var(--s));
    pointer-events:none;
    filter:drop-shadow(0 0 calc(8px * var(--s)) rgba(255,0,0,.8));
  }
  #styleTag{
    position:absolute;top:calc(14px * var(--s));right:calc(16px * var(--s));
    font-size:calc(11px * var(--s));font-weight:700;
    padding:calc(4px * var(--s)) calc(9px * var(--s));
    border-radius:calc(8px * var(--s));
    background:rgba(46,16,74,.7);
    border:calc(1px * var(--s)) solid rgba(168,85,247,.6);
    color:#e9d5ff;
    pointer-events:none;
    letter-spacing:calc(0.5px * var(--s));
    text-transform:uppercase;
  }
  #styleTag.pretty{color:#fbbf24;border-color:rgba(251,191,36,.7);}

  #pad{
    position:absolute;
    left:calc(16px * var(--s));
    bottom:calc(16px * var(--s));
    display:grid;
    grid-template-columns:repeat(3,auto);
    grid-template-rows:repeat(2,auto);
    gap:calc(9px * var(--s));
  }
  .btn{
    width:calc(70px * var(--s));
    height:calc(70px * var(--s));
    border-radius:calc(15px * var(--s));
    border:calc(2px * var(--s)) solid rgba(168,85,247,.55);
    background:rgba(46,16,74,.62);
    color:#e9d5ff;
    font-size:calc(24px * var(--s));
    display:flex;align-items:center;justify-content:center;
    touch-action:none;cursor:pointer;
    -webkit-backdrop-filter:blur(6px);backdrop-filter:blur(6px);
    transition:background .08s, transform .08s;
    box-shadow:0 0 calc(12px * var(--s)) rgba(120,40,200,.35);
  }
  .btn.active{background:rgba(168,85,247,.9);transform:scale(.93);}
  #btnUp   {grid-column:2;grid-row:1;}
  #btnLeft {grid-column:1;grid-row:2;}
  #btnDown {grid-column:2;grid-row:2;}
  #btnRight{grid-column:3;grid-row:2;}

  #btnKnife{
    position:absolute;
    right:calc(22px * var(--s));
    bottom:calc(26px * var(--s));
    width:calc(90px * var(--s));
    height:calc(90px * var(--s));
    border-radius:50%;
    border:calc(3px * var(--s)) solid rgba(248,113,113,.75);
    background:radial-gradient(circle at 35% 30%, rgba(248,113,113,.9), rgba(120,10,25,.9));
    font-size:calc(40px * var(--s));
    line-height:1;
    display:flex;align-items:center;justify-content:center;
    touch-action:none;cursor:pointer;color:#fff;
    box-shadow:0 0 calc(24px * var(--s)) rgba(239,68,68,.55);
    transition:transform .08s;
  }
  #btnKnife.active{transform:scale(.9);}

  #overlay{
    position:absolute;top:0;left:0;right:0;bottom:0;
    background:rgba(5,3,10,.88);
    display:none;flex-direction:column;align-items:center;justify-content:center;
    gap:calc(16px * var(--s));color:#fff;
    -webkit-backdrop-filter:blur(5px);backdrop-filter:blur(5px);
    z-index:10;
  }
  #overlay.show{display:flex;}
  #overlayTitle{
    font-size:calc(46px * var(--s));color:#ef4444;letter-spacing:calc(2px * var(--s));
    text-shadow:0 0 calc(22px * var(--s)) currentColor;
    text-align:center;padding:0 calc(20px * var(--s));
  }
  #overlay p{font-size:calc(20px * var(--s));opacity:.9;}
  #btnRestart{
    padding:calc(13px * var(--s)) calc(34px * var(--s));
    font-size:calc(19px * var(--s));font-weight:700;
    border-radius:calc(14px * var(--s));border:none;color:#fff;cursor:pointer;
    background:linear-gradient(180deg,#a855f7,#7e22ce);
    box-shadow:0 0 calc(22px * var(--s)) rgba(168,85,247,.65);
  }
</style>
</head>
<body>
<div id="wrap">
  <canvas id="game"></canvas>

  <div id="score">0 / 100</div>
  <div id="lives">❤️❤️❤️</div>
  <div id="styleTag">Daga simple</div>

  <div id="pad">
    <button class="btn" id="btnUp"    aria-label="Saltar">▲</button>
    <button class="btn" id="btnLeft"  aria-label="Izquierda">◀</button>
    <button class="btn" id="btnDown"  aria-label="Agacharse">▼</button>
    <button class="btn" id="btnRight" aria-label="Derecha">▶</button>
  </div>

  <button id="btnKnife" aria-label="Lanzar cuchillo">🔪</button>

  <div id="overlay">
    <h1 id="overlayTitle">GAME OVER</h1>
    <p>Puntuación: <span id="finalScore">0</span></p>
    <button id="btnRestart">REINTENTAR</button>
  </div>
</div>

<script>
(function(){
'use strict';

const VH = 500;
const GROUND = VH * 0.86;
const WORLD_W = 3200;
const BOSS_SCORE = 100;

let VW = 900, scale = 1, GW = 0, GH = 0;
let camX = 0;

const wrap    = document.getElementById('wrap');
const canvas  = document.getElementById('game');
const ctx     = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const livesEl = document.getElementById('lives');
const styleTag= document.getElementById('styleTag');
const overlay = document.getElementById('overlay');
const overlayTitle = document.getElementById('overlayTitle');
const finalScoreEl = document.getElementById('finalScore');

let player, knives, zombies, particles, pickups, boss;
let score, lives, gameOver, won, invuln, throwCd;
let spawnTimer, pickupTimer, shake, bossDefeated, winTimer;

const keys = { left:false, right:false, up:false, down:false };

const stars = [];
for (let i = 0; i < 100; i++){
  stars.push({
    x: Math.random(),
    y: Math.random() * 0.65,
    r: Math.random() * 1.6 + 0.4,
    a: Math.random() * 0.6 + 0.25,
    par: 0.1 + Math.random() * 0.2
  });
}

const tombstones = [];
for (let x = 120; x < WORLD_W; x += 220){
  tombstones.push({ x: x + Math.random() * 140, s: 0.7 + Math.random() * 0.6 });
}

function layout(){
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const portrait = vh > vw;

  if (portrait){
    GW = vh; GH = vw;
    wrap.style.width  = GW + 'px';
    wrap.style.height = GH + 'px';
    wrap.style.transform = 'rotate(-90deg) translateX(-100%)';
  } else {
    GW = vw; GH = vh;
    wrap.style.width  = GW + 'px';
    wrap.style.height = GH + 'px';
    wrap.style.transform = 'none';
  }

  wrap.style.setProperty('--s', String(GH / 500));

  const dpr = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width  = Math.round(GW * dpr);
  canvas.height = Math.round(GH * dpr);
  canvas.style.width  = GW + 'px';
  canvas.style.height = GH + 'px';

  scale = GH / VH;
  VW = GW / scale;

  ctx.setTransform(dpr * scale, 0, 0, dpr * scale, 0, 0);
}

function rr(x, y, w, h, r){
  r = Math.min(r, w/2, h/2);
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.arcTo(x + w, y,     x + w, y + h, r);
  ctx.arcTo(x + w, y + h, x,     y + h, r);
  ctx.arcTo(x,     y + h, x,     y,     r);
  ctx.arcTo(x,     y,     x + w, y,     r);
  ctx.closePath();
}

function aabb(a, b){
  return a.x < b.x + b.w && a.x + a.w > b.x &&
         a.y < b.y + b.h && a.y + a.h > b.y;
}

function updateHUD(){
  if (score < BOSS_SCORE) scoreEl.textContent = score + ' / ' + BOSS_SCORE;
  else if (boss)          scoreEl.textContent = '¡JEFE FINAL!';
  else                    scoreEl.textContent = '¡VICTORIA!';
  livesEl.textContent = '❤️'.repeat(Math.max(0, lives)) || '💀';
  if (player){
    if (player.currentStyle === 'pretty'){
      styleTag.textContent = 'Daga élfica';
      styleTag.classList.add('pretty');
    } else {
      styleTag.textContent = 'Daga simple';
      styleTag.classList.remove('pretty');
    }
  }
}

function reset(){
  player = {
    x: WORLD_W * 0.5, y: GROUND,
    w: 46, h: 96,
    vy: 0, onGround: true, crouching: false,
    facing: 1,
    currentStyle: 'simple'
  };
  knives = [];
  zombies = [];
  particles = [];
  pickups = [];
  boss = null;
  score = 0;
  lives = 3;
  gameOver = false;
  won = false;
  bossDefeated = false;
  invuln = 0;
  throwCd = 0;
  spawnTimer = 1.2;
  pickupTimer = 3;
  shake = 0;
  winTimer = 0;
  camX = Math.max(0, Math.min(WORLD_W - VW, player.x - VW * 0.35));
  updateHUD();
  overlay.classList.remove('show');
}

function spawnZombie(){
  const w = 44, h = 92;
  const speed = 95 + Math.random() * 55 + Math.min(score * 1.6, 140);
  const roll = Math.random();

  if (roll < 0.20){
    zombies.push({
      x: camX + 80 + Math.random() * (VW - 160),
      y: -h - Math.random() * 60,
      w, h, speed, vy: 0, falling: true,
      seed: Math.random() * Math.PI * 2,
      walk: Math.random() * 10
    });
  } else if (roll < 0.60){
    zombies.push({
      x: camX - 80,
      y: GROUND,
      w, h, speed, vy: 0, falling: false,
      seed: Math.random() * Math.PI * 2,
      walk: Math.random() * 10
    });
  } else {
    zombies.push({
      x: camX + VW + 80,
      y: GROUND,
      w, h, speed, vy: 0, falling: false,
      seed: Math.random() * Math.PI * 2,
      walk: Math.random() * 10
    });
  }
}

function spawnPickup(){
  pickups.push({
    x: camX + 80 + Math.random() * (VW - 160),
    y: -60, vy: 0,
    rot: Math.random() * Math.PI * 2,
    style: Math.random() < 0.5 ? 'simple' : 'pretty',
    landed: false
  });
}

function spawnBoss(){
  const spawnRight = player.x + player.w/2 < WORLD_W * 0.5;
  boss = {
    x: spawnRight ? (camX + VW + 200) : (camX - 200 - 140),
    y: GROUND,
    w: 140, h: 260,
    speed: 70,
    hp: 15, maxHp: 15,
    hitFlash: 0,
    walk: 0,
    seed: Math.random() * 6
  };
  shake = 40;
  for (let i = 0; i < 25; i++){
    burst(camX + VW/2 + (Math.random()-0.5) * 300,
          VH * 0.4 + (Math.random()-0.5) * 200,
          '#a855f7');
  }
}

function burst(x, y, color){
  for (let i = 0; i < 12; i++){
    const a  = Math.random() * Math.PI * 2;
    const sp = 60 + Math.random() * 240;
    const life = 0.4 + Math.random() * 0.3;
    particles.push({
      x, y,
      vx: Math.cos(a) * sp,
      vy: Math.sin(a) * sp - 120,
      life, maxLife: life,
      r: 2 + Math.random() * 4,
      color: color || (Math.random() < 0.5 ? '#7ab84a' : '#3f6b22')
    });
  }
}

function endGame(){
  gameOver = true;
  won = false;
  overlayTitle.textContent = 'GAME OVER';
  overlayTitle.style.color = '#ef4444';
  finalScoreEl.textContent = score;
  overlay.classList.add('show');
}

function winGame(){
  gameOver = true;
  won = true;
  overlayTitle.textContent = '¡GANASTE!';
  overlayTitle.style.color = '#a855f7';
  finalScoreEl.textContent = score;
  overlay.classList.add('show');
  updateHUD();
}

function throwKnife(){
  if (gameOver || throwCd > 0) return;
  throwCd = 0.20;
  const h = player.crouching ? player.h * 0.62 : player.h;
  const dir = player.facing;
  knives.push({
    x: player.x + (dir > 0 ? player.w * 0.85 : player.w * 0.15),
    y: player.y - h * 0.62,
    vx: 950 * dir,
    rot: 0
  });
}

function update(dt){
  for (let i = particles.length - 1; i >= 0; i--){
    const pt = particles[i];
    pt.x  += pt.vx * dt;
    pt.y  += pt.vy * dt;
    pt.vy += 900 * dt;
    pt.life -= dt;
    if (pt.life <= 0) particles.splice(i, 1);
  }

  if (winTimer > 0){
    winTimer -= dt;
    if (winTimer <= 0) winGame();
  }

  if (gameOver) return;

  const p = player;

  const baseSpeed = 380;
  const speed = (p.crouching && p.onGround) ? baseSpeed * 0.45 : baseSpeed;

  let mx = 0;
  if (keys.left)  mx -= 1;
  if (keys.right) mx += 1;
  if (mx !== 0) p.facing = mx;

  p.x += mx * speed * dt;
  p.x = Math.max(0, Math.min(WORLD_W - p.w, p.x));

  p.crouching = keys.down && p.onGround;

  if (keys.up && p.onGround && !p.crouching){
    p.vy = -600;
    p.onGround = false;
  }

  p.vy += 1800 * dt;
  p.y  += p.vy * dt;
  if (p.y >= GROUND){
    p.y = GROUND;
    p.vy = 0;
    p.onGround = true;
  }

  const targetCam = p.x - VW * 0.35;
  const maxCam = Math.max(0, WORLD_W - VW);
  camX = Math.max(0, Math.min(maxCam, targetCam));

  throwCd = Math.max(0, throwCd - dt);
  for (let i = knives.length - 1; i >= 0; i--){
    const k = knives[i];
    k.x  += k.vx * dt;
    k.rot += 22 * dt * (k.vx > 0 ? 1 : -1);
    if (k.x < camX - 120 || k.x > camX + VW + 120) knives.splice(i, 1);
  }

  if (!boss && !bossDefeated && score < BOSS_SCORE){
    spawnTimer -= dt;
    if (spawnTimer <= 0){
      spawnZombie();
      spawnTimer = Math.max(0.45, 1.5 - score * 0.015) * (0.65 + Math.random() * 0.7);
    }
  }

  pickupTimer -= dt;
  if (pickupTimer <= 0){
    spawnPickup();
    pickupTimer = 4 + Math.random() * 4;
  }

  for (let i = zombies.length - 1; i >= 0; i--){
    const z = zombies[i];

    if (z.falling){
      z.vy += 1600 * dt;
      z.y  += z.vy * dt;
      if (z.y >= GROUND){
        z.y = GROUND;
        z.vy = 0;
        z.falling = false;
        burst(z.x + z.w/2, GROUND, '#7ab84a');
      }
    } else {
      const dir = (p.x + p.w/2 > z.x + z.w/2) ? 1 : -1;
      z.x += z.speed * dir * dt;
      z.walk += dt * 8;
    }

    if (z.x < camX - 600 || z.x > camX + VW + 600) zombies.splice(i, 1);
  }

  for (let i = pickups.length - 1; i >= 0; i--){
    const pk = pickups[i];
    if (!pk.landed){
      pk.vy += 1000 * dt;
      pk.y  += pk.vy * dt;
      pk.rot += 3.5 * dt;
      if (pk.y >= GROUND - 10){
        pk.y = GROUND - 10;
        pk.vy = 0;
        pk.landed = true;
        pk.rot = Math.PI / 4;
      }
    }
    if (pk.x < camX - 500 || pk.x > camX + VW + 500) pickups.splice(i, 1);
  }

  const pheight = p.crouching ? p.h * 0.62 : p.h;
  const pbox = { x: p.x, y: p.y - pheight, w: p.w, h: pheight };

  for (let i = pickups.length - 1; i >= 0; i--){
    const pk = pickups[i];
    if (!pk.landed) continue;
    const pkBox = { x: pk.x - 22, y: pk.y - 22, w: 44, h: 44 };
    if (aabb(pbox, pkBox)){
      const color = pk.style === 'pretty' ? '#fbbf24' : '#94a3b8';
      burst(pk.x, pk.y, color);
      p.currentStyle = pk.style;
      pickups.splice(i, 1);
      updateHUD();
    }
  }

  for (let i = knives.length - 1; i >= 0; i--){
    const k = knives[i];
    const kb = { x: k.x - 22, y: k.y - 10, w: 44, h: 20 };
    let hit = false;
    for (let j = zombies.length - 1; j >= 0; j--){
      const z = zombies[j];
      const zb = { x: z.x, y: z.y - z.h, w: z.w, h: z.h };
      if (aabb(kb, zb)){
        zombies.splice(j, 1);
        hit = true;
        score++;
        updateHUD();
        burst(z.x + z.w * 0.5, z.y - z.h * 0.6);
        break;
      }
    }
    if (hit) knives.splice(i, 1);
  }

  if (boss){
    for (let i = knives.length - 1; i >= 0; i--){
      const k = knives[i];
      const kb = { x: k.x - 22, y: k.y - 12, w: 44, h: 24 };
      const bb = { x: boss.x, y: boss.y - boss.h, w: boss.w, h: boss.h };
      if (aabb(kb, bb)){
        knives.splice(i, 1);
        boss.hp--;
        boss.hitFlash = 0.12;
        burst(k.x, k.y, '#fbbf24');
        if (boss.hp <= 0){
          for (let n = 0; n < 50; n++){
            burst(boss.x + boss.w/2 + (Math.random()-0.5)*boss.w,
                  boss.y - boss.h*0.6 + (Math.random()-0.5)*boss.h*0.7,
                  Math.random() < 0.5 ? '#fbbf24' : '#a855f7');
          }
          shake = 50;
          boss = null;
          bossDefeated = true;
          winTimer = 1.0;
          updateHUD();
        }
        break;
      }
    }
  }

  if (boss){
    boss.hitFlash = Math.max(0, boss.hitFlash - dt);
    const dir = (p.x + p.w/2 > boss.x + boss.w/2) ? 1 : -1;
    boss.x += boss.speed * dir * dt;
    boss.walk += dt * 5;
  }

  if (!boss && !bossDefeated && score >= BOSS_SCORE){
    spawnBoss();
    updateHUD();
  }

  invuln = Math.max(0, invuln - dt);
  const pheight2 = p.crouching ? p.h * 0.62 : p.h;
  const pbox2 = { x: p.x, y: p.y - pheight2, w: p.w, h: pheight2 };

  for (let j = zombies.length - 1; j >= 0; j--){
    const z = zombies[j];
    const zb = { x: z.x, y: z.y - z.h, w: z.w, h: z.h };
    if (aabb(pbox2, zb)){
      zombies.splice(j, 1);
      if (invuln <= 0){
        lives--;
        invuln = 1.6;
        shake = 14;
        updateHUD();
        if (lives <= 0) endGame();
      }
    }
  }

  if (boss){
    const bb = { x: boss.x, y: boss.y - boss.h, w: boss.w, h: boss.h };
    if (aabb(pbox2, bb)){
      if (invuln <= 0){
        lives -= 2;
        invuln = 2.0;
        shake = 25;
        updateHUD();
        if (lives <= 0) endGame();
      }
    }
  }

  shake = Math.max(0, shake - dt * 45);
}

function drawTomb(x, y, s){
  ctx.fillStyle = '#1e1630';
  ctx.beginPath();
  ctx.moveTo(x - 16*s, y);
  ctx.lineTo(x - 16*s, y - 40*s);
  ctx.arc(x, y - 40*s, 16*s, Math.PI, 0);
  ctx.lineTo(x + 16*s, y);
  ctx.closePath();
  ctx.fill();
  ctx.fillStyle = 'rgba(0,0,0,.35)';
  ctx.fillRect(x - 16*s, y - 7*s, 32*s, 7*s);
}

function drawBackground(){
  const g = ctx.createLinearGradient(0, 0, 0, VH);
  g.addColorStop(0,   '#0c0616');
  g.addColorStop(0.5, '#1a0b2e');
  g.addColorStop(1,   '#08040e');
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, VW, VH);

  ctx.fillStyle = '#ffffff';
  for (const s of stars){
    ctx.globalAlpha = s.a;
    let sx = (s.x * VW - camX * s.par) % VW;
    if (sx < 0) sx += VW;
    ctx.beginPath();
    ctx.arc(sx, s.y * VH, s.r, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;

  const moonX = VW * 0.80 - camX * 0.08;
  const my = VH * 0.17;
  const glow = ctx.createRadialGradient(moonX, my, 12, moonX, my, 130);
  glow.addColorStop(0, 'rgba(190,160,255,.35)');
  glow.addColorStop(1, 'rgba(190,160,255,0)');
  ctx.fillStyle = glow;
  ctx.beginPath(); ctx.arc(moonX, my, 130, 0, Math.PI * 2); ctx.fill();

  ctx.fillStyle = '#ded4ff';
  ctx.beginPath(); ctx.arc(moonX, my, 40, 0, Math.PI * 2); ctx.fill();
  ctx.fillStyle = 'rgba(160,150,210,.45)';
  ctx.beginPath(); ctx.arc(moonX - 13, my - 9,  9, 0, Math.PI * 2); ctx.fill();
  ctx.beginPath(); ctx.arc(moonX + 11, my + 9,  6, 0, Math.PI * 2); ctx.fill();
  ctx.beginPath(); ctx.arc(moonX + 2,  my - 19, 4, 0, Math.PI * 2); ctx.fill();

  const hillOffset = camX * 0.4;
  ctx.fillStyle = '#150c24';
  ctx.beginPath();
  ctx.moveTo(0, GROUND);
  for (let sx = 0; sx <= VW; sx += 14){
    const wx = sx + hillOffset;
    const y = GROUND - 70 - Math.sin(wx * 0.005) * 26 - Math.sin(wx * 0.014 + 1.7) * 16;
    ctx.lineTo(sx, y);
  }
  ctx.lineTo(VW, GROUND);
  ctx.closePath();
  ctx.fill();

  for (const t of tombstones){
    const sx = t.x - camX;
    if (sx < -60 || sx > VW + 60) continue;
    drawTomb(sx, GROUND, t.s);
  }

  ctx.fillStyle = '#0b0712';
  ctx.fillRect(0, GROUND, VW, VH - GROUND);
  const gg = ctx.createLinearGradient(0, GROUND, 0, GROUND + 24);
  gg.addColorStop(0, 'rgba(130,80,200,.38)');
  gg.addColorStop(1, 'rgba(130,80,200,0)');
  ctx.fillStyle = gg;
  ctx.fillRect(0, GROUND, VW, 24);
}

function drawKnifeSimple(x, y, rot, scale){
  scale = scale || 1;
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(rot);
  ctx.scale(scale, scale);

  ctx.globalAlpha = 0.18;
  ctx.fillStyle = '#94a3b8';
  ctx.fillRect(-52, -2, 36, 4);
  ctx.globalAlpha = 1;

  ctx.fillStyle = '#cbd5e1';
  ctx.beginPath();
  ctx.moveTo(-20, -3);
  ctx.lineTo(16, -2);
  ctx.lineTo(22, 0);
  ctx.lineTo(16, 2);
  ctx.lineTo(-20, 3);
  ctx.closePath();
  ctx.fill();

  ctx.strokeStyle = '#f1f5f9';
  ctx.lineWidth = 1;
  ctx.beginPath();
  ctx.moveTo(-15, -1);
  ctx.lineTo(14, 0);
  ctx.stroke();

  ctx.fillStyle = '#6b3410';
  rr(-28, -3.5, 9, 7, 2);
  ctx.fill();

  ctx.restore();
}

function drawKnifePretty(x, y, rot, scale){
  scale = scale || 1;
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(rot);
  ctx.scale(scale, scale);

  const trail = ctx.createLinearGradient(-60, 0, -20, 0);
  trail.addColorStop(0, 'rgba(168,85,247,0)');
  trail.addColorStop(1, 'rgba(168,85,247,0.65)');
  ctx.fillStyle = trail;
  ctx.fillRect(-60, -3.5, 40, 7);

  ctx.shadowColor = '#a855f7';
  ctx.shadowBlur = 14;

  ctx.fillStyle = '#e2e8f0';
  ctx.beginPath();
  ctx.moveTo(-22, -3.5);
  ctx.quadraticCurveTo(-5, -5, 10, -3);
  ctx.lineTo(20, -1);
  ctx.lineTo(24, 0);
  ctx.lineTo(20, 1);
  ctx.lineTo(10, 3);
  ctx.quadraticCurveTo(-5, 5, -22, 3.5);
  ctx.closePath();
  ctx.fill();
  ctx.shadowBlur = 0;

  ctx.strokeStyle = 'rgba(168,85,247,0.85)';
  ctx.lineWidth = 1;
  ctx.beginPath();
  ctx.moveTo(-18, 0);
  ctx.lineTo(14, 0);
  ctx.stroke();

  ctx.fillStyle = '#fbbf24';
  rr(-26, -5.5, 4.5, 11, 1.5);
  ctx.fill();

  ctx.fillStyle = '#7c2d12';
  rr(-36, -3.5, 11, 7, 3);
  ctx.fill();

  ctx.fillStyle = '#dc2626';
  ctx.beginPath();
  ctx.arc(-30, 0, 2.2, 0, Math.PI * 2);
  ctx.fill();
  ctx.fillStyle = '#fca5a5';
  ctx.beginPath();
  ctx.arc(-30.6, -0.6, 0.9, 0, Math.PI * 2);
  ctx.fill();

  ctx.restore();
}

function drawKnifeByStyle(x, y, rot, style, scale){
  if (style === 'pretty') drawKnifePretty(x, y, rot, scale);
  else                    drawKnifeSimple(x, y, rot, scale);
}

function drawPlayer(){
  const p = player;
  const h = p.crouching ? p.h * 0.62 : p.h;
  const w = p.w;
  const cx = p.x + w * 0.5;
  const bottom = p.y;
  const top = bottom - h;
  const headR = w * 0.40;
  const headCY = top + headR * 1.02;

  ctx.save();
  if (p.facing < 0){
    ctx.translate(cx, 0);
    ctx.scale(-1, 1);
    ctx.translate(-cx, 0);
  }

  ctx.fillStyle = '#2b0c42';
  ctx.beginPath();
  ctx.moveTo(cx, headCY + headR * 0.3);
  ctx.quadraticCurveTo(cx - w * 1.15, top + h * 0.55, cx - w * 0.85, bottom);
  ctx.lineTo(cx + w * 0.5, bottom);
  ctx.quadraticCurveTo(cx + w * 0.85, top + h * 0.55, cx, headCY + headR * 0.3);
  ctx.closePath();
  ctx.fill();

  const legTop = bottom - h * 0.22;
  ctx.fillStyle = '#180a22';
  ctx.fillRect(cx - w * 0.30, legTop, w * 0.24, h * 0.22);
  ctx.fillRect(cx + w * 0.06, legTop, w * 0.24, h * 0.22);

  const bodyTop = headCY + headR * 0.55;
  const bodyH = legTop - bodyTop;
  ctx.fillStyle = '#43155e';
  rr(cx - w * 0.36, bodyTop, w * 0.72, bodyH + 4, w * 0.16);
  ctx.fill();

  ctx.fillStyle = '#5c1a80';
  rr(cx + w * 0.02, bodyTop, w * 0.30, bodyH, w * 0.10);
  ctx.fill();

  ctx.fillStyle = '#8b1a1a';
  ctx.beginPath();
  ctx.moveTo(cx - w * 0.34, bodyTop + 2);
  ctx.lineTo(cx, bodyTop + bodyH * 0.45);
  ctx.lineTo(cx + w * 0.34, bodyTop + 2);
  ctx.closePath();
  ctx.fill();

  ctx.fillStyle = '#efe2f0';
  ctx.beginPath(); ctx.arc(cx, headCY, headR, 0, Math.PI * 2); ctx.fill();

  ctx.fillStyle = '#140819';
  ctx.beginPath();
  ctx.arc(cx, headCY, headR, Math.PI * 1.02, Math.PI * 2.02);
  ctx.fill();
  ctx.beginPath();
  ctx.moveTo(cx - headR * 0.98, headCY - headR * 0.10);
  ctx.lineTo(cx,                 headCY + headR * 0.45);
  ctx.lineTo(cx + headR * 0.98, headCY - headR * 0.10);
  ctx.closePath();
  ctx.fill();

  ctx.fillStyle = '#ff2b2b';
  ctx.beginPath();
  ctx.arc(cx + headR * 0.18, headCY + headR * 0.05, headR * 0.15, 0, Math.PI * 2);
  ctx.arc(cx + headR * 0.62, headCY + headR * 0.05, headR * 0.15, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = '#ffffff';
  ctx.beginPath();
  ctx.moveTo(cx + headR * 0.25, headCY + headR * 0.55);
  ctx.lineTo(cx + headR * 0.33, headCY + headR * 0.88);
  ctx.lineTo(cx + headR * 0.41, headCY + headR * 0.55);
  ctx.closePath();
  ctx.fill();

  ctx.restore();
}

function drawZombie(z){
  const w = z.w, h = z.h;
  const cx = z.x + w * 0.5;
  const bottom = z.y;
  const top = bottom - h;
  const headR = w * 0.38;
  const headCY = top + headR * 1.0;
  const sw = Math.sin(z.walk + z.seed) * 0.25;

  const playerIsRight = (player.x + player.w/2 > z.x + z.w/2);

  ctx.save();
  if (playerIsRight){
    ctx.translate(cx, 0);
    ctx.scale(-1, 1);
    ctx.translate(-cx, 0);
  }

  ctx.fillStyle = '#243d16';
  ctx.fillRect(cx - w * 0.32, bottom - h * 0.24, w * 0.24, h * 0.24);
  ctx.fillRect(cx + w * 0.08, bottom - h * 0.24, w * 0.24, h * 0.24);

  ctx.fillStyle = '#5d9438';
  ctx.save();
  ctx.translate(cx - w * 0.05, headCY + headR * 1.0);
  ctx.rotate(-0.18 + sw * 0.25);
  rr(-w * 1.00, -w * 0.115, w * 1.00, w * 0.23, w * 0.115);
  ctx.fill();
  ctx.restore();

  ctx.save();
  ctx.translate(cx - w * 0.05, headCY + headR * 1.7);
  ctx.rotate(-0.05 - sw * 0.25);
  rr(-w * 0.95, -w * 0.115, w * 0.95, w * 0.23, w * 0.115);
  ctx.fill();
  ctx.restore();

  const bodyTop = headCY + headR * 0.55;
  const bodyH = (bottom - h * 0.24) - bodyTop;
  ctx.fillStyle = '#4d7d2b';
  rr(cx - w * 0.34, bodyTop, w * 0.68, bodyH + 2, w * 0.14);
  ctx.fill();

  ctx.fillStyle = '#2f5219';
  rr(cx - w * 0.34, bodyTop + bodyH * 0.45, w * 0.68, bodyH * 0.35, w * 0.08);
  ctx.fill();

  ctx.fillStyle = '#79b94a';
  ctx.beginPath(); ctx.arc(cx, headCY, headR, 0, Math.PI * 2); ctx.fill();

  ctx.fillStyle = 'rgba(60,100,30,.9)';
  ctx.beginPath();
  ctx.arc(cx + headR * 0.2, headCY - headR * 0.55, headR * 0.35, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = '#1c1a08';
  ctx.beginPath();
  ctx.arc(cx - headR * 0.50, headCY - headR * 0.10, headR * 0.20, 0, Math.PI * 2);
  ctx.arc(cx - headR * 0.05, headCY - headR * 0.10, headR * 0.20, 0, Math.PI * 2);
  ctx.fill();
  ctx.fillStyle = '#ffe14d';
  ctx.beginPath();
  ctx.arc(cx - headR * 0.50, headCY - headR * 0.10, headR * 0.08, 0, Math.PI * 2);
  ctx.arc(cx - headR * 0.05, headCY - headR * 0.10, headR * 0.08, 0, Math.PI * 2);
  ctx.fill();

  ctx.strokeStyle = '#1c1a08';
  ctx.lineWidth = 2.5;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(cx - headR * 0.62, headCY + headR * 0.50);
  ctx.lineTo(cx - headR * 0.05, headCY + headR * 0.50);
  ctx.stroke();

  ctx.restore();
}

function drawBoss(b){
  const w = b.w, h = b.h;
  const cx = b.x + w * 0.5;
  const bottom = b.y;
  const top = bottom - h;
  const headR = w * 0.36;
  const headCY = top + headR * 1.05;
  const sw = Math.sin(b.walk + b.seed) * 0.22;
  const flash = b.hitFlash > 0;

  const playerIsRight = (player.x + player.w/2 > b.x + b.w/2);

  ctx.save();
  if (playerIsRight){
    ctx.translate(cx, 0);
    ctx.scale(-1, 1);
    ctx.translate(-cx, 0);
  }

  ctx.save();
  const aura = ctx.createRadialGradient(cx, bottom - h * 0.5, 20, cx, bottom - h * 0.5, w * 1.2);
  aura.addColorStop(0, 'rgba(220,38,38,0.28)');
  aura.addColorStop(1, 'rgba(220,38,38,0)');
  ctx.fillStyle = aura;
  ctx.beginPath();
  ctx.arc(cx, bottom - h * 0.5, w * 1.2, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();

  ctx.fillStyle = flash ? '#ff6b6b' : '#1a2a10';
  ctx.fillRect(cx - w * 0.34, bottom - h * 0.26, w * 0.30, h * 0.26);
  ctx.fillRect(cx + w * 0.04, bottom - h * 0.26, w * 0.30, h * 0.26);

  ctx.fillStyle = flash ? '#ff8a8a' : '#3d6020';
  ctx.save();
  ctx.translate(cx - w * 0.05, headCY + headR * 1.1);
  ctx.rotate(-0.20 + sw * 0.22);
  rr(-w * 1.08, -w * 0.14, w * 1.08, w * 0.28, w * 0.14);
  ctx.fill();
  ctx.restore();

  ctx.save();
  ctx.translate(cx - w * 0.05, headCY + headR * 2.0);
  ctx.rotate(-0.06 - sw * 0.22);
  rr(-w * 1.02, -w * 0.14, w * 1.02, w * 0.28, w * 0.14);
  ctx.fill();
  ctx.restore();

  const bodyTop = headCY + headR * 0.55;
  const bodyH = (bottom - h * 0.26) - bodyTop;
  ctx.fillStyle = flash ? '#ff7070' : '#3a5c1f';
  rr(cx - w * 0.40, bodyTop, w * 0.80, bodyH + 2, w * 0.14);
  ctx.fill();

  ctx.strokeStyle = 'rgba(0,0,0,0.4)';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(cx - w * 0.10, bodyTop + bodyH * 0.15);
  ctx.lineTo(cx + w * 0.05, bodyTop + bodyH * 0.5);
  ctx.lineTo(cx - w * 0.08, bodyTop + bodyH * 0.85);
  ctx.stroke();

  ctx.fillStyle = flash ? '#ff9090' : '#2f5219';
  ctx.beginPath();
  ctx.arc(cx - w * 0.40, bodyTop + 8, w * 0.15, 0, Math.PI * 2);
  ctx.arc(cx + w * 0.40, bodyTop + 8, w * 0.15, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = flash ? '#ff9d9d' : '#5f8f38';
  ctx.beginPath(); ctx.arc(cx, headCY, headR, 0, Math.PI * 2); ctx.fill();

  ctx.shadowColor = '#ff0000';
  ctx.shadowBlur = 16;
  ctx.fillStyle = '#ff2020';
  ctx.beginPath();
  ctx.arc(cx - headR * 0.42, headCY - headR * 0.10, headR * 0.22, 0, Math.PI * 2);
  ctx.arc(cx + headR * 0.42, headCY - headR * 0.10, headR * 0.22, 0, Math.PI * 2);
  ctx.fill();
  ctx.shadowBlur = 0;

  ctx.fillStyle = '#220000';
  ctx.beginPath();
  ctx.arc(cx - headR * 0.42, headCY - headR * 0.10, headR * 0.08, 0, Math.PI * 2);
  ctx.arc(cx + headR * 0.42, headCY - headR * 0.10, headR * 0.08, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = '#1a0505';
  rr(cx - headR * 0.6, headCY + headR * 0.35, headR * 1.2, headR * 0.5, headR * 0.15);
  ctx.fill();
  ctx.fillStyle = '#ffffff';
  ctx.beginPath();
  ctx.moveTo(cx - headR * 0.45, headCY + headR * 0.4);
  ctx.lineTo(cx - headR * 0.35, headCY + headR * 0.7);
  ctx.lineTo(cx - headR * 0.25, headCY + headR * 0.4);
  ctx.closePath();
  ctx.fill();
  ctx.beginPath();
  ctx.moveTo(cx + headR * 0.25, headCY + headR * 0.4);
  ctx.lineTo(cx + headR * 0.35, headCY + headR * 0.7);
  ctx.lineTo(cx + headR * 0.45, headCY + headR * 0.4);
  ctx.closePath();
  ctx.fill();

  ctx.restore();

  const barW = w * 1.3;
  const barH = 12;
  const barX = cx - barW/2;
  const barY = top - 32;
  ctx.fillStyle = 'rgba(0,0,0,0.75)';
  rr(barX - 3, barY - 3, barW + 6, barH + 6, 5);
  ctx.fill();
  const hpGrad = ctx.createLinearGradient(barX, 0, barX + barW, 0);
  hpGrad.addColorStop(0, '#dc2626');
  hpGrad.addColorStop(1, '#f97316');
  ctx.fillStyle = hpGrad;
  rr(barX, barY, barW * (b.hp / b.maxHp), barH, 4);
  ctx.fill();
  ctx.strokeStyle = 'rgba(255,255,255,0.9)';
  ctx.lineWidth = 1.5;
  rr(barX, barY, barW, barH, 4);
  ctx.stroke();

  ctx.fillStyle = '#fff';
  ctx.font = 'bold 14px system-ui, sans-serif';
  ctx.textAlign = 'center';
  ctx.fillText('JEFE FINAL', cx, barY - 8);
  ctx.textAlign = 'start';
}

function drawPickup(pk){
  const sc = 1.4;
  if (pk.landed){
    const c = pk.style === 'pretty' ? '251,191,36' : '148,163,184';
    const glow = ctx.createRadialGradient(pk.x, pk.y + 14, 2, pk.x, pk.y + 14, 32);
    glow.addColorStop(0, `rgba(${c},0.55)`);
    glow.addColorStop(1, `rgba(${c},0)`);
    ctx.fillStyle = glow;
    ctx.beginPath();
    ctx.arc(pk.x, pk.y + 14, 32, 0, Math.PI * 2);
    ctx.fill();
  }

  drawKnifeByStyle(pk.x, pk.y, pk.rot, pk.style, sc);

  if (pk.landed){
    ctx.strokeStyle = pk.style === 'pretty' ? 'rgba(251,191,36,0.85)' : 'rgba(148,163,184,0.85)';
    ctx.lineWidth = 2;
    ctx.setLineDash([4, 4]);
    ctx.beginPath();
    ctx.arc(pk.x, pk.y, 28, 0, Math.PI * 2);
    ctx.stroke();
    ctx.setLineDash([]);
  }
}

function drawParticles(){
  for (const pt of particles){
    ctx.globalAlpha = Math.max(0, pt.life / pt.maxLife);
    ctx.fillStyle = pt.color;
    ctx.beginPath();
    ctx.arc(pt.x, pt.y, pt.r, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;
}

function draw(){
  ctx.save();
  if (shake > 0){
    ctx.translate((Math.random() - 0.5) * shake, (Math.random() - 0.5) * shake);
  }

  drawBackground();

  ctx.save();
  ctx.translate(-camX, 0);

  for (const pk of pickups) drawPickup(pk);
  for (const z of zombies) drawZombie(z);
  if (boss) drawBoss(boss);

  for (const k of knives){
    drawKnifeByStyle(k.x, k.y, k.rot, player.currentStyle, 1);
  }

  if (invuln <= 0 || Math.floor(performance.now() / 100) % 2 === 0){
    drawPlayer();
  }

  drawParticles();

  ctx.restore();
  ctx.restore();
}

let last = 0;
function loop(t){
  if (!last) last = t;
  let dt = (t - last) / 1000;
  last = t;
  if (dt > 0.05) dt = 0.05;

  update(dt);
  draw();
  requestAnimationFrame(loop);
}

function bindHold(el, key){
  const on  = (e) => { e.preventDefault(); keys[key] = true;  el.classList.add('active'); };
  const off = (e) => { e.preventDefault(); keys[key] = false; el.classList.remove('active'); };
  el.addEventListener('touchstart', on,  {passive:false});
  el.addEventListener('touchend',   off, {passive:false});
  el.addEventListener('touchcancel',off, {passive:false});
  el.addEventListener('mousedown',  on);
  el.addEventListener('mouseup',    off);
  el.addEventListener('mouseleave', off);
}

bindHold(document.getElementById('btnLeft'),  'left');
bindHold(document.getElementById('btnRight'), 'right');
bindHold(document.getElementById('btnUp'),    'up');
bindHold(document.getElementById('btnDown'),  'down');

const btnKnife = document.getElementById('btnKnife');
function knifeOn(e){ e.preventDefault(); btnKnife.classList.add('active'); throwKnife(); }
function knifeOff(e){ e.preventDefault(); btnKnife.classList.remove('active'); }
btnKnife.addEventListener('touchstart', knifeOn,  {passive:false});
btnKnife.addEventListener('touchend',   knifeOff, {passive:false});
btnKnife.addEventListener('touchcancel',knifeOff, {passive:false});
btnKnife.addEventListener('mousedown',  knifeOn);
btnKnife.addEventListener('mouseup',    knifeOff);
btnKnife.addEventListener('mouseleave', knifeOff);

document.addEventListener('keydown', (e) => {
  if (e.key === 'ArrowLeft'  || e.key === 'a') keys.left  = true;
  if (e.key === 'ArrowRight' || e.key === 'd') keys.right = true;
  if (e.key === 'ArrowUp'    || e.key === 'w') keys.up    = true;
  if (e.key === 'ArrowDown'  || e.key === 's') keys.down  = true;
  if (e.key === ' ' || e.key === 'Enter') throwKnife();
});
document.addEventListener('keyup', (e) => {
  if (e.key === 'ArrowLeft'  || e.key === 'a') keys.left  = false;
  if (e.key === 'ArrowRight' || e.key === 'd') keys.right = false;
  if (e.key === 'ArrowUp'    || e.key === 'w') keys.up    = false;
  if (e.key === 'ArrowDown'  || e.key === 's') keys.down  = false;
});

document.getElementById('btnRestart').addEventListener('click', reset);
document.getElementById('btnRestart').addEventListener('touchstart',
  (e) => { e.preventDefault(); reset(); }, {passive:false});

window.addEventListener('resize', () => {
  const oldVW = VW;
  layout();
  if (player){
    if (oldVW > 0) player.x = (player.x / oldVW) * VW;
    player.x = Math.max(0, Math.min(WORLD_W - player.w, player.x));
  }
});
window.addEventListener('orientationchange', () => setTimeout(layout, 150));

layout();
reset();
requestAnimationFrame(loop);

})();
</script>
</body>
</html>