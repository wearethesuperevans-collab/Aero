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
    background:#55dcff;
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

#title{
    position:absolute;
    top:17px;
    left:20px;
    color:white;
    font-size:23px;
    font-weight:900;
    text-shadow:
        0 2px 5px rgba(0,70,120,.8),
        0 0 15px rgba(255,255,255,.4);
}

#subtitle{
    position:absolute;
    top:48px;
    left:21px;
    color:white;
    font-size:12px;
    text-shadow:0 2px 5px rgba(0,70,120,.8);
}

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
        0 10px 30px rgba(0,70,120,.3),
        inset 0 0 25px rgba(255,255,255,.3);
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
    background:linear-gradient(
        145deg,
        #ffffff,
        #b9f5ff
    );
    border:3px solid white;
    box-shadow:
        0 5px 18px rgba(0,70,120,.3),
        inset 0 0 10px white;
}

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
            #a2f8ff,
            #18a9d9
        );
    box-shadow:
        0 8px 24px rgba(0,70,120,.32),
        inset 0 3px 9px rgba(255,255,255,.6);
    pointer-events:auto;
    touch-action:none;
}

button:active{
    transform:scale(.9);
}

#shiftButton.active{
    background:
        linear-gradient(
            #fff37b,
            #ffad22
        );
    color:#614300;
}

