<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width,
initial-scale=1,
maximum-scale=1,
user-scalable=no">

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
    background:#58ddff;
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

#title{
    position:fixed;
    top:15px;
    left:20px;
    color:white;
    font-size:21px;
    font-weight:900;
    text-shadow:0 2px 5px #17658a;
    pointer-events:none;
    z-index:5;
}

#subtitle{
    position:fixed;
    top:43px;
    left:21px;
    color:white;
    font-size:11px;
    text-shadow:0 2px 5px #17658a;
    pointer-events:none;
    z-index:5;
}

#joystick{
    position:fixed;
    left:22px;
    bottom:22px;
    width:128px;
    height:128px;
    border-radius:50%;
    background:rgba(255,255,255,.17);
    border:3px solid rgba(255,255,255,.8);
    box-shadow:
        0 8px 25px rgba(0,70,110,.25),
        inset 0 0 20px rgba(255,255,255,.3);
    z-index:20;
    touch-action:none;
}

#stick{
    position:absolute;
    width:58px;
    height:58px;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    border-radius:50%;
    background:linear-gradient(
        145deg,
        #fff,
        #a7efff
    );
    border:3px solid white;
    box-shadow:
        0 5px 15px rgba(0,80,120,.25),
        inset 0 0 10px white;
}

#buttons{
    position:fixed;
    right:20px;
    bottom:20px;
    display:flex;
    gap:10px;
    z-index:20;
}

button{
    width:74px;
    height:74px;
    border-radius:50%;
    border:3px solid white;
    background:linear-gradient(
        #a5f7ff,
        #159ed0
    );
    color:white;
    font-size:10px;
    font-weight:900;
    box-shadow:
        0 7px 20px rgba(0,70,110,.3),
        inset 0 3px 9px rgba(255,255,255,.6);
    touch-action:none;
}

button:active{
    transform:scale(.9);
}

#shift.active{
    background:linear-gradient(
        #fff38a,
        #ffad22
    );
    color:#684600;
}

#rotate{
    display:none;
    position:fixed;
    inset:0;
    z-index:100;
    align-items:center;
    justify-content:center;
    flex-direction:column;
    text-align:center;
    color:#087b9e;
    font-weight:900;
    background:linear-gradient(
        #58ddff,
        #e9ffff
    );
}

#rotateIcon{
    font-size:65px;
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

<div id="title">
FRUTIGER AERO WORLD
</div>

<div id="subtitle">
Beach • Grassland • Forest • Explore
</div>

<div id="joystick">
    <div id="stick"></div>
</div>

<div id="buttons">
    <button id="shift">SHIFT<br>LOCK</button>
    <button id="jump">JUMP</button>
    <button id="ride">RIDE</button>
</div>

<div id="rotate">
    <div id="rotateIcon">↔️</div>
    TURN YOUR iPAD SIDEWAYS
    <small style="margin-top:8px">
        Landscape mode gives you the full world.
    </small>
</div>


<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

/* =========================================================
   RENDERER
========================================================= */

const canvas=document.getElementById("game");

const renderer=new THREE.WebGLRenderer({
    canvas:canvas,
    antialias:true,
    powerPreference:"high-performance"
});

renderer.setSize(innerWidth,innerHeight);

renderer.setPixelRatio(
    Math.min(window.devicePixelRatio,1.5)
);

renderer.outputColorSpace=THREE.SRGBColorSpace;

renderer.toneMapping=THREE.ACESFilmicToneMapping;

renderer.toneMappingExposure=1.12;

renderer.shadowMap.enabled=true;

renderer.shadowMap.type=THREE.PCFSoftShadowMap;


/* =========================================================
   SCENE
========================================================= */

const scene=new THREE.Scene();

scene.background=new THREE.Color(0x5bdfff);

scene.fog=new THREE.Fog(
    0x72e4ff,
    120,
    650
);


/* =========================================================
   CAMERA
========================================================= */

const camera=new THREE.PerspectiveCamera(
    67,
    innerWidth/innerHeight,
    .1,
    1000
);


/* =========================================================
   LIGHT
========================================================= */

scene.add(
    new THREE.HemisphereLight(
        0xeaffff,
        0x287142,
        2.3
    )
);

const sun=new THREE.DirectionalLight(
    0xffffff,
    3.5
);

sun.position.set(
    -150,
    210,
    120
);

sun.castShadow=true;

