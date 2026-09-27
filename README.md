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
    background:#51ddff;
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

/* TOP */

#title{
    position:absolute;
    left:20px;
    top:17px;
    color:white;
    font-size:23px;
    font-weight:900;
    text-shadow:
        0 2px 5px rgba(0,80,130,.7),
        0 0 15px rgba(255,255,255,.4);
}

#subtitle{
    position:absolute;
    left:21px;
    top:48px;
    color:white;
    font-size:12px;
    text-shadow:0 2px 5px rgba(0,80,130,.7);
}

/* JOYSTICK */

#joystick{
    position:absolute;
    left:22px;
    bottom:22px;
    width:132px;
    height:132px;
    border-radius:50%;

    background:rgba(255,255,255,.18);

    border:3px solid rgba(255,255,255,.75);

    box-shadow:
        0 9px 28px rgba(0,80,130,.25),
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

    background:
        linear-gradient(
            145deg,
            #ffffff,
            #b5f5ff
        );

    border:3px solid white;

    box-shadow:
        0 5px 18px rgba(0,90,130,.25),
        inset 0 0 10px white;
}

/* BUTTONS */

#buttons{
    position:absolute;

    right:20px;
    bottom:22px;

    display:flex;
    gap:11px;

    align-items:end;
}

button{
    width:76px;
    height:76px;

    border-radius:50%;

    border:3px solid white;

    color:white;

    font-size:10px;
    font-weight:900;

    background:
        linear-gradient(
            #9cf8ff,
            #17a9d8
        );

    box-shadow:
        0 7px 22px rgba(0,80,130,.3),
        inset 0 3px 9px rgba(255,255,255,.55);

    pointer-events:auto;

    touch-action:none;
}

button:active{
    transform:scale(.9);
}

#shiftButton.active{
    background:
        linear-gradient(
            #fff56c,
            #ffae25
        );

    color:#5b4300;
}

/* PORTRAIT */

#rotate{
    display:none;

    position:fixed;
    inset:0;

    z-index:100;

    background:
        linear-gradient(
            #54e2ff,
            #c9fbff
        );

    align-items:center;
    justify-content:center;

    flex-direction:column;

    text-align:center;

    color:#08799e;

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
        Explore the open world • Find the dolphin
    </div>

    <div id="joystick">
        <div id="stick"></div>
    </div>

    <div id="buttons">

        <button id="shiftButton">
            SHIFT<br>LOCK
        </button>

        <button id="jumpButton">
            JUMP
        </button>

        <button id="rideButton">
            RIDE
        </button>

    </div>

</div>

<div id="rotate">

    <div id="rotateIcon">
        ↔️
    </div>

    <div>
        TURN YOUR iPAD SIDEWAYS
    </div>

    <small>
        Landscape gives you the full world.
    </small>

</div>


<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>


<script>

/* =========================================================
   FRUTIGER AERO WORLD
   ROBLOX-STYLE THIRD PERSON CAMERA
   ========================================================= */


/* =========================================================
   RENDERER
   ========================================================= */

const canvas =
document.getElementById("game");

const renderer =
new THREE.WebGLRenderer({

    canvas,

    antialias:true,

    powerPreference:
        "high-performance"
});

