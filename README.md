<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<title>Frutiger Aero World</title>

<style>
html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#5bdcff;
    touch-action:none;
    font-family:Arial,sans-serif;
}

canvas{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
}

#ui{
    position:fixed;
    inset:0;
    pointer-events:none;
    color:white;
}

.glass{
    background:linear-gradient(
        135deg,
        rgba(255,255,255,.52),
        rgba(110,220,255,.20)
    );
    border:1.5px solid rgba(255,255,255,.85);
    box-shadow:
        0 8px 25px rgba(0,80,130,.22),
        inset 0 1px 3px rgba(255,255,255,.9);
    backdrop-filter:blur(10px);
}

#logo{
    position:absolute;
    top:14px;
    left:50%;
    transform:translateX(-50%);
    padding:10px 20px;
    border-radius:25px;
    font-weight:bold;
    font-size:20px;
    text-shadow:0 2px 5px #08749b;
}

#stats{
    position:absolute;
    top:14px;
    left:14px;
    padding:10px 14px;
    border-radius:18px;
    font-size:14px;
    text-shadow:0 2px 4px #08749b;
}

#status{
    position:absolute;
    right:14px;
    top:14px;
    padding:10px 14px;
    border-radius:18px;
    display:none;
    font-weight:bold;
}

#message{
    position:absolute;
    bottom:25px;
    left:50%;
    transform:translateX(-50%);
    padding:9px 16px;
    border-radius:18px;
    font-size:13px;
    white-space:nowrap;
}

#joystick{
    position:absolute;
    left:20px;
    bottom:22px;
    width:125px;
    height:125px;
    border-radius:50%;
    background:rgba(255,255,255,.16);
    border:2px solid rgba(255,255,255,.65);
    pointer-events:auto;
}

#knob{
    position:absolute;
    left:50%;
    top:50%;
    width:55px;
    height:55px;
    transform:translate(-50%,-50%);
    border-radius:50%;
    background:rgba(255,255,255,.65);
    border:2px solid white;
    box-shadow:0 5px 15px rgba(0,100,160,.3);
}

#buttons{
    position:absolute;
    right:18px;
    bottom:25px;
    display:flex;
    gap:12px;
    pointer-events:auto;
}

.gameButton{
    width:70px;
    height:70px;
    border-radius:50%;
    border:2px solid white;
    background:linear-gradient(
        rgba(255,255,255,.72),
        rgba(70,210,255,.55)
    );
    color:#05749d;
    font-weight:bold;
    box-shadow:0 7px 18px rgba(0,100,150,.3);
}

.gameButton:active{
    transform:scale(.9);
}

#rotate{
    position:absolute;
    inset:0;
    background:linear-gradient(#54d9ff,#b8f8ff);
    display:none;
    align-items:center;
    justify-content:center;
    text-align:center;
    z-index:20;
}

#rotate div{
    padding:25px;
    border-radius:25px;
    font-size:20px;
    color:#08749a;
}

@media (orientation:portrait) and (max-width:600px){
    #rotate{
        display:flex;
    }
}
</style>
</head>

<body>

<div id="rotate">
    <div class="glass">
        📱↔️<br><br>
        Turn your iPad sideways<br>
        for the full Aero World experience.
    </div>
</div>

<div id="ui">

    <div id="logo" class="glass">
        🌊 FRUTIGER AERO WORLD
    </div>

    <div id="stats" class="glass">
        💎 <span id="score">0</span>
        &nbsp;&nbsp;
        🌊 <span id="speed">0</span>
    </div>

    <div id="status" class="glass">
        🐬 DOLPHIN RIDE
    </div>

    <div id="message" class="glass">
        Explore the Aero World
    </div>

    <div id="joystick">
        <div id="knob"></div>
    </div>

    <div id="buttons">
        <button class="gameButton" id="jump">JUMP</button>
        <button class="gameButton" id="ride">RIDE</button>
    </div>

</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

/* =========================================================
   FRUTIGER AERO WORLD
   Real WebGL 3D
========================================================= */

let scene;
let camera;
let renderer;
let clock;

let player;
let dolphin;

let score=0;
let riding=false;

let velocity=new THREE.Vector3();
let verticalVelocity=0;

let cameraYaw=0;
let cameraPitch=.32;

let joystickX=0;
let joystickY=0;