sun.shadow.mapSize.width=2048;
sun.shadow.mapSize.height=2048;

sun.shadow.camera.left=-230;
sun.shadow.camera.right=230;
sun.shadow.camera.top=230;
sun.shadow.camera.bottom=-230;

sun.shadow.camera.near=10;
sun.shadow.camera.far=500;

sun.shadow.bias=-.0002;
sun.shadow.normalBias=.015;

scene.add(sun);


/* =========================================================
   TERRAIN
   ONE CONTINUOUS MESH
========================================================= */

const WORLD=205;
const GRID=120;

const pos=[];
const col=[];
const idx=[];

function heightAt(x,z){

    const r=Math.sqrt(x*x+z*z);

    let h=
        3.8+
        Math.sin(x*.028)*1.5+
        Math.cos(z*.034)*1.4+
        Math.sin((x+z)*.018)*1.2+
        Math.cos((x-z)*.022)*.8;

    /*
       Make the outer area a gently
       sloping beach.
    */

    if(r>125){

        const t=Math.min(
            1,
            (r-125)/62
        );

        h-=t*5;
    }

    return h;
}

const beach=new THREE.Color(0xf2dfa0);
const grass=new THREE.Color(0x39ca58);
const forest=new THREE.Color(0x239d4c);

for(let z=0;z<=GRID;z++){

    for(let x=0;x<=GRID;x++){

        const px=
            -WORLD+
            x/GRID*WORLD*2;

        const pz=
            -WORLD+
            z/GRID*WORLD*2;

        const r=
            Math.sqrt(px*px+pz*pz);

        let py=heightAt(px,pz);

        /*
           Outside island = deep underwater.
           This gives the island a clean edge.
        */

        if(r>190){

            py=-12-(r-190)*1.5;
        }

        pos.push(px,py,pz);

        let c;

        if(r>126){

            c=beach.clone();

        }else if(
            r>65 &&
            px<85 &&
            pz<45
        ){

            c=forest.clone();

        }else{

            c=grass.clone();
        }

        const variation=
            .95+
            Math.sin(
                px*.15+pz*.11
            )*.035;

        c.multiplyScalar(variation);

        col.push(
            c.r,
            c.g,
            c.b
        );
    }
}

for(let z=0;z<GRID;z++){

    for(let x=0;x<GRID;x++){

        const a=z*(GRID+1)+x;
        const b=a+1;
        const c=a+GRID+1;
        const d=c+1;

        idx.push(a,b,d);
        idx.push(a,d,c);
    }
}

const terrainGeometry=
    new THREE.BufferGeometry();

terrainGeometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(
        pos,3
    )
);

terrainGeometry.setAttribute(
    "color",
    new THREE.Float32BufferAttribute(
        col,3
    )
);

terrainGeometry.setIndex(idx);

terrainGeometry.computeVertexNormals();

const terrainMaterial=
    new THREE.MeshStandardMaterial({
        vertexColors:true,
        roughness:.85,
        metalness:.01
    });

const terrain=
    new THREE.Mesh(
        terrainGeometry,
        terrainMaterial
    );

terrain.receiveShadow=true;

scene.add(terrain);


/* =========================================================
   WATER
========================================================= */

const water=
    new THREE.Mesh(
        new THREE.PlaneGeometry(
            1000,
            1000
        ),
        new THREE.MeshPhysicalMaterial({
            color:0x12c7e8,
            transparent:true,
            opacity:.88,
            roughness:.12,
            metalness:.02,
            clearcoat:.8
        })
    );

water.rotation.x=-Math.PI/2;
water.position.y=-1.8;

scene.add(water);


/* =========================================================
   TREE GEOMETRY
========================================================= */

const trunkGeo=
    new THREE.CylinderGeometry(
        .38,
        .58,
        6,
        10
    );

const leafGeo=
    new THREE.SphereGeometry(
        1,
        12,
        10
    );

const trunkMat=
    new THREE.MeshStandardMaterial({
        color:0x96603a,
        roughness:.9
    });

const leafMat=
    new THREE.MeshStandardMaterial({
        color:0x159b49,
        roughness:.72
    });

const leafLightMat=
    new THREE.MeshStandardMaterial({
        color:0x36d65d,
        roughness:.68
    });


