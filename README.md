<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

<title>Frutiger Aero World</title>

<style>
*{
  box-sizing:border-box;
  -webkit-tap-highlight-color:transparent;
}

html,body{
  margin:0;
  width:100%;
  height:100%;
  overflow:hidden;
  touch-action:none;
  background:#55dafa;
  font-family:Arial,sans-serif;
}

canvas{
  display:block;
}

#ui{
  position:fixed;
  inset:0;
  pointer-events:none;
}

button{
  border:0;
  color:white;
  font-weight:900;
  pointer-events:auto;
  touch-action:none;
  user-select:none;
  text-shadow:0 2px 4px rgba(0,60,100,.7);
  box-shadow:
    0 7px 20px rgba(0,60,100,.25),
    inset 0 2px 6px rgba(255,255,255,.7),
    inset 0 -5px 12px rgba(0,80,130,.16);
}

#title{
  position:absolute;
  top:12px;
  left:50%;
  transform:translateX(-50%);
  color:white;
  font-size:17px;
  white-space:nowrap;
  text-shadow:
    0 2px 5px rgba(0,70,130,.8),
    0 0 12px white;
}

#shift{
  position:absolute;
  right:12px;
  top:12px;
  padding:12px 14px;
  border-radius:17px;
  background:rgba(0,110,185,.78);
}

#shift.active{
  background:rgba(0,190,110,.95);
}

#run,#jump{
  width:76px;
  height:76px;
  border-radius:50%;
}

#run{
  position:absolute;
  right:22px;
  bottom:145px;
  background:rgba(0,125,220,.9);
}

#run.active{
  background:#00bc70;
  transform:scale(1.08);
}

#jump{
  position:absolute;
  right:22px;
  bottom:55px;
  background:rgba(0,170,120,.9);
}

#ride{
  position:absolute;
  right:112px;
  bottom:67px;
  width:66px;
  height:66px;
  border-radius:50%;
  background:rgba(0,120,205,.9);
}

#joystick{
  position:absolute;
  left:22px;
  bottom:35px;
  width:132px;
  height:132px;
  border-radius:50%;
  pointer-events:auto;
  background:rgba(255,255,255,.12);
  border:2px solid rgba(255,255,255,.55);
  box-shadow:
    inset 0 0 30px rgba(255,255,255,.25),
    0 8px 25px rgba(0,60,100,.25);
}

#stick{
  position:absolute;
  left:50%;
  top:50%;
  width:59px;
  height:59px;
  margin:-29.5px;
  border-radius:50%;
  background:
    radial-gradient(circle at 30% 20%,
      white,
      #e6fbff 38%,
      #8eddf3);
  box-shadow:
    0 6px 15px rgba(0,60,100,.25),
    inset 0 2px 5px white;
}
</style>
</head>

<body>

<div id="ui">

  <div id="title">
    FRUTIGER AERO WORLD
  </div>

  <button id="shift">SHIFT LOCK</button>
  <button id="run">RUN</button>
  <button id="ride">RIDE</button>
  <button id="jump">JUMP</button>

  <div id="joystick">
    <div id="stick"></div>
  </div>

</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

/* =========================================================
   BASIC THREE.JS SETUP
========================================================= */

const scene = new THREE.Scene();

scene.background = new THREE.Color(0x63daf7);

scene.fog = new THREE.FogExp2(
  0x9cecf8,
  0.00145
);

const camera = new THREE.PerspectiveCamera(
  64,
  innerWidth / innerHeight,
  0.1,
  1000
);

const renderer = new THREE.WebGLRenderer({
  antialias:true,
  powerPreference:"high-performance"
});

renderer.setSize(innerWidth,innerHeight);

renderer.setPixelRatio(
  Math.min(window.devicePixelRatio || 1,1.35)
);

renderer.outputColorSpace =
  THREE.SRGBColorSpace;

renderer.toneMapping =
  THREE.ACESFilmicToneMapping;

renderer.toneMappingExposure = 1.2;

