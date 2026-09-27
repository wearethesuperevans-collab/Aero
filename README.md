<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Frutiger Aero World</title>

<style>
html,body{
    margin:0;
    padding:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#71dcff;
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
    color:white;
    font-weight:bold;
    pointer-events:auto;
}

#shift{
    position:absolute;
    top:18px;
    right:18px;
    padding:13px 17px;
    border-radius:15px;
    background:#168ac2;
}

#shift.active{
    background:#09ad70;
}

#ride{
    position:absolute;
    right:25px;
    bottom:145px;
    width:72px;
    height:72px;
    border-radius:50%;
    background:#159dd5;
}

#jump{
    position:absolute;
    right:25px;
    bottom:55px;
    width:72px;
    height:72px;
    border-radius:50%;
    background:#10b978;
}

#joystick{
    position:absolute;
    left:25px;
    bottom:42px;
    width:125px;
    height:125px;
    border-radius:50%;
    background:rgba(255,255,255,.25);
    border:2px solid rgba(255,255,255,.5);
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
    background:rgba(255,255,255,.75);
}

#title{
    position:absolute;
    top:18px;
    left:50%;
    transform:translateX(-50%);
    color:white;
    font-weight:bold;
    text-shadow:0 2px 5px #16719a;
}
</style>
</head>

<body>

<div id="ui">

<div id="title">FRUTIGER AERO WORLD</div>

<button id="shift">SHIFT LOCK</button>
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

scene.background = new THREE.Color(0x72dcff);

scene.fog = new THREE.Fog(
    0x72dcff,
    180,
    500
);


/* =========================================================
   CAMERA
========================================================= */

const camera = new THREE.PerspectiveCamera(
    65,
    innerWidth / innerHeight,
    .1,
    700
);

camera.position.set(0,8,14);


/* =========================================================
   RENDERER
========================================================= */

const renderer = new THREE.WebGLRenderer({
    antialias:true,
    powerPreference:"high-performance"
});

renderer.setSize(
    innerWidth,
    innerHeight
);

renderer.setPixelRatio(
    Math.min(devicePixelRatio,1.25)
);

renderer.outputColorSpace =
    THREE.SRGBColorSpace;

renderer.shadowMap.enabled=true;
renderer.shadowMap.type=
    THREE.PCFSoftShadowMap;

document.body.appendChild(
    renderer.domElement
);


/* =========================================================
   LIGHT
========================================================= */

scene.add(
    new THREE.HemisphereLight(
        0xd7faff,
        0x477a36,
        2.4
    )
);

const sun =
    new THREE.DirectionalLight(
        0xffffff,
        3
    );

sun.position.set(
    -100,
    150,
    80
);

sun.castShadow=true;

sun.shadow.mapSize.width=1024;
sun.shadow.mapSize.height=1024;

sun.shadow.camera.left=-220;
sun.shadow.camera.right=220;
sun.shadow.camera.top=220;
sun.shadow.camera.bottom=-220;

scene.add(sun);


/* =========================================================
   WATER
========================================================= */

const water = new THREE.Mesh(
    new THREE.PlaneGeometry(
        700,
        700
    ),
    new THREE.MeshStandardMaterial({
        color:0x20bce6,
        roughness:.18,
        metalness:.05,
        transparent:true,
        opacity:.8
    })
);

water.rotation.x=-Math.PI/2;

water.position.y=-4;

scene.add(water);


/* =========================================================
   NEW GUARANTEED GROUND
========================================================= */

/*
   THIS IS THE IMPORTANT FIX.

   Instead of relying on a CircleGeometry,
   the island is an enormous SOLID BOX.

   The top is always visible.
*/

const groundBase =
    new THREE.Mesh(
        new THREE.BoxGeometry(
            390,
            4,
            390
        ),
        new THREE.MeshStandardMaterial({
            color:0x4eb957,
            roughness:1
        })
    );

groundBase.position.y=-1.7;

groundBase.receiveShadow=true;

scene.add(groundBase);


/* =========================================================
   GRASS TOP
========================================================= */

