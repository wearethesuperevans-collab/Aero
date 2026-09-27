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
    background:#62dfff;
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
    color:white;
    font-weight:800;
    font-size:12px;
    pointer-events:auto;
    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
    box-shadow:
        0 7px 20px rgba(0,40,80,.25),
        inset 0 2px 5px rgba(255,255,255,.55),
        inset 0 -5px 10px rgba(0,80,120,.15);
    backdrop-filter:blur(8px);
}

#title{
    position:absolute;
    top:15px;
    left:50%;
    transform:translateX(-50%);
    color:white;
    font-size:17px;
    letter-spacing:.5px;
    text-shadow:
        0 2px 4px rgba(0,70,110,.8),
        0 0 15px rgba(255,255,255,.8);
    white-space:nowrap;
}

#shift{
    position:absolute;
    top:13px;
    right:13px;
    padding:12px 15px;
    border-radius:16px;
    background:rgba(8,110,175,.72);
}

#shift.active{
    background:rgba(0,190,120,.88);
}

#run{
    position:absolute;
    right:23px;
    bottom:145px;
    width:74px;
    height:74px;
    border-radius:50%;
    background:rgba(20,135,210,.84);
}

#run.active{
    background:rgba(0,190,115,.95);
    transform:scale(1.07);
}

#jump{
    position:absolute;
    right:23px;
    bottom:55px;
    width:74px;
    height:74px;
    border-radius:50%;
    background:rgba(0,170,120,.84);
}

#ride{
    position:absolute;
    right:110px;
    bottom:66px;
    width:65px;
    height:65px;
    border-radius:50%;
    background:rgba(0,130,205,.85);
}

#joystick{
    position:absolute;
    left:23px;
    bottom:37px;
    width:130px;
    height:130px;
    border-radius:50%;
    background:
        radial-gradient(
            circle,
            rgba(255,255,255,.28),
            rgba(255,255,255,.10)
        );
    border:2px solid rgba(255,255,255,.5);
    box-shadow:
        inset 0 0 25px rgba(255,255,255,.2),
        0 8px 25px rgba(0,70,100,.2);
    pointer-events:auto;
}

#stick{
    position:absolute;
    width:58px;
    height:58px;
    left:50%;
    top:50%;
    margin-left:-29px;
    margin-top:-29px;
    border-radius:50%;
    background:
        radial-gradient(
            circle at 35% 25%,
            #ffffff,
            #dffaff 35%,
            #a9e7f5
        );
    box-shadow:
        0 6px 15px rgba(0,60,100,.22),
        inset 0 2px 4px white;
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

/* ============================================================
   SCENE
============================================================ */

const scene = new THREE.Scene();

scene.background =
    new THREE.Color(0x71dcff);

scene.fog =
    new THREE.FogExp2(
        0x8be5f5,
        0.0019
    );


/* ============================================================
   CAMERA
============================================================ */

const camera =
    new THREE.PerspectiveCamera(
        65,
        innerWidth / innerHeight,
        0.1,
        800
    );


/* ============================================================
   RENDERER
============================================================ */

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
    1.18;

renderer.shadowMap.enabled = true;

renderer.shadowMap.type =
    THREE.PCFSoftShadowMap;

document.body.appendChild(
    renderer.domElement
);


/* ============================================================
   LIGHTING
============================================================ */

const hemi =
    new THREE.HemisphereLight(
        0xe9fcff,
        0x245b3c,
        2.8
    );

scene.add(hemi);


const sun =
    new THREE.DirectionalLight(
        0xfff5dc,
        4.2
    );

sun.position.set(
    -100,
    170,
    80
);

sun.castShadow = true;

sun.shadow.mapSize.width = 1536;
sun.shadow.mapSize.height = 1536;

sun.shadow.camera.left = -230;
sun.shadow.camera.right = 230;
sun.shadow.camera.top = 230;
sun.shadow.camera.bottom = -230;

sun.shadow.camera.near = 1;
sun.shadow.camera.far = 500;

sun.shadow.bias = -0.00025;

scene.add(sun);


/* ============================================================
   SUN GLOW
============================================================ */

const sunGlow =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            13,
            24,
            16
        ),

        new THREE.MeshBasicMaterial({
            color:0xfff6bc,
            transparent:true,
            opacity:.35
        })
    );

sunGlow.position.set(
    -120,
    130,
    -180
);

