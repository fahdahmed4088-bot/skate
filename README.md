<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Ultimate Browser Skate Game</title>
<style>
  body { margin:0; overflow:hidden; }
  canvas { display:block; }
  #hud {
    position:absolute; top:10px; left:10px;
    background:rgba(0,0,0,0.7); color:white;
    font-family:sans-serif; padding:10px; border-radius:8px;
  }
  #shop {
    position:absolute; bottom:10px; left:10px;
    background:rgba(0,0,0,0.8); color:white;
    font-family:sans-serif; padding:10px; border-radius:8px;
  }
  button {
    background:#222; color:white; border:none;
    padding:5px 10px; margin:5px; border-radius:5px; cursor:pointer;
  }
  button:hover { background:#555; }
  #tutorial {
    position:absolute; top:50%; left:50%; transform:translate(-50%,-50%);
    background:rgba(0,0,0,0.8); color:white; font-family:sans-serif;
    padding:20px; border-radius:10px; max-width:400px; text-align:center;
  }
</style>
</head>
<body>

<div id="hud">Money: $<span id="money">0</span></div>
<div id="shop"><b>Skate Shop</b></div>
<div id="tutorial">
  <h2>Welcome to Browser Skate Game!</h2>
  <p><b>Controls:</b></p>
  <ul>
    <li>Move: W / S</li>
    <li>Turn: A / D</li>
    <li>Jump/Ollie: Space</li>
    <li>Flip Tricks: ArrowLeft / ArrowRight</li>
    <li>Spin Tricks: Q / E</li>
    <li>Kickflip / Heelflip: Z / X</li>
    <li>Get off skateboard / walk: G</li>
  </ul>
  <p>Press <b>Enter</b> to start!</p>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.156.1/build/three.min.js"></script>
<script>
// === SCENE SETUP ===
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xa0c4ff);
const camera = new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth,window.innerHeight);
document.body.appendChild(renderer.domElement);

// LIGHT
const light = new THREE.DirectionalLight(0xffffff,1);
light.position.set(50,50,50);
scene.add(light);

// GROUND
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(1000,1000),
  new THREE.MeshStandardMaterial({color:0x66cc66})
);
ground.rotation.x=-Math.PI/2;
scene.add(ground);

// === CITY BUILDINGS ===
const buildingColors=[0xff9999,0x99ff99,0x9999ff,0xffff99,0xffcc99];
for(let i=0;i<20;i++){
  const b=new THREE.Mesh(
    new THREE.BoxGeometry(10+Math.random()*10,20+Math.random()*30,10+Math.random()*10),
    new THREE.MeshStandardMaterial({color:buildingColors[Math.floor(Math.random()*buildingColors.length)]})
  );
  b.position.set(Math.random()*400-200,b.geometry.parameters.height/2,Math.random()*400-200);
  scene.add(b);
}

// === RAMPS AND RAILS ===
const ramps=[];
for(let i=0;i<10;i++){
  const ramp=new THREE.Mesh(
    new THREE.BoxGeometry(10,1,5),
    new THREE.MeshStandardMaterial({color:0x884422})
  );
  ramp.position.set(Math.random()*400-200,0.5,Math.random()*400-200);
  ramp.rotation.x=Math.PI/12;
  scene.add(ramp);
  ramps.push(ramp);
}
const rails=[];
for(let i=0;i<10;i++){
  const rail=new THREE.Mesh(
    new THREE.BoxGeometry(8,0.3,0.3),
    new THREE.MeshStandardMaterial({color:0x222222})
  );
  rail.position.set(Math.random()*400-200,0.15,Math.random()*400-200);
  scene.add(rail);
  rails.push(rail);
}

// === SKATEBOARD ===
const deckGeo=new THREE.BoxGeometry(1.5,0.1,0.4);
let deckMat=new THREE.MeshStandardMaterial({color:0xff3333});
const deck=new THREE.Mesh(deckGeo,deckMat);

// WHEELS
const wheelGeo=new THREE.CylinderGeometry(0.15,0.15,0.2,16);
const wheelMat=new THREE.MeshStandardMaterial({color:0x000000});
const offsets=[[0.6,-0.1,0.2],[-0.6,-0.1,0.2],[0.6,-0.1,-0.2],[-0.6,-0.1,-0.2]];
for(let o of offsets){
  const w=new THREE.Mesh(wheelGeo,wheelMat);
  w.rotation.z=Math.PI/2;
  w.position.set(o[0],o[1],o[2]);
  deck.add(w);
}
deck.position.set(0,0.2,0);
scene.add(deck);

// CHARACTER
const body=new THREE.Mesh(new THREE.BoxGeometry(0.5,1,0.3), new THREE.MeshStandardMaterial({color:0x00aaff}));
body.position.y=0.8;
deck.add(body);
const head=new THREE.Mesh(new THREE.SphereGeometry(0.25,16,16), new THREE.MeshStandardMaterial({color:0xffcc99}));
head.position.y=1.4;
body.add(head);
const hair=new THREE.Mesh(new THREE.SphereGeometry(0.27,16,16,0,Math.PI*2,0,Math.PI/2), new THREE.MeshStandardMaterial({color:0x222222}));
hair.position.y=0.05;
head.add(hair);
const legs=new THREE.Mesh(new THREE.BoxGeometry(0.4,0.8,0.2), new THREE.MeshStandardMaterial({color:0x3333ff}));
legs.position.y=-0.8;
body.add(legs);

