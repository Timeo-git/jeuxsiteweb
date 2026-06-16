<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Le Serpent du Verger</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet">
<style>
  :root{
    --bg-wood:#caa468;
    --bg-wood-dark:#9c7c4d;
    --cabinet:#2e4a3f;
    --cabinet-dark:#1f352c;
    --screen:#16241e;
    --cream:#f4ead6;
    --gold:#e8b34c;
    --snake:#5fae71;
    --apple:#d1495b;
    --bonus:#f2c14e;
    --obstacle:#6b5b4b;
    --shadow: rgba(0,0,0,0.35);
  }

  *{ box-sizing:border-box; }

  body{
    margin:0;
    min-height:100vh;
    display:flex;
    align-items:center;
    justify-content:center;
    background: linear-gradient(180deg, var(--bg-wood) 0%, var(--bg-wood-dark) 100%);
    font-family:'Space Mono', monospace;
    color:var(--cream);
    padding:24px;
  }

  .cabinet{
    background: linear-gradient(145deg, var(--cabinet), var(--cabinet-dark));
    border-radius:24px;
    padding:28px 28px 36px;
    box-shadow: 0 20px 40px var(--shadow), inset 0 0 0 4px rgba(0,0,0,0.15);
    max-width:480px;
    width:100%;
  }

  .marquee{
    text-align:center;
    font-family:'Press Start 2P', monospace;
    font-size:15px;
    color:var(--gold);
    letter-spacing:1px;
    margin-bottom:6px;
    text-shadow: 0 2px 0 rgba(0,0,0,0.3);
  }

  .subtitle{
    text-align:center;
    font-size:11px;
    color:var(--cream);
    opacity:0.65;
    margin-bottom:18px;
    line-height:1.6;
  }

  .screen-frame{
    background: var(--screen);
    border-radius:10px;
    padding:10px;
    box-shadow: inset 0 0 18px rgba(0,0,0,0.6), 0 0 0 6px var(--cabinet-dark);
    position:relative;
    transition: box-shadow 0.15s ease;
  }
  .screen-frame.flash-gold{
    box-shadow: inset 0 0 18px rgba(242,193,78,0.6), 0 0 0 6px var(--cabinet-dark);
  }
  .screen-frame.flash-blue{
    box-shadow: inset 0 0 18px rgba(123,178,213,0.6), 0 0 0 6px var(--cabinet-dark);
  }

  canvas{
    display:block;
    width:100%;
    background: var(--screen);
    border-radius:4px;
    image-rendering:pixelated;
    touch-action:none;
  }

  .scanlines{
    position:absolute;
    inset:10px;
    border-radius:4px;
    pointer-events:none;
    background: repeating-linear-gradient(
      to bottom,
      rgba(0,0,0,0) 0px,
      rgba(0,0,0,0) 2px,
      rgba(0,0,0,0.08) 3px
    );
  }

  .hud{
    display:flex;
    justify-content:space-between;
    align-items:center;
    margin:18px 4px 14px;
    font-size:12px;
  }
  .hud .stat{ text-align:center; }
  .hud .stat:first-child{ text-align:left; }
  .hud .stat:last-child{ text-align:right; }

  .hud .label{ opacity:0.6; font-size:10px; display:block; margin-bottom:2px; }
  .hud .value{ color:var(--gold); font-size:16px; font-weight:bold; }
  .hud .value.bonus-active{ color:var(--bonus); }

  .message{
    text-align:center;
    font-size:11px;
    line-height:1.7;
    opacity:0.85;
    min-height:32px;
  }

  .controls{
    display:flex;
    justify-content:center;
    gap:10px;
    margin-top:14px;
  }

  button{
    font-family:'Space Mono', monospace;
    font-weight:bold;
    font-size:12px;
    padding:10px 18px;
    border-radius:8px;
    border:none;
    background:var(--gold);
    color:var(--cabinet-dark);
    cursor:pointer;
    box-shadow: 0 4px 0 rgba(0,0,0,0.25);
    transition: transform 0.05s ease;
  }
  button:active{ transform: translateY(3px); box-shadow:none; }
  button.secondary{
    background: rgba(244,234,214,0.15);
    color:var(--cream);
  }

  .dpad{
    display:grid;
    grid-template-columns:48px 48px 48px;
    grid-template-rows:42px 42px 42px;
    gap:6px;
    justify-content:center;
    margin-top:16px;
  }
  .dpad button{
    padding:0;
    font-size:16px;
    background: rgba(244,234,214,0.12);
    color:var(--cream);
    box-shadow:none;
  }
  .dpad .up{ grid-column:2; grid-row:1; }
  .dpad .left{ grid-column:1; grid-row:2; }
  .dpad .down{ grid-column:2; grid-row:2; }
  .dpad .right{ grid-column:3; grid-row:2; }

  @media (hover:hover){ .dpad{ display:none; } }
