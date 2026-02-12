# fuzzy-octo-waddle

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Contra-style Game</title>
<style>
body { margin:0; overflow:hidden; background:#111; }
canvas { display:block; background:#222; }
#ui { position:absolute; top:10px; left:10px; color:white; font-family:Arial; font-size:20px; }
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="ui">Score: 0 | Lives: 3 | Level: 1</div>

<script>
// --- Canvas Setup ---
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// --- Game State ---
let score = 0, lives = 3, level = 1;
const ui = document.getElementById("ui");

// --- Player ---
const player = {
  x: 100, y: canvas.height-150, width:50, height:60,
  dy:0, speed:5, jump:15, grounded:true,
  cooldown:0, frame:0, frameMax:4
};

// --- Gravity ---
const gravity = 0.8;

// --- Bullets ---
let bullets = [];

// --- Enemies ---
let enemies = [];
function spawnEnemy(){
  const y = canvas.height - 150 - Math.random()*50;
  enemies.push({x: canvas.width+50, y: y, width:40, height:60, speed:2+level, frame:0, frameMax:4});
}
setInterval(spawnEnemy, 2000);

// --- Platforms ---
const platforms = [
  {x:200, y:canvas.height-200, width:200, height:20},
  {x:600, y:canvas.height-300, width:250, height:20},
  {x:1000, y:canvas.height-250, width:200, height:20}
];

// --- Power-ups ---
let powerUps = [];
function spawnPowerUp(){
  powerUps.push({x: Math.random()*(canvas.width-30), y: canvas.height-100, width:30, height:30, type:"life"});
}
setInterval(spawnPowerUp, 15000);

// --- Background Layers ---
const clouds = [];
const trees = [];
const mountains = [];
for(let i=0;i<5;i++){
  clouds.push({x:i*300, y:50+Math.random()*50, width:150, height:60});
  mountains.push({x:i*300, y:canvas.height-300, width:300, height:150});
  trees.push({x:i*200+50, y:canvas.height-120, width:50, height:100});
}

// --- Input ---
let keys = {};
document.addEventListener("keydown", e=>keys[e.code]=true);
document.addEventListener("keyup", e=>keys[e.code]=false);

// --- Shoot ---
function shoot(){
  if(player.cooldown>0) return;
  bullets.push({x:player.x+player.width, y:player.y+player.height/2-5, speed:10, damage:10});
  player.cooldown = 15;
}

// --- Update ---
function update(){
  // Background scroll
  clouds.forEach(c=>{ c.x -= 0.5; if(c.x+c.width<0) c.x=canvas.width; });
  mountains.forEach(m=>{ m.x -= 1; if(m.x+m.width<0) m.x=canvas.width; });
  trees.forEach(t=>{ t.x -= 2; if(t.x+t.width<0) t.x=canvas.width; });

  // Player Movement
  if(keys["ArrowLeft"]) player.x -= player.speed;
  if(keys["ArrowRight"]) player.x += player.speed;
  if(keys["Space"] && player.grounded){ player.dy = -player.jump; player.grounded=false; }
  if(keys["KeyZ"]) shoot();

  // Gravity
  player.dy += gravity;
  player.y += player.dy;
  player.grounded = false;

  // Platform collision
  platforms.forEach(p=>{
    if(player.x + player.width > p.x && player.x < p.x + p.width &&
       player.y + player.height > p.y && player.y + player.height < p.y + p.height + 20 &&
       player.dy >=0){
      player.y = p.y - player.height;
      player.dy = 0;
      player.grounded = true;
    }
  });

  // Ground collision
  if(player.y + player.height > canvas.height-50){
    player.y = canvas.height-50 - player.height;
    player.dy=0;
    player.grounded = true;
  }

  if(player.cooldown>0) player.cooldown--;

  // Bullets
  bullets.forEach((b,i)=>{
    b.x += b.speed;
    if(b.x>canvas.width) bullets.splice(i,1);
    enemies.forEach((e,ei)=>{
      if(b.x < e.x+e.width && b.x+10>e.x && b.y<e.y+e.height && b.y+10>e.y){
        enemies.splice(ei,1);
        bullets.splice(i,1);
        score += 10;
      }
    });
  });

  // Enemies
  enemies.forEach(e=>{
    e.x -= e.speed;
    e.frame = (e.frame + 0.2)%e.frameMax;
    if(player.x < e.x+e.width && player.x+player.width>e.x && player.y<e.y+e.height && player.y+player.height>e.y){
      lives--;
      resetPlayer();
      if(lives<=0){ alert("Game Over! Score: "+score); location.reload(); }
    }
  });

  // Power-ups
  powerUps.forEach((p,pi)=>{
    if(player.x < p.x+p.width && player.x+player.width>p.x && player.y< p.y+p.height && player.y+player.height>p.y){
      if(p.type==="life") lives++;
      powerUps.splice(pi,1);
    }
  });

  ui.innerText = `Score: ${score} | Lives: ${lives} | Level: ${level}`;
}

// --- Draw ---
function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Sky
  ctx.fillStyle="#88ccff";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  // Mountains
  ctx.fillStyle="#555";
  mountains.forEach(m=>ctx.fillRect(m.x,m.y,m.width,m.height));

  // Trees
  ctx.fillStyle="green";
  trees.forEach(t=>ctx.fillRect(t.x,t.y,t.width,t.height));

  // Clouds
  ctx.fillStyle="white";
  clouds.forEach(c=>ctx.fillRect(c.x,c.y,c.width,c.height));

  // Platforms
  ctx.fillStyle="#666";
  platforms.forEach(p=>ctx.fillRect(p.x,p.y,p.width,p.height));

  // Player
  ctx.fillStyle="blue";
  ctx.fillRect(player.x,player.y,player.width,player.height);

  // Bullets
  ctx.fillStyle="yellow";
  bullets.forEach(b=>ctx.fillRect(b.x,b.y,10,10));

  // Enemies
  ctx.fillStyle="red";
  enemies.forEach(e=>ctx.fillRect(e.x,e.y,e.width,e.height));

  // Power-ups
  ctx.fillStyle="lime";
  powerUps.forEach(p=>ctx.fillRect(p.x,p.y,p.width,p.height));

  // Ground
  ctx.fillStyle="#555";
  ctx.fillRect(0,canvas.height-50,canvas.width,50);
}

// --- Reset Player ---
function resetPlayer(){ player.x = 100; player.y = canvas.height-150; player.dy=0; }

// --- Game Loop ---
function loop(){ update(); draw(); requestAnimationFrame(loop); }
loop();
</script>
</body>
</html>
