<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">

<meta name="viewport"
content="width=device-width,
initial-scale=1,
maximum-scale=1,
user-scalable=no,
viewport-fit=cover">

<title>Frutiger Aero World 3D</title>

<style>
*{
  box-sizing:border-box;
  user-select:none;
  -webkit-user-select:none;
  -webkit-tap-highlight-color:transparent;
}

html,body{
  margin:0;
  width:100%;
  height:100%;
  overflow:hidden;
  background:#59dcff;
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
  text-shadow:0 3px 7px #087da8;
}

#subtitle{
  position:absolute;
  top:51px;
  left:21px;
  color:white;
  font-size:12px;
  text-shadow:0 2px 5px #087da8;
}

#joystick{
  position:absolute;
  left:22px;
  bottom:22px;
  width:130px;
  height:130px;
  border-radius:50%;
  background:rgba(255,255,255,.22);
  border:3px solid rgba(255,255,255,.75);
  box-shadow:
    0 8px 25px rgba(0,90,140,.22),
    inset 0 0 20px rgba(255,255,255,.25);
  pointer-events:auto;
  touch-action:none;
}

#stick{
  position:absolute;
  left:50%;
  top:50%;
  width:60px;
  height:60px;
  transform:translate(-50%,-50%);
  border-radius:50%;
  background:linear-gradient(
    145deg,
    rgba(255,255,255,.95),
    rgba(180,244,255,.7)
  );
  border:3px solid white;
  box-shadow:0 5px 18px rgba(0,100,150,.25);
}

#buttons{
  position:absolute;
  right:22px;
  bottom:22px;
  display:flex;
  gap:12px;
}

button{
  width:78px;
  height:78px;
  border-radius:50%;
  border:3px solid white;
  color:white;
  font-weight:900;
  font-size:11px;
  background:linear-gradient(
    #91f7ff,
    #20abd5
  );
  box-shadow:0 7px 22px rgba(0,90,130,.3);
  pointer-events:auto;
  touch-action:none;
}

button:active{
  transform:scale(.9);
}

#rotate{
  display:none;
  position:fixed;
  inset:0;
  z-index:99;
  background:linear-gradient(
    #5ee3ff,
    #c5fbff
  );
  align-items:center;
  justify-content:center;
  flex-direction:column;
  text-align:center;
  color:#087ba1;
  font-weight:900;
}

#rotateIcon{
  font-size:70px;
  margin-bottom:15px;
}

@media (orientation:portrait) and (max-width:900px){
  #rotate{
    display:flex;
  }
}
</style>
</head>

<body>

<canvas id="game"></canvas>

<div id="ui">

  <div id="title">
    FRUTIGER AERO WORLD
  </div>

  <div id="subtitle">
    Explore the ocean • Find the dolphin
  </div>

  <div id="joystick">
    <div id="stick"></div>
  </div>

  <div id="buttons">
    <button id="jump">JUMP</button>
    <button id="ride">RIDE</button>
  </div>

</div>

<div id="rotate">
  <div id="rotateIcon">↔️</div>
  <div>TURN YOUR iPAD SIDEWAYS</div>
  <small>Landscape gives you the full world.</small>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

/* =========================================================
   FRUTIGER AERO WORLD
   ========================================================= */

const canvas =
document.getElementById("game");

const renderer =
new THREE.WebGLRenderer({
  canvas:canvas,
  antialias:true,
  powerPreference:"high-performance"
});

renderer.setPixelRatio(
  Math.min(window.devicePixelRatio,2)
);

renderer.setSize(
  window.innerWidth,
  window.innerHeight
);

renderer.shadowMap.enabled=true;
renderer.shadowMap.type=
THREE.PCFSoftShadowMap;

const scene =
new THREE.Scene();

scene.background =
new THREE.Color(0x61ddff);

scene.fog =
new THREE.Fog(
  0x61ddff,
  100,
  420
);

const camera =
new THREE.PerspectiveCamera(
  65,
  innerWidth/innerHeight,
  .1,
  700
);

camera.position.set(
  0,
  7,
  18
);

