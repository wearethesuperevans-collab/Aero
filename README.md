<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<title>Aero Ocean 3D</title>

<style>
*{
  box-sizing:border-box;
  -webkit-tap-highlight-color:transparent;
  user-select:none;
}

html,body{
  margin:0;
  width:100%;
  height:100%;
  overflow:hidden;
  background:#55dfff;
  font-family:Arial,sans-serif;
}

#game{
  position:fixed;
  inset:0;
  width:100%;
  height:100%;
  display:block;
  touch-action:none;
}

#ui{
  position:fixed;
  inset:0;
  pointer-events:none;
}

#title{
  position:absolute;
  top:18px;
  left:20px;
  color:white;
  font-size:24px;
  font-weight:900;
  text-shadow:0 3px 5px #168bb0;
}

#hint{
  position:absolute;
  top:53px;
  left:21px;
  color:white;
  font-size:12px;
  text-shadow:0 2px 4px #168bb0;
}

#joystick{
  pointer-events:auto;
  position:absolute;
  left:24px;
  bottom:25px;
  width:125px;
  height:125px;
  border-radius:50%;
  background:rgba(255,255,255,.22);
  border:3px solid rgba(255,255,255,.65);
  box-shadow:0 8px 25px rgba(0,100,150,.18);
}

#stick{
  position:absolute;
  width:58px;
  height:58px;
  left:50%;
  top:50%;
  transform:translate(-50%,-50%);
  border-radius:50%;
  background:rgba(255,255,255,.75);
  border:3px solid white;
  box-shadow:0 4px 15px rgba(0,80,120,.2);
}

.buttons{
  position:absolute;
  right:22px;
  bottom:25px;
  display:flex;
  gap:12px;
  align-items:end;
}

button{
  pointer-events:auto;
  border:0;
  width:75px;
  height:75px;
  border-radius:50%;
  color:white;
  font-size:12px;
  font-weight:900;
  background:linear-gradient(#8ff5ff,#20acd5);
  border:3px solid rgba(255,255,255,.9);
  box-shadow:0 7px 20px rgba(0,80,120,.3);
}

button:active{
  transform:scale(.9);
}

#rotate{
  display:none;
  position:fixed;
  inset:0;
  z-index:20;
  background:linear-gradient(#62eaff,#b7faff);
  align-items:center;
  justify-content:center;
  flex-direction:column;
  color:#087da0;
  text-align:center;
  font-weight:900;
}

#rotate .icon{
  font-size:65px;
  margin-bottom:15px;
}

@media (orientation:portrait) and (max-width:900px){
  #rotate{
    display:flex;
  }
}

@media (max-width:600px){
  #title{font-size:18px}
  #hint{font-size:10px}
}
</style>
</head>

<body>

<canvas id="game"></canvas>

<div id="ui">
  <div id="title">AERO OCEAN</div>
  <div id="hint">Explore • Collect bubbles • Find the dolphin</div>

  <div id="joystick">
    <div id="stick"></div>
  </div>

  <div class="buttons">
    <button id="jump">JUMP</button>
    <button id="ride">RIDE</button>
  </div>
</div>

<div id="rotate">
  <div class="icon">↔️</div>
  <div>TURN YOUR iPAD SIDEWAYS</div>
  <small>Landscape mode gives you the full 3D world.</small>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>
/* =========================================================
   AERO OCEAN
   Single-file 3D Frutiger-Aero-inspired game
   ========================================================= */

const canvas = document.getElementById("game");

const renderer = new THREE.WebGLRenderer({
  canvas,
  antialias:true,
  powerPreference:"high-performance"
});

renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.setSize(innerWidth,innerHeight);
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;

const scene = new THREE.Scene();

scene.background = new THREE.Color(0x5ddfff);
scene.fog = new THREE.Fog(0x75e7ff,90,360);

const camera = new THREE.PerspectiveCamera(
  65,
  innerWidth/innerHeight,
  .1,
  700
);

camera.position.set(0,7,18);

const clock = new THREE.Clock();

/* ---------------- LIGHT ---------------- */

const hemi = new THREE.HemisphereLight(
  0xdfffff,
  0x4aa77b,
  2.2
);

scene.add(hemi);

const sun = new THREE.DirectionalLight(
  0xffffff,
  3.2
);

sun.position.set(-70,100,50);
sun.castShadow=true;
sun.shadow.mapSize.width=2048;
sun.shadow.mapSize.height=2048;
scene.add(sun);