renderer.shadowMap.enabled = true;
renderer.shadowMap.type =
  THREE.PCFSoftShadowMap;

document.body.appendChild(renderer.domElement);


/* =========================================================
   LIGHTING
========================================================= */

scene.add(
  new THREE.HemisphereLight(
    0xeaffff,
    0x28633a,
    3.3
  )
);

const sun = new THREE.DirectionalLight(
  0xfff1cf,
  4.8
);

sun.position.set(-120,180,100);
sun.castShadow = true;

sun.shadow.mapSize.width = 1536;
sun.shadow.mapSize.height = 1536;

sun.shadow.camera.left = -220;
sun.shadow.camera.right = 220;
sun.shadow.camera.top = 220;
sun.shadow.camera.bottom = -220;

sun.shadow.camera.near = 1;
sun.shadow.camera.far = 550;

sun.shadow.bias = -0.0003;

scene.add(sun);


/* =========================================================
   SKY
========================================================= */

const skyCanvas = document.createElement("canvas");

skyCanvas.width = 1024;
skyCanvas.height = 512;

const skyCtx = skyCanvas.getContext("2d");

const skyGradient =
  skyCtx.createLinearGradient(0,0,0,512);

skyGradient.addColorStop(0,"#147bd3");
skyGradient.addColorStop(.23,"#28a9e8");
skyGradient.addColorStop(.48,"#61d5f5");
skyGradient.addColorStop(.75,"#a9edf9");
skyGradient.addColorStop(1,"#e6fcff");

skyCtx.fillStyle = skyGradient;
skyCtx.fillRect(0,0,1024,512);

const sunlight =
  skyCtx.createRadialGradient(
    760,80,10,
    760,80,270
  );

sunlight.addColorStop(
  0,
  "rgba(255,255,225,.9)"
);

sunlight.addColorStop(
  .35,
  "rgba(255,255,225,.25)"
);

sunlight.addColorStop(
  1,
  "rgba(255,255,255,0)"
);

skyCtx.fillStyle = sunlight;
skyCtx.fillRect(430,0,594,350);

const skyTexture =
  new THREE.CanvasTexture(skyCanvas);

skyTexture.colorSpace =
  THREE.SRGBColorSpace;

const sky = new THREE.Mesh(
  new THREE.SphereGeometry(500,64,40),
  new THREE.MeshBasicMaterial({
    map:skyTexture,
    side:THREE.BackSide
  })
);

scene.add(sky);


/* =========================================================
   CLOUDS
========================================================= */

const clouds = [];

function createCloud(x,y,z,speed,scale){

  const group = new THREE.Group();

  const material =
    new THREE.MeshPhysicalMaterial({
      color:0xffffff,
      roughness:.96,
      transparent:true,
      opacity:.9
    });

  const parts = [
    [-3,0,0,1],
    [-1.5,.45,.1,1.3],
    [0,.7,0,1.5],
    [1.6,.4,0,1.25],
    [3,0,0,1],
    [.2,1.05,-.1,.8]
  ];

  for(const p of parts){

    const mesh = new THREE.Mesh(
      new THREE.SphereGeometry(3,24,18),
      material
    );

    mesh.position.set(
      p[0],
      p[1],
      p[2]
    );

    mesh.scale.set(
      p[3]*1.4,
      p[3]*.62,
      p[3]
    );

    group.add(mesh);
  }

  group.position.set(x,y,z);
  group.scale.setScalar(scale);

  scene.add(group);

  clouds.push({
    object:group,
    speed,
    phase:Math.random()*20,
    baseY:y
  });
}

createCloud(-190,105,-240,.75,3.1);
createCloud(-50,125,-290,.58,3.7);
createCloud(110,100,-250,.8,2.8);
createCloud(225,115,-90,.68,3.2);
createCloud(-250,120,20,.7,3.5);
createCloud(-120,110,230,.6,2.8);
createCloud(90,125,250,.72,3.1);
createCloud(230,108,180,.8,2.7);


/* =========================================================
   WATER
========================================================= */

