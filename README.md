<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Aero World</title>

<style>
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#62dfff}
body{font-family:Arial,sans-serif;touch-action:none}
canvas{display:block;width:100%;height:100%}

#hud{
 position:fixed;
 inset:0;
 pointer-events:none;
 color:white;
}

.glass{
 background:linear-gradient(135deg,rgba(255,255,255,.45),rgba(80,210,255,.18));
 border:2px solid rgba(255,255,255,.75);
 box-shadow:0 8px 25px rgba(0,100,160,.2),inset 0 1px 0 rgba(255,255,255,.8);
 backdrop-filter:blur(9px);
}

#title{
 position:absolute;
 top:14px;
 left:50%;
 transform:translateX(-50%);
 padding:9px 20px;
 border-radius:22px;
 font-size:20px;
 font-weight:bold;
 text-shadow:0 2px 4px #16749b;
}

#stats{
 position:absolute;
 top:14px;
 left:14px;
 padding:10px 14px;
 border-radius:18px;
 font-size:14px;
 text-shadow:0 1px 3px #146e92;
}

#message{
 position:absolute;
 left:50%;
 bottom:24px;
 transform:translateX(-50%);
 padding:10px 16px;
 border-radius:18px;
 font-size:13px;
 white-space:nowrap;
 text-shadow:0 1px 3px #16749b;
}

#mount{
 position:absolute;
 right:14px;
 top:14px;
 padding:10px 14px;
 border-radius:18px;
 display:none;
 font-weight:bold;
}

#joy{
 position:absolute;
 left:18px;
 bottom:20px;
 width:120px;
 height:120px;
 border-radius:50%;
 background:rgba(255,255,255,.17);
 border:2px solid rgba(255,255,255,.65);
 pointer-events:auto;
}

#stick{
 position:absolute;
 width:55px;
 height:55px;
 left:50%;
 top:50%;
 transform:translate(-50%,-50%);
 border-radius:50%;
 background:rgba(255,255,255,.65);
 border:2px solid white;
 box-shadow:0 4px 15px rgba(0,100,150,.3);
}

#buttons{
 position:absolute;
 right:18px;
 bottom:22px;
 display:flex;
 gap:12px;
 pointer-events:auto;
}

button{
 width:72px;
 height:72px;
 border-radius:50%;
 border:2px solid white;
 background:linear-gradient(#ffffffaa,#63dcff99);
 color:#08769d;
 font-weight:bold;
 box-shadow:0 6px 18px rgba(0,100,150,.3);
}

button:active{
 transform:scale(.92);
}

#crosshair{
 position:absolute;
 left:50%;
 top:50%;
 width:9px;
 height:9px;
 transform:translate(-50%,-50%);
 border:2px solid white;
 border-radius:50%;
 box-shadow:0 0 8px #087da9;
}
</style>
</head>

<body>

<canvas id="game"></canvas>

<div id="hud">

 <div id="title" class="glass">
   🌊 AERO WORLD
 </div>

 <div id="stats" class="glass">
   💎 <span id="score">0</span>
   &nbsp; 🌊 <span id="speed">0</span>
 </div>

 <div id="mount" class="glass">
   🐬 DOLPHIN RIDE
 </div>

 <div id="message" class="glass">
   Explore the ocean • Find the dolphin 🐬
 </div>

 <div id="crosshair"></div>

 <div id="joy">
   <div id="stick"></div>
 </div>

 <div id="buttons">
   <button id="jump">JUMP</button>
   <button id="ride">RIDE</button>
 </div>

</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

let W=innerWidth;
let H=innerHeight;
let DPR=Math.min(devicePixelRatio||1,2);

function resize(){
 W=innerWidth;
 H=innerHeight;
 canvas.width=W*DPR;
 canvas.height=H*DPR;
 canvas.style.width=W+"px";
 canvas.style.height=H+"px";
 ctx.setTransform(DPR,0,0,DPR,0,0);
}

addEventListener("resize",resize);
resize();

const scoreEl=document.getElementById("score");
const speedEl=document.getElementById("speed");
const message=document.getElementById("message");
const mountUI=document.getElementById("mount");

let score=0;
let time=0;

const player={
 x:0,
 z:0,
 y:0,
 vx:0,
 vz:0,
 vy:0,
 riding:false,
 jump:false
};