function tree(x,z,s=.8){

    const y=heightAt(x,z);

    const g=new THREE.Group();

    const trunk=new THREE.Mesh(
        trunkGeo,
        trunkMat
    );

    trunk.scale.set(s,s,s);
    trunk.position.y=3*s;
    trunk.castShadow=true;

    g.add(trunk);


    for(let i=0;i<5;i++){

        const leaf=new THREE.Mesh(
            leafGeo,
            i%2
            ?leafMat
            :leafLightMat
        );

        const angle=
            i*Math.PI*2/5;

        leaf.scale.set(
            1.65*s,
            .9*s,
            1.2*s
        );

        leaf.position.set(
            Math.cos(angle)*1.15*s,
            (6.1+(i%2)*.5)*s,
            Math.sin(angle)*1.15*s
        );

        leaf.castShadow=true;

        g.add(leaf);
    }

    g.position.set(x,y,z);

    g.rotation.y=
        Math.random()*Math.PI*2;

    scene.add(g);
}


/* =========================================================
   FOREST
========================================================= */

for(let i=0;i<145;i++){

    const x=
        -110+
        Math.random()*180;

    const z=
        -120+
        Math.random()*150;

    const r=
        Math.sqrt(x*x+z*z);

    if(
        r>67 &&
        r<130 &&
        x<85 &&
        z<45
    ){

        tree(
            x,
            z,
            .75+
            Math.random()*.5
        );
    }
}


/* =========================================================
   SCATTERED GRASSLAND TREES
========================================================= */

for(let i=0;i<35;i++){

    const a=Math.random()*Math.PI*2;
    const r=45+Math.random()*75;

    const x=Math.cos(a)*r;
    const z=Math.sin(a)*r;

    if(
        !(r>67 && x<85 && z<45)
    ){

        tree(
            x,
            z,
            .65+
            Math.random()*.35
        );
    }
}


/* =========================================================
   FRUTIGER AERO GUY
========================================================= */

function aeroGuy(){

    const g=new THREE.Group();


    const blue=new THREE.MeshPhysicalMaterial({
        color:0x27c8f5,
        roughness:.12,
        clearcoat:1,
        clearcoatRoughness:.035
    });


    const darkBlue=new THREE.MeshPhysicalMaterial({
        color:0x118dd0,
        roughness:.13,
        clearcoat:1
    });


    /* BODY */

    const body=new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            32,
            24
        ),
        blue
    );

    body.scale.set(
        1.28,
        1.42,
        .9
    );

    body.position.y=1.8;

    body.castShadow=true;

    g.add(body);


    /* HEAD */

    const head=new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            32,
            24
        ),
        blue
    );

    head.position.y=3.75;

    head.castShadow=true;

    g.add(head);


    /* HEAD INNER SHADING */

    const innerHead=new THREE.Mesh(
        new THREE.SphereGeometry(
            .94,
            24,
            18
        ),
        darkBlue
    );

    innerHead.position.y=3.72;
    innerHead.scale.set(
        .96,
        .96,
        .85
    );

    innerHead.material.transparent=true;
    innerHead.material.opacity=.28;

    g.add(innerHead);


    /* HEAD HIGHLIGHT */

    const headHighlight=new THREE.Mesh(
        new THREE.SphereGeometry(
            .3,
            20,
            16
        ),
        new THREE.MeshBasicMaterial({
            color:whiteColor()
        })
    );

    headHighlight.scale.set(
        1.5,
        .55,
        .35
    );

    headHighlight.position.set(
        -.35,
        4.12,
        -.84
    );

    g.add(headHighlight);


    /* ARMS */

    const leftArm=new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            24,
            18
        ),
        blue
    );

    leftArm.scale.set(
        .42,
        .98,
        .48
    );

    leftArm.position.set(
        -1.3,
        1.45,
        0
    );

    leftArm.rotation.z=.16;

    leftArm.castShadow=true;

    g.add(leftArm);


    const rightArm=new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            24,
            18
        ),
        blue
    );

    rightArm.scale.set(
        .42,
        .98,
        .48
    );

    rightArm.position.set(
        1.3,
        1.45,
        0
    );

    rightArm.rotation.z=-.16;

    rightArm.castShadow=true;

    g.add(rightArm);


    /* LEGS */

    const leftLeg=new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            24,
            18
        ),
        blue
    );

    leftLeg.scale.set(
        .5,
        .85,
        .55
    );

    leftLeg.position.set(
        -.53,
        .25,
        0
    );

    leftLeg.castShadow=true;

    g.add(leftLeg);


    const rightLeg=new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            24,
            18
        ),
        blue
    );

    rightLeg.scale.set(
        .5,
        .85,
        .55
    );

    rightLeg.position.set(
        .53,
        .25,
        0
    );

    rightLeg.castShadow=true;

    g.add(rightLeg);


    g.userData.parts={
        body,
        head,
        leftArm,
        rightArm,
        leftLeg,
        rightLeg
    };

    return g;
}