const waterGeometry =
  new THREE.PlaneGeometry(
    850,
    850,
    70,
    70
  );

const waterMaterial =
  new THREE.MeshPhysicalMaterial({
    color:0x1dc6e5,
    roughness:.07,
    metalness:.06,
    transmission:.08,
    transparent:true,
    opacity:.86,
    clearcoat:1,
    clearcoatRoughness:.03
  });

const water = new THREE.Mesh(
  waterGeometry,
  waterMaterial
);

water.rotation.x = -Math.PI/2;
water.position.y = -4;

scene.add(water);


/* =========================================================
   ISLAND
========================================================= */

const island = new THREE.Mesh(
  new THREE.BoxGeometry(390,4,390),
  new THREE.MeshStandardMaterial({
    color:0x3d9848,
    roughness:.98
  })
);

island.position.y = -1.7;
island.receiveShadow = true;

scene.add(island);


/* =========================================================
   BEACH + GRASS
========================================================= */

const beach = new THREE.Mesh(
  new THREE.CylinderGeometry(
    154,154,.55,128
  ),
  new THREE.MeshStandardMaterial({
    color:0xf0df9d,
    roughness:.92
  })
);

beach.position.y = .43;
beach.receiveShadow = true;

scene.add(beach);


const grass = new THREE.Mesh(
  new THREE.CylinderGeometry(
    132,132,.6,128
  ),
  new THREE.MeshStandardMaterial({
    color:0x42c94c,
    roughness:.87
  })
);

grass.position.y = .75;
grass.receiveShadow = true;

scene.add(grass);


/* =========================================================
   TREES
========================================================= */

const trunkMat =
  new THREE.MeshStandardMaterial({
    color:0x704b2b,
    roughness:.9
  });

const leafMat =
  new THREE.MeshPhysicalMaterial({
    color:0x20a84c,
    roughness:.62,
    clearcoat:.2
  });

for(let i=0;i<105;i++){

  const angle =
    Math.random()*Math.PI*2;

  const radius =
    42+Math.random()*82;

  const tree = new THREE.Group();

  const trunk = new THREE.Mesh(
    new THREE.CylinderGeometry(
      .32,.62,5.8,10
    ),
    trunkMat
  );

  trunk.position.y=3;
  trunk.castShadow=true;

  tree.add(trunk);

  const crown = new THREE.Mesh(
    new THREE.IcosahedronGeometry(3.3,2),
    leafMat
  );

  crown.position.y=7;
  crown.scale.set(1.05,1.2,1.05);
  crown.castShadow=true;

  tree.add(crown);

  const crown2 = new THREE.Mesh(
    new THREE.IcosahedronGeometry(2,2),
    leafMat
  );

  crown2.position.set(1,7.7,.2);
  crown2.castShadow=true;

  tree.add(crown2);

  tree.position.set(
    Math.cos(angle)*radius,
    .8,
    Math.sin(angle)*radius
  );

  scene.add(tree);
}


/* =========================================================
   CHARACTER
   BASED ON THE UPLOADED PICTURE
========================================================= */

/*
  The important proportions:

  HEAD:
    large round blank sphere

  BODY:
    large smooth wide blob,
    widest toward lower-middle

  ARMS:
    long, narrow rounded pieces,
    hanging diagonally outward

  LEGS:
    short chunky rounded pieces,
    clearly separated

  NO FACE.
  NO CLOTHES.
  NO ARMOR.
  NO EXTRA DETAILS.
*/


function gradientGeometry(geometry){

  const pos =
    geometry.attributes.position;

  const colors=[];

  for(let i=0;i<pos.count;i++){

    const y=pos.getY(i);

    const t =
      THREE.MathUtils.clamp(
        (y+1)/2,
        0,
        1
      );

    const dark =
      new THREE.Color(0x0879d2);

    const mid =
      new THREE.Color(0x26cee8);

    const light =
      new THREE.Color(0xe0ffff);

    const c=new THREE.Color();

    if(t<.5){

      c.lerpColors(
        dark,
        mid,
        t*2
      );

    }else{

      c.lerpColors(
        mid,
        light,
        (t-.5)*2
      );
    }

    colors.push(
      c.r,c.g,c.b
    );
  }

  geometry.setAttribute(
    "color",
    new THREE.Float32BufferAttribute(
      colors,3
    )
  );

  return geometry;
}