const dolphin={
 x:250,
 z:-250,
 y:0,
 phase:0
};

const islands=[];
const bubbles=[];
const fish=[];
const clouds=[];
const trees=[];
const crystals=[];

function rand(a,b){
 return a+Math.random()*(b-a);
}

/* islands */

for(let i=0;i<18;i++){
 islands.push({
  x:rand(-1500,1500),
  z:rand(-1700,800),
  r:rand(90,210),
  hue:rand(0,1)
 });
}

/* bubbles */

for(let i=0;i<140;i++){
 bubbles.push({
  x:rand(-1800,1800),
  z:rand(-1800,1000),
  y:rand(0,500),
  s:rand(2,9),
  speed:rand(.2,1)
 });
}

/* fish */

for(let i=0;i<45;i++){
 fish.push({
  x:rand(-1600,1600),
  z:rand(-1600,1000),
  y:rand(30,280),
  dir:Math.random()<.5?-1:1,
  speed:rand(.3,1.2),
  size:rand(7,15)
 });
}

/* clouds */

for(let i=0;i<25;i++){
 clouds.push({
  x:rand(-2000,2000),
  z:rand(-2500,1000),
  y:rand(350,700),
  s:rand(.7,1.8)
 });
}

/* trees */

for(let i=0;i<80;i++){
 let a=rand(0,Math.PI*2);
 let d=rand(80,1600);

 trees.push({
  x:Math.cos(a)*d,
  z:Math.sin(a)*d,
  s:rand(.7,1.4)
 });
}

/* crystals */

for(let i=0;i<70;i++){
 crystals.push({
  x:rand(-1600,1600),
  z:rand(-1700,900),
  y:0,
  rot:rand(0,6)
 });
}


/* joystick */

const joy=document.getElementById("joy");
const stick=document.getElementById("stick");

let joyX=0;
let joyY=0;
let joyActive=false;

function setJoy(e){
 const r=joy.getBoundingClientRect();
 let x=e.clientX-r.left-r.width/2;
 let y=e.clientY-r.top-r.height/2;

 const max=43;
 const len=Math.hypot(x,y);

 if(len>max){
  x=x/len*max;
  y=y/len*max;
 }

 joyX=x/max;
 joyY=y/max;

 stick.style.left=`calc(50% + ${x}px)`;
 stick.style.top=`calc(50% + ${y}px)`;
}

joy.addEventListener("pointerdown",e=>{
 joyActive=true;
 joy.setPointerCapture(e.pointerId);
 setJoy(e);
});

joy.addEventListener("pointermove",e=>{
 if(joyActive)setJoy(e);
});

joy.addEventListener("pointerup",()=>{
 joyActive=false;
 joyX=0;
 joyY=0;
 stick.style.left="50%";
 stick.style.top="50%";
});


/* keyboard */

const keys={};

addEventListener("keydown",e=>{
 keys[e.key.toLowerCase()]=true;

 if(e.key===" "){
  jump();
 }

 if(e.key.toLowerCase()==="e"){
  toggleRide();
 }
});

addEventListener("keyup",e=>{
 keys[e.key.toLowerCase()]=false;
});


/* movement */

function jump(){

 if(player.riding){
  player.vy=15;
  player.jump=true;
  return;
 }

 if(player.y<=1){
  player.vy=12;
  player.jump=true;
 }
}

document.getElementById("jump").addEventListener("pointerdown",jump);

function nearDolphin(){

 let dx=player.x-dolphin.x;
 let dz=player.z-dolphin.z;

 return Math.hypot(dx,dz)<100;
}

function toggleRide(){

 if(!player.riding && nearDolphin()){
  player.riding=true;
  mountUI.style.display="block";
  message.textContent="🐬 You're riding the dolphin! Use the joystick to swim.";
 }

 else if(player.riding){
  player.riding=false;
  player.vy=5;
  mountUI.style.display="none";
  message.textContent="The dolphin is ready whenever you are 🐬";
 }
}

document.getElementById("ride").addEventListener("pointerdown",toggleRide);


/* projection */

function project(x,y,z){

 const dx=x-player.x;
 const dz=z-player.z;

 const depth=dz+700;

 if(depth<20)return null;

 const scale=650/depth;

 return {
  x:W/2+dx*scale,
  y:H*.48-y*scale,
  s:scale
 };
}


