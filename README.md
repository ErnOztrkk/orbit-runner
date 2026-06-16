
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Orbit Runner</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: #000; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; font-family: 'Arial', sans-serif; }
canvas { display: block; cursor: pointer; }
#ui { position: absolute; top: 0; left: 50%; transform: translateX(-50%); width: 400px; pointer-events: none; }
#share-btn {
  position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%);
  background: linear-gradient(135deg, #1da1f2, #0d8ecf);
  color: white; border: none; padding: 12px 28px; border-radius: 25px;
  font-size: 16px; font-weight: bold; cursor: pointer; display: none;
  box-shadow: 0 4px 15px rgba(29,161,242,0.5); pointer-events: all;
  transition: transform 0.2s;
}
#share-btn:hover { transform: translateX(-50%) scale(1.05); }
#leaderboard {
  position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
  background: rgba(0,0,0,0.95); border: 1px solid rgba(255,255,255,0.1);
  border-radius: 16px; padding: 20px; width: 300px; display: none;
  pointer-events: all; z-index: 10;
}
#leaderboard h3 { color: gold; text-align: center; margin-bottom: 15px; font-size: 18px; }
#leaderboard-list { list-style: none; }
#leaderboard-list li { color: white; padding: 8px 0; border-bottom: 1px solid rgba(255,255,255,0.1); display: flex; justify-content: space-between; font-size: 14px; }
#leaderboard-list li:first-child { color: gold; }
#leaderboard-list li:nth-child(2) { color: #C0C0C0; }
#leaderboard-list li:nth-child(3) { color: #CD7F32; }
#close-lb { display: block; margin: 15px auto 0; background: rgba(255,255,255,0.1); color: white; border: none; padding: 8px 20px; border-radius: 20px; cursor: pointer; }
#lb-btn {
  position: absolute; top: 10px; right: 10px;
  background: rgba(255,255,255,0.1); color: white; border: none;
  padding: 6px 12px; border-radius: 15px; font-size: 12px; cursor: pointer;
  pointer-events: all;
}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="ui">
  <button id="lb-btn">🏆 Liderlik</button>
  <button id="share-btn">📤 Skoru Paylaş</button>
  <div id="leaderboard">
    <h3>🏆 Liderlik Tablosu</h3>
    <ul id="leaderboard-list"></ul>
    <button id="close-lb">Kapat</button>
  </div>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = 400;
canvas.height = 600;

// ── LEADERBOARD ──
let scores = JSON.parse(localStorage.getItem("orbitScores") || "[]");
function saveScore(s) {
  scores.push({ score: s, date: new Date().toLocaleDateString("tr-TR") });
  scores.sort((a, b) => b.score - a.score);
  scores = scores.slice(0, 10);
  localStorage.setItem("orbitScores", JSON.stringify(scores));
  renderLeaderboard();
}
function renderLeaderboard() {
  const medals = ["🥇","🥈","🥉"];
  const list = document.getElementById("leaderboard-list");
  list.innerHTML = scores.length === 0
    ? "<li style='color:#888;justify-content:center'>Henüz skor yok!</li>"
    : scores.map((s, i) => `<li><span>${medals[i] || (i+1)+". "} ${s.date}</span><span>${s.score}</span></li>`).join("");
}
renderLeaderboard();
document.getElementById("lb-btn").onclick = () => {
  const lb = document.getElementById("leaderboard");
  lb.style.display = lb.style.display === "block" ? "none" : "block";
};
document.getElementById("close-lb").onclick = () => {
  document.getElementById("leaderboard").style.display = "none";
};
document.getElementById("share-btn").onclick = () => {
  const text = `🚀 Orbit Runner'da ${score} puan yaptım! Sen yapabilir misin? https://ernoztrkk.github.io/orbit-runner/`;
  if (navigator.share) {
    navigator.share({ title: "Orbit Runner", text });
  } else {
    navigator.clipboard.writeText(text).then(() => alert("Skor panoya kopyalandı!"));
  }
};