/* ---------------- MATERIALS ---------------- */

const blue = new THREE.MeshStandardMaterial({
  color:0x22bde8,
  roughness:.22,
  metalness:.05
});

const white = new THREE.MeshStandardMaterial({
  color:0xffffff,
  roughness:.3
});

const green = new THREE.MeshStandardMaterial({
  color:0x54d86a,
  roughness:.8
});

const darkGreen = new THREE.MeshStandardMaterial({
  color:0x1d9d58,
  roughness:.8
});

const yellow = new THREE.MeshStandardMaterial({
  color:0xffec54,
  roughness:.25
});

const pink = new THREE.MeshStandardMaterial({
  color:0xff76c8,
  roughness:.3
});

const glass = new THREE.MeshPhysicalMaterial({
  color:0xbefaff,
  transparent:true,
  opacity:.42,
  roughness:.05,
  metalness:.05
});

/* ---------------- OCEAN ---------------- */

const oceanGeo = new THREE.PlaneGeometry(700,700,100,100);

const oceanMat = new THREE.MeshStandardMaterial({
  color:0x19bfe9,
  roughness:.18,
  metalness:.05
});

const ocean = new THREE.Mesh(oceanGeo,oceanMat);

ocean.rotation.x=-Math.PI/2;
ocean.position.y=-1.5;
ocean.receiveShadow=true;

scene.add(ocean);

/* ---------------- WORLD ---------------- */

function island(x,z,s=1){

  const group=new THREE.Group();

  const base=new THREE.Mesh(
    new THREE.CylinderGeometry(13*s,18*s,4*s,32),
    new THREE.MeshStandardMaterial({
      color:0x3dba62,
      roughness:.9
    })
  );

  base.position.y=.2;
  base.castShadow=true;
  base.receiveShadow=true;
  group.add(base);

  const grass=new THREE.Mesh(
    new THREE.CylinderGeometry(13*s,15*s,1.2*s,32),
    green
  );

  grass.position.y=2.1*s;
  grass.castShadow=true;
  group.add(grass);

  group.position.set(x,0,z);
  scene.add(group);

  return group;
}

island(0,0,1.4);
island(-70,-40,1.1);
island(65,-65,1.25);
island(85,55,.9);
island(-90,65,1);

/* ---------------- TREES ---------------- */

function palm(x,z,s=1){

  const g=new THREE.Group();

  const trunk=new THREE.Mesh(
    new THREE.CylinderGeometry(.35*s,.65*s,7*s,10),
    new THREE.MeshStandardMaterial({
      color:0xc98a48,
      roughness:.9
    })
  );

  trunk.position.y=5*s;
  trunk.castShadow=true;
  g.add(trunk);

  for(let i=0;i<7;i++){

    const leaf=new THREE.Mesh(
      new THREE.CapsuleGeometry(.22*s,3*s,4,8),
      darkGreen
    );

    const a=i*Math.PI*2/7;

    leaf.position.set(
      Math.cos(a)*1.8*s,
      8*s,
      Math.sin(a)*1.8*s
    );

    leaf.rotation.z=Math.cos(a)*.8;
    leaf.rotation.x=Math.sin(a)*.8;

    g.add(leaf);
  }

  g.position.set(x,0,z);
  scene.add(g);
}

for(let i=0;i<18;i++){

  const a=Math.random()*Math.PI*2;
  const r=8+Math.random()*10;

  palm(
    Math.cos(a)*r,
    Math.sin(a)*r,
    .7+Math.random()*.5
  );
}

/* ---------------- CLOUDS ---------------- */

function cloud(x,y,z,s){

  const g=new THREE.Group();

  for(let i=0;i<7;i++){

    const p=new THREE.Mesh(
      new THREE.SphereGeometry(
        (2+Math.random()*2)*s,
        20,
        20
      ),
      white
    );

    p.position.set(
      (i-3)*2*s,
      Math.random()*1.5*s,
      Math.random()*2*s
    );

    g.add(p);
  }

  g.position.set(x,y,z);
  scene.add(g);

  return g;
}

cloud(-45,35,-80,2.3);
cloud(50,42,-120,3);
cloud(100,28,20,1.7);
cloud(-120,32,60,2);

/* ---------------- AERO CHARACTER ---------------- */

