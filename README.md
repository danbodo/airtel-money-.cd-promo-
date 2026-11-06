<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kinshasa Drive 3D</title>
<style>
body { margin:0; overflow:hidden; background:#111; font-family:Arial; }
#hud { position:absolute; top:12px; left:12px; color:white; z-index:2; }
#controls { position:absolute; bottom:20px; width:100%; display:flex;
justify-content:space-around; z-index:3; }
button { padding:18px; border:0; border-radius:12px;
background:#222; color:white; font-size:18px; opacity:.9; }
#steering { position:absolute; bottom:105px; left:50%;
transform:translateX(-50%); color:white; z-index:3; }
</style>
</head>
<body>

<div id="hud">
  <h3>KINSHASA DRIVE 3D</h3>
  <div>Vitesse : <span id="speed">0</span> km/h</div>
  <div>Quartier : Limeté</div>
</div>

<div id="steering">VOLANT : ◀ ▶</div>

<div id="controls">
  <button id="left">◀</button>
  <button id="horn">KLAXON</button>
  <button id="right">▶</button>
  <button id="brake">FREIN</button>
  <button id="gas">ACCÉLÉRER</button>
</div>

<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);
scene.fog = new THREE.Fog(0x87ceeb, 35, 180);

const camera = new THREE.PerspectiveCamera(
  75, innerWidth / innerHeight, 0.1, 300
);
camera.position.set(0, 2.2, 5);
camera.rotation.x = -0.08;

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff, 0x555555, 2));

const sun = new THREE.DirectionalLight(0xffffff, 2);
sun.position.set(-5, 12, 8);
scene.add(sun);

// Route
const road = new THREE.Mesh(
  new THREE.PlaneGeometry(14, 300),
  new THREE.MeshStandardMaterial({color:0x333333})
);
road.rotation.x = -Math.PI/2;
road.position.z = -100;
scene.add(road);

// Lignes de la route
for(let z=10; z>-250; z-=12){
  const line = new THREE.Mesh(
    new THREE.BoxGeometry(.18,.03,5),
    new THREE.MeshStandardMaterial({color:0xffffff})
  );
  line.position.set(0,.025,z);
  scene.add(line);
}

// Bâtiments
for(let z=10; z>-250; z-=18){
  for(let side of [-1,1]){
    const height = 5 + Math.random()*12;
    const building = new THREE.Mesh(
      new THREE.BoxGeometry(5,height,8),
      new THREE.MeshStandardMaterial({
        color:[0xc99b68,0xb5a18b,0x999999,0xd5b45c]
          [Math.floor(Math.random()*4)]
      })
    );
    building.position.set(side*12,height/2,z);
    scene.add(building);
  }
}

// Cockpit
const dashboard = new THREE.Mesh(
  new THREE.BoxGeometry(4.5,.65,1.1),
  new THREE.MeshStandardMaterial({color:0x171717})
);
dashboard.position.set(0,.65,1.2);
scene.add(dashboard);

// Capot
const hood = new THREE.Mesh(
  new THREE.BoxGeometry(2.5,.25,1.8),
  new THREE.MeshStandardMaterial({color:0x174c75})
);
hood.position.set(0,1.1,-.1);
scene.add(hood);

// Volant
const wheel = new THREE.Mesh(
  new THREE.TorusGeometry(.48,.07,12,40),
  new THREE.MeshStandardMaterial({color:0x111111})
);
wheel.position.set(0,1.45,1.1);
wheel.rotation.x=.25;
scene.add(wheel);

// Circulation
const traffic=[];

function createCar(x,z,color,bus=false){
  const group=new THREE.Group();

  const body=new THREE.Mesh(
    new THREE.BoxGeometry(bus?2.8:1.8,bus?2.4:1.1,bus?6:3.5),
    new THREE.MeshStandardMaterial({color})
  );
  body.position.y=bus?1.3:.65;
  group.add(body);

  const windows=new THREE.Mesh(
    new THREE.BoxGeometry(bus?2.5:1.6,.65,bus?4:1.8),
    new THREE.MeshStandardMaterial({color:0x9dd7e8})
  );
  windows.position.y=bus?2.2:1.25;
  group.add(windows);

  group.position.set(x,0,z);
  scene.add(group);
  traffic.push(group);
}

for(let i=0;i<9;i++){
  createCar(
    [-4,4][i%2],
    -25-i*24,
    i%3===0?0xffcc00:0x2366aa,
    i%3===0
  );
}

let speed=0;
let steering=0;
let moving=0;
let direction=0;

function hold(id, action){
  const button=document.getElementById(id);
  button.addEventListener('pointerdown',e=>{
    e.preventDefault(); action(true);
  });
  button.addEventListener('pointerup',()=>action(false));
  button.addEventListener('pointerleave',()=>action(false));
  button.addEventListener('pointercancel',()=>action(false));
}

hold('gas',v=>moving=v?1:0);
hold('brake',v=>moving=v?-1:0);
hold('left',v=>direction=v?-1:0);
hold('right',v=>direction=v?1:0);

document.getElementById('horn').onclick=()=>{
  alert('POOOON !');
};

function animate(){
  requestAnimationFrame(animate);

  speed += moving===1 ? .15 : moving===-1 ? -.3 : -.04;
  speed=Math.max(0,Math.min(160,speed));

  steering += (direction-steering)*.08;
  camera.position.x += steering*.045;
  camera.position.x=Math.max(-2.5,Math.min(2.5,camera.position.x));
  wheel.rotation.z=-steering*.7;

  traffic.forEach(car=>{
    car.position.z += speed*.012;
    if(car.position.z>8) car.position.z=-220;
  });

  document.getElementById('speed').textContent=Math.round(speed);
  renderer.render(scene,camera);
}

animate();

addEventListener('resize',()=>{
  camera.aspect=innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth,innerHeight)
});
</script>
</body>
</html>