let cameraTouch=false;
let cameraTouchX=0;
let cameraTouchY=0;

const scoreText=document.getElementById("score");
const speedText=document.getElementById("speed");
const status=document.getElementById("status");
const message=document.getElementById("message");

init();
animate();


/* =========================================================
   RENDERER
========================================================= */

function init(){

    scene=new THREE.Scene();

    scene.background=new THREE.Color(0x69dcff);

    scene.fog=new THREE.Fog(0x69dcff,250,1800);

    camera=new THREE.PerspectiveCamera(
        65,
        innerWidth/innerHeight,
        .1,
        3000
    );

    camera.position.set(0,70,130);

    renderer=new THREE.WebGLRenderer({
        antialias:true,
        powerPreference:"high-performance"
    });

    renderer.setPixelRatio(Math.min(devicePixelRatio,2));

    renderer.setSize(innerWidth,innerHeight);

    renderer.shadowMap.enabled=true;

    renderer.shadowMap.type=THREE.PCFSoftShadowMap;

    renderer.outputColorSpace=THREE.SRGBColorSpace;

    document.body.prepend(renderer.domElement);

    clock=new THREE.Clock();

    setupLights();
    setupSky();
    setupOcean();
    setupIslands();
    setupClouds();
    setupCharacters();
    setupDolphin();
    setupCollectibles();
    setupUI();

    addEventListener("resize",resize);
}


/* =========================================================
   LIGHTING
========================================================= */

function setupLights(){

    const hemi=new THREE.HemisphereLight(
        0xcfffff,
        0x3c9a68,
        2.5
    );

    scene.add(hemi);

    const sun=new THREE.DirectionalLight(
        0xffffff,
        3
    );

    sun.position.set(-300,500,-250);

    sun.castShadow=true;

    sun.shadow.mapSize.width=2048;
    sun.shadow.mapSize.height=2048;

    sun.shadow.camera.left=-700;
    sun.shadow.camera.right=700;
    sun.shadow.camera.top=700;
    sun.shadow.camera.bottom=-700;

    scene.add(sun);
}


/* =========================================================
   SKY
========================================================= */

function setupSky(){

    const skyGeo=new THREE.SphereGeometry(
        2200,
        32,
        32
    );

    const skyMat=new THREE.MeshBasicMaterial({
        color:0x63dfff,
        side:THREE.BackSide
    });

    const sky=new THREE.Mesh(
        skyGeo,
        skyMat
    );

    scene.add(sky);

    /* giant glowing sun */

    const sunCanvas=document.createElement("canvas");

    sunCanvas.width=256;
    sunCanvas.height=256;

    const sctx=sunCanvas.getContext("2d");

    const gradient=sctx.createRadialGradient(
        128,128,5,
        128,128,125
    );

    gradient.addColorStop(0,"rgba(255,255,255,1)");
    gradient.addColorStop(.25,"rgba(255,255,255,.8)");
    gradient.addColorStop(1,"rgba(255,255,255,0)");

    sctx.fillStyle=gradient;
    sctx.fillRect(0,0,256,256);

    const texture=new THREE.CanvasTexture(sunCanvas);

    const sunSprite=new THREE.Sprite(
        new THREE.SpriteMaterial({
            map:texture,
            transparent:true
        })
    );

    sunSprite.position.set(-500,600,-900);
    sunSprite.scale.set(500,500,1);

    scene.add(sunSprite);
}


/* =========================================================
   WATER
========================================================= */

function setupOcean(){

    const geometry=new THREE.PlaneGeometry(
        4000,
        4000,
        100,
        100
    );

    const material=new THREE.MeshPhysicalMaterial({
        color:0x12bce8,
        roughness:.12,
        metalness:.05,
        transmission:.05,
        transparent:true,
        opacity:.91
    });

    const ocean=new THREE.Mesh(
        geometry,
        material
    );

    ocean.rotation.x=-Math.PI/2;

    ocean.position.y=-3;

    ocean.receiveShadow=true;

    ocean.userData.water=true;

    scene.add(ocean);

    /* floating white highlights */

    for(let i=0;i<150;i++){

        const geometry=new THREE.SphereGeometry(
            THREE.MathUtils.randFloat(.5,2.2),
            8,
            8
        );

        const material=new THREE.MeshBasicMaterial({
            color:0xffffff,
            transparent:true,
            opacity:.3
        });

        const bubble=new THREE.Mesh(
            geometry,
            material
        );

        bubble.position.set(
            THREE.MathUtils.randFloatSpread(1800),
            THREE.MathUtils.randFloat(-1,8),
            THREE.MathUtils.randFloatSpread(1800)
        );

        scene.add(bubble);
    }
}