renderer.setPixelRatio(
    Math.min(
        window.devicePixelRatio,
        2
    )
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
new THREE.Color(
    0x61ddff
);

scene.fog =
new THREE.Fog(
    0x70e3ff,
    140,
    650
);


/* =========================================================
   CAMERA
   ========================================================= */

const camera =
new THREE.PerspectiveCamera(
    67,
    innerWidth/innerHeight,
    .1,
    1000
);

camera.position.set(
    0,
    7,
    18
);


/* =========================================================
   LIGHTING
   ========================================================= */

const hemisphere =
new THREE.HemisphereLight(
    0xeaffff,
    0x328a50,
    2.7
);

scene.add(hemisphere);


const sun =
new THREE.DirectionalLight(
    0xffffff,
    4
);

sun.position.set(
    -120,
    170,
    100
);

sun.castShadow=true;

sun.shadow.mapSize.width=4096;
sun.shadow.mapSize.height=4096;

sun.shadow.camera.left=-220;
sun.shadow.camera.right=220;
sun.shadow.camera.top=220;
sun.shadow.camera.bottom=-220;

sun.shadow.bias=-.00025;

scene.add(sun);


/* =========================================================
   MATERIALS
   ========================================================= */

const blueBody =
new THREE.MeshPhysicalMaterial({

    color:0x1a9fe3,

    roughness:.14,

    metalness:.03,

    clearcoat:1,

    clearcoatRoughness:.04
});


const cyanBody =
new THREE.MeshPhysicalMaterial({

    color:0x45dfff,

    roughness:.08,

    metalness:.02,

    transparent:true,

    opacity:.94,

    clearcoat:1,

    clearcoatRoughness:.025
});


const whiteGloss =
new THREE.MeshBasicMaterial({

    color:0xffffff,

    transparent:true,

    opacity:.72
});


const greenGrass =
new THREE.MeshPhysicalMaterial({

    color:0x45d36b,

    roughness:.78,

    clearcoat:.25
});


const greenDark =
new THREE.MeshPhysicalMaterial({

    color:0x15994f,

    roughness:.7
});


const dirt =
new THREE.MeshStandardMaterial({

    color:0x58a65e,

    roughness:.95
});


const trunkMaterial =
new THREE.MeshStandardMaterial({

    color:0xc98a4b,

    roughness:.9
});


/* =========================================================
   WATER
   ========================================================= */

const waterGeometry =
new THREE.PlaneGeometry(
    1400,
    1400,
    180,
    180
);

const waterMaterial =
new THREE.MeshPhysicalMaterial({

    color:0x10bee9,

    roughness:.12,

    metalness:.08,

    transparent:true,

    opacity:.94,

    clearcoat:1,

    clearcoatRoughness:.05
});

const water =
new THREE.Mesh(
    waterGeometry,
    waterMaterial
);

water.rotation.x=
-Math.PI/2;

water.position.y=-1.55;

water.receiveShadow=true;

scene.add(water);


/* =========================================================
   OPEN WORLD ISLANDS
   ========================================================= */

const worldIslands=[];


function makeIsland(
    x,
    z,
    radius,
    height
){

    const island =
    new THREE.Group();


    /* ROCK */

    const rock =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            radius*.88,
            radius*1.18,
            height,
            64
        ),

        dirt
    );

    rock.position.y=
    -height/2;

    rock.castShadow=true;

    rock.receiveShadow=true;

    island.add(rock);


    /* GRASS TOP */

    const grass =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            radius,
            radius*1.05,
            .8,
            64
        ),

        greenGrass
    );

    grass.position.y=-.4;

    grass.castShadow=true;

    grass.receiveShadow=true;

    island.add(grass);


    island.position.set(
        x,
        0,
        z
    );

    scene.add(island);


    worldIslands.push({

        x:x,

        z:z,

        radius:radius
    });
}


/* MAIN CONTINENT */

makeIsland(
    0,
    0,
    55,
    9
);


/* DISTANT ISLANDS */

makeIsland(
    -115,
    -75,
    35,
    8
);

makeIsland(
    120,
    -105,
    42,
    9
);

makeIsland(
    150,
    85,
    32,
    8
);

makeIsland(
    -150,
    110,
    40,
    8
);

makeIsland(
    280,
    10,
    48,
    9
);

makeIsland(
    -300,
    -30,
    50,
    9
);

makeIsland(
    340,
    -180,
    38,
    8
);

makeIsland(
    -350,
    200,
    45,
    9
);


/* =========================================================
   AERO CHARACTER
   BODY DESIGN BASED ON YOUR IMAGE
   ========================================================= */

