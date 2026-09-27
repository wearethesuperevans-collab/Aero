<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

<title>Frutiger Aero World</title>

<style>
html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#72dcff;
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
    user-select:none;
    -webkit-user-select:none;
}

#title{
    position:absolute;
    top:16px;
    left:50%;
    transform:translateX(-50%);
    color:white;
    font-size:18px;
    font-weight:bold;
    text-shadow:
        0 2px 5px #08719b,
        0 0 10px rgba(255,255,255,.4);
}

#shift{
    position:absolute;
    right:18px;
    top:16px;
    padding:12px 16px;
    border-radius:15px;
    background:rgba(14,119,183,.85);
    box-shadow:0 5px 15px rgba(0,0,0,.2);
}

#shift.active{
    background:#00ad70;
}

#ride{
    position:absolute;
    right:24px;
    bottom:145px;
    width:72px;
    height:72px;
    border-radius:50%;
    background:rgba(12,147,211,.9);
    box-shadow:0 5px 18px rgba(0,0,0,.25);
}

#jump{
    position:absolute;
    right:24px;
    bottom:55px;
    width:72px;
    height:72px;
    border-radius:50%;
    background:rgba(8,184,115,.9);
    box-shadow:0 5px 18px rgba(0,0,0,.25);
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
    left:50%;
    top:50%;
    width:58px;
    height:58px;
    margin:-29px;
    border-radius:50%;
    background:rgba(255,255,255,.72);
    box-shadow:0 4px 12px rgba(0,0,0,.15);
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
   WORLD
========================================================= */

const scene = new THREE.Scene();

scene.background = new THREE.Color(0x78ddff);

scene.fog = new THREE.Fog(
    0x78ddff,
    170,
    520
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
    Math.min(window.devicePixelRatio || 1,1.35)
);

renderer.outputColorSpace =
    THREE.SRGBColorSpace;

renderer.toneMapping =
    THREE.ACESFilmicToneMapping;

renderer.toneMappingExposure=1.15;

renderer.shadowMap.enabled=true;

renderer.shadowMap.type =
    THREE.PCFSoftShadowMap;

document.body.appendChild(renderer.domElement);


/* =========================================================
   LIGHTING
========================================================= */

const skyLight =
    new THREE.HemisphereLight(
        0xcff8ff,
        0x315f35,
        2.4
    );

scene.add(skyLight);


const sun =
    new THREE.DirectionalLight(
        0xffffff,
        3.5
    );

sun.position.set(
    -100,
    150,
    80
);

sun.castShadow=true;

sun.shadow.mapSize.width=1536;
sun.shadow.mapSize.height=1536;

sun.shadow.camera.left=-210;
sun.shadow.camera.right=210;
sun.shadow.camera.top=210;
sun.shadow.camera.bottom=-210;

sun.shadow.camera.near=1;
sun.shadow.camera.far=450;

sun.shadow.bias=-.00015;

scene.add(sun);


/* =========================================================
   WATER
========================================================= */

const waterMaterial =
    new THREE.MeshPhysicalMaterial({

        color:0x18bce5,

        roughness:.12,

        metalness:.05,

        transmission:.05,

        transparent:true,

        opacity:.82,

        clearcoat:.7,

        clearcoatRoughness:.12
    });


const water =
    new THREE.Mesh(
        new THREE.PlaneGeometry(
            700,
            700
        ),
        waterMaterial
    );

water.rotation.x=-Math.PI/2;

water.position.y=-4;

water.receiveShadow=true;

scene.add(water);


/* =========================================================
   SOLID ISLAND BASE
   THIS PREVENTS THE INVISIBLE-GROUND BUG
========================================================= */

const islandBase =
    new THREE.Mesh(
        new THREE.BoxGeometry(
            390,
            4,
            390
        ),
        new THREE.MeshStandardMaterial({

            color:0x429f4c,

            roughness:.95,

            metalness:0
        })
    );

islandBase.position.y=-1.7;

islandBase.receiveShadow=true;

scene.add(islandBase);


/* =========================================================
   GRASS SURFACE
========================================================= */

