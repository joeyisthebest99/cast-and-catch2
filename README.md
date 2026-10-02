<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Fishing Game Deluxe</title>
<style>
  body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #7ec0ee;
    overflow: hidden;
  }

  #topBar {
    background: #1e1e1e;
    color: white;
    padding: 10px;
    display: flex;
    gap: 20px;
    align-items: center;
    font-size: 14px;
  }

  #topBar button {
    padding: 6px 12px;
    cursor: pointer;
  }

  #log {
    background: #222;
    color: #ddd;
    padding: 8px;
    height: 90px;
    overflow-y: auto;
    font-size: 13px;
  }

  #shop {
    position: absolute;
    right: 10px;
    top: 60px;
    background: #333;
    color: white;
    padding: 12px;
    width: 260px;
    display: none;
    border-radius: 6px;
  }

  #shop button {
    width: 100%;
    margin-top: 6px;
    padding: 6px;
    cursor: pointer;
  }

  canvas {
    background: #4fa3d1;
    display: block;
  }
</style>
</head>
<body>

<div id="topBar">
  <div><b>Money:</b> $<span id="money">0</span></div>
  <div><b>Rod:</b> <span id="rodName">Basic Rod</span></div>
  <div><b>Fish:</b> <span id="fishCount">0</span></div>

  <button id="castBtn">Cast</button>
  <button id="reelBtn" disabled>Reel</button>
  <button id="sellBtn">Sell Fish</button>
  <button id="shopBtn">Shop</button>
</div>

<div id="log"></div>

<div id="shop">
  <h3>Rod Shop</h3>
  <button data-rod="basic">Basic Rod — $0</button>
  <button data-rod="spinning">Spinning Rod — $150</button>
  <button data-rod="baitcaster">Baitcaster — $300</button>
  <button data-rod="pro">Pro Tournament Rod — $600</button>
  <button id="closeShop">Close</button>
</div>

<canvas id="gameCanvas" width="900" height="500"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const moneyEl = document.getElementById("money");
const rodNameEl = document.getElementById("rodName");
const fishCountEl = document.getElementById("fishCount");
const logEl = document.getElementById("log");

const castBtn = document.getElementById("castBtn");
const reelBtn = document.getElementById("reelBtn");
const sellBtn = document.getElementById("sellBtn");
const shopBtn = document.getElementById("shopBtn");
const shop = document.getElementById("shop");
const closeShop = document.getElementById("closeShop");

let money = 0;
let inventory = [];
let currentRod = "basic";

const rods = {
  basic: { name: "Basic Rod", bite: 0.45, strength: 1 },
  spinning: { name: "Spinning Rod", bite: 0.65, strength: 1.2 },
  baitcaster: { name: "Baitcaster", bite: 0.55, strength: 1.6 },
  pro: { name: "Pro Tournament Rod", bite: 0.8, strength: 2 }
};

const fishTypes = [
  { name: "Bluegill", base: 1, value: 5 },
  { name: "Bass", base: 2, value: 10 },
  { name: "Catfish", base: 3, value: 12 },
  { name: "Salmon", base: 4, value: 20 },
  { name: "Pike", base: 5, value: 25 }
];

let hookX = canvas.width / 2;
let hookY = 100;
let targetY = 350;

let casting = false;
let inWater = false;
let bite = false;

let reelProgress = 0;
let tension = 0;

function log(msg) {
  const div = document.createElement("div");
  div.textContent = msg;
  logEl.appendChild(div);
  logEl.scrollTop = logEl.scrollHeight;
}

function updateUI() {
  moneyEl.textContent = money;
  rodNameEl.textContent = rods[currentRod].name;
  fishCountEl.textContent = inventory.length;
}

castBtn.onclick = () => {
  if (casting || inWater) return;
  casting = true;
  bite = false;
  reelBtn.disabled = true;
  log("Casting...");
};

reelBtn.onclick = () => {
  if (!bite) return;
  reelProgress += rods[currentRod].strength * 0.3;
  tension += Math.random() * 0.4;

  if (tension > 1.4) {
    log("The line snapped! Fish escaped.");
    resetLine();
    return;
  }

  if (reelProgress >= 1) {
    catchFish();
    resetLine();
  }
};

sellBtn.onclick = () => {
  if (inventory.length === 0) {
    log("No fish to sell.");
    return;
  }
  let total = inventory.reduce((sum, f) => sum + f.value, 0);
  money += total;
  log(`Sold ${inventory.length} fish for $${total.toFixed(2)}`);
  inventory = [];
  updateUI();
};

shopBtn.onclick = () => shop.style.display = "block";
closeShop.onclick = () => shop.style.display = "none";

shop.onclick = (e) => {
  const btn = e.target.closest("button[data-rod]");
  if (!btn) return;

  const rodKey = btn.getAttribute("data-rod");
  const price = { basic: 0, spinning: 150, baitcaster: 300, pro: 600 }[rodKey];

  if (money < price) {
    log("Not enough money.");
    return;
  }

  money -= price;
  currentRod = rodKey;
  log(`Purchased ${rods[rodKey].name}`);
  updateUI();
};

function resetLine() {
  casting = false;
  inWater = false;
  bite = false;
  reelProgress = 0;
  tension = 0;
  reelBtn.disabled = true;
  castBtn.disabled = false;
}

function catchFish() {
  const fish = fishTypes[Math.floor(Math.random() * fishTypes.length)];
  const weight = fish.base * (Math.random() * 1.2 + 0.6);
  const value = weight * fish.value;

  inventory.push({ name: fish.name, weight, value });
  log(`Caught a ${fish.name} (${weight.toFixed(2)} kg) worth $${value.toFixed(2)}`);
  updateUI();
}

function update(dt) {
  if (casting) {
    hookY += 200 * dt;
    if (hookY >= targetY) {
      hookY = targetY;
      casting = false;
      inWater = true;
      log("Line in water...");
    }
  }

  if (inWater && !bite) {
    if (Math.random() < rods[currentRod].bite * dt * 0.5) {
      bite = true;
      reelBtn.disabled = false;
      log("A fish bites! Reel it in!");
    }
  }

  if (bite) {
    tension -= dt * 0.3;
    if (tension < 0) tension = 0;
  }
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = "#87ceeb";
  ctx.fillRect(0, 0, canvas.width, 120);

  ctx.fillStyle = "#4fa3d1";
  ctx.fillRect(0, 120, canvas.width, canvas.height);

  ctx.strokeStyle = "#fff";
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(hookX, 100);
  ctx.lineTo(hookX, hookY);
  ctx.stroke();

  ctx.fillStyle = bite ? "red" : "black";
  ctx.beginPath();
  ctx.arc(hookX, hookY, 6, 0, Math.PI * 2);
  ctx.fill();

  if (bite) {
    ctx.fillStyle = "#222";
    ctx.fillRect(20, 20, 200, 20);

    ctx.fillStyle = "#0f0";
    ctx.fillRect(20, 20, reelProgress * 200, 20);

    ctx.fillStyle = "#f00";
    ctx.fillRect(20, 50, tension * 200, 20);

    ctx.strokeStyle = "#fff";
    ctx.strokeRect(20, 20, 200, 20);
    ctx.strokeRect(20, 50, 200, 20);

    ctx.fillStyle = "#fff";
    ctx.fillText("Reel Progress", 20, 15);
    ctx.fillText("Line Tension", 20, 45);
  }
}

let last = 0;
function loop(ts) {
  const dt = (ts - last) / 1000;
  last = ts;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}

updateUI();
requestAnimationFrame(loop);
</script>

</body>
</html>
