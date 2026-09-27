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

html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#72ddff;
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
    border:none;
    color:white;
    font-weight:bold;
    pointer-events:auto;
    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
    box-shadow:
        0 5px 16px rgba(0,0,0,.22),
        inset 0 1px 2px rgba(255,255,255,.4);
}

#title{
    position:absolute;
    top:15px;
    left:50%;
    transform:translateX(-50%);
    color:white;
    font-size:17px;
    font-weight:bold;
    text-shadow:
        0 2px 5px #16769a,
        0 0 12px rgba(255,255,255,.5);
    white-space:nowrap;
}

#shift{
    position:absolute;
    top:15px;
    right:15px;
    padding:12px 15px;
    border-radius:15px;
    background:rgba(18,120,190,.88);
}

#shift.active{
    background:#08ad73;
}

#run{
    position:absolute;
    right:25px;
    bottom:145px;
    width:72px;
    height:72px;
    border-radius:50%;
    background:rgba(35,145,210,.9);
}

#run.active{
    background:#08b875;
    transform:scale(1.06);
}

#jump{
    position:absolute;
    right:25px;
    bottom:55px;
    width:72px;
    height:72px;
    border-radius:50%;
    background:rgba(10,185,115,.9);
}

#ride{
    position:absolute;
    right:110px;
    bottom:65px;
    width:66px;
    height:66px;
    border-radius:50%;
    background:rgba(20,150,215,.88);
}

#joystick{
    position:absolute;
    left:24px;
    bottom:40px;
    width:126px;
    height:126px;
    border-radius:50%;
    background:rgba(255,255,255,.22);
    border:2px solid rgba(255,255,255,.5);
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
    background:rgba(255,255,255,.72);
    box-shadow:0 4px 12px rgba(0,0,0,.18);
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
    new THREE.Color(0x78ddff);

scene.fog =
    new THREE.Fog(
        0x78ddff,
        170,
        520
    );


/* =========================================================
   CAMERA
========================================================= */

const camera =
    new THREE.PerspectiveCamera(
        65,
        innerWidth/innerHeight,
        .1,
        700
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
        1.3
    )
);

renderer.outputColorSpace =
    THREE.SRGBColorSpace;

renderer.toneMapping =
    THREE.ACESFilmicToneMapping;

renderer.toneMappingExposure =
    1.12;

renderer.shadowMap.enabled=true;

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
        0xd8faff,
        0x38673a,
        2.5
    );

scene.add(hemisphere);


const sun =
    new THREE.DirectionalLight(
        0xffffff,
        3.4
    );

sun.position.set(
    -100,
    150,
    90
);

sun.castShadow=true;

sun.shadow.mapSize.width=1024;
sun.shadow.mapSize.height=1024;

sun.shadow.camera.left=-220;
sun.shadow.camera.right=220;
sun.shadow.camera.top=220;
sun.shadow.camera.bottom=-220;

sun.shadow.camera.near=1;
sun.shadow.camera.far=450;

sun.shadow.bias=-0.0002;

scene.add(sun);


/* =========================================================
   WATER
========================================================= */

const water =
    new THREE.Mesh(
        new THREE.PlaneGeometry(
            700,
            700
        ),

        new THREE.MeshPhysicalMaterial({

            color:0x22bde5,

            roughness:.12,

            metalness:.05,

            transparent:true,

            opacity:.84,

            clearcoat:.8,

            clearcoatRoughness:.1
        })
    );

water.rotation.x =
    -Math.PI/2;

water.position.y=-4;

scene.add(water);


/* =========================================================
   SOLID GROUND
========================================================= */

const ground =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            390,
            4,
            390
        ),

        new THREE.MeshStandardMaterial({

            color:0x42a94d,

            roughness:.95
        })
    );

ground.position.y=-1.7;

ground.receiveShadow=true;

scene.add(ground);


/* =========================================================
   GRASS
========================================================= */

const grass =
    new THREE.Mesh(

        new THREE.BoxGeometry(
            370,
            .4,
            370
        ),

        new THREE.MeshStandardMaterial({

            color:0x4fc75a,

            roughness:.95
        })
    );

grass.position.y=.15;

grass.receiveShadow=true;

scene.add(grass);


/* =========================================================
   BEACH
========================================================= */

const beach =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            153,
            153,
            .42,
            128
        ),

        new THREE.MeshStandardMaterial({

            color:0xf5e19e,

            roughness:.92
        })
    );

beach.position.y=.42;

beach.receiveShadow=true;

scene.add(beach);


/* =========================================================
   INNER GRASS
========================================================= */

