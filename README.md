<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Gummy Land — Исправленная версия</title>
<style>
  :root {
    --pink: #ff6b9d;
    --purple: #b56bff;
    --orange: #ff9d6b;
    --green: #6bff9d;
    --yellow: #ffdb6b;
    --blue: #6bb5ff;
    --dark: #3d2848;
    --mid: #6b4d7a;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; -webkit-tap-highlight-color: transparent; }
  html, body {
    width: 100%; height: 100%; overflow: hidden;
    font-family: 'Segoe UI', system-ui, sans-serif;
    background: linear-gradient(135deg, #fff8f0, #ffe8f5, #f0e8ff);
    color: var(--dark);
    user-select: none;
    touch-action: none;
  }
  #game { position: relative; width: 100vw; height: 100vh; }
  canvas { display: block; width: 100%; height: 100%; }

  /* UI OVERLAY */
  .screen {
    position: absolute; inset: 0;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    background: linear-gradient(135deg, rgba(255,248,240,0.95), rgba(255,232,245,0.95), rgba(240,232,255,0.95));
    backdrop-filter: blur(10px);
    z-index: 10;
    padding: 20px;
    text-align: center;
  }
  .screen.hidden { display: none; }

  .screen h1 {
    font-size: clamp(2.5rem, 8vw, 5rem);
    font-weight: 800;
    background: linear-gradient(135deg, var(--pink), var(--purple), var(--orange));
    -webkit-background-clip: text; background-clip: text;
    -webkit-text-fill-color: transparent;
    margin-bottom: 10px;
    letter-spacing: -2px;
  }
  .screen h2 {
    font-size: clamp(1.5rem, 5vw, 2.5rem);
    color: var(--purple);
    margin-bottom: 20px;
  }
  .screen p {
    font-size: 1.1rem; color: var(--mid);
    margin-bottom: 20px; max-width: 500px;
  }

  .logo-blob {
    width: 100px; height: 100px;
    background: linear-gradient(135deg, var(--pink), var(--purple));
    border-radius: 30% 70% 70% 30% / 30% 30% 70% 70%;
    margin-bottom: 20px;
    box-shadow: 0 12px 32px rgba(255,107,157,0.3), inset 0 -10px 20px rgba(0,0,0,0.1), inset 0 10px 20px rgba(255,255,255,0.3);
    animation: jello 3s ease-in-out infinite;
    position: relative;
  }
  .logo-blob::after {
    content: ''; position: absolute;
    top: 15px; left: 20px;
    width: 30px; height: 20px;
    background: rgba(255,255,255,0.5);
    border-radius: 50%; filter: blur(5px);
  }
  @keyframes jello {
    0%,100% { border-radius: 30% 70% 70% 30% / 30% 30% 70% 70%; }
    50% { border-radius: 45% 55% 55% 45% / 45% 45% 55% 55%; }
  }

  .btn {
    background: linear-gradient(135deg, var(--pink), var(--purple));
    color: white; border: none;
    padding: 14px 32px; border-radius: 50px;
    font-size: 1.1rem; font-weight: 600;
    cursor: pointer; margin: 6px;
    box-shadow: 0 6px 18px rgba(255,107,157,0.4);
    transition: transform 0.2s, box-shadow 0.2s;
    min-width: 180px;
  }
  .btn:hover { transform: translateY(-2px); box-shadow: 0 8px 24px rgba(255,107,157,0.5); }
  .btn:active { transform: translateY(0); }
  .btn.secondary {
    background: linear-gradient(135deg, var(--orange), var(--yellow));
    box-shadow: 0 6px 18px rgba(255,157,107,0.4);
  }
  .btn.ghost {
    background: rgba(255,255,255,0.7);
    color: var(--purple);
    box-shadow: 0 4px 12px rgba(181,107,255,0.15);
  }

  .stats {
    display: flex; gap: 20px; flex-wrap: wrap;
    justify-content: center; margin: 20px 0;
  }
  .stat {
    background: rgba(255,255,255,0.8);
    padding: 12px 20px; border-radius: 16px;
    box-shadow: 0 4px 12px rgba(181,107,255,0.1);
    border: 2px solid rgba(255,255,255,0.9);
  }
  .stat-label { font-size: 0.85rem; color: var(--mid); text-transform: uppercase; letter-spacing: 1px; }
  .stat-value { font-size: 1.5rem; font-weight: 700; color: var(--purple); }

  /* HUD */
  #hud {
    position: absolute; top: 0; left: 0; right: 0;
    padding: 16px;
    display: flex; justify-content: space-between; align-items: flex-start;
    pointer-events: none; z-index: 5;
    font-weight: 700;
  }
  #hud.hidden { display: none; }
  .hud-left, .hud-right, .hud-center {
    background: rgba(255,255,255,0.85);
    backdrop-filter: blur(10px);
    padding: 10px 16px; border-radius: 16px;
    box-shadow: 0 4px 12px rgba(181,107,255,0.15);
    border: 2px solid rgba(255,255,255,0.9);
  }
  .hud-left { display: flex; gap: 4px; }
  .heart {
    display: inline-block; width: 24px; height: 24px;
    background: linear-gradient(135deg, #ff6b9d, #ff3d7a);
    border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
    transform: rotate(-45deg);
    position: relative;
    transition: transform 0.3s;
  }
  .heart::before, .heart::after {
    content: ''; position: absolute;
    width: 24px; height: 24px;
    background: linear-gradient(135deg, #ff6b9d, #ff3d7a);
    border-radius: 50%;
  }
  .heart::before { top: -12px; left: 0; }
  .heart::after { top: 0; left: 12px; }
  .heart.lost { opacity: 0.2; transform: rotate(-45deg) scale(0.8); }

  .hud-center {
    font-size: 1rem; color: var(--purple);
    position: absolute; left: 50%; transform: translateX(-50%);
    transition: opacity 0.5s;
  }
  .hud-right { font-size: 1.1rem; color: var(--orange); }

  /* PAUSE BUTTON */
  #pauseBtn {
    position: absolute; bottom: 20px; right: 20px;
    width: 56px; height: 56px; border-radius: 50%;
    background: rgba(255,255,255,0.85);
    backdrop-filter: blur(10px);
    border: 2px solid rgba(255,255,255,0.9);
    box-shadow: 0 4px 12px rgba(181,107,255,0.2);
    cursor: pointer; z-index: 6;
    display: flex; align-items: center; justify-content: center;
    font-size: 1.5rem; color: var(--purple);
    pointer-events: auto;
  }
  #pauseBtn.hidden { display: none; }

  /* MOBILE CONTROLS */
  #mobileControls {
    position: absolute; bottom: 0; left: 0; right: 0;
    padding: 20px; display: none;
    justify-content: space-between; align-items: flex-end;
    z-index: 5; pointer-events: none;
  }
  #mobileControls.active { display: flex; }
  .touch-btn {
    width: 70px; height: 70px; border-radius: 50%;
    background: rgba(255,255,255,0.5);
    backdrop-filter: blur(10px);
    border: 2px solid rgba(255,255,255,0.7);
    display: flex; align-items: center; justify-content: center;
    font-size: 2rem; color: var(--purple);
    pointer-events: auto;
    user-select: none;
  }
  .touch-btn.jump {
    width: 90px; height: 90px;
    background: rgba(255,107,157,0.4);
    color: white;
  }
  .touch-left { display: flex; gap: 10px; }

  /* TUTORIAL */
  #tutorial {
    position: absolute; top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(255,255,255,0.95);
    padding: 20px 30px; border-radius: 20px;
    box-shadow: 0 12px 32px rgba(181,107,255,0.3);
    border: 3px solid var(--pink);
    font-size: 1.2rem; color: var(--dark);
    font-weight: 600;
    z-index: 7;
    opacity: 0; transition: opacity 0.4s;
    pointer-events: none;
    text-align: center;
    max-width: 90%;
  }
  #tutorial.show { opacity: 1; }
  #tutorial kbd {
    display: inline-block;
    background: linear-gradient(135deg, var(--purple), var(--pink));
    color: white; padding: 2px 10px; border-radius: 8px;
    margin: 0 3px; font-family: inherit;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  }

  /* ERROR */
  #errorScreen {
    background: linear-gradient(135deg, #ffe0e0, #ffb3b3);
  }
  #errorScreen h2 { color: #c0392b; }

  @media (max-width: 700px) {
    .btn { padding: 12px 24px; font-size: 1rem; min-width: 150px; }
    .hud-center { font-size: 0.85rem; padding: 6px 12px; }
    .hud-right { font-size: 0.95rem; padding: 6px 12px; }
    .hud-left { padding: 6px 10px; }
    .heart { width: 18px; height: 18px; }
    .heart::before, .heart::after { width: 18px; height: 18px; }
    .heart::before { top: -9px; }
    .heart::after { left: 9px; }
  }