function makeAeroGuy(){

    const guy =
    new THREE.Group();


    /*
      TORSO

      Rounded pear-like body
      instead of a capsule.
    */

    const torso =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            1,
            64,
            48
        ),

        cyanBody
    );

    torso.scale.set(
        1.25,
        1.45,
        .92
    );

    torso.position.y=
    1.72;

    torso.castShadow=true;

    torso.receiveShadow=true;

    guy.add(torso);


    /*
      DARKER INNER BODY
    */

    const inner =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            .98,
            48,
            36
        ),

        blueBody
    );

    inner.scale.set(
        1.20,
        1.36,
        .89
    );

    inner.position.y=
    1.69;

    inner.material.transparent=true;

    inner.material.opacity=.38;

    guy.add(inner);


    /*
      BIG ROUND HEAD
    */

    const head =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            1.03,
            64,
            48
        ),

        cyanBody
    );

    head.position.y=
    3.68;

    head.castShadow=true;

    head.receiveShadow=true;

    guy.add(head);


    /*
      HEAD GLASS INNER COLOR
    */

    const headInner =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            .98,
            48,
            36
        ),

        blueBody
    );

    headInner.position.y=
    3.67;

    headInner.material.transparent=true;

    headInner.material.opacity=.32;

    guy.add(headInner);


    /*
      GLOSS ON HEAD
    */

    const headGloss =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            .34,
            32,
            24
        ),

        whiteGloss
    );

    headGloss.scale.set(
        1.45,
        .5,
        .4
    );

    headGloss.position.set(
        -.32,
        4.10,
        -.82
    );

    guy.add(headGloss);


    /*
      BODY HIGHLIGHT
    */

    const bodyHighlight =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            .42,
            32,
            24
        ),

        whiteGloss
    );

    bodyHighlight.scale.set(
        1.4,
        .65,
        .3
    );

    bodyHighlight.position.set(
        -.6,
        2.25,
        -.78
    );

    bodyHighlight.material.opacity=.38;

    guy.add(bodyHighlight);


    /*
      LEFT ARM
    */

    const leftArm =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            1,
            48,
            36
        ),

        cyanBody
    );

    leftArm.scale.set(
        .40,
        .98,
        .47
    );

    leftArm.position.set(
        -1.28,
        1.48,
        0
    );

    leftArm.rotation.z=.18;

    leftArm.castShadow=true;

    guy.add(leftArm);


    /*
      RIGHT ARM
    */

    const rightArm =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            1,
            48,
            36
        ),

        cyanBody
    );

    rightArm.scale.set(
        .40,
        .98,
        .47
    );

    rightArm.position.set(
        1.28,
        1.48,
        0
    );

    rightArm.rotation.z=-.18;

    rightArm.castShadow=true;

    guy.add(rightArm);


    /*
      LEGS

      Separate rounded legs like the image.
    */

    const leftLeg =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            1,
            48,
            36
        ),

        cyanBody
    );

    leftLeg.scale.set(
        .50,
        .82,
        .56
    );

    leftLeg.position.set(
        -.54,
        .18,
        0
    );

    leftLeg.castShadow=true;

    guy.add(leftLeg);


    const rightLeg =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            1,
            48,
            36
        ),

        cyanBody
    );

    rightLeg.scale.set(
        .50,
        .82,
        .56
    );

    rightLeg.position.set(
        .54,
        .18,
        0
    );

    rightLeg.castShadow=true;

    guy.add(rightLeg);


    return guy;
}


/* =========================================================
   PLAYER
   ========================================================= */

const player =
makeAeroGuy();

/*
   Bottom of legs is around -0.64.
   Raise character so feet sit directly
   above the world.
*/

player.position.set(
    0,
    .65,
    10
);

scene.add(player);


/* =========================================================
   NPC GUYS
   ========================================================= */

const npcs=[];


const npcLocations=[

    [-12,5],

    [14,-5],

    [-27,-18],

    [28,15],

    [-8,30],

    [35,-30],

    [-40,10],

    [5,-38]

];


npcLocations.forEach(
    position=>{

        const npc =
        makeAeroGuy();

        npc.scale.setScalar(.78);

        npc.position.set(
            position[0],
            .65,
            position[1]
        );

        scene.add(npc);

        npcs.push(npc);
    }
);


