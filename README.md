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
    -webkit-tap-highlight-color:transparent;
}

html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#58d9f7;
    touch-action:none;
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
    outline:0;
    color:#fff;
    font-weight:900;
    pointer-events:auto;
    touch-action:none;
    user-select:none;
    -webkit-user-select:none;

    text-shadow:
        0 1px 3px rgba(0,60,100,.65);

    box-shadow:
        0 8px 20px rgba(0,60,100,.25),
        inset 0 2px 6px rgba(255,255,255,.65),
        inset 0 -6px 12px rgba(0,80,130,.15);

    backdrop-filter:blur(8px);
}

#title{
    position:absolute;
    top:13px;
    left:50%;
    transform:translateX(-50%);

    font-size:17px;
    letter-spacing:.7px;
    white-space:nowrap;

    color:white;

    text-shadow:
        0 2px 5px rgba(0,70,120,.8),
        0 0 14px rgba(255,255,255,.8);
}

#shift{
    position:absolute;
    right:12px;
    top:12px;

    padding:12px 14px;

    border-radius:17px;

    background:rgba(0,112,182,.72);
}

#shift.active{
    background:rgba(0,190,110,.92);
}

#run{
    position:absolute;
    right:22px;
    bottom:145px;

    width:76px;
    height:76px;

    border-radius:50%;

    background:rgba(10,130,220,.88);
}

#run.active{
    background:rgba(0,195,115,.96);
    transform:scale(1.08);
}

#jump{
    position:absolute;
    right:22px;
    bottom:55px;

    width:76px;
    height:76px;

    border-radius:50%;

    background:rgba(0,174,120,.9);
}

#ride{
    position:absolute;
    right:112px;
    bottom:67px;

    width:66px;
    height:66px;

    border-radius:50%;

    background:rgba(0,126,207,.9);
}

#joystick{
    position:absolute;

    left:22px;
    bottom:35px;

    width:132px;
    height:132px;

    border-radius:50%;

    background:
        radial-gradient(
            circle,
            rgba(255,255,255,.3),
            rgba(255,255,255,.07)
        );

    border:
        2px solid rgba(255,255,255,.5);

    box-shadow:
        inset 0 0 30px rgba(255,255,255,.25),
        0 9px 27px rgba(0,60,100,.23);

    pointer-events:auto;
}

#stick{
    position:absolute;

    width:59px;
    height:59px;

    left:50%;
    top:50%;

    margin-left:-29.5px;
    margin-top:-29.5px;

    border-radius:50%;

    background:
        radial-gradient(
            circle at 30% 20%,
            #fff,
            #e3fbff 38%,
            #9adff3
        );

    box-shadow:
        0 7px 17px rgba(0,55,95,.25),
        inset 0 2px 5px #fff;
}
</style>
</head>

<body>

<div id="ui">

    <div id="title">
        FRUTIGER AERO WORLD
    </div>

    <button id="shift">
        SHIFT LOCK
    </button>

    <button id="run">
        RUN
    </button>

    <button id="ride">
        RIDE
    </button>

    <button id="jump">
        JUMP
    </button>

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
    new THREE.Color(0x61dafa);

scene.fog =
    new THREE.FogExp2(
        0x9beafa,
        0.00145
    );


/* =========================================================
   CAMERA
========================================================= */

const camera =
    new THREE.PerspectiveCamera(
        64,
        innerWidth / innerHeight,
        0.1,
        1000
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
        window.devicePixelRatio || 1,
        1.35
    )
);

renderer.outputColorSpace =
    THREE.SRGBColorSpace;

renderer.toneMapping =
    THREE.ACESFilmicToneMapping;

renderer.toneMappingExposure =
    1.2;

renderer.shadowMap.enabled = true;

renderer.shadowMap.type =
    THREE.PCFSoftShadowMap;

document.body.appendChild(
    renderer.domElement
);


/* =========================================================
   LIGHTING
========================================================= */

const hemisphere =
    new THREE.HemisphereLight(
        0xeaffff,
        0x275d38,
        3.4
    );

scene.add(hemisphere);


const sun =
    new THREE.DirectionalLight(
        0xfff3d1,
        4.8
    );