const innerGrass =
    new THREE.Mesh(

        new THREE.CylinderGeometry(
            130,
            130,
            .5,
            128
        ),

        new THREE.MeshStandardMaterial({

            color:0x4fc95b,

            roughness:.9
        })
    );

innerGrass.position.y=.68;

innerGrass.receiveShadow=true;

scene.add(innerGrass);


/* =========================================================
   ROCKS
========================================================= */

const rockMaterial =
    new THREE.MeshStandardMaterial({

        color:0x91ccd3,

        roughness:.82
    });

for(let i=0;i<50;i++){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        110+
        Math.random()*38;

    const rock =
        new THREE.Mesh(

            new THREE.IcosahedronGeometry(
                .7+
                Math.random()*1.7,
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
        .5+
        Math.random()*.5;

    rock.rotation.set(
        Math.random(),
        Math.random(),
        Math.random()
    );

    rock.castShadow=true;

    scene.add(rock);
}


/* =========================================================
   TREES
========================================================= */

const trunkGeo =
    new THREE.CylinderGeometry(
        .43,
        .65,
        5.5,
        8
    );

const trunkMat =
    new THREE.MeshStandardMaterial({

        color:0x79512f,

        roughness:.9
    });

const leafGeo =
    new THREE.IcosahedronGeometry(
        3.1,
        1
    );

const leafMat =
    new THREE.MeshStandardMaterial({

        color:0x20a94a,

        roughness:.8
    });

for(let i=0;i<95;i++){

    const angle =
        Math.random()*Math.PI*2;

    const radius =
        35+
        Math.random()*88;

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


    const leaves =
        new THREE.Mesh(
            leafGeo,
            leafMat
        );

    leaves.position.y=7;

    leaves.scale.setScalar(
        .8+
        Math.random()*.4
    );

    leaves.castShadow=true;

    tree.add(leaves);


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
   CHARACTER GRADIENT MATERIAL
========================================================= */

/*
   This makes the character look like the
   reference: light cyan on top,
   blue/cyan through the middle,
   brighter blue toward the bottom.
*/

function gradientGeometry(
    geometry
){

    const pos =
        geometry.attributes.position;

    const colors=[];

    const topColor =
        new THREE.Color(
            0xbaf8ff
        );

    const middleColor =
        new THREE.Color(
            0x25c9e8
        );

    const bottomColor =
        new THREE.Color(
            0x0794d7
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
            (maxY-minY || 1);

        const color=
            new THREE.Color();

        if(t>.5){

            color.lerpColors(
                middleColor,
                topColor,
                (t-.5)*2
            );

        }else{

            color.lerpColors(
                bottomColor,
                middleColor,
                t*2
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


function characterMaterial(){

    return new THREE.MeshPhysicalMaterial({

        vertexColors:true,

        roughness:.18,

        metalness:.03,

        clearcoat:1,

        clearcoatRoughness:.12
    });
}


/* =========================================================
   CHARACTER
========================================================= */

const player =
    new THREE.Group();

scene.add(player);


/*
   HEAD

   Round and floating above the body,
   just like the reference.
*/

const headGeo =
    gradientGeometry(
        new THREE.SphereGeometry(
            1.08,
            32,
            24
        )
    );

const head =
    new THREE.Mesh(
        headGeo,
        characterMaterial()
    );

head.position.y=4.35;

head.castShadow=true;

player.add(head);


/*
   BODY

   Wide rounded blob shape.
*/

const bodyGeo =
    gradientGeometry(

        new THREE.SphereGeometry(
            1.25,
            32,
            24
        )
    );

const body =
    new THREE.Mesh(
        bodyGeo,
        characterMaterial()
    );

body.scale.set(
    1.12,
    1.25,
    .76
);

body.position.y=2.55;

body.castShadow=true;

player.add(body);


/*
   LEFT ARM
*/

const armGeoL =
    gradientGeometry(

        new THREE.CapsuleGeometry(
            .38,
            1.65,
            12,
            20
        )
    );

const leftArm =
    new THREE.Mesh(
        armGeoL,
        characterMaterial()
    );

leftArm.position.set(
    -1.5,
    2.25,
    0
);

leftArm.rotation.z=
    -.22;

leftArm.rotation.x=
    .02;

leftArm.castShadow=true;

player.add(leftArm);


/*
   RIGHT ARM
*/

const armGeoR =
    gradientGeometry(

        new THREE.CapsuleGeometry(
            .38,
            1.65,
            12,
            20
        )
    );

const rightArm =
    new THREE.Mesh(
        armGeoR,
        characterMaterial()
    );

rightArm.position.set(
    1.5,
    2.25,
    0
);

rightArm.rotation.z=
    .22;

rightArm.rotation.x=
    .02;

rightArm.castShadow=true;

player.add(rightArm);


/*
   LEFT LEG
*/

const legGeoL =
    gradientGeometry(

        new THREE.CapsuleGeometry(
            .43,
            1.55,
            12,
            20
        )
    );

const leftLeg =
    new THREE.Mesh(
        legGeoL,
        characterMaterial()
    );

leftLeg.position.set(
    -.57,
    .75,
    0
);

leftLeg.scale.set(
    .9,
    1.08,
    .9
);

leftLeg.castShadow=true;

player.add(leftLeg);


/*
   RIGHT LEG
*/

const legGeoR =
    gradientGeometry(

        new THREE.CapsuleGeometry(
            .43,
            1.55,
            12,
            20
        )
    );

const rightLeg =
    new THREE.Mesh(
        legGeoR,
        characterMaterial()
    );

rightLeg.position.set(
    .57,
    .75,
    0
);

rightLeg.scale.set(
    .9,
    1.08,
    .9
);

rightLeg.castShadow=true;

player.add(rightLeg);


/* =========================================================
   DOLPHIN
========================================================= */

const dolphin =
    new THREE.Group();

const dolphinMat =
    new THREE.MeshPhysicalMaterial({

        color:0x46bce2,

        roughness:.2,

        metalness:.05,

        clearcoat:.8
    });


const dolphinBody =
    new THREE.Mesh(

        new THREE.SphereGeometry(
            2,
            28,
            20
        ),

        dolphinMat
    );

dolphinBody.scale.set(
    1.8,
    .62,
    .8
);

dolphinBody.castShadow=true;

dolphin.add(dolphinBody);


const nose =
    new THREE.Mesh(

        new THREE.ConeGeometry(
            .5,
            1.8,
            18
        ),

        dolphinMat
    );

nose.rotation.z=
    -Math.PI/2;

nose.position.x=3.2;

dolphin.add(nose);


const tail =
    new THREE.Mesh(

        new THREE.ConeGeometry(
            1.2,
            1.8,
            4
        ),

        dolphinMat
    );

tail.rotation.z=
    Math.PI/2;

tail.position.x=-3;

dolphin.add(tail);


dolphin.position.set(
    18,
    2,
    -20
);

dolphin.rotation.y=
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
    stopRunning
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
        e.clientX-
        (
            rect.left+
            rect.width/2
        );

    let y =
        e.clientY-
        (
            rect.top+
            rect.height/2
        );

    const max=43;

    const distance =
        Math.sqrt(
            x*x+y*y
        );

    if(distance>max){

        x=
            x/distance*max;

        y=
            y/distance*max;
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

    stick.style.transform=
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

let yaw=0;

const shiftButton =
    document.getElementById(
        "shift"
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

        if(shiftLock){

            yaw=
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

        if(shiftLock)
            return;

        dragging=true;

        lastPointerX=e.clientX;
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

        const dx =
            e.clientX-
            lastPointerX;

        lastPointerX=
            e.clientX;

        yaw-=dx*.008;
    }
);

renderer.domElement.addEventListener(
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

        const distance =
            player.position.distanceTo(
                dolphin.position
            );

        if(distance<13){

            riding=
                !riding;

            player.visible=
                !riding;
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


    /* =====================================
       MOVEMENT INPUT

       IMPORTANT:
       UP = FORWARD
       DOWN = BACKWARD
    ===================================== */

    let inputX=0;

    let inputY=0;


    if(
        keys["a"] ||
        keys["arrowleft"]
    ){

        inputX=-1;
    }


    if(
        keys["d"] ||
        keys["arrowright"]
    ){

        inputX=1;
    }


    if(
        keys["w"] ||
        keys["arrowup"]
    ){

        inputY=-1;
    }


    if(
        keys["s"] ||
        keys["arrowdown"]
    ){

        inputY=1;
    }


    /*
       Joystick overrides keyboard
       only when actually being used.
    */

    if(
        Math.abs(joyX)>.08 ||
        Math.abs(joyY)>.08
    ){

        inputX=joyX;
        inputY=joyY;
    }


    const inputLength =
        Math.sqrt(
            inputX*inputX+
            inputY*inputY
        );

    if(inputLength>1){

        inputX/=inputLength;
        inputY/=inputLength;
    }


    const moving =
        inputLength>.08;


    /* =====================================
       CAMERA DIRECTIONS
    ===================================== */

    const forward =
        new THREE.Vector3(
            Math.sin(yaw),
            0,
            -Math.cos(yaw)
        );

    const right =
        new THREE.Vector3(
            Math.cos(yaw),
            0,
            Math.sin(yaw)
        );


    const movement =
        new THREE.Vector3();


    movement.addScaledVector(
        right,
        inputX
    );


    /*
       inputY is negative when pushing UP.

       Therefore:

       UP (-1) -> +forward
       DOWN (+1) -> -forward
    */

    movement.addScaledVector(
        forward,
        -inputY
    );


    if(
        movement.lengthSq()>.001
    ){

        movement.normalize();


        /* =================================
           RUN SPEED
        ================================= */

        let speed=8.5;

        if(running)
            speed=15;

        if(riding)
            speed=14;


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


        /* =================================
           CHARACTER ROTATION
        ================================= */

        if(shiftLock){

            /*
               Character faces the same
               direction as Shift Lock.
            */

            player.rotation.y=
                yaw;

        }else{

            /*
               Character faces actual
               movement direction.
            */

            player.rotation.y=
                Math.atan2(
                    movement.x,
                    -movement.z
                );
        }
    }


    /* =====================================
       SHIFT LOCK
    ===================================== */

    if(shiftLock){

        yaw=
            player.rotation.y;
    }


    /* =====================================
       JUMP
    ===================================== */

    if(
        jumping &&
        !riding
    ){

        verticalVelocity-=22*dt;

        player.position.y+=
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


    /* =====================================
       WORLD LIMIT
    ===================================== */

    const worldLimit=173;

    function keepInside(object){

        const distance =
            Math.sqrt(
                object.position.x*
                object.position.x+
                object.position.z*
                object.position.z
            );

        if(distance>worldLimit){

            const scale=
                worldLimit/distance;

            object.position.x*=scale;

            object.position.z*=scale;
        }
    }

    keepInside(player);

    keepInside(dolphin);


    /* =====================================
       WALK / RUN ANIMATION
    ===================================== */

    if(
        moving &&
        !riding
    ){

        /*
           Walking is slower.
           Running is much faster.
        */

        const animationSpeed =
            running ? 17 : 10;

        animationTime+=
            dt*animationSpeed;


        const swingAmount =
            running ? .78 : .43;

        const swing =
            Math.sin(
                animationTime
            )*
            swingAmount;


        /*
           LEGS
        */

        leftLeg.rotation.x=
            swing;

        rightLeg.rotation.x=
            -swing;


        /*
           ARMS
        */

        leftArm.rotation.x=
            -swing*.85;

        rightArm.rotation.x=
            swing*.85;


        /*
           Running gets a little
           more body bounce.
        */

        const bounce =
            running
            ? Math.abs(
                Math.sin(
                    animationTime*2
                )
            )*.07
            : Math.abs(
                Math.sin(
                    animationTime*2
                )
            )*.035;


        body.position.y=
            2.55+bounce;


        head.position.y=
            4.35+bounce;

    }else{

        /*
           Smoothly return to idle.
        */

        leftLeg.rotation.x=
            THREE.MathUtils.lerp(
                leftLeg.rotation.x,
                0,
                .14
            );

        rightLeg.rotation.x=
            THREE.MathUtils.lerp(
                rightLeg.rotation.x,
                0,
                .14
            );

        leftArm.rotation.x=
            THREE.MathUtils.lerp(
                leftArm.rotation.x,
                0,
                .14
            );

        rightArm.rotation.x=
            THREE.MathUtils.lerp(
                rightArm.rotation.x,
                0,
                .14
            );

        body.position.y=
            THREE.MathUtils.lerp(
                body.position.y,
                2.55,
                .12
            );

        head.position.y=
            THREE.MathUtils.lerp(
                head.position.y,
                4.35,
                .12
            );
    }


    /* =====================================
       DOLPHIN BOBBING
    ===================================== */

    dolphin.position.y=
        2+
        Math.sin(
            performance.now()*.002
        )*.3;


    /* =====================================
       CAMERA TARGET
    ===================================== */

    const target =
        riding
        ? dolphin.position.clone()
        : player.position.clone();

    target.y+=2.1;


    const cameraDistance =
        shiftLock
        ? 8.8
        : 10;


    const desiredCamera =
        target.clone();


    desiredCamera.x +=
        Math.sin(yaw)*
        cameraDistance;


    desiredCamera.z +=
        Math.cos(yaw)*
        cameraDistance;


    desiredCamera.y +=
        5;


    camera.position.lerp(
        desiredCamera,
        .12
    );


    camera.lookAt(
        target
    );


    /* =====================================
       RENDER
    ===================================== */

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
    player.position
);

animate();


/* =========================================================
   RESIZE
========================================================= */

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