/* =========================================================
   ISLANDS
========================================================= */

function setupIslands(){

    for(let i=0;i<18;i++){

        const x=THREE.MathUtils.randFloatSpread(1800);
        const z=THREE.MathUtils.randFloatSpread(1600);

        const radius=THREE.MathUtils.randFloat(55,150);

        createIsland(x,z,radius);
    }
}


function createIsland(x,z,r){

    const group=new THREE.Group();

    group.position.set(x,0,z);

    /* sand */

    const sandGeometry=new THREE.CylinderGeometry(
        r,
        r*1.2,
        18,
        32
    );

    const sandMaterial=new THREE.MeshStandardMaterial({
        color:0xf3df91,
        roughness:.8
    });

    const sand=new THREE.Mesh(
        sandGeometry,
        sandMaterial
    );

    sand.position.y=3;

    sand.castShadow=true;
    sand.receiveShadow=true;

    group.add(sand);

    /* grass */

    const grassGeometry=new THREE.CylinderGeometry(
        r*.9,
        r*1.05,
        9,
        32
    );

    const grassMaterial=new THREE.MeshStandardMaterial({
        color:0x43c866,
        roughness:.7
    });

    const grass=new THREE.Mesh(
        grassGeometry,
        grassMaterial
    );

    grass.position.y=14;

    grass.castShadow=true;

    group.add(grass);

    /* futuristic glass dome */

    const domeGeometry=new THREE.SphereGeometry(
        r*.25,
        24,
        16,
        0,
        Math.PI*2,
        0,
        Math.PI/2
    );

    const domeMaterial=new THREE.MeshPhysicalMaterial({
        color:0x9ef4ff,
        transparent:true,
        opacity:.58,
        roughness:.05,
        metalness:.05,
        transmission:.25
    });

    const dome=new THREE.Mesh(
        domeGeometry,
        domeMaterial
    );

    dome.position.y=25;

    dome.castShadow=true;

    group.add(dome);

    /* center crystal */

    const crystal=new THREE.Mesh(
        new THREE.OctahedronGeometry(r*.13),
        new THREE.MeshPhysicalMaterial({
            color:0x66efff,
            emissive:0x0bbbdc,
            emissiveIntensity:.5,
            transparent:true,
            opacity:.82
        })
    );

    crystal.position.y=42;

    group.add(crystal);

    /* palm trees */

    for(let i=0;i<4;i++){

        const palm=createPalm();

        palm.position.set(
            THREE.MathUtils.randFloatSpread(r*1.2),
            17,
            THREE.MathUtils.randFloatSpread(r*1.2)
        );

        palm.scale.setScalar(
            THREE.MathUtils.randFloat(.7,1.2)
        );

        group.add(palm);
    }

    scene.add(group);
}


function createPalm(){

    const group=new THREE.Group();

    const trunk=new THREE.Mesh(
        new THREE.CylinderGeometry(.8,1.2,15,10),
        new THREE.MeshStandardMaterial({
            color:0x9c713e
        })
    );

    trunk.position.y=7;

    group.add(trunk);

    const leafMaterial=new THREE.MeshStandardMaterial({
        color:0x27b85b,
        side:THREE.DoubleSide
    });

    for(let i=0;i<7;i++){

        const leaf=new THREE.Mesh(
            new THREE.ConeGeometry(
                .9,
                8,
                6
            ),
            leafMaterial
        );

        leaf.rotation.z=Math.PI/2;
        leaf.rotation.y=(i/7)*Math.PI*2;

        leaf.position.y=14;

        group.add(leaf);
    }

    return group;
}


/* =========================================================
   CLOUDS
========================================================= */