const characterMaterial =
  new THREE.MeshPhysicalMaterial({
    vertexColors:true,
    roughness:.10,
    metalness:.01,
    clearcoat:1,
    clearcoatRoughness:.045,
    sheen:.2,
    sheenRoughness:.15
  });


/* =========================================================
   BODY SHAPE
========================================================= */

function makePictureBody(){

  const geometry =
    new THREE.SphereGeometry(
      1,
      64,
      48
    );

  const p =
    geometry.attributes.position;

  for(let i=0;i<p.count;i++){

    let x=p.getX(i);
    let y=p.getY(i);
    let z=p.getZ(i);

    /*
      Custom silhouette:
      small rounded shoulder area,
      very broad center,
      rounded lower body.
    */

    const n=(y+1)/2;

    let width;

    if(n>.78){

      width=.64+(1-n)*1.35;

    }else if(n>.45){

      width=1.03;

    }else{

      width=.92+n*.25;
    }

    x*=width;
    z*=.76;

    if(y<-.55){

      x*=.92;
      z*=.94;
    }

    p.setXYZ(
      i,x,y,z
    );
  }

  return gradientGeometry(
    geometry
  );
}


/* =========================================================
   PLAYER
========================================================= */

const player =
  new THREE.Group();

scene.add(player);


/* HEAD */

const head = new THREE.Mesh(
  gradientGeometry(
    new THREE.SphereGeometry(
      1.05,
      64,
      48
    )
  ),
  characterMaterial
);

head.position.set(
  0,
  4.32,
  0
);

head.scale.set(
  1.03,
  1.04,
  .97
);

head.castShadow=true;

player.add(head);


/* BODY */

const body = new THREE.Mesh(
  makePictureBody(),
  characterMaterial
);

body.position.y=2.55;

body.scale.set(
  1.37,
  1.43,
  .93
);

body.castShadow=true;

player.add(body);


/* =========================================================
   ARMS
========================================================= */

function makeArm(){

  const geometry =
    new THREE.CapsuleGeometry(
      .28,
      1.85,
      24,
      32
    );

  return new THREE.Mesh(
    gradientGeometry(geometry),
    characterMaterial
  );
}

const leftArm=makeArm();
const rightArm=makeArm();

leftArm.position.set(
  -1.48,
  2.3,
  0
);

rightArm.position.set(
  1.48,
  2.3,
  0
);

/*
  Match the picture's downward
  outward arm angle.
*/

leftArm.rotation.z=-.28;
rightArm.rotation.z=.28;

leftArm.scale.set(
  .82,1.08,.88
);

rightArm.scale.set(
  .82,1.08,.88
);

leftArm.castShadow=true;
rightArm.castShadow=true;

player.add(leftArm);
player.add(rightArm);


/* =========================================================
   LEGS
========================================================= */

function makeLeg(){

  const geometry =
    new THREE.CapsuleGeometry(
      .40,
      1.35,
      24,
      32
    );

  return new THREE.Mesh(
    gradientGeometry(geometry),
    characterMaterial
  );
}

const leftLeg=makeLeg();
const rightLeg=makeLeg();

leftLeg.position.set(
  -.58,
  .72,
  0
);

rightLeg.position.set(
  .58,
  .72,
  0
);

leftLeg.scale.set(
  1.05,1.12,.96
);

rightLeg.scale.set(
  1.05,1.12,.96
);

leftLeg.castShadow=true;
rightLeg.castShadow=true;

player.add(leftLeg);
player.add(rightLeg);


/* =========================================================
   DOLPHIN
========================================================= */

const dolphin =
  new THREE.Group();

