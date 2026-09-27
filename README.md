<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

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
    background:#69dfff;
    font-family:Arial,sans-serif;
}

canvas{
    display:block;
    width:100%;
    height:100%;
}

#ui{
    position:fixed;
    inset:0;
    pointer-events:none;
}

button{
    border:0;
    color:white;
    font-weight:800;
    pointer-events:auto;
    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
    box-shadow:
        0 7px 20px rgba(0,50,90,.25),
        inset 0 2px 5px rgba(255,255,255,.55);
    backdrop-filter:blur(8px);
}

#title{
    position:absolute;
    top:14px;
    left:50%;
    transform:translateX(-50%);
    color:white;
    font-size:17px;
    white-space:nowrap;
    text-shadow:
        0 2px 5px #15799c,
        0 0 15px rgba(255,255,255,.8);
}

#shift{
    position:absolute;
    top:12px;
    right:12px;
    padding:12px 15px;
    border-radius:16px;
    background:rgba(7,112,180,.75);
}

#shift.active{
    background:rgba(0,190,115,.9);
}

#run{
    position:absolute;
    right:23px;
    bottom:145px;
    width:74px;
    height:74px;
    border-radius:50%;
    background:rgba(15,135,215,.86);
}

#run.active{
    background:#00bd76;
    transform:scale(1.07);
}

#jump{
    position:absolute;
    right:23px;
    bottom:55px;
    width:74px;
    height:74px;
    border-radius:50%;
    background:rgba(0,174,120,.88);
}

#ride{
    position:absolute;
    right:110px;
    bottom:66px;
    width:65px;
    height:65px;
    border-radius:50%;
    background:rgba(0,130,205,.88);
}

#joystick{
    position:absolute;
    left:23px;
    bottom:37px;
    width:130px;
    height:130px;
    border-radius:50%;
    background:rgba(255,255,255,.18);
    border:2px solid rgba(255,255,255,.48);
    box-shadow:
        inset 0 0 25px rgba(255,255,255,.25),
        0 8px 25px rgba(0,60,100,.2);
    pointer-events:auto;
}

#stick{
    position:absolute;
    width:58px;
    height:58px;
    left:50%;
    top:50%;
    margin:-29px;
    border-radius:50%;
    background:
        radial-gradient(
            circle at 30% 25%,
            white,
            #dcfaff 35%,
            #9de4f5
        );
    box-shadow:
        0 6px 16px rgba(0,50,90,.2),
        inset 0 2px 4px white;
}
</style>
</head>

<body>

<div id="ui">

<div id="title">FRUTIGER AERO WORLD</div>

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
   SCENE
========================================================= */

const scene = new THREE.Scene();

scene.background =
    new THREE.Color(0x62d9fa);

scene.fog =
    new THREE.FogExp2(
        0x8be6f5,
        .00165
    );


/* =========================================================
   CAMERA
========================================================= */

const camera =
    new THREE.PerspectiveCamera(
        65,
        innerWidth/innerHeight,
        .1,
        900
    );


/* =========================================================
   RENDERER
========================================================= */

const renderer =
    new THREE.WebGLRenderer({
        antialias:true,
        powerPreference:"high-performance"
    });

renderer.setSize(
    innerWidth,
    innerHeight
);

renderer.setPixelRatio(
    Math.min(
        devicePixelRatio || 1,
        1.35
    )
);

renderer.outputColorSpace =
    THREE.SRGBColorSpace;

renderer.toneMapping =
    THREE.ACESFilmicToneMapping;

renderer.toneMappingExposure =
    1.2;

renderer.shadowMap.enabled=true;

renderer.shadowMap.type =
    THREE.PCFSoftShadowMap;

document.body.appendChild(renderer.domElement);


/* =========================================================
   LIGHT
========================================================= */

const skyLight =
    new THREE.HemisphereLight(
        0xe9fdff,
        0x285b39,
        3
    );