/* =========================================================
   LIGHT
   ========================================================= */

const hemisphere =
new THREE.HemisphereLight(
  0xe8ffff,
  0x4ba76a,
  2.5
);

scene.add(hemisphere);

const sunlight =
new THREE.DirectionalLight(
  0xffffff,
  3.5
);

sunlight.position.set(
  -80,
  120,
  70
);

sunlight.castShadow=true;

sunlight.shadow.mapSize.width=2048;
sunlight.shadow.mapSize.height=2048;

scene.add(sunlight);

/* =========================================================
   MATERIALS
   ========================================================= */

const blueMaterial =
new THREE.MeshPhysicalMaterial({
  color:0x129fe0,
  roughness:.14,
  metalness:.05,
  clearcoat:1,
  clearcoatRoughness:.08
});

const blueGlass =
new THREE.MeshPhysicalMaterial({
  color:0x42d9ff,
  roughness:.08,
  metalness:.05,
  transmission:.18,
  transparent:true,
  opacity:.94,
  clearcoat:1,
  clearcoatRoughness:.05
});

const cyanMaterial =
new THREE.MeshPhysicalMaterial({
  color:0x00e9f4,
  roughness:.12,
  clearcoat:1
});

const whiteMaterial =
new THREE.MeshPhysicalMaterial({
  color:0xffffff,
  roughness:.28,
  clearcoat:1
});

const greenMaterial =
new THREE.MeshStandardMaterial({
  color:0x43d366,
  roughness:.85
});

const darkGreenMaterial =
new THREE.MeshStandardMaterial({
  color:0x15994e,
  roughness:.85
});

/* =========================================================
   OCEAN
   ========================================================= */

const waterGeometry =
new THREE.PlaneGeometry(
  800,
  800,
  120,
  120
);

const waterMaterial =
new THREE.MeshPhysicalMaterial({
  color:0x13bfe9,
  roughness:.13,
  metalness:.08,
  transmission:.08,
  clearcoat:1
});

const water =
new THREE.Mesh(
  waterGeometry,
  waterMaterial
);

water.rotation.x=-Math.PI/2;
water.position.y=-1.6;

scene.add(water);

/* =========================================================
   ISLAND
   ========================================================= */

function createIsland(x,z,scale){

  const group =
  new THREE.Group();

  const rock =
  new THREE.Mesh(
    new THREE.CylinderGeometry(
      14*scale,
      18*scale,
      5*scale,
      32
    ),
    new THREE.MeshStandardMaterial({
      color:0x48aa61,
      roughness:.9
    })
  );

  rock.position.y=.2;
  rock.castShadow=true;
  rock.receiveShadow=true;

  group.add(rock);

  const grass =
  new THREE.Mesh(
    new THREE.CylinderGeometry(
      14*scale,
      15*scale,
      1.5*scale,
      32
    ),
    greenMaterial
  );

  grass.position.y=
  2.8*scale;

  grass.castShadow=true;

  group.add(grass);

  group.position.set(
    x,
    0,
    z
  );

  scene.add(group);
}

createIsland(0,0,1.4);
createIsland(-70,-45,1.1);
createIsland(65,-65,1.25);
createIsland(90,50,.9);
createIsland(-90,60,1);

/* =========================================================
   FRUTIGER AERO GUY
   Based on the rounded glossy reference
   ========================================================= */