sun.position.set(
    -120,
    180,
    90
);

sun.castShadow = true;

sun.shadow.mapSize.width = 1536;
sun.shadow.mapSize.height = 1536;

sun.shadow.camera.left = -230;
sun.shadow.camera.right = 230;
sun.shadow.camera.top = 230;
sun.shadow.camera.bottom = -230;

sun.shadow.camera.near = 1;
sun.shadow.camera.far = 550;

sun.shadow.bias = -0.00025;

scene.add(sun);


/* =========================================================
   SKY
========================================================= */

const skyCanvas =
    document.createElement("canvas");

skyCanvas.width = 1024;
skyCanvas.height = 512;

const skyCtx =
    skyCanvas.getContext("2d");

const gradient =
    skyCtx.createLinearGradient(
        0,
        0,
        0,
        512
    );

gradient.addColorStop(
    0,
    "#167fd5"
);

gradient.addColorStop(
    .22,
    "#2ca9e8"
);

gradient.addColorStop(
    .48,
    "#63d5f4"
);

gradient.addColorStop(
    .72,
    "#a7ecfa"
);

gradient.addColorStop(
    1,
    "#e6fcff"
);

skyCtx.fillStyle = gradient;

skyCtx.fillRect(
    0,
    0,
    1024,
    512
);


/* atmospheric sunlight */

const skyGlow =
    skyCtx.createRadialGradient(
        735,
        85,
        15,
        735,
        85,
        270
    );

skyGlow.addColorStop(
    0,
    "rgba(255,255,225,.85)"
);

skyGlow.addColorStop(
    .35,
    "rgba(255,255,225,.3)"
);

skyGlow.addColorStop(
    1,
    "rgba(255,255,255,0)"
);

skyCtx.fillStyle =
    skyGlow;

skyCtx.fillRect(
    430,
    0,
    594,
    350
);

const skyTexture =
    new THREE.CanvasTexture(
        skyCanvas
    );

skyTexture.colorSpace =
    THREE.SRGBColorSpace;

const sky =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            500,
            64,
            40
        ),

        new THREE.MeshBasicMaterial({
            map:skyTexture,
            side:THREE.BackSide
        })
    );

scene.add(sky);


/* =========================================================
   MOVING CLOUDS
========================================================= */

const clouds = [];

function makeCloud(
    x,
    y,
    z,
    scale,
    speed
){

    const group =
        new THREE.Group();

    const cloudMaterial =
        new THREE.MeshPhysicalMaterial({

            color:0xffffff,

            roughness:.97,

            transparent:true,

            opacity:.9
        });


    const puffs = [
        [-3.0,0,0,1.0],
        [-1.5,.4,.1,1.35],
        [0,.7,0,1.55],
        [1.6,.42,.05,1.25],
        [3,0,0,1.0],
        [.2,1.05,-.15,.85]
    ];


    for(
        const p of puffs
    ){

        const puff =
            new THREE.Mesh(

                new THREE.SphereGeometry(
                    3,
                    24,
                    18
                ),

                cloudMaterial
            );

        puff.position.set(
            p[0],
            p[1],
            p[2]
        );

        puff.scale.set(
            p[3] * 1.4,
            p[3] * .63,
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
        speed:speed,
        baseY:y,
        phase:Math.random()*10
    });
}


makeCloud(-190,105,-230,3.1,.75);
makeCloud(-55,125,-290,3.8,.55);
makeCloud(105,100,-245,2.8,.8);
makeCloud(225,115,-100,3.3,.68);
makeCloud(-245,125,20,3.5,.7);
makeCloud(-120,110,220,2.8,.6);
makeCloud(85,125,250,3.2,.72);
makeCloud(225,108,180,2.8,.82);


/* =========================================================
   DISTANT AERO MOUNTAINS
========================================================= */

const mountainMaterial =
    new THREE.MeshPhysicalMaterial({

        color:0x5cc5a7,

        roughness:.86,

        clearcoat:.1
    });