#rotate{
    display:none;
    position:fixed;
    inset:0;
    z-index:100;
    background:
        linear-gradient(
            #54e2ff,
            #d2fcff
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
        Explore the island • Beaches • Grasslands • Forest
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

    <div id="rotateIcon">↔️</div>

    <div>TURN YOUR iPAD SIDEWAYS</div>

    <small>
        Landscape mode gives you the full world.
    </small>

</div>


<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

/* ============================================================
   FRUTIGER AERO WORLD
   LARGE SINGLE ISLAND
   ============================================================ */


/* ============================================================
   RENDERER
   ============================================================ */

const canvas =
document.getElementById("game");

const renderer =
new THREE.WebGLRenderer({
    canvas:canvas,
    antialias:true,
    powerPreference:"high-performance"
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

renderer.toneMappingExposure=1.15;

renderer.shadowMap.enabled=true;

renderer.shadowMap.type =
THREE.PCFSoftShadowMap;


/* ============================================================
   SCENE
   ============================================================ */

const scene =
new THREE.Scene();

scene.background =
new THREE.Color(
    0x61ddff
);

scene.fog =
new THREE.Fog(
    0x72e5ff,
    120,
    700
);


/* ============================================================
   CAMERA
   ============================================================ */

const camera =
new THREE.PerspectiveCamera(
    67,
    innerWidth/innerHeight,
    .1,
    1200
);

camera.position.set(
    0,
    7,
    17
);


/* ============================================================
   LIGHTING
   ============================================================ */

const skyLight =
new THREE.HemisphereLight(
    0xeaffff,
    0x276e43,
    2.8
);

scene.add(skyLight);


const sun =
new THREE.DirectionalLight(
    0xffffff,
    4.5
);

sun.position.set(
    -180,
    230,
    150
);

sun.castShadow=true;

/* HIGH QUALITY SHADOWS */

sun.shadow.mapSize.width=4096;
sun.shadow.mapSize.height=4096;

sun.shadow.camera.left=-260;
sun.shadow.camera.right=260;
sun.shadow.camera.top=260;
sun.shadow.camera.bottom=-260;

sun.shadow.camera.near=1;
sun.shadow.camera.far=650;

sun.shadow.bias=-0.00015;

sun.shadow.normalBias=.015;

scene.add(sun);


/* ============================================================
   MATERIALS
   ============================================================ */

const beachMaterial =
new THREE.MeshPhysicalMaterial({
    color:0xf4e5a5,
    roughness:.9,
    clearcoat:.15
});

const grassMaterial =
new THREE.MeshPhysicalMaterial({
    color:0x42ce61,
    roughness:.78,
    clearcoat:.25,
    clearcoatRoughness:.4
});

const forestGrassMaterial =
new THREE.MeshPhysicalMaterial({
    color:0x239d50,
    roughness:.84,
    clearcoat:.2
});

const rockMaterial =
new THREE.MeshStandardMaterial({
    color:0x55955a,
    roughness:.95
});

const waterMaterial =
new THREE.MeshPhysicalMaterial({
    color:0x13c4ea,
    roughness:.1,
    metalness:.05,
    transparent:true,
    opacity:.92,
    clearcoat:1,
    clearcoatRoughness:.05,
    depthWrite:false
});

const trunkMaterial =
new THREE.MeshStandardMaterial({
    color:0xb97943,
    roughness:.9
});

const leafMaterial =
new THREE.MeshPhysicalMaterial({
    color:0x14984d,
    roughness:.6,
    clearcoat:.3,
    clearcoatRoughness:.3
});

const leafLightMaterial =
new THREE.MeshPhysicalMaterial({
    color:0x2acb61,
    roughness:.58,
    clearcoat:.3
});


/* ============================================================
   LARGE OCEAN
   ============================================================ */

const water =
new THREE.Mesh(
    new THREE.PlaneGeometry(
        1600,
        1600,
        64,
        64
    ),
    waterMaterial
);

water.rotation.x=
-Math.PI/2;

water.position.y=-1.8;

water.receiveShadow=true;

scene.add(water);


/* ============================================================
   SINGLE HUGE ISLAND TERRAIN
   ============================================================

   IMPORTANT:

   There is ONLY ONE terrain mesh.

   No overlapping cylinders.
   No stacked ground pieces.
   No coplanar surfaces.

   This prevents the old ground flickering/
   texture-glitch effect.
   ============================================================ */

const GRID=170;
const HALF=210;

const vertices=[];
const indices=[];
const zoneGroups={
    beach:[],
    grass:[],
    forest:[]
};


/* deterministic terrain noise */

function noise2(x,z){

    return (
        Math.sin(x*.035)*
        .9+
        Math.sin(z*.047)*
        .7+
        Math.sin((x+z)*.021)*
        1.1+
        Math.cos((x-z)*.031)*
        .6
    );
}


function terrainHeight(x,z){

    const r=
        Math.sqrt(
            x*x+
            z*z
        );

    /*
       Island radius ~190.
    */

    const edge=
        THREE.MathUtils.clamp(
            (r-135)/60,
            0,
            1
        );

    /*
       Soft island edge.
    */

    const base=
        2.5+
        Math.max(
            0,
            3.5*(1-edge)
        );

    const hills=
        noise2(x,z)*.8;

    /*
       Beach stays low.
    */

    if(r>132){

        const beachBlend=
            THREE.MathUtils.clamp(
                (r-132)/48,
                0,
                1
            );

        return (
            base+
            hills*.25-
            beachBlend*2.5
        );
    }

    return base+hills;
}


/* CREATE GRID */

for(let z=0;z<=GRID;z++){

    for(let x=0;x<=GRID;x++){

        const px=
            -HALF+
            (x/GRID)*
            HALF*2;

        const pz=
            -HALF+
            (z/GRID)*
            HALF*2;

        const radius=
            Math.sqrt(
                px*px+
                pz*pz
            );

        /*
           Outside island:
           force terrain far below ocean.
        */

        let py=
            terrainHeight(
                px,
                pz
            );

        if(radius>188){

            const drop=
                Math.min(
                    35,
                    (radius-188)*2
                );

            py-=drop;
        }

        vertices.push(
            px,
            py,
            pz
        );
    }
}


/* CREATE TRIANGLES */

for(let z=0;z<GRID;z++){

    for(let x=0;x<GRID;x++){

        const a=
            z*(GRID+1)+x;

        const b=a+1;

        const c=
            a+(GRID+1);

        const d=c+1;


        /*
           Alternate diagonal directions
           to reduce visible grid patterns.
        */

        if((x+z)%2===0){

            indices.push(
                a,b,d,
                a,d,c
            );

        }else{

            indices.push(
                a,b,c,
                b,d,c
            );
        }
    }
}


const terrainGeometry =
new THREE.BufferGeometry();

terrainGeometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(
        vertices,
        3
    )
);

terrainGeometry.setIndex(
    indices
);

terrainGeometry.computeVertexNormals();


/* ============================================================
   TERRAIN ZONE MATERIALS
   ============================================================ */

terrainGeometry.clearGroups();

let triangleCounter=0;


for(let z=0;z<GRID;z++){

    for(let x=0;x<GRID;x++){

        const px=
            -HALF+
            (x/GRID)*
            HALF*2;

        const pz=
            -HALF+
            (z/GRID)*
            HALF*2;

        const radius=
            Math.sqrt(
                px*px+
                pz*pz
            );

        /*
           Beach:
           outer ring

           Grassland:
           middle ring

           Forest:
           northern/western portion
           of the huge island
        */

        let materialIndex=1;

        if(radius>132){

            materialIndex=0;

        }else{

            /*
               Forest zone is mostly
               toward the north-west.
            */

            const forestDirection=
                pz < 20 &&
                px < 75;

            const forestRadius=
                radius>65;

            if(
                forestDirection &&
                forestRadius
            ){

                materialIndex=2;
            }
        }


        terrainGeometry.addGroup(
            triangleCounter*4,
            6,
            materialIndex
        );

        triangleCounter++;
    }
}


const terrain =
new THREE.Mesh(
    terrainGeometry,
    [
        beachMaterial,
        grassMaterial,
        forestGrassMaterial
    ]
);

terrain.castShadow=true;

terrain.receiveShadow=true;

scene.add(terrain);


/* ============================================================
   BEACH DETAILS
   ============================================================ */

function createBeachRock(
    x,
    z,
    scale
){

    const rock =
    new THREE.Mesh(
        new THREE.IcosahedronGeometry(
            1,
            2
        ),
        new THREE.MeshStandardMaterial({
            color:0xc8c3a1,
            roughness:.9
        })
    );

    const y=
        terrainHeight(
            x,
            z
        );

    rock.position.set(
        x,
        y+.35*scale,
        z
    );

    rock.scale.set(
        1.2*scale,
        .45*scale,
        .8*scale
    );

    rock.rotation.y=
        Math.random()*Math.PI;

    rock.castShadow=true;

    rock.receiveShadow=true;

    scene.add(rock);
}


/* beach rocks */

for(let i=0;i<65;i++){

    const angle=
        Math.random()*
        Math.PI*2;

    const radius=
        137+
        Math.random()*38;

    const x=
        Math.cos(angle)*radius;

    const z=
        Math.sin(angle)*radius;

    if(
        Math.sqrt(x*x+z*z)<190
    ){

        createBeachRock(
            x,
            z,
            .3+
            Math.random()*.8
        );
    }
}


/* ============================================================
   TREE SYSTEM
   ============================================================ */

function createTree(
    x,
    z,
    scale
){

    const y=
        terrainHeight(
            x,
            z
        );

    const tree =
        new THREE.Group();


    /* trunk */

    const trunk =
        new THREE.Mesh(
            new THREE.CylinderGeometry(
                .32*scale,
                .55*scale,
                7*scale,
                20
            ),
            trunkMaterial
        );

    trunk.position.y=
        3.5*scale;

    trunk.castShadow=true;

    trunk.receiveShadow=true;

    tree.add(trunk);


    /* branch */

    const branch =
        new THREE.Mesh(
            new THREE.CylinderGeometry(
                .13*scale,
                .2*scale,
                2.7*scale,
                12
            ),
            trunkMaterial
        );

    branch.position.set(
        -.7*scale,
        4.3*scale,
        0
    );

    branch.rotation.z=
        -.8;

    branch.castShadow=true;

    tree.add(branch);


    /* crown */

    const crown =
        new THREE.Group();

    crown.position.y=
        6.8*scale;

    tree.add(crown);


    for(let i=0;i<11;i++){

        const angle=
            i*Math.PI*2/11;

        const leaf =
            new THREE.Mesh(
                new THREE.SphereGeometry(
                    1,
                    28,
                    20
                ),
                i%3===0
                ? leafLightMaterial
                : leafMaterial
            );

        leaf.scale.set(
            2.2*scale,
            .72*scale,
            1.35*scale
        );

        leaf.position.set(
            Math.cos(angle)*
                1.45*scale,
            (i%3)*.45*scale,
            Math.sin(angle)*
                1.45*scale
        );

        leaf.rotation.y=
            -angle;

        leaf.castShadow=true;

        crown.add(leaf);
    }


    /*
       Extra center foliage.
    */

    const center =
        new THREE.Mesh(
            new THREE.SphereGeometry(
                1.7*scale,
                32,
                24
            ),
            leafMaterial
        );

    center.position.y=
        6.9*scale;

    center.castShadow=true;

    tree.add(center);


    tree.position.set(
        x,
        y,
        z
    );

    tree.rotation.y=
        Math.random()*
        Math.PI*2;

    scene.add(tree);
}


/* ============================================================
   FOREST

   Dense forest in one region.
   ============================================================ */

for(let i=0;i<210;i++){

    const x=
        -105+
        Math.random()*180;

    const z=
        -120+
        Math.random()*125;

    const radius=
        Math.sqrt(
            x*x+
            z*z
        );

    /*
       Keep trees inside the island
       and in the forest region.
    */

    if(
        radius>68 &&
        radius<130 &&
        z<20 &&
        x<75
    ){

        createTree(
            x,
            z,
            .75+
            Math.random()*.65
        );
    }
}


/* ============================================================
   GRASSLAND TREES
   ============================================================ */

for(let i=0;i<45;i++){

    const angle=
        Math.random()*
        Math.PI*2;

    const radius=
        45+
        Math.random()*65;

    const x=
        Math.cos(angle)*radius;

    const z=
        Math.sin(angle)*radius;

    if(
        !(z<20 &&
          x<75 &&
          radius>68)
    ){

        createTree(
            x,
            z,
            .65+
            Math.random()*.4
        );
    }
}


/* ============================================================
   GRASS FLOWERS
   ============================================================ */

function createFlower(
    x,
    z
){

    const group=
        new THREE.Group();

    const stem =
        new THREE.Mesh(
            new THREE.CylinderGeometry(
                .025,
                .04,
                .35,
                8
            ),
            new THREE.MeshStandardMaterial({
                color:0x38a94e
            })
        );

    stem.position.y=.18;

    group.add(stem);


    const flower =
        new THREE.Mesh(
            new THREE.SphereGeometry(
                .12,
                16,
                12
            ),
            new THREE.MeshPhysicalMaterial({
                color:
                    Math.random()>.5
                    ?0xffffff
                    :0xfff04c,
                roughness:.4,
                clearcoat:.4
            })
        );

    flower.position.y=.4;

    group.add(flower);


    const y=
        terrainHeight(
            x,
            z
        );

    group.position.set(
        x,
        y,
        z
    );

    scene.add(group);
}


for(let i=0;i<280;i++){

    const angle=
        Math.random()*
        Math.PI*2;

    const radius=
        20+
        Math.random()*105;

    const x=
        Math.cos(angle)*radius;

    const z=
        Math.sin(angle)*radius;

    if(
        Math.sqrt(x*x+z*z)<125
    ){

        createFlower(
            x,
            z
        );
    }
}


/* ============================================================
   CLOUDS
   ============================================================ */

function createCloud(
    x,
    y,
    z,
    scale
){

    const cloud =
        new THREE.Group();


    for(let i=0;i<12;i++){

        const puff =
            new THREE.Mesh(

                new THREE.SphereGeometry(
                    1,
                    32,
                    24
                ),

                new THREE.MeshPhysicalMaterial({

                    color:0xffffff,

                    roughness:.3,

                    clearcoat:.75,

                    clearcoatRoughness:.1
                })
            );

        puff.scale.setScalar(
            (.8+
            Math.random()*.9)*
            scale
        );

        puff.position.set(
            (i-5.5)*
                1.4*
                scale,

            Math.random()*
                1.2*
                scale,

            Math.random()*
                .8*
                scale
        );

        cloud.add(puff);
    }


    cloud.position.set(
        x,y,z
    );

    scene.add(cloud);
}


createCloud(
    -120,
    48,
    -170,
    2.5
);

createCloud(
    100,
    54,
    -230,
    3
);

createCloud(
    220,
    44,
    30,
    2
);

createCloud(
    -230,
    48,
    140,
    2.5
);


/* ============================================================
   AERO CHARACTER
   ============================================================ */

function makeAeroGuy(){

    const guy =
        new THREE.Group();


    /*
       TORSO
    */

    const torso =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                1,
                64,
                48
            ),

            new THREE.MeshPhysicalMaterial({

                color:0x32caff,

                roughness:.12,

                metalness:.02,

                transparent:true,

                opacity:.94,

                clearcoat:1,

                clearcoatRoughness:.025
            })
        );

    torso.scale.set(
        1.27,
        1.43,
        .92
    );

    torso.position.y=
        1.75;

    torso.castShadow=true;

    torso.receiveShadow=true;

    guy.add(torso);


    /*
       INNER TORSO
    */

    const innerTorso =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                .98,
                48,
                36
            ),

            new THREE.MeshPhysicalMaterial({

                color:0x168ee0,

                roughness:.13,

                transparent:true,

                opacity:.4,

                clearcoat:1
            })
        );

    innerTorso.scale.set(
        1.18,
        1.34,
        .87
    );

    innerTorso.position.y=
        1.72;

    guy.add(innerTorso);


    /*
       HEAD
    */

    const head =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                1.05,
                64,
                48
            ),

            new THREE.MeshPhysicalMaterial({

                color:0x38cfff,

                roughness:.1,

                metalness:.02,

                transparent:true,

                opacity:.95,

                clearcoat:1,

                clearcoatRoughness:.02
            })
        );

    head.position.y=
        3.72;

    head.castShadow=true;

    head.receiveShadow=true;

    guy.add(head);


    /*
       HEAD INNER COLOR
    */

    const headInner =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                .98,
                48,
                36
            ),

            new THREE.MeshPhysicalMaterial({

                color:0x1597df,

                roughness:.12,

                transparent:true,

                opacity:.3,

                clearcoat:1
            })
        );

    headInner.position.y=
        3.70;

    guy.add(headInner);


    /*
       HEAD GLOSS
    */

    const headGloss =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                .36,
                32,
                24
            ),

            new THREE.MeshBasicMaterial({
                color:0xffffff,
                transparent:true,
                opacity:.78
            })
        );

    headGloss.scale.set(
        1.5,
        .52,
        .42
    );

    headGloss.position.set(
        -.32,
        4.15,
        -.84
    );

    guy.add(headGloss);


    /*
       BODY GLOSS
    */

    const bodyGloss =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                .42,
                32,
                24
            ),

            new THREE.MeshBasicMaterial({
                color:0xffffff,
                transparent:true,
                opacity:.42
            })
        );

    bodyGloss.scale.set(
        1.45,
        .65,
        .3
    );

    bodyGloss.position.set(
        -.62,
        2.28,
        -.8
    );

    guy.add(bodyGloss);


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

            new THREE.MeshPhysicalMaterial({

                color:0x32caff,

                roughness:.1,

                transparent:true,

                opacity:.94,

                clearcoat:1
            })
        );

    leftArm.scale.set(
        .42,
        .98,
        .48
    );

    leftArm.position.set(
        -1.32,
        1.48,
        0
    );

    leftArm.rotation.z=.16;

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

            new THREE.MeshPhysicalMaterial({

                color:0x32caff,

                roughness:.1,

                transparent:true,

                opacity:.94,

                clearcoat:1
            })
        );

    rightArm.scale.set(
        .42,
        .98,
        .48
    );

    rightArm.position.set(
        1.32,
        1.48,
        0
    );

    rightArm.rotation.z=-.16;

    rightArm.castShadow=true;

    guy.add(rightArm);


    /*
       LEFT LEG
    */

    const leftLeg =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                1,
                48,
                36
            ),

            new THREE.MeshPhysicalMaterial({

                color:0x32caff,

                roughness:.1,

                transparent:true,

                opacity:.95,

                clearcoat:1
            })
        );

    leftLeg.scale.set(
        .50,
        .84,
        .56
    );

    leftLeg.position.set(
        -.55,
        .18,
        0
    );

    leftLeg.castShadow=true;

    guy.add(leftLeg);


    /*
       RIGHT LEG
    */

    const rightLeg =
        new THREE.Mesh(

            new THREE.SphereGeometry(
                1,
                48,
                36
            ),

            new THREE.MeshPhysicalMaterial({

                color:0x32caff,

                roughness:.1,

                transparent:true,

                opacity:.95,

                clearcoat:1
            })
        );

    rightLeg.scale.set(
        .50,
        .84,
        .56
    );

    rightLeg.position.set(
        .55,
        .18,
        0
    );

    rightLeg.castShadow=true;

    guy.add(rightLeg);


    /*
       STORE ANIMATION PARTS
    */

    guy.userData.animation={

        leftArm:leftArm,

        rightArm:rightArm,

        leftLeg:leftLeg,

        rightLeg:rightLeg,

        torso:torso,

        head:head
    };


    return guy;
}