</style>
</head>
<body>
<div id="game">
  <canvas id="canvas"></canvas>

  <!-- HUD -->
  <div id="hud" class="hidden">
    <div class="hud-left" id="hearts"></div>
    <div class="hud-center" id="levelLabel">Уровень 1</div>
    <div class="hud-right">🍬 <span id="scoreLabel">0</span></div>
  </div>
  <button id="pauseBtn" class="hidden">⏸</button>
  <div id="tutorial"></div>

  <!-- MOBILE CONTROLS -->
  <div id="mobileControls">
    <div class="touch-left">
      <div class="touch-btn" data-key="ArrowLeft">←</div>
      <div class="touch-btn" data-key="ArrowRight">→</div>
    </div>
    <div class="touch-btn jump" data-key="Space">⤴</div>
  </div>

  <!-- MENU -->
  <div id="menuScreen" class="screen">
    <div class="logo-blob"></div>
    <h1>Gummy Land</h1>
    <p>Путешествуй по мармеладной стране, прыгай по сладким платформам и собирай конфеты!</p>
    <button class="btn" id="playBtn">▶ Играть</button>
    <button class="btn secondary" id="continueBtn" style="display:none;">↻ Продолжить</button>
    <div class="stats" id="menuStats"></div>
  </div>

  <!-- PAUSE -->
  <div id="pauseScreen" class="screen hidden">
    <h2>⏸ Пауза</h2>
    <button class="btn" id="resumeBtn">▶ Продолжить</button>
    <button class="btn secondary" id="restartBtn">↻ Рестарт</button>
    <button class="btn ghost" id="menuBtn">🏠 В меню</button>
  </div>

  <!-- LEVEL COMPLETE -->
  <div id="completeScreen" class="screen hidden">
    <h2>🏆 Уровень пройден!</h2>
    <div class="stats" id="completeStats"></div>
    <button class="btn" id="nextBtn">→ Следующий уровень</button>
    <button class="btn ghost" id="completeMenuBtn">🏠 В меню</button>
  </div>

  <!-- GAME OVER -->
  <div id="gameOverScreen" class="screen hidden">
    <h2>💀 Игра окончена</h2>
    <p>Не расстраивайся — мармеладный мир ждёт тебя снова!</p>
    <div class="stats" id="gameOverStats"></div>
    <button class="btn" id="retryBtn">↻ Попробовать снова</button>
    <button class="btn ghost" id="gameOverMenuBtn">🏠 В меню</button>
  </div>

  <!-- ERROR -->
  <div id="errorScreen" class="screen hidden">
    <h2>⚠ Произошла ошибка</h2>
    <p id="errorMsg">Попробуйте перезагрузить страницу.</p>
    <button class="btn" onclick="location.reload()">Перезагрузить</button>
  </div>
</div>

<script>
'use strict';

// ============================================================
// ERROR HANDLING
// ============================================================
window.addEventListener('error', (e) => {
  console.error('Game error:', e.message, e.filename, e.lineno);
  showError('Произошла ошибка: ' + e.message);
});
window.addEventListener('unhandledrejection', (e) => {
  console.error('Promise error:', e.reason);
  showError('Произошла ошибка: ' + e.reason);
});
function showError(msg) {
  document.getElementById('errorMsg').textContent = msg;
  document.getElementById('errorScreen').classList.remove('hidden');
}

// ============================================================
// UTILS
// ============================================================
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const lerp = (a, b, t) => a + (b - a) * t;
const rand = (a, b) => a + Math.random() * (b - a);
const randInt = (a, b) => Math.floor(rand(a, b + 1));

function seededRandom(seed) {
  let s = seed % 2147483647;
  if (s <= 0) s += 2147483646;
  return () => {
    s = (s * 16807) % 2147483647;
    return (s - 1) / 2147483646;
  };
}

// ============================================================
// CANVAS
// ============================================================
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let W = 0, H = 0, DPR = 1;

function resize() {
  DPR = Math.min(window.devicePixelRatio || 1, 2);
  W = window.innerWidth;
  H = window.innerHeight;
  canvas.width = W * DPR;
  canvas.height = H * DPR;
  canvas.style.width = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
}
window.addEventListener('resize', resize);
resize();

// ============================================================
// INPUT
// ============================================================
const Input = {
  keys: {},
  justPressed: {},
  _prev: {},
  init() {
    window.addEventListener('keydown', (e) => {
      if (['ArrowUp','ArrowDown','ArrowLeft','ArrowRight','Space'].includes(e.code)) {
        e.preventDefault();
      }
      this.keys[e.code] = true;
    });
    window.addEventListener('keyup', (e) => {
      this.keys[e.code] = false;
    });
    // Mobile
    document.querySelectorAll('.touch-btn').forEach(btn => {
      const key = btn.dataset.key;
      const press = (e) => { e.preventDefault(); this.keys[key] = true; };
      const release = (e) => { e.preventDefault(); this.keys[key] = false; };
      btn.addEventListener('touchstart', press, { passive: false });
      btn.addEventListener('touchend', release);
      btn.addEventListener('touchcancel', release);
      btn.addEventListener('mousedown', press);
      btn.addEventListener('mouseup', release);
      btn.addEventListener('mouseleave', release);
    });
  },
  update() {
    for (const k in this.keys) {
      this.justPressed[k] = this.keys[k] && !this._prev[k];
    }
    this._prev = { ...this.keys };
  },
  isDown(code) { return !!this.keys[code]; },
  isJust(code) { return !!this.justPressed[code]; },
  left()  { return this.isDown('ArrowLeft')  || this.isDown('KeyA'); },
  right() { return this.isDown('ArrowRight') || this.isDown('KeyD'); },
  jump()  { return this.isDown('Space') || this.isDown('ArrowUp') || this.isDown('KeyW'); },
  jumpJust() { return this.isJust('Space') || this.isJust('ArrowUp') || this.isJust('KeyW'); }
};
Input.init();

// Detect mobile
const isMobile = /Mobi|Android|iPhone|iPad/i.test(navigator.userAgent) || ('ontouchstart' in window && window.innerWidth < 900);
if (isMobile) document.getElementById('mobileControls').classList.add('active');

