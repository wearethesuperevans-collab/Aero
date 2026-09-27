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

<title>Frutiger Aero World</title>

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
  background:#43dfff;
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
  font-size:23px;
  font-weight:900;
  text-shadow:
    0 2px 4px rgba(0,80,130,.7),
    0 0 12px rgba(255,255,255,.35);
}

#subtitle{
  position:absolute;
  top:49px;
  left:21px;
  color:white;
  font-size:12px;
  text-shadow:0 2px 4px rgba(0,80,130,.7);
}

#joystick{
  position:absolute;
  left:22px;
  bottom:22px;
  width:132px;
  height:132px;
  border-radius:50%;
  background:rgba(255,255,255,.20);
  border:3px solid rgba(255,255,255,.75);
  box-shadow:
    0 10px 30px rgba(0,80,130,.25),
    inset 0 0 25px rgba(255,255,255,.25);
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
    #ffffff,
    #b8f5ff
  );
  border:3px solid white;
  box-shadow:
    0 5px 18px rgba(0,80,130,.25),
    inset 0 0 10px white;
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
  font-size:11px;
  font-weight:900;
  background:linear-gradient(
    #a0f8ff,
    #16a8d7
  );
  box-shadow:
    0 8px 22px rgba(0,80,130,.3),
    inset 0 3px 8px rgba(255,255,255,.55);
  pointer-events:auto;
  touch-action:none;
}

button:active{
  transform:scale(.90);
}

#rotate{
  display:none;
  position:fixed;
  inset:0;
  z-index:100;
  background:linear-gradient(
    #53e1ff,
    #c7fbff
  );
  align-items:center;
  justify-content:center;
  flex-direction:column;
  color:#08799e;
  text-align:center;
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
  <small>Landscape mode gives you the full world.</small>
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
  canvas,
  antialias:true,
  powerPreference:"high-performance"
});

renderer.setPixelRatio(
  Math.min(window.devicePixelRatio,2)
);

renderer.setSize(
  innerWidth,
  innerHeight
);

renderer.outputColorSpace =
THREE.SRGBColorSpace;

renderer.toneMapping =
THREE.ACESFilmicToneMapping;

renderer.toneMappingExposure=1.18;

renderer.shadowMap.enabled=true;

renderer.shadowMap.type =
THREE.PCFSoftShadowMap;


/* =========================================================
   SCENE
   ========================================================= */

const scene =
new THREE.Scene();

scene.background =
new THREE.Color(0x65ddff);

scene.fog =
new THREE.Fog(
  0x72e4ff,
  120,
  430
);


/* =========================================================
   CAMERA
   ========================================================= */

const camera =
new THREE.PerspectiveCamera(
  63,
  innerWidth/innerHeight,
  .1,
  800
);

camera.position.set(
  0,
  7,
  18
);


/* =========================================================
   LIGHTING
   ========================================================= */

const skyLight =
new THREE.HemisphereLight(
  0xeaffff,
  0x348b55,
  2.8
);

scene.add(skyLight);


const sun =
new THREE.DirectionalLight(
  0xffffff,
  4
);

sun.position.set(
  -100,
  150,
  90
);

sun.castShadow=true;

sun.shadow.mapSize.width=4096;
sun.shadow.mapSize.height=4096;

sun.shadow.camera.left=-150;
sun.shadow.camera.right=150;
sun.shadow.camera.top=150;
sun.shadow.camera.bottom=-150;

sun.shadow.bias=-.0003;

scene.add(sun);


/* =========================================================
   MATERIALS
   ========================================================= */

const aeroBlue =
new THREE.MeshPhysicalMaterial({
  color:0x168fe1,
  roughness:.13,
  metalness:.03,
  clearcoat:1,
  clearcoatRoughness:.04
});


const aeroGlass =
new THREE.MeshPhysicalMaterial({
  color:0x27caff,
  roughness:.09,
  metalness:.02,
  transparent:true,
  opacity:.96,
  clearcoat:1,
  clearcoatRoughness:.025
});