function setupClouds(){

    for(let i=0;i<35;i++){

        const cloud=new THREE.Group();

        const amount=THREE.MathUtils.randInt(5,11);

        for(let j=0;j<amount;j++){

            const sphere=new THREE.Mesh(
                new THREE.SphereGeometry(
                    THREE.MathUtils.randFloat(18,38),
                    16,
                    16
                ),
                new THREE.MeshStandardMaterial({
                    color:0xffffff,
                    roughness:.9
                })
            );

            sphere.position.set(
                THREE.MathUtils.randFloat(-55,55),
                THREE.MathUtils.randFloat(-15,15),
                THREE.MathUtils.randFloat(-30,30)
            );

            cloud.add(sphere);
        }

        cloud.position.set(
            THREE.MathUtils.randFloatSpread(2500),
            THREE.MathUtils.randFloat(280,550),
            THREE.MathUtils.randFloatSpread(2200)
        );

        const scale=THREE.MathUtils.randFloat(.7,2);

        cloud.scale.setScalar(scale);

        scene.add(cloud);
    }
}


/* =========================================================
   PLAYER
========================================================= */

function createCharacter(color=0x36b9e8){

    const group=new THREE.Group();

    /* legs */

    const legMaterial=new THREE.MeshStandardMaterial({
        color:0xffffff,
        roughness:.35
    });

    const legGeo=new THREE.CapsuleGeometry(
        3,
        12,
        6,
        12
    );

    const leg1=new THREE.Mesh(legGeo,legMaterial);
    const leg2=new THREE.Mesh(legGeo,legMaterial);

    leg1.position.set(-4,10,0);
    leg2.position.set(4,10,0);

    group.add(leg1,leg2);

    /* shoes */

    const shoeMat=new THREE.MeshStandardMaterial({
        color:0x0f7bb0,
        roughness:.25
    });

    const shoeGeo=new THREE.SphereGeometry(4,16,12);

    const shoe1=new THREE.Mesh(shoeGeo,shoeMat);
    const shoe2=new THREE.Mesh(shoeGeo,shoeMat);

    shoe1.scale.z=1.5;
    shoe2.scale.z=1.5;

    shoe1.position.set(-4,3,2);
    shoe2.position.set(4,3,2);

    group.add(shoe1,shoe2);

    /* torso */

    const torso=new THREE.Mesh(
        new THREE.CapsuleGeometry(7,14,8,16),
        new THREE.MeshStandardMaterial({
            color:color,
            roughness:.3,
            metalness:.05
        })
    );

    torso.position.y=27;

    group.add(torso);

    /* arms */

    const armGeo=new THREE.CapsuleGeometry(
        2.5,
        11,
        6,
        10
    );

    const armMat=new THREE.MeshStandardMaterial({
        color:color,
        roughness:.3
    });

    const leftArm=new THREE.Mesh(armGeo,armMat);
    const rightArm=new THREE.Mesh(armGeo,armMat);

    leftArm.position.set(-10,28,0);
    rightArm.position.set(10,28,0);

    leftArm.rotation.z=-.12;
    rightArm.rotation.z=.12;

    group.add(leftArm,rightArm);

    /* hands */

    const handMat=new THREE.MeshStandardMaterial({
        color:0xffd0a7,
        roughness:.5
    });

    const handGeo=new THREE.SphereGeometry(3.2,16,12);

    const hand1=new THREE.Mesh(handGeo,handMat);
    const hand2=new THREE.Mesh(handGeo,handMat);

    hand1.position.set(-11,19,0);
    hand2.position.set(11,19,0);

    group.add(hand1,hand2);

    /* head */

    const head=new THREE.Mesh(
        new THREE.SphereGeometry(8.5,24,18),
        new THREE.MeshStandardMaterial({
            color:0xffd0a7,
            roughness:.55
        })
    );

    head.position.y=47;

    group.add(head);

    /* hair */

    const hair=new THREE.Mesh(
        new THREE.SphereGeometry(8.8,20,12),
        new THREE.MeshStandardMaterial({
            color:0x164d5f,
            roughness:.65
        })
    );

    hair.scale.y=.6;
    hair.position.set(0,52,0);

    group.add(hair);

    /* eyes */

    const eyeMat=new THREE.MeshStandardMaterial({
        color:0x073b55,
        emissive:0x0b6f96,
        emissiveIntensity:.25
    });

    const eyeGeo=new THREE.SphereGeometry(1.4,12,8);

    const eye1=new THREE.Mesh(eyeGeo,eyeMat);
    const eye2=new THREE.Mesh(eyeGeo,eyeMat);

    eye1.position.set(-3,48,7.3);
    eye2.position.set(3,48,7.3);

    group.add(eye1,eye2);

    /* smile */

    const smile=new THREE.Mesh(
        new THREE.TorusGeometry(
            2.2,
            .35,
            8,
            20,
            Math.PI
        ),
        new THREE.MeshBasicMaterial({
            color:0x9c3f54
        })
    );

    smile.position.set(0,44.5,7.6);
    smile.rotation.x=Math.PI/2;

    group.add(smile);

    group.userData={
        leftArm,
        rightArm,
        leftLeg:leg1,
        rightLeg:leg2
    };

    return group;
}