/* =========================================================
   PALM TREES
   ========================================================= */

function makePalm(
    x,
    z,
    scale
){

    const tree =
    new THREE.Group();


    const height=
    7*scale;


    const trunk =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            .25*scale,
            .45*scale,
            height,
            20
        ),

        trunkMaterial
    );

    /*
       IMPORTANT:
       Tree starts at y=0.
       It no longer sinks deep into ground.
    */

    trunk.position.y=
    height/2;

    trunk.castShadow=true;

    tree.add(trunk);


    const crown =
    new THREE.Group();

    crown.position.y=
    height;

    tree.add(crown);


    for(let i=0;i<9;i++){

        const angle=
        i*Math.PI*2/9;

        const leaf =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                1,
                24,
                16
            ),

            greenDark
        );

        leaf.scale.set(
            2.8*scale,
            .14*scale,
            .48*scale
        );

        leaf.position.set(
            Math.cos(angle)*1.5*scale,
            0,
            Math.sin(angle)*1.5*scale
        );

        leaf.rotation.y=
        -angle;

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


/* TREES ON MAIN ISLAND */

const treeSpots=[

    [-30,10,.9],
    [-22,-20,.8],
    [-10,35,.8],
    [12,30,.9],
    [27,16,.75],
    [35,-5,.8],
    [28,-25,.9],
    [5,-34,.8],
    [-20,-35,.75],
    [-38,-15,.75],
    [40,35,.7],
    [-40,30,.8]
];


treeSpots.forEach(
    p=>{
        makePalm(
            p[0],
            p[1],
            p[2]
        );
    }
);


/* TREES ON DISTANT ISLANDS */

for(let islandIndex=0;
    islandIndex<worldIslands.length;
    islandIndex++){

    const island=
    worldIslands[islandIndex];

    if(islandIndex===0)
        continue;

    for(let i=0;i<9;i++){

        const angle=
        Math.random()*Math.PI*2;

        const radius=
        Math.random()*
        island.radius*.7;

        makePalm(

            island.x+
            Math.cos(angle)*radius,

            island.z+
            Math.sin(angle)*radius,

            .65+
            Math.random()*.3
        );
    }
}


/* =========================================================
   CLOUDS
   ========================================================= */

function makeCloud(
    x,
    y,
    z,
    size
){

    const cloud =
    new THREE.Group();


    for(let i=0;i<10;i++){

        const puff =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                1,
                32,
                24
            ),

            new THREE.MeshPhysicalMaterial({

                color:0xffffff,

                roughness:.32,

                clearcoat:.7,

                clearcoatRoughness:.12
            })
        );

        puff.scale.setScalar(
            (.8+
            Math.random()*.9)*
            size
        );

        puff.position.set(

            (i-4.5)*
            1.5*
            size,

            Math.random()*
            1.2*
            size,

            Math.random()*
            .8*
            size
        );

        cloud.add(puff);
    }


    cloud.position.set(
        x,
        y,
        z
    );

    scene.add(cloud);
}


makeCloud(
    -70,
    48,
    -130,
    2.3
);

makeCloud(
    80,
    52,
    -190,
    3
);

makeCloud(
    180,
    40,
    40,
    2
);

makeCloud(
    -180,
    45,
    120,
    2.5
);


/* =========================================================
   DOLPHIN
   ========================================================= */

function makeDolphin(){

    const dolphin =
    new THREE.Group();


    const material =
    new THREE.MeshPhysicalMaterial({

        color:0x42c8ea,

        roughness:.13,

        metalness:.08,

        clearcoat:1
    });


    const body =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            1,
            48,
            32
        ),

        material
    );

    body.scale.set(
        2.4,
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

        material
    );

    nose.rotation.z=
    -Math.PI/2;

    nose.position.z=
    -2.25;

    dolphin.add(nose);


    const dorsal =
    new THREE.Mesh(

        new THREE.ConeGeometry(
            .45,
            1.5,
            20
        ),

        material
    );

    dorsal.rotation.z=
    Math.PI;

    dorsal.position.y=.85;

    dolphin.add(dorsal);


    const tail =
    new THREE.Group();


    for(
        const side of [-1,1]
    ){

        const fin =
        new THREE.Mesh(

            new THREE.ConeGeometry(
                .5,
                1.5,
                20
            ),

            material
        );

        fin.rotation.z=
        side*.75;

        fin.position.x=
        side*.45;

        tail.add(fin);
    }


    tail.position.z=2.1;

    dolphin.add(tail);

    return dolphin;
}