scene.add(skyLight);


const sun =
    new THREE.DirectionalLight(
        0xfff5d6,
        4.5
    );

sun.position.set(
    -120,
    180,
    80
);

sun.castShadow=true;

sun.shadow.mapSize.width=1536;
sun.shadow.mapSize.height=1536;

sun.shadow.camera.left=-240;
sun.shadow.camera.right=240;
sun.shadow.camera.top=240;
sun.shadow.camera.bottom=-240;

sun.shadow.camera.near=1;
sun.shadow.camera.far=520;

sun.shadow.bias=-.00025;

scene.add(sun);


/* =========================================================
   SKY GRADIENT DOME
========================================================= */

const skyCanvas =
    document.createElement("canvas");

skyCanvas.width=512;
skyCanvas.height=512;

const skyCtx =
    skyCanvas.getContext("2d");

const skyGradient =
    skyCtx.createLinearGradient(
        0,
        0,
        0,
        512
    );

skyGradient.addColorStop(
    0,
    "#238fe2"
);

skyGradient.addColorStop(
    .32,
    "#55c8f3"
);

skyGradient.addColorStop(
    .68,
    "#9be9fa"
);

skyGradient.addColorStop(
    1,
    "#d8f8ff"
);

skyCtx.fillStyle=skyGradient;
skyCtx.fillRect(
    0,
    0,
    512,
    512
);

const skyTexture =
    new THREE.CanvasTexture(
        skyCanvas
    );

skyTexture.colorSpace=
    THREE.SRGBColorSpace;

const sky =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            480,
            48,
            32
        ),

        new THREE.MeshBasicMaterial({
            map:skyTexture,
            side:THREE.BackSide
        })
    );

scene.add(sky);


/* =========================================================
   CLOUD SYSTEM
========================================================= */

const clouds=[];

function createCloud(
    x,
    y,
    z,
    scale
){

    const group =
        new THREE.Group();

    const material =
        new THREE.MeshPhysicalMaterial({
            color:0xffffff,
            roughness:.9,
            metalness:0,
            transparent:true,
            opacity:.9
        });

    const pieces=[
        [-2,0,0,1.0],
        [-.7,.35,.1,1.25],
        [.8,.45,0,1.45],
        [2,.05,0,1.0],
        [.1,.7,-.1,.95]
    ];

    for(
        const p of pieces
    ){

        const puff =
            new THREE.Mesh(

                new THREE.SphereGeometry(
                    3,
                    20,
                    14
                ),

                material
            );

        puff.position.set(
            p[0],
            p[1],
            p[2]
        );

        puff.scale.set(
            p[3]*1.35,
            p[3]*.62,
            p[3]
        );

        group.add(puff);
    }

    group.position.set(
        x,
        y,
        z
    );

    group.scale.setScalar(
        scale
    );

    scene.add(group);

    clouds.push({
        object:group,
        speed:.7+
            Math.random()*1.1,
        baseY:y,
        phase:Math.random()*10
    });
}


/* distant clouds */

createCloud(
    -120,
    95,
    -170,
    2.8
);

createCloud(
    30,
    120,
    -230,
    3.4
);

createCloud(
    150,
    92,
    -120,
    2.5
);

createCloud(
    -180,
    110,
    -20,
    3.1
);

createCloud(
    180,
    125,
    80,
    3.2
);

createCloud(
    -40,
    105,
    180,
    2.6
);

createCloud(
    90,
    100,
    210,
    2.9
);

createCloud(
    -200,
    135,
    160,
    3.5
);


/* =========================================================
   WATER
========================================================= */

const waterGeo =
    new THREE.PlaneGeometry(
        800,
        800,
        70,
        70
    );

const waterMat =
    new THREE.MeshPhysicalMaterial({
        color:0x19bddd,
        roughness:.1,
        metalness:.08,
        transmission:.08,
        transparent:true,
        opacity:.84,
        clearcoat:1,
        clearcoatRoughness:.06
    });