scene.add(sunGlow);


/* ============================================================
   WATER
============================================================ */

const waterGeometry =
    new THREE.PlaneGeometry(
        800,
        800,
        80,
        80
    );

const waterPosition =
    waterGeometry.attributes.position;

for(let i=0;i<waterPosition.count;i++){

    const x =
        waterPosition.getX(i);

    const z =
        waterPosition.getY(i);

    waterPosition.setZ(
        i,
        Math.sin(x*.035)*
        .18 +
        Math.cos(z*.041)*
        .15
    );
}

waterGeometry.computeVertexNormals();

const water =
    new THREE.Mesh(

        waterGeometry,

        new THREE.MeshPhysicalMaterial({

            color:0x20bde0,

            roughness:.12,

            metalness:.08,

            transmission:.08,

            transparent:true,

            opacity:.82,

            clearcoat:1,

            clearcoatRoughness:.08
        })
    );

water.rotation.x =
    -Math.PI/2;

water.position.y=-4;

scene.add(water);


/* ============================================================
   SOLID ISLAND BASE
============================================================ */

const islandBase =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            390,
            4,
            390
        ),

        new THREE.MeshStandardMaterial({

            color:0x3f9c48,

            roughness:.98
        })
    );

islandBase.position.y=-1.7;

islandBase.receiveShadow=true;

scene.add(islandBase);


/* ============================================================
   GRASS
============================================================ */

const grass =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            370,
            .42,
            370
        ),

        new THREE.MeshStandardMaterial({

            color:0x35ad43,

            roughness:.96
        })
    );

grass.position.y=.15;

grass.receiveShadow=true;

scene.add(grass);


/* ============================================================
   BEACH
============================================================ */

const beach =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            154,
            154,
            .55,
            128
        ),

        new THREE.MeshStandardMaterial({

            color:0xf0df9c,

            roughness:.91
        })
    );

beach.position.y=.43;

beach.receiveShadow=true;

scene.add(beach);


/* ============================================================
   INNER GRASSLAND
============================================================ */

const innerGrass =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            132,
            132,
            .58,
            128
        ),

        new THREE.MeshStandardMaterial({

            color:0x48c953,

            roughness:.88
        })
    );

innerGrass.position.y=.73;

innerGrass.receiveShadow=true;

scene.add(innerGrass);


/* ============================================================
   GRASS BLADES
============================================================ */

const bladeGeo =
    new THREE.PlaneGeometry(
        .13,
        .75
    );

const bladeMat =
    new THREE.MeshStandardMaterial({

        color:0x299f3c,

        side:THREE.DoubleSide,

        roughness:.95
    });

const blades =
    new THREE.InstancedMesh(
        bladeGeo,
        bladeMat,
        1800
    );

const dummy =
    new THREE.Object3D();

for(let i=0;i<1800;i++){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        35+
        Math.random()*105;

    const x =
        Math.cos(angle)*radius;

    const z =
        Math.sin(angle)*radius;

    dummy.position.set(
        x,
        1,
        z
    );

    dummy.rotation.y =
        Math.random()*Math.PI;

    const scale =
        .55+
        Math.random()*.9;

    dummy.scale.set(
        scale,
        scale,
        scale
    );

    dummy.updateMatrix();

    blades.setMatrixAt(
        i,
        dummy.matrix
    );
}

blades.instanceMatrix.needsUpdate=true;

scene.add(blades);


/* ============================================================
   ROCK MATERIALS
============================================================ */

const rockMat =
    new THREE.MeshStandardMaterial({

        color:0x9acbd0,

        roughness:.78
    });