// ============================================================
// AUDIO (Web Audio API procedural)
// ============================================================
const Audio = {
  ctx: null,
  masterGain: null,
  musicGain: null,
  sfxGain: null,
  musicOsc: null,
  musicLfo: null,
  enabled: true,
  init() {
    try {
      this.ctx = new (window.AudioContext || window.webkitAudioContext)();
      this.masterGain = this.ctx.createGain();
      this.masterGain.gain.value = 0.5;
      this.masterGain.connect(this.ctx.destination);
      this.musicGain = this.ctx.createGain();
      this.musicGain.gain.value = 0.15;
      this.musicGain.connect(this.masterGain);
      this.sfxGain = this.ctx.createGain();
      this.sfxGain.gain.value = 0.4;
      this.sfxGain.connect(this.masterGain);
    } catch (e) {
      console.warn('Audio init failed:', e);
      this.enabled = false;
    }
  },
  resume() {
    if (this.ctx && this.ctx.state === 'suspended') this.ctx.resume();
  },
  _tone(freq, dur, type = 'sine', vol = 0.3, slide = null) {
    if (!this.enabled) return;
    const t = this.ctx.currentTime;
    const osc = this.ctx.createOscillator();
    const g = this.ctx.createGain();
    osc.type = type;
    osc.frequency.setValueAtTime(freq, t);
    if (slide !== null) osc.frequency.exponentialRampToValueAtTime(Math.max(20, slide), t + dur);
    g.gain.setValueAtTime(vol, t);
    g.gain.exponentialRampToValueAtTime(0.001, t + dur);
    osc.connect(g); g.connect(this.sfxGain);
    osc.start(t); osc.stop(t + dur);
  },
  jump()    { this._tone(400, 0.12, 'sine', 0.25, 800); },
  dblJump() { this._tone(600, 0.15, 'triangle', 0.25, 1200); },
  land()    { this._tone(150, 0.08, 'sine', 0.2, 80); },
  candy()   { this._tone(800 + Game.combo * 40, 0.1, 'triangle', 0.2); },
  hurt()    { this._tone(200, 0.3, 'sawtooth', 0.25, 80); },
  kill()    { this._tone(600, 0.1, 'square', 0.15, 1200); this._tone(900, 0.1, 'square', 0.1, 1400); },
  win() {
    [523, 659, 784, 1047].forEach((f, i) => {
      setTimeout(() => this._tone(f, 0.2, 'triangle', 0.25), i * 100);
    });
  },
  lose() {
    [400, 300, 200].forEach((f, i) => {
      setTimeout(() => this._tone(f, 0.3, 'sine', 0.2), i * 150);
    });
  },
  startMusic() {
    if (!this.enabled) return;
    this.stopMusic();
    const t = this.ctx.currentTime;
    this.musicOsc = this.ctx.createOscillator();
    this.musicOsc.type = 'triangle';
    this.musicOsc.frequency.value = 220;
    const lfo = this.ctx.createOscillator();
    lfo.frequency.value = 0.3;
    const lfoGain = this.ctx.createGain();
    lfoGain.gain.value = 30;
    lfo.connect(lfoGain);
    lfoGain.connect(this.musicOsc.frequency);
    this.musicOsc.connect(this.musicGain);
    this.musicOsc.start(t);
    lfo.start(t);
    this.musicLfo = lfo;
  },
  stopMusic() {
    try {
      if (this.musicOsc) { this.musicOsc.stop(); this.musicOsc = null; }
      if (this.musicLfo) { this.musicLfo.stop(); this.musicLfo = null; }
    } catch (e) {}
  }
};

// ============================================================
// SAVE SYSTEM
// ============================================================
const Save = {
  KEY: 'gummyLandSave_v1',
  data: null,
  load() {
    try {
      const raw = localStorage.getItem(this.KEY);
      if (!raw) return this.defaultData();
      const parsed = JSON.parse(raw);
      if (typeof parsed.level !== 'number' || typeof parsed.highScore !== 'number') {
        throw new Error('Invalid save');
      }
      return parsed;
    } catch (e) {
      console.warn('Save corrupted:', e);
      return this.defaultData();
    }
  },
  save() {
    try { localStorage.setItem(this.KEY, JSON.stringify(this.data)); }
    catch (e) { console.warn('Save failed:', e); }
  },
  defaultData() {
    return { level: 1, highScore: 0, maxLevel: 1 };
  },
  reset() {
    this.data = this.defaultData();
    this.save();
  }
};
Save.data = Save.load();

// ============================================================
// LEVEL GENERATOR
// ============================================================
const LevelGen = {
  getConfig(n) {
    return {
      length: 2000 + n * 400,
      minWidth: Math.max(55, 120 - n * 8),
      maxWidth: Math.max(85, 160 - n * 8),
      minGap: Math.min(180, 60 + n * 15),
      maxGap: Math.min(230, 90 + n * 18),
      maxHDiff: 120,
      enemyCount: Math.min(20, Math.max(0, (n - 1) * 2)),
      hasMoving: n >= 4,
      hasTraps: n >= 5,
      hasDisappearing: n >= 7,
      tutorial: n === 1,
      doubleJump: n >= 3
    };
  },

  generate(levelNum) {
    const cfg = this.getConfig(levelNum);
    const seed = levelNum * 9973 + 42;
    const rng = seededRandom(seed);

    const platforms = [];
    const enemies = [];
    const items = [];
    const traps = [];

    // Start platform
    platforms.push({
      x: 0, y: 400, width: 300, height: 30,
      type: 'start', color: '#b56bff'
    });

    let cx = 300, cy = 400;
    const finishX = cfg.length - 300;

    while (cx < finishX - 200) {
      const gap = cfg.minGap + rng() * (cfg.maxGap - cfg.minGap);
      const width = cfg.minWidth + rng() * (cfg.maxWidth - cfg.minWidth);
      const dy = (rng() - 0.5) * cfg.maxHDiff;

      cx += gap;
      cy = clamp(cy + dy, 180, 520);

      let type = 'normal';
      if (cfg.hasMoving && rng() < 0.15) type = 'moving';
      else if (cfg.hasDisappearing && rng() < 0.1) type = 'disappearing';

      const plat = {
        x: cx, y: cy, width, height: 28,
        type,
        color: this._platColor(rng),
        origY: cy,
        moveAmp: type === 'moving' ? 40 + rng() * 40 : 0,
        moveSpeed: type === 'moving' ? 1 + rng() * 1.5 : 0,
        movePhase: rng() * Math.PI * 2,
        disappearTimer: 0,
        visible: true
      };
      platforms.push(plat);

      // Items on platform
      const itemCount = Math.floor(width / 40);
      for (let i = 0; i < itemCount; i++) {
        if (rng() < 0.7) {
          const isGolden = rng() < 0.1;
          items.push({
            x: plat.x + 20 + i * 40,
            y: plat.y - 30 - (rng() < 0.3 ? 30 : 0),
            type: isGolden ? 'golden' : 'candy',
            collected: false,
            bobPhase: rng() * Math.PI * 2
          });
        }
      }

      // Enemies
      if (cx > cfg.length * 0.3 && enemies.length < cfg.enemyCount && width >= 80) {
        if (rng() < 0.5) {
          const enemyType = this._pickEnemy(levelNum, rng);
          if (enemyType === 'bee') {
            enemies.push({
              type: 'bee',
              x: plat.x + plat.width / 2,
              y: plat.y - 80,
              baseY: plat.y - 80,
              vx: 0, vy: 0,
              width: 30, height: 30,
              alive: true,
              phase: rng() * Math.PI * 2,
              patrolLeft: plat.x - 40,
              patrolRight: plat.x + plat.width + 40,
              dir: 1
            });
          } else if (enemyType === 'ghost') {
            enemies.push({
              type: 'ghost',
              x: plat.x + plat.width / 2,
              y: plat.y - 60,
              vx: 0, vy: 0,
              width: 34, height: 34,
              alive: true,
              speed: 50 + levelNum * 5
            });
          } else {
            enemies.push({
              type: enemyType,
              x: plat.x + 20,
              y: plat.y - 24,
              vx: enemyType === 'mushroom' ? 0 : 60 + levelNum * 5,
              width: 28, height: 28,
              alive: true,
              platform: plat,
              chargeTimer: 0,
              charging: false
            });
          }
        }
      }

      // Traps (spikes)
      if (cfg.hasTraps && rng() < 0.2 && width >= 100) {
        traps.push({
          x: plat.x + plat.width / 2 - 15,
          y: plat.y - 14,
          width: 30, height: 14
        });
      }
    }

    // Finish platform
    platforms.push({
      x: finishX, y: 400, width: 300, height: 30,
      type: 'finish', color: '#6bff9d'
    });

    // Rare powerups
    if (rng() < 0.5) {
      items.push({
        x: cfg.length * 0.5, y: 200,
        type: 'heart', collected: false, bobPhase: 0
      });
    }
    if (levelNum >= 3 && rng() < 0.4) {
      items.push({
        x: cfg.length * 0.7, y: 220,
        type: 'shield', collected: false, bobPhase: 0
      });
    }

    // Validate
    if (!this.validate(platforms, cfg)) {
      return this.generate(levelNum + 0.0001); // retry with different seed
    }

    return {
      platforms, enemies, items, traps,
      finishX: finishX + 150,
      length: cfg.length,
      config: cfg
    };
  },

  _platColor(rng) {
    const colors = ['#b56bff', '#ff6b9d', '#ff9d6b', '#6bff9d', '#6bb5ff'];
    return colors[Math.floor(rng() * colors.length)];
  },

  _pickEnemy(levelNum, rng) {
    const r = rng();
    if (levelNum >= 8 && r < 0.2) return 'ghost';
    if (levelNum >= 6 && r < 0.4) return 'bee';
    if (levelNum >= 4 && r < 0.5) return 'mushroom';
    return 'slime';
  },

  validate(platforms, cfg) {
    const maxJumpDist = 280;
    const maxJumpHeight = 180;
    for (let i = 1; i < platforms.length; i++) {
      const prev = platforms[i - 1];
      const curr = platforms[i];
      const dx = curr.x - (prev.x + prev.width);
      const dy = prev.y - curr.y;
      if (dx > maxJumpDist || dy > maxJumpHeight) return false;
    }
    return true;
  }
};