const cyanGlow =
new THREE.MeshBasicMaterial({
  color:0x00f3ff,
  transparent:true,
  opacity:.82
});


const highlightMaterial =
new THREE.MeshBasicMaterial({
  color:0xffffff,
  transparent:true,
  opacity:.76
});


const grassMaterial =
new THREE.MeshPhysicalMaterial({
  color:0x48d56b,
  roughness:.78,
  clearcoat:.3,
  clearcoatRoughness:.4
});


const dirtMaterial =
new THREE.MeshStandardMaterial({
  color:0x52a85d,
  roughness:.95
});


const leafMaterial =
new THREE.MeshPhysicalMaterial({
  color:0x19a958,
  roughness:.62,
  clearcoat:.25
});


const trunkMaterial =
new THREE.MeshStandardMaterial({
  color:0xc58b4e,
  roughness:.9
});


/* =========================================================
   WATER
   ========================================================= */

const waterGeometry =
new THREE.PlaneGeometry(
  900,
  900,
  160,
  160
);

const waterMaterial =
new THREE.MeshPhysicalMaterial({
  color:0x0fbfe8,
  roughness:.12,
  metalness:.08,
  transparent:true,
  opacity:.94,
  clearcoat:1,
  clearcoatRoughness:.06
});

const water =
new THREE.Mesh(
  waterGeometry,
  waterMaterial
);

water.rotation.x=-Math.PI/2;

water.position.y=-1.55;

water.receiveShadow=true;

scene.add(water);


/* =========================================================
   ISLANDS
   ========================================================= */

const islands=[];

function createIsland(
  x,
  z,
  radius,
  scale
){

  const island =
  new THREE.Group();

  /*
    The top surface is ALWAYS at y=0.
    Everything placed on the island uses
    this same height.
  */

  const underside =
  new THREE.Mesh(
    new THREE.CylinderGeometry(
      radius*.92*scale,
      radius*1.28*scale,
      4*scale,
      48
    ),
    dirtMaterial
  );

  underside.position.y=-1.8*scale;

  underside.castShadow=true;
  underside.receiveShadow=true;

  island.add(underside);


  const top =
  new THREE.Mesh(
    new THREE.CylinderGeometry(
      radius*scale,
      radius*1.04*scale,
      .75*scale,
      48
    ),
    grassMaterial
  );

  /*
    Cylinder top is at y=0.
  */

  top.position.y=-.375*scale;

  top.castShadow=true;
  top.receiveShadow=true;

  island.add(top);


  island.position.set(
    x,
    0,
    z
  );

  scene.add(island);

  islands.push({
    object:island,
    radius:radius*scale,
    x,
    z
  });
}


createIsland(0,0,16,1.0);
createIsland(-70,-48,14,1.0);
createIsland(67,-68,15,1.0);
createIsland(88,52,12,1.0);
createIsland(-95,65,13,1.0);


/* =========================================================
   FRUTIGER AERO GUY
   =========================================================

   IMPORTANT:
   This is deliberately built around the shape
   in the supplied reference:

   - huge smooth spherical head
   - smooth rounded torso
   - no visible legs
   - short rounded arms
   - blue/cyan translucent body
   - glossy white highlights
   - simple clean silhouette
   ========================================================= */