const water =
    new THREE.Mesh(
        waterGeo,
        waterMat
    );

water.rotation.x=-Math.PI/2;
water.position.y=-4;

scene.add(water);


/* =========================================================
   ISLAND
========================================================= */

const island =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            390,
            4,
            390
        ),

        new THREE.MeshStandardMaterial({
            color:0x419b48,
            roughness:1
        })
    );

island.position.y=-1.7;

island.receiveShadow=true;

scene.add(island);


/* grass */

const grass =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            370,
            .42,
            370
        ),

        new THREE.MeshStandardMaterial({
            color:0x36ad43,
            roughness:.96
        })
    );

grass.position.y=.15;

grass.receiveShadow=true;

scene.add(grass);


/* beach */

const beach =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            154,
            154,
            .55,
            128
        ),

        new THREE.MeshStandardMaterial({
            color:0xf1dfa0,
            roughness:.9
        })
    );

beach.position.y=.43;

beach.receiveShadow=true;

scene.add(beach);


/* inner grass */

const innerGrass =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            132,
            132,
            .58,
            128
        ),

        new THREE.MeshStandardMaterial({
            color:0x48c952,
            roughness:.87
        })
    );

innerGrass.position.y=.73;

innerGrass.receiveShadow=true;

scene.add(innerGrass);


/* =========================================================
   GRASS DETAIL
========================================================= */

const grassBladeGeo =
    new THREE.PlaneGeometry(
        .12,
        .7
    );

const grassBladeMat =
    new THREE.MeshStandardMaterial({
        color:0x239d39,
        side:THREE.DoubleSide,
        roughness:1
    });

const grassBlades =
    new THREE.InstancedMesh(
        grassBladeGeo,
        grassBladeMat,
        1700
    );

const temp =
    new THREE.Object3D();

for(let i=0;i<1700;i++){

    const a=
        Math.random()*Math.PI*2;

    const r=
        30+
        Math.random()*105;

    temp.position.set(
        Math.cos(a)*r,
        1,
        Math.sin(a)*r
    );

    temp.rotation.y=
        Math.random()*Math.PI;

    const s=
        .5+
        Math.random()*.8;

    temp.scale.set(
        s,
        s,
        s
    );

    temp.updateMatrix();

    grassBlades.setMatrixAt(
        i,
        temp.matrix
    );
}

grassBlades.instanceMatrix.needsUpdate=true;

scene.add(grassBlades);


/* =========================================================
   TREES
========================================================= */

const trunkGeo =
    new THREE.CylinderGeometry(
        .32,
        .62,
        5.8,
        10
    );

const trunkMat =
    new THREE.MeshStandardMaterial({
        color:0x704b2c,
        roughness:.92
    });

const leafGeo =
    new THREE.IcosahedronGeometry(
        3.2,
        2
    );

const leafMat =
    new THREE.MeshPhysicalMaterial({
        color:0x20aa4c,
        roughness:.65,
        clearcoat:.2
    });


for(let i=0;i<105;i++){

    const a=
        Math.random()*Math.PI*2;

    const r=
        38+
        Math.random()*87;

    const tree=
        new THREE.Group();

    const trunk=
        new THREE.Mesh(
            trunkGeo,
            trunkMat
        );

    trunk.position.y=3;
    trunk.castShadow=true;

    tree.add(trunk);


    const crown=
        new THREE.Mesh(
            leafGeo,
            leafMat
        );

    crown.position.y=7;
    crown.scale.set(
        1.05,
        1.18,
        1.05
    );

    crown.castShadow=true;

    tree.add(crown);


    tree.position.set(
        Math.cos(a)*r,
        .8,
        Math.sin(a)*r
    );

    tree.rotation.y=
        Math.random()*Math.PI;

    scene.add(tree);
}


/* =========================================================
   CHARACTER MATERIAL
========================================================= */