// ============================================================
// PLAYER
// ============================================================
const Player = {
  x: 100, y: 300,
  vx: 0, vy: 0,
  width: 28, height: 36,
  onGround: false,
  facing: 1,
  lives: 3,
  jumpsLeft: 2, // ИЗМЕНЕНО: Теперь по умолчанию 2 прыжка
  maxJumps: 2,  // ИЗМЕНЕНО: Максимум 2
  coyoteTime: 0,
  jumpBuffer: 0,
  jumpHeld: false,
  invuln: 0,
  shield: 0,
  speedBoost: 0,
  squashX: 1, squashY: 1,
  targetSquashX: 1, targetSquashY: 1,
  runPhase: 0,
  blinkTimer: 0,

  reset() {
    this.x = 100; this.y = 300;
    this.vx = 0; this.vy = 0;
    this.onGround = false;
    this.facing = 1;
    this.jumpsLeft = this.maxJumps;
    this.coyoteTime = 0;
    this.jumpBuffer = 0;
    this.invuln = 0;
    this.squashX = 1; this.squashY = 1;
  },

  update(dt, level) {
    const maxSpeed = this.speedBoost > 0 ? 400 : 320;
    const accel = 1800;
    const decel = 2400;

    // Horizontal
    let targetVx = 0;
    if (Input.left())  targetVx = -maxSpeed;
    if (Input.right()) targetVx = maxSpeed;

    if (targetVx !== 0) {
      this.vx = lerp(this.vx, targetVx, Math.min(1, accel * dt / maxSpeed));
      this.facing = targetVx > 0 ? 1 : -1;
      if (this.onGround) this.runPhase += dt * 10;
    } else {
      if (Math.abs(this.vx) < decel * dt) this.vx = 0;
      else this.vx -= Math.sign(this.vx) * decel * dt;
    }

    // Jump buffer
    if (Input.jumpJust()) this.jumpBuffer = 0.1;
    this.jumpBuffer -= dt;

    // Coyote time (увеличено до 0.15 для более легкого прыжка)
    if (this.onGround) {
      this.coyoteTime = 0.15; 
      this.jumpsLeft = this.maxJumps;
    } else {
      this.coyoteTime -= dt;
    }

    // Jump Logic
    if (this.jumpBuffer > 0) {
      // Прыжок с земли или двойной прыжок в воздухе
      if (this.coyoteTime > 0 || this.jumpsLeft > 0) {
        const isDouble = this.coyoteTime <= 0 && this.jumpsLeft > 0;
        
        // УВЕЛИЧЕНА сила прыжка с 580 до 620
        this.vy = -620; 
        
        this.jumpHeld = true;
        this.jumpBuffer = 0;
        this.coyoteTime = 0;
        
        if (isDouble) {
          this.jumpsLeft--;
          Audio.dblJump();
          Particles.burst(this.x + this.width/2, this.y + this.height, '#ffdb6b', 8);
        } else {
          Audio.jump();
          Particles.burst(this.x + this.width/2, this.y + this.height, '#ff6b9d', 5);
        }
        this.targetSquashX = 0.75;
        this.targetSquashY = 1.25;
      }
    }

    // Variable jump (удержание кнопки)
    if (!Input.jump() && this.vy < 0 && this.jumpHeld) {
      this.vy *= 0.5;
      this.jumpHeld = false;
    }
    if (!Input.jump()) this.jumpHeld = false;

    // Gravity
    let gravity = 1800;
    if (this.vy < 0 && this.jumpHeld) gravity *= 0.5;
    this.vy += gravity * dt;
    if (this.vy > 900) this.vy = 900;

    // Squash recovery
    this.squashX = lerp(this.squashX, this.targetSquashX, 0.2);
    this.squashY = lerp(this.squashY, this.targetSquashY, 0.2);
    this.targetSquashX = lerp(this.targetSquashX, 1, 0.15);
    this.targetSquashY = lerp(this.targetSquashY, 1, 0.15);

    // Run squash
    if (this.onGround && Math.abs(this.vx) > 50) {
      this.targetSquashX = 1.05 + Math.sin(this.runPhase) * 0.05;
      this.targetSquashY = 0.95 - Math.sin(this.runPhase) * 0.05;
    }

    // Fall squash
    if (this.vy > 300) {
      this.targetSquashX = 1.15;
      this.targetSquashY = 0.85;
    }

    // Blink
    this.blinkTimer -= dt;
    if (this.blinkTimer <= 0) this.blinkTimer = 3 + Math.random() * 3;

    // Timers
    if (this.invuln > 0) this.invuln -= dt;
    if (this.shield > 0) this.shield -= dt;
    if (this.speedBoost > 0) this.speedBoost -= dt;

    // Move & collide
    this._moveAndCollide(dt, level);
  },

  _moveAndCollide(dt, level) {
    // X axis
    this.x += this.vx * dt;
    for (const p of level.platforms) {
      if (!p.visible) continue;
      if (this._aabb(p)) {
        if (this.vx > 0) this.x = p.x - this.width;
        else if (this.vx < 0) this.x = p.x + p.width;
        this.vx = 0;
      }
    }

    // Y axis
    this.y += this.vy * dt;
    this.onGround = false;
    for (const p of level.platforms) {
      if (!p.visible) continue;
      if (this._aabb(p)) {
        if (this.vy > 0) {
          this.y = p.y - this.height;
          this.onGround = true;
          if (this.vy > 300) {
            this.targetSquashX = 1.3;
            this.targetSquashY = 0.7;
            Audio.land();
            Particles.burst(this.x + this.width/2, this.y + this.height, '#ffffff', 4);
          }
          // Moving platform carry
          if (p.type === 'moving') {
            this.x += p._lastDx || 0;
          }
          // Disappearing
          if (p.type === 'disappearing' && p.disappearTimer === 0) {
            p.disappearTimer = 0.6;
          }
        } else if (this.vy < 0) {
          this.y = p.y + p.height;
        }
        this.vy = 0;
      }
    }

    // Bounds
    if (this.x < 0) { this.x = 0; this.vx = 0; }

    // Fall off
    if (this.y > 800) {
      this.hurt(true);
    }
  },

  _aabb(p) {
    return this.x < p.x + p.width &&
           this.x + this.width > p.x &&
           this.y < p.y + p.height &&
           this.y + this.height > p.y;
  },

  hurt(instant) {
    if (this.invuln > 0 && !instant) return;
    if (this.shield > 0 && !instant) {
      this.shield = 0;
      this.invuln = 1;
      Particles.burst(this.x + this.width/2, this.y + this.height/2, '#6bb5ff', 12);
      Audio.hurt();
      return;
    }
    this.lives--;
    Game.onHurt();
    Audio.hurt();
    Particles.burst(this.x + this.width/2, this.y + this.height/2, '#ff6b9d', 15);
    if (this.lives <= 0) {
      Game.gameOver();
    } else {
      this.invuln = 1.5;
      if (instant) {
        // Respawn at start
        this.x = 100; this.y = 300;
        this.vx = 0; this.vy = 0;
      } else {
        this.vy = -300;
        this.vx = -this.facing * 200;
      }
    }
  },

  draw(ctx, cam) {
    const x = this.x - cam.x;
    const y = this.y - cam.y;
    const cx = x + this.width / 2;
    const cy = y + this.height / 2;

    // Invuln blink
    if (this.invuln > 0 && Math.floor(this.invuln * 10) % 2 === 0) return;

    ctx.save();
    ctx.translate(cx, cy);
    ctx.scale(this.facing * this.squashX, this.squashY);

    // Body
    const grad = ctx.createRadialGradient(-5, -5, 0, 0, 0, 22);
    grad.addColorStop(0, '#ffb3d0');
    grad.addColorStop(1, '#ff6b9d');
    ctx.fillStyle = grad;
    ctx.beginPath();
    ctx.ellipse(0, 0, 14, 18, 0, 0, Math.PI * 2);
    ctx.fill();

    // Highlight
    ctx.fillStyle = 'rgba(255,255,255,0.5)';
    ctx.beginPath();
    ctx.ellipse(-5, -8, 5, 3, -0.5, 0, Math.PI * 2);
    ctx.fill();

    // Eyes
    const eyeOpen = this.blinkTimer < 0.1 ? 0.2 : 1;
    ctx.fillStyle = '#3d2848';
    ctx.beginPath();
    ctx.ellipse(-5, -3, 2, 3 * eyeOpen, 0, 0, Math.PI * 2);
    ctx.ellipse(5, -3, 2, 3 * eyeOpen, 0, 0, Math.PI * 2);
    ctx.fill();

    // Mouth
    ctx.strokeStyle = '#3d2848';
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    ctx.arc(0, 5, 3, 0, Math.PI);
    ctx.stroke();

    // Shield aura
    if (this.shield > 0) {
      ctx.strokeStyle = 'rgba(107,181,255,0.6)';
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.arc(0, 0, 22, 0, Math.PI * 2);
      ctx.stroke();
    }

    ctx.restore();
  }
};