// ── GEZEGENLER ──
const planets = [
  { name: "Merkür", emoji: "🪨", color: "#aaa", bg: "#1a1a2e" },
  { name: "Venüs", emoji: "🌕", color: "#e8c96a", bg: "#2d1b00" },
  { name: "Dünya", emoji: "🌍", color: "#4fa3e0", bg: "#001428" },
  { name: "Mars", emoji: "🔴", color: "#c1440e", bg: "#1a0500" },
  { name: "Jüpiter", emoji: "🪐", color: "#c88b3a", bg: "#1a1000" },
  { name: "Satürn", emoji: "💛", color: "#e4d191", bg: "#1a1500" },
  { name: "Uranüs", emoji: "🔵", color: "#7de8e8", bg: "#001a1a" },
  { name: "Neptün", emoji: "💙", color: "#5b5bd6", bg: "#00001a" }
];

// ── YILDIZLAR ──
let stars = [];
for (let i = 0; i < 80; i++) {
  stars.push({ x: Math.random()*400, y: Math.random()*600, speed: 0.2+Math.random()*0.5, size: Math.random()*1.5, twinkle: Math.random()*Math.PI*2 });
}

// ── PARTİKÜLLER ──
let particles = [];
function spawnParticles(x, y, color, count=20) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = 2 + Math.random() * 5;
    particles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      life: 1, decay: 0.02 + Math.random() * 0.03,
      size: 3 + Math.random() * 5,
      color
    });
  }
}

// ── GEZEGEN GEÇİŞ ANİMASYONU ──
let planetTransition = { active: false, alpha: 0, fadeIn: false };
function triggerPlanetTransition() {
  planetTransition = { active: true, alpha: 1, fadeIn: true };
  spawnParticles(planet.x, planet.y, planets[planetIndex].color, 30);
}

// ── OYUN DEĞİŞKENLERİ ──
let planetIndex = 0;
let planet = { x: 200, y: 270, r: 50 };
let satellite = { angle: 0, r: 115, speed: 0.032 };
let obstacles = [];
let score = 0, bestScore = JSON.parse(localStorage.getItem("orbitBest") || "0");
let started = false, gameOver = false, level = 1, newRecord = false;
let bgColor = planets[0].bg;
let currentBg = planets[0].bg;
let shakeTimer = 0;
let powerups = [];
let shield = false, shieldTimer = 0;

function spawnObstacle() {
  if (obstacles.length > 22) return;
  obstacles.push({
    angle: Math.random() * Math.PI * 2,
    r: 270,
    speed: 0.7 + Math.random() * 0.8 + level * 0.08,
    size: 10 + Math.random() * 8,
    emoji: ["☄️","💥","⚡","🌑","💫"][Math.floor(Math.random()*5)],
    trail: []
  });
}

function spawnPowerup() {
  if (powerups.length > 1) return;
  powerups.push({
    angle: Math.random() * Math.PI * 2,
    r: 190,
    emoji: "🛡️",
    life: 300
  });
}

function resetGame() {
  obstacles = []; particles = []; powerups = [];
  score = 0; planetIndex = 0;
  satellite.angle = 0; satellite.speed = 0.032;
  gameOver = false; level = 1; newRecord = false;
  shield = false; shieldTimer = 0;
  bgColor = planets[0].bg;
  document.getElementById("share-btn").style.display = "none";
}

function lerpColor(a, b, t) {
  const ah = parseInt(a.slice(1),16), bh = parseInt(b.slice(1),16);
  const ar=(ah>>16)&0xff, ag=(ah>>8)&0xff, ab=ah&0xff;
  const br=(bh>>16)&0xff, bg_=(bh>>8)&0xff, bb=bh&0xff;
  const rr=Math.round(ar+(br-ar)*t), rg=Math.round(ag+(bg_-ag)*t), rb=Math.round(ab+(bb-ab)*t);
  return `#${((rr<<16)|(rg<<8)|rb).toString(16).padStart(6,'0')}`;
}