function createAeroGuy(){

  const guy =
  new THREE.Group();


  /* ---------------- BODY ---------------- */

  const body =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1,
      64,
      48
    ),
    aeroGlass
  );

  body.scale.set(
    1.30,
    1.35,
    .92
  );

  /*
    Bottom of sphere after scaling ≈ 0.65.
    This prevents the body from disappearing
    into the ground.
  */

  body.position.y=1.36;

  body.castShadow=true;
  body.receiveShadow=true;

  guy.add(body);


  /* ---------------- BODY INNER BLUE ---------------- */

  const innerBody =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .98,
      48,
      36
    ),
    aeroBlue
  );

  innerBody.scale.set(
    1.25,
    1.12,
    .88
  );

  innerBody.position.y=1.31;

  innerBody.material.transparent=true;
  innerBody.material.opacity=.40;

  guy.add(innerBody);


  /* ---------------- CYAN LOWER GLOW ---------------- */

  const lowerGlow =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .92,
      48,
      32
    ),
    cyanGlow
  );

  lowerGlow.scale.set(
    1.20,
    .48,
    .86
  );

  lowerGlow.position.set(
    0,
    .68,
    -.08
  );

  guy.add(lowerGlow);


  /* ---------------- HEAD ---------------- */

  const head =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1,
      64,
      48
    ),
    aeroGlass
  );

  head.scale.set(
    1.03,
    1.03,
    1.03
  );

  /*
    Head overlaps body slightly,
    exactly like the rounded reference.
  */

  head.position.y=3.35;

  head.castShadow=true;
  head.receiveShadow=true;

  guy.add(head);


  /* ---------------- HEAD INNER BLUE ---------------- */

  const headInner =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .965,
      48,
      36
    ),
    aeroBlue
  );

  headInner.position.y=3.34;

  headInner.material.transparent=true;
  headInner.material.opacity=.28;

  guy.add(headInner);


  /* ---------------- HEAD GLOSS ---------------- */

  const headGloss =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .38,
      32,
      24
    ),
    highlightMaterial
  );

  headGloss.scale.set(
    1.45,
    .52,
    .42
  );

  headGloss.position.set(
    -.30,
    3.78,
    -.78
  );

  guy.add(headGloss);


  /* ---------------- SECOND BODY GLOSS ---------------- */

  const bodyGloss =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .55,
      32,
      24
    ),
    highlightMaterial
  );

  bodyGloss.scale.set(
    1.35,
    .65,
    .35
  );

  bodyGloss.position.set(
    -.62,
    1.93,
    -.76
  );

  bodyGloss.material.opacity=.42;

  guy.add(bodyGloss);


  /* ---------------- LEFT ARM ---------------- */

  const leftArm =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1,
      48,
      36
    ),
    aeroGlass
  );

  leftArm.scale.set(
    .42,
    .88,
    .48
  );

  leftArm.position.set(
    -1.28,
    1.48,
    0
  );

  leftArm.rotation.z=.22;

  leftArm.castShadow=true;

  guy.add(leftArm);


  /* ---------------- RIGHT ARM ---------------- */

  const rightArm =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1,
      48,
      36
    ),
    aeroGlass
  );

  rightArm.scale.set(
    .42,
    .88,
    .48
  );

  rightArm.position.set(
    1.28,
    1.48,
    0
  );

  rightArm.rotation.z=-.22;

  rightArm.castShadow=true;

  guy.add(rightArm);


  /*
    No legs.
    No shoes.
    No facial features.

    This keeps the silhouette much closer
    to the supplied Frutiger-Aero character.
  */

  return guy;
}


/* =========================================================
   PLAYER
   ========================================================= */

const player =
createAeroGuy();

/*
  Island top = y 0.
  Character bottom ≈ 0.65.
  So the character is lifted only enough
  to sit naturally above the ground.
*/

player.position.set(
  0,
  -.55,
  7
);

scene.add(player);


/* =========================================================
   OTHER GUYS
   ========================================================= */

const otherGuys=[];

const positions=[
  [-8,0,-7],
  [8,0,-11],
  [-19,0,7],
  [20,0,2],
  [4,0,14]
];

positions.forEach((p)=>{

  const guy =
  createAeroGuy();

  guy.scale.setScalar(.72);

  guy.position.set(
    p[0],
    -.55,
    p[2]
  );

  scene.add(guy);

  otherGuys.push(guy);
});


/* =========================================================
   PALM TREES
   ========================================================= */