const dolphinMat =
  new THREE.MeshPhysicalMaterial({
    color:0x61d6ed,
    roughness:.1,
    metalness:.02,
    clearcoat:1
  });

const dolphinBody =
  new THREE.Mesh(
    new THREE.SphereGeometry(2,40,28),
    dolphinMat
  );

dolphinBody.scale.set(
  1.8,.62,.82
);

dolphin.add(dolphinBody);

const nose =
  new THREE.Mesh(
    new THREE.CapsuleGeometry(
      .34,1.3,14,22
    ),
    dolphinMat
  );

nose.rotation.z=-Math.PI/2;
nose.position.x=2.9;

dolphin.add(nose);

const fin =
  new THREE.Mesh(
    new THREE.ConeGeometry(
      .6,1.6,4
    ),
    dolphinMat
  );

fin.rotation.z=Math.PI;
fin.position.y=1;

dolphin.add(fin);

dolphin.position.set(
  18,2,-18
);

dolphin.rotation.y=Math.PI/2;

scene.add(dolphin);


/* =========================================================
   INPUT
========================================================= */

const keys={};

addEventListener("keydown",e=>{
  keys[e.key.toLowerCase()]=true;
});

addEventListener("keyup",e=>{
  keys[e.key.toLowerCase()]=false;
});


/* =========================================================
   JOYSTICK
========================================================= */

const joystick =
  document.getElementById("joystick");

const stick =
  document.getElementById("stick");

let joyX=0;
let joyY=0;
let joystickActive=false;

function moveStick(e){

  const r =
    joystick.getBoundingClientRect();

  let x =
    e.clientX-
    (r.left+r.width/2);

  let y =
    e.clientY-
    (r.top+r.height/2);

  const max=43;

  const length=
    Math.hypot(x,y);

  if(length>max){

    x=x/length*max;
    y=y/length*max;
  }

  joyX=x/max;
  joyY=y/max;

  stick.style.transform=
    `translate(${x}px,${y}px)`;
}

joystick.addEventListener(
  "pointerdown",
  e=>{
    joystickActive=true;
    joystick.setPointerCapture(e.pointerId);
    moveStick(e);
  }
);

joystick.addEventListener(
  "pointermove",
  e=>{
    if(joystickActive)
      moveStick(e);
  }
);

function resetJoystick(){

  joystickActive=false;
  joyX=0;
  joyY=0;

  stick.style.transform=
    "translate(0,0)";
}

joystick.addEventListener(
  "pointerup",
  resetJoystick
);

joystick.addEventListener(
  "pointercancel",
  resetJoystick
);


/* =========================================================
   RUN
========================================================= */

let running=false;

const runButton =
  document.getElementById("run");

function startRun(e){

  e.preventDefault();

  running=true;
  runButton.classList.add("active");
}

function stopRun(){

  running=false;
  runButton.classList.remove("active");
}

runButton.addEventListener(
  "pointerdown",
  startRun
);

runButton.addEventListener(
  "pointerup",
  stopRun
);

runButton.addEventListener(
  "pointercancel",
  stopRun
);


/* =========================================================
   JUMP
========================================================= */

let jumping=false;
let verticalVelocity=0;

document.getElementById("jump")
.addEventListener(
  "pointerdown",
  ()=>{

    if(!jumping && !riding){

      jumping=true;
      verticalVelocity=9;
    }
  }
);


/* =========================================================
   SHIFT LOCK
========================================================= */

let shiftLock=false;
let cameraYaw=0;

const shiftButton =
  document.getElementById("shift");

shiftButton.addEventListener(
  "pointerdown",
  ()=>{

    shiftLock=!shiftLock;

    shiftButton.classList.toggle(
      "active",
      shiftLock
    );

    if(shiftLock){

      cameraYaw=
        player.rotation.y;
    }
  }
);


/* =========================================================
   CAMERA DRAG
========================================================= */

let dragging=false;
let lastX=0;

renderer.domElement.addEventListener(
  "pointerdown",
  e=>{

    /*
      Shift Lock means NO camera
      dragging.
    */

    if(shiftLock)return;

    dragging=true;
    lastX=e.clientX;
  }
);