// === BRIDGE CHALLENGE ===
const bridge=new THREE.Mesh(new THREE.BoxGeometry(10,0.5,50),new THREE.MeshStandardMaterial({color:0x4444ff}));
bridge.position.set(0,1,-50); scene.add(bridge);
const goal=new THREE.Mesh(new THREE.BoxGeometry(12,0.5,2),new THREE.MeshStandardMaterial({color:0x00ff00}));
goal.position.set(0,1,-80); scene.add(goal);
let challengeComplete=false;

// === SHOP ===
let money=0;
const boards=[
  {color:0xff3333,name:"Red Board",price:50,stockChance:1},
  {color:0x33ff33,name:"Green Board",price:100,stockChance:0.8},
  {color:0x3333ff,name:"Blue Board",price:150,stockChance:0.6},
  {color:0xffff33,name:"Gold Board",price:200,stockChance:0.4}
];

function addMoney(amount){ money+=amount; document.getElementById("money").innerText=money; }
function buyBoard(color,price){ if(money>=price){ money-=price; deck.material.color.setHex(color); document.getElementById("money").innerText=money; alert("You bought a new skateboard!"); }else alert("Not enough money!"); }
function showShop(){
  const shopDiv=document.getElementById("shop");
  shopDiv.innerHTML="<b>Skate Shop</b><br>";
  boards.forEach(b=>{
    const inStock=Math.random()<b.stockChance;
    const btn=document.createElement("button");
    btn.innerText=`${b.name} ($${b.price})`+(inStock?"":" - Out of stock");
    if(inStock) btn.onclick=()=>buyBoard(b.color,b.price);
    btn.disabled=!inStock;
    shopDiv.appendChild(btn);
  });
}
showShop();

// === CONTROLS ===
const keys={}; window.addEventListener("keydown",(e)=>keys[e.key]=true);
window.addEventListener("keyup",(e)=>keys[e.key]=false);

// TRICKS
let airborne=false,airRotation={x:0,y:0,z:0},landed=true;
let onSkateboard=true;

// MOVEMENT
let velocity={x:0,z:0,y:0},speed=0.08,friction=0.95,onGround=true;

// CAMERA
camera.position.set(0,3,6);

// === TUTORIAL ===
const tutorial=document.getElementById("tutorial");
window.addEventListener("keydown",(e)=>{ if(e.key==="Enter") tutorial.style.display="none"; });

// === ANIMATE ===
function animate(){
  requestAnimationFrame(animate);

  // GET ON/OFF SKATEBOARD
  if(keys['g']) onSkateboard=!onSkateboard; keys['g']=false;

  if(onSkateboard){
    // MOVE
    if(keys["w"]) velocity.z -= speed;
    if(keys["s"]) velocity.z += speed;
    if(keys["a"]) deck.rotation.y += 0.05;
    if(keys["d"]) deck.rotation.y -= 0.05;

    // JUMP
    if(keys[" "] && onGround){ velocity.y=0.2; airborne=true; onGround=false; }

    // AIR ROTATION (TRICKS)
    if(airborne){
      if(keys['ArrowLeft']) { deck.rotation.x -= 0.1; airRotation.x -= 36; }
      if(keys['ArrowRight']) { deck.rotation.x += 0.1; airRotation.x += 36; }
      if(keys['q']) { deck.rotation.y -= 0.05; airRotation.y -= 18; }
      if(keys['e']) { deck.rotation.y += 0.05; airRotation.y += 18; }
      if(keys['z']) { deck.rotation.z -= 0.1; airRotation.z -= 36; }
      if(keys['x']) { deck.rotation.z += 0.1; airRotation.z += 36; }
    }

    // APPLY FRICTION & GRAVITY
    velocity.x*=friction; velocity.z*=friction; velocity.y-=0.01;
    deck.position.x+=Math.sin(deck.rotation.y)*velocity.z;
    deck.position.z+=Math.cos(deck.rotation.y)*velocity.z;
    deck.position.y+=velocity.y;

    // GROUND COLLISION
    if(deck.position.y<=0.2){
      deck.position.y=0.2; velocity.y=0; onGround=true;
      if(airborne){ airborne=false; landed=true;

        // TRICKS DETECTION
        let trickName='';
        if(Math.abs(airRotation.z)>=360) trickName='Kickflip!';
        else if(Math.abs(airRotation.y)>=360) trickName='360 Spin!';
        else if(Math.abs(airRotation.x)>=360) trickName='Flip Trick!';
        else if(Math.abs(airRotation.y)>=180) trickName='Shove-It!';
        else trickName='Ollie';

        let points=Math.floor(Math.abs(airRotation.x)+Math.abs(airRotation.y)+Math.abs(airRotation.z));
        let moneyEarn=Math.floor(points/50);
        if(points>0){ addMoney(moneyEarn); alert(`You did a ${trickName}! +$${moneyEarn}`); }
        airRotation={x:0,y:0,z:0};
      }
    }

    // CHALLENGE
    if(!challengeComplete && deck.position.z<-75 && Math.abs(deck.position.x)<6){
      challengeComplete=true;
      addMoney(100);
      alert("You cleared the bridge gap! +$100");
    }
  } else {
    // WALK MODE
    if(keys["w"]) deck.position.z-=speed;
    if(keys["s"]) deck.position.z+=speed;
    if(keys["a"]) deck.position.x-=speed;
    if(keys["d"]) deck.position.x+=speed;
  }

  // CAMERA
  camera.position.x=deck.position.x;
  camera.position.z=deck.position.z+6;
  camera.position.y=deck.position.y+2;
  camera.lookAt(deck.position);

  renderer.render(scene,camera);
}
animate();
</script>
</body>
</html>