/* sky */

function drawSky(){

 let g=ctx.createLinearGradient(0,0,0,H);
 g.addColorStop(0,"#4bcfff");
 g.addColorStop(.48,"#baf6ff");
 g.addColorStop(1,"#eaffff");

 ctx.fillStyle=g;
 ctx.fillRect(0,0,W,H);

 /* sun */

 const sun=ctx.createRadialGradient(
  W*.78,H*.15,10,
  W*.78,H*.15,180
 );

 sun.addColorStop(0,"rgba(255,255,255,.95)");
 sun.addColorStop(1,"rgba(255,255,255,0)");

 ctx.fillStyle=sun;
 ctx.fillRect(0,0,W,H);
}


/* clouds */

function drawClouds(){

 for(const c of clouds){

  const p=project(c.x,c.y,c.z);

  if(!p)continue;
  if(p.x<-300||p.x>W+300)continue;

  ctx.save();

  ctx.globalAlpha=.65;
  ctx.fillStyle="white";

  let s=70*p.s*p.s;

  ctx.beginPath();

  ctx.arc(p.x,p.y,s*.45,0,Math.PI*2);
  ctx.arc(p.x+s*.45,p.y+5,s*.55,0,Math.PI*2);
  ctx.arc(p.x+s*.9,p.y+12,s*.38,0,Math.PI*2);

  ctx.fill();

  ctx.restore();
 }
}


/* ocean */

function drawOcean(){

 let horizon=H*.48;

 ctx.fillStyle="#16bce8";
 ctx.fillRect(0,horizon,W,H-horizon);

 for(let i=0;i<35;i++){

  let y=horizon+i*i*.75;

  ctx.strokeStyle=`rgba(255,255,255,${.08+i/500})`;
  ctx.lineWidth=2;

  ctx.beginPath();

  for(let x=0;x<W;x+=30){

   let wave=Math.sin(x*.025+i+time*.02)*3;

   if(x===0)ctx.moveTo(x,y+wave);
   else ctx.lineTo(x,y+wave);
  }

  ctx.stroke();
 }
}


/* islands */

function drawIslands(){

 for(const island of islands){

  const p=project(island.x,0,island.z);

  if(!p)continue;

  let r=island.r*p.s;

  if(r<2)continue;

  ctx.save();

  /* shadow */

  ctx.fillStyle="rgba(0,70,100,.25)";
  ctx.beginPath();
  ctx.ellipse(p.x+5,p.y+12,r*1.1,r*.35,0,0,Math.PI*2);
  ctx.fill();

  /* sand */

  ctx.fillStyle="#f4e59d";
  ctx.beginPath();
  ctx.ellipse(p.x,p.y,r,r*.48,0,0,Math.PI*2);
  ctx.fill();

  /* grass */

  ctx.fillStyle="#54c96b";
  ctx.beginPath();
  ctx.ellipse(p.x,p.y-8,r*.82,r*.36,0,0,Math.PI*2);
  ctx.fill();

  /* futuristic dome */

  if(r>35){

   ctx.fillStyle="rgba(130,240,255,.7)";
   ctx.strokeStyle="rgba(255,255,255,.8)";
   ctx.lineWidth=2;

   ctx.beginPath();
   ctx.ellipse(p.x,p.y-r*.15,r*.25,r*.16,0,Math.PI,Math.PI*2);
   ctx.fill();
   ctx.stroke();

   ctx.fillStyle="#d9ffff";
   ctx.beginPath();
   ctx.ellipse(p.x,p.y-r*.17,r*.06,r*.04,0,0,Math.PI*2);
   ctx.fill();
  }

  ctx.restore();
 }
}


/* trees */

function drawTrees(){

 for(const t of trees){

  const p=project(t.x,0,t.z);

  if(!p)continue;

  let s=65*t.s*p.s;

  if(s<3)continue;

  ctx.save();

  ctx.fillStyle="#986b38";
  ctx.fillRect(p.x-s*.08,p.y-s*.5,s*.16,s*.55);

  ctx.fillStyle="#36bd62";

  ctx.beginPath();
  ctx.arc(p.x,p.y-s*.65,s*.35,0,Math.PI*2);
  ctx.arc(p.x-s*.25,p.y-s*.52,s*.3,0,Math.PI*2);
  ctx.arc(p.x+s*.25,p.y-s*.52,s*.3,0,Math.PI*2);
  ctx.fill();

  ctx.restore();
 }
}