for(
    let i=0;
    i<20;
    i++
){

    const angle =
        Math.random() *
        Math.PI *
        2;

    const radius =
        225+
        Math.random()*30;

    const mountain =
        new THREE.Mesh(

            new THREE.ConeGeometry(
                18+
                Math.random()*20,

                40+
                Math.random()*45,

                7
            ),

            mountainMaterial
        );

    mountain.position.set(
        Math.cos(angle)*radius,
        14,
        Math.sin(angle)*radius
    );

    mountain.rotation.y =
        Math.random()*Math.PI;

    scene.add(mountain);
}


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

        color:0x20c6e5,

        roughness:.075,

        metalness:.07,

        transmission:.08,

        transparent:true,

        opacity:.85,

        clearcoat:1,

        clearcoatRoughness:.035
    });

const water =
    new THREE.Mesh(
        waterGeometry,
        waterMaterial
    );

water.rotation.x =
    -Math.PI/2;

water.position.y =
    -4;

scene.add(water);


/* =========================================================
   SOLID ISLAND
========================================================= */

const island =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            390,
            4,
            390
        ),

        new THREE.MeshStandardMaterial({

            color:0x3c9847,

            roughness:.98
        })
    );

island.position.y =
    -1.7;

island.receiveShadow = true;

scene.add(island);


/* =========================================================
   GRASS
========================================================= */

const grass =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            370,
            .42,
            370
        ),

        new THREE.MeshStandardMaterial({

            color:0x31ad42,

            roughness:.96
        })
    );

grass.position.y =
    .15;

grass.receiveShadow = true;

scene.add(grass);


/* =========================================================
   BEACH
========================================================= */

const beach =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            154,
            154,
            .55,
            128
        ),

        new THREE.MeshStandardMaterial({

            color:0xf1df9e,

            roughness:.9
        })
    );

beach.position.y =
    .43;

beach.receiveShadow = true;

scene.add(beach);


/* =========================================================
   INNER GRASS
========================================================= */

const innerGrass =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            132,
            132,
            .58,
            128
        ),

        new THREE.MeshStandardMaterial({

            color:0x48ca51,

            roughness:.86
        })
    );

innerGrass.position.y =
    .73;

innerGrass.receiveShadow = true;

scene.add(innerGrass);


/* =========================================================
   GRASS DETAIL
========================================================= */

const bladeGeometry =
    new THREE.PlaneGeometry(
        .12,
        .72
    );

const bladeMaterial =
    new THREE.MeshStandardMaterial({

        color:0x269d39,

        side:THREE.DoubleSide,

        roughness:1
    });

const grassBlades =
    new THREE.InstancedMesh(
        bladeGeometry,
        bladeMaterial,
        1700
    );

const temp =
    new THREE.Object3D();

for(
    let i=0;
    i<1700;
    i++
){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        30+
        Math.random()*105;

    temp.position.set(
        Math.cos(angle)*radius,
        1,
        Math.sin(angle)*radius
    );

    temp.rotation.y =
        Math.random()*Math.PI;

    const scale =
        .5+
        Math.random()*.8;

    temp.scale.set(
        scale,
        scale,
        scale
    );

    temp.updateMatrix();

    grassBlades.setMatrixAt(
        i,
        temp.matrix
    );
}

grassBlades.instanceMatrix.needsUpdate =
    true;

scene.add(grassBlades);


/* =========================================================
   ROCKS
========================================================= */

const rockMaterial =
    new THREE.MeshStandardMaterial({

        color:0x88bec0,

        roughness:.8
    });

for(
    let i=0;
    i<60;
    i++
){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        105+
        Math.random()*48;

    const rock =
        new THREE.Mesh(

            new THREE.DodecahedronGeometry(
                .7+
                Math.random()*2,
                1
            ),

            rockMaterial
        );

    rock.position.set(
        Math.cos(angle)*radius,
        1,
        Math.sin(angle)*radius
    );

    rock.scale.y =
        .45+
        Math.random()*.55;

    rock.rotation.set(
        Math.random(),
        Math.random(),
        Math.random()
    );

    rock.castShadow = true;

    scene.add(rock);
}


/* =========================================================
   TREES
========================================================= */

const trunkGeometry =
    new THREE.CylinderGeometry(
        .32,
        .62,
        5.8,
        10
    );