function setupCharacters(){

    player=createCharacter(0x23b9df);

    player.position.set(0,0,0);

    scene.add(player);

    /* Aero citizens */

    for(let i=0;i<12;i++){

        const npc=createCharacter(
            [0x32c4e8,0x6ddc77,0xffc94a,0xc878ff][i%4]
        );

        npc.scale.setScalar(.75);

        npc.position.set(
            THREE.MathUtils.randFloatSpread(1000),
            0,
            THREE.MathUtils.randFloatSpread(1000)
        );

        npc.userData.phase=Math.random()*10;

        scene.add(npc);
    }
}


/* =========================================================
   DOLPHIN
========================================================= */

function setupDolphin(){

    dolphin=new THREE.Group();

    /* body */

    const body=new THREE.Mesh(
        new THREE.SphereGeometry(15,32,20),
        new THREE.MeshPhysicalMaterial({
            color:0x65c9e8,
            roughness:.22,
            metalness:.05
        })
    );

    body.scale.set(2.2,.72,.8);

    dolphin.add(body);

    /* snout */

    const snout=new THREE.Mesh(
        new THREE.SphereGeometry(6,20,12),
        new THREE.MeshPhysicalMaterial({
            color:0x58bddc,
            roughness:.2
        })
    );

    snout.scale.set(1.6,.45,.5);
    snout.position.set(30,0,0);

    dolphin.add(snout);

    /* dorsal fin */

    const fin=new THREE.Mesh(
        new THREE.ConeGeometry(7,14,4),
        new THREE.MeshStandardMaterial({
            color:0x45aaca
        })
    );

    fin.rotation.z=Math.PI/2;
    fin.position.set(-2,10,0);

    dolphin.add(fin);

    /* side fins */

    const fin1=fin.clone();
    const fin2=fin.clone();

    fin1.scale.set(.6,.6,.6);
    fin2.scale.set(.6,.6,.6);

    fin1.position.set(3,-1,11);
    fin2.position.set(3,-1,-11);

    fin1.rotation.z=-Math.PI/2;
    fin2.rotation.z=-Math.PI/2;

    dolphin.add(fin1,fin2);

    /* tail */

    const tail=new THREE.Group();

    const tail1=new THREE.Mesh(
        new THREE.ConeGeometry(8,14,4),
        new THREE.MeshStandardMaterial({
            color:0x45aaca
        })
    );

    const tail2=tail1.clone();

    tail1.rotation.z=-Math.PI/2;
    tail2.rotation.z=-Math.PI/2;

    tail1.position.y=7;
    tail2.position.y=-7;

    tail.add(tail1,tail2);

    tail.position.x=-35;

    dolphin.add(tail);

    /* eyes */

    const eyeMat=new THREE.MeshStandardMaterial({
        color:0x082f48
    });

    const e1=new THREE.Mesh(
        new THREE.SphereGeometry(1.8,12,8),
        eyeMat
    );

    const e2=e1.clone();

    e1.position.set(24,5,7);
    e2.position.set(24,5,-7);

    dolphin.add(e1,e2);

    dolphin.position.set(100,15,-180);

    scene.add(dolphin);
}


/* =========================================================
   COLLECTIBLES
========================================================= */

const collectibles=[];

function setupCollectibles(){

    for(let i=0;i<45;i++){

        const crystal=new THREE.Mesh(
            new THREE.OctahedronGeometry(4,1),
            new THREE.MeshPhysicalMaterial({
                color:0x65f4ff,
                emissive:0x00b9df,
                emissiveIntensity:.7,
                transparent:true,
                opacity:.88,
                roughness:.05
            })
        );

        crystal.position.set(
            THREE.MathUtils.randFloatSpread(1500),
            THREE.MathUtils.randFloat(8,35),
            THREE.MathUtils.randFloatSpread(1400)
        );

        scene.add(crystal);

        collectibles.push(crystal);
    }
}


