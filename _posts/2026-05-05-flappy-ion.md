---
title: "Flappy Ion"
date: 2026-05-05
permalink: /posts/2026/05/flappy-ion/
tags:
  - games
  - fun
---

Navigate a charged ion through the quadrupole rods without getting deflected. Each gap you pass through counts as one m/z crossing. How far can your ion travel?

Click anywhere on the game or press Space to flap!

<style>
#fi-wrap { display: flex; flex-direction: column; align-items: center; font-family: monospace; }
#fi-hud { display: flex; justify-content: space-between; width: 600px; padding: 6px 4px; font-size: 13px; color: #888; }
#fi-canvas { display: block; border-radius: 8px; border: 1px solid #333; cursor: pointer; }
</style>

<div id="fi-wrap">
  <div id="fi-hud">
    <span id="fi-score">Score: 0</span>
    <span id="fi-best">Best: 0</span>
    <span id="fi-state">Click or Space to Start</span>
  </div>
  <canvas id="fi-canvas" width="600" height="400"></canvas>
</div>

{% raw %}
<script>
(function(){
const canvas = document.getElementById('fi-canvas');
const ctx = canvas.getContext('2d');
const W = 600, H = 400;
const ION_COLOR = '#00ffcc';
const SPARK_COLORS = ['#00ffcc','#ffcc00','#ff6600','#ffffff'];
const ROD_SPEED_BASE = 2.8;
const GRAVITY = 0.38;

let state = 'idle', score = 0, highScore = 0, frame = 0;
let ion, rods, particles, flashes;

function reset() {
  ion = { x: 120, y: H/2, vy: 0, radius: 10, trail: [] };
  rods = []; particles = []; flashes = [];
  score = 0; frame = 0;
  spawnRod();
}

function spawnRod() {
  const minThick = 18, maxThick = 70;
  const topThick = minThick + Math.random()*(maxThick-minThick);
  const botThick = minThick + Math.random()*(maxThick-minThick);
  const gap = 90 + Math.random()*40;
  const gapY = gap + topThick + Math.random()*(H - gap - topThick - botThick - 60) + 30;
  rods.push({ x: W+30, topThick, botThick, gapTop: gapY-gap/2, gapBot: gapY+gap/2, scored: false, phase: Math.random()*Math.PI*2 });
}

function flap() {
  if (state === 'idle' || state === 'dead') { reset(); state = 'playing'; return; }
  if (state === 'playing') {
    ion.vy = -7.5;
    for (let i=0; i<6; i++) {
      particles.push({ x: ion.x-ion.radius, y: ion.y+(Math.random()-0.5)*10,
        vx: -2-Math.random()*3, vy: (Math.random()-0.5)*2,
        life: 1, decay: 0.08+Math.random()*0.05,
        color: SPARK_COLORS[Math.floor(Math.random()*4)], size: 2+Math.random()*3 });
    }
  }
}

function update() {
  frame++;
  ion.vy += GRAVITY;
  ion.y += ion.vy;
  ion.trail.push({ x: ion.x, y: ion.y });
  if (ion.trail.length > 18) ion.trail.shift();
  const speed = ROD_SPEED_BASE + score*0.04;
  const last = rods[rods.length-1];
  if (!last || last.x < W-220) spawnRod();
  rods.forEach(r => { r.x -= speed; });
  rods = rods.filter(r => r.x > -60);
  rods.forEach(r => {
    if (!r.scored && r.x+20 < ion.x) {
      r.scored = true; score++;
      if (score > highScore) highScore = score;
      flashes.push({ life: 1 });
    }
  });
  if (checkCollision()) {
    state = 'dead';
    for (let i=0; i<30; i++) {
      const a = Math.random()*Math.PI*2, s = 2+Math.random()*5;
      particles.push({ x: ion.x, y: ion.y, vx: Math.cos(a)*s, vy: Math.sin(a)*s,
        life: 1, decay: 0.03+Math.random()*0.04,
        color: SPARK_COLORS[Math.floor(Math.random()*4)], size: 2+Math.random()*4 });
    }
  }
  particles.forEach(p => { p.x+=p.vx; p.y+=p.vy; p.vy+=0.1; p.life-=p.decay; });
  particles = particles.filter(p => p.life > 0);
  flashes = flashes.filter(f => { f.life-=0.08; return f.life>0; });
  document.getElementById('fi-score').textContent = 'Score: ' + score;
  document.getElementById('fi-best').textContent = 'Best: ' + highScore;
  document.getElementById('fi-state').textContent =
    state==='playing' ? 'm/z crossings: ' + score : state==='dead' ? 'Ion lost! Click to retry' : 'Click or Space to Start';
}

function checkCollision() {
  if (ion.y-ion.radius < 0 || ion.y+ion.radius > H) return true;
  for (const r of rods) {
    if (ion.x+ion.radius > r.x-12 && ion.x-ion.radius < r.x+12) {
      if (ion.y-ion.radius < r.gapTop || ion.y+ion.radius > r.gapBot) return true;
    }
  }
  return false;
}

function drawBg() {
  const g = ctx.createLinearGradient(0,0,0,H);
  g.addColorStop(0,'#0a0a1a'); g.addColorStop(1,'#0d1030');
  ctx.fillStyle=g; ctx.fillRect(0,0,W,H);
  ctx.strokeStyle='rgba(80,80,160,0.15)'; ctx.lineWidth=0.5;
  const off=(frame*ROD_SPEED_BASE)%40;
  for(let x=-off;x<W;x+=40){ctx.beginPath();ctx.moveTo(x,0);ctx.lineTo(x,H);ctx.stroke();}
  for(let y=0;y<H;y+=40){ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(W,y);ctx.stroke();}
  ctx.fillStyle='rgba(100,100,200,0.4)'; ctx.font='11px monospace';
  ctx.fillText('m/z \u2192', W-50, H-8);
}

function drawRodBar(x1,y1,x2,y2,isTop) {
  const g=ctx.createLinearGradient(0,y1,0,y2);
  g.addColorStop(0,'#4444aa'); g.addColorStop(0.3,'#aaaaee');
  g.addColorStop(0.7,'#8888cc'); g.addColorStop(1,'#222255');
  ctx.fillStyle=g; ctx.fillRect(x1,y1,x2-x1,y2-y1);
  ctx.fillStyle='rgba(200,200,255,0.15)';
  ctx.fillRect(x1, isTop?y2-4:y1, x2-x1, 4);
  const capX=isTop?x2:x1;
  const cg=ctx.createRadialGradient(capX,(y1+y2)/2,2,capX,(y1+y2)/2,(y2-y1)/2);
  cg.addColorStop(0,'#aaaaee'); cg.addColorStop(1,'#333388');
  ctx.fillStyle=cg; ctx.beginPath();
  ctx.ellipse(capX,(y1+y2)/2,6,(y2-y1)/2,0,0,Math.PI*2);
  ctx.fill(); ctx.strokeStyle='#6666bb'; ctx.lineWidth=1; ctx.stroke();
  ctx.fillStyle='rgba(150,150,255,0.7)'; ctx.font='bold 9px monospace';
  ctx.fillText(isTop?'+V':'-V', isTop?Math.max(x1+4,capX-16):Math.min(x2-24,capX+8),(y1+y2)/2+3);
}

function drawRods() {
  rods.forEach(r => {
    const gg=ctx.createLinearGradient(0,r.gapTop,0,r.gapBot);
    gg.addColorStop(0,'rgba(80,80,255,0.0)');
    gg.addColorStop(0.5,'rgba(80,80,255,0.08)');
    gg.addColorStop(1,'rgba(80,80,255,0.0)');
    ctx.fillStyle=gg; ctx.fillRect(r.x-32,r.gapTop,64,r.gapBot-r.gapTop);
    drawRodBar(0,0,r.x+12,r.gapTop,true);
    drawRodBar(r.x-12,r.gapBot,W,H,false);
  });
}

function drawIon() {
  if(state==='dead') return;
  ion.trail.forEach((t,i)=>{
    const a=(i/ion.trail.length)*0.5, s=ion.radius*(i/ion.trail.length)*0.7;
    ctx.beginPath(); ctx.arc(t.x,t.y,s,0,Math.PI*2);
    ctx.fillStyle='rgba(0,255,200,'+a+')'; ctx.fill();
  });
  const glow=ctx.createRadialGradient(ion.x,ion.y,0,ion.x,ion.y,ion.radius*3);
  glow.addColorStop(0,'rgba(0,255,200,0.4)'); glow.addColorStop(1,'rgba(0,255,200,0)');
  ctx.fillStyle=glow; ctx.beginPath(); ctx.arc(ion.x,ion.y,ion.radius*3,0,Math.PI*2); ctx.fill();
  const ig=ctx.createRadialGradient(ion.x-3,ion.y-3,1,ion.x,ion.y,ion.radius);
  ig.addColorStop(0,'#ffffff'); ig.addColorStop(0.4,ION_COLOR); ig.addColorStop(1,'#006644');
  ctx.fillStyle=ig; ctx.beginPath(); ctx.arc(ion.x,ion.y,ion.radius,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#003322'; ctx.font='bold 11px monospace';
  ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.fillText('+',ion.x,ion.y);
  ctx.textAlign='left'; ctx.textBaseline='alphabetic';
}

function drawParticles() {
  particles.forEach(p=>{
    ctx.globalAlpha=p.life; ctx.fillStyle=p.color;
    ctx.beginPath(); ctx.arc(p.x,p.y,p.size*p.life,0,Math.PI*2); ctx.fill();
  });
  ctx.globalAlpha=1;
}

function drawOverlay() {
  if(state==='idle'){
    ctx.fillStyle='rgba(0,0,20,0.6)'; ctx.fillRect(0,0,W,H);
    ctx.fillStyle=ION_COLOR; ctx.font='bold 32px monospace'; ctx.textAlign='center';
    ctx.fillText('FLAPPY ION',W/2,H/2-50);
    ctx.font='16px monospace'; ctx.fillStyle='#aaaaff';
    ctx.fillText('Navigate the ion through the quadrupole',W/2,H/2-10);
    ctx.fillText('Click or press Space to start',W/2,H/2+20);
    ctx.font='12px monospace'; ctx.fillStyle='#6666aa';
    ctx.fillText('Each gap crossed = +1 m/z unit',W/2,H/2+55);
    ctx.textAlign='left';
  }
  if(state==='dead'){
    ctx.fillStyle='rgba(0,0,20,0.7)'; ctx.fillRect(0,0,W,H);
    ctx.fillStyle='#ff4444'; ctx.font='bold 28px monospace'; ctx.textAlign='center';
    ctx.fillText('ION LOST',W/2,H/2-50);
    ctx.fillStyle='#aaaaff'; ctx.font='16px monospace';
    ctx.fillText('m/z crossings: '+score,W/2,H/2-10);
    ctx.fillText('Best: '+highScore,W/2,H/2+20);
    ctx.fillStyle=ION_COLOR; ctx.font='14px monospace';
    ctx.fillText('Click or Space to retry',W/2,H/2+55);
    ctx.textAlign='left';
  }
}

function drawScore() {
  if(state!=='playing') return;
  ctx.fillStyle='rgba(0,255,200,0.9)'; ctx.font='bold 28px monospace';
  ctx.textAlign='center'; ctx.fillText(score,W/2,44); ctx.textAlign='left';
}

function loop() {
  if(state==='playing') update();
  drawBg(); drawRods();
  flashes.forEach(f=>{ ctx.fillStyle='rgba(0,255,200,'+(f.life*0.08)+')'; ctx.fillRect(0,0,W,H); });
  drawIon(); drawParticles(); drawScore(); drawOverlay();
  requestAnimationFrame(loop);
}

canvas.addEventListener('click', flap);
document.addEventListener('keydown', e=>{ if(e.code==='Space'){e.preventDefault();flap();} });
loop();
})();
</script>
{% endraw %}