function whiteColor(){
    return 0xffffff;
}


const player=aeroGuy();

const startX=0;
const startZ=35;

player.position.set(
    startX,
    heightAt(startX,startZ)+.72,
    startZ
);

scene.add(player);


/* =========================================================
   DOLPHIN
========================================================= */

function makeDolphin(){

    const g=new THREE.Group();

    const mat=new THREE.MeshPhysicalMaterial({
        color:0x42c8e9,
        roughness:.12,
        clearcoat:1
    });


    const body=new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            32,
            20
        ),
        mat
    );

    body.scale.set(
        2.4,
        .7,
        1
    );

    g.add(body);


    const nose=new THREE.Mesh(
        new THREE.ConeGeometry(
            .42,
            1.7,
            16
        ),
        mat
    );

    nose.rotation.z=-Math.PI/2;

    nose.position.z=-2.25;

    g.add(nose);


    const fin=new THREE.Mesh(
        new THREE.ConeGeometry(
            .4,
            1.35,
            16
        ),
        mat
    );

    fin.rotation.z=Math.PI;

    fin.position.y=.8;

    g.add(fin);


    const tail1=new THREE.Mesh(
        new THREE.ConeGeometry(
            .4,
            1.4,
            16
        ),
        mat
    );

    tail1.rotation.z=.8;
    tail1.position.set(-.4,0,2.1);

    g.add(tail1);


    const tail2=tail1.clone();

    tail2.rotation.z=-.8;

    g.add(tail2);

    return g;
}


const dolphin=makeDolphin();

dolphin.position.set(
    0,
    -.15,
    -145
);

scene.add(dolphin);


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