for(let i=0;i<65;i++){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        110+
        Math.random()*43;

    const rock =
        new THREE.Mesh(

            new THREE.DodecahedronGeometry(
                .6+
                Math.random()*2.1,
                1
            ),

            rockMat
        );

    rock.position.set(
        Math.cos(angle)*radius,
        .9,
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

    rock.castShadow=true;

    scene.add(rock);
}


/* ============================================================
   TREES
============================================================ */

const trunkGeo =
    new THREE.CylinderGeometry(
        .32,
        .62,
        5.8,
        10
    );

const trunkMat =
    new THREE.MeshStandardMaterial({

        color:0x76502f,

        roughness:.9
    });


const leafGeo =
    new THREE.IcosahedronGeometry(
        3.2,
        2
    );

const leafMat =
    new THREE.MeshPhysicalMaterial({

        color:0x20a94d,

        roughness:.68,

        clearcoat:.18
    });


for(let i=0;i<105;i++){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        40+
        Math.random()*86;

    const tree =
        new THREE.Group();


    const trunk =
        new THREE.Mesh(
            trunkGeo,
            trunkMat
        );

    trunk.position.y=3;

    trunk.castShadow=true;

    tree.add(trunk);


    const crown1 =
        new THREE.Mesh(
            leafGeo,
            leafMat
        );

    crown1.position.y=7;

    crown1.scale.set(
        1.05,
        1.15,
        1.05
    );

    crown1.castShadow=true;

    tree.add(crown1);


    const crown2 =
        new THREE.Mesh(
            leafGeo,
            leafMat
        );

    crown2.position.set(
        1.25,
        7.5,
        .25
    );

    crown2.scale.setScalar(.65);

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


/* ============================================================
   SMALL FLOWERS
============================================================ */

const flowerStemGeo =
    new THREE.CylinderGeometry(
        .025,
        .025,
        .5,
        5
    );

const flowerStemMat =
    new THREE.MeshStandardMaterial({
        color:0x228d35
    });

const flowerGeo =
    new THREE.SphereGeometry(
        .22,
        10,
        8
    );

const flowerMat =
    new THREE.MeshStandardMaterial({
        color:0xffd738,
        roughness:.5
    });


for(let i=0;i<180;i++){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        25+
        Math.random()*105;

    const flower =
        new THREE.Group();

    const stem =
        new THREE.Mesh(
            flowerStemGeo,
            flowerStemMat
        );

    stem.position.y=.5;

    flower.add(stem);

    const head =
        new THREE.Mesh(
            flowerGeo,
            flowerMat
        );

    head.position.y=.85;

    flower.add(head);

    flower.position.set(
        Math.cos(angle)*radius,
        .8,
        Math.sin(angle)*radius
    );

    flower.scale.setScalar(
        .6+
        Math.random()*.7
    );

    scene.add(flower);
}


/* ============================================================
   CHARACTER MATERIAL
============================================================ */

function makeCharacterGeometry(
    geometry
){

    const pos =
        geometry.attributes.position;

    const colors=[];

    const top =
        new THREE.Color(
            0xe1fcff
        );

    const cyan =
        new THREE.Color(
            0x52e7ef
        );

    const blue =
        new THREE.Color(
            0x159edc
        );

    let minY=Infinity;
    let maxY=-Infinity;

    for(let i=0;i<pos.count;i++){

        const y=pos.getY(i);

        minY=Math.min(minY,y);
        maxY=Math.max(maxY,y);
    }

    for(let i=0;i<pos.count;i++){

        const y=pos.getY(i);

        const t=
            (y-minY)/
            Math.max(
                maxY-minY,
                .001
            );

        const c =
            new THREE.Color();

        if(t>.58){

            c.lerpColors(
                cyan,
                top,
                (t-.58)/.42
            );

        }else{

            c.lerpColors(
                blue,
                cyan,
                t/.58
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


const characterMaterial =
    new THREE.MeshPhysicalMaterial({

        vertexColors:true,

        roughness:.12,

        metalness:.02,

        clearcoat:1,

        clearcoatRoughness:.07,

        sheen:.18,

        sheenRoughness:.18
    });


/* ============================================================
   CHARACTER
   SAME BASIC SILHOUETTE — NO FACE
============================================================ */

const player =
    new THREE.Group();

scene.add(player);


/* HEAD */

const head =
    new THREE.Mesh(

        makeCharacterGeometry(
            new THREE.SphereGeometry(
                1.12,
                48,
                32
            )
        ),

        characterMaterial
    );

head.position.y=4.42;

head.scale.set(
    1.03,
    1.03,
    .96
);

head.castShadow=true;

player.add(head);


/* BODY */

const body =
    new THREE.Mesh(

        makeCharacterGeometry(
            new THREE.SphereGeometry(
                1.3,
                48,
                32
            )
        ),

        characterMaterial
    );

body.position.y=2.56;

body.scale.set(
    1.17,
    1.28,
    .78
);

body.castShadow=true;

player.add(body);


/* LEFT ARM */

const leftArm =
    new THREE.Mesh(

        makeCharacterGeometry(

            new THREE.CapsuleGeometry(
                .39,
                1.72,
                16,
                24
            )
        ),

        characterMaterial
    );

leftArm.position.set(
    -1.53,
    2.3,
    0
);

leftArm.rotation.z=-.20;

leftArm.castShadow=true;

player.add(leftArm);


/* RIGHT ARM */

const rightArm =
    new THREE.Mesh(

        makeCharacterGeometry(

            new THREE.CapsuleGeometry(
                .39,
                1.72,
                16,
                24
            )
        ),

        characterMaterial
    );

rightArm.position.set(
    1.53,
    2.3,
    0
);

rightArm.rotation.z=.20;

rightArm.castShadow=true;

player.add(rightArm);


/* LEFT LEG */

const leftLeg =
    new THREE.Mesh(

        makeCharacterGeometry(

            new THREE.CapsuleGeometry(
                .45,
                1.58,
                16,
                24
            )
        ),

        characterMaterial
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


/* RIGHT LEG */

const rightLeg =
    new THREE.Mesh(

        makeCharacterGeometry(

            new THREE.CapsuleGeometry(
                .45,
                1.58,
                16,
                24
            )
        ),

        characterMaterial
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


/* ============================================================
   DOLPHIN
============================================================ */

const dolphin =
    new THREE.Group();

const dolphinMaterial =
    new THREE.MeshPhysicalMaterial({

        color:0x59c9e8,

        roughness:.16,

        metalness:.04,

        clearcoat:1,

        clearcoatRoughness:.08
    });


const dolphinBody =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            2,
            36,
            24
        ),

        dolphinMaterial
    );

dolphinBody.scale.set(
    1.8,
    .62,
    .82
);

dolphinBody.castShadow=true;

dolphin.add(dolphinBody);


const dolphinSnout =
    new THREE.Mesh(

        new THREE.CapsuleGeometry(
            .38,
            1.25,
            10,
            18
        ),

        dolphinMaterial
    );

dolphinSnout.rotation.z=
    -Math.PI/2;

dolphinSnout.position.x=3;

dolphin.add(dolphinSnout);


/* FIN */

const finGeo =
    new THREE.ConeGeometry(
        .7,
        1.7,
        4
    );

const topFin =
    new THREE.Mesh(
        finGeo,
        dolphinMaterial
    );

topFin.rotation.z=
    Math.PI;

topFin.position.set(
    0,
    1,
    0
);

dolphin.add(topFin);


/* TAIL */

const tail =
    new THREE.Mesh(

        new THREE.ConeGeometry(
            1.1,
            1.7,
            5
        ),

        dolphinMaterial
    );

tail.rotation.z=
    Math.PI/2;

tail.position.x=-3;

dolphin.add(tail);


dolphin.position.set(
    20,
    2,
    -20
);

dolphin.rotation.y=
    Math.PI/2;

scene.add(dolphin);


/* ============================================================
   INPUT
============================================================ */

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


/* ============================================================
   RUN
============================================================ */

let running=false;

const runButton=
    document.getElementById(
        "run"
    );

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

    runButton.classList.remove(
        "active"
    );
}

runButton.addEventListener(
    "pointerup",
    stopRun
);

runButton.addEventListener(
    "pointercancel",
    stopRun
);


/* ============================================================
   JOYSTICK
============================================================ */

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

function moveStick(e){

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

        moveStick(e);
    }
);

joystick.addEventListener(
    "pointermove",
    e=>{

        if(joyActive)
            moveStick(e);
    }
);

function releaseStick(){

    joyActive=false;

    joyX=0;
    joyY=0;

    stick.style.transform=
        "translate(0,0)";
}

joystick.addEventListener(
    "pointerup",
    releaseStick
);

joystick.addEventListener(
    "pointercancel",
    releaseStick
);


/* ============================================================
   JUMP
============================================================ */

let jumping=false;
let velocityY=0;

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
            velocityY=9;
        }
    }
);


/* ============================================================
   SHIFT LOCK
============================================================ */

let shiftLock=false;

let yaw=0;

const shiftButton=
    document.getElementById(
        "shift"
    );

shiftButton.addEventListener(
    "pointerdown",
    ()=>{

        shiftLock=!shiftLock;

        shiftButton.classList.toggle(
            "active",
            shiftLock
        );

        if(shiftLock){

            yaw=
                player.rotation.y;
        }
    }
);


/* ============================================================
   CAMERA DRAG
============================================================ */

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
            e.clientX-
            previousX;

        previousX=
            e.clientX;

        yaw-=dx*.007;
    }
);