function update() {
  if (!started || gameOver) return;

  satellite.angle += satellite.speed;

  // Arka plan geçişi
  currentBg = lerpColor(currentBg, planets[planetIndex].bg, 0.02);

  // Gezegen geçiş animasyonu
  if (planetTransition.active) {
    planetTransition.alpha -= 0.03;
    if (planetTransition.alpha <= 0) planetTransition.active = false;
  }

  // Titreme
  if (shakeTimer > 0) shakeTimer--;

  // Partiküller
  particles = particles.filter(p => p.life > 0);
  particles.forEach(p => {
    p.x += p.vx; p.y += p.vy;
    p.vy += 0.1;
    p.vx *= 0.95; p.vy *= 0.95;
    p.life -= p.decay;
    p.size *= 0.97;
  });

  // Kalkan
  if (shield) { shieldTimer--; if (shieldTimer <= 0) shield = false; }

  // Güç artırıcılar
  powerups.forEach(pw => pw.life--);
  powerups = powerups.filter(pw => pw.life > 0);

  if (Math.random() < 0.025) spawnObstacle();
  if (Math.random() < 0.003) spawnPowerup();

  let sx = planet.x + Math.cos(satellite.angle) * satellite.r;
  let sy = planet.y + Math.sin(satellite.angle) * satellite.r;

  // Güç artırıcı çarpışma
  powerups = powerups.filter(pw => {
    let px2 = planet.x + Math.cos(pw.angle) * pw.r;
    let py2 = planet.y + Math.sin(pw.angle) * pw.r;
    let dx = sx-px2, dy = sy-py2;
    if (dx*dx+dy*dy < 35*35) {
      shield = true; shieldTimer = 300;
      spawnParticles(px2, py2, "#00ffff", 15);
      return false;
    }
    return true;
  });

  // Engel çarpışma
  for (let o of obstacles) {
    o.r -= o.speed;
    o.trail.push({x: planet.x+Math.cos(o.angle)*o.r, y: planet.y+Math.sin(o.angle)*o.r});
    if (o.trail.length > 5) o.trail.shift();

    let ox = planet.x+Math.cos(o.angle)*o.r;
    let oy = planet.y+Math.sin(o.angle)*o.r;
    let dx = sx-ox, dy = sy-oy;
    if (dx*dx+dy*dy < (o.size+14)*(o.size+14)) {
      if (shield) {
        shield = false; shieldTimer = 0;
        spawnParticles(ox, oy, "#00ffff", 20);
        obstacles = obstacles.filter(ob => ob !== o);
      } else {
        gameOver = true;
        spawnParticles(sx, sy, "#ff4444", 40);
        spawnParticles(sx, sy, "#ffaa00", 20);
        shakeTimer = 20;
        saveScore(score);
        if (score > bestScore) { bestScore = score; localStorage.setItem("orbitBest", bestScore); }
        document.getElementById("share-btn").style.display = "block";
      }
      break;
    }
  }
  obstacles = obstacles.filter(o => o.r > planet.r+5);

  score++;
  if (score%350===0) { level++; obstacles.forEach(o=>o.speed+=0.12); }
  if (score>bestScore && !gameOver) { bestScore=score; newRecord=true; localStorage.setItem("orbitBest",bestScore); }

  // Gezegen değişimi
  let newIdx = Math.min(Math.floor(score/500), planets.length-1);
  if (newIdx !== planetIndex) {
    planetIndex = newIdx;
    triggerPlanetTransition();
  }

  // Yıldızlar
  stars.forEach(s => {
    s.y += s.speed; s.twinkle += 0.05;
    if (s.y > canvas.height) { s.y=0; s.x=Math.random()*400; }
  });
}

function drawEmoji(emoji, x, y, size) {
  ctx.save();
  ctx.font = size+"px serif";
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.fillText(emoji, x, y);
  ctx.restore();
}