function updateJoystick(e){

    const r=
        joystick.getBoundingClientRect();

    let x=
        e.clientX-
        (r.left+r.width/2);

    let y=
        e.clientY-
        (r.top+r.height/2);

    const max=43;

    const len=Math.hypot(x,y);

    if(len>max){

        x=x/len*max;
        y=y/len*max;
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

    moveX=0;
    moveZ=0;

    joystickPointer=null;

    stick.style.transform=
        "translate(-50%,-50%)";
}


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

        if(
            e.pointerId===joystickPointer
        ){

            updateJoystick(e);
        }
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


/* =========================================================
   CAMERA CONTROL
========================================================= */

let yaw=0;
let pitch=.18;

let dragging=false;

let pointerId=null;

let lastX=0;
let lastY=0;


/*
   SHIFT LOCK STATE
*/

let shiftLock=false;


/*
   Normal camera can rotate.

   Shift Lock camera CAN NOT be manually
   rotated around the player.
*/

canvas.addEventListener(
    "pointerdown",
    e=>{

        if(shiftLock)
            return;

        dragging=true;

        pointerId=e.pointerId;

        lastX=e.clientX;
        lastY=e.clientY;

        canvas.setPointerCapture(
            e.pointerId
        );
    }
);


canvas.addEventListener(
    "pointermove",
    e=>{

        if(
            !dragging ||
            e.pointerId!==pointerId ||
            shiftLock
        )
            return;

        const dx=
            e.clientX-lastX;

        const dy=
            e.clientY-lastY;

        yaw-=dx*.006;

        pitch-=dy*.004;

        pitch=THREE.MathUtils.clamp(
            pitch,
            -.3,
            .65
        );

        lastX=e.clientX;
        lastY=e.clientY;
    }
);


canvas.addEventListener(
    "pointerup",
    ()=>{
        dragging=false;
        pointerId=null;
    }
);


/* =========================================================
   SHIFT LOCK BUTTON
========================================================= */

const shiftButton=
    document.getElementById("shift");


shiftButton.addEventListener(
    "pointerdown",
    e=>{

        e.stopPropagation();

        shiftLock=!shiftLock;

        shiftButton.classList.toggle(
            "active",
            shiftLock
        );


        if(shiftLock){

            /*
               Snap camera to the direction
               the character is facing.

               After this, the camera angle
               is NOT user controlled.
            */

            yaw=
                player.rotation.y;

            pitch=.18;
        }
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

    velocityY=10.5;
}

document
.getElementById("jump")
.addEventListener(
    "pointerdown",
    e=>{

        e.stopPropagation();

        jump();
    }
);


/* =========================================================
   RIDE
========================================================= */

let riding=false;

document
.getElementById("ride")
.addEventListener(
    "pointerdown",
    e=>{

        e.stopPropagation();

        const d=
            player.position.distanceTo(
                dolphin.position
            );

        if(!riding && d<12){

            riding=true;

        }else if(riding){

            riding=false;
        }
    }
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

        if(
            e.key.toLowerCase()==="e"
        ){

            const d=
                player.position.distanceTo(
                    dolphin.position
                );

            if(!riding && d<12)
                riding=true;
            else if(riding)
                riding=false;
        }

        if(
            e.key.toLowerCase()==="shift"
        ){

            shiftLock=!shiftLock;

            shiftButton.classList.toggle(
                "active",
                shiftLock
            );

            if(shiftLock)
                yaw=player.rotation.y;
        }
    }
);

window.addEventListener(
    "keyup",
    e=>{
        keys[e.key.toLowerCase()]=false;
    }
);


/* =========================================================
   CHARACTER ANIMATION
========================================================= */

function animateGuy(
    guy,
    moving,
    time,
    speed
){

    const p=guy.userData.parts;

    if(!p)
        return;


    if(moving){

        const cycle=
            time*
            (7+
            speed*.45);

        const s=
            Math.sin(cycle);

        const o=
            Math.sin(cycle+Math.PI);


        p.leftLeg.rotation.x=
            s*.62;

        p.rightLeg.rotation.x=
            o*.62;

        p.leftArm.rotation.x=
            o*.48;

        p.rightArm.rotation.x=
            s*.48;

        p.body.position.y=
            1.8+
            Math.abs(
                Math.sin(cycle*2)
            )*.055;

        p.head.position.y=
            3.75+
            Math.abs(
                Math.sin(cycle*2)
            )*.045;

    }else{

        p.leftLeg.rotation.x=
            THREE.MathUtils.lerp(
                p.leftLeg.rotation.x,
                0,
                .16
            );

        p.rightLeg.rotation.x=
            THREE.MathUtils.lerp(
                p.rightLeg.rotation.x,
                0,
                .16
            );

        p.leftArm.rotation.x=
            THREE.MathUtils.lerp(
                p.leftArm.rotation.x,
                0,
                .16
            );

        p.rightArm.rotation.x=
            THREE.MathUtils.lerp(
                p.rightArm.rotation.x,
                0,
                .16
            );
    }
}


/* =========================================================
   MAIN LOOP
========================================================= */

const clock=new THREE.Clock();


function animate(){

    requestAnimationFrame(animate);

    const dt=
        Math.min(
            clock.getDelta(),
            .033
        );

    const time=
        clock.elapsedTime;


    /* -------------------------------------------------------
       KEYBOARD MOVEMENT
    ------------------------------------------------------- */

    let kx=0;
    let kz=0;

    if(
        keys["a"]||
        keys["arrowleft"]
    )
        kx=-1;

    if(
        keys["d"]||
        keys["arrowright"]
    )
        kx=1;

    if(
        keys["w"]||
        keys["arrowup"]
    )
        kz=-1;

    if(
        keys["s"]||
        keys["arrowdown"]
    )
        kz=1;


    if(kx!==0||kz!==0){

        moveX=kx;
        moveZ=kz;
    }


    /* -------------------------------------------------------
       CAMERA DIRECTIONS
    ------------------------------------------------------- */

    const forward=
        new THREE.Vector3(
            Math.sin(yaw),
            0,
            -Math.cos(yaw)
        );

    const right=
        new THREE.Vector3(
            Math.cos(yaw),
            0,
            Math.sin(yaw)
        );


    /*
       IMPORTANT:

       joystick UP = forward

       This fixes the old inverted movement.
    */

    const direction=
        new THREE.Vector3();

    direction.addScaledVector(
        right,
        moveX
    );

    direction.addScaledVector(
        forward,
        -moveZ
    );


    let moving=false;
    let speed=0;


    if(direction.lengthSq()>0.001){

        direction.normalize();

        moving=true;

        speed=
            Math.min(
                1,
                Math.hypot(
                    moveX,
                    moveZ
                )
            )*
            (riding?16:7.5);


        if(riding){

            dolphin.position.addScaledVector(
                direction,
                speed*dt
            );

        }else{

            player.position.addScaledVector(
                direction,
                speed*dt
            );
        }


        /*
           CHARACTER FACES FORWARD.

           No backwards walking animation.
           No turning toward the camera.
        */

        let desired;

        if(shiftLock){

            /*
               In Shift Lock, movement is
               camera-relative and the character
               stays facing camera-forward.
            */

            desired=yaw;

        }else{

            /*
               Normal mode: face actual movement.
            */

            desired=
                Math.atan2(
                    direction.x,
                    direction.z
                );
        }


        let diff=
            desired-
            player.rotation.y;

        while(diff>Math.PI)
            diff-=Math.PI*2;

        while(diff<-Math.PI)
            diff+=Math.PI*2;

        player.rotation.y+=
            diff*
            Math.min(
                1,
                dt*12
            );
    }


    /* -------------------------------------------------------
       GROUND
    ------------------------------------------------------- */

    if(!riding){

        const ground=
            heightAt(
                player.position.x,
                player.position.z
            )+.72;


        if(!jumping){

            player.position.y=
                THREE.MathUtils.lerp(
                    player.position.y,
                    ground,
                    .35
                );
        }

    }


    /* -------------------------------------------------------
       JUMP
    ------------------------------------------------------- */

    if(jumping){

        velocityY-=25*dt;

        if(riding){

            dolphin.position.y+=
                velocityY*dt;

            if(
                dolphin.position.y<=-.15
            ){

                dolphin.position.y=-.15;

                velocityY=0;

                jumping=false;
            }

        }else{

            player.position.y+=
                velocityY*dt;

            const ground=
                heightAt(
                    player.position.x,
                    player.position.z
                )+.72;

            if(
                player.position.y<=ground
            ){

                player.position.y=ground;

                velocityY=0;

                jumping=false;
            }
        }
    }


    /* -------------------------------------------------------
       RIDING DOLPHIN
    ------------------------------------------------------- */

    if(riding){

        player.position.x=
            dolphin.position.x;

        player.position.y=
            dolphin.position.y+2.25;

        player.position.z=
            dolphin.position.z;

        /*
           Character follows dolphin direction.
        */

        player.rotation.y=yaw;
    }


    /*
       Dolphin idle movement.
    */

    if(!riding){

        dolphin.position.y=
            -.15+
            Math.sin(time*2.1)*.13;
    }


    dolphin.rotation.z=
        Math.sin(time*2.5)*.04;


    /* -------------------------------------------------------
       WALK ANIMATION
    ------------------------------------------------------- */

    animateGuy(
        player,
        moving,
        time,
        speed
    );


    /* -------------------------------------------------------
       SHIFT LOCK CAMERA
    ------------------------------------------------------- */

    if(shiftLock){

        /*
           Camera is ALWAYS behind the player.

           There is deliberately NO camera drag
           code while Shift Lock is active.
        */

        yaw=
            player.rotation.y;

        pitch=.18;
    }


    /* -------------------------------------------------------
       THIRD PERSON CAMERA
    ------------------------------------------------------- */

    const target=
        riding?
        dolphin:
        player;

    const distance=
        riding?
        15:
        12.5;

    const horizontal=
        Math.cos(pitch)*
        distance;


    const desiredX=
        target.position.x-
        Math.sin(yaw)*
        horizontal;

    const desiredZ=
        target.position.z+
        Math.cos(yaw)*
        horizontal;

    const desiredY=
        target.position.y+
        3+
        Math.sin(pitch)*
        distance;


    camera.position.x=
        THREE.MathUtils.lerp(
            camera.position.x,
            desiredX,
            .13
        );

    camera.position.y=
        THREE.MathUtils.lerp(
            camera.position.y,
            desiredY,
            .13
        );

    camera.position.z=
        THREE.MathUtils.lerp(
            camera.position.z,
            desiredZ,
            .13
        );


    camera.lookAt(
        target.position.x,
        target.position.y+1.65,
        target.position.z
    );


    /* -------------------------------------------------------
       WATER
    ------------------------------------------------------- */

    water.position.y=
        -1.8+
        Math.sin(time)*.012;


    /* -------------------------------------------------------
       RENDER
    ------------------------------------------------------- */

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
        Math.min(
            window.devicePixelRatio,
            1.5
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

animate();

</script>

</body>
</html>