function createAeroGuy(){

  const guy =
  new THREE.Group();

  /* BODY */

  const body =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1.45,
      32,
      24
    ),
    blueGlass
  );

  body.scale.set(
    1.18,
    1.35,
    .9
  );

  body.position.y=2.0;

  body.castShadow=true;

  guy.add(body);

  /* LOWER BODY GLOW */

  const glow =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1.15,
      28,
      20
    ),
    cyanMaterial
  );

  glow.scale.set(
    1.15,
    .42,
    .88
  );

  glow.position.set(
    0,
    1.25,
    -.05
  );

  guy.add(glow);

  /* HEAD */

  const head =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1.0,
      32,
      24
    ),
    blueGlass
  );

  head.position.y=4.05;

  head.castShadow=true;

  guy.add(head);

  /* HEAD HIGHLIGHT */

  const highlight =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .32,
      20,
      16
    ),
    new THREE.MeshBasicMaterial({
      color:0xffffff,
      transparent:true,
      opacity:.72
    })
  );

  highlight.scale.set(
    1.35,
    .45,
    .45
  );

  highlight.position.set(
    -.28,
    4.45,
    -.82
  );

  guy.add(highlight);

  /* FACE GLOW */

  const faceGlow =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .7,
      24,
      18
    ),
    new THREE.MeshBasicMaterial({
      color:0x8fffff,
      transparent:true,
      opacity:.18
    })
  );

  faceGlow.scale.set(
    1,
    .65,
    .25
  );

  faceGlow.position.set(
    0,
    3.95,
    -.83
  );

  guy.add(faceGlow);

  /* EYES */

  const eyeMaterial =
  new THREE.MeshStandardMaterial({
    color:0x00577c,
    roughness:.15
  });

  for(const side of [-1,1]){

    const eye =
    new THREE.Mesh(
      new THREE.SphereGeometry(
        .11,
        16,
        16
      ),
      eyeMaterial
    );

    eye.position.set(
      side*.25,
      4.02,
      -.94
    );

    guy.add(eye);
  }

  /* ARMS */

  for(const side of [-1,1]){

    const arm =
    new THREE.Mesh(
      new THREE.SphereGeometry(
        .55,
        24,
        18
      ),
      blueGlass
    );

    arm.scale.set(
      .55,
      1.35,
      .65
    );

    arm.position.set(
      side*1.35,
      2.1,
      0
    );

    arm.rotation.z=
    side*.15;

    arm.castShadow=true;

    guy.add(arm);
  }

  /* SMALL BLUE FEET */

  for(const side of [-1,1]){

    const foot =
    new THREE.Mesh(
      new THREE.SphereGeometry(
        .42,
        20,
        16
      ),
      blueMaterial
    );

    foot.scale.set(
      1.1,
      .55,
      1.25
    );

    foot.position.set(
      side*.55,
      .85,
      .05
    );

    guy.add(foot);
  }

  return guy;
}

/* =========================================================
   PLAYER
   ========================================================= */

const player =
createAeroGuy();

player.position.set(
  0,
  1.1,
  8
);

scene.add(player);

/* =========================================================
   OTHER AERO GUYS
   ========================================================= */

const otherGuys=[];

const locations=[
  [-8,2,-8],
  [8,2,-13],
  [-20,2,8],
  [20,2,2],
  [4,2,16]
];

locations.forEach((p)=>{

  const guy =
  createAeroGuy();

  guy.scale.setScalar(.75);

  guy.position.set(
    p[0],
    p[1],
    p[2]
  );

  scene.add(guy);

  otherGuys.push(guy);
});

/* =========================================================
   DOLPHIN
   ========================================================= */

function createDolphin(){

  const dolphin =
  new THREE.Group();

  const body =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1.35,
      32,
      22
    ),
    new THREE.MeshPhysicalMaterial({
      color:0x48c8e9,
      roughness:.18,
      metalness:.12,
      clearcoat:1
    })
  );

  body.scale.set(
    2.2,
    .72,
    1
  );

  dolphin.add(body);

  const nose =
  new THREE.Mesh(
    new THREE.ConeGeometry(
      .42,
      1.7,
      24
    ),
    blueMaterial
  );

  nose.rotation.z=-Math.PI/2;

  nose.position.z=-2.15;

  dolphin.add(nose);

  const dorsal =
  new THREE.Mesh(
    new THREE.ConeGeometry(
      .42,
      1.4,
      20
    ),
    blueMaterial
  );

  dorsal.rotation.z=Math.PI;

  dorsal.position.y=.85;

  dolphin.add(dorsal);

  const tail =
  new THREE.Group();

  for(const side of [-1,1]){

    const fin =
    new THREE.Mesh(
      new THREE.ConeGeometry(
        .5,
        1.5,
        20
      ),
      blueMaterial
    );

    fin.rotation.z=
    side*.75;

    fin.position.x=
    side*.45;

    tail.add(fin);
  }

  tail.position.z=2;

  dolphin.add(tail);

  return dolphin;
}