const trunkMaterial =
    new THREE.MeshStandardMaterial({

        color:0x714c2d,

        roughness:.92
    });

const leafGeometry =
    new THREE.IcosahedronGeometry(
        3.2,
        2
    );

const leafMaterial =
    new THREE.MeshPhysicalMaterial({

        color:0x20a94c,

        roughness:.64,

        clearcoat:.2
    });


for(
    let i=0;
    i<105;
    i++
){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        40+
        Math.random()*85;

    const tree =
        new THREE.Group();


    const trunk =
        new THREE.Mesh(
            trunkGeometry,
            trunkMaterial
        );

    trunk.position.y=3;

    trunk.castShadow=true;

    tree.add(trunk);


    const crown =
        new THREE.Mesh(
            leafGeometry,
            leafMaterial
        );

    crown.position.y=7;

    crown.scale.set(
        1.05,
        1.18,
        1.05
    );

    crown.castShadow=true;

    tree.add(crown);


    const crown2 =
        new THREE.Mesh(
            leafGeometry,
            leafMaterial
        );

    crown2.position.set(
        1.1,
        7.6,
        .2
    );

    crown2.scale.setScalar(.58);

    crown2.castShadow=true;

    tree.add(crown2);


    tree.position.set(
        Math.cos(angle)*radius,
        .8,
        Math.sin(angle)*radius
    );

    tree.rotation.y =
        Math.random()*Math.PI;

    scene.add(tree);
}


/* =========================================================
   CHARACTER
   MATCHES THE PICTURE'S SILHOUETTE
========================================================= */


/*
    The picture has:

    - completely blank round head
    - large rounded triangular/teardrop body
    - long rounded arms
    - two separated thick rounded legs
    - no face
    - no armor
    - no clothing
*/


function makeBlueMaterial(){

    return new THREE.MeshPhysicalMaterial({

        vertexColors:true,

        roughness:.11,

        metalness:.015,

        clearcoat:1,

        clearcoatRoughness:.045,

        sheen:.18,

        sheenRoughness:.16
    });
}