const dolphin =
makeDolphin();

dolphin.position.set(
    0,
    -.2,
    -45
);

scene.add(dolphin);


/* =========================================================
   BUBBLES
   ========================================================= */

const bubbles=[];


for(let i=0;i<80;i++){

    const bubble =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            .2+
            Math.random()*.4,

            24,
            20
        ),

        new THREE.MeshPhysicalMaterial({

            color:0xeaffff,

            transparent:true,

            opacity:.42,

            roughness:0,

            transmission:.3,

            clearcoat:1
        })
    );


    bubble.position.set(

        (Math.random()-.5)*600,

        Math.random()*16,

        (Math.random()-.5)*600
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
document.getElementById(
    "joystick"
);

const stick=
document.getElementById(
    "stick"
);


joystick.addEventListener(
    "pointerdown",
    e=>{

        joystickPointer=
            e.pointerId;

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
            e.pointerId !==
            joystickPointer
        )
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

    const distance=
    Math.hypot(x,y);

    if(distance>max){

        x=
        x/distance*max;

        y=
        y/distance*max;
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
   ROBLOX STYLE CAMERA
   ========================================================= */

let cameraYaw=0;

let cameraPitch=.18;

let cameraDistance=13;

let cameraDragging=false;

let previousX=0;

let previousY=0;


/*
   Drag the world with your finger
   to orbit the camera.
*/

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
            e.clientX-
            previousX;

        const dy=
            e.clientY-
            previousY;


        cameraYaw -=
            dx*.006;


        cameraPitch -=
            dy*.004;


        cameraPitch=
            Math.max(
                -.35,
                Math.min(
                    .75,
                    cameraPitch
                )
            );


        previousX=
            e.clientX;

        previousY=
            e.clientY;
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
   SHIFT LOCK
   ========================================================= */

let shiftLock=false;


const shiftButton=
document.getElementById(
    "shiftButton"
);


shiftButton.addEventListener(
    "pointerdown",
    ()=>{

        shiftLock=
            !shiftLock;

        shiftButton.classList.toggle(
            "active",
            shiftLock
        );
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

    verticalVelocity=11;
}


document
.getElementById(
    "jumpButton"
)
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

        if(distance<11){

            riding=true;

            player.position.copy(
                dolphin.position
            );

            player.position.y+=2.1;
        }

    }else{

        riding=false;

        player.position.y=.65;
    }
}


document
.getElementById(
    "rideButton"
)
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

        keys[
            e.key.toLowerCase()
        ]=true;

        if(e.code==="Space")
            jump();

        if(
            e.key.toLowerCase()==="e"
        )
            toggleRide();

        if(
            e.key.toLowerCase()==="shift"
        ){

            shiftLock=
                !shiftLock;

            shiftButton.classList.toggle(
                "active",
                shiftLock
            );
        }
    }
);