const grassTop =
    new THREE.Mesh(
        new THREE.BoxGeometry(
            370,
            .35,
            370
        ),
        new THREE.MeshStandardMaterial({
            color:0x55c95b,
            roughness:1
        })
    );

grassTop.position.y=.15;

grassTop.receiveShadow=true;

scene.add(grassTop);


/* =========================================================
   BEACH
========================================================= */

const beachMat =
    new THREE.MeshStandardMaterial({
        color:0xf4df9b,
        roughness:1
    });

const beach =
    new THREE.Mesh(
        new THREE.CylinderGeometry(
            150,
            150,
            .42,
            96
        ),
        beachMat
    );

beach.position.y=.38;

beach.receiveShadow=true;

scene.add(beach);


/* =========================================================
   INNER GRASS ISLAND
========================================================= */

const innerGrass =
    new THREE.Mesh(
        new THREE.CylinderGeometry(
            130,
            130,
            .48,
            96
        ),
        new THREE.MeshStandardMaterial({
            color:0x4fc85a,
            roughness:1
        })
    );

innerGrass.position.y=.6;

innerGrass.receiveShadow=true;

scene.add(innerGrass);


/* =========================================================
   GRASS PATCHES
========================================================= */

const patchMat =
    new THREE.MeshStandardMaterial({
        color:0x6bd866,
        roughness:1
    });

for(let i=0;i<100;i++){

    const a=Math.random()*Math.PI*2;

    const r=
        15+
        Math.random()*110;

    const patch=
        new THREE.Mesh(
            new THREE.CircleGeometry(
                1.5+
                Math.random()*5,
                12
            ),
            patchMat
        );

    patch.rotation.x=-Math.PI/2;

    patch.position.set(
        Math.cos(a)*r,
        .87,
        Math.sin(a)*r
    );

    scene.add(patch);
}


/* =========================================================
   TREES
========================================================= */

const trunkGeo =
    new THREE.CylinderGeometry(
        .45,
        .7,
        5.5,
        8
    );

const trunkMat =
    new THREE.MeshStandardMaterial({
        color:0x86522e,
        roughness:.9
    });

const leafGeo =
    new THREE.IcosahedronGeometry(
        3,
        1
    );

const leafMat =
    new THREE.MeshStandardMaterial({
        color:0x22a94c,
        roughness:.9
    });

for(let i=0;i<100;i++){

    const a=Math.random()*Math.PI*2;

    const r=
        38+
        Math.random()*83;

    const tree=
        new THREE.Group();

    const trunk=
        new THREE.Mesh(
            trunkGeo,
            trunkMat
        );

    trunk.position.y=3;

    tree.add(trunk);

    const leaves=
        new THREE.Mesh(
            leafGeo,
            leafMat
        );

    leaves.position.y=7;

    leaves.scale.setScalar(
        .8+
        Math.random()*.4
    );

    tree.add(leaves);

    tree.position.set(
        Math.cos(a)*r,
        .8,
        Math.sin(a)*r
    );

    tree.rotation.y=
        Math.random()*Math.PI;

    trunk.castShadow=true;
    leaves.castShadow=true;

    scene.add(tree);
}


/* =========================================================
   PLAYER
========================================================= */

const player =
    new THREE.Group();

scene.add(player);


const blueMat =
    new THREE.MeshStandardMaterial({
        color:0x168dcc,
        roughness:.32
    });

const lightBlueMat =
    new THREE.MeshStandardMaterial({
        color:0x62d8f4,
        roughness:.3
    });

const whiteMat =
    new THREE.MeshStandardMaterial({
        color:0xf4ffff,
        roughness:.25
    });

const darkMat =
    new THREE.MeshStandardMaterial({
        color:0x075d8e,
        roughness:.4
    });


/* BODY */

const body=
    new THREE.Mesh(
        new THREE.SphereGeometry(
            1.2,
            24,
            16
        ),
        blueMat
    );

body.scale.set(
    1,
    1.25,
    .72
);

body.position.y=2.45;

body.castShadow=true;

player.add(body);


/* CHEST */

const chest=
    new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            20,
            14
        ),
        lightBlueMat
    );