function createAeroGuy(color=0xffffff){

  const g=new THREE.Group();

  const body=new THREE.Mesh(
    new THREE.CapsuleGeometry(.75,1.8,8,16),
    new THREE.MeshStandardMaterial({
      color,
      roughness:.25,
      metalness:.05
    })
  );

  body.position.y=2.1;
  body.castShadow=true;
  g.add(body);

  const head=new THREE.Mesh(
    new THREE.SphereGeometry(.78,24,24),
    white
  );

  head.position.y=4;
  head.castShadow=true;
  g.add(head);

  const visor=new THREE.Mesh(
    new THREE.SphereGeometry(.62,24,16),
    new THREE.MeshPhysicalMaterial({
      color:0x28d9ff,
      roughness:.05,
      metalness:.2,
      transparent:true,
      opacity:.72
    })
  );

  visor.scale.set(1,.58,.3);
  visor.position.set(0,4.08,-.57);
  g.add(visor);

  const eyeMat=new THREE.MeshStandardMaterial({
    color:0x064d75
  });

  for(const x of [-.22,.22]){

    const eye=new THREE.Mesh(
      new THREE.SphereGeometry(.09,12,12),
      eyeMat
    );

    eye.position.set(x,4.05,-.82);
    g.add(eye);
  }

  for(const side of [-1,1]){

    const arm=new THREE.Mesh(
      new THREE.CapsuleGeometry(.22,1.25,6,10),
      white
    );

    arm.position.set(side*1,2.25,0);
    arm.rotation.z=side*.25;
    arm.castShadow=true;

    g.add(arm);

    const hand=new THREE.Mesh(
      new THREE.SphereGeometry(.27,12,12),
      yellow
    );

    hand.position.set(side*1.2,1.5,0);
    g.add(hand);
  }

  const belt=new THREE.Mesh(
    new THREE.TorusGeometry(.62,.11,10,32),
    blue
  );

  belt.rotation.x=Math.PI/2;
  belt.position.y=1.55;
  g.add(belt);

  return g;
}

const player=createAeroGuy(0xffffff);
player.position.set(0,2.7,7);
scene.add(player);

/* ---------------- OTHER AERO GUYS ---------------- */

const guys=[];

[
  [-8,2,-7],
  [7,2,-12],
  [-20,2,8],
  [20,2,2],
  [3,2,15]
].forEach((p,i)=>{

  const guy=createAeroGuy(
    i%2 ? 0xffffff : 0xeefcff
  );

  guy.position.set(p[0],p[1],p[2]);
  guy.scale.setScalar(.9);
  scene.add(guy);
  guys.push(guy);
});

/* ---------------- DOLPHIN ---------------- */

function createDolphin(){

  const d=new THREE.Group();

  const body=new THREE.Mesh(
    new THREE.SphereGeometry(1.4,28,18),
    new THREE.MeshStandardMaterial({
      color:0x5dcae8,
      roughness:.25,
      metalness:.15
    })
  );

  body.scale.set(2.2,.8,1);
  body.castShadow=true;
  d.add(body);

  const nose=new THREE.Mesh(
    new THREE.ConeGeometry(.48,1.7,20),
    blue
  );

  nose.rotation.z=-Math.PI/2;
  nose.position.z=-2.2;
  nose.scale.set(1,.65,1);
  d.add(nose);

  const dorsal=new THREE.Mesh(
    new THREE.ConeGeometry(.45,1.4,16),
    blue
  );

  dorsal.position.y=.9;
  dorsal.rotation.z=Math.PI;
  d.add(dorsal);

  for(const side of [-1,1]){

    const fin=new THREE.Mesh(
      new THREE.ConeGeometry(.38,1.2,16),
      blue
    );

    fin.rotation.z=side*.8;
    fin.rotation.x=-.4;
    fin.position.set(side*1.25,-.1,-.2);
    d.add(fin);
  }

  const tail=new THREE.Group();

  for(const side of [-1,1]){

    const fin=new THREE.Mesh(
      new THREE.ConeGeometry(.55,1.4,16),
      blue
    );

    fin.rotation.z=side*.75;
    fin.position.x=side*.45;
    tail.add(fin);
  }

  tail.position.z=2.1;
  d.add(tail);

  const eyeMat=new THREE.MeshStandardMaterial({
    color:0x073c55
  });

  for(const side of [-1,1]){

    const eye=new THREE.Mesh(
      new THREE.SphereGeometry(.11,12,12),
      eyeMat
    );

    eye.position.set(side*.5,-.1,-1.45);
    d.add(eye);
  }

  d.scale.setScalar(1.15);

  return d;
}