function characterGeometry(
    geometry
){

    const pos=
        geometry.attributes.position;

    const colors=[];

    const darkBlue=
        new THREE.Color(
            0x0875d0
        );

    const blue=
        new THREE.Color(
            0x10bde5
        );

    const cyan=
        new THREE.Color(
            0x79f3f2
        );

    const highlight=
        new THREE.Color(
            0xd9ffff
        );

    let min=-Infinity;
    let max=-Infinity;

    let minY=Infinity;
    let maxY=-Infinity;

    for(
        let i=0;
        i<pos.count;
        i++
    ){

        const y=pos.getY(i);

        minY=
            Math.min(
                minY,
                y
            );

        maxY=
            Math.max(
                maxY,
                y
            );
    }

    for(
        let i=0;
        i<pos.count;
        i++
    ){

        const y=pos.getY(i);

        const t=
            (y-minY)/
            Math.max(
                maxY-minY,
                .001
            );

        const c=
            new THREE.Color();

        if(t>.72){

            c.lerpColors(
                cyan,
                highlight,
                (t-.72)/.28
            );

        }else if(t>.35){

            c.lerpColors(
                blue,
                cyan,
                (t-.35)/.37
            );

        }else{

            c.lerpColors(
                darkBlue,
                blue,
                t/.35
            );
        }

        colors.push(
            c.r,
            c.g,
            c.b
        );
    }

    geometry.setAttribute(
        "color",
        new THREE.Float32BufferAttribute(
            colors,
            3
        )
    );

    return geometry;
}


const characterMat =
    new THREE.MeshPhysicalMaterial({

        vertexColors:true,

        roughness:.09,

        metalness:.015,

        clearcoat:1,

        clearcoatRoughness:.045,

        sheen:.2,

        sheenRoughness:.15
    });


/* =========================================================
   PLAYER
========================================================= */

const player=
    new THREE.Group();

scene.add(player);


/* head */

const head=
    new THREE.Mesh(

        characterGeometry(
            new THREE.SphereGeometry(
                1.12,
                56,
                40
            )
        ),

        characterMat
    );

head.position.y=4.42;

head.scale.set(
    1.03,
    1.03,
    .96
);

head.castShadow=true;

player.add(head);


/* body */

const body=
    new THREE.Mesh(

        characterGeometry(
            new THREE.SphereGeometry(
                1.3,
                56,
                40
            )
        ),

        characterMat
    );

body.position.y=2.56;

body.scale.set(
    1.17,
    1.28,
    .78
);

body.castShadow=true;

player.add(body);


/* arms */

const leftArm=
    new THREE.Mesh(

        characterGeometry(
            new THREE.CapsuleGeometry(
                .39,
                1.72,
                18,
                28
            )
        ),

        characterMat
    );

leftArm.position.set(
    -1.53,
    2.3,
    0
);

leftArm.rotation.z=-.20;

leftArm.castShadow=true;

player.add(leftArm);


const rightArm=
    new THREE.Mesh(

        characterGeometry(
            new THREE.CapsuleGeometry(
                .39,
                1.72,
                18,
                28
            )
        ),

        characterMat
    );

rightArm.position.set(
    1.53,
    2.3,
    0
);

rightArm.rotation.z=.20;

rightArm.castShadow=true;

player.add(rightArm);


/* legs */

const leftLeg=
    new THREE.Mesh(

        characterGeometry(
            new THREE.CapsuleGeometry(
                .45,
                1.58,
                18,
                28
            )
        ),

        characterMat
    );

leftLeg.position.set(
    -.58,
    .78,
    0
);

leftLeg.scale.set(
    .92,
    1.08,
    .92
);

leftLeg.castShadow=true;

player.add(leftLeg);


const rightLeg=
    new THREE.Mesh(

        characterGeometry(
            new THREE.CapsuleGeometry(
                .45,
                1.58,
                18,
                28
            )
        ),

        characterMat
    );