chest.scale.set(
    .76,
    .72,
    .3
);

chest.position.set(
    0,
    2.55,
    -.7
);

player.add(chest);


/* HEAD */

const head=
    new THREE.Mesh(
        new THREE.SphereGeometry(
            1.05,
            28,
            20
        ),
        blueMat
    );

head.position.y=4;

head.castShadow=true;

player.add(head);


/* FACE */

const face=
    new THREE.Mesh(
        new THREE.SphereGeometry(
            .78,
            24,
            16
        ),
        whiteMat
    );

face.scale.set(
    .95,
    .72,
    .22
);

face.position.set(
    0,
    3.98,
    -.88
);

player.add(face);


/* EYES */

function makeEye(x){

    const eye=
        new THREE.Mesh(
            new THREE.SphereGeometry(
                .13,
                12,
                8
            ),
            darkMat
        );

    eye.position.set(
        x,
        4.05,
        -1.05
    );

    player.add(eye);
}

makeEye(-.27);
makeEye(.27);


/* ARMS */

function arm(x){

    const a=
        new THREE.Mesh(
            new THREE.CapsuleGeometry(
                .31,
                1.15,
                8,
                12
            ),
            blueMat
        );

    a.position.set(
        x,
        2.4,
        0
    );

    a.rotation.z=
        x<0 ? -.12 : .12;

    a.castShadow=true;

    player.add(a);

    return a;
}

const leftArm=arm(-1.35);
const rightArm=arm(1.35);


/* LEGS */

function leg(x){

    const l=
        new THREE.Mesh(
            new THREE.CapsuleGeometry(
                .36,
                1.15,
                8,
                12
            ),
            darkMat
        );

    l.position.set(
        x,
        .95,
        0
    );

    l.castShadow=true;

    player.add(l);

    return l;
}

const leftLeg=leg(-.52);
const rightLeg=leg(.52);


/* =========================================================
   DOLPHIN
========================================================= */

const dolphin=
    new THREE.Group();

const dolphinMat=
    new THREE.MeshStandardMaterial({
        color:0x48bce2,
        roughness:.3
    });

const dolphinBody=
    new THREE.Mesh(
        new THREE.SphereGeometry(
            2,
            24,
            16
        ),
        dolphinMat
    );

dolphinBody.scale.set(
    1.8,
    .62,
    .8
);

dolphin.add(dolphinBody);


const nose=
    new THREE.Mesh(
        new THREE.ConeGeometry(
            .5,
            1.8,
            16
        ),
        dolphinMat
    );

nose.rotation.z=-Math.PI/2;
nose.position.x=3.2;

dolphin.add(nose);


const tail=
    new THREE.Mesh(
        new THREE.ConeGeometry(
            1.25,
            1.8,
            4
        ),
        dolphinMat
    );

tail.rotation.z=Math.PI/2;
tail.position.x=-3;

dolphin.add(tail);


dolphin.position.set(
    18,
    2,
    -20
);

dolphin.rotation.y=Math.PI/2;

scene.add(dolphin);


/* =========================================================
   CONTROLS
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


/* JOYSTICK */

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

function moveJoystick(e){

    const r=
        joystick.getBoundingClientRect();

    let x=
        e.clientX-
        (r.left+r.width/2);

    let y=
        e.clientY-
        (r.top+r.height/2);

    const max=43;

    const len=
        Math.sqrt(x*x+y*y);

    if(len>max){

        x=x/len*max;
        y=y/len*max;

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
        moveJoystick(e);
    }
);

joystick.addEventListener(
    "pointermove",
    e=>{
        if(joyActive)
            moveJoystick(e);
    }
);

joystick.addEventListener(
    "pointerup",
    ()=>{
        joyActive=false;
        joyX=0;
        joyY=0;
        stick.style.transform=
            "translate(0,0)";
    }
);


/* =========================================================
   CAMERA
========================================================= */

let yaw=0;
let shiftLock=false;
let dragging=false;
let lastX=0;