/* crystals */

function drawCrystals(){

 for(const c of crystals){

  const p=project(c.x,0,c.z);

  if(!p)continue;

  let s=25*p.s;

  if(s<2)continue;

  ctx.save();

  ctx.translate(p.x,p.y);
  ctx.rotate(c.rot+time*.001);

  ctx.fillStyle="rgba(120,245,255,.75)";
  ctx.strokeStyle="white";

  ctx.beginPath();
  ctx.moveTo(0,-s);
  ctx.lineTo(s*.35,0);
  ctx.lineTo(0,s*.5);
  ctx.lineTo(-s*.35,0);
  ctx.closePath();

  ctx.fill();
  ctx.stroke();

  ctx.restore();
 }
}


/* bubbles */

function drawBubbles(){

 for(const b of bubbles){

  b.y+=b.speed;

  if(b.y>500)b.y=0;

  const p=project(b.x,b.y,b.z);

  if(!p)continue;

  let r=Math.max(2,b.s*p.s);

  if(r<1)continue;

  ctx.strokeStyle="rgba(255,255,255,.55)";
  ctx.lineWidth=1.5;

  ctx.beginPath();
  ctx.arc(p.x,p.y,r,0,Math.PI*2);
  ctx.stroke();
 }
}


/* fish */

function drawFish(){

 for(const f of fish){

  f.x+=f.dir*f.speed;

  if(f.x>1800)f.x=-1800;
  if(f.x<-1800)f.x=1800;

  const p=project(f.x,f.y,f.z);

  if(!p)continue;

  let s=f.size*p.s;

  if(s<2)continue;

  ctx.save();

  ctx.translate(p.x,p.y);

  if(f.dir<0)ctx.scale(-1,1);

  ctx.fillStyle="#ffb83d";

  ctx.beginPath();
  ctx.ellipse(0,0,s,s*.55,0,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.moveTo(-s,0);
  ctx.lineTo(-s*1.6,-s*.7);
  ctx.lineTo(-s*1.6,s*.7);
  ctx.closePath();
  ctx.fill();

  ctx.fillStyle="white";
  ctx.beginPath();
  ctx.arc(s*.45,-s*.15,s*.18,0,Math.PI*2);
  ctx.fill();

  ctx.fillStyle="#174f69";
  ctx.beginPath();
  ctx.arc(s*.48,-s*.15,s*.08,0,Math.PI*2);
  ctx.fill();

  ctx.restore();
 }
}


/* dolphin */

function drawDolphin(){

 let bob=Math.sin(time*.004)*15;

 let x=player.riding?player.x:dolphin.x;
 let z=player.riding?player.z:dolphin.z;

 if(!player.riding){
  dolphin.phase+=.04;
  x=dolphin.x+Math.sin(dolphin.phase)*30;
  z=dolphin.z+Math.cos(dolphin.phase*.7)*25;
 }

 let p=project(x,45+bob,z);

 if(!p)return;

 let s=90*p.s;

 if(s<3)return;

 ctx.save();
 ctx.translate(p.x,p.y);

 ctx.fillStyle="#6ec9e5";
 ctx.strokeStyle="#dfffff";
 ctx.lineWidth=Math.max(1,s*.035);

 /* body */

 ctx.beginPath();
 ctx.ellipse(0,0,s,s*.36,0,0,Math.PI*2);
 ctx.fill();
 ctx.stroke();

 /* nose */

 ctx.beginPath();
 ctx.moveTo(s*.75,-s*.05);
 ctx.quadraticCurveTo(s*1.35,-s*.08,s*1.55,0);
 ctx.quadraticCurveTo(s*1.25,s*.1,s*.75,s*.1);
 ctx.fill();
 ctx.stroke();

 /* dorsal fin */

 ctx.beginPath();
 ctx.moveTo(-s*.05,-s*.25);
 ctx.lineTo(s*.18,-s*.75);
 ctx.lineTo(s*.35,-s*.25);
 ctx.fill();

 /* tail */

 ctx.beginPath();
 ctx.moveTo(-s*.75,0);
 ctx.lineTo(-s*1.25,-s*.45);
 ctx.lineTo(-s*1.05,0);
 ctx.lineTo(-s*1.25,s*.45);
 ctx.closePath();
 ctx.fill();

 /* eye */

 ctx.fillStyle="#123d58";
 ctx.beginPath();
 ctx.arc(s*.58,-s*.14,s*.07,0,Math.PI*2);
 ctx.fill();

 /* highlight */

 ctx.fillStyle="rgba(255,255,255,.45)";
 ctx.beginPath();
 ctx.ellipse(-s*.15,-s*.13,s*.42,s*.08,0,0,Math.PI*2);
 ctx.fill();

 /* rider */

 if(player.riding){

  ctx.fillStyle="#ffffff";

  ctx.beginPath();
  ctx.arc(-s*.05,-s*.6,s*.12,0,Math.PI*2);
  ctx.fill();

  ctx.strokeStyle="#ffffff";
  ctx.lineWidth=s*.08;

  ctx.beginPath();
  ctx.moveTo(-s*.05,-s*.48);
  ctx.lineTo(-s*.05,-s*.15);
  ctx.stroke();
 }

 ctx.restore();
}


/* update */

function update(){

 time++;

 let forward=0;
 let strafe=0;

 if(Math.abs(joyY)>.05)forward=-joyY;
 if(Math.abs(joyX)>.05)strafe=joyX;

 if(keys["w"]||keys["arrowup"])forward=1;
 if(keys["s"]||keys["arrowdown"])forward=-1;
 if(keys["a"]||keys["arrowleft"])strafe=-1;
 if(keys["d"]||keys["arrowright"])strafe=1;

 let maxSpeed=player.riding?8:4;

 player.vx+=(strafe*maxSpeed-player.vx)*.12;
 player.vz+=(forward*maxSpeed-player.vz)*.12;

 player.x+=player.vx;
 player.z+=player.vz;

 if(player.riding){

  dolphin.x=player.x;
  dolphin.z=player.z;
  dolphin.y=player.y;

  player.y+=player.vy;

  player.vy-=.55;

  if(player.y<20){
   player.y=20;
   player.vy=0;
  }

 }else{

  player.y+=player.vy;

  player.vy-=.6;

  if(player.y<0){
   player.y=0;
   player.vy=0;
  }
 }

 /* collect crystals */

 for(const c of crystals){

  let dx=player.x-c.x;
  let dz=player.z-c.z;

  if(Math.hypot(dx,dz)<35){

   c.x=rand(-1600,1600);
   c.z=rand(-1700,900);

   score++;
   scoreEl.textContent=score;
  }
 }

 /* dolphin proximity */

 if(!player.riding){

  if(nearDolphin()){

   message.textContent="🐬 Press RIDE or E to ride the dolphin!";

  }else{

   message.textContent="Explore the ocean • Find the dolphin 🐬";
  }
 }

 let sp=Math.round(Math.hypot(player.vx,player.vz)*10);
 speedEl.textContent=sp;
}


/* render */

function render(){

 drawSky();
 drawClouds();
 drawOcean();

 /*
 Objects are intentionally drawn in a simple
 painter's order to keep the game lightweight
 and mobile-friendly.
 */

 drawIslands();
 drawTrees();
 drawCrystals();
 drawFish();
 drawBubbles();
 drawDolphin();

 /* water shine */

 ctx.fillStyle="rgba(255,255,255,.08)";

 for(let i=0;i<20;i++){

  let x=(i*137+time*.35)%W;
  let y=H*.52+(i*47)%((H*.48));

  ctx.fillRect(x,y,80,2);
 }
}


/* game loop */

function loop(){

 update();
 render();

 requestAnimationFrame(loop);
}

loop();


/* autosave */

setInterval(()=>{

 localStorage.setItem("aeroWorldSave",JSON.stringify({
  x:player.x,
  z:player.z,
  score
 }));

},2000);


/* load */

try{

 const save=JSON.parse(localStorage.getItem("aeroWorldSave"));

 if(save){

  player.x=save.x||0;
  player.z=save.z||0;
  score=save.score||0;

  scoreEl.textContent=score;
 }
}catch(e){}


/* prevent page scrolling */

document.addEventListener("touchmove",e=>{
 e.preventDefault();
},{passive:false});

</script>

</body>
</html>
