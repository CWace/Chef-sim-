<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"/>
<title>Chef Sim Pure</title>
<style>
body { margin:0; overflow:hidden; font-family:monospace; background:#000; }
#hud {
  position:absolute; top:10px; left:10px;
  background:rgba(0,0,0,0.6);
  color:white; padding:10px;
}
#chat {
  position:absolute; bottom:10px; left:10px;
  width:320px;
  background:rgba(0,0,0,0.7);
  color:white;
}
#log { height:120px; overflow:auto; }
</style>
</head>
<body>

<div id="hud"></div>

<div id="chat">
  <div id="log"></div>
  <input id="input" style="width:100%" placeholder="Ask AI..." />
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>

<script>
// ================= CORE =================
let scene,camera,renderer;
let keys={};
let raycaster=new THREE.Raycaster();
let interactables=[];

let state={
  cash:2000,
  rep:0,
  cooking:false,
  cookTimer:0,
  order:null,
  npc:null
};

// ================= INIT =================
init();
loop();

function init(){
  scene=new THREE.Scene();

  camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,0.1,1000);
  camera.position.set(0,2,5);

  renderer=new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(innerWidth,innerHeight);
  document.body.appendChild(renderer.domElement);

  scene.add(new THREE.AmbientLight(0xffffff,0.4));

  const light=new THREE.DirectionalLight(0xffffff,1);
  light.position.set(5,10,5);
  scene.add(light);

  buildKitchen();

  spawnCustomer();

  document.addEventListener("keydown",e=>keys[e.key.toLowerCase()]=true);
  document.addEventListener("keyup",e=>keys[e.key.toLowerCase()]=false);

  document.addEventListener("keydown",e=>{
    if(e.key==="e") interact();
  });

  document.addEventListener("click",()=>document.body.requestPointerLock());
}

// ================= WORLD =================
function buildKitchen(){
  const floor=new THREE.Mesh(
    new THREE.PlaneGeometry(50,50),
    new THREE.MeshStandardMaterial({color:0x222222})
  );
  floor.rotation.x=-Math.PI/2;
  scene.add(floor);

  addObj(0,1,-2,2,2,2,0x444444,"stove");
}

// ================= OBJECT =================
function addObj(x,y,z,sx,sy,sz,color,name){
  const m=new THREE.Mesh(
    new THREE.BoxGeometry(sx,sy,sz),
    new THREE.MeshStandardMaterial({color})
  );
  m.position.set(x,y,z);
  m.name=name;
  scene.add(m);
  interactables.push(m);
}

// ================= CUSTOMER =================
function spawnCustomer(){
  state.npc={
    name:"Guest_"+Math.floor(Math.random()*999),
    patience:100
  };

  state.order="burger";

  log("Customer arrived: wants "+state.order);
}

// ================= INTERACTION =================
function interact(){
  raycaster.setFromCamera(new THREE.Vector2(0,0),camera);
  const hits=raycaster.intersectObjects(interactables);

  if(!hits.length) return;

  const obj=hits[0].object;

  if(obj.name==="stove") startCook();
}

// ================= COOKING =================
function startCook(){
  if(state.cooking) return;

  state.cooking=true;
  state.cookTimer=4;

  log("Cooking started...");
}

function finishCook(){
  state.cooking=false;

  let success=Math.random();

  if(success>0.3){
    state.cash+=25;
    state.rep+=1;
    log("Perfect dish. +$25");
  }else{
    state.cash+=5;
    log("Bad dish. customer still paid a bit.");
  }

  spawnCustomer();
}

// ================= AI =================
function ai(msg){
  msg=msg.toLowerCase();

  if(msg.includes("money")) return "$"+state.cash;
  if(msg.includes("order")) return state.order;
  if(msg.includes("customer")) return state.npc.name;

  return "No deep AI system yet. Just kitchen instinct.";
}

// ================= INPUT =================
document.getElementById("input").addEventListener("keydown",e=>{
  if(e.key==="Enter"){
    let v=e.target.value;
    log("You: "+v);
    log("AI: "+ai(v));
    e.target.value="";
  }
});

// ================= LOOP =================
function loop(){
  requestAnimationFrame(loop);

  move();

  if(state.cooking){
    state.cookTimer-=0.016;

    if(state.cookTimer<=0){
      finishCook();
    }
  }

  renderer.render(scene,camera);
}

// ================= MOVEMENT =================
function move(){
  let speed=0.08;

  if(keys["w"]) camera.translateZ(-speed);
  if(keys["s"]) camera.translateZ(speed);
  if(keys["a"]) camera.translateX(-speed);
  if(keys["d"]) camera.translateX(speed);
}

// ================= CHAT =================
function log(t){
  let el=document.getElementById("log");
  el.innerHTML+=t+"<br>";
  el.scrollTop=999999;
}

// ================= HUD =================
setInterval(()=>{
  document.getElementById("hud").innerHTML=`
  Cash: $${state.cash}<br>
  Reputation: ${state.rep}<br>
  Status: ${state.cooking?"Cooking":"Idle"}<br>
  Customer: ${state.npc?.name || "none"}
  `;
},100);

</script>

</body>
</html>