const dolphin =
createDolphin();

dolphin.position.set(
  0,
  0,
  -20
);

scene.add(dolphin);

/* =========================================================
   CLOUDS
   ========================================================= */

function createCloud(x,y,z,s){

  const cloud =
  new THREE.Group();

  for(let i=0;i<8;i++){

    const puff =
    new THREE.Mesh(
      new THREE.SphereGeometry(
        (1.5+Math.random()*1.5)*s,
        20,
        20
      ),
      whiteMaterial
    );

    puff.position.set(
      (i-3.5)*1.7*s,
      Math.random()*1.4*s,
      Math.random()*1.5*s
    );

    cloud.add(puff);
  }

  cloud.position.set(
    x,y,z
  );

  scene.add(cloud);
}

createCloud(-50,38,-80,2);
createCloud(50,45,-120,2.5);
createCloud(105,32,15,1.6);
createCloud(-120,35,70,2);

/* =========================================================
   PALM TREES
   ========================================================= */

function createPalm(x,z,s){

  const tree =
  new THREE.Group();

  const trunk =
  new THREE.Mesh(
    new THREE.CylinderGeometry(
      .3*s,
      .55*s,
      7*s,
      12
    ),
    new THREE.MeshStandardMaterial({
      color:0xc88746,
      roughness:.9
    })
  );

  trunk.position.y=4*s;

  tree.add(trunk);

  for(let i=0;i<8;i++){

    const leaf =
    new THREE.Mesh(
      new THREE.CapsuleGeometry(
        .18*s,
        3*s,
        5,
        8
      ),
      darkGreenMaterial
    );

    const angle=
    i*Math.PI*2/8;

    leaf.position.set(
      Math.cos(angle)*1.7*s,
      7.5*s,
      Math.sin(angle)*1.7*s
    );

    leaf.rotation.z=
    Math.cos(angle)*.9;

    leaf.rotation.x=
    Math.sin(angle)*.9;

    tree.add(leaf);
  }

  tree.position.set(
    x,
    0,
    z
  );

  scene.add(tree);
}

for(let i=0;i<22;i++){

  const angle=
  Math.random()*Math.PI*2;

  const radius=
  8+Math.random()*10;

  createPalm(
    Math.cos(angle)*radius,
    Math.sin(angle)*radius,
    .7+Math.random()*.5
  );
}

/* =========================================================
   BUBBLES
   ========================================================= */

const bubbles=[];

for(let i=0;i<40;i++){

  const bubble =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .25+Math.random()*.35,
      18,
      18
    ),
    new THREE.MeshPhysicalMaterial({
      color:0xe7ffff,
      transparent:true,
      opacity:.48,
      roughness:0,
      clearcoat:1
    })
  );

  bubble.position.set(
    (Math.random()-.5)*140,
    Math.random()*12,
    (Math.random()-.5)*140
  );

  scene.add(bubble);

  bubbles.push(bubble);
}

/* =========================================================
   CONTROLS
   ========================================================= */

let moveX=0;
let moveZ=0;

let joystickPointer=null;

const joystick=
document.getElementById("joystick");

const stick=
document.getElementById("stick");

joystick.addEventListener(
  "pointerdown",
  e=>{

    joystickPointer=e.pointerId;

    joystick.setPointerCapture(
      e.pointerId
    );

    updateJoystick(e);
  }
);

joystick.addEventListener(
  "pointermove",
  e=>{

    if(e.pointerId!==joystickPointer)
      return;

    updateJoystick(e);
  }
);

joystick.addEventListener(
  "pointerup",
  resetJoystick
);

joystick.addEventListener(
  "pointercancel",
  resetJoystick
);

function updateJoystick(e){

  const rect=
  joystick.getBoundingClientRect();

  let x=
  e.clientX-
  (rect.left+rect.width/2);

  let y=
  e.clientY-
  (rect.top+rect.height/2);

  const max=44;

  const length=
  Math.hypot(x,y);

  if(length>max){

    x=x/length*max;
    y=y/length*max;
  }

  moveX=x/max;
  moveZ=y/max;

  stick.style.transform=
  `translate(
    calc(-50% + ${x}px),
    calc(-50% + ${y}px)
  )`;
}