function draw() {
  // Titreme efekti
  let sx2 = 0, sy2 = 0;
  if (shakeTimer > 0) { sx2=(Math.random()-0.5)*8; sy2=(Math.random()-0.5)*8; }
  ctx.save();
  ctx.translate(sx2, sy2);

  // Arka plan
  ctx.fillStyle = currentBg;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Yıldızlar
  stars.forEach(s => {
    const alpha = 0.4 + 0.6*Math.abs(Math.sin(s.twinkle));
    ctx.fillStyle = `rgba(255,255,255,${alpha})`;
    ctx.fillRect(s.x, s.y, s.size, s.size);
  });

  // Gezegen geçiş flaşı
  if (planetTransition.active) {
    ctx.fillStyle = `rgba(255,255,255,${planetTransition.alpha * 0.3})`;
    ctx.fillRect(0,0,canvas.width,canvas.height);
  }

  const p = planets[planetIndex];

  // Orbit halkası
  ctx.strokeStyle = `rgba(255,255,255,0.08)`;
  ctx.lineWidth = 1;
  ctx.setLineDash([5,8]);
  ctx.beginPath();
  ctx.arc(planet.x, planet.y, satellite.r, 0, Math.PI*2);
  ctx.stroke();
  ctx.setLineDash([]);

  // Gezegen
  ctx.save();
  if (planetTransition.active) {
    ctx.globalAlpha = 0.5 + 0.5 * (1-planetTransition.alpha);
  }
  drawEmoji(p.emoji, planet.x, planet.y, planet.r*1.9);
  ctx.restore();

  // Gezegen adı
  ctx.fillStyle = p.color;
  ctx.font = "bold 13px Arial";
  ctx.textAlign = "center";
  ctx.fillText(p.name, planet.x, planet.y+planet.r+22);

  // Uydu izi
  const satX = planet.x+Math.cos(satellite.angle)*satellite.r;
  const satY = planet.y+Math.sin(satellite.angle)*satellite.r;

  // Kalkan efekti
  if (shield) {
    const pulse = 0.5+0.5*Math.sin(Date.now()*0.01);
    ctx.strokeStyle = `rgba(0,255,255,${0.4+0.4*pulse})`;
    ctx.lineWidth = 2+pulse*2;
    ctx.beginPath();
    ctx.arc(satX, satY, 28, 0, Math.PI*2);
    ctx.stroke();
  }

  // Uydu (roket)
  ctx.save();
  ctx.translate(satX, satY);
  ctx.rotate(satellite.angle + Math.PI/2);
  drawEmoji("🚀", 0, 0, 28);
  ctx.restore();

  // Güç artırıcılar
  powerups.forEach(pw => {
    const ppx = planet.x+Math.cos(pw.angle)*pw.r;
    const ppy = planet.y+Math.sin(pw.angle)*pw.r;
    const pulse = 0.7+0.3*Math.sin(Date.now()*0.008);
    ctx.globalAlpha = pulse;
    drawEmoji(pw.emoji, ppx, ppy, 24);
    ctx.globalAlpha = 1;
  });

  // Engeller
  obstacles.forEach(o => {
    const ox = planet.x+Math.cos(o.angle)*o.r;
    const oy = planet.y+Math.sin(o.angle)*o.r;

    // İz efekti
    o.trail.forEach((t,i) => {
      ctx.globalAlpha = (i/o.trail.length)*0.3;
      drawEmoji(o.emoji, t.x, t.y, o.size*1.5);
    });
    ctx.globalAlpha = 1;
    drawEmoji(o.emoji, ox, oy, o.size*2);
  });

  // Partiküller
  particles.forEach(p2 => {
    ctx.globalAlpha = p2.life;
    ctx.fillStyle = p2.color;
    ctx.beginPath();
    ctx.arc(p2.x, p2.y, p2.size, 0, Math.PI*2);
    ctx.fill();
  });
  ctx.globalAlpha = 1;

  // HUD
  ctx.fillStyle = "rgba(0,0,0,0.4)";
  ctx.beginPath();
  ctx.roundRect(6, 6, 130, 80, 10);
  ctx.fill();

  ctx.fillStyle = "#fff";
  ctx.font = "bold 18px Arial";
  ctx.textAlign = "left";
  ctx.fillText("⭐ "+score, 16, 30);
  ctx.font = "13px Arial";
  ctx.fillStyle = "#ffaa00";
  ctx.fillText("🏆 "+bestScore, 16, 52);
  ctx.fillStyle = "#aaf";
  ctx.fillText("Seviye "+level, 16, 72);

  if (newRecord && score>0) {
    ctx.fillStyle = "gold";
    ctx.font = "bold 13px Arial";
    ctx.textAlign = "right";
    ctx.fillText("✨ YENİ REKOR!", 394, 25);
  }

  if (shield) {
    ctx.fillStyle = "#00ffff";
    ctx.font = "13px Arial";
    ctx.textAlign = "right";
    ctx.fillText("🛡️ Kalkan: "+shieldTimer, 394, 50);
  }

  // Başlangıç ekranı
  if (!started) {
    ctx.fillStyle = "rgba(0,0,0,0.75)";
    ctx.fillRect(0,0,400,600);

    ctx.fillStyle = "#fff";
    ctx.font = "bold 32px Arial";
    ctx.textAlign = "center";
    ctx.fillText("🚀 ORBIT RUNNER", 200, 180);

    ctx.font = "14px Arial";
    ctx.fillStyle = "#aaa";
    ctx.fillText("Roketi orbit üzerinde döndür", 200, 230);
    ctx.fillText("Tıkla = Yön değiştir", 200, 255);
    ctx.fillText("☄️ Engellerden kaç!", 200, 280);
    ctx.fillText("🛡️ Kalkan topla!", 200, 305);

    ctx.fillStyle = "#00ffff";
    ctx.font = "bold 20px Arial";
    ctx.fillText("► BAŞLAMAK İÇİN TIKLA", 200, 370);

    ctx.fillStyle = "#666";
    ctx.font = "12px Arial";
    ctx.fillText("veya Space / Yukarı ok tuşu", 200, 400);
  }

  // Game over ekranı
  if (gameOver) {
    ctx.fillStyle = "rgba(0,0,0,0.8)";
    ctx.fillRect(0,0,400,600);

    ctx.fillStyle = "#ff4444";
    ctx.font = "bold 40px Arial";
    ctx.textAlign = "center";
    ctx.fillText("💥 GAME OVER", 200, 200);

    ctx.fillStyle = "#fff";
    ctx.font = "bold 22px Arial";
    ctx.fillText("Skor: "+score, 200, 255);
    ctx.fillStyle = "#ffaa00";
    ctx.font = "18px Arial";
    ctx.fillText("🏆 En İyi: "+bestScore, 200, 290);
    ctx.fillStyle = "#aaa";
    ctx.font = "14px Arial";
    ctx.fillText("Seviye "+level+"'e ulaştın!", 200, 325);

    ctx.fillStyle = "#00ffff";
    ctx.font = "bold 18px Arial";
    ctx.fillText("► Tekrar oynamak için tıkla", 200, 390);
  }

  ctx.restore();
}

function loop() { update(); draw(); requestAnimationFrame(loop); }

canvas.addEventListener("click", () => {
  if (!started) { started=true; resetGame(); return; }
  if (gameOver) { resetGame(); return; }
  satellite.speed *= -1;
});

document.addEventListener("keydown", e => {
  if (["Space","ArrowUp"].includes(e.code)) {
    e.preventDefault();
    if (!started) { started=true; resetGame(); return; }
    if (gameOver) { resetGame(); return; }
    satellite.speed *= -1;
  }
});

loop();
</script>
</body>
</html>