renderer.domElement.addEventListener(
  "pointermove",
  e=>{

    if(!dragging || shiftLock)
      return;

    const dx=e.clientX-lastX;

    lastX=e.clientX;

    cameraYaw-=dx*.007;
  }
);

addEventListener(
  "pointerup",
  ()=>{
    dragging=false;
  }
);


/* =========================================================
   RIDE
========================================================= */

let riding=false;

document.getElementById("ride")
.addEventListener(
  "pointerdown",
  ()=>{

    const distance =
      player.position.distanceTo(
        dolphin.position
      );

    if(distance<13 || riding){

      riding=!riding;

      if(riding){

        player.visible=false;

        dolphin.add(player);

        player.position.set(
          0,2.25,0
        );

      }else{

        scene.add(player);

        player.visible=true;

        player.position.set(
          dolphin.position.x,
          .85,
          dolphin.position.z
        );
      }
    }
  }
);


/* =========================================================
   GAME LOOP
========================================================= */

const clock=new THREE.Clock();

let animTime=0;

function animate(){

  requestAnimationFrame(animate);

  const dt=Math.min(
    clock.getDelta(),
    .033
  );

  const time=performance.now()*.001;


/* CLOUD MOVEMENT */

  for(const cloud of clouds){

    cloud.object.position.x+=
      cloud.speed*dt;

    cloud.object.position.y=
      cloud.baseY+
      Math.sin(
        time*.07+
        cloud.phase
      )*.45;

    if(cloud.object.position.x>330)
      cloud.object.position.x=-330;
  }


/* =========================================================
   MOVEMENT
========================================================= */

  let x=0;
  let y=0;

  /*
    CORRECT CONTROLS:

    UP / W    = y -1 = FORWARD
    DOWN / S  = y +1 = BACKWARD
    LEFT / A  = x -1
    RIGHT / D = x +1
  */

  if(keys["a"]||keys["arrowleft"])
    x=-1;

  if(keys["d"]||keys["arrowright"])
    x=1;

  if(keys["w"]||keys["arrowup"])
    y=-1;

  if(keys["s"]||keys["arrowdown"])
    y=1;


  /*
    Mobile joystick takes priority.
  */

  if(
    Math.abs(joyX)>.06 ||
    Math.abs(joyY)>.06
  ){

    x=joyX;
    y=joyY;
  }


  const magnitude=
    Math.hypot(x,y);

  if(magnitude>1){

    x/=magnitude;
    y/=magnitude;
  }


/* =========================================================
   CAMERA DIRECTIONS
========================================================= */

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
    x
  );

  /*
    THIS -y IS THE IMPORTANT FIX.
    Pushing joystick UP gives y=-1,
    therefore -y becomes +1 forward.
  */

  movement.addScaledVector(
    forward,
    -y
  );

  if(movement.lengthSq()>.001)
    movement.normalize();


  const moving=
    magnitude>.07;


/* =========================================================
   SPEED
========================================================= */

  let speed=
    running ? 14 : 8;

  if(riding)
    speed=13;


  if(moving){

    const amount=speed*dt;

    if(riding){

      dolphin.position.addScaledVector(
        movement,
        amount
      );

    }else{

      player.position.addScaledVector(
        movement,
        amount
      );
    }


    if(!riding){

      /*
        Shift Lock:
        character stays facing the
        locked camera direction.
      */

      if(shiftLock){

        player.rotation.y=
          cameraYaw;

      }else{

        player.rotation.y=
          Math.atan2(
            movement.x,
            -movement.z
          );
      }
    }
  }


/* =========================================================
   JUMP
========================================================= */

  if(jumping&&!riding){

    verticalVelocity-=22*dt;

    player.position.y+=
      verticalVelocity*dt;

    if(player.position.y<=.85){

      player.position.y=.85;
      verticalVelocity=0;
      jumping=false;
    }

  }else if(!riding){

    player.position.y=.85;
  }