const dolphin=createDolphin();
dolphin.position.set(0,-.1,-20);
scene.add(dolphin);

/* ---------------- BUBBLES ---------------- */

const bubbles=[];

for(let i=0;i<35;i++){

  const b=new THREE.Mesh(
    new THREE.SphereGeometry(.35+Math.random()*.3,16,16),
    new THREE.MeshPhysicalMaterial({
      color:0xdfffff,
      transparent:true,
      opacity:.48,
      roughness:0,
      metalness:0
    })
  );

  b.position.set(
    (Math.random()-.5)*120,
    Math.random()*10,
    (Math.random()-.5)*120
  );

  scene.add(b);
  bubbles.push(b);
}

/* ---------------- COLLECTIBLES ---------------- */

const collectibles=[];

for(let i=0;i<18;i++){

  const ring=new THREE.Mesh(
    new THREE.TorusGeometry(.65,.16,12,30),
    yellow
  );

  ring.position.set(
    (Math.random()-.5)*90,
    3+Math.random()*4,
    (Math.random()-.5)*90
  );

  ring.rotation.x=Math.PI/2;

  scene.add(ring);
  collectibles.push(ring);
}

/* ---------------- MOVEMENT ---------------- */

let riding=false;
let jumping=false;
let velocityY=0;

let moveX=0;
let moveZ=0;

let yaw=0;
let pitch=.18;

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

/* ---------------- JOYSTICK ---------------- */

const joystick=document.getElementById("joystick");
const stick=document.getElementById("stick");

let joyId=null;

joystick.addEventListener("pointerdown",e=>{

  joyId=e.pointerId;
  joystick.setPointerCapture(joyId);

  updateJoystick(e);
});

joystick.addEventListener("pointermove",e=>{

  if(e.pointerId===joyId){
    updateJoystick(e);
  }
});

joystick.addEventListener("pointerup",resetJoystick);
joystick.addEventListener("pointercancel",resetJoystick);

function updateJoystick(e){

  const r=joystick.getBoundingClientRect();

  let x=e.clientX-(r.left+r.width/2);
  let y=e.clientY-(r.top+r.height/2);

  const max=43;

  const len=Math.hypot(x,y);

  if(len>max){
    x=x/len*max;
    y=y/len*max;
  }

  stick.style.transform=
    `translate(calc(-50% + ${x}px),calc(-50% + ${y}px))`;

  moveX=x/max;
  moveZ=y/max;
}

function resetJoystick(){

  joyId=null;
  moveX=0;
  moveZ=0;

  stick.style.transform="translate(-50%,-50%)";
}

/* ---------------- CAMERA SWIPE ---------------- */

let dragging=false;
let lastX=0;
let lastY=0;

canvas.addEventListener("pointerdown",e=>{

  dragging=true;
  lastX=e.clientX;
  lastY=e.clientY;
});

canvas.addEventListener("pointermove",e=>{

  if(!dragging)return;

  const dx=e.clientX-lastX;
  const dy=e.clientY-lastY;

  yaw-=dx*.006;
  pitch-=dy*.004;

  pitch=Math.max(-.25,Math.min(.8,pitch));

  lastX=e.clientX;
  lastY=e.clientY;
});

canvas.addEventListener("pointerup",()=>{
  dragging=false;
});

canvas.addEventListener("pointercancel",()=>{
  dragging=false;
});

/* ---------------- BUTTONS ---------------- */

document.getElementById("jump").addEventListener("pointerdown",jump);

document.getElementById("ride").addEventListener("pointerdown",toggleRide);

function jump(){

  if(jumping)return;

  jumping=true;
  velocityY=riding?18:10;
}

function toggleRide(){

  const dist=player.position.distanceTo(dolphin.position);

  if(!riding){

    if(dist<9){

      riding=true;
      player.position.y=1.8;
      dolphin.position.y=0;
    }

  }else{

    riding=false;
    player.position.y=2.7;
    dolphin.position.y=-.1;
  }
}

/* ---------------- ANIMATION ---------------- */