window.addEventListener(
    "pointerup",
    ()=>{
        dragging=false;
    }
);


/* ============================================================
   RIDE
============================================================ */

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

            player.visible=
                !riding;
        }
    }
);


/* ============================================================
   ANIMATION
============================================================ */

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


    /* --------------------------------------------------------
       INPUT
    -------------------------------------------------------- */

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


    /* --------------------------------------------------------
       CAMERA DIRECTIONS
    -------------------------------------------------------- */

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
            ? 15
            : 8.5;


        if(riding)
            speed=14;


        const object=
            riding
            ? dolphin
            : player;


        object.position.addScaledVector(
            movement,
            speed*dt
        );


        if(!riding){

            if(shiftLock){

                player.rotation.y=
                    yaw;

            }else{

                player.rotation.y=
                    Math.atan2(
                        movement.x,
                        -movement.z
                    );
            }
        }
    }


    /* --------------------------------------------------------
       JUMP
    -------------------------------------------------------- */

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


    /* --------------------------------------------------------
       WORLD BOUNDARY
    -------------------------------------------------------- */

    function keepInside(object){

        const d=
            Math.sqrt(
                object.position.x*
                object.position.x+
                object.position.z*
                object.position.z
            );

        if(d>174){

            const s=
                174/d;

            object.position.x*=s;
            object.position.z*=s;
        }
    }

    keepInside(player);
    keepInside(dolphin);


    /* --------------------------------------------------------
       CHARACTER ANIMATION
    -------------------------------------------------------- */

    if(
        moving &&
        !riding
    ){

        animTime +=
            dt*
            (running ? 18 : 10);


        const swing=
            Math.sin(animTime)*
            (running ? .76 : .43);


        leftLeg.rotation.x=
            swing;

        rightLeg.rotation.x=
            -swing;


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
            (running ? .075 : .035);


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

        body.position.y=
            THREE.MathUtils.lerp(
                body.position.y,
                2.56,
                .12
            );

        head.position.y=
            THREE.MathUtils.lerp(
                head.position.y,
                4.42,
                .12
            );
    }


    /* --------------------------------------------------------
       IDLE FLOAT
    -------------------------------------------------------- */

    if(!moving && !jumping && !riding){

        const idle=
            Math.sin(
                performance.now()*.0018
            )*.025;

        body.position.y=
            2.56+idle;

        head.position.y=
            4.42+idle;
    }


    /* --------------------------------------------------------
       DOLPHIN MOTION
    -------------------------------------------------------- */

    dolphin.position.y=
        2+
        Math.sin(
            performance.now()*.002
        )*.3;


    dolphin.rotation.z=
        Math.sin(
            performance.now()*.0015
        )*.035;


    /* --------------------------------------------------------
       CAMERA
    -------------------------------------------------------- */

    const target=
        riding
        ? dolphin.position.clone()
        : player.position.clone();

    target.y+=2.15;


    const distance=
        shiftLock
        ? 8.8
        : 10.5;


    const desired=
        target.clone();


    desired.x +=
        Math.sin(yaw)*
        distance;


    desired.z +=
        Math.cos(yaw)*
        distance;


    desired.y +=
        4.8;


    camera.position.lerp(
        desired,
        .105
    );


    camera.lookAt(
        target
    );


    /* --------------------------------------------------------
       WATER ANIMATION
    -------------------------------------------------------- */

    const time=
        performance.now()*.0007;

    for(
        let i=0;
        i<waterPosition.count;
        i++
    ){

        const x=
            waterPosition.getX(i);

        const z=
            waterPosition.getY(i);

        waterPosition.setZ(
            i,
            Math.sin(
                x*.035+
                time
            )*.18+
            Math.cos(
                z*.041+
                time*1.2
            )*.15
        );
    }

    waterGeometry.attributes.position
        .needsUpdate=true;

    waterGeometry.computeVertexNormals();


    /* --------------------------------------------------------
       RENDER
    -------------------------------------------------------- */

    renderer.render(
        scene,
        camera
    );
}


/* ============================================================
   START
============================================================ */

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


/* ============================================================
   RESIZE
============================================================ */

window.addEventListener(
    "resize",
    ()=>{

        camera.aspect=
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