</style>
</head>
<body>

<div class="cabinet">
  <div class="marquee">LE SERPENT DU VERGER</div>
  <div class="subtitle">Flèches / WASD pour bouger &middot; Espace pour pause<br>Des obstacles apparaissent en montant de niveau</div>

  <div class="screen-frame" id="screenFrame">
    <canvas id="game" width="320" height="320"></canvas>
    <div class="scanlines"></div>
  </div>

  <div class="hud">
    <div class="stat">
      <span class="label">SCORE</span>
      <span class="value" id="score">0</span>
    </div>
    <div class="stat">
      <span class="label">NIVEAU</span>
      <span class="value" id="level">1</span>
    </div>
    <div class="stat">
      <span class="label">MEILLEUR</span>
      <span class="value" id="best">0</span>
    </div>
  </div>

  <div class="message" id="message">Appuie sur une flèche pour commencer</div>

  <div class="controls">
    <button class="secondary" id="pause">Pause</button>
    <button id="restart">Recommencer</button>
  </div>

  <div class="dpad">
    <button class="up" data-dir="up">▲</button>
    <button class="left" data-dir="left">◀</button>
    <button class="down" data-dir="down">▼</button>
    <button class="right" data-dir="right">▶</button>
  </div>
</div>

<script>
// ---- Configuration de base ----
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const screenFrame = document.getElementById('screenFrame');
const cellSize = 16;
const gridCount = canvas.width / cellSize;

const scoreEl = document.getElementById('score');
const levelEl = document.getElementById('level');
const bestEl = document.getElementById('best');
const messageEl = document.getElementById('message');
const restartBtn = document.getElementById('restart');
const pauseBtn = document.getElementById('pause');

const BONUS_EVERY = 4;            // une pomme dorée tous les 4 fruits normaux
const BONUS_SCORE = 3;
const BONUS_DURATION_MS = 6000;

const LEVEL_UP_EVERY = 5;         // un niveau de plus tous les 5 fruits normaux
const OBSTACLES_PER_LEVEL = 2;    // nombre d'obstacles ajoutés à chaque niveau
const OBSTACLE_MIN_DIST_FROM_HEAD = 4; // distance minimale (en cases) pour rester juste

function localStorageGet(key){
  try { return localStorage.getItem(key); } catch(e){ return null; }
}
function localStorageSet(key, value){
  try { localStorage.setItem(key, value); } catch(e){ /* ignore */ }
}

let best = Number(localStorageGet('snakeBest')) || 0;
bestEl.textContent = best;