function animate(){

  requestAnimationFrame(animate);

  const dt=Math.min(clock.getDelta(),.04);
  const t=clock.elapsedTime;

  /* Keyboard */

  let kx=0;
  let kz=0;

  if(keys["a"]||keys["arrowleft"])kx-=1;
  if(keys["d"]||keys["arrowright"])kx+=1;
  if(keys["w"]||keys["arrowup"])kz-=1;
  if(keys["s"]||keys["arrowdown"])kz+=1;

  moveX=Math.abs(kx)>0?kx:moveX;
  moveZ=Math.abs(kz)>0?kz:moveZ;

  const strength=Math.min(
    1,
    Math.hypot(moveX,moveZ)
  );

  const speed=riding?18:7;

  /* Direction relative to camera */

  const forward=new THREE.Vector3(
    Math.sin(yaw),
    0,
    Math.cos(yaw)
  );

  const right=new THREE.Vector3(
    Math.cos(yaw),
    0,
    -Math.sin(yaw)
  );

  const movement=new THREE.Vector3();

  movement.addScaledVector(
    right,
    moveX
  );

  movement.addScaledVector(
    forward,
    moveZ
  );

  if(movement.lengthSq()>0){

    movement.normalize();

    const targetRotation=
      Math.atan2(
        movement.x,
        movement.z
      );

    player.rotation.y=THREE.MathUtils.lerp(
      player.rotation.y,
      targetRotation,
      .15
    );

    if(riding){

      dolphin.rotation.y=THREE.MathUtils.lerp(
        dolphin.rotation.y,
        targetRotation,
        .12
      );
    }

    if(riding){

      dolphin.position.addScaledVector(
        movement,
        speed*dt*strength
      );

    }else{

      player.position.addScaledVector(
        movement,
        speed*dt*strength
      );
    }
  }

  /* Jump physics */

  if(jumping){

    velocityY-=28*dt;

    if(riding){

      dolphin.position.y+=velocityY*dt;

      if(dolphin.position.y<=0){

        dolphin.position.y=0;
        velocityY=0;
        jumping=false;
      }

    }else{

      player.position.y+=velocityY*dt;

      if(player.position.y<=2.7){

        player.position.y=2.7;
        velocityY=0;
        jumping=false;
      }
    }
  }

  /* Dolphin swimming animation */

  dolphin.position.y +=
    Math.sin(t*3.5)*.015;

  dolphin.rotation.z=
    Math.sin(t*3)*.04;

  /* Character idle animation */

  guys.forEach((g,i)=>{

    g.position.y=
      2+
      Math.sin(t*1.8+i)*.12;

    g.rotation.y=
      Math.sin(t*.5+i)*.25;
  });

  /* Bubble animation */

  bubbles.forEach((b,i)=>{

    b.position.y+=dt*(.4+(i%3)*.15);

    if(b.position.y>15){
      b.position.y=-1;
    }

    b.scale.setScalar(
      1+Math.sin(t*2+i)*.08
    );
  });

  /* Rings */

  collectibles.forEach((r,i)=>{

    r.rotation.z+=dt*1.5;
    r.rotation.y+=dt;

    r.position.y+=
      Math.sin(t*2+i)*.002;
  });

  /* Player/dolphin relationship */

  if(riding){

    player.position.copy(
      dolphin.position
    );

    player.position.y+=2;

    player.rotation.y=
      dolphin.rotation.y;
  }

  /* Camera */

  const target=riding?dolphin:player;

  const distance=riding?18:14;

  const camX=
    target.position.x-
    Math.sin(yaw)*distance;

  const camZ=
    target.position.z-
    Math.cos(yaw)*distance;

  const camY=
    target.position.y+
    6+
    pitch*8;

  camera.position.x=THREE.MathUtils.lerp(
    camera.position.x,
    camX,
    .08
  );

  camera.position.y=THREE.MathUtils.lerp(
    camera.position.y,
    camY,
    .08
  );

  camera.position.z=THREE.MathUtils.lerp(
    camera.position.z,
    camZ,
    .08
  );

  camera.lookAt(
    target.position.x,
    target.position.y+1.5,
    target.position.z
  );

  /* Water movement */

  ocean.position.y=
    -1.5+
    Math.sin(t)*.04;

  renderer.render(scene,camera);
}

animate();

/* ---------------- RESIZE ---------------- */

function resize(){

  const w=innerWidth;
  const h=innerHeight;

  camera.aspect=w/h;
  camera.updateProjectionMatrix();

  renderer.setSize(w,h);
  renderer.setPixelRatio(Math.min(devicePixelRatio,2));
}

addEventListener("resize",resize);
addEventListener("orientationchange",resize);

resize();
</script>

</body>
</html>