const grass =
    new THREE.Mesh(
        new THREE.BoxGeometry(
            370,
            .35,
            370
        ),
        new THREE.MeshStandardMaterial({

            color:0x51c75a,

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

            color:0xf3df9b,

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

            color:0x4fc85b,

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

        color:0x91c8d0,

        roughness:.82,

        metalness:.02
    });

for(let i=0;i<55;i++){

    const angle=
        Math.random()*Math.PI*2;

    const radius=
        115+
        Math.random()*35;

    const rock =
        new THREE.Mesh(
            new THREE.IcosahedronGeometry(
                .8+
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

    rock.scale.y=
        .5+
        Math.random()*.6;

    rock.rotation.set(
        Math.random(),
        Math.random(),
        Math.random()
    );

    rock.castShadow=true;

    rock.receiveShadow=true;

    scene.add(rock);
}


/* =========================================================
   TREES
========================================================= */

const trunkGeometry =
    new THREE.CylinderGeometry(
        .42,
        .65,
        5.5,
        8
    );

const trunkMaterial =
    new THREE.MeshStandardMaterial({

        color:0x79502e,

        roughness:.9
    });

const leafGeometry =
    new THREE.IcosahedronGeometry(
        3.2,
        1
    );

const leafMaterial =
    new THREE.MeshStandardMaterial({

        color:0x1da84a,

        roughness:.8
    });


for(let i=0;i<105;i++){

    const angle=
        Math.random()*Math.PI*2;

    const radius=
        35+
        Math.random()*90;

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


    const leaves =
        new THREE.Mesh(
            leafGeometry,
            leafMaterial
        );

    leaves.position.y=7;

    leaves.scale.setScalar(
        .85+
        Math.random()*.35
    );

    leaves.castShadow=true;

    tree.add(leaves);


    tree.position.set(

        Math.cos(angle)*radius,

        .8,

        Math.sin(angle)*radius
    );

    tree.rotation.y=
        Math.random()*Math.PI;

    scene.add(tree);
}


/* =========================================================
   SMALL PLANTS
========================================================= */

const plantMaterial =
    new THREE.MeshStandardMaterial({
        color:0x198f42,
        roughness:.9
    });

for(let i=0;i<130;i++){

    const angle=
        Math.random()*Math.PI*2;

    const radius=
        20+
        Math.random()*110;

    const plant =
        new THREE.Mesh(
            new THREE.ConeGeometry(
                .25+
                Math.random()*.2,

                .8+
                Math.random()*.7,

                5
            ),
            plantMaterial
        );

    plant.position.set(

        Math.cos(angle)*radius,

        1,

        Math.sin(angle)*radius
    );

    scene.add(plant);
}


/* =========================================================
   CHARACTER
   BLUE VERSION OF THE REFERENCE DESIGN
========================================================= */

const player =
    new THREE.Group();

scene.add(player);


/*
   Main blue suit
*/

const suitBlue =
    new THREE.MeshStandardMaterial({

        color:0x087fc5,

        roughness:.25,

        metalness:.15,

        clearcoat:.5,

        clearcoatRoughness:.18
    });


const suitLight =
    new THREE.MeshStandardMaterial({

        color:0x43c8f0,

        roughness:.22,

        metalness:.12,

        clearcoat:.6
    });


const suitDark =
    new THREE.MeshStandardMaterial({

        color:0x04527e,

        roughness:.3,

        metalness:.12
    });


const visorWhite =
    new THREE.MeshStandardMaterial({

        color:0xf4ffff,

        roughness:.16,

        metalness:.05,

        clearcoat:.8
    });


const visorBlue =
    new THREE.MeshStandardMaterial({

        color:0x9cecff,

        roughness:.12,

        metalness:.2,

        transparent:true,

        opacity:.82,

        clearcoat:1
    });


/* =========================================================
   BODY ARMOR
========================================================= */

const torso =
    new THREE.Mesh(
        new THREE.SphereGeometry(
            1.25,
            28,
            20
        ),
        suitBlue
    );

torso.scale.set(
    1,
    1.25,
    .74
);

torso.position.y=2.45;

torso.castShadow=true;

player.add(torso);


/* center armor */

const centerArmor =
    new THREE.Mesh(
        new THREE.SphereGeometry(
            1,
            24,
            16
        ),
        suitLight
    );

centerArmor.scale.set(
    .75,
    .82,
    .3
);

centerArmor.position.set(
    0,
    2.52,
    -.78
);

centerArmor.castShadow=true;

player.add(centerArmor);


/* =========================================================
   HEAD
========================================================= */

const head =
    new THREE.Mesh(
        new THREE.SphereGeometry(
            1.05,
            32,
            24
        ),
        suitBlue
    );

head.position.y=4.02;

head.castShadow=true;

player.add(head);


/* face/visor */

const visor =
    new THREE.Mesh(
        new THREE.SphereGeometry(
            .78,
            28,
            20
        ),
        visorWhite
    );

visor.scale.set(
    .96,
    .7,
    .2
);

visor.position.set(
    0,
    4,
    -.91
);

player.add(visor);


/* blue visor center */

const visorGlass =
    new THREE.Mesh(
        new THREE.SphereGeometry(
            .54,
            24,
            16
        ),
        visorBlue
    );

visorGlass.scale.set(
    1,
    .55,
    .13
);

visorGlass.position.set(
    0,
    4,
    -1.05
);

player.add(visorGlass);


/* =========================================================
   SHOULDER ARMOR
========================================================= */

function shoulder(x){

    const s =
        new THREE.Mesh(
            new THREE.SphereGeometry(
                .52,
                20,
                14
            ),
            suitLight
        );

    s.scale.set(
        1.25,
        .75,
        1
    );

    s.position.set(
        x,
        2.9,
        0
    );

    s.castShadow=true;

    player.add(s);

    return s;
}

shoulder(-1.25);
shoulder(1.25);


/* =========================================================
   ARMS
========================================================= */

function makeArm(x){

    const group=
        new THREE.Group();

    group.position.set(
        x,
        2.35,
        0
    );

    const upper =
        new THREE.Mesh(
            new THREE.CapsuleGeometry(
                .32,
                1.05,
                8,
                14
            ),
            suitBlue
        );

    upper.castShadow=true;

    group.add(upper);


    const glove =
        new THREE.Mesh(
            new THREE.SphereGeometry(
                .38,
                18,
                14
            ),
            suitDark
        );

    glove.position.y=-.75;

    glove.castShadow=true;

    group.add(glove);

    player.add(group);

    return group;
}

const leftArm=makeArm(-1.4);
const rightArm=makeArm(1.4);


/* =========================================================
   LEGS
========================================================= */

function makeLeg(x){

    const group=
        new THREE.Group();

    group.position.set(
        x,
        1,
        0
    );

    const leg =
        new THREE.Mesh(
            new THREE.CapsuleGeometry(
                .37,
                1.2,
                8,
                14
            ),
            suitDark
        );

    leg.castShadow=true;

    group.add(leg);


    const boot =
        new THREE.Mesh(
            new THREE.SphereGeometry(
                .43,
                18,
                12
            ),
            suitBlue
        );

    boot.scale.set(
        1.15,
        .65,
        1.35
    );

    boot.position.set(
        0,
        -.8,
        -.12
    );

    boot.castShadow=true;

    group.add(boot);

    player.add(group);

    return group;
}

const leftLeg=makeLeg(-.52);
const rightLeg=makeLeg(.52);


/* =========================================================
   DOLPHIN
========================================================= */

const dolphin =
    new THREE.Group();

const dolphinMaterial =
    new THREE.MeshPhysicalMaterial({

        color:0x43b9df,

        roughness:.22,

        metalness:.08,

        clearcoat:.8,

        clearcoatRoughness:.12
    });


const dolphinBody =
    new THREE.Mesh(
        new THREE.SphereGeometry(
            2,
            28,
            20
        ),
        dolphinMaterial
    );

dolphinBody.scale.set(
    1.8,
    .62,
    .8
);

dolphinBody.castShadow=true;

dolphin.add(dolphinBody);


/* nose */

const dolphinNose =
    new THREE.Mesh(
        new THREE.ConeGeometry(
            .5,
            1.8,
            18
        ),
        dolphinMaterial
    );

dolphinNose.rotation.z=
    -Math.PI/2;

dolphinNose.position.x=3.2;

dolphin.add(dolphinNose);


/* tail */

const dolphinTail =
    new THREE.Mesh(
        new THREE.ConeGeometry(
            1.2,
            1.8,
            4
        ),
        dolphinMaterial
    );

dolphinTail.rotation.z=
    Math.PI/2;

dolphinTail.position.x=-3;

dolphin.add(dolphinTail);


/* fin */

const dolphinFin =
    new THREE.Mesh(
        new THREE.ConeGeometry(
            .7,
            1.7,
            4
        ),
        dolphinMaterial
    );

dolphinFin.position.y=1.1;

dolphinFin.rotation.z=Math.PI;

dolphin.add(dolphinFin);


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

    const length=
        Math.sqrt(x*x+y*y);

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

        if(shiftLock)
            return;

        dragging=true;

        lastX=e.clientX;
    }
);

renderer.domElement.addEventListener(
    "pointermove",
    e=>{

        if(!dragging)
            return;

        if(shiftLock)
            return;

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

let verticalVelocity=0;

document.getElementById("jump")
.addEventListener(
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
   RIDE
========================================================= */

let riding=false;

document.getElementById("ride")
.addEventListener(
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

    let moveX=0;
    let moveY=0;


    if(
        keys["a"] ||
        keys["arrowleft"]
    )
        moveX=-1;


    if(
        keys["d"] ||
        keys["arrowright"]
    )
        moveX=1;


    if(
        keys["w"] ||
        keys["arrowup"]
    )
        moveY=-1;


    if(
        keys["s"] ||
        keys["arrowdown"]
    )
        moveY=1;


    if(
        Math.abs(joyX)>.08 ||
        Math.abs(joyY)>.08
    ){

        moveX=joyX;
        moveY=joyY;
    }


    const inputLength=
        Math.sqrt(
            moveX*moveX+
            moveY*moveY
        );

    if(inputLength>1){

        moveX/=inputLength;
        moveY/=inputLength;
    }


    const moving=
        inputLength>.08;


    /* CAMERA DIRECTIONS */

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
        moveX
    );

    direction.addScaledVector(
        forward,
        -moveY
    );


    /* MOVEMENT */

    if(
        direction.lengthSq()>.001
    ){

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

            player.rotation.y=
                yaw;

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


    /* WORLD LIMIT */

    const limit=172;

    function limitObject(obj){

        const distance=
            Math.sqrt(
                obj.position.x*
                obj.position.x+
                obj.position.z*
                obj.position.z
            );

        if(distance>limit){

            const scale=
                limit/distance;

            obj.position.x*=scale;
            obj.position.z*=scale;
        }
    }

    limitObject(player);
    limitObject(dolphin);


    /* WALK ANIMATION */

    if(
        moving &&
        !riding
    ){

        walkTime+=
            dt*10;

        const swing=
            Math.sin(
                walkTime
            )*.45;


        leftLeg.rotation.x=
            swing;

        rightLeg.rotation.x=
            -swing;


        leftArm.rotation.x=
            -swing*.8;

        rightArm.rotation.x=
            swing*.8;

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


    /* DOLPHIN BOB */

    dolphin.position.y=
        2+
        Math.sin(
            performance.now()*.002
        )*.3;


    /* CAMERA */

    const target=
        riding
        ? dolphin.position.clone()
        : player.position.clone();

    target.y+=2;


    const distance=
        shiftLock ? 9 : 10;


    const desiredCamera=
        target.clone();


    desiredCamera.x+=
        Math.sin(yaw)*
        distance;

    desiredCamera.z+=
        Math.cos(yaw)*
        distance;

    desiredCamera.y+=5;


    camera.position.lerp(
        desiredCamera,
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
   INITIAL POSITION
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


/* =========================================================
   RESIZE
========================================================= */

addEventListener(
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


/* =========================================================
   START
========================================================= */

animate();

</script>

</body>
</html>