/* =========================================================
   WORLD BOUNDARY
========================================================= */

  const object=
    riding ? dolphin : player;

  const distance=
    Math.hypot(
      object.position.x,
      object.position.z
    );

  if(distance>174){

    const scale=174/distance;

    object.position.x*=scale;
    object.position.z*=scale;
  }


/* =========================================================
   WALK / RUN ANIMATION
========================================================= */

  if(moving&&!riding){

    animTime+=
      dt*(running?17:9);

    const swing=
      Math.sin(animTime)*
      (running?.65:.38);

    /*
      Arms move opposite legs.
    */

    leftArm.rotation.x=
      -swing*.82;

    rightArm.rotation.x=
      swing*.82;

    leftLeg.rotation.x=
      swing;

    rightLeg.rotation.x=
      -swing;


    /*
      Keep the body silhouette
      intact while adding a tiny
      movement bounce.
    */

    const bounce=
      Math.abs(
        Math.sin(animTime*2)
      )*
      (running?.055:.025);

    body.position.y=
      2.55+bounce;

    head.position.y=
      4.32+bounce;

  }else{

    leftArm.rotation.x=
      THREE.MathUtils.lerp(
        leftArm.rotation.x,0,.16
      );

    rightArm.rotation.x=
      THREE.MathUtils.lerp(
        rightArm.rotation.x,0,.16
      );

    leftLeg.rotation.x=
      THREE.MathUtils.lerp(
        leftLeg.rotation.x,0,.16
      );

    rightLeg.rotation.x=
      THREE.MathUtils.lerp(
        rightLeg.rotation.x,0,.16
      );

    body.position.y=
      THREE.MathUtils.lerp(
        body.position.y,2.55,.14
      );

    head.position.y=
      THREE.MathUtils.lerp(
        head.position.y,4.32,.14
      );
  }


/* =========================================================
   IDLE FLOAT
========================================================= */

  if(
    !moving &&
    !jumping &&
    !riding
  ){

    const idle=
      Math.sin(time*1.6)*.025;

    body.position.y=
      2.55+idle;

    head.position.y=
      4.32+idle;
  }


/* =========================================================
   DOLPHIN ANIMATION
========================================================= */

  dolphin.position.y=
    2+
    Math.sin(time*1.6)*.28;

  dolphin.rotation.z=
    Math.sin(time*1.1)*.035;


/* =========================================================
   WATER WAVES
========================================================= */

  const wp=
    waterGeometry.attributes.position;

  for(
    let i=0;
    i<wp.count;
    i++
  ){

    const px=wp.getX(i);
    const pz=wp.getY(i);

    wp.setZ(
      i,
      Math.sin(
        px*.035+
        time*.8
      )*.15+
      Math.cos(
        pz*.04+
        time
      )*.13
    );
  }

  wp.needsUpdate=true;


/* =========================================================
   CAMERA
========================================================= */

  const target=
    riding
      ? dolphin.position.clone()
      : player.position.clone();

  target.y+=
    riding ? 1.8 : 2.1;

  const distanceBack=
    shiftLock ? 8.2 : 10.5;

  const desired=
    target.clone();

  desired.x+=
    Math.sin(cameraYaw)*
    distanceBack;

  desired.z+=
    Math.cos(cameraYaw)*
    distanceBack;

  desired.y+=4.5;

  camera.position.lerp(
    desired,
    .105
  );

  camera.lookAt(target);


/* =========================================================
   RENDER
========================================================= */

  renderer.render(
    scene,
    camera
  );
}


/* =========================================================
   START
========================================================= */

player.position.set(
  0,.85,0
);

camera.position.set(
  0,7,13
);

camera.lookAt(
  0,2,0
);

animate();


/* =========================================================
   RESIZE
========================================================= */

addEventListener(
  "resize",
  ()=>{

    camera.aspect=
      innerWidth/innerHeight;

    camera.updateProjectionMatrix();

    renderer.setSize(
      innerWidth,
      innerHeight
    );
  }
);

</script>
</body>
</html>