// ============================================================
// ENEMIES
// ============================================================
function updateEnemies(dt, level) {
  for (const e of level.enemies) {
    if (!e.alive) continue;

    if (e.type === 'slime') {
      e.x += e.vx * dt;
      if (e.x < e.platform.x + 5) { e.x = e.platform.x + 5; e.vx = Math.abs(e.vx); }
      if (e.x > e.platform.x + e.platform.width - 30) {
        e.x = e.platform.x + e.platform.width - 30;
        e.vx = -Math.abs(e.vx);
      }
    }
    else if (e.type === 'mushroom') {
      const dx = Player.x - e.x;
      const dist = Math.abs(dx);
      if (dist < 150 && !e.charging) {
        e.chargeTimer += dt;
        if (e.chargeTimer > 0.5) {
          e.charging = true;
          e.vx = Math.sign(dx) * 250;
          e.chargeTimer = 0;
        }
      } else {
        e.chargeTimer = Math.max(0, e.chargeTimer - dt);
      }
      if (e.charging) {
        e.x += e.vx * dt;
        e.chargeTimer += dt;
        if (e.chargeTimer > 0.6) {
          e.charging = false;
          e.vx = 0;
          e.chargeTimer = 0;
        }
      }
    }
    else if (e.type === 'bee') {
      e.phase += dt * 3;
      e.y = e.baseY + Math.sin(e.phase) * 30;
      e.x += e.dir * 80 * dt;
      if (e.x < e.patrolLeft) { e.x = e.patrolLeft; e.dir = 1; }
      if (e.x > e.patrolRight) { e.x = e.patrolRight; e.dir = -1; }
    }
    else if (e.type === 'ghost') {
      const dx = Player.x - e.x;
      const dy = Player.y - e.y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      if (dist > 1 && dist < 400) {
        e.x += (dx / dist) * e.speed * dt;
        e.y += (dy / dist) * e.speed * dt;
      }
    }
  }
}

