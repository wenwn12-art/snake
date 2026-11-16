[index.html](https://github.com/user-attachments/files/34902682/index.html)
# snake
贪吃蛇
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="theme-color" content="#10b981">
<meta name="description" content="经典贪吃蛇游戏 - 方向键控制，吃食物变长，撞墙或撞到自己即结束。">
<title>贪吃蛇 · Snake Game</title>
<style>
  /* ============ 基础重置 & 变量 ============ */
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --bg: #f8fafc; --bg-2: #eef2f7;
    --card: #ffffff; --card-fg: #0f172a;
    --fg: #0f172a; --muted: #64748b; --muted-2: #94a3b8;
    --border: #e2e8f0; --border-2: #cbd5e1;
    --primary: #10b981; --primary-d: #059669; --primary-l: #d1fae5;
    --rose: #e11d48; --rose-l: #fecdd3; --rose-bg: #fff1f2;
    --amber: #f59e0b; --amber-l: #fde68a; --amber-bg: #fffbeb;
    --slate: #475569; --slate-l: #e2e8f0;
    --radius: 14px; --radius-sm: 10px;
    --shadow: 0 4px 16px rgba(15,23,42,.06), 0 1px 3px rgba(15,23,42,.04);
    --shadow-lg: 0 10px 30px rgba(15,23,42,.10);
    --font: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Hiragino Sans GB",
            "Microsoft YaHei", "Helvetica Neue", Helvetica, Arial, sans-serif;
  }
  html, body { height: 100%; }
  body {
    font-family: var(--font);
    background: linear-gradient(to bottom, var(--bg), var(--bg-2));
    color: var(--fg);
    line-height: 1.5;
    -webkit-font-smoothing: antialiased;
    -webkit-tap-highlight-color: transparent;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  /* ============ 布局 ============ */
  .header {
    position: sticky; top: 0; z-index: 10;
    background: rgba(255,255,255,.8);
    backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px);
    border-bottom: 1px solid rgba(226,232,240,.6);
  }
  .header-inner {
    max-width: 1080px; margin: 0 auto; padding: 12px 16px;
    display: flex; align-items: center; justify-content: space-between;
  }
  .brand { display: flex; align-items: center; gap: 12px; }
  .logo {
    width: 36px; height: 36px; border-radius: 11px;
    background: linear-gradient(135deg, #34d399, #059669);
    display: flex; align-items: center; justify-content: center;
    color: #fff; font-weight: 700; font-size: 18px;
    box-shadow: 0 2px 8px rgba(16,185,129,.35);
  }
  .brand h1 { font-size: 17px; font-weight: 600; letter-spacing: -.01em; }
  .brand p { font-size: 11px; color: var(--muted); margin-top: 1px; }

  .badge {
    display: inline-flex; align-items: center; gap: 6px;
    padding: 5px 10px; border-radius: 999px; font-size: 12px; font-weight: 500;
    border: 1px solid var(--border); background: #f1f5f9; color: var(--muted);
  }
  .badge .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--muted-2); position: relative; }
  .badge.playing { background: #ecfdf5; color: #047857; border-color: rgba(16,185,129,.3); }
  .badge.playing .dot { background: var(--primary); }
  .badge.playing .dot::after {
    content: ""; position: absolute; inset: 0; border-radius: 50%; background: var(--primary);
    animation: ping 1.4s cubic-bezier(0,0,.2,1) infinite;
  }
  .badge.paused { background: #fffbeb; color: #b45309; border-color: rgba(245,158,11,.3); }
  .badge.paused .dot { background: var(--amber); }
  .badge.over { background: var(--rose-bg); color: #be123c; border-color: rgba(225,29,72,.3); }
  .badge.over .dot { background: var(--rose); }
  @keyframes ping { 75%,100% { transform: scale(2.2); opacity: 0; } }

  .main { flex: 1; width: 100%; }
  .container { max-width: 1080px; margin: 0 auto; padding: 24px 16px; }
  .grid {
    display: grid; gap: 24px;
    grid-template-columns: 1fr;
  }
  @media (min-width: 880px) {
    .grid { grid-template-columns: 1fr 320px; align-items: start; }
  }

  /* ============ 卡片 ============ */
  .card {
    background: var(--card); border: 1px solid rgba(226,232,240,.8);
    border-radius: var(--radius); box-shadow: var(--shadow);
  }
  .card-header { padding: 16px 20px 8px; display: flex; align-items: center; justify-content: space-between; }
  .card-title { font-size: 15px; font-weight: 600; }
  .card-meta { font-size: 12px; color: var(--muted); font-variant-numeric: tabular-nums; }
  .card-body { padding: 0 20px 20px; }

  /* ============ 游戏区 ============ */
  .board-wrap {
    display: flex; flex-direction: column; align-items: center;
  }
  .board {
    position: relative; border-radius: 16px; overflow: hidden;
    box-shadow: inset 0 1px 3px rgba(15,23,42,.08);
    border: 1px solid rgba(15,23,42,.08);
    width: 100%; max-width: 480px; aspect-ratio: 1 / 1;
    touch-action: none; /* 阻止滑动滚动 */
  }
  canvas { display: block; width: 100%; height: 100%; }

  /* 遮罩层 */
  .overlay {
    position: absolute; inset: 0;
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    gap: 12px; padding: 24px; text-align: center;
    background: rgba(15,23,42,.62); backdrop-filter: blur(6px); -webkit-backdrop-filter: blur(6px);
    animation: fadeIn .25s ease;
  }
  .overlay.hidden { display: none; }
  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
  .ov-icon {
    width: 48px; height: 48px; border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    background: rgba(16,185,129,.2); color: #6ee7b7; margin-bottom: 2px;
  }
  .ov-icon.over { background: rgba(225,29,72,.2); color: #fda4af; }
  .ov-title { font-size: 20px; font-weight: 600; color: #fff; letter-spacing: -.01em; }
  .ov-sub { font-size: 13px; color: #cbd5e1; max-width: 260px; }
  .ov-btn {
    margin-top: 6px; border: none; cursor: pointer;
    padding: 10px 22px; border-radius: 10px; font-size: 14px; font-weight: 600;
    background: var(--primary); color: #fff; box-shadow: 0 4px 14px rgba(16,185,129,.4);
    transition: transform .1s, background .15s;
    display: inline-flex; align-items: center; gap: 6px;
  }
  .ov-btn:active { transform: scale(.97); }
  .ov-btn.over { background: var(--rose); box-shadow: 0 4px 14px rgba(225,29,72,.4); }

  /* ============ 方向键 D-pad（移动端） ============ */
  .dpad {
    margin-top: 18px; display: grid; grid-template-columns: repeat(3, 48px); gap: 8px;
    width: 160px;
  }
  .dpad .empty { visibility: hidden; }
  .dpad button {
    height: 48px; border: 1px solid var(--border); background: var(--card);
    border-radius: 10px; cursor: pointer; color: var(--fg);
    display: flex; align-items: center; justify-content: center;
    transition: background .15s, border-color .15s;
  }
  .dpad button:active { background: #f1f5f9; border-color: var(--primary); }
  .dpad svg { width: 22px; height: 22px; }
  @media (min-width: 640px) { .dpad { display: none; } }

  /* ============ 分数卡片 ============ */
  .stats { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
  .stat { background: var(--card); border: 1px solid rgba(226,232,240,.8); border-radius: var(--radius); padding: 14px; box-shadow: var(--shadow); }
  .stat-head { display: flex; align-items: center; gap: 8px; margin-bottom: 6px; }
  .stat-ic { width: 24px; height: 24px; border-radius: 7px; display: flex; align-items: center; justify-content: center; }
  .stat-ic svg { width: 14px; height: 14px; }
  .stat-ic.emerald { background: var(--primary-l); color: var(--primary-d); }
  .stat-ic.rose { background: var(--rose-l); color: var(--rose); }
  .stat-ic.amber { background: var(--amber-l); color: #b45309; }
  .stat-ic.slate { background: var(--slate-l); color: var(--slate); }
  .stat-label { font-size: 12px; color: var(--muted); }
  .stat-value { font-size: 24px; font-weight: 700; font-variant-numeric: tabular-nums; }
  .stat-value .suffix { font-size: 13px; font-weight: 400; color: var(--muted-2); }

  /* ============ 按钮 ============ */
  .controls { padding: 12px 20px 20px; }
  .btn {
    border: none; cursor: pointer; padding: 12px 16px; border-radius: 10px;
    font-size: 15px; font-weight: 600; display: inline-flex; align-items: center; justify-content: center; gap: 8px;
    transition: background .15s, transform .1s; font-family: inherit;
  }
  .btn:active { transform: scale(.98); }
  .btn svg { width: 16px; height: 16px; }
  .btn-primary { background: var(--primary); color: #fff; box-shadow: 0 2px 8px rgba(16,185,129,.3); }
  .btn-primary:hover { background: var(--primary-d); }
  .btn-outline { background: var(--card); color: var(--fg); border: 1px solid var(--border); }
  .btn-outline:hover { background: #f8fafc; }
  .btn-block { width: 100%; }
  .btn-lg { padding: 13px 16px; }
  .btn-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }

  /* ============ 说明 ============ */
  .help { background: rgba(248,250,252,.5); }
  .help-body { padding: 8px 20px 18px; font-size: 12px; color: var(--muted); }
  .help-row { display: flex; align-items: center; justify-content: space-between; padding: 10px 0; }
  .help-row + .help-row { border-top: 1px solid var(--border); }
  .help-row .v { color: var(--muted-2); font-size: 12px; }
  .keys { display: inline-flex; gap: 4px; }
  kbd {
    display: inline-flex; align-items: center; justify-content: center;
    min-width: 24px; height: 24px; padding: 0 6px;
    border-radius: 6px; background: #fff; border: 1px solid var(--border-2);
    color: var(--slate); font-size: 11px; font-weight: 500; font-family: inherit;
    box-shadow: 0 1px 2px rgba(15,23,42,.04);
  }

  .side { display: flex; flex-direction: column; gap: 16px; }

  /* ============ 页脚 ============ */
  .footer {
    margin-top: auto;
    border-top: 1px solid rgba(226,232,240,.6);
    background: rgba(255,255,255,.7); backdrop-filter: blur(8px);
  }
  .footer-inner {
    max-width: 1080px; margin: 0 auto; padding: 10px 16px;
    display: flex; align-items: center; justify-content: space-between;
    font-size: 12px; color: var(--muted);
    padding-bottom: calc(10px + env(safe-area-inset-bottom));
  }
</style>
</head>
<body>

<header class="header">
  <div class="header-inner">
    <div class="brand">
      <div class="logo">S</div>
      <div>
        <h1>贪吃蛇</h1>
        <p>Snake Game · 经典再现</p>
      </div>
    </div>
    <span class="badge" id="statusBadge"><span class="dot"></span><span id="statusText">准备开始</span></span>
  </div>
</header>

<main class="main">
  <div class="container">
    <div class="grid">

      <!-- 游戏区 -->
      <section class="card">
        <div class="card-header">
          <div class="card-title">游戏区域</div>
          <div class="card-meta">20 × 20</div>
        </div>
        <div class="card-body board-wrap">
          <div class="board" id="board">
            <canvas id="canvas" width="480" height="480"></canvas>
            <div class="overlay" id="overlay">
              <div class="ov-icon" id="ovIcon"></div>
              <div class="ov-title" id="ovTitle">准备好了吗？</div>
              <div class="ov-sub" id="ovSub">按方向键或点击开始按钮启动游戏</div>
              <button class="ov-btn" id="ovBtn">开始游戏</button>
            </div>
          </div>

          <!-- 移动端方向键 -->
          <div class="dpad" id="dpad">
            <span class="empty"></span>
            <button data-dir="UP" aria-label="上"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 19V5M5 12l7-7 7 7"/></svg></button>
            <span class="empty"></span>
            <button data-dir="LEFT" aria-label="左"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M19 12H5M12 19l-7-7 7-7"/></svg></button>
            <button data-dir="DOWN" aria-label="下"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 5v14M5 12l7 7 7-7"/></svg></button>
            <button data-dir="RIGHT" aria-label="右"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 12h14M12 5l7 7-7 7"/></svg></button>
          </div>
        </div>
      </section>

      <!-- 侧栏 -->
      <aside class="side">
        <div class="stats">
          <div class="stat">
            <div class="stat-head"><span class="stat-ic emerald"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 9H4.5a2.5 2.5 0 0 1 0-5H6"/><path d="M18 9h1.5a2.5 2.5 0 0 0 0-5H18"/><path d="M4 22h16"/><path d="M10 14.66V17c0 .55-.47.98-.97 1.21C7.85 18.75 7 20.24 7 22"/><path d="M14 14.66V17c0 .55.47.98.97 1.21C16.15 18.75 17 20.24 17 22"/><path d="M18 2H6v7a6 6 0 0 0 12 0V2Z"/></svg></span><span class="stat-label">当前分数</span></div>
            <div class="stat-value" id="score">0</div>
          </div>
          <div class="stat">
            <div class="stat-head"><span class="stat-ic rose"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 20.94c1.5 0 2.75 1.06 4 1.06 3 0 6-8 6-12.22A4.91 4.91 0 0 0 17 5c-2.22 0-4 1.44-5 2-1-.56-2.78-2-5-2a4.9 4.9 0 0 0-5 4.78C2 14 5 22 8 22c1.25 0 2.5-1.06 4-1.06Z"/><path d="M10 2c1 .5 2 2 2 5"/></svg></span><span class="stat-label">已吃食物</span></div>
            <div class="stat-value" id="foodCount">0</div>
          </div>
          <div class="stat">
            <div class="stat-head"><span class="stat-ic amber"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 9H4.5a2.5 2.5 0 0 1 0-5H6"/><path d="M18 9h1.5a2.5 2.5 0 0 0 0-5H18"/><path d="M4 22h16"/><path d="M10 14.66V17c0 .55-.47.98-.97 1.21C7.85 18.75 7 20.24 7 22"/><path d="M14 14.66V17c0 .55.47.98.97 1.21C16.15 18.75 17 20.24 17 22"/><path d="M18 2H6v7a6 6 0 0 0 12 0V2Z"/></svg></span><span class="stat-label">最高记录</span></div>
            <div class="stat-value" id="highScore">0</div>
          </div>
          <div class="stat">
            <div class="stat-head"><span class="stat-ic slate"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m12 14 4-4"/><path d="M3.34 19a10 10 0 1 1 17.32 0"/></svg></span><span class="stat-label">速度等级</span></div>
            <div class="stat-value"><span id="speedLevel">0</span><span class="suffix">/10</span></div>
          </div>
        </div>

        <!-- 控制 -->
        <section class="card">
          <div class="card-header"><div class="card-title">控制</div></div>
          <div class="controls" id="controls"></div>
        </section>

        <!-- 说明 -->
        <section class="card help">
          <div class="card-header"><div class="card-title" style="font-size:13px;">操作说明</div></div>
          <div class="help-body">
            <div class="help-row"><span>方向控制</span><span class="keys"><kbd>↑</kbd><kbd>←</kbd><kbd>↓</kbd><kbd>→</kbd></span></div>
            <div class="help-row"><span>替代按键</span><span class="keys"><kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd></span></div>
            <div class="help-row"><span>暂停 / 继续</span><kbd>Space</kbd></div>
            <div class="help-row"><span>移动端</span><span class="v">滑动屏幕或点按方向键</span></div>
            <div class="help-row"><span>规则</span><span class="v">撞墙或撞到自己即结束</span></div>
          </div>
        </section>
      </aside>

    </div>
  </div>
</main>

<footer class="footer">
  <div class="footer-inner">
    <span>贪吃蛇 · 用方向键开始你的旅程</span>
    <span>v1.0</span>
  </div>
</footer>

<script>
(function(){
  "use strict";

  // ============ 常量 ============
  var GRID = 20, CELL = 24, BOARD = GRID * CELL;
  var INIT_SPEED = 160, MIN_SPEED = 70, SPEED_STEP = 8;
  var INIT_SNAKE = [{x:10,y:10},{x:9,y:10},{x:8,y:10}];
  var INIT_DIR = "RIGHT";
  var VEC = { UP:{x:0,y:-1}, DOWN:{x:0,y:1}, LEFT:{x:-1,y:0}, RIGHT:{x:1,y:0} };
  var OPP = { UP:"DOWN", DOWN:"UP", LEFT:"RIGHT", RIGHT:"LEFT" };

  // ============ 状态 ============
  var canvas = document.getElementById("canvas");
  var ctx = canvas.getContext("2d");
  var snake, dir, queue, food, status, score, foodCount, highScore, speed, tickTimer, rafId, foodPulse;

  function loadHigh(){
    try { return parseInt(localStorage.getItem("snake-high-score")||"0",10) || 0; } catch(e){ return 0; }
  }
  function saveHigh(v){
    try { localStorage.setItem("snake-high-score", String(v)); } catch(e){}
  }

  snake = INIT_SNAKE.slice();
  dir = INIT_DIR; queue = [];
  food = randomFood(snake);
  status = "idle"; score = 0; foodCount = 0; highScore = loadHigh(); speed = INIT_SPEED;
  foodPulse = 0; tickTimer = null;

  // ============ DOM 引用 ============
  var elStatusBadge = document.getElementById("statusBadge");
  var elStatusText = document.getElementById("statusText");
  var elScore = document.getElementById("score");
  var elFoodCount = document.getElementById("foodCount");
  var elHighScore = document.getElementById("highScore");
  var elSpeedLevel = document.getElementById("speedLevel");
  var elOverlay = document.getElementById("overlay");
  var elOvIcon = document.getElementById("ovIcon");
  var elOvTitle = document.getElementById("ovTitle");
  var elOvSub = document.getElementById("ovSub");
  var elOvBtn = document.getElementById("ovBtn");
  var elControls = document.getElementById("controls");

  // SVG 图标
  var ICONS = {
    play: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="width:20px;height:20px"><polygon points="6 3 20 12 6 21 6 3"/></svg>',
    pause: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="width:16px;height:16px"><rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/></svg>',
    restart: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="width:16px;height:16px"><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/><path d="M3 3v5h5"/></svg>'
  };

  // ============ 工具函数 ============
  function randomFood(s){
    var occ = {}; s.forEach(function(p){ occ[p.x+","+p.y] = 1; });
    var free = [];
    for (var y=0; y<GRID; y++) for (var x=0; x<GRID; x++) {
      if (!occ[x+","+y]) free.push({x:x,y:y});
    }
    if (!free.length) return {x:0,y:0};
    return free[Math.floor(Math.random()*free.length)];
  }
  function clampSpeed(s){ return Math.max(MIN_SPEED, s); }
  function roundedRect(x,y,w,h,r){
    var rad = Math.min(r, w/2, h/2);
    ctx.beginPath();
    ctx.moveTo(x+rad,y); ctx.lineTo(x+w-rad,y);
    ctx.quadraticCurveTo(x+w,y,x+w,y+rad); ctx.lineTo(x+w,y+h-rad);
    ctx.quadraticCurveTo(x+w,y+h,x+w-rad,y+h); ctx.lineTo(x+rad,y+h);
    ctx.quadraticCurveTo(x,y+h,x,y+h-rad); ctx.lineTo(x,y+rad);
    ctx.quadraticCurveTo(x,y,x+rad,y); ctx.closePath();
  }

  // ============ 渲染 ============
  function draw(){
    ctx.clearRect(0,0,BOARD,BOARD);
    // 背景
    var bg = ctx.createLinearGradient(0,0,BOARD,BOARD);
    bg.addColorStop(0,"#f8fafc"); bg.addColorStop(1,"#eef2f7");
    ctx.fillStyle = bg; ctx.fillRect(0,0,BOARD,BOARD);
    // 网格
    ctx.strokeStyle = "rgba(15,23,42,0.05)"; ctx.lineWidth = 1;
    for (var i=1; i<GRID; i++){
      ctx.beginPath(); ctx.moveTo(i*CELL+0.5,0); ctx.lineTo(i*CELL+0.5,BOARD); ctx.stroke();
      ctx.beginPath(); ctx.moveTo(0,i*CELL+0.5); ctx.lineTo(BOARD,i*CELL+0.5); ctx.stroke();
    }
    // 边框
    ctx.strokeStyle = "rgba(15,23,42,0.12)"; ctx.lineWidth = 2;
    ctx.strokeRect(1,1,BOARD-2,BOARD-2);

    // 食物（脉动苹果）
    var pulse = 0.5 + 0.5*Math.sin(foodPulse);
    var fPad = 4 + pulse*1.5;
    var fx = food.x*CELL, fy = food.y*CELL, fSize = CELL - fPad*2;
    ctx.save();
    ctx.shadowColor = "rgba(244,63,94,0.55)"; ctx.shadowBlur = 12 + pulse*6;
    var fg = ctx.createLinearGradient(fx,fy,fx+CELL,fy+CELL);
    fg.addColorStop(0,"#fb7185"); fg.addColorStop(1,"#e11d48");
    ctx.fillStyle = fg; roundedRect(fx+fPad,fy+fPad,fSize,fSize,7); ctx.fill();
    ctx.restore();
    // 叶子
    ctx.fillStyle = "#16a34a";
    ctx.beginPath(); ctx.ellipse(fx+CELL/2+2, fy+fPad+1, 2.5, 1.5, -0.7, 0, Math.PI*2); ctx.fill();

    // 蛇
    for (var i=snake.length-1; i>=0; i--){
      var seg = snake[i], sx = seg.x*CELL, sy = seg.y*CELL;
      var isHead = (i===0);
      var pad = isHead ? 2 : 3, size = CELL - pad*2, radius = isHead ? 8 : 7;
      ctx.save();
      if (isHead){ ctx.shadowColor = "rgba(16,185,129,0.5)"; ctx.shadowBlur = 14; }
      var t = i / Math.max(snake.length-1, 1);
      var g = ctx.createLinearGradient(sx,sy,sx+CELL,sy+CELL);
      g.addColorStop(0, "oklch("+(0.62 - t*0.12)+" 0.15 160)");
      g.addColorStop(1, "oklch("+(0.55 + t*0.25)+" 0.13 160)");
      ctx.fillStyle = g; roundedRect(sx+pad,sy+pad,size,size,radius); ctx.fill();
      ctx.restore();
      // 眼睛
      if (isHead){
        var cx = sx+CELL/2, cy = sy+CELL/2, off = 5, e1, e2;
        if (dir==="RIGHT"){ e1={x:cx+3,y:cy-off}; e2={x:cx+3,y:cy+off}; }
        else if (dir==="LEFT"){ e1={x:cx-3,y:cy-off}; e2={x:cx-3,y:cy+off}; }
        else if (dir==="UP"){ e1={x:cx-off,y:cy-3}; e2={x:cx+off,y:cy-3}; }
        else { e1={x:cx-off,y:cy+3}; e2={x:cx+off,y:cy+3}; }
        ctx.fillStyle = "#fff";
        ctx.beginPath(); ctx.arc(e1.x,e1.y,2.6,0,Math.PI*2); ctx.arc(e2.x,e2.y,2.6,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = "#0f172a";
        ctx.beginPath(); ctx.arc(e1.x,e1.y,1.3,0,Math.PI*2); ctx.arc(e2.x,e2.y,1.3,0,Math.PI*2); ctx.fill();
      }
    }
  }

  function animate(){
    foodPulse += 0.08; draw();
    rafId = requestAnimationFrame(animate);
  }

  // ============ 游戏逻辑 ============
  function enqueueDirection(d){
    var lastDir = queue.length>0 ? queue[queue.length-1] : dir;
    if (d===lastDir || d===OPP[lastDir]) return;
    if (queue.length>=2) return;
    queue.push(d);
  }

  function step(){
    if (queue.length>0){ dir = queue.shift(); }
    var v = VEC[dir], head = snake[0];
    var nh = { x: head.x+v.x, y: head.y+v.y };
    // 撞墙
    if (nh.x<0 || nh.x>=GRID || nh.y<0 || nh.y>=GRID){ gameOver(); return; }
    // 自撞
    var willEat = (nh.x===food.x && nh.y===food.y);
    var body = willEat ? snake : snake.slice(0, -1);
    for (var i=0; i<body.length; i++){
      if (body[i].x===nh.x && body[i].y===nh.y){ gameOver(); return; }
    }
    snake.unshift(nh);
    if (willEat){
      score += 10; foodCount++;
      if (score > highScore){ highScore = score; saveHigh(highScore); }
      food = randomFood(snake);
      speed = clampSpeed(speed - SPEED_STEP);
      if (tickTimer){ clearInterval(tickTimer); tickTimer = setInterval(step, speed); }
    } else {
      snake.pop();
    }
    updateUI();
  }

  function gameOver(){
    status = "over";
    if (score > highScore){ highScore = score; saveHigh(highScore); }
    if (tickTimer){ clearInterval(tickTimer); tickTimer = null; }
    render();
  }

  function resetGame(){
    if (tickTimer){ clearInterval(tickTimer); tickTimer = null; }
    snake = INIT_SNAKE.slice();
    dir = INIT_DIR; queue = [];
    food = randomFood(snake);
    score = 0; foodCount = 0; speed = INIT_SPEED;
    updateUI();
  }

  function startGame(){ resetGame(); status = "playing"; startTick(); render(); }
  function pauseGame(){
    if (status!=="playing") return;
    status = "paused";
    if (tickTimer){ clearInterval(tickTimer); tickTimer = null; }
    render();
  }
  function resumeGame(){
    if (status!=="paused") return;
    status = "playing"; startTick(); render();
  }
  function startTick(){
    if (tickTimer) clearInterval(tickTimer);
    tickTimer = setInterval(step, speed);
  }

  // ============ UI 渲染 ============
  var STATUS_TEXT = { idle:"准备开始", playing:"游戏中", paused:"已暂停", over:"游戏结束" };

  function updateUI(){
    elScore.textContent = score;
    elFoodCount.textContent = foodCount;
    elHighScore.textContent = highScore;
    elSpeedLevel.textContent = Math.round(((INIT_SPEED - speed)/(INIT_SPEED - MIN_SPEED))*10) || 0;
  }

  function render(){
    // 状态徽章
    elStatusBadge.className = "badge " + status;
    elStatusText.textContent = STATUS_TEXT[status];

    // 遮罩层
    if (status === "playing"){
      elOverlay.classList.add("hidden");
    } else {
      elOverlay.classList.remove("hidden");
      if (status === "idle"){
        elOvIcon.className = "ov-icon"; elOvIcon.innerHTML = ICONS.play;
        elOvTitle.textContent = "准备好了吗？";
        elOvSub.textContent = "按方向键或点击开始按钮启动游戏";
        elOvBtn.textContent = ""; elOvBtn.className = "ov-btn"; elOvBtn.innerHTML = ICONS.play + "开始游戏";
      } else if (status === "paused"){
        elOvIcon.className = "ov-icon"; elOvIcon.innerHTML = ICONS.play;
        elOvTitle.textContent = "已暂停";
        elOvSub.textContent = "按空格或点击继续";
        elOvBtn.textContent = ""; elOvBtn.className = "ov-btn"; elOvBtn.innerHTML = ICONS.play + "继续";
      } else if (status === "over"){
        elOvIcon.className = "ov-icon over"; elOvIcon.innerHTML = ICONS.restart;
        elOvTitle.textContent = "游戏结束";
        elOvSub.textContent = "本局得分 " + score + " · 最高 " + highScore;
        elOvBtn.textContent = ""; elOvBtn.className = "ov-btn over"; elOvBtn.innerHTML = ICONS.restart + "重新开始";
      }
    }

    // 控制按钮
    elControls.innerHTML = "";
    if (status === "idle" || status === "over"){
      var b = document.createElement("button");
      b.className = "btn btn-primary btn-lg btn-block";
      b.innerHTML = ICONS.play + (status === "over" ? "重新开始" : "开始游戏");
      b.addEventListener("click", startGame);
      elControls.appendChild(b);
    } else {
      var row = document.createElement("div"); row.className = "btn-row";
      if (status === "playing"){
        var p = document.createElement("button");
        p.className = "btn btn-outline btn-lg"; p.innerHTML = ICONS.pause + "暂停";
        p.addEventListener("click", pauseGame); row.appendChild(p);
      } else {
        var r = document.createElement("button");
        r.className = "btn btn-primary btn-lg"; r.innerHTML = ICONS.play + "继续";
        r.addEventListener("click", resumeGame); row.appendChild(r);
      }
      var rs = document.createElement("button");
      rs.className = "btn btn-outline btn-lg"; rs.innerHTML = ICONS.restart + "重开";
      rs.addEventListener("click", startGame); row.appendChild(rs);
      elControls.appendChild(row);
    }
  }

  // 遮罩按钮事件（委托）
  elOvBtn.addEventListener("click", function(){
    if (status === "idle" || status === "over") startGame();
    else if (status === "paused") resumeGame();
  });

  // ============ 键盘 ============
  var KEYMAP = {
    ArrowUp:"UP", ArrowDown:"DOWN", ArrowLeft:"LEFT", ArrowRight:"RIGHT",
    w:"UP", s:"DOWN", a:"LEFT", d:"RIGHT", W:"UP", S:"DOWN", A:"LEFT", D:"RIGHT"
  };
  window.addEventListener("keydown", function(e){
    var k = e.key;
    if (KEYMAP[k]){
      e.preventDefault();
      if (status === "playing") enqueueDirection(KEYMAP[k]);
    } else if (k === " " || k === "Spacebar"){
      e.preventDefault();
      if (status === "playing") pauseGame();
      else if (status === "paused") resumeGame();
      else if (status === "idle" || status === "over") startGame();
    }
  });

  // ============ 触摸滑动 ============
  var touchStart = null;
  var board = document.getElementById("board");
  board.addEventListener("touchstart", function(e){
    var t = e.touches[0]; touchStart = { x:t.clientX, y:t.clientY };
  }, { passive: true });
  board.addEventListener("touchend", function(e){
    if (!touchStart) return;
    var t = e.changedTouches[0];
    var dx = t.clientX - touchStart.x, dy = t.clientY - touchStart.y;
    if (Math.abs(dx) < 20 && Math.abs(dy) < 20){ touchStart = null; return; }
    var d = Math.abs(dx) > Math.abs(dy) ? (dx>0?"RIGHT":"LEFT") : (dy>0?"DOWN":"UP");
    if (status === "playing") enqueueDirection(d);
    touchStart = null;
  }, { passive: true });

  // ============ D-pad 按钮 ============
  var dpad = document.getElementById("dpad");
  dpad.addEventListener("click", function(e){
    var btn = e.target.closest("button[data-dir]");
    if (!btn) return;
    if (status === "playing") enqueueDirection(btn.getAttribute("data-dir"));
  });

  // ============ 启动 ============
  updateUI();
  render();
  animate();

})();
</script>
</body>
</html>