function resetJoystick(){

  joystickPointer=null;

  moveX=0;
  moveZ=0;

  stick.style.transform=
  "translate(-50%,-50%)";
}

/* =========================================================
   CAMERA
   ========================================================= */

let cameraYaw=0;
let cameraPitch=.18;

let cameraDragging=false;

let previousX=0;
let previousY=0;

canvas.addEventListener(
  "pointerdown",
  e=>{

    cameraDragging=true;

    previousX=e.clientX;
    previousY=e.clientY;
  }
);

canvas.addEventListener(
  "pointermove",
  e=>{

    if(!cameraDragging)
      return;

    const dx=
    e.clientX-previousX;

    const dy=
    e.clientY-previousY;

    cameraYaw-=dx*.006;

    cameraPitch-=dy*.004;

    cameraPitch=
    Math.max(
      -.2,
      Math.min(.65,cameraPitch)
    );

    previousX=e.clientX;
    previousY=e.clientY;
  }
);

canvas.addEventListener(
  "pointerup",
  ()=>{
    cameraDragging=false;
  }
);

canvas.addEventListener(
  "pointercancel",
  ()=>{
    cameraDragging=false;
  }
);

/* =========================================================
   JUMP
   ========================================================= */

let jumping=false;
let verticalVelocity=0;

function jump(){

  if(jumping)
    return;

  jumping=true;

  verticalVelocity=10;
}

document
.getElementById("jump")
.addEventListener(
  "pointerdown",
  jump
);

/* =========================================================
   RIDE DOLPHIN
   ========================================================= */

let riding=false;

function toggleRide(){

  const distance=
  player.position.distanceTo(
    dolphin.position
  );

  if(!riding){

    if(distance<10){

      riding=true;

      player.position.copy(
        dolphin.position
      );

      player.position.y+=2;
    }

  }else{

    riding=false;

    player.position.y=1.1;
  }
}

document
.getElementById("ride")
.addEventListener(
  "pointerdown",
  toggleRide
);

/* =========================================================
   KEYBOARD
   ========================================================= */

const keys={};

window.addEventListener(
  "keydown",
  e=>{

    keys[e.key.toLowerCase()]=true;

    if(e.code==="Space")
      jump();

    if(e.key.toLowerCase()==="e")
      toggleRide();
  }
);

window.addEventListener(
  "keyup",
  e=>{
    keys[e.key.toLowerCase()]=false;
  }
);

/* =========================================================
   ANIMATION
   ========================================================= */

const clock=
new THREE.Clock();