function createPalm(
  x,
  z,
  size
){

  /*
    Trees are placed at y=0,
    which is the island top.

    Their trunks therefore NEVER start
    several units underground.
  */

  const tree =
  new THREE.Group();


  const trunkHeight=
  6.5*size;


  const trunk =
  new THREE.Mesh(
    new THREE.CylinderGeometry(
      .28*size,
      .48*size,
      trunkHeight,
      18
    ),
    trunkMaterial
  );

  trunk.position.y=
  trunkHeight/2;

  trunk.castShadow=true;
  trunk.receiveShadow=true;

  tree.add(trunk);


  const crown =
  new THREE.Group();

  crown.position.y=
  trunkHeight;

  tree.add(crown);


  for(let i=0;i<8;i++){

    const angle=
    i*Math.PI*2/8;

    const leaf =
    new THREE.Mesh(
      new THREE.SphereGeometry(
        1,
        20,
        12
      ),
      leafMaterial
    );

    leaf.scale.set(
      2.4*size,
      .16*size,
      .48*size
    );

    leaf.position.set(
      Math.cos(angle)*1.35*size,
      -.15*size,
      Math.sin(angle)*1.35*size
    );

    leaf.rotation.y=-angle;

    leaf.rotation.z=
    -.18;

    leaf.castShadow=true;

    crown.add(leaf);
  }


  tree.position.set(
    x,
    0,
    z
  );

  scene.add(tree);
}


/*
  Every tree is deliberately placed near
  the TOP of the main island.
*/

const treePositions=[
  [-8,3,.9],
  [8,-3,.8],
  [-5,-7,.72],
  [6,8,.85],
  [-11,-8,.72],
  [12,7,.75],
  [-2,10,.7],
  [0,-10,.78],
  [11,-11,.68],
  [-12,9,.7]
];

treePositions.forEach(p=>{
  createPalm(
    p[0],
    p[1],
    p[2]
  );
});


/* =========================================================
   CLOUDS
   ========================================================= */

function createCloud(
  x,
  y,
  z,
  scale
){

  const cloud =
  new THREE.Group();

  for(let i=0;i<9;i++){

    const puff =
    new THREE.Mesh(
      new THREE.SphereGeometry(
        1,
        32,
        24
      ),
      new THREE.MeshPhysicalMaterial({
        color:0xffffff,
        roughness:.35,
        clearcoat:.7,
        clearcoatRoughness:.15
      })
    );

    puff.scale.setScalar(
      (.9+Math.random()*.8)*scale
    );

    puff.position.set(
      (i-4)*1.6*scale,
      Math.random()*1.1*scale,
      Math.random()*.9*scale
    );

    cloud.add(puff);
  }

  cloud.position.set(
    x,y,z
  );

  scene.add(cloud);
}


createCloud(-50,42,-90,2.1);
createCloud(55,46,-125,2.7);
createCloud(110,34,10,1.7);
createCloud(-125,37,70,2.2);


/* =========================================================
   DOLPHIN
   ========================================================= */

function createDolphin(){

  const dolphin =
  new THREE.Group();


  const dolphinMaterial =
  new THREE.MeshPhysicalMaterial({
    color:0x49c7ea,
    roughness:.14,
    metalness:.1,
    clearcoat:1,
    clearcoatRoughness:.04
  });


  const body =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      1,
      48,
      32
    ),
    dolphinMaterial
  );

  body.scale.set(
    2.3,
    .72,
    1
  );

  body.castShadow=true;

  dolphin.add(body);


  const nose =
  new THREE.Mesh(
    new THREE.ConeGeometry(
      .42,
      1.7,
      24
    ),
    dolphinMaterial
  );

  nose.rotation.z=-Math.PI/2;

  nose.position.z=-2.2;

  dolphin.add(nose);


  const fin =
  new THREE.Mesh(
    new THREE.ConeGeometry(
      .45,
      1.5,
      20
    ),
    dolphinMaterial
  );

  fin.rotation.z=Math.PI;

  fin.position.y=.85;

  dolphin.add(fin);


  const tail =
  new THREE.Group();

  for(const side of [-1,1]){

    const f =
    new THREE.Mesh(
      new THREE.ConeGeometry(
        .5,
        1.5,
        20
      ),
      dolphinMaterial
    );

    f.rotation.z=
    side*.75;

    f.position.x=
    side*.45;

    tail.add(f);
  }

  tail.position.z=2.1;

  dolphin.add(tail);


  return dolphin;
}


const dolphin =
createDolphin();