rightLeg.position.set(
    .58,
    .78,
    0
);

rightLeg.scale.set(
    .92,
    1.08,
    .92
);

rightLeg.castShadow=true;

player.add(rightLeg);


/* =========================================================
   DOLPHIN
========================================================= */

const dolphin=
    new THREE.Group();

const dolphinMat=
    new THREE.MeshPhysicalMaterial({
        color:0x58cae9,
        roughness:.13,
        metalness:.03,
        clearcoat:1,
        clearcoatRoughness:.06
    });

const dolphinBody=
    new THREE.Mesh(
        new THREE.SphereGeometry(
            2,
            36,
            24
        ),
        dolphinMat
    );

dolphinBody.scale.set(
    1.8,
    .62,
    .82
);

dolphinBody.castShadow=true;

dolphin.add(dolphinBody);


const snout=
    new THREE.Mesh(
        new THREE.CapsuleGeometry(
            .38,
            1.25,
            12,
            20
        ),
        dolphinMat
    );

snout.rotation.z=-Math.PI/2;

snout.position.x=3;

dolphin.add(snout);


const fin=
    new THREE.Mesh(
        new THREE.ConeGeometry(
            .7,
            1.7,
            4
        ),
        dolphinMat
    );

fin.rotation.z=Math.PI;
fin.position.y=1;

dolphin.add(fin);


const tail=
    new THREE.Mesh(
        new THREE.ConeGeometry(
            1.1,
            1.7,
            5
        ),
        dolphinMat
    );

tail.rotation.z=Math.PI/2;
tail.position.x=-3;

dolphin.add(tail);


dolphin.position.set(
    20,
    2,
    -20
);

dolphin.rotation.y=Math.PI/2;

scene.add(dolphin);


/* =========================================================
   INPUT
========================================================= */

const keys={};

addEventListener(
    "keydown",
    e=>{
        keys[e.key.toLowerCase()]=true;
    }
);

addEventListener(
    "keyup",
    e=>{
        keys[e.key.toLowerCase()]=false;
    }
);


/* =========================================================
   RUN
========================================================= */

let running=false;

const runButton=
    document.getElementById("run");

runButton.addEventListener(
    "pointerdown",
    e=>{
        e.preventDefault();
        running=true;
        runButton.classList.add("active");
    }
);

function stopRun(){
    running=false;
    runButton.classList.remove("active");
}

runButton.addEventListener(
    "pointerup",
    stopRun
);

runButton.addEventListener(
    "pointercancel",
    stopRun
);


/* =========================================================
   JOYSTICK
========================================================= */

const joystick=
    document.getElementById(
        "joystick"
    );

const stick=
    document.getElementById(
        "stick"
    );

let joyX=0;
let joyY=0;
let joyActive=false;

function updateJoy(e){

    const r=
        joystick.getBoundingClientRect();

    let x=
        e.clientX-
        (r.left+r.width/2);

    let y=
        e.clientY-
        (r.top+r.height/2);

    const max=43;

    const d=
        Math.sqrt(
            x*x+y*y
        );

    if(d>max){

        x=x/d*max;
        y=y/d*max;
    }

    joyX=x/max;
    joyY=y/max;

    stick.style.transform=
        `translate(${x}px,${y}px)`;
}

joystick.addEventListener(
    "pointerdown",
    e=>{
        joyActive=true;
        joystick.setPointerCapture(
            e.pointerId
        );
        updateJoy(e);
    }
);

joystick.addEventListener(
    "pointermove",
    e=>{
        if(joyActive)
            updateJoy(e);
    }
);

function releaseJoy(){

    joyActive=false;
    joyX=0;
    joyY=0;

    stick.style.transform=
        "translate(0,0)";
}

joystick.addEventListener(
    "pointerup",
    releaseJoy
);

joystick.addEventListener(
    "pointercancel",
    releaseJoy
);


