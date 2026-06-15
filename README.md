[index (2).html](https://github.com/user-attachments/files/28968757/index.2.html)
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Orbit Runner</title>
<style>
body{margin:0;background:black;overflow:hidden;display:flex;justify-content:center;align-items:center;height:100vh;}
canvas{display:block;background:black;}
</style>
</head>
<body>
<canvas id="game"></canvas>
<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = 400;
canvas.height = 600;

const planets = [
  { name: "Merkür", emoji: "🪨", color: "#888" },
  { name: "Venüs", emoji: "🌕", color: "#e8c96a" },
  { name: "Dünya", emoji: "🌍", color: "#4fa3e0" },
  { name: "Mars", emoji: "🔴", color: "#c1440e" },
  { name: "Jüpiter", emoji: "🟠", color: "#c88b3a" },
  { name: "Satürn", emoji: "🪐", color: "#e4d191" },
  { name: "Uranüs", emoji: "🔵", color: "#7de8e8" },
  { name: "Neptün", emoji: "💙", color: "#5b5bd6" }
];

let stars = [];
for (let i = 0; i < 60; i++) {
  stars.push({ x: Math.random() * 400, y: Math.random() * 600, speed: 0.3 + Math.random() * 0.7, size: Math.random() * 2 });
}

let planetIndex = 0;
let planet = { x: 200, y: 280, r: 50 };
let satellite = { angle: 0, r: 110, speed: 0.03 };
let obstacles = [];
let score = 0, bestScore = 0, started = false, gameOver = false;
let level = 1, newRecord = false;

function spawnObstacle() {
  if (obstacles.length > 20) return;
  obstacles.push({
    angle: Math.random() * Math.PI * 2,
    r: 260,
    speed: 0.8 + Math.random() * 0.8 + level * 0.1,
    size: 10 + Math.random() * 8,
    emoji: ["☄️","💥","⚡","🌑"][Math.floor(Math.random()*4)]
  });
}

function resetGame() {
  obstacles = [];
  score = 0;
  planetIndex = 0;
  satellite.angle = 0;
  satellite.speed = 0.03;
  gameOver = false;
  level = 1;
  newRecord = false;
}

function update() {
  if (!started || gameOver) return;

  satellite.angle += satellite.speed;

  if (Math.random() < 0.025) spawnObstacle();

  let sx = planet.x + Math.cos(satellite.angle) * satellite.r;
  let sy = planet.y + Math.sin(satellite.angle) * satellite.r;

  for (let o of obstacles) {
    o.r -= o.speed;
    let ox = planet.x + Math.cos(o.angle) * o.r;
    let oy = planet.y + Math.sin(o.angle) * o.r;
    let dx = sx - ox, dy = sy - oy;
    if (dx * dx + dy * dy < (o.size + 15) * (o.size + 15)) {
      gameOver = true;
    }
  }
  obstacles = obstacles.filter(o => o.r > planet.r + 10);

  score++;
  if (score % 300 === 0) {
    level++;
    obstacles.forEach(o => o.speed += 0.15);
  }
  if (score > bestScore) { bestScore = score; newRecord = true; }
  if (score % 400 === 0 && planetIndex < planets.length - 1) planetIndex++;

  for (let s of stars) {
    s.y += s.speed;
    if (s.y > canvas.height) { s.y = 0; s.x = Math.random() * 400; }
  }
}

function drawEmoji(emoji, x, y, size) {
  ctx.font = size + "px serif";
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.fillText(emoji, x, y);
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Yıldızlar
  ctx.fillStyle = "white";
  for (let s of stars) {
    ctx.globalAlpha = 0.5 + Math.random() * 0.5;
    ctx.fillRect(s.x, s.y, s.size + 1, s.size + 1);
  }
  ctx.globalAlpha = 1;

  // Gezegen
  let p = planets[planetIndex];
  drawEmoji(p.emoji, planet.x, planet.y, planet.r * 1.8);

  // Gezegen adı
  ctx.fillStyle = p.color;
  ctx.font = "bold 14px Arial";
  ctx.textAlign = "center";
  ctx.fillText(p.name, planet.x, planet.y + planet.r + 20);

  // Orbit çizgisi
  ctx.strokeStyle = "rgba(255,255,255,0.1)";
  ctx.lineWidth = 1;
  ctx.beginPath();
  ctx.arc(planet.x, planet.y, satellite.r, 0, Math.PI * 2);
  ctx.stroke();

  // Uydu (uzay mekiği)
  let sx = planet.x + Math.cos(satellite.angle) * satellite.r;
  let sy = planet.y + Math.sin(satellite.angle) * satellite.r;
  ctx.save();
  ctx.translate(sx, sy);
  ctx.rotate(satellite.angle + Math.PI / 2);
  ctx.font = "28px serif";
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.fillText("🚀", 0, 0);
  ctx.restore();

  // Engeller
  for (let o of obstacles) {
    let ox = planet.x + Math.cos(o.angle) * o.r;
    let oy = planet.y + Math.sin(o.angle) * o.r;
    drawEmoji(o.emoji, ox, oy, o.size * 2);
  }

  // Skor
  ctx.fillStyle = "#00ffff";
  ctx.font = "bold 16px Arial";
  ctx.textAlign = "left";
  ctx.fillText("Skor: " + score, 10, 25);
  ctx.fillText("Seviye: " + level, 10, 48);
  ctx.fillStyle = "#ffaa00";
  ctx.fillText("En İyi: " + bestScore, 10, 71);

  if (newRecord && score > 0) {
    ctx.fillStyle = "gold";
    ctx.font = "bold 14px Arial";
    ctx.textAlign = "right";
    ctx.fillText("🏆 YENİ REKOR!", 390, 25);
  }

  if (!started) {
    ctx.fillStyle = "rgba(0,0,0,0.6)";
    ctx.fillRect(0, 0, 400, 600);
    ctx.fillStyle = "white";
    ctx.font = "bold 28px Arial";
    ctx.textAlign = "center";
    ctx.fillText("🚀 ORBIT RUNNER", 200, 220);
    ctx.font = "16px Arial";
    ctx.fillStyle = "#aaa";
    ctx.fillText("Tıklayarak yönü değiştir", 200, 270);
    ctx.fillText("Engellerden kaç!", 200, 300);
    ctx.fillStyle = "#00ffff";
    ctx.font = "bold 20px Arial";
    ctx.fillText("► BAŞLAMAK İÇİN TIKLA", 200, 360);
  }

  if (gameOver) {
    ctx.fillStyle = "rgba(0,0,0,0.7)";
    ctx.fillRect(0, 0, 400, 600);
    ctx.fillStyle = "#ff4444";
    ctx.font = "bold 36px Arial";
    ctx.textAlign = "center";
    ctx.fillText("GAME OVER", 200, 230);
    ctx.fillStyle = "white";
    ctx.font = "18px Arial";
    ctx.fillText("Skor: " + score, 200, 275);
    ctx.fillText("En İyi: " + bestScore, 200, 305);
    ctx.fillStyle = "#00ffff";
    ctx.font = "bold 20px Arial";
    ctx.fillText("► Tekrar Oynamak İçin Tıkla", 200, 360);
  }
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

canvas.addEventListener("click", () => {
  if (!started) { started = true; resetGame(); return; }
  if (gameOver) { resetGame(); return; }
  satellite.speed *= -1;
});

// Klavye desteği
document.addEventListener("keydown", (e) => {
  if (e.code === "Space" || e.code === "ArrowUp") {
    if (!started) { started = true; resetGame(); return; }
    if (gameOver) { resetGame(); return; }
    satellite.speed *= -1;
  }
});

loop();
</script>
</body>
</html>