function drawEnemy(ctx, e, cam) {
  if (!e.alive) return;
  const x = e.x - cam.x;
  const y = e.y - cam.y;
  if (x < -50 || x > W + 50) return;

  ctx.save();
  ctx.translate(x + e.width/2, y + e.height/2);

  if (e.type === 'slime') {
    const wob = Math.sin(Date.now() * 0.008) * 0.1;
    ctx.scale(1 + wob, 1 - wob);
    const grad = ctx.createRadialGradient(-4, -4, 0, 0, 0, 16);
    grad.addColorStop(0, '#b3ffd0');
    grad.addColorStop(1, '#6bff9d');
    ctx.fillStyle = grad;
    ctx.beginPath();
    ctx.ellipse(0, 0, 14, 12, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillStyle = 'rgba(255,255,255,0.4)';
    ctx.beginPath();
    ctx.ellipse(-4, -5, 4, 2, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillStyle = '#3d2848';
    ctx.beginPath();
    ctx.arc(-4, 0, 1.5, 0, Math.PI * 2);
    ctx.arc(4, 0, 1.5, 0, Math.PI * 2);
    ctx.fill();
  }
  else if (e.type === 'mushroom') {
    // Stem
    ctx.fillStyle = '#fff0e6';
    ctx.fillRect(-5, 0, 10, 10);
    // Cap
    const grad = ctx.createRadialGradient(-4, -6, 0, 0, -4, 14);
    grad.addColorStop(0, '#ffc59d');
    grad.addColorStop(1, '#ff9d6b');
    ctx.fillStyle = grad;
    ctx.beginPath();
    ctx.arc(0, -4, 14, Math.PI, 0);
    ctx.fill();
    // Spots
    ctx.fillStyle = '#fff';
    ctx.beginPath();
    ctx.arc(-6, -8, 2, 0, Math.PI * 2);
    ctx.arc(4, -6, 2, 0, Math.PI * 2);
    ctx.fill();
    // Eyes
    ctx.fillStyle = '#3d2848';
    const angry = e.charging ? 1 : 0;
    ctx.beginPath();
    ctx.arc(-4, 2, 1.5, 0, Math.PI * 2);
    ctx.arc(4, 2, 1.5, 0, Math.PI * 2);
    ctx.fill();
    if (angry) {
      ctx.strokeStyle = '#3d2848';
      ctx.lineWidth = 1.5;
      ctx.beginPath();
      ctx.moveTo(-6, 0); ctx.lineTo(-2, 1);
      ctx.moveTo(6, 0); ctx.lineTo(2, 1);
      ctx.stroke();
    }
  }
  else if (e.type === 'bee') {
    const wob = Math.sin(Date.now() * 0.02) * 0.1;
    ctx.scale(1, 1 + wob);
    // Wings
    ctx.fillStyle = 'rgba(255,255,255,0.6)';
    ctx.beginPath();
    ctx.ellipse(-8, -8, 6, 10, -0.3, 0, Math.PI * 2);
    ctx.ellipse(8, -8, 6, 10, 0.3, 0, Math.PI * 2);
    ctx.fill();
    // Body
    const grad = ctx.createRadialGradient(-3, -3, 0, 0, 0, 14);
    grad.addColorStop(0, '#ffeab3');
    grad.addColorStop(1, '#ffdb6b');
    ctx.fillStyle = grad;
    ctx.beginPath();
    ctx.ellipse(0, 0, 12, 10, 0, 0, Math.PI * 2);
    ctx.fill();
    // Stripes
    ctx.fillStyle = '#3d2848';
    ctx.fillRect(-10, -2, 20, 2);
    ctx.fillRect(-10, 3, 20, 2);
    // Eyes
    ctx.fillStyle = '#fff';
    ctx.beginPath();
    ctx.arc(-4, -3, 2, 0, Math.PI * 2);
    ctx.arc(4, -3, 2, 0, Math.PI * 2);
    ctx.fill();
  }
  else if (e.type === 'ghost') {
    const wob = Math.sin(Date.now() * 0.005) * 0.15;
    ctx.globalAlpha = 0.7 + wob * 0.3;
    const grad = ctx.createRadialGradient(-4, -4, 0, 0, 0, 18);
    grad.addColorStop(0, '#fff');
    grad.addColorStop(1, '#d9b3ff');
    ctx.fillStyle = grad;
    ctx.beginPath();
    ctx.arc(0, -4, 14, Math.PI, 0);
    ctx.lineTo(14, 12);
    ctx.lineTo(8, 8);
    ctx.lineTo(4, 12);
    ctx.lineTo(0, 8);
    ctx.lineTo(-4, 12);
    ctx.lineTo(-8, 8);
    ctx.lineTo(-14, 12);
    ctx.closePath();
    ctx.fill();
    ctx.globalAlpha = 1;
    ctx.fillStyle = '#3d2848';
    ctx.beginPath();
    ctx.arc(-5, -4, 2, 0, Math.PI * 2);
    ctx.arc(5, -4, 2, 0, Math.PI * 2);
    ctx.fill();
  }

  ctx.restore();
}

// ============================================================
// PARTICLES
// ============================================================
const Particles = {
  pool: [],
  active: [],
  init() {
    for (let i = 0; i < 200; i++) {
      this.pool.push({ x:0, y:0, vx:0, vy:0, life:0, maxLife:1, color:'#fff', size:3 });
    }
  },
  burst(x, y, color, count = 8) {
    for (let i = 0; i < count; i++) {
      const p = this.pool.pop();
      if (!p) break;
      const angle = Math.random() * Math.PI * 2;
      const speed = 50 + Math.random() * 150;
      p.x = x; p.y = y;
      p.vx = Math.cos(angle) * speed;
      p.vy = Math.sin(angle) * speed - 50;
      p.life = 0.6 + Math.random() * 0.4;
      p.maxLife = p.life;
      p.color = color;
      p.size = 2 + Math.random() * 3;
      this.active.push(p);
    }
  },
  update(dt) {
    for (let i = this.active.length - 1; i >= 0; i--) {
      const p = this.active[i];
      p.x += p.vx * dt;
      p.y += p.vy * dt;
      p.vy += 400 * dt;
      p.life -= dt;
      if (p.life <= 0) {
        this.pool.push(p);
        this.active.splice(i, 1);
      }
    }
  },
  draw(ctx, cam) {
    for (const p of this.active) {
      const alpha = p.life / p.maxLife;
      ctx.globalAlpha = alpha;
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.arc(p.x - cam.x, p.y - cam.y, p.size, 0, Math.PI * 2);
      ctx.fill();
    }
    ctx.globalAlpha = 1;
  },
  clear() {
    while (this.active.length) this.pool.push(this.active.pop());
  }
};
Particles.init();

// ============================================================
// CAMERA
// ============================================================
const Camera = {
  x: 0, y: 0,
  targetX: 0, targetY: 0,
  update(dt, level) {
    this.targetX = Player.x + Player.width/2 - W/2;
    this.targetY = Player.y + Player.height/2 - H/2 + 50;
    this.targetX = clamp(this.targetX, 0, Math.max(0, level.length - W));
    this.targetY = clamp(this.targetY, -100, 200);
    this.x = lerp(this.x, this.targetX, 0.1);
    this.y = lerp(this.y, this.targetY, 0.08);
  }
};

// ============================================================
// GAME STATE
// ============================================================
const Game = {
  state: 'MENU', // MENU, PLAYING, PAUSED, COMPLETE, GAMEOVER
  level: 1,
  score: 0,
  combo: 0,
  comboTimer: 0,
  levelData: null,
  time: 0,
  tutorialShown: {},
  levelStartTime: 0,

  start(levelNum) {
    this.level = levelNum;
    this.score = 0;
    this.combo = 0;
    this.comboTimer = 0;
    this.time = 0;
    this.tutorialShown = {};
    Player.lives = 3;
    Player.maxJumps = 2; // Всегда 2 прыжка для удобства
    this.levelData = LevelGen.generate(levelNum);
    Player.reset();
    Camera.x = 0; Camera.y = 0;
    Particles.clear();
    this.state = 'PLAYING';
    this.levelStartTime = Date.now();
    Audio.startMusic();
    UI.showHud();
    UI.updateHearts();
    UI.updateScore();
    document.getElementById('levelLabel').textContent = 'Уровень ' + levelNum;
    document.getElementById('levelLabel').style.opacity = '1';
    setTimeout(() => {
      document.getElementById('levelLabel').style.opacity = '0';
    }, 3000);
  },

  update(dt) {
    if (this.state !== 'PLAYING') return;
    this.time += dt;

    // Combo decay
    this.comboTimer -= dt;
    if (this.comboTimer <= 0) this.combo = 0;

    // Update moving platforms
    const prevX = {};
    for (const p of this.levelData.platforms) {
      if (p.type === 'moving') {
        const oldX = p.x;
        const oldY = p.y;
        p.y = p.origY + Math.sin(this.time * p.moveSpeed + p.movePhase) * p.moveAmp;
        p._lastDx = p.x - oldX;
        p._lastDy = p.y - oldY;
      }
      if (p.type === 'disappearing' && p.disappearTimer > 0) {
        p.disappearTimer -= dt;
        if (p.disappearTimer <= 0) {
          p.visible = false;
          setTimeout(() => {
            p.visible = true;
            p.disappearTimer = 0;
          }, 2000);
        }
      }
    }

    Player.update(dt, this.levelData);
    updateEnemies(dt, this.levelData);
    Particles.update(dt);
    Camera.update(dt, this.levelData);

    // Collisions: player vs enemies
    for (const e of this.levelData.enemies) {
      if (!e.alive) continue;
      if (this._aabb(Player, e)) {
        // Stomp from above
        if (Player.vy > 0 && Player.y + Player.height - e.y < 15) {
          e.alive = false;
          Player.vy = -400;
          this.score += 50;
          UI.updateScore();
          Audio.kill();
          Particles.burst(e.x + e.width/2, e.y + e.height/2, '#ff6b9d', 12);
        } else {
          Player.hurt(false);
        }
      }
    }

    // Traps
    for (const t of this.levelData.traps) {
      if (this._aabb(Player, t)) {
        Player.hurt(false);
      }
    }

    // Items
    for (const it of this.levelData.items) {
      if (it.collected) continue;
      if (Math.abs(Player.x + Player.width/2 - it.x) < 20 &&
          Math.abs(Player.y + Player.height/2 - it.y) < 20) {
        it.collected = true;
        this._collectItem(it);
      }
    }

    // Finish
    if (Player.x + Player.width/2 >= this.levelData.finishX) {
      this.completeLevel();
    }

    // Tutorial
    if (this.levelData.config.tutorial) {
      this._checkTutorial();
    }
  },

  _aabb(a, b) {
    return a.x < b.x + b.width &&
           a.x + a.width > b.x &&
           a.y < b.y + b.height &&
           a.y + a.height > b.y;
  },

  _collectItem(it) {
    if (it.type === 'candy') {
      this.combo++;
      this.comboTimer = 2;
      this.score += 10 * Math.max(1, this.combo);
      Audio.candy();
      Particles.burst(it.x, it.y, '#ffdb6b', 6);
    } else if (it.type === 'golden') {
      this.score += 50;
      Audio.candy();
      Particles.burst(it.x, it.y, '#ffdb6b', 12);
    } else if (it.type === 'heart') {
      Player.lives = Math.min(5, Player.lives + 1);
      Particles.burst(it.x, it.y, '#ff6b9d', 12);
      Audio.candy();
    } else if (it.type === 'shield') {
      Player.shield = 10;
      Particles.burst(it.x, it.y, '#6bb5ff', 12);
      Audio.candy();
    }
    UI.updateScore();
    UI.updateHearts();
  },

  _checkTutorial() {
    const t = this.tutorialShown;
    if (!t.move && this.time > 0.5) {
      UI.showTutorial('Используй <kbd>←</kbd> <kbd>→</kbd> или <kbd>A</kbd> <kbd>D</kbd> для движения', 4000);
      t.move = true;
    }
    if (!t.jump && Player.x > 350) {
      UI.showTutorial('Нажми <kbd>Space</kbd> или <kbd>↑</kbd> для прыжка', 4000);
      t.jump = true;
    }
    if (!t.candy && this.levelData.items.some(i => !i.collected && i.x < Player.x + 200)) {
      UI.showTutorial('Собирай 🍬 мармеладки для очков!', 3000);
      t.candy = true;
    }
    if (!t.stomp && this.levelData.enemies.some(e => e.alive && e.x < Player.x + 200)) {
      UI.showTutorial('Прыгай на врагов сверху, чтобы победить их!', 4000);
      t.stomp = true;
    }
    if (!t.dbl && this.level >= 3 && Player.x > this.levelData.length * 0.3) {
      UI.showTutorial('У тебя появился <kbd>двойной прыжок</kbd>!', 4000);
      t.dbl = true;
    }
  },

  onHurt() {
    UI.updateHearts();
  },

  completeLevel() {
    this.state = 'COMPLETE';
    Audio.stopMusic();
    Audio.win();
    const elapsed = Math.floor((Date.now() - this.levelStartTime) / 1000);
    if (this.score > Save.data.highScore) Save.data.highScore = this.score;
    if (this.level + 1 > Save.data.maxLevel) Save.data.maxLevel = this.level + 1;
    Save.data.level = this.level + 1;
    Save.save();
    UI.showComplete(this.score, elapsed);
  },

  gameOver() {
    this.state = 'GAMEOVER';
    Audio.stopMusic();
    Audio.lose();
    if (this.score > Save.data.highScore) Save.data.highScore = this.score;
    Save.save();
    UI.showGameOver(this.score, this.level);
  },

  pause() {
    if (this.state !== 'PLAYING') return;
    this.state = 'PAUSED';
    Audio.stopMusic();
    UI.showPause();
  },

  resume() {
    if (this.state !== 'PAUSED') return;
    this.state = 'PLAYING';
    Audio.startMusic();
    UI.hidePause();
  }
};

// ============================================================
// UI
// ============================================================
const UI = {
  showHud() {
    document.getElementById('hud').classList.remove('hidden');
    document.getElementById('pauseBtn').classList.remove('hidden');
  },
  hideHud() {
    document.getElementById('hud').classList.add('hidden');
    document.getElementById('pauseBtn').classList.add('hidden');
  },
  updateHearts() {
    const el = document.getElementById('hearts');
    el.innerHTML = '';
    for (let i = 0; i < 5; i++) {
      const h = document.createElement('div');
      h.className = 'heart' + (i >= Player.lives ? ' lost' : '');
      el.appendChild(h);
    }
  },
  updateScore() {
    document.getElementById('scoreLabel').textContent = Game.score;
  },
  showTutorial(html, duration) {
    const el = document.getElementById('tutorial');
    el.innerHTML = html;
    el.classList.add('show');
    clearTimeout(this._tutTimeout);
    this._tutTimeout = setTimeout(() => el.classList.remove('show'), duration);
  },
  showMenu() {
    document.getElementById('menuScreen').classList.remove('hidden');
    document.getElementById('pauseScreen').classList.add('hidden');
    document.getElementById('completeScreen').classList.add('hidden');
    document.getElementById('gameOverScreen').classList.add('hidden');
    this.hideHud();
    // Continue button
    const contBtn = document.getElementById('continueBtn');
    if (Save.data.level > 1) {
      contBtn.style.display = '';
      contBtn.textContent = '↻ Продолжить (ур. ' + Save.data.level + ')';
    } else {
      contBtn.style.display = 'none';
    }
    // Stats
    document.getElementById('menuStats').innerHTML = `
      <div class="stat"><div class="stat-label">Рекорд</div><div class="stat-value">${Save.data.highScore}</div></div>
      <div class="stat"><div class="stat-label">Макс. уровень</div><div class="stat-value">${Save.data.maxLevel}</div></div>
    `;
  },
  showPause() {
    document.getElementById('pauseScreen').classList.remove('hidden');
  },
  hidePause() {
    document.getElementById('pauseScreen').classList.add('hidden');
  },
  showComplete(score, time) {
    this.hideHud();
    document.getElementById('completeScreen').classList.remove('hidden');
    document.getElementById('completeStats').innerHTML = `
      <div class="stat"><div class="stat-label">Очки</div><div class="stat-value">${score}</div></div>
      <div class="stat"><div class="stat-label">Время</div><div class="stat-value">${time}с</div></div>
      <div class="stat"><div class="stat-label">Уровень</div><div class="stat-value">${Game.level}</div></div>
    `;
  },
  showGameOver(score, level) {
    this.hideHud();
    document.getElementById('gameOverScreen').classList.remove('hidden');
    document.getElementById('gameOverStats').innerHTML = `
      <div class="stat"><div class="stat-label">Очки</div><div class="stat-value">${score}</div></div>
      <div class="stat"><div class="stat-label">Уровень</div><div class="stat-value">${level}</div></div>
      <div class="stat"><div class="stat-label">Рекорд</div><div class="stat-value">${Save.data.highScore}</div></div>
    `;
  }
};

// ============================================================
// RENDER
// ============================================================
function drawBackground(cam) {
  // Sky gradient
  const grad = ctx.createLinearGradient(0, 0, 0, H);
  grad.addColorStop(0, '#ffe8f5');
  grad.addColorStop(0.5, '#fff0e6');
  grad.addColorStop(1, '#f0e8ff');
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, W, H);

  // Parallax clouds
  ctx.fillStyle = 'rgba(255,255,255,0.7)';
  for (let i = 0; i < 6; i++) {
    const cx = ((i * 300 - cam.x * 0.2) % (W + 300) + W + 300) % (W + 300) - 150;
    const cy = 50 + (i % 3) * 40;
    ctx.beginPath();
    ctx.arc(cx, cy, 20, 0, Math.PI * 2);
    ctx.arc(cx + 20, cy, 25, 0, Math.PI * 2);
    ctx.arc(cx + 45, cy, 20, 0, Math.PI * 2);
    ctx.fill();
  }

  // Far hills
  ctx.fillStyle = 'rgba(181,107,255,0.15)';
  ctx.beginPath();
  ctx.moveTo(0, H);
  for (let x = 0; x <= W; x += 40) {
    const y = H - 150 + Math.sin((x + cam.x * 0.3) * 0.01) * 30;
    ctx.lineTo(x, y);
  }
  ctx.lineTo(W, H);
  ctx.closePath();
  ctx.fill();

  // Near hills
  ctx.fillStyle = 'rgba(255,107,157,0.15)';
  ctx.beginPath();
  ctx.moveTo(0, H);
  for (let x = 0; x <= W; x += 30) {
    const y = H - 80 + Math.sin((x + cam.x * 0.5) * 0.015) * 20;
    ctx.lineTo(x, y);
  }
  ctx.lineTo(W, H);
  ctx.closePath();
  ctx.fill();
}

function drawPlatform(p, cam) {
  if (!p.visible) return;
  const x = p.x - cam.x;
  const y = p.y - cam.y;
  if (x + p.width < 0 || x > W) return;

  ctx.save();

  // Disappearing flicker
  if (p.type === 'disappearing' && p.disappearTimer > 0) {
    ctx.globalAlpha = 0.3 + Math.sin(p.disappearTimer * 20) * 0.3;
  }

  // Shadow
  ctx.fillStyle = 'rgba(181,107,255,0.2)';
  ctx.beginPath();
  ctx.roundRect(x + 4, y + 6, p.width, p.height, 12);
  ctx.fill();

  // Body
  const grad = ctx.createLinearGradient(x, y, x, y + p.height);
  let c1 = p.color, c2 = p.color;
  if (p.type === 'finish') { c1 = '#a8ffc5'; c2 = '#6bff9d'; }
  else if (p.type === 'start') { c1 = '#d9b3ff'; c2 = '#b56bff'; }
  else if (p.type === 'moving') { c1 = '#ffc59d'; c2 = '#ff9d6b'; }
  else if (p.type === 'disappearing') { c1 = '#ffeab3'; c2 = '#ffdb6b'; }
  else {
    c2 = p.color;
    c1 = lighten(p.color, 0.3);
  }
  grad.addColorStop(0, c1);
  grad.addColorStop(1, c2);
  ctx.fillStyle = grad;
  ctx.beginPath();
  ctx.roundRect(x, y, p.width, p.height, 12);
  ctx.fill();

  // Highlight
  ctx.fillStyle = 'rgba(255,255,255,0.4)';
  ctx.beginPath();
  ctx.roundRect(x + 6, y + 3, p.width - 12, 4, 2);
  ctx.fill();

  // Finish flag
  if (p.type === 'finish') {
    const fx = x + p.width / 2;
    const fy = y - 50;
    ctx.strokeStyle = '#3d2848';
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.moveTo(fx, y);
    ctx.lineTo(fx, fy);
    ctx.stroke();
    ctx.fillStyle = '#ff6b9d';
    ctx.beginPath();
    ctx.moveTo(fx, fy);
    ctx.lineTo(fx + 25, fy + 8);
    ctx.lineTo(fx, fy + 16);
    ctx.closePath();
    ctx.fill();
  }

  ctx.restore();
}

function lighten(hex, amt) {
  const r = parseInt(hex.slice(1,3), 16);
  const g = parseInt(hex.slice(3,5), 16);
  const b = parseInt(hex.slice(5,7), 16);
  const nr = Math.min(255, Math.floor(r + (255 - r) * amt));
  const ng = Math.min(255, Math.floor(g + (255 - g) * amt));
  const nb = Math.min(255, Math.floor(b + (255 - b) * amt));
  return '#' + [nr,ng,nb].map(v => v.toString(16).padStart(2,'0')).join('');
}

function drawTrap(t, cam) {
  const x = t.x - cam.x;
  const y = t.y - cam.y;
  if (x + t.width < 0 || x > W) return;
  ctx.fillStyle = '#ff6b6b';
  const spikes = 3;
  const sw = t.width / spikes;
  for (let i = 0; i < spikes; i++) {
    ctx.beginPath();
    ctx.moveTo(x + i * sw, y + t.height);
    ctx.lineTo(x + i * sw + sw/2, y);
    ctx.lineTo(x + (i+1) * sw, y + t.height);
    ctx.closePath();
    ctx.fill();
  }
}

function drawItem(it, cam) {
  if (it.collected) return;
  const bob = Math.sin(Game.time * 3 + it.bobPhase) * 4;
  const x = it.x - cam.x;
  const y = it.y - cam.y + bob;
  if (x < -20 || x > W + 20) return;

  ctx.save();
  if (it.type === 'candy') {
    const grad = ctx.createRadialGradient(x - 2, y - 2, 0, x, y, 8);
    grad.addColorStop(0, '#fff');
    grad.addColorStop(0.3, '#ffdb6b');
    grad.addColorStop(1, '#ff9d6b');
    ctx.fillStyle = grad;
    ctx.beginPath();
    ctx.arc(x, y, 7, 0, Math.PI * 2);
    ctx.fill();
  } else if (it.type === 'golden') {
    const grad = ctx.createRadialGradient(x - 3, y - 3, 0, x, y, 10);
    grad.addColorStop(0, '#fff');
    grad.addColorStop(0.4, '#ffdb6b');
    grad.addColorStop(1, '#ff9d6b');
    ctx.fillStyle = grad;
    ctx.beginPath();
    // Star
    for (let i = 0; i < 10; i++) {
      const r = i % 2 === 0 ? 10 : 5;
      const a = (i / 10) * Math.PI * 2 - Math.PI / 2;
      const px = x + Math.cos(a) * r;
      const py = y + Math.sin(a) * r;
      if (i === 0) ctx.moveTo(px, py);
      else ctx.lineTo(px, py);
    }
    ctx.closePath();
    ctx.fill();
  } else if (it.type === 'heart') {
    ctx.fillStyle = '#ff6b9d';
    ctx.beginPath();
    ctx.arc(x - 4, y - 2, 5, 0, Math.PI * 2);
    ctx.arc(x + 4, y - 2, 5, 0, Math.PI * 2);
    ctx.moveTo(x - 9, y);
    ctx.lineTo(x, y + 9);
    ctx.lineTo(x + 9, y);
    ctx.closePath();
    ctx.fill();
  } else if (it.type === 'shield') {
    ctx.fillStyle = '#6bb5ff';
    ctx.beginPath();
    ctx.moveTo(x, y - 9);
    ctx.lineTo(x + 8, y - 4);
    ctx.lineTo(x + 8, y + 3);
    ctx.lineTo(x, y + 9);
    ctx.lineTo(x - 8, y + 3);
    ctx.lineTo(x - 8, y - 4);
    ctx.closePath();
    ctx.fill();
    ctx.fillStyle = 'rgba(255,255,255,0.5)';
    ctx.beginPath();
    ctx.arc(x - 2, y - 3, 2, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.restore();
}

function render() {
  ctx.clearRect(0, 0, W, H);

  if (Game.state === 'PLAYING' || Game.state === 'PAUSED') {
    const cam = Camera;
    drawBackground(cam);

    // Platforms
    for (const p of Game.levelData.platforms) drawPlatform(p, cam);
    // Traps
    for (const t of Game.levelData.traps) drawTrap(t, cam);
    // Items
    for (const it of Game.levelData.items) drawItem(it, cam);
    // Enemies
    for (const e of Game.levelData.enemies) drawEnemy(ctx, e, cam);
    // Player
    Player.draw(ctx, cam);
    // Particles
    Particles.draw(ctx, cam);

    // Combo indicator
    if (Game.combo > 1) {
      ctx.save();
      ctx.globalAlpha = Math.min(1, Game.comboTimer);
      ctx.fillStyle = '#ff6b9d';
      ctx.font = 'bold 24px system-ui';
      ctx.textAlign = 'center';
      ctx.fillText('x' + Game.combo + ' COMBO!', Player.x + Player.width/2 - cam.x, Player.y - 30 - cam.y);
      ctx.restore();
    }
  } else if (Game.state === 'MENU') {
    drawMenuBackground();
  }
}

function drawMenuBackground() {
  const grad = ctx.createLinearGradient(0, 0, 0, H);
  grad.addColorStop(0, '#ffe8f5');
  grad.addColorStop(1, '#f0e8ff');
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, W, H);

  // Floating candies
  const t = Date.now() * 0.001;
  for (let i = 0; i < 15; i++) {
    const x = (i * 137 + t * 20) % W;
    const y = (i * 89 + Math.sin(t + i) * 30) % H;
    const colors = ['#ff6b9d', '#b56bff', '#ff9d6b', '#6bff9d', '#ffdb6b'];
    ctx.fillStyle = colors[i % colors.length];
    ctx.globalAlpha = 0.3;
    ctx.beginPath();
    ctx.arc(x, y, 8 + (i % 3) * 4, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;
}

// ============================================================
// MAIN LOOP
// ============================================================
let lastTime = performance.now();
function loop(now) {
  const dt = Math.min(0.05, (now - lastTime) / 1000);
  lastTime = now;

  Input.update();

  // Global controls
  if (Input.isJust('Escape')) {
    if (Game.state === 'PLAYING') Game.pause();
    else if (Game.state === 'PAUSED') Game.resume();
  }
  if (Input.isJust('KeyR') && (Game.state === 'PLAYING' || Game.state === 'PAUSED')) {
    Game.start(Game.level);
  }

  if (Game.state === 'PLAYING') {
    Game.update(dt);
  }

  render();
  requestAnimationFrame(loop);
}

// ============================================================
// BUTTON HANDLERS
// ============================================================
document.getElementById('playBtn').onclick = () => {
  Audio.resume();
  Audio.init();
  document.getElementById('menuScreen').classList.add('hidden');
  Game.start(1);
};
document.getElementById('continueBtn').onclick = () => {
  Audio.resume();
  document.getElementById('menuScreen').classList.add('hidden');
  Game.start(Save.data.level);
};
document.getElementById('pauseBtn').onclick = () => Game.pause();
document.getElementById('resumeBtn').onclick = () => Game.resume();
document.getElementById('restartBtn').onclick = () => {
  document.getElementById('pauseScreen').classList.add('hidden');
  Game.start(Game.level);
};
document.getElementById('menuBtn').onclick = () => {
  document.getElementById('pauseScreen').classList.add('hidden');
  Game.state = 'MENU';
  Audio.stopMusic();
  UI.showMenu();
};
document.getElementById('nextBtn').onclick = () => {
  document.getElementById('completeScreen').classList.add('hidden');
  Game.start(Game.level + 1);
};
document.getElementById('completeMenuBtn').onclick = () => {
  document.getElementById('completeScreen').classList.add('hidden');
  Game.state = 'MENU';
  UI.showMenu();
};
document.getElementById('retryBtn').onclick = () => {
  document.getElementById('gameOverScreen').classList.add('hidden');
  Game.start(Game.level);
};
document.getElementById('gameOverMenuBtn').onclick = () => {
  document.getElementById('gameOverScreen').classList.add('hidden');
  Game.state = 'MENU';
  UI.showMenu();
};

// Pause on blur
window.addEventListener('blur', () => {
  if (Game.state === 'PLAYING') Game.pause();
});

// roundRect polyfill
if (!CanvasRenderingContext2D.prototype.roundRect) {
  CanvasRenderingContext2D.prototype.roundRect = function(x, y, w, h, r) {
    if (typeof r === 'number') r = [r, r, r, r];
    this.beginPath();
    this.moveTo(x + r[0], y);
    this.arcTo(x + w, y, x + w, y + h, r[1]);
    this.arcTo(x + w, y + h, x, y + h, r[2]);
    this.arcTo(x, y + h, x, y, r[3]);
    this.arcTo(x, y, x + w, y, r[0]);
    this.closePath();
    return this;
  };
}

// Start
UI.showMenu();
requestAnimationFrame(loop);
</script>
</body>
</html>