/* =========================================================
   UI
========================================================= */

function setupUI(){

    const joystick=document.getElementById("joystick");
    const knob=document.getElementById("knob");

    let active=false;

    function moveJoystick(e){

        const rect=joystick.getBoundingClientRect();

        let x=e.clientX-rect.left-rect.width/2;
        let y=e.clientY-rect.top-rect.height/2;

        const max=45;

        const length=Math.hypot(x,y);

        if(length>max){

            x=x/length*max;
            y=y/length*max;
        }

        joystickX=x/max;
        joystickY=y/max;

        knob.style.left=`calc(50% + ${x}px)`;
        knob.style.top=`calc(50% + ${y}px)`;
    }

    joystick.addEventListener("pointerdown",e=>{

        active=true;
        joystick.setPointerCapture(e.pointerId);

        moveJoystick(e);
    });

    joystick.addEventListener("pointermove",e=>{

        if(active)moveJoystick(e);
    });

    joystick.addEventListener("pointerup",()=>{

        active=false;

        joystickX=0;
        joystickY=0;

        knob.style.left="50%";
        knob.style.top="50%";
    });


    /* camera swipe */

    renderer.domElement.addEventListener("pointerdown",e=>{

        if(e.clientX<160)return;

        cameraTouch=true;

        cameraTouchX=e.clientX;
        cameraTouchY=e.clientY;
    });

    renderer.domElement.addEventListener("pointermove",e=>{

        if(!cameraTouch)return;

        const dx=e.clientX-cameraTouchX;
        const dy=e.clientY-cameraTouchY;

        cameraYaw-=dx*.006;
        cameraPitch-=dy*.004;

        cameraPitch=Math.max(-.25,Math.min(.8,cameraPitch));

        cameraTouchX=e.clientX;
        cameraTouchY=e.clientY;
    });

    renderer.domElement.addEventListener("pointerup",()=>{
        cameraTouch=false;
    });


    document.getElementById("jump").addEventListener(
        "pointerdown",
        jump
    );

    document.getElementById("ride").addEventListener(
        "pointerdown",
        toggleRide
    );
}


/* =========================================================
   JUMP
========================================================= */

function jump(){

    if(player.position.y<=.1){

        verticalVelocity=14;
    }

    if(riding){

        verticalVelocity=18;
    }
}


/* =========================================================
   RIDE DOLPHIN
========================================================= */

function toggleRide(){

    if(!riding){

        const distance=player.position.distanceTo(
            dolphin.position
        );

        if(distance<100){

            riding=true;

            status.style.display="block";

            message.textContent=
                "🐬 You're riding! Swipe to look around.";
        }

    }else{

        riding=false;

        verticalVelocity=7;

        status.style.display="none";

        message.textContent=
            "Explore the Aero World and find more islands!";
    }
}


/* =========================================================
   UPDATE PLAYER
========================================================= */

function updatePlayer(dt){

    let forward=-joystickY;
    let sideways=joystickX;

    const speed=riding?150:65;

    const direction=new THREE.Vector3(
        Math.sin(cameraYaw),
        0,
        Math.cos(cameraYaw)
    );

    const right=new THREE.Vector3(
        Math.cos(cameraYaw),
        0,
        -Math.sin(cameraYaw)
    );

    const movement=new THREE.Vector3();

    movement.addScaledVector(direction,forward);
    movement.addScaledVector(right,sideways);

    if(movement.lengthSq()>0){

        movement.normalize();

        velocity.lerp(
            movement.multiplyScalar(speed),
            .12
        );

        player.rotation.y=
            Math.atan2(
                velocity.x,
                velocity.z
            );

    }else{

        velocity.multiplyScalar(.88);
    }

    player.position.x+=velocity.x*dt;
    player.position.z+=velocity.z*dt;

    /* gravity */

    player.position.y+=verticalVelocity*dt;

    verticalVelocity-=35*dt;

    if(player.position.y<0){

        player.position.y=0;
        verticalVelocity=0;
    }

    /* walking animation */

    const moving=velocity.length()>5;

    if(moving){

        const t=performance.now()*.012;

        player.userData.leftArm.rotation.x=
            Math.sin(t)*.5;

        player.userData.rightArm.rotation.x=
            -Math.sin(t)*.5;

        player.userData.leftLeg.rotation.x=
            -Math.sin(t)*.65;

        player.userData.rightLeg.rotation.x=
            Math.sin(t)*.65;

    }else{

        player.userData.leftArm.rotation.x*=.8;
        player.userData.rightArm.rotation.x*=.8;

        player.userData.leftLeg.rotation.x*=.8;
        player.userData.rightLeg.rotation.x*=.8;
    }

    /* riding */

    if(riding){

        dolphin.position.copy(player.position);

        dolphin.position.y+=12;

        dolphin.rotation.y=
            player.rotation.y;

        player.position.y=
            dolphin.position.y+23;

        player.rotation.y=
            dolphin.rotation.y;
    }
}