function animate(){

  requestAnimationFrame(
    animate
  );

  const dt=
  Math.min(
    clock.getDelta(),
    .04
  );

  const time=
  clock.elapsedTime;

  /* -------------------------------------
     KEYBOARD
     ------------------------------------- */

  let keyboardX=0;
  let keyboardZ=0;

  if(
    keys["a"] ||
    keys["arrowleft"]
  )
    keyboardX=-1;

  if(
    keys["d"] ||
    keys["arrowright"]
  )
    keyboardX=1;

  if(
    keys["w"] ||
    keys["arrowup"]
  )
    keyboardZ=-1;

  if(
    keys["s"] ||
    keys["arrowdown"]
  )
    keyboardZ=1;

  if(keyboardX!==0)
    moveX=keyboardX;

  if(keyboardZ!==0)
    moveZ=keyboardZ;

  /* -------------------------------------
     CORRECT MOVEMENT DIRECTIONS
     ------------------------------------- */

  /*
    IMPORTANT:

    joystick UP    = forward
    joystick DOWN  = backward
    joystick LEFT  = left
    joystick RIGHT = right
  */

  const forward=
  new THREE.Vector3(
    Math.sin(cameraYaw),
    0,
    -Math.cos(cameraYaw)
  );

  const right=
  new THREE.Vector3(
    Math.cos(cameraYaw),
    0,
    Math.sin(cameraYaw)
  );

  const movement=
  new THREE.Vector3();

  movement.addScaledVector(
    right,
    moveX
  );

  movement.addScaledVector(
    forward,
    -moveZ
  );

  const strength=
  Math.min(
    1,
    Math.hypot(
      moveX,
      moveZ
    )
  );

  if(movement.lengthSq()>0){

    movement.normalize();

    const speed=
    riding ? 17 : 7;

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

  /* -------------------------------------
     PLAYER ALWAYS FACES FORWARD
     ------------------------------------- */

  /*
    The character does NOT rotate toward
    the direction he is walking.

    If you walk backward, he walks backward
    while still facing forward.
  */

  player.rotation.y=
  cameraYaw;

  /* -------------------------------------
     JUMP
     ------------------------------------- */

  if(jumping){

    verticalVelocity-=25*dt;

    if(riding){

      dolphin.position.y+=
      verticalVelocity*dt;

      if(dolphin.position.y<=0){

        dolphin.position.y=0;

        verticalVelocity=0;

        jumping=false;
      }

    }else{

      player.position.y+=
      verticalVelocity*dt;

      if(player.position.y<=1.1){

        player.position.y=1.1;

        verticalVelocity=0;

        jumping=false;
      }
    }
  }

  /* -------------------------------------
     DOLPHIN
     ------------------------------------- */

  dolphin.position.y+=
  Math.sin(time*3)*.012;

  dolphin.rotation.z=
  Math.sin(time*2.8)*.035;

  if(riding){

    player.position.copy(
      dolphin.position
    );

    player.position.y+=2;

    /*
      Still facing the same forward
      direction instead of turning
      based on movement.
    */

    player.rotation.y=
    cameraYaw;
  }

  /* -------------------------------------
     OTHER GUYS
     ------------------------------------- */

  otherGuys.forEach(
    (g,i)=>{

      g.position.y=
      2+
      Math.sin(
        time*1.5+i
      )*.08;

      /*
        Their faces stay forward too.
      */

      g.rotation.y=
      Math.sin(i)*.15;
    }
  );

  /* -------------------------------------
     BUBBLES
     ------------------------------------- */

  bubbles.forEach(
    (b,i)=>{

      b.position.y+=
      dt*(.3+(i%3)*.15);

      b.rotation.y+=
      dt*.4;

      if(b.position.y>15)
        b.position.y=-1;
    }
  );

  /* -------------------------------------
     CAMERA
     ------------------------------------- */

  const target=
  riding
  ? dolphin
  : player;

  const distance=
  riding ? 17 : 14;

  const desiredX=
  target.position.x-
  Math.sin(cameraYaw)*distance;

  const desiredZ=
  target.position.z+
  Math.cos(cameraYaw)*distance;

  const desiredY=
  target.position.y+
  6+
  cameraPitch*7;

  camera.position.x=
  THREE.MathUtils.lerp(
    camera.position.x,
    desiredX,
    .09
  );

  camera.position.y=
  THREE.MathUtils.lerp(
    camera.position.y,
    desiredY,
    .09
  );

  camera.position.z=
  THREE.MathUtils.lerp(
    camera.position.z,
    desiredZ,
    .09
  );

  camera.lookAt(
    target.position.x,
    target.position.y+2,
    target.position.z
  );

  /* -------------------------------------
     WATER
     ------------------------------------- */

  water.position.y=
  -1.6+
  Math.sin(time)*.025;

  renderer.render(
    scene,
    camera
  );
}

animate();

/* =========================================================
   RESIZE
   ========================================================= */

function resize(){

  const width=
  window.innerWidth;

  const height=
  window.innerHeight;

  camera.aspect=
  width/height;

  camera.updateProjectionMatrix();

  renderer.setSize(
    width,
    height
  );

  renderer.setPixelRatio(
    Math.min(
      window.devicePixelRatio,
      2
    )
  );
}

window.addEventListener(
  "resize",
  resize
);

window.addEventListener(
  "orientationchange",
  resize
);

resize();

</script>

</body>
</html>
