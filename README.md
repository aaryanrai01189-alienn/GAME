# GAME
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Plane Shooter - Game</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  overflow:hidden;
  background:#111;
  font-family:sans-serif;
}

canvas{
  position:absolute;
  top:0;
  left:0;
  touch-action:none;
}

#ui{
  position:absolute;
  top:1vh;
  left:1vw;
  color:#fff;
  font-size:4vw;
  z-index:5;
}

#waveBarContainer{
  position:absolute;
  top:5vh;
  left:1vw;
  width:120px;
  height:12px;
  background:#555;
  border-radius:6px;
  z-index:5;
}
#waveBar{
  width:0%;
  height:100%;
  background:lime;
  border-radius:6px;
}

/* 🔥 GAME CONTROL BUTTONS */
.controlBtn{
  position:absolute;
  width:14vw;
  height:14vw;
  border-radius:50%;
  background:rgba(255,255,255,0.3);
  color:#fff;
  font-size:6vw;
  display:flex;
  align-items:center;
  justify-content:center;
  z-index:10;

  user-select:none;
  -webkit-user-select:none;
  -webkit-touch-callout:none;
  touch-action:none;
}

.fire{
  width:16vw;
  height:16vw;
  background:red;
}
</style>
</head>

<body>

<canvas id="gameCanvas"></canvas>
<div id="ui">Score: 0</div>
<div id="waveBarContainer"><div id="waveBar"></div></div>

<!-- 🎮 CONTROLS (DIV, NOT BUTTON) -->
<div id="leftBtn" class="controlBtn" style="left:5%;bottom:15%;">◀</div>
<div id="rightBtn" class="controlBtn" style="left:22%;bottom:15%;">▶</div>
<div id="fireBtn" class="controlBtn fire" style="right:5%;bottom:15%;">FIRE</div>

<script>
// --- Canvas ---
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
canvas.width = innerWidth;
canvas.height = innerHeight;
let w = canvas.width, h = canvas.height;

// --- Game State ---
let px=w/2, py=h*0.85;
let pw=w/25, ph=h/15;
let speed=w/150;

let bullets=[], zombies=[];
let score=0, waveProgress=0;
let keys={}, fireInterval=null;

const ui=document.getElementById("ui");
const waveBar=document.getElementById("waveBar");

// --- Resize ---
addEventListener("resize",()=>{
  canvas.width=innerWidth;
  canvas.height=innerHeight;
  w=canvas.width; h=canvas.height;
  pw=w/25; ph=h/15; py=h*0.85;
});

// --- Shooting ---
function startFiring(){
  if(fireInterval) return;
  fireInterval=setInterval(()=>{
    bullets.push({x:px+pw/2,y:py,dy:-8});
  },200);
}
function stopFiring(){
  clearInterval(fireInterval);
  fireInterval=null;
}

// --- Movement ---
function movePlayer(){
  if(keys.left) px-=speed;
  if(keys.right) px+=speed;
  px=Math.max(0,Math.min(px,w-pw));
}

// --- 🔥 MOBILE CONTROLS (DIV + POINTER EVENTS) ---
const leftBtn=document.getElementById("leftBtn");
const rightBtn=document.getElementById("rightBtn");
const fireBtn=document.getElementById("fireBtn");

leftBtn.onpointerdown=()=>keys.left=true;
leftBtn.onpointerup=()=>keys.left=false;
leftBtn.onpointerleave=()=>keys.left=false;

rightBtn.onpointerdown=()=>keys.right=true;
rightBtn.onpointerup=()=>keys.right=false;
rightBtn.onpointerleave=()=>keys.right=false;

fireBtn.onpointerdown=startFiring;
fireBtn.onpointerup=stopFiring;
fireBtn.onpointerleave=stopFiring;

// --- Enemies ---
setInterval(()=>{
  let s=w/15;
  zombies.push({x:Math.random()*(w-s),y:-s,size:s,speed:1.5});
},1500);

// --- Update ---
function update(){
  movePlayer();

  bullets.forEach((b,i)=>{
    b.y+=b.dy;
    if(b.y<0) bullets.splice(i,1);
  });

  zombies.forEach((z,zi)=>{
    z.y+=z.speed;
    bullets.forEach((b,bi)=>{
      if(b.x>z.x && b.x<z.x+z.size && b.y>z.y && b.y<z.y+z.size){
        zombies.splice(zi,1);
        bullets.splice(bi,1);
        score+=10;
        waveProgress+=10;
      }
    });
  });
}

// --- Draw ---
function draw(){
  ctx.clearRect(0,0,w,h);

  // Player
  ctx.fillStyle="blue";
  ctx.beginPath();
  ctx.moveTo(px+pw/2,py);
  ctx.lineTo(px,py+ph);
  ctx.lineTo(px+pw,py+ph);
  ctx.fill();

  // Bullets
  ctx.fillStyle="yellow";
  bullets.forEach(b=>ctx.fillRect(b.x,b.y,4,15));

  // Enemies
  ctx.fillStyle="green";
  zombies.forEach(z=>ctx.fillRect(z.x,z.y,z.size,z.size));

  ui.textContent="Score: "+score;
  waveBar.style.width=waveProgress+"%";
}

// --- Loop ---
(function loop(){
  update();
  draw();
  requestAnimationFrame(loop);
})();
</script>
</body>
</html>