renderer.domElement.addEventListener(
    "pointerdown",
    e=>{

        if(shiftLock)return;

        dragging=true;
        lastX=e.clientX;
    }
);

renderer.domElement.addEventListener(
    "pointermove",
    e=>{

        if(!dragging)return;
        if(shiftLock)return;

        const dx=
            e.clientX-lastX;

        lastX=e.clientX;

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
   SHIFT LOCK
========================================================= */

document.getElementById("shift")
.addEventListener(
    "pointerdown",
    ()=>{

        shiftLock=!shiftLock;

        document
        .getElementById("shift")
        .classList.toggle(
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
   JUMP
========================================================= */

let jumping=false;
let velocityY=0;

document.getElementById("jump")
.addEventListener(
    "pointerdown",
    ()=>{

        if(!jumping){

            jumping=true;
            velocityY=9;
        }
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

        const d=
            player.position.distanceTo(
                dolphin.position
            );

        if(d<13){

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

let walkTime=0;

function animate(){

    requestAnimationFrame(
        animate
    );

    const dt=
        Math.min(
            clock.getDelta(),
            .033
        );


    /* INPUT */

    let x=0;
    let z=0;

    if(keys["a"]||
       keys["arrowleft"])
        x--;

    if(keys["d"]||
       keys["arrowright"])
        x++;

    if(keys["w"]||
       keys["arrowup"])
        z--;

    if(keys["s"]||
       keys["arrowdown"])
        z++;


    if(
        Math.abs(joyX)>.08||
        Math.abs(joyY)>.08
    ){

        x=joyX;
        z=joyY;
    }


    const length=
        Math.sqrt(
            x*x+z*z
        );

    if(length>1){

        x/=length;
        z/=length;
    }


    const moving=
        length>.08;


    /* DIRECTIONS */

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

    const direction=
        new THREE.Vector3();

    direction.addScaledVector(
        right,
        x
    );

    direction.addScaledVector(
        forward,
        -z
    );


    if(direction.lengthSq()>.001){

        direction.normalize();

        const speed=
            riding ? 14 : 9;

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


        if(shiftLock){

            player.rotation.y=yaw;

        }else{

            player.rotation.y=
                Math.atan2(
                    direction.x,
                    -direction.z
                );
        }
    }


    /* SHIFT LOCK */

    if(shiftLock){

        yaw=
            player.rotation.y;
    }


    /* JUMP */

    if(jumping && !riding){

        velocityY-=22*dt;

        player.position.y+=
            velocityY*dt;

        if(player.position.y<=.82){

            player.position.y=.82;
            velocityY=0;
            jumping=false;
        }

    }else if(!riding){

        player.position.y=.82;
    }


    /* KEEP PLAYER ON WORLD */

    const max=175;

    function clampObject(obj){

        const d=Math.sqrt(
            obj.position.x*
            obj.position.x+
            obj.position.z*
            obj.position.z
        );

        if(d>max){

            const s=max/d;

            obj.position.x*=s;
            obj.position.z*=s;
        }
    }

    if(!riding)
        clampObject(player);

    clampObject(dolphin);


    /* WALK ANIMATION */

    if(moving && !riding){

        walkTime+=dt*10;

        const swing=
            Math.sin(walkTime)*.45;

        leftLeg.rotation.x=swing;
        rightLeg.rotation.x=-swing;

        leftArm.rotation.x=-swing*.8;
        rightArm.rotation.x=swing*.8;

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


    /* DOLPHIN FLOAT */

    dolphin.position.y=
        2+
        Math.sin(
            performance.now()*.002
        )*.3;


    /* CAMERA TARGET */

    const target=
        riding
        ? dolphin.position.clone()
        : player.position.clone();

    target.y+=2;


    const distance=
        shiftLock ? 9 : 10;

    const camTarget=
        target.clone();

    camTarget.x+=
        Math.sin(yaw)*distance;

    camTarget.z+=
        Math.cos(yaw)*distance;

    camTarget.y+=5;


    camera.position.lerp(
        camTarget,
        .12
    );

    camera.lookAt(
        target
    );


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
    .82,
    0
);

animate();


/* RESIZE */

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