/* ============================================================
   PLAYER
   ============================================================ */

const player =
    makeAeroGuy();

/*
   Feet bottom ≈ -.66.
   Ground ≈ 5 around starting area.
   Player root is adjusted to terrain.
*/

const startX=0;
const startZ=35;

player.position.set(
    startX,
    terrainHeight(
        startX,
        startZ
    )+.67,
    startZ
);

scene.add(player);


/* ============================================================
   NPCS
   ============================================================ */

const npcs=[];

const npcLocations=[

    [-20,20],
    [18,15],
    [-45,35],
    [50,20],
    [-35,-25],
    [40,-35],
    [5,-55],
    [70,-10]
];


npcLocations.forEach(
    p=>{

        const npc=
            makeAeroGuy();

        npc.scale.setScalar(.78);

        npc.position.set(
            p[0],
            terrainHeight(
                p[0],
                p[1]
            )+.67,
            p[1]
        );

        scene.add(npc);

        npcs.push(npc);
    }
);


/* ============================================================
   DOLPHIN
   ============================================================ */

function makeDolphin(){

    const dolphin=
        new THREE.Group();


    const material=
        new THREE.MeshPhysicalMaterial({

            color:0x48c8eb,

            roughness:.11,

            metalness:.07,

            clearcoat:1,

            clearcoatRoughness:.03
        });


    const body=
        new THREE.Mesh(

            new THREE.SphereGeometry(
                1,
                64,
                40
            ),

            material
        );

    body.scale.set(
        2.45,
        .72,
        1
    );

    body.castShadow=true;

    dolphin.add(body);


    const nose=
        new THREE.Mesh(

            new THREE.ConeGeometry(
                .43,
                1.75,
                32
            ),

            material
        );

    nose.rotation.z=
        -Math.PI/2;

    nose.position.z=
        -2.3;

    dolphin.add(nose);


    const dorsal=
        new THREE.Mesh(

            new THREE.ConeGeometry(
                .45,
                1.55,
                24
            ),

            material
        );

    dorsal.rotation.z=
        Math.PI;

    dorsal.position.y=.85;

    dolphin.add(dorsal);


    const tail=
        new THREE.Group();


    for(
        const side of [-1,1]
    ){

        const fin=
            new THREE.Mesh(

                new THREE.ConeGeometry(
                    .5,
                    1.55,
                    24
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


const dolphin=
    makeDolphin();

dolphin.position.set(
    0,
    -.2,
    -145
);

scene.add(dolphin);


/* ============================================================
   BUBBLES
   ============================================================ */

const bubbles=[];

for(let i=0;i<100;i++){

    const bubble=
        new THREE.Mesh(

            new THREE.SphereGeometry(
                .15+
                Math.random()*.38,
                24,
                20
            ),

            new THREE.MeshPhysicalMaterial({

                color:0xeaffff,

                transparent:true,

                opacity:.4,

                roughness:0,

                transmission:.25,

                clearcoat:1
            })
        );

    bubble.position.set(

        (Math.random()-.5)*700,

        Math.random()*16,

        (Math.random()-.5)*700
    );

    scene.add(bubble);

    bubbles.push(bubble);
}


/* ============================================================
   MOVEMENT
   ============================================================ */

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

    const length=
        Math.hypot(x,y);

    if(length>max){

        x=
            x/length*max;

        y=
            y/length*max;
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


/* ============================================================
   CAMERA
   ============================================================ */

let cameraYaw=0;

let cameraPitch=.18;

let cameraDistance=13;

let cameraDragging=false;

let cameraPointer=null;

let previousX=0;
let previousY=0;


/*
   Right-side camera drag.
*/

canvas.addEventListener(
    "pointerdown",
    e=>{

        cameraDragging=true;

        cameraPointer=
            e.pointerId;

        previousX=e.clientX;
        previousY=e.clientY;
    }
);


canvas.addEventListener(
    "pointermove",
    e=>{

        if(
            !cameraDragging ||
            e.pointerId!==cameraPointer
        )
            return;

        const dx=
            e.clientX-
            previousX;

        const dy=
            e.clientY-
            previousY;


        /*
           CAMERA ROTATION ONLY.

           This does NOT rotate the player.

           This is what fixes the old
           Shift Lock spinning problem.
        */

        cameraYaw -=
            dx*.006;


        cameraPitch -=
            dy*.004;


        cameraPitch=
            THREE.MathUtils.clamp(
                cameraPitch,
                -.35,
                .7
            );


        previousX=e.clientX;
        previousY=e.clientY;
    }
);


canvas.addEventListener(
    "pointerup",
    e=>{

        if(
            e.pointerId===
            cameraPointer
        ){

            cameraDragging=false;
            cameraPointer=null;
        }
    }
);


canvas.addEventListener(
    "pointercancel",
    e=>{

        if(
            e.pointerId===
            cameraPointer
        ){

            cameraDragging=false;
            cameraPointer=null;
        }
    }
);


/* ============================================================
   SHIFT LOCK
   ============================================================ */

let shiftLock=false;

const shiftButton=
    document.getElementById(
        "shiftButton"
    );


shiftButton.addEventListener(
    "pointerdown",
    e=>{

        e.stopPropagation();

        shiftLock=
            !shiftLock;

        shiftButton.classList.toggle(
            "active",
            shiftLock
        );
    }
);


/* ============================================================
   JUMP
   ============================================================ */

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
    e=>{

        e.stopPropagation();

        jump();
    }
);


/* ============================================================
   RIDE
   ============================================================ */

let riding=false;


function toggleRide(){

    const distance=
        player.position.distanceTo(
            dolphin.position
        );


    if(!riding){

        if(distance<12){

            riding=true;

            player.position.copy(
                dolphin.position
            );

            player.position.y+=2.2;
        }

    }else{

        riding=false;

        player.position.y=
            terrainHeight(
                player.position.x,
                player.position.z
            )+.67;
    }
}


document
.getElementById(
    "rideButton"
)
.addEventListener(
    "pointerdown",
    e=>{

        e.stopPropagation();

        toggleRide();
    }
);


/* ============================================================
   KEYBOARD
   ============================================================ */

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


/* ============================================================
   CHARACTER ANIMATION
   ============================================================ */

function animateCharacter(
    character,
    speed,
    time,
    dt
){

    const a=
        character.userData.animation;

    if(!a)
        return;


    const moving=
        speed>.05;


    if(moving){

        /*
           Walking/running cycle.
        */

        const cycle=
            time*
            (
                speed>5
                ?10
                :7
            );


        const swing=
            Math.sin(cycle)*
            .65;


        const opposite=
            Math.sin(cycle+
                Math.PI)*
            .65;


        /*
           LEGS
        */

        a.leftLeg.rotation.x=
            swing;

        a.rightLeg.rotation.x=
            opposite;


        /*
           ARMS
        */

        a.leftArm.rotation.x=
            opposite*.7;

        a.rightArm.rotation.x=
            swing*.7;


        /*
           BODY BOB
        */

        character.position.y +=
            Math.sin(cycle*2)*
            .018;


    }else{

        /*
           Smoothly return limbs.
        */

        a.leftLeg.rotation.x=
            THREE.MathUtils.lerp(
                a.leftLeg.rotation.x,
                0,
                .15
            );

        a.rightLeg.rotation.x=
            THREE.MathUtils.lerp(
                a.rightLeg.rotation.x,
                0,
                .15
            );

        a.leftArm.rotation.x=
            THREE.MathUtils.lerp(
                a.leftArm.rotation.x,
                0,
                .15
            );

        a.rightArm.rotation.x=
            THREE.MathUtils.lerp(
                a.rightArm.rotation.x,
                0,
                .15
            );
    }
}


/* ============================================================
   MAIN ANIMATION
   ============================================================ */

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


    /* ========================================================
       KEYBOARD
       ======================================================== */

    let keyboardX=0;
    let keyboardZ=0;


    if(
        keys["a"]||
        keys["arrowleft"]
    )
        keyboardX=-1;


    if(
        keys["d"]||
        keys["arrowright"]
    )
        keyboardX=1;


    if(
        keys["w"]||
        keys["arrowup"]
    )
        keyboardZ=-1;


    if(
        keys["s"]||
        keys["arrowdown"]
    )
        keyboardZ=1;


    if(
        keyboardX!==0||
        keyboardZ!==0
    ){

        moveX=keyboardX;
        moveZ=keyboardZ;

    }


    /* ========================================================
       CAMERA-RELATIVE MOVEMENT
       ======================================================== */

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


    let actualSpeed=0;


    if(
        movement.lengthSq()>0
    ){

        movement.normalize();


        actualSpeed=
            strength*
            (
                riding
                ?20
                :8
            );


        if(riding){

            dolphin.position.addScaledVector(
                movement,
                actualSpeed*dt
            );

        }else{

            player.position.addScaledVector(
                movement,
                actualSpeed*dt
            );
        }


        /* ====================================================
           SHIFT LOCK

           Character faces exactly where
           the camera is facing.

           NO CAMERA-YAW FEEDBACK.
           NO SPINNING.
           ==================================================== */

        if(shiftLock){

            player.rotation.y=
                cameraYaw;

        }else{

            /*
               Normal mode:
               face direction of movement.
            */

            const desiredRotation=
                Math.atan2(
                    movement.x,
                    movement.z
                );

            let difference=
                desiredRotation-
                player.rotation.y;


            while(
                difference>Math.PI
            )
                difference-=Math.PI*2;


            while(
                difference<-Math.PI
            )
                difference+=Math.PI*2;


            player.rotation.y+=
                difference*.14;
        }
    }


    /* ========================================================
       TERRAIN HEIGHT
       ======================================================== */

    if(!riding){

        const groundY=
            terrainHeight(
                player.position.x,
                player.position.z
            )+.67;


        /*
           Keep character above ground
           unless jumping.
        */

        if(!jumping){

            player.position.y=
                THREE.MathUtils.lerp(
                    player.position.y,
                    groundY,
                    .3
                );
        }
    }


    /* ========================================================
       JUMP
       ======================================================== */

    if(jumping){

        verticalVelocity-=25*dt;


        if(riding){

            dolphin.position.y+=
                verticalVelocity*dt;


            if(
                dolphin.position.y<=-.2
            ){

                dolphin.position.y=-.2;

                verticalVelocity=0;

                jumping=false;
            }

        }else{

            player.position.y+=
                verticalVelocity*dt;


            const groundY=
                terrainHeight(
                    player.position.x,
                    player.position.z
                )+.67;


            if(
                player.position.y<=groundY
            ){

                player.position.y=
                    groundY;

                verticalVelocity=0;

                jumping=false;
            }
        }
    }


    /* ========================================================
       DOLPHIN
       ======================================================== */

    dolphin.rotation.z=
        Math.sin(time*2.6)*.035;


    if(!riding){

        dolphin.position.y=
            -.2+
            Math.sin(time*2.2)*.12;
    }


    if(riding){

        dolphin.position.y+=
            Math.sin(time*4)*.006;


        player.position.copy(
            dolphin.position
        );

        player.position.y+=2.2;

        player.rotation.y=
            cameraYaw;
    }


    /* ========================================================
       PLAYER ANIMATION
       ======================================================== */

    animateCharacter(
        player,
        actualSpeed,
        time,
        dt
    );


    /* ========================================================
       NPC ANIMATION
       ======================================================== */

    npcs.forEach(
        (npc,index)=>{

            const npcTime=
                time+
                index*1.7;


            npc.position.y=
                terrainHeight(
                    npc.position.x,
                    npc.position.z
                )+.67+
                Math.sin(
                    npcTime*1.4
                )*.025;


            animateCharacter(
                npc,
                .8,
                npcTime,
                dt
            );
        }
    );


    /* ========================================================
       BUBBLES
       ======================================================== */

    bubbles.forEach(
        (bubble,index)=>{

            bubble.position.y+=
                dt*
                (
                    .22+
                    (index%5)*.1
                );


            bubble.rotation.y+=
                dt*.4;


            if(
                bubble.position.y>17
            ){

                bubble.position.y=-1;
            }
        }
    );


    /* ========================================================
       ROBLOX-LIKE CAMERA
       ======================================================== */

    const target=
        riding
        ?dolphin
        :player;


    const targetDistance=
        riding
        ?17
        :13;


    cameraDistance=
        THREE.MathUtils.lerp(
            cameraDistance,
            targetDistance,
            .08
        );


    /*
       Shift Lock:

       Camera stays behind the character.

       Normal:

       Camera stays behind the camera
       orientation.
    */

    const horizontal=
        Math.cos(
            cameraPitch
        )*
        cameraDistance;


    const desiredCameraX=
        target.position.x-
        Math.sin(cameraYaw)*
        horizontal;


    const desiredCameraZ=
        target.position.z+
        Math.cos(cameraYaw)*
        horizontal;


    const desiredCameraY=
        target.position.y+
        3.4+
        Math.sin(cameraPitch)*
        cameraDistance;


    camera.position.x=
        THREE.MathUtils.lerp(
            camera.position.x,
            desiredCameraX,
            .13
        );


    camera.position.y=
        THREE.MathUtils.lerp(
            camera.position.y,
            desiredCameraY,
            .13
        );


    camera.position.z=
        THREE.MathUtils.lerp(
            camera.position.z,
            desiredCameraZ,
            .13
        );


    camera.lookAt(
        target.position.x,
        target.position.y+1.6,
        target.position.z
    );


    /* ========================================================
       WATER ANIMATION
       ======================================================== */

    water.position.y=
        -1.8+
        Math.sin(time)*.015;


    /* ========================================================
       RENDER
       ======================================================== */

    renderer.render(
        scene,
        camera
    );
}


animate();


/* ============================================================
   RESIZE
   ============================================================ */

function resize(){

    camera.aspect=
        innerWidth/
        innerHeight;

    camera.updateProjectionMatrix();

    renderer.setSize(
        innerWidth,
        innerHeight
    );

    renderer.setPixelRatio(
        Math.min(
            window.devicePixelRatio,
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