function addGradient(
    geometry,
    dark,
    middle,
    light
){

    const position =
        geometry.attributes.position;

    let minY=Infinity;
    let maxY=-Infinity;


    for(
        let i=0;
        i<position.count;
        i++
    ){

        const y =
            position.getY(i);

        minY =
            Math.min(
                minY,
                y
            );

        maxY =
            Math.max(
                maxY,
                y
            );
    }


    const colors=[];

    const cDark =
        new THREE.Color(dark);

    const cMiddle =
        new THREE.Color(middle);

    const cLight =
        new THREE.Color(light);


    for(
        let i=0;
        i<position.count;
        i++
    ){

        const y =
            position.getY(i);

        const t =
            (y-minY) /
            Math.max(
                maxY-minY,
                .001
            );

        const color =
            new THREE.Color();


        if(t<.48){

            color.lerpColors(
                cDark,
                cMiddle,
                t/.48
            );

        }else{

            color.lerpColors(
                cMiddle,
                cLight,
                (t-.48)/.52
            );
        }


        colors.push(
            color.r,
            color.g,
            color.b
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


/* =========================================================
   BODY BLOB
========================================================= */

function makeBodyGeometry(){

    const geometry =
        new THREE.SphereGeometry(
            1,
            56,
            40
        );

    const p =
        geometry.attributes.position;


    for(
        let i=0;
        i<p.count;
        i++
    ){

        let x =
            p.getX(i);

        let y =
            p.getY(i);

        let z =
            p.getZ(i);


        /*
           Shape follows the picture:

           narrow rounded top
           wider middle
           broad soft lower body
        */

        const normalized =
            (y+1)/2;


        let width;

        if(normalized>.72){

            width =
                .70+
                (1-normalized)*1.2;

        }else{

            width =
                1.0+
                normalized*.20;
        }


        x*=width;

        z*=.76;


        /* slightly flatten bottom */

        if(y<-.65){

            x*=.94;
            z*=.92;
        }


        p.setXYZ(
            i,
            x,
            y,
            z
        );
    }


    return geometry;
}


/* =========================================================
   PLAYER GROUP
========================================================= */

const player =
    new THREE.Group();

scene.add(player);


/* =========================================================
   HEAD
========================================================= */

const headGeometry =
    addGradient(

        new THREE.SphereGeometry(
            1.13,
            56,
            40
        ),

        0x0a86d5,
        0x35d4e9,
        0xe0ffff
    );

const characterMaterial =
    makeBlueMaterial();

const head =
    new THREE.Mesh(
        headGeometry,
        characterMaterial
    );


/*
   IMPORTANT:
   There is intentionally NOTHING
   on the front of the head.
*/

head.position.y =
    4.42;

head.scale.set(
    1.02,
    1.02,
    .96
);

head.castShadow = true;

player.add(head);


/* =========================================================
   BODY
========================================================= */

const bodyGeometry =
    addGradient(

        makeBodyGeometry(),

        0x0879d0,
        0x21c9e7,
        0xd4ffff
    );

const body =
    new THREE.Mesh(
        bodyGeometry,
        characterMaterial
    );

body.position.y =
    2.55;

body.scale.set(
    1.34,
    1.42,
    .91
);

body.castShadow = true;

player.add(body);


/* =========================================================
   ARMS
========================================================= */

function makeArm(){

    const geometry =
        new THREE.CapsuleGeometry(
            .39,
            1.72,
            20,
            30
        );

    addGradient(
        geometry,
        0x087ad0,
        0x31d0e7,
        0xe0ffff
    );

    return new THREE.Mesh(
        geometry,
        characterMaterial
    );
}


const leftArm =
    makeArm();

leftArm.position.set(
    -1.56,
    2.30,
    0
);

leftArm.rotation.z =
    -0.18;

leftArm.scale.set(
    .93,
    1.05,
    .92
);

leftArm.castShadow=true;

player.add(leftArm);


const rightArm =
    makeArm();

rightArm.position.set(
    1.56,
    2.30,
    0
);

rightArm.rotation.z =
    .18;

rightArm.scale.set(
    .93,
    1.05,
    .92
);

rightArm.castShadow=true;

player.add(rightArm);


/* =========================================================
   LEGS
========================================================= */

function makeLeg(){

    const geometry =
        new THREE.CapsuleGeometry(
            .47,
            1.55,
            20,
            30
        );

    addGradient(
        geometry,
        0x087bd1,
        0x30cde7,
        0xd5ffff
    );

    return new THREE.Mesh(
        geometry,
        characterMaterial
    );
}


const leftLeg =
    makeLeg();

leftLeg.position.set(
    -.60,
    .76,
    0
);

leftLeg.scale.set(
    1.02,
    1.08,
    .95
);

leftLeg.castShadow=true;

player.add(leftLeg);


const rightLeg =
    makeLeg();

rightLeg.position.set(
    .60,
    .76,
    0
);

rightLeg.scale.set(
    1.02,
    1.08,
    .95
);

rightLeg.castShadow=true;

player.add(rightLeg);


/* =========================================================
   DOLPHIN
========================================================= */

const dolphin =
    new THREE.Group();

const dolphinMaterial =
    new THREE.MeshPhysicalMaterial({

        color:0x56cce9,

        roughness:.12,

        metalness:.025,

        clearcoat:1,

        clearcoatRoughness:.055
    });


const dolphinBody =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            2,
            40,
            28
        ),

        dolphinMaterial
    );

dolphinBody.scale.set(
    1.85,
    .63,
    .84
);

dolphinBody.castShadow=true;

dolphin.add(dolphinBody);


const snout =
    new THREE.Mesh(

        new THREE.CapsuleGeometry(
            .37,
            1.3,
            14,
            22
        ),

        dolphinMaterial
    );

snout.rotation.z =
    -Math.PI/2;

snout.position.x =
    3;

dolphin.add(snout);


const dorsal =
    new THREE.Mesh(

        new THREE.ConeGeometry(
            .68,
            1.7,
            4
        ),

        dolphinMaterial
    );

dorsal.rotation.z =
    Math.PI;

dorsal.position.y =
    1;

dolphin.add(dorsal);


const tail =
    new THREE.Mesh(

        new THREE.ConeGeometry(
            1.05,
            1.7,
            5
        ),

        dolphinMaterial
    );

tail.rotation.z =
    Math.PI/2;

tail.position.x =
    -3;

dolphin.add(tail);


dolphin.position.set(
    18,
    2,
    -18
);

dolphin.rotation.y =
    Math.PI/2;

scene.add(dolphin);


/* =========================================================
   INPUT
========================================================= */

const keys={};

window.addEventListener(
    "keydown",
    e=>{
        keys[
            e.key.toLowerCase()
        ]=true;
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
   RUN BUTTON
========================================================= */

let running=false;

const runButton =
    document.getElementById(
        "run"
    );

runButton.addEventListener(
    "pointerdown",
    e=>{
        e.preventDefault();

        running=true;

        runButton.classList.add(
            "active"
        );
    }
);

function stopRunning(){

    running=false;

    runButton.classList.remove(
        "active"
    );
}

runButton.addEventListener(
    "pointerup",
    stopRunning
);

runButton.addEventListener(
    "pointercancel",
    stopRunning
);

runButton.addEventListener(
    "pointerleave",
    e=>{
        if(e.buttons===0)
            stopRunning();
    }
);


/* =========================================================
   JOYSTICK
========================================================= */

const joystick =
    document.getElementById(
        "joystick"
    );

const stick =
    document.getElementById(
        "stick"
    );

let joyX=0;
let joyY=0;
let joyActive=false;


function updateJoystick(e){

    const rect =
        joystick.getBoundingClientRect();

    let x =
        e.clientX -
        (rect.left+
        rect.width/2);

    let y =
        e.clientY -
        (rect.top+
        rect.height/2);

    const max=43;

    const distance =
        Math.sqrt(
            x*x+y*y
        );

    if(distance>max){

        x =
            x/distance*max;

        y =
            y/distance*max;
    }


    joyX =
        x/max;

    joyY =
        y/max;


    stick.style.transform =
        `translate(${x}px,${y}px)`;
}


joystick.addEventListener(
    "pointerdown",
    e=>{

        joyActive=true;

        joystick.setPointerCapture(
            e.pointerId
        );

        updateJoystick(e);
    }
);


joystick.addEventListener(
    "pointermove",
    e=>{

        if(joyActive)
            updateJoystick(e);
    }
);


function releaseJoystick(){

    joyActive=false;

    joyX=0;
    joyY=0;

    stick.style.transform =
        "translate(0,0)";
}


joystick.addEventListener(
    "pointerup",
    releaseJoystick
);

joystick.addEventListener(
    "pointercancel",
    releaseJoystick
);


/* =========================================================
   JUMP
========================================================= */

let jumping=false;
let verticalVelocity=0;

document.getElementById(
    "jump"
).addEventListener(
    "pointerdown",
    ()=>{

        if(
            !jumping &&
            !riding
        ){

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
    document.getElementById(
        "shift"
    );


shiftButton.addEventListener(
    "pointerdown",
    ()=>{

        shiftLock =
            !shiftLock;

        shiftButton.classList.toggle(
            "active",
            shiftLock
        );


        if(shiftLock){

            /*
               Lock the camera to the
               direction the character
               is currently facing.
            */

            cameraYaw =
                player.rotation.y;
        }
    }
);


/* =========================================================
   CAMERA DRAG
========================================================= */

let dragging=false;
let lastPointerX=0;


renderer.domElement.addEventListener(
    "pointerdown",
    e=>{

        /*
           IMPORTANT:
           Shift Lock completely blocks
           camera dragging.
        */

        if(shiftLock)
            return;

        dragging=true;

        lastPointerX =
            e.clientX;
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


        const delta =
            e.clientX -
            lastPointerX;

        lastPointerX =
            e.clientX;


        cameraYaw -=
            delta*.007;
    }
);


window.addEventListener(
    "pointerup",
    ()=>{
        dragging=false;
    }
);


/* =========================================================
   RIDE
========================================================= */

let riding=false;

const rideButton =
    document.getElementById(
        "ride"
    );


rideButton.addEventListener(
    "pointerdown",
    ()=>{

        const distance =
            player.position.distanceTo(
                dolphin.position
            );


        if(
            distance<13 ||
            riding
        ){

            riding =
                !riding;

            player.visible =
                !riding;


            if(riding){

                dolphin.add(
                    player
                );

                player.position.set(
                    0,
                    2.25,
                    0
                );

                player.rotation.set(
                    0,
                    0,
                    0
                );

            }else{

                scene.add(
                    player
                );

                player.position.y=
                    .85;
            }
        }
    }
);


/* =========================================================
   ANIMATION
========================================================= */

const clock =
    new THREE.Clock();

let animationTime=0;


function animate(){

    requestAnimationFrame(
        animate
    );


    const dt =
        Math.min(
            clock.getDelta(),
            .033
        );


    const time =
        performance.now()*.001;


    /* =====================================================
       MOVING CLOUDS
    ===================================================== */

    for(
        const cloud of clouds
    ){

        cloud.object.position.x +=
            cloud.speed*dt;


        cloud.object.position.y =
            cloud.baseY+
            Math.sin(
                time*.07+
                cloud.phase
            )*.45;


        cloud.object.rotation.y =
            Math.sin(
                time*.025+
                cloud.phase
            )*.018;


        if(
            cloud.object.position.x>320
        ){

            cloud.object.position.x=
                -320;
        }
    }


    /* =====================================================
       INPUT
    ===================================================== */

    let inputX=0;
    let inputY=0;


    /*
       KEYBOARD

       W / UP    = FORWARD
       S / DOWN  = BACKWARD
       A / LEFT  = LEFT
       D / RIGHT = RIGHT
    */

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


    /*
       MOBILE JOYSTICK OVERRIDES
       KEYBOARD INPUT
    */

    if(
        Math.abs(joyX)>.06 ||
        Math.abs(joyY)>.06
    ){

        inputX=joyX;

        inputY=joyY;
    }


    const inputMagnitude =
        Math.sqrt(
            inputX*inputX+
            inputY*inputY
        );


    if(
        inputMagnitude>1
    ){

        inputX /=
            inputMagnitude;

        inputY /=
            inputMagnitude;
    }


    const moving =
        inputMagnitude>.07;


    /* =====================================================
       CAMERA RELATIVE DIRECTIONS

       THIS IS THE IMPORTANT FIX.

       Joystick UP:
           joyY = -1

       -inputY:
           +1

       +forward:
           character moves FORWARD.

       Therefore the controls are NOT inverted.
    ===================================================== */

    const forward =
        new THREE.Vector3(
            Math.sin(cameraYaw),
            0,
            -Math.cos(cameraYaw)
        );


    const right =
        new THREE.Vector3(
            Math.cos(cameraYaw),
            0,
            Math.sin(cameraYaw)
        );


    const movement =
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
        movement.lengthSq()>.0001
    ){

        movement.normalize();


        let speed =
            running
            ?15
            :8.5;


        if(riding)
            speed=13.5;


        if(riding){

            dolphin.position.addScaledVector(
                movement,
                speed*dt
            );

        }else{

            player.position.addScaledVector(
                movement,
                speed*dt
            );
        }


        /*
           Character direction.

           In normal mode it faces
           exactly where it walks.

           In Shift Lock it faces
           the locked camera direction.
        */

        if(!riding){

            if(shiftLock){

                player.rotation.y =
                    cameraYaw;

            }else{

                player.rotation.y =
                    Math.atan2(
                        movement.x,
                        -movement.z
                    );
            }
        }
    }


    /* =====================================================
       JUMP
    ===================================================== */

    if(
        jumping &&
        !riding
    ){

        verticalVelocity -=
            22*dt;

        player.position.y +=
            verticalVelocity*dt;


        if(
            player.position.y<=.85
        ){

            player.position.y=.85;

            verticalVelocity=0;

            jumping=false;
        }

    }else if(!riding){

        player.position.y=.85;
    }


    /* =====================================================
       WORLD BOUNDARY
    ===================================================== */

    function keepInside(
        object
    ){

        const distance =
            Math.sqrt(
                object.position.x*
                object.position.x+
                object.position.z*
                object.position.z
            );


        if(distance>174){

            const scale =
                174/distance;

            object.position.x *=
                scale;

            object.position.z *=
                scale;
        }
    }


    if(riding){

        keepInside(dolphin);

    }else{

        keepInside(player);
    }


    /* =====================================================
       WALK / RUN ANIMATION
    ===================================================== */

    if(
        moving &&
        !riding
    ){

        animationTime +=
            dt*
            (
                running
                ?18
                :10
            );


        const swing =
            Math.sin(
                animationTime
            )*
            (
                running
                ?.72
                :.40
            );


        /*
           Arms swing opposite
           legs.
        */

        leftLeg.rotation.x =
            swing;

        rightLeg.rotation.x =
            -swing;


        leftArm.rotation.x =
            -swing*.82;

        rightArm.rotation.x =
            swing*.82;


        /*
           Small natural bounce.
        */

        const bounce =
            Math.abs(
                Math.sin(
                    animationTime*2
                )
            )*
            (
                running
                ?.075
                :.035
            );


        body.position.y =
            2.55+bounce;

        head.position.y =
            4.42+bounce;


        /*
           Slight run lean.
        */

        body.rotation.x =
            running
            ?-.045
            :0;

    }else{

        leftLeg.rotation.x =
            THREE.MathUtils.lerp(
                leftLeg.rotation.x,
                0,
                .15
            );

        rightLeg.rotation.x =
            THREE.MathUtils.lerp(
                rightLeg.rotation.x,
                0,
                .15
            );

        leftArm.rotation.x =
            THREE.MathUtils.lerp(
                leftArm.rotation.x,
                0,
                .15
            );

        rightArm.rotation.x =
            THREE.MathUtils.lerp(
                rightArm.rotation.x,
                0,
                .15
            );

        body.rotation.x =
            THREE.MathUtils.lerp(
                body.rotation.x,
                0,
                .12
            );


        body.position.y =
            THREE.MathUtils.lerp(
                body.position.y,
                2.55,
                .12
            );

        head.position.y =
            THREE.MathUtils.lerp(
                head.position.y,
                4.42,
                .12
            );
    }


    /* =====================================================
       IDLE FLOAT
    ===================================================== */

    if(
        !moving &&
        !jumping &&
        !riding
    ){

        const idle =
            Math.sin(
                time*1.7
            )*.025;


        body.position.y =
            2.55+idle;

        head.position.y =
            4.42+idle;
    }


    /* =====================================================
       DOLPHIN FLOAT
    ===================================================== */

    if(!riding){

        dolphin.position.y =
            2+
            Math.sin(
                time*1.6
            )*.3;
    }


    dolphin.rotation.z =
        Math.sin(
            time*1.15
        )*.035;


    /* =====================================================
       WATER
    ===================================================== */

    /*
       Don't rebuild normals every frame.
       The material's reflection and
       moving vertices already give
       the water movement.
    */

    const waterPosition =
        waterGeometry.attributes.position;


    for(
        let i=0;
        i<waterPosition.count;
        i++
    ){

        const x =
            waterPosition.getX(i);

        const z =
            waterPosition.getY(i);


        waterPosition.setZ(
            i,

            Math.sin(
                x*.035+
                time*.8
            )*.17+

            Math.cos(
                z*.041+
                time
            )*.14
        );
    }


    waterPosition.needsUpdate=true;


    /* =====================================================
       CAMERA
    ===================================================== */

    let target;


    if(riding){

        target =
            dolphin.position.clone();

        target.y += 1.8;

    }else{

        target =
            player.position.clone();

        target.y += 2.15;
    }


    const cameraDistance =
        shiftLock
        ?8.5
        :10.5;


    const desired =
        target.clone();


    /*
       Camera stays behind character.
    */

    desired.x +=
        Math.sin(cameraYaw)*
        cameraDistance;


    desired.z +=
        Math.cos(cameraYaw)*
        cameraDistance;


    desired.y +=
        4.5;


    camera.position.lerp(
        desired,
        .105
    );


    camera.lookAt(
        target
    );


    /* =====================================================
       RENDER
    ===================================================== */

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

window.addEventListener(
    "resize",
    ()=>{

        camera.aspect =
            innerWidth/
            innerHeight;

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