window.addEventListener(
    "keyup",
    e=>{

        keys[
            e.key.toLowerCase()
        ]=false;
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


    /* ---------------------------------------
       KEYBOARD
       --------------------------------------- */

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


    /* ---------------------------------------
       MOVEMENT RELATIVE TO CAMERA
       --------------------------------------- */

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


    if(
        movement.lengthSq()>0
    ){

        movement.normalize();


        const speed=
            riding
            ? 20
            : 8;


        if(riding){

            dolphin.position
                .addScaledVector(
                    movement,
                    speed*
                    dt*
                    strength
                );

        }else{

            player.position
                .addScaledVector(
                    movement,
                    speed*
                    dt*
                    strength
                );
        }


        /*
           Normal camera mode:
           character faces movement.

           Shift lock:
           character faces camera direction.
        */

        if(shiftLock){

            player.rotation.y=
                cameraYaw;

        }else{

            const targetRotation=
                Math.atan2(
                    movement.x,
                    movement.z
                );

            player.rotation.y=
                THREE.MathUtils.lerp(
                    player.rotation.y,
                    targetRotation,
                    .15
                );
        }
    }


    /* ---------------------------------------
       SHIFT LOCK CAMERA
       --------------------------------------- */

    if(shiftLock){

        /*
          Camera stays behind character.
        */

        const targetYaw=
            player.rotation.y;

        cameraYaw=
            THREE.MathUtils.lerp(
                cameraYaw,
                targetYaw,
                .04
            );
    }


    /* ---------------------------------------
       JUMP
       --------------------------------------- */

    if(jumping){

        verticalVelocity -=
            25*dt;


        if(riding){

            dolphin.position.y +=
                verticalVelocity*dt;


            if(
                dolphin.position.y<=-.2
            ){

                dolphin.position.y=-.2;

                verticalVelocity=0;

                jumping=false;
            }

        }else{

            player.position.y +=
                verticalVelocity*dt;


            if(
                player.position.y<=.65
            ){

                player.position.y=.65;

                verticalVelocity=0;

                jumping=false;
            }
        }
    }


    /* ---------------------------------------
       DOLPHIN
       --------------------------------------- */

    dolphin.rotation.z=
        Math.sin(time*2.5)*.035;


    dolphin.position.y +=
        Math.sin(time*3)*.008;


    if(riding){

        player.position.copy(
            dolphin.position
        );

        player.position.y+=2.1;

        player.rotation.y=
            cameraYaw;
    }


    /* ---------------------------------------
       NPC IDLE
       --------------------------------------- */

    npcs.forEach(
        (npc,index)=>{

            npc.position.y=
                .65+
                Math.sin(
                    time*1.4+
                    index
                )*.045;
        }
    );


    /* ---------------------------------------
       BUBBLES
       --------------------------------------- */

    bubbles.forEach(
        (bubble,index)=>{

            bubble.position.y +=
                dt*
                (.25+
                (index%4)*.12);

            bubble.rotation.y +=
                dt*.5;


            if(
                bubble.position.y>17
            ){

                bubble.position.y=-1;
            }
        }
    );


    /* ---------------------------------------
       ROBLOX-STYLE CAMERA FOLLOW
       --------------------------------------- */

    const target=
        riding
        ? dolphin
        : player;


    /*
       Smooth zoom distance.
    */

    const targetDistance=
        riding
        ? 17
        : 13;


    cameraDistance=
        THREE.MathUtils.lerp(
            cameraDistance,
            targetDistance,
            .08
        );


    /*
       Spherical camera position.
    */

    const horizontal=
        Math.cos(cameraPitch)*
        cameraDistance;


    const desiredX=
        target.position.x-
        Math.sin(cameraYaw)*
        horizontal;


    const desiredZ=
        target.position.z+
        Math.cos(cameraYaw)*
        horizontal;


    const desiredY=
        target.position.y+
        3+
        Math.sin(cameraPitch)*
        cameraDistance;


    /*
       Smooth Roblox-like follow.
    */

    camera.position.x=
        THREE.MathUtils.lerp(
            camera.position.x,
            desiredX,
            .12
        );


    camera.position.y=
        THREE.MathUtils.lerp(
            camera.position.y,
            desiredY,
            .12
        );


    camera.position.z=
        THREE.MathUtils.lerp(
            camera.position.z,
            desiredZ,
            .12
        );


    camera.lookAt(
        target.position.x,
        target.position.y+1.6,
        target.position.z
    );


    /* ---------------------------------------
       WATER
       --------------------------------------- */

    water.position.y=
        -1.55+
        Math.sin(time)*.018;


    /* ---------------------------------------
       RENDER
       --------------------------------------- */

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