// ---- Son (Web Audio, sans fichier externe) ----
let audioCtx = null;
function getAudioCtx(){
  if(!audioCtx){
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  return audioCtx;
}
function playTone(freq, duration, type, volume){
  try{
    const ac = getAudioCtx();
    const osc = ac.createOscillator();
    const gain = ac.createGain();
    osc.type = type || 'square';
    osc.frequency.value = freq;
    gain.gain.value = volume || 0.04;
    osc.connect(gain);
    gain.connect(ac.destination);
    osc.start();
    osc.stop(ac.currentTime + duration);
  } catch(e){ /* audio indisponible, on ignore */ }
}
function soundEat(){ playTone(440, 0.07, 'square', 0.04); }
function soundBonus(){
  playTone(660, 0.06, 'square', 0.05);
  setTimeout(() => playTone(880, 0.09, 'square', 0.05), 70);
}
function soundLevelUp(){
  playTone(523, 0.08, 'triangle', 0.05);
  setTimeout(() => playTone(659, 0.08, 'triangle', 0.05), 90);
  setTimeout(() => playTone(784, 0.12, 'triangle', 0.05), 180);
}
function soundGameOver(){ playTone(160, 0.35, 'sawtooth', 0.05); }

// ---- État du jeu ----
let snake, direction, nextDirection, food, bonusFood, bonusTimeoutId;
let obstacles, level;
let score, foodsEaten, speed, loopId;
let started, running, paused;
let messageTimeoutId;

function resetGame(){
  snake = [
    {x: 7, y: 8},
    {x: 6, y: 8},
    {x: 5, y: 8}
  ];
  direction = 'right';
  nextDirection = 'right';
  score = 0;
  foodsEaten = 0;
  level = 1;
  obstacles = [];
  speed = 130;
  started = false;
  running = false;
  paused = false;
  clearInterval(loopId);
  clearTimeout(bonusTimeoutId);
  clearTimeout(messageTimeoutId);
  bonusFood = null;
  scoreEl.textContent = score;
  levelEl.textContent = level;
  scoreEl.classList.remove('bonus-active');
  pauseBtn.textContent = 'Pause';
  placeFood();
  messageEl.textContent = 'Appuie sur une flèche pour commencer';
  draw();
}

function manhattan(a, b){
  return Math.abs(a.x - b.x) + Math.abs(a.y - b.y);
}

function randomFreeCell(opts){
  opts = opts || {};
  const minDistFromHead = opts.minDistFromHead || 0;
  let cell, valid = false, attempts = 0;
  while(!valid && attempts < 300){
    attempts++;
    cell = {
      x: Math.floor(Math.random() * gridCount),
      y: Math.floor(Math.random() * gridCount)
    };
    const onSnake = snake.some(seg => seg.x === cell.x && seg.y === cell.y);
    const onFood = food && food.x === cell.x && food.y === cell.y;
    const onBonus = bonusFood && bonusFood.x === cell.x && bonusFood.y === cell.y;
    const onObstacle = obstacles.some(o => o.x === cell.x && o.y === cell.y);
    const tooCloseToHead = minDistFromHead > 0 && manhattan(cell, snake[0]) < minDistFromHead;
    valid = !onSnake && !onFood && !onBonus && !onObstacle && !tooCloseToHead;
  }
  return cell;
}

function placeFood(){
  food = randomFreeCell();
}

function maybeSpawnBonus(){
  if(foodsEaten > 0 && foodsEaten % BONUS_EVERY === 0 && !bonusFood){
    bonusFood = randomFreeCell();
    scoreEl.classList.add('bonus-active');
    bonusTimeoutId = setTimeout(() => {
      bonusFood = null;
      scoreEl.classList.remove('bonus-active');
    }, BONUS_DURATION_MS);
  }
}

function clearBonus(){
  bonusFood = null;
  clearTimeout(bonusTimeoutId);
  scoreEl.classList.remove('bonus-active');
}

function maybeLevelUp(){
  if(foodsEaten > 0 && foodsEaten % LEVEL_UP_EVERY === 0){
    level += 1;
    levelEl.textContent = level;
    for(let i = 0; i < OBSTACLES_PER_LEVEL; i++){
      obstacles.push(randomFreeCell({minDistFromHead: OBSTACLE_MIN_DIST_FROM_HEAD}));
    }
    soundLevelUp();
    flashScreen('flash-blue');
    showTempMessage('Niveau ' + level + ' ! Nouveaux obstacles', 1400);
  }
}

function showTempMessage(text, duration){
  messageEl.textContent = text;
  clearTimeout(messageTimeoutId);
  messageTimeoutId = setTimeout(() => {
    if(running) messageEl.textContent = '';
  }, duration);
}

// ---- Boucle de jeu ----
function tick(){
  direction = nextDirection;
  const head = {...snake[0]};

  if(direction === 'up') head.y -= 1;
  if(direction === 'down') head.y += 1;
  if(direction === 'left') head.x -= 1;
  if(direction === 'right') head.x += 1;

  if(head.x < 0 || head.x >= gridCount || head.y < 0 || head.y >= gridCount){
    return gameOver();
  }
  if(snake.some(seg => seg.x === head.x && seg.y === head.y)){
    return gameOver();
  }
  if(obstacles.some(o => o.x === head.x && o.y === head.y)){
    return gameOver();
  }

  snake.unshift(head);

  if(head.x === food.x && head.y === food.y){
    score += 1;
    foodsEaten += 1;
    scoreEl.textContent = score;
    soundEat();
    placeFood();
    maybeSpawnBonus();
    maybeLevelUp();
    if(speed > 60) speed -= 3;
    restartLoop();
  } else if(bonusFood && head.x === bonusFood.x && head.y === bonusFood.y){
    score += BONUS_SCORE;
    scoreEl.textContent = score;
    soundBonus();
    clearBonus();
    flashScreen('flash-gold');
  } else {
    snake.pop();
  }

  draw();
}

function flashScreen(className){
  screenFrame.classList.add(className);
  setTimeout(() => screenFrame.classList.remove(className), 200);
}

function restartLoop(){
  clearInterval(loopId);
  loopId = setInterval(tick, speed);
}

function gameOver(){
  running = false;
  clearInterval(loopId);
  clearBonus();
  clearTimeout(messageTimeoutId);
  soundGameOver();
  if(score > best){
    best = score;
    bestEl.textContent = best;
    localStorageSet('snakeBest', best);
    messageEl.textContent = 'Nouveau record : ' + score + ' 🎉';
  } else {
    messageEl.textContent = 'Perdu ! Score : ' + score + ' — recommence';
  }
}

// ---- Dessin ----
function draw(){
  ctx.fillStyle = '#16241e';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = '#6b5b4b';
  obstacles.forEach(o => roundedCell(o.x, o.y, 2));

  ctx.fillStyle = '#d1495b';
  roundedCell(food.x, food.y, 4);

  if(bonusFood){
    ctx.fillStyle = '#f2c14e';
    roundedCell(bonusFood.x, bonusFood.y, 5);
  }

  snake.forEach((seg, i) => {
    ctx.fillStyle = i === 0 ? '#7cc98d' : '#5fae71';
    roundedCell(seg.x, seg.y, 3);
  });
}

function roundedCell(gx, gy, r){
  const x = gx * cellSize + 1;
  const y = gy * cellSize + 1;
  const size = cellSize - 2;
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.arcTo(x + size, y, x + size, y + size, r);
  ctx.arcTo(x + size, y + size, x, y + size, r);
  ctx.arcTo(x, y + size, x, y, r);
  ctx.arcTo(x, y, x + size, y, r);
  ctx.closePath();
  ctx.fill();
}

// ---- Contrôles ----
function setDirection(dir){
  if(paused) return;
  const opposites = {up:'down', down:'up', left:'right', right:'left'};
  if(opposites[dir] === direction) return;
  nextDirection = dir;

  if(!started){
    started = true;
    running = true;
    messageEl.textContent = '';
    restartLoop();
  }
}

function togglePause(){
  if(!started || !running && !paused) return;
  if(paused){
    paused = false;
    running = true;
    pauseBtn.textContent = 'Pause';
    messageEl.textContent = '';
    restartLoop();
  } else {
    paused = true;
    running = false;
    clearInterval(loopId);
    pauseBtn.textContent = 'Reprendre';
    messageEl.textContent = 'Pause — appuie sur Espace';
  }
}

window.addEventListener('keydown', (e) => {
  const map = {
    ArrowUp:'up', ArrowDown:'down', ArrowLeft:'left', ArrowRight:'right',
    w:'up', s:'down', a:'left', d:'right',
    W:'up', S:'down', A:'left', D:'right'
  };
  if(map[e.key]){
    e.preventDefault();
    setDirection(map[e.key]);
  }
  if(e.code === 'Space'){
    e.preventDefault();
    togglePause();
  }
});

document.querySelectorAll('.dpad button').forEach(btn => {
  btn.addEventListener('click', () => setDirection(btn.dataset.dir));
});

restartBtn.addEventListener('click', resetGame);
pauseBtn.addEventListener('click', togglePause);

// ---- Glissement tactile (mobile) ----
let touchStartX = 0, touchStartY = 0;
canvas.addEventListener('touchstart', (e) => {
  const t = e.changedTouches[0];
  touchStartX = t.clientX;
  touchStartY = t.clientY;
}, {passive:true});

canvas.addEventListener('touchend', (e) => {
  const t = e.changedTouches[0];
  const dx = t.clientX - touchStartX;
  const dy = t.clientY - touchStartY;
  const minSwipe = 20;
  if(Math.max(Math.abs(dx), Math.abs(dy)) < minSwipe) return;
  if(Math.abs(dx) > Math.abs(dy)){
    setDirection(dx > 0 ? 'right' : 'left');
  } else {
    setDirection(dy > 0 ? 'down' : 'up');
  }
}, {passive:true});

// ---- Démarrage ----
resetGame();
</script>

</body>
</html>