dolphin.position.set(
  0,
  -.3,
  -20
);

scene.add(dolphin);


/* =========================================================
   BUBBLES
   ========================================================= */

const bubbles=[];

for(let i=0;i<55;i++){

  const bubble =
  new THREE.Mesh(
    new THREE.SphereGeometry(
      .2+Math.random()*.35,
      24,
      20
    ),
    new THREE.MeshPhysicalMaterial({
      color:0xeaffff,
      transparent:true,
      opacity:.45,
      roughness:0,
      transmission:.3,
      clearcoat:1
    })
  );

  bubble.position.set(
    (Math.random()-.5)*160,
    Math.random()*13,
    (Math.random()-.5)*160
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

  const r=
  joystick.getBoundingClientRect();

  let x=
  e.clientX-
  (r.left+r.width/2);

  let y=
  e.clientY-
  (r.top+r.height/2);

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
let velocityY=0;

function jump(){

  if(jumping)
    return;

  jumping=true;

  velocityY=10;
}

document
.getElementById("jump")
.addEventListener(
  "pointerdown",
  jump
);


/* =========================================================
   RIDE
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

    player.position.y=-.55;
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


  /* -----------------------------------------
     KEYBOARD
     ----------------------------------------- */

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


  /* -----------------------------------------
     MOVEMENT
     ----------------------------------------- */

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
    riding ? 18 : 7;

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


  /* -----------------------------------------
     CHARACTER ALWAYS FACES FORWARD
     ----------------------------------------- */

  player.rotation.y=
  cameraYaw;


  /* -----------------------------------------
     JUMP
     ----------------------------------------- */

  if(jumping){

    velocityY-=25*dt;

    if(riding){

      dolphin.position.y+=
      velocityY*dt;

      if(dolphin.position.y<=-.3){

        dolphin.position.y=-.3;

        velocityY=0;

        jumping=false;
      }

    }else{

      player.position.y+=
      velocityY*dt;

      if(player.position.y<=-.55){

        player.position.y=-.55;

        velocityY=0;

        jumping=false;
      }
    }
  }


  /* -----------------------------------------
     DOLPHIN
     ----------------------------------------- */

  dolphin.rotation.z=
  Math.sin(time*2.5)*.035;

  dolphin.position.y+=
  Math.sin(time*3)*.008;


  if(riding){

    player.position.copy(
      dolphin.position
    );

    player.position.y+=2;

    player.rotation.y=
    cameraYaw;
  }


  /* -----------------------------------------
     OTHER GUYS
     ----------------------------------------- */

  otherGuys.forEach(
    (g,i)=>{

      g.position.y=
      -.55+
      Math.sin(
        time*1.5+i
      )*.045;

      /*
        They remain facing forward.
      */

      g.rotation.y=
      0;
    }
  );


  /* -----------------------------------------
     BUBBLES
     ----------------------------------------- */

  bubbles.forEach(
    (b,i)=>{

      b.position.y+=
      dt*(.3+(i%4)*.13);

      b.rotation.y+=
      dt*.5;

      if(b.position.y>15)
        b.position.y=-1;
    }
  );


  /* -----------------------------------------
     CAMERA
     ----------------------------------------- */

  const target=
  riding
  ? dolphin
  : player;

  const distance=
  riding ? 18 : 15;

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
    .085
  );

  camera.position.y=
  THREE.MathUtils.lerp(
    camera.position.y,
    desiredY,
    .085
  );

  camera.position.z=
  THREE.MathUtils.lerp(
    camera.position.z,
    desiredZ,
    .085
  );


  camera.lookAt(
    target.position.x,
    target.position.y+2,
    target.position.z
  );


  /* -----------------------------------------
     WATER
     ----------------------------------------- */

  water.position.y=
  -1.55+
  Math.sin(time)*.018;


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
  innerWidth;

  const height=
  innerHeight;

  camera.aspect=
  width/height;

  camera.updateProjectionMatrix();

  renderer.setSize(
    width,
    height
  );

  renderer.setPixelRatio(
    Math.min(
      devicePixelRatio,
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