/* =========================================================
   DOLPHIN AI
========================================================= */

function updateDolphin(){

    if(riding){

        dolphin.rotation.z=
            Math.sin(performance.now()*.005)*.08;

        return;
    }

    const t=performance.now()*.0005;

    dolphin.position.x=
        100+Math.sin(t)*100;

    dolphin.position.z=
        -180+Math.cos(t*.8)*100;

    dolphin.position.y=
        15+Math.sin(t*3)*7;

    dolphin.rotation.y=
        Math.atan2(
            Math.cos(t),
            -Math.sin(t)
        );

    dolphin.rotation.z=
        Math.sin(t*3)*.08;
}


/* =========================================================
   COLLECTIBLES
========================================================= */

function updateCollectibles(){

    for(const crystal of collectibles){

        crystal.rotation.y+=.025;
        crystal.rotation.x+=.012;

        crystal.position.y+=
            Math.sin(
                performance.now()*.002+
                crystal.position.x
            )*.008;

        if(
            crystal.position.distanceTo(
                player.position
            )<28
        ){

            crystal.position.set(
                THREE.MathUtils.randFloatSpread(1500),
                THREE.MathUtils.randFloat(8,35),
                THREE.MathUtils.randFloatSpread(1400)
            );

            score++;

            scoreText.textContent=score;
        }
    }
}


/* =========================================================
   CAMERA
========================================================= */

function updateCamera(){

    const distance=riding?115:135;

    const target=new THREE.Vector3();

    target.copy(player.position);

    target.y+=riding?25:38;

    const offset=new THREE.Vector3(
        Math.sin(cameraYaw)*distance,
        45+cameraPitch*80,
        Math.cos(cameraYaw)*distance
    );

    const desired=new THREE.Vector3();

    desired.copy(target).add(offset);

    camera.position.lerp(
        desired,
        .08
    );

    camera.lookAt(target);
}


/* =========================================================
   ANIMATION
========================================================= */

function animate(){

    requestAnimationFrame(animate);

    const dt=Math.min(
        clock.getDelta(),
        .033
    );

    updatePlayer(dt);
    updateDolphin();
    updateCollectibles();
    updateCamera();

    const currentSpeed=
        Math.round(velocity.length());

    speedText.textContent=currentSpeed;

    if(!riding){

        const distance=
            player.position.distanceTo(
                dolphin.position
            );

        if(distance<100){

            message.textContent=
                "🐬 Press RIDE to ride the dolphin!";
        }
    }

    renderer.render(
        scene,
        camera
    );
}


/* =========================================================
   RESIZE
========================================================= */

function resize(){

    camera.aspect=
        innerWidth/innerHeight;

    camera.updateProjectionMatrix();

    renderer.setSize(
        innerWidth,
        innerHeight
    );

    renderer.setPixelRatio(
        Math.min(devicePixelRatio,2)
    );
}


/* =========================================================
   KEYBOARD SUPPORT
========================================================= */

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


/* keyboard movement */

setInterval(()=>{

    let x=0;
    let y=0;

    if(keys["w"]||keys["arrowup"])y=-1;
    if(keys["s"]||keys["arrowdown"])y=1;
    if(keys["a"]||keys["arrowleft"])x=-1;
    if(keys["d"]||keys["arrowright"])x=1;

    if(x||y){

        joystickX=x;
        joystickY=y;

    }else if(
        !document.getElementById("joystick").matches(":active")
    ){

        /* don't override touch joystick */
    }

},16);

</script>
</body>
</html>