/* =========================================================
   JUMP
========================================================= */

let jumping=false;
let velocityY=0;

document.getElementById(
    "jump"
).addEventListener(
    "pointerdown",
    ()=>{
        if(!jumping && !riding){
            jumping=true;
            velocityY=9;
        }
    }
);


/* =========================================================
   SHIFT LOCK
========================================================= */

let shiftLock=false;
let yaw=0;

const shift=
    document.getElementById(
        "shift"
    );

shift.addEventListener(
    "pointerdown",
    ()=>{

        shiftLock=!shiftLock;

        shift.classList.toggle(
            "active",
            shiftLock
        );

        if(shiftLock)
            yaw=player.rotation.y;
    }
);


/* =========================================================
   CAMERA
========================================================= */

let dragging=false;
let previousX=0;

renderer.domElement.addEventListener(
    "pointerdown",
    e=>{

        if(shiftLock)
            return;

        dragging=true;
        previousX=e.clientX;
    }
);

renderer.domElement.addEventListener(
    "pointermove",
    e=>{

        if(
            !dragging ||
            shiftLock
        )
            return;

        const dx=
            e.clientX-previousX;

        previousX=e.clientX;

        yaw-=dx*.007;
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

document.getElementById(
    "ride"
).addEventListener(
    "pointerdown",
    ()=>{

        const distance=
            player.position.distanceTo(
                dolphin.position
            );

        if(distance<13){

            riding=!riding;

            player.visible=!riding;
        }
    }
);


/* =========================================================
   ANIMATION
========================================================= */

const clock=
    new THREE.Clock();

let animTime=0;

function animate(){

    requestAnimationFrame(
        animate
    );

    const dt=
        Math.min(
            clock.getDelta(),
            .033
        );


    /* -----------------------------------------------------
       CLOUD MOVEMENT
    ----------------------------------------------------- */

    const now=
        performance.now()*.001;

    for(
        const cloud of clouds
    ){

        cloud.object.position.x +=
            cloud.speed*dt;

        cloud.object.position.y=
            cloud.baseY+
            Math.sin(
                now*.08+
                cloud.phase
            )*.45;

        /*
           Loop clouds around the world
           so they never disappear.
        */

        if(
            cloud.object.position.x>300
        ){

            cloud.object.position.x=-300;
        }
    }


    /* -----------------------------------------------------
       PLAYER INPUT
    ----------------------------------------------------- */

    let inputX=0;
    let inputY=0;

    if(
        keys["a"] ||
        keys["arrowleft"]
    )
        inputX=-1;

    if(
        keys["d"] ||
        keys["arrowright"]
    )
        inputX=1;

    if(
        keys["w"] ||
        keys["arrowup"]
    )
        inputY=-1;

    if(
        keys["s"] ||
        keys["arrowdown"]
    )
        inputY=1;


    if(
        Math.abs(joyX)>.08 ||
        Math.abs(joyY)>.08
    ){

        inputX=joyX;
        inputY=joyY;
    }


    const length=
        Math.sqrt(
            inputX*inputX+
            inputY*inputY
        );

    if(length>1){

        inputX/=length;
        inputY/=length;
    }

    const moving=
        length>.08;


    /* -----------------------------------------------------
       CAMERA RELATIVE MOVEMENT

       UP = FORWARD
       DOWN = BACKWARD
    ----------------------------------------------------- */

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

    const movement=
        new THREE.Vector3();

    movement.addScaledVector(
        right,
        inputX
    );

    movement.addScaledVector(
        forward,
        -inputY
    );


    if(
        movement.lengthSq()>.001
    ){

        movement.normalize();

        let speed=
            running
            ?15
            :8.5;

        if(riding)
            speed=14;

        const object=
            riding
            ?dolphin
            :player;

        object.position.addScaledVector(
            movement,
            speed*dt
        );


        if(!riding){

            if(shiftLock){

                player.rotation.y=yaw;

            }else{

                player.rotation.y=
                    Math.atan2(
                        movement.x,
                        -movement.z
                    );
            }
        }
    }


    /* -----------------------------------------------------
       JUMP
    ----------------------------------------------------- */

    if(
        jumping &&
        !riding
    ){

        velocityY-=22*dt;

        player.position.y+=
            velocityY*dt;

        if(
            player.position.y<=.85
        ){

            player.position.y=.85;
            velocityY=0;
            jumping=false;
        }

    }else if(!riding){

        player.position.y=.85;
    }


    /* -----------------------------------------------------
       BOUNDARY
    ----------------------------------------------------- */

    function keepInside(object){

        const d=
            Math.sqrt(
                object.position.x**2+
                object.position.z**2
            );

        if(d>174){

            const scale=174/d;

            object.position.x*=scale;
            object.position.z*=scale;
        }
    }

    keepInside(player);
    keepInside(dolphin);


    /* -----------------------------------------------------
       RUN / WALK ANIMATION
    ----------------------------------------------------- */

    if(
        moving &&
        !riding
    ){

        animTime+=
            dt*
            (running?18:10);

        const swing=
            Math.sin(animTime)*
            (running?.76:.43);

        leftLeg.rotation.x=swing;
        rightLeg.rotation.x=-swing;

        leftArm.rotation.x=
            -swing*.82;

        rightArm.rotation.x=
            swing*.82;

        const bounce=
            Math.abs(
                Math.sin(
                    animTime*2
                )
            )*
            (running?.075:.035);

        body.position.y=
            2.56+bounce;

        head.position.y=
            4.42+bounce;

    }else{

        leftLeg.rotation.x=
            THREE.MathUtils.lerp(
                leftLeg.rotation.x,
                0,
                .15
            );

        rightLeg.rotation.x=
            THREE.MathUtils.lerp(
                rightLeg.rotation.x,
                0,
                .15
            );

        leftArm.rotation.x=
            THREE.MathUtils.lerp(
                leftArm.rotation.x,
                0,
                .15
            );

        rightArm.rotation.x=
            THREE.MathUtils.lerp(
                rightArm.rotation.x,
                0,
                .15
            );
    }


    /* -----------------------------------------------------
       DOLPHIN
    ----------------------------------------------------- */

    dolphin.position.y=
        2+
        Math.sin(
            now*1.5
        )*.3;

    dolphin.rotation.z=
        Math.sin(
            now*1.1
        )*.035;


    /* -----------------------------------------------------
       WATER WAVES
    ----------------------------------------------------- */

    const pos=
        waterGeo.attributes.position;

    for(
        let i=0;
        i<pos.count;
        i++
    ){

        const x=pos.getX(i);
        const z=pos.getY(i);

        pos.setZ(
            i,
            Math.sin(
                x*.035+
                now*.8
            )*.17+
            Math.cos(
                z*.041+
                now
            )*.14
        );
    }

    pos.needsUpdate=true;

    waterGeo.computeVertexNormals();


    /* -----------------------------------------------------
       CAMERA
    ----------------------------------------------------- */

    const target=
        riding
        ?dolphin.position.clone()
        :player.position.clone();

    target.y+=2.15;

    const distance=
        shiftLock
        ?8.8
        :10.5;

    const desired=
        target.clone();

    desired.x+=
        Math.sin(yaw)*
        distance;

    desired.z+=
        Math.cos(yaw)*
        distance;

    desired.y+=4.8;

    camera.position.lerp(
        desired,
        .105
    );

    camera.lookAt(target);


    /* -----------------------------------------------------
       RENDER
    ----------------------------------------------------- */

    renderer.render(
        scene,
        camera
    );
}


/* =========================================================
   START
========================================================= */

player.position.set(
    0,
    .85,
    0
);

camera.position.set(
    0,
    7,
    13
);

camera.lookAt(
    0,
    2,
    0
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
