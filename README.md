<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>BlockCraft Science Review</title>
<meta name="claude:viewport-fit-sides" content="cover">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Pixelify+Sans:wght@400;600;700&family=Atkinson+Hyperlegible:wght@400;700&display=swap">
<style>
/* Layout: HUD bar, then a walkable top-down block world on a canvas (three biomes left to right), then touch controls. Question and lab cards pop over the world. Single dark look by choice. */
:root{
  --sky:#141a2b; --sky2:#1d2540; --ink:#f3efe2; --ink2:#b9b7c9; --panel:#232a42; --panel2:#30395a; --edge:#0b0e18;
  --gold:#f2c14e; --ok:#5fd068; --bad:#ef6a5b; --cyan:#47d6c8; --leaf:#5fd068;
  --f-display:"Pixelify Sans","Courier New",monospace; --f-body:"Atkinson Hyperlegible",system-ui,sans-serif;
  color-scheme:dark;
}
*{box-sizing:border-box}
[hidden]{display:none!important}
html,body{background:var(--sky);color:var(--ink)}
body{font-family:var(--f-body);font-size:17px;line-height:1.5;margin:0;
  padding-left:max(16px,env(safe-area-inset-left,0px));padding-right:max(16px,env(safe-area-inset-right,0px));padding-block:18px 40px;
  background:linear-gradient(var(--sky) 0 40%,var(--sky2)) fixed}
.wrap{max-width:1100px;margin:0 auto;display:flex;flex-direction:column;gap:14px}
h1,h2,h3{font-family:var(--f-display);font-weight:700;margin:0;text-wrap:balance;letter-spacing:.5px}
h1{font-size:clamp(28px,5vw,46px);line-height:1;color:var(--gold);text-shadow:4px 4px 0 var(--edge)}
.lede{color:var(--ink2);max-width:70ch;margin:6px 0 0;font-size:16px}
header{display:flex;flex-wrap:wrap;gap:12px;align-items:flex-end;justify-content:space-between}
.row{display:flex;flex-wrap:wrap;gap:8px;align-items:center}
.chip{font-family:var(--f-display);font-size:17px;background:var(--panel);border:3px solid var(--edge);padding:5px 11px;box-shadow:inset -3px -3px 0 rgba(0,0,0,.35),inset 3px 3px 0 rgba(255,255,255,.08);font-variant-numeric:tabular-nums;white-space:nowrap}
.chip.turn{outline:3px solid var(--gold)}
.chip b{color:var(--gold)}
button{font:inherit;cursor:pointer;color:var(--ink)}
.btn{font-family:var(--f-display);font-size:16px;background:#3b4468;border:3px solid var(--edge);padding:6px 12px;box-shadow:inset -3px -3px 0 rgba(0,0,0,.4),inset 3px 3px 0 rgba(255,255,255,.12)}
.btn:hover{background:#4a5582}.btn:active{transform:translateY(2px)}
.btn.on{background:var(--gold);color:#2a1f05}
.btn:focus-visible,.opt:focus-visible,.tile:focus-visible,.zone:focus-visible,canvas:focus-visible{outline:3px solid var(--gold);outline-offset:2px}
.hotbar{display:flex;gap:6px;flex-wrap:wrap}
.slot{width:58px;height:58px;background:#2a3150;border:3px solid var(--edge);box-shadow:inset 3px 3px 0 rgba(0,0,0,.4);position:relative;display:flex;align-items:center;justify-content:center}
.slot .g{width:22px;height:22px;transform:rotate(45deg);border:2px solid var(--edge);box-shadow:inset -3px -3px 0 rgba(0,0,0,.25),inset 3px 3px 0 rgba(255,255,255,.5)}
.slot .n{position:absolute;right:3px;bottom:0;font-family:var(--f-display);font-size:15px;text-shadow:2px 2px 0 var(--edge);font-variant-numeric:tabular-nums}
.slot .t{font-family:var(--f-display);font-size:11px;text-align:center;line-height:1.1;padding:2px;color:var(--ink2)}
.slot.got{background:#4a3f17;outline:2px solid var(--gold)}
.slot.got .t{color:var(--gold)}
.stage{position:relative;border:4px solid var(--edge);background:#000;box-shadow:8px 8px 0 var(--edge)}
canvas{display:block;width:100%;height:clamp(300px,58vh,620px);image-rendering:pixelated;touch-action:none;cursor:pointer}
.status{position:absolute;left:10px;top:10px;font-family:var(--f-display);font-size:16px;background:rgba(11,14,24,.82);padding:4px 10px;border:2px solid var(--edge);pointer-events:none;max-width:calc(100% - 20px)}
.toast{position:absolute;left:50%;top:44%;transform:translate(-50%,-50%);font-family:var(--f-display);font-size:clamp(20px,4vw,32px);color:var(--gold);background:rgba(11,14,24,.9);border:3px solid var(--edge);padding:10px 18px;text-align:center;pointer-events:none;opacity:0;transition:opacity .3s}
.toast.show{opacity:1}
.controls{display:flex;flex-wrap:wrap;gap:16px;align-items:center;justify-content:space-between}
.dpad{display:grid;grid-template-columns:repeat(3,52px);grid-template-rows:repeat(2,52px);gap:6px}
.dpad .btn{font-size:20px;padding:0}
.help{color:var(--ink2);font-size:15px;max-width:60ch;margin:0}
.help kbd{font-family:var(--f-display);background:var(--panel2);border:2px solid var(--edge);padding:0 6px;color:var(--ink)}
.mine{font-size:22px;padding:12px 26px;background:#7a4d22}
.mine:hover{background:#94602d}
/* cards */
.scrim{position:fixed;inset:0;background:rgba(5,7,14,.8);display:flex;align-items:center;justify-content:center;z-index:20;
  padding:max(16px,env(safe-area-inset-top,0px)) max(16px,env(safe-area-inset-right,0px)) max(16px,env(safe-area-inset-bottom,0px)) max(16px,env(safe-area-inset-left,0px))}
.card{background:var(--panel);border:4px solid var(--edge);width:min(720px,100%);max-height:100%;overflow:auto;box-shadow:8px 8px 0 var(--edge)}
.card .head{padding:11px 18px;border-bottom:4px solid var(--edge);display:flex;justify-content:space-between;gap:12px;font-family:var(--f-display);color:var(--gold);font-size:17px}
.card .body{padding:18px;display:flex;flex-direction:column;gap:14px}
.q{font-size:21px;font-weight:700;margin:0;line-height:1.35}
.opts{display:grid;gap:10px}
.opt{text-align:left;background:var(--panel2);border:3px solid var(--edge);padding:12px 14px;font-size:18px;display:flex;gap:12px;align-items:flex-start;box-shadow:inset -3px -3px 0 rgba(0,0,0,.35)}
.opt:hover:not(:disabled){background:#3d4875}
.opt kbd{font-family:var(--f-display);background:var(--edge);color:var(--gold);padding:0 7px;flex:none}
.opt.right{background:#2c6b36;border-color:var(--ok)}.opt.wrong{background:#7a3029;border-color:var(--bad)}
.opt:disabled{cursor:default}
.fb{padding:12px 14px;border:3px solid var(--edge);background:var(--edge);display:flex;flex-direction:column;gap:4px}
.fb strong{font-family:var(--f-display);font-size:20px}
.fb.good strong{color:var(--ok)}.fb.miss strong{color:var(--bad)}
.std{font-size:13px;color:var(--ink2);letter-spacing:.06em;text-transform:uppercase}
.pool{display:flex;flex-wrap:wrap;gap:8px;min-height:52px;padding:10px;background:var(--edge);border:3px dashed #3b4468}
.tile{background:#5b4630;border:3px solid var(--edge);padding:7px 12px;font-size:16px;font-weight:700;box-shadow:inset -3px -3px 0 rgba(0,0,0,.35),inset 3px 3px 0 rgba(255,255,255,.15)}
.tile.sel{background:var(--gold);color:#2a1f05;transform:translateY(-3px)}
.tile.lock{background:#2c6b36;cursor:default}
.tile.shake{animation:shake .35s}
@keyframes shake{25%{transform:translateX(-6px)}75%{transform:translateX(6px)}}
.zones{display:grid;grid-template-columns:repeat(auto-fit,minmax(170px,1fr));gap:10px}
.zone{background:var(--panel2);border:3px solid var(--edge);padding:10px;min-height:120px;display:flex;flex-direction:column;gap:8px;text-align:left;align-items:flex-start}
.zone h3{font-size:17px;color:var(--gold)}
.zone .in{display:flex;flex-wrap:wrap;gap:6px}
.zone.target{outline:3px dashed var(--gold)}
.diagram{display:flex;flex-wrap:wrap;gap:14px;align-items:center}
.diagram svg{width:200px;max-width:100%;flex:none;background:#2a3a5a;border:3px solid var(--edge)}
.slots{display:flex;flex-direction:column;gap:8px;flex:1;min-width:0}
.slotrow{display:flex;gap:10px;align-items:center;background:var(--panel2);border:3px solid var(--edge);padding:6px 10px;min-height:52px;text-align:left;width:100%}
.slotrow .L{font-family:var(--f-display);font-size:20px;color:var(--gold);width:22px;flex:none}
.slotrow.target{outline:3px dashed var(--gold)}
.codebar{display:flex;gap:4px;align-items:center;font-family:var(--f-display)}
.codebar .lbl{font-size:13px;color:var(--ink2);margin-right:4px;text-transform:uppercase;letter-spacing:.08em}
.codebar .c{width:30px;height:40px;background:#2a3150;border:3px solid var(--edge);display:flex;align-items:center;justify-content:center;font-size:22px;color:var(--gold)}
.bigcode{font-family:var(--f-display);font-size:clamp(40px,10vw,72px);letter-spacing:.18em;color:var(--gold);background:var(--edge);border:4px solid var(--gold);text-align:center;padding:10px 6px;text-shadow:4px 4px 0 #5a4410}
.end h2{font-size:32px;color:var(--gold)}
@media (prefers-reduced-motion:reduce){*{animation:none!important;transition:none!important}}
</style>
</head>
<body>


<div class="wrap">
  <header>
    <div>
      <h1>BlockCraft Science Review</h1>
      <p class="lede">Explore three biomes, mine the glowing ore blocks and answer each question to collect gems. Find the crafting lab in every biome to finish a hands-on challenge. Collect all 10 gems and clear the lab to craft that biome's tool and unlock letters of the secret code.</p>
    </div>
    <div class="row" id="scores"></div>
  </header>
  <div class="row" style="justify-content:space-between">
    <div class="row"><div class="hotbar" id="hotbar"></div><div class="codebar" id="codebar" aria-label="Secret code"></div></div>
    <div class="row">
      <button class="btn" id="mode1">1 player</button>
      <button class="btn" id="mode2">2 teams</button>
      <button class="btn" id="sound">Sound: on</button>
      <button class="btn" id="reset">New world</button>
    </div>
  </div>
  <div class="stage">
    <canvas id="cv" tabindex="0" aria-label="Game world. Use arrow keys to walk and Space to mine."></canvas>
    <div class="status" id="status"></div>
    <div class="toast" id="toast"></div>
  </div>
  <div class="controls">
    <p class="help"><kbd>←↑↓→</kbd> or <kbd>WASD</kbd> to walk, <kbd>Space</kbd> to mine the block you face. On a touch screen or smartboard, tap any ore block or lab and your miner walks there and mines it.</p>
    <div class="row" style="gap:18px">
      <div class="dpad" aria-label="Move">
        <span></span><button class="btn" data-d="0,-1" aria-label="Up">▲</button><span></span>
        <button class="btn" data-d="-1,0" aria-label="Left">◀</button><button class="btn" data-d="0,1" aria-label="Down">▼</button><button class="btn" data-d="1,0" aria-label="Right">▶</button>
      </div>
      <button class="btn mine" id="mineBtn">Mine</button>
    </div>
  </div>
</div>
<div class="scrim" id="scrim" hidden><div class="card" role="dialog" aria-modal="true" aria-labelledby="ctitle" id="card"></div></div>

<script>
/* ---------- content ---------- */
const BIOMES=[
 {name:"Cell Cave",code:"SUR",tool:"Microscope Pickaxe",gem:"#47d6c8",std:"Cells",qs:[
  ["What is the smallest unit of life that makes up all living things?",["Cell","Organ","Tissue","Bone"],"All living things are made of cells. Some living things are just one cell!"],
  ["Which part is the control center of the cell?",["Nucleus","Cell wall","Vacuole","Cytoplasm"],"The nucleus controls what the cell does, like a brain for the cell."],
  ["Which part is found in plant cells but NOT animal cells, and makes food from sunlight?",["Chloroplast","Nucleus","Cell membrane","Cytoplasm"],"Chloroplasts use sunlight to make food. Animal cells don't have them."],
  ["Which thin layer controls what goes in and out of a cell?",["Cell membrane","Cell wall","Nucleus","Chloroplast"],"The cell membrane is like a gate that lets some things in and keeps others out."],
  ["Which stiff outer layer gives plant cells their box shape?",["Cell wall","Cell membrane","Cytoplasm","Nucleus"],"Only plant cells have a stiff cell wall. It gives support and shape."],
  ["What is the jelly-like fluid that fills a cell?",["Cytoplasm","Chlorophyll","Vacuole","Nucleus"],"Cytoplasm fills the cell and holds the other parts in place."],
  ["Which part stores water, and is very large in plant cells?",["Vacuole","Nucleus","Cell wall","Chloroplast"],"The vacuole is a storage sac. Plant cells have one big one."],
  ["Which part do BOTH plant and animal cells have?",["Nucleus","Cell wall","Chloroplasts","Leaves"],"Plant and animal cells both have a nucleus, cell membrane and cytoplasm."],
  ["Why are most leaves green?",["Chloroplasts hold green chlorophyll","The cell wall is green","Water is green","The nucleus is green"],"Chlorophyll inside chloroplasts is green, and it captures sunlight."],
  ["What tool do scientists use to see cells?",["Microscope","Telescope","Magnet","Thermometer"],"Cells are too small to see with our eyes, so we use a microscope."]]},
 {name:"Plant Plains",code:"VI",tool:"Golden Seed Hoe",gem:"#5fd068",std:"4-LS1-1",qs:[
  ["Which part holds a plant in the ground and takes in water from the soil?",["Roots","Stem","Leaves","Flower"],"Roots anchor the plant and soak up water and nutrients."],
  ["Which part carries water from the roots up to the leaves and holds the plant up?",["Stem","Roots","Petals","Seeds"],"The stem supports the plant and moves water and food through tubes inside it."],
  ["Where does a plant make most of its food?",["Leaves","Roots","Seeds","Bark"],"Leaves catch sunlight and use it to make food for the plant."],
  ["Which part helps a plant reproduce by making seeds?",["Flower","Leaf","Root","Thorn"],"Flowers make seeds so new plants can grow."],
  ["How do bright petals and a sweet smell help a flowering plant?",["They attract pollinators like bees","They keep the plant warm","They store water","They scare away the rain"],"Bees and butterflies visit the flower and carry pollen so seeds can form."],
  ["Which part protects seeds and helps spread them to new places?",["Fruit","Root","Stem","Leaf"],"Animals eat fruit and carry the seeds away to new places."],
  ["How do a rose bush's thorns help it survive?",["They keep animals from eating it","They make food","They soak up water","They attract bees"],"Thorns are an external structure that protects the plant."],
  ["A cactus has a thick, waxy stem. How does that help it live in the desert?",["It stores water and keeps it from drying out","It helps it swim","It makes it grow flowers faster","It keeps bees away"],"The thick stem stores water and the waxy coat keeps water in."],
  ["Which of these is an INTERNAL plant structure?",["Tiny tubes inside the stem that carry water","Thorns","Petals","Bark"],"The tubes are inside the stem, so they're internal. The others are on the outside."],
  ["What does tree bark do for a tree?",["Protects it from insects and disease","Makes seeds","Takes in sunlight","Attracts birds"],"Bark is like a tough skin that protects the inside of the tree."]]},
 {name:"Animal Armor Hills",code:"VE",tool:"Armor Chestplate",gem:"#f2c14e",std:"4-LS1-1 / 4-LS1-2",qs:[
  ["How does a turtle's shell help it survive?",["It protects its soft body from predators","It helps it fly","It keeps it cool","It helps it find food"],"A hard shell is an external structure that protects the turtle."],
  ["What do a porcupine's sharp quills do?",["Defend it from predators","Help it swim","Keep it warm","Help it see"],"Predators get hurt by the quills, so they leave the porcupine alone."],
  ["When an animal's color helps it blend in with its surroundings, it is called…",["Camouflage","Hibernation","Migration","Pollination"],"Camouflage helps an animal hide from predators or sneak up on prey."],
  ["Which of these is an INTERNAL animal structure?",["Heart","Fur","Claws","Feathers"],"The heart is inside the body, so it's internal. It pumps blood."],
  ["How do a duck's webbed feet help it?",["They help it swim","They help it climb trees","They keep it warm","They help it dig"],"Webbed feet work like paddles to push through water."],
  ["How does a thick layer of blubber help a polar bear or seal?",["It keeps it warm in icy water","It helps it run faster","It helps it see","It makes it lighter"],"Blubber is fat under the skin that holds in body heat."],
  ["How does a skunk protect itself?",["It sprays a bad smell","It changes color","It grows a shell","It flies away"],"The strong smell makes predators stay far away."],
  ["A monarch butterfly's bright orange color warns birds that it tastes bad. This is called…",["Warning coloration","Camouflage","Hibernation","Molting"],"Bright colors can be a warning: don't eat me!"],
  ["How do an owl's very large eyes help it survive?",["They help it see at night to hunt","They scare away rain","They keep it warm","They help it swim"],"Big eyes take in more light, so owls can hunt in the dark."],
  ["An animal's senses send messages to its brain. What does the brain do next?",["Decides how the animal should respond","Makes food from sunlight","Grows new fur","Stores water"],"The brain processes the information and tells the body what to do, like run away."]]}
];
const LABS=[
 {title:"Cell Lab: Sort the cell parts",kind:"sort",how:"Tap a part, then tap the cell it belongs to.",
  zones:["Plant cell only","Animal cell only","Both cells"],
  items:[["Nucleus",2],["Cell membrane",2],["Cytoplasm",2],["Cell wall",0],["Chloroplasts",0],["One large vacuole",0],["Round, flexible shape",1],["Many small vacuoles",1]],
  done:"Plant cells have a cell wall, chloroplasts and one big vacuole. Both cells have a nucleus, membrane and cytoplasm."},
 {title:"Plant Lab: Label the plant",kind:"label",how:"Tap a label, then tap the letter it belongs to.",
  slots:["A","B","C","D","E"],items:[["Flower",0],["Leaf",1],["Fruit",2],["Stem",3],["Roots",4]],
  jobs:["makes seeds","makes food","protects seeds","carries water","takes in water"],
  done:"Each part has a job that helps the plant survive, grow and reproduce."},
 {title:"Animal Lab: Inside or outside?",kind:"sort",how:"Tap a structure, then tap Internal or External.",
  zones:["Internal (inside the body)","External (outside the body)"],
  items:[["Heart",0],["Brain",0],["Lungs",0],["Stomach",0],["Turtle shell",1],["Porcupine quills",1],["Fur",1],["Claws",1],["Feathers",1]],
  done:"Internal structures work inside the body. External structures protect the animal and help it move and find food."}
];
/* ---------- world ---------- */
const W=36,H=16,T=48;
function rng(seed){return function(){seed|=0;seed=seed+0x6D2B79F5|0;let t=Math.imul(seed^seed>>>15,1|seed);t=t+Math.imul(t^t>>>7,61|t)^t;return((t^t>>>14)>>>0)/4294967296;};}
const biomeOf=x=>x<12?0:x<24?1:2;
let tiles,objs,START={x:3,y:8};
function build(seed){
  for(let tries=0;tries<50;tries++){
    const r=rng(seed+tries);tiles=[];objs=[];
    for(let y=0;y<H;y++){tiles.push([]);for(let x=0;x<W;x++){
      let t=(x==0||y==0||x==W-1||y==H-1)?1:0;
      if((x==12||x==24)&&!(y>=6&&y<=9))t=1;
      if(!t&&r()<.11&&Math.abs(x-START.x)+Math.abs(y-START.y)>2&&!(x%12==11||x%12==1&&x>1))t=1;
      if(x>=15&&x<=18&&y>=10&&y<=12)t=2;
      tiles[y].push(t);}}
    const free=(x,y)=>tiles[y][x]==0&&!objs.find(o=>o.x==x&&o.y==y)&&!(x==START.x&&y==START.y)&&!(x%12==11||x%12==1&&x>1)&&!(x==13||x==23||x==25||x==11);
    const labs=[[6,3],[19,3],[30,3]];
    labs.forEach(([x,y],b)=>{tiles[y][x]=0;objs.push({x,y,b,lab:true,state:"new"});});
    for(let b=0;b<3;b++){let q=0,guard=0;while(q<10&&guard++<999){const x=b*12+1+Math.floor(r()*10),y=1+Math.floor(r()*(H-2));if(free(x,y)){objs.push({x,y,b,q,state:"new"});q++;}}}
    if(objs.length===33&&reachableAll())return;
  }
}
function walkable(x,y){return x>=0&&y>=0&&x<W&&y<H&&tiles[y][x]===0&&!objAt(x,y);}
function objAt(x,y){return objs.find(o=>o.x==x&&o.y==y&&o.state!=="mined");}
function bfs(sx,sy){const d=new Map(),k=(x,y)=>y*W+x,q=[[sx,sy]];d.set(k(sx,sy),null);
  while(q.length){const [x,y]=q.shift();for(const [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){const nx=x+dx,ny=y+dy;if(!d.has(k(nx,ny))&&walkable(nx,ny)){d.set(k(nx,ny),[x,y]);q.push([nx,ny]);}}}return d;}
function reachableAll(){const d=bfs(START.x,START.y);return objs.every(o=>[[1,0],[-1,0],[0,1],[0,-1]].some(([dx,dy])=>d.has((o.y+dy)*W+o.x+dx)));}
/* ---------- state ---------- */
let S;
function fresh(teams,seed){seed=seed??Math.floor(Math.random()*1e6);build(seed);
  return {seed,teams,turn:0,score:teams===2?[0,0]:[0],gems:[0,0,0],tools:[false,false,false],
    objs:objs.map(o=>o.state),px:START.x,py:START.y,face:[0,1],finished:false};}
function load(d){if(d&&d.objs){S=d;build(S.seed);objs.forEach((o,i)=>o.state=S.objs[i]);}else S=fresh(1);
  P.x=P.rx=S.px;P.y=P.ry=S.py;P.face=S.face;hud();}
function save(){S.objs=objs.map(o=>o.state);S.px=P.x;S.py=P.y;S.face=P.face;}
/* ---------- sound ---------- */
let AC=null,soundOn=true;
function tone(f,d=.08,type="square",v=.06,delay=0){if(!soundOn)return;try{AC=AC||new (window.AudioContext||window.webkitAudioContext)();const t=AC.currentTime+delay,o=AC.createOscillator(),g=AC.createGain();o.type=type;o.frequency.setValueAtTime(f,t);g.gain.setValueAtTime(v,t);g.gain.exponentialRampToValueAtTime(.0001,t+d);o.connect(g).connect(AC.destination);o.start(t);o.stop(t+d+.02);}catch(e){}}
const sfx={hit:()=>tone(140+Math.random()*40,.07,"square",.05),step:()=>tone(90,.03,"triangle",.03),
  good:()=>[523,659,784,1047].forEach((f,i)=>tone(f,.12,"square",.05,i*.08)),bad:()=>{tone(220,.18,"sawtooth",.05);tone(160,.25,"sawtooth",.05,.15);},
  craft:()=>[392,523,659,784,1047,1319].forEach((f,i)=>tone(f,.14,"square",.05,i*.09))};
/* ---------- player & render ---------- */
const P={x:3,y:8,rx:3,ry:8,face:[0,1],path:[],moving:false,swing:0,target:null,mining:0};
const cv=document.getElementById("cv"),ctx=cv.getContext("2d");
let parts=[],floats=[],cam={x:0,y:0},scale=1;
function hash(x,y){let h=x*374761393+y*668265263;h=(h^(h>>13))*1274126177;return((h^(h>>16))>>>0)/4294967296;}
function resize(){const r=cv.getBoundingClientRect(),dpr=Math.min(2,window.devicePixelRatio||1);cv.width=Math.round(r.width*dpr);cv.height=Math.round(r.height*dpr);
  scale=Math.max(.5,Math.min(1.6,cv.height/(T*9.5)));ctx.imageSmoothingEnabled=false;}
const PAL=[{f:"#3a3c48",f2:"#33353f",w:"#22232b",w2:"#575a68"},{f:"#5aa83c",f2:"#4f9a33",w:"#2f6b20",w2:"#6d4528"},{f:"#d9b46a",f2:"#cda65c",w:"#8c7a6a",w2:"#a8968a"}];
function rect(x,y,w,h,c){ctx.fillStyle=c;ctx.fillRect(x,y,w,h);}
function drawTile(x,y,sx,sy){
  const t=tiles[y][x],p=PAL[biomeOf(x)],h=hash(x,y),s=T;
  if(t===2){rect(sx,sy,s,s,"#3a7bd5");rect(sx+6,sy+10+((Date.now()/300+x)%3|0)*4,14,4,"#6fa6e8");rect(sx+26,sy+28,12,4,"#6fa6e8");return;}
  rect(sx,sy,s,s,h<.5?p.f:p.f2);
  if(biomeOf(x)===0){rect(sx+(h*30|0),sy+(h*97%30|0),6,6,"#454856");}
  if(biomeOf(x)===1){ctx.fillStyle="#3f8a28";for(let i=0;i<3;i++)ctx.fillRect(sx+((h*(i+3)*37)%40|0),sy+((h*(i+5)*53)%40|0),4,8);}
  if(biomeOf(x)===2){rect(sx+((h*41)%38|0),sy+((h*67)%38|0),6,4,"#c49a52");}
  if(t===1){
    const b=biomeOf(x);
    if(x==0||y==0||x==W-1||y==H-1||x==12||x==24){rect(sx,sy,s,s,"#1a1b22");rect(sx+4,sy+4,s-8,s-8,"#26272f");rect(sx+10,sy+14,10,8,"#33343e");return;}
    if(b===0){rect(sx+2,sy+2,s-4,s-4,p.w);rect(sx+6,sy+6,s-20,s-24,p.w2);rect(sx+20,sy+26,14,10,"#4a4d5a");}
    else if(b===1){rect(sx+18,sy+26,12,20,"#6d4528");rect(sx+4,sy+2,40,30,"#2f6b20");rect(sx+10,sy+6,14,10,"#3d8a2a");rect(sx+28,sy+16,8,6,"#e04848");}
    else{rect(sx+4,sy+10,40,34,p.w);rect(sx+8,sy+12,20,12,p.w2);rect(sx+30,sy+30,10,8,"#6f6052");}
  }
}
function drawObj(o,sx,sy){
  const s=T,g=BIOMES[o.b].gem,pulse=(Math.sin(Date.now()/300+o.x)+1)/2;
  if(o.lab){rect(sx+2,sy+2,s-4,s-4,"#8b5a2b");rect(sx+2,sy+2,s-4,12,"#b07a3c");rect(sx+8,sy+18,12,12,"#6b4220");rect(sx+28,sy+18,12,12,"#6b4220");
    rect(sx+16,sy+4,16,6,g);ctx.globalAlpha=.25+.35*pulse;rect(sx-3,sy-3,s+6,s+6,g);ctx.globalAlpha=1;
    if(o.state==="done"){rect(sx+14,sy+30,20,12,"#2c6b36");ctx.fillStyle="#fff";ctx.font="bold 12px sans-serif";ctx.fillText("✓",sx+19,sy+41);}
    else{ctx.fillStyle="#fff";ctx.font="bold 14px 'Pixelify Sans',monospace";ctx.fillText("LAB",sx+12,sy+44);}
    return;}
  const base=["#7d7f8a","#8a5a35","#b8904a"][o.b];
  rect(sx+2,sy+2,s-4,s-4,base);rect(sx+2,sy+2,s-4,5,"rgba(255,255,255,.18)");rect(sx+2,sy+s-7,s-4,5,"rgba(0,0,0,.25)");
  if(o.b===1){rect(sx+2,sy+2,s-4,12,"#5aa83c");}
  const spots=[[10,14],[28,10],[18,28],[32,30],[8,32]];
  ctx.globalAlpha=.75+.25*pulse;spots.forEach(([a,b])=>{rect(sx+a,sy+b,7,7,g);rect(sx+a+1,sy+b+1,3,3,"#fff");});ctx.globalAlpha=1;
  const cracks=(o.state==="cracked"?2:0)+(P.target===o?P.mining:0);
  if(cracks){ctx.strokeStyle="rgba(10,10,10,.8)";ctx.lineWidth=3;ctx.beginPath();
    ctx.moveTo(sx+8,sy+8);ctx.lineTo(sx+20,sy+20);ctx.lineTo(sx+16,sy+34);
    if(cracks>1){ctx.moveTo(sx+40,sy+10);ctx.lineTo(sx+28,sy+22);ctx.lineTo(sx+36,sy+40);}
    if(cracks>2){ctx.moveTo(sx+6,sy+40);ctx.lineTo(sx+22,sy+30);ctx.lineTo(sx+42,sy+28);}ctx.stroke();}
}
function drawPlayer(sx,sy){
  const s=T,[fx,fy]=P.face,bob=P.moving?Math.sin(Date.now()/60)*2:0;
  rect(sx+12,sy+38+bob*0,10,8,"#2f3a4f");rect(sx+26,sy+38,10,8,"#2f3a4f");
  rect(sx+10,sy+22,28,18,"#3f8f5a");rect(sx+10,sy+22,28,4,"#4fa86c");
  rect(sx+11,sy+4+bob,26,20,"#a86b3c");
  rect(sx+9,sy+1+bob,30,8,"#f08a24");rect(sx+20,sy-2+bob,8,4,"#f08a24");rect(sx+21,sy+2+bob,6,4,"#ffd54a");
  if(fy!==-1){const ex=fx*4;rect(sx+16+ex,sy+12+bob,5,5,"#fff");rect(sx+27+ex,sy+12+bob,5,5,"#fff");rect(sx+18+ex,sy+13+bob,3,4,"#3b2314");rect(sx+29+ex,sy+13+bob,3,4,"#3b2314");rect(sx+20+ex,sy+19+bob,8,2,"#6e3d1e");}
  else rect(sx+11,sy+8+bob,26,6,"#5a3a20");
  // pickaxe
  ctx.save();ctx.translate(sx+s/2+fx*16,sy+28+fy*12);ctx.rotate((fx||1)*(-.6+Math.sin(P.swing)*1.1));
  rect(-2,-18,4,22,"#8b5a2b");rect(-12,-22,24,6,"#9aa3b5");rect(-12,-22,4,10,"#9aa3b5");rect(8,-22,4,10,"#9aa3b5");ctx.restore();
}
function frame(){
  const k=.25;P.rx+=(P.x-P.rx)*k;P.ry+=(P.y-P.ry)*k;
  if(P.swing>0)P.swing=Math.max(0,P.swing-.25);
  const vw=cv.width/scale,vh=cv.height/scale;
  cam.x=Math.max(0,Math.min(W*T-vw,P.rx*T+T/2-vw/2));cam.y=Math.max(0,Math.min(H*T-vh,P.ry*T+T/2-vh/2));
  if(W*T<vw)cam.x=(W*T-vw)/2;if(H*T<vh)cam.y=(H*T-vh)/2;
  ctx.setTransform(scale,0,0,scale,0,0);rect(0,0,vw,vh,"#0b0e18");ctx.translate(-Math.round(cam.x),-Math.round(cam.y));
  const x0=Math.max(0,Math.floor(cam.x/T)),x1=Math.min(W,Math.ceil((cam.x+vw)/T)+1),y0=Math.max(0,Math.floor(cam.y/T)),y1=Math.min(H,Math.ceil((cam.y+vh)/T)+1);
  for(let y=y0;y<y1;y++)for(let x=x0;x<x1;x++)drawTile(x,y,x*T,y*T);
  objs.forEach(o=>{if(o.state!=="mined")drawObj(o,o.x*T,o.y*T);});
  // biome signs
  ctx.font="bold 16px 'Pixelify Sans',monospace";
  [[1,"CELL CAVE"],[13,"PLANT PLAINS"],[25,"ANIMAL ARMOR HILLS"]].forEach(([x,t])=>{const w=ctx.measureText(t).width+16;rect(x*T+4,T*14+10,w,26,"rgba(11,14,24,.85)");ctx.fillStyle="#f2c14e";ctx.fillText(t,x*T+12,T*14+29);});
  // facing highlight
  const fx=P.x+P.face[0],fy=P.y+P.face[1],fo=objAt(fx,fy);
  if(fo){ctx.strokeStyle="#f2c14e";ctx.lineWidth=3;ctx.strokeRect(fx*T+1,fy*T+1,T-2,T-2);}
  drawPlayer(P.rx*T,P.ry*T);
  parts=parts.filter(p=>(p.life-=1)>0);parts.forEach(p=>{p.x+=p.vx;p.y+=p.vy;p.vy+=.35;rect(p.x,p.y,p.s,p.s,p.c);});
  floats=floats.filter(f=>(f.life-=1)>0);ctx.font="bold 22px 'Pixelify Sans',monospace";floats.forEach(f=>{f.y-=.8;ctx.globalAlpha=Math.min(1,f.life/30);ctx.fillStyle="#0b0e18";ctx.fillText(f.t,f.x+2,f.y+2);ctx.fillStyle=f.c;ctx.fillText(f.t,f.x,f.y);ctx.globalAlpha=1;});
  requestAnimationFrame(frame);
}
function burst(x,y,c,n=14){for(let i=0;i<n;i++)parts.push({x:x*T+T/2,y:y*T+T/2,vx:(Math.random()-.5)*7,vy:-Math.random()*6,s:4+Math.random()*5,c,life:30+Math.random()*20});}
/* ---------- movement ---------- */
function status(){
  const b=biomeOf(P.x),left=objs.filter(o=>o.b===b&&!o.lab&&o.state!=="mined").length,lab=objs.find(o=>o.b===b&&o.lab);
  document.getElementById("status").textContent=`${BIOMES[b].name} · ${10-left}/10 gems · Lab ${lab.state==="done"?"done":"open"}`;
}
function step(dx,dy){
  if(busy())return;P.face=[dx,dy];const nx=P.x+dx,ny=P.y+dy;
  if(walkable(nx,ny)){P.x=nx;P.y=ny;sfx.step();P.moving=true;clearTimeout(P.mt);P.mt=setTimeout(()=>P.moving=false,200);status();save();}
}
function busy(){return !document.getElementById("scrim").hidden||P.mining>0;}
function walkPath(path,then){
  P.path=path;clearInterval(P.iv);
  if(!path.length){then&&then();return;}
  P.iv=setInterval(()=>{if(!P.path.length){clearInterval(P.iv);then&&then();return;}
    const [x,y]=P.path.shift();P.face=[x-P.x,y-P.y];P.x=x;P.y=y;P.moving=true;sfx.step();status();save();
    if(!P.path.length){setTimeout(()=>P.moving=false,150);}},130);
}
function goTo(tx,ty){
  if(busy())return;const o=objAt(tx,ty);
  if(!o){if(!walkable(tx,ty))return;const d=bfs(P.x,P.y),k=ty*W+tx;if(!d.has(k))return;walkPath(trace(d,tx,ty));return;}
  const adj=[[1,0],[-1,0],[0,1],[0,-1]].map(([dx,dy])=>[o.x+dx,o.y+dy]);
  if(adj.some(([x,y])=>x===P.x&&y===P.y)){P.face=[o.x-P.x,o.y-P.y];mine();return;}
  const d=bfs(P.x,P.y);let best=null;
  adj.forEach(([x,y])=>{if(d.has(y*W+x)){const p=trace(d,x,y);if(!best||p.length<best.length)best=p;}});
  if(best)walkPath(best,()=>{P.face=[o.x-P.x,o.y-P.y];mine();});
}
function trace(d,x,y){const p=[];let c=[x,y];while(c&&!(c[0]===P.x&&c[1]===P.y)){p.unshift(c);c=d.get(c[1]*W+c[0]);}return p;}
function mine(){
  if(busy())return;const o=objAt(P.x+P.face[0],P.y+P.face[1]);if(!o)return;
  if(o.lab){burst(o.x,o.y,BIOMES[o.b].gem,8);sfx.hit();openLab(o);return;}
  P.target=o;let hits=0;
  const hit=()=>{hits++;P.mining=hits;P.swing=Math.PI;sfx.hit();burst(o.x,o.y,["#7d7f8a","#8a5a35","#b8904a"][o.b],8);
    if(hits<3)setTimeout(hit,220);else setTimeout(()=>{P.mining=0;openQ(o);},250);};
  hit();
}
/* ---------- question ---------- */
let cur=null;
function shuffle(a){a=a.slice();for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];}return a;}
function who(){return S.teams===2?`Team ${S.turn+1}'s turn`:"";}
function openQ(o){
  const b=BIOMES[o.b],q=b.qs[o.q];cur={o,answered:false,opts:shuffle(q[1])};
  card(`<div class="head"><span>${b.name} ore</span><span>${who()}</span></div>
   <div class="body"><span class="std">${b.std} · ${o.state==="cracked"?"Retry for 5 points":"10 points"}</span>
   <p class="q" id="ctitle">${q[0]}</p>
   <div class="opts">${cur.opts.map((t,i)=>`<button class="opt" data-i="${i}" id="opt${i}"><kbd>${i+1}</kbd><span>${t}</span></button>`).join("")}</div>
   <div id="fb"></div></div>`);
  document.getElementById("opt0").focus();
}
function answer(i){
  if(!cur||cur.answered)return;cur.answered=true;const {o,opts}=cur,q=BIOMES[o.b].qs[o.q],ok=opts[i]===q[1][0];
  document.querySelectorAll(".opt").forEach((el,k)=>{el.disabled=true;if(opts[k]===q[1][0])el.classList.add("right");else if(k===i)el.classList.add("wrong");});
  if(ok){const pts=o.state==="cracked"?5:10;S.score[S.turn]+=pts;o.state="mined";S.gems[o.b]++;burst(o.x,o.y,BIOMES[o.b].gem,22);
    floats.push({x:o.x*T,y:o.y*T,t:"+"+pts,c:"#f2c14e",life:60});sfx.good();}
  else{o.state="cracked";sfx.bad();floats.push({x:o.x*T,y:o.y*T,t:"Cracked!",c:"#ef6a5b",life:60});}
  document.getElementById("fb").innerHTML=`<div class="fb ${ok?"good":"miss"}"><strong>${ok?"Gem collected!":"Block cracked!"}</strong><span>${ok?"":"The answer is: "+q[1][0]+". "}${q[2]}</span></div>
    <button class="btn" id="next">${S.teams===2?"Next team":"Keep exploring"}</button>`;
  document.getElementById("next").onclick=()=>closeCard(true);document.getElementById("next").focus();
  save();hud();
}
/* ---------- labs ---------- */
let lab=null;
function openLab(o){
  if(o.state==="done"){toast("Lab already cleared!");return;}
  const L=LABS[o.b];lab={o,L,place:{},sel:null,wrongs:0,items:shuffle(L.items.map((it,i)=>i))};
  renderLab();
}
function renderLab(){
  const {L}=lab,placed=i=>lab.place[i]!==undefined;
  const chip=i=>`<button class="tile ${lab.sel===i?"sel":""} ${lab.lock?.[i]?"lock":""}" data-t="${i}">${L.items[i][0]}</button>`;
  const pool=lab.items.filter(i=>!placed(i)).map(chip).join("")||`<span class="std">All placed. Press Check!</span>`;
  let area;
  if(L.kind==="sort"){
    area=`<div class="zones">${L.zones.map((z,zi)=>`<button class="zone ${lab.sel!==null?"target":""}" data-z="${zi}"><h3>${z}</h3><div class="in">${lab.items.filter(i=>lab.place[i]===zi).map(chip).join("")}</div></button>`).join("")}</div>`;
  }else{
    area=`<div class="diagram">${plantSVG()}<div class="slots">${L.slots.map((s,si)=>{const i=lab.items.find(i=>lab.place[i]===si);
      return `<button class="slotrow ${lab.sel!==null?"target":""}" data-z="${si}"><span class="L">${s}</span>${i!==undefined?chip(i)+(lab.lock?.[i]?`<span class="std">${L.jobs[si]}</span>`:""):`<span class="std">empty</span>`}</button>`;}).join("")}</div></div>`;
  }
  card(`<div class="head"><span>${L.title}</span><span>${who()}</span></div>
   <div class="body"><span class="std">${BIOMES[lab.o.b].std} · 30 points</span><p class="q" id="ctitle">${L.how}</p>
   <div class="pool">${pool}</div>${area}
   <div id="lfb"></div>
   <div class="row"><button class="btn" id="check">Check</button><button class="btn" id="later">Come back later</button></div></div>`);
  document.getElementById("check").onclick=checkLab;
  document.getElementById("later").onclick=()=>{lab=null;closeCard(false);};
}
function plantSVG(){return `<svg viewBox="0 0 200 260" role="img" aria-label="Plant diagram with parts labeled A to E">
  <rect x="0" y="190" width="200" height="70" fill="#6e4527"/><rect x="0" y="186" width="200" height="8" fill="#5aa83c"/>
  <path d="M100 190 L100 250 M100 205 L70 245 M100 210 L135 248 M100 225 L82 255" stroke="#e8d3a8" stroke-width="5" fill="none"/>
  <rect x="95" y="70" width="10" height="120" fill="#3f8a28"/>
  <ellipse cx="62" cy="130" rx="34" ry="14" fill="#5fd068" transform="rotate(-20 62 130)"/><path d="M95 138 L40 120" stroke="#3f8a28" stroke-width="3"/>
  <ellipse cx="140" cy="150" rx="30" ry="12" fill="#5fd068" transform="rotate(20 140 150)"/>
  <circle cx="100" cy="58" r="10" fill="#f2c14e"/><g fill="#e868a0"><circle cx="100" cy="38" r="12"/><circle cx="120" cy="58" r="12"/><circle cx="80" cy="58" r="12"/><circle cx="100" cy="78" r="9"/></g><circle cx="100" cy="58" r="9" fill="#f2c14e"/>
  <circle cx="148" cy="100" r="13" fill="#e04848"/><path d="M148 87 L140 80" stroke="#3f8a28" stroke-width="3"/><path d="M105 110 L140 96" stroke="#3f8a28" stroke-width="4"/>
  <g font-family="Pixelify Sans,monospace" font-size="20" font-weight="700" fill="#0b0e18">
  ${[["A",130,30],["B",30,112],["C",170,96],["D",112,178],["E",150,240]].map(([t,x,y])=>`<rect x="${x-12}" y="${y-18}" width="24" height="24" fill="#f2c14e" stroke="#0b0e18" stroke-width="3"/><text x="${x-6}" y="${y}">${t}</text>`).join("")}</g></svg>`;}
function labClick(e){
  const t=e.target.closest(".tile"),z=e.target.closest("[data-z]");
  if(t&&!t.classList.contains("lock")&&(!z||lab.sel===null||+t.dataset.t===lab.sel)){const i=+t.dataset.t;
    if(lab.place[i]!==undefined&&lab.sel===i){delete lab.place[i];lab.sel=null;}else lab.sel=lab.sel===i?null:i;sfx.step();renderLab();return;}
  if(z&&lab.sel!==null){const zi=+z.dataset.z;
    if(lab.L.kind==="label"){const prev=lab.items.find(i=>lab.place[i]===zi);if(prev!==undefined&&!lab.lock?.[prev])delete lab.place[prev];else if(prev!==undefined)return;}
    lab.place[lab.sel]=zi;lab.sel=null;sfx.hit();renderLab();}
}
function checkLab(){
  const {L}=lab;lab.lock=lab.lock||{};let wrong=[],all=true;
  L.items.forEach((it,i)=>{if(lab.place[i]===undefined){all=false;return;}if(lab.place[i]===it[1])lab.lock[i]=true;else{wrong.push(i);delete lab.place[i];}});
  if(wrong.length){lab.wrongs++;sfx.bad();}
  const done=L.items.every((it,i)=>lab.lock[i]);
  renderLab();
  wrong.forEach(i=>document.querySelector(`.tile[data-t="${i}"]`)?.classList.add("shake"));
  const fb=document.getElementById("lfb");
  if(done){const o=lab.o;o.state="done";S.score[S.turn]+=30;sfx.good();burst(o.x,o.y,BIOMES[o.b].gem,26);floats.push({x:o.x*T,y:o.y*T,t:"+30",c:"#f2c14e",life:70});
    fb.innerHTML=`<div class="fb good"><strong>Lab cleared!</strong><span>${L.done}</span></div>`;
    document.getElementById("check").textContent=S.teams===2?"Next team":"Keep exploring";document.getElementById("check").onclick=()=>{lab=null;closeCard(true);};
    document.getElementById("later").remove();save();hud();}
  else if(wrong.length)fb.innerHTML=`<div class="fb miss"><strong>${wrong.length} went back to the pile</strong><span>Green ones are correct and locked in. Try the others again.</span></div>`;
  else if(!all)fb.innerHTML=`<div class="fb miss"><strong>Keep going</strong><span>Everything you placed is right. Place the rest, then check again.</span></div>`;
}
/* ---------- cards, HUD ---------- */
function card(html){document.getElementById("card").innerHTML=html;document.getElementById("scrim").hidden=false;}
function closeCard(turnDone){
  document.getElementById("scrim").hidden=true;cur=null;
  if(turnDone&&S.teams===2)S.turn=1-S.turn;
  for(let b=0;b<3;b++){if(!S.tools[b]&&S.gems[b]===10&&objs.find(o=>o.b===b&&o.lab).state==="done"){S.tools[b]=true;sfx.craft();toast(`Crafted: ${BIOMES[b].tool}! Code letters: ${BIOMES[b].code}`);}}
  save();hud();status();cv.focus();
  if(S.tools.every(Boolean)&&!S.finished){S.finished=true;setTimeout(showEnd,1600);}
}
function hud(){
  document.getElementById("scores").innerHTML=(S.teams===2?S.score.map((s,i)=>`<span class="chip ${S.turn===i?"turn":""}">Team ${i+1} <b>${s}</b></span>`).join(""):`<span class="chip">Score <b>${S.score[0]}</b></span>`);
  document.getElementById("hotbar").innerHTML=BIOMES.map((b,i)=>`<div class="slot" title="${b.name} gems"><span class="g" style="background:${b.gem}"></span><span class="n">${S.gems[i]}</span></div>`).join("")+
    BIOMES.map((b,i)=>`<div class="slot ${S.tools[i]?"got":""}" title="${b.tool}"><span class="t">${S.tools[i]?b.tool:"?"}</span></div>`).join("");
  const word=BIOMES.map((b,i)=>b.code.split("").map(ch=>S.tools[i]?ch:"")).flat();
  document.getElementById("codebar").innerHTML=`<span class="lbl">Secret code</span>`+word.map(ch=>`<span class="c">${ch||"?"}</span>`).join("");
  document.getElementById("mode1").classList.toggle("on",S.teams===1);document.getElementById("mode2").classList.toggle("on",S.teams===2);
  status();
}
let tt;function toast(t){const el=document.getElementById("toast");el.textContent=t;el.classList.add("show");clearTimeout(tt);tt=setTimeout(()=>el.classList.remove("show"),2200);}
function showEnd(){
  const msg=S.teams===2?(S.score[0]===S.score[1]?"It's a tie!":`Team ${S.score[0]>S.score[1]?1:2} wins!`):`You scored ${S.score[0]} points!`;
  card(`<div class="head"><span>World complete</span><span>3 tools crafted</span></div>
   <div class="body end"><h2 id="ctitle">${msg}</h2><p>You crafted the Microscope Pickaxe, the Golden Seed Hoe and the Armor Chestplate. Your secret code is:</p><div class="bigcode">SURVIVE</div><p>Plants and animals have structures, inside and out, that help them <strong>survive</strong>, grow and reproduce. Write the code and your score on your exit ticket.</p>
   <button class="btn" id="again">Play a new world</button></div>`);sfx.craft();
  document.getElementById("again").onclick=()=>{newWorld(S.teams);document.getElementById("scrim").hidden=true;};
}
function newWorld(teams){clearInterval(P.iv);P.path=[];S=fresh(teams);P.x=P.rx=S.px;P.y=P.ry=S.py;P.face=[0,1];hud();}
/* ---------- input ---------- */
const KEYS={ArrowUp:[0,-1],ArrowDown:[0,1],ArrowLeft:[-1,0],ArrowRight:[1,0],w:[0,-1],s:[0,1],a:[-1,0],d:[1,0],W:[0,-1],S:[0,1],A:[-1,0],D:[1,0]};
document.addEventListener("keydown",e=>{
  if(!document.getElementById("scrim").hidden){
    if(cur&&!cur.answered&&/^[1-4]$/.test(e.key)){answer(+e.key-1);e.preventDefault();}
    return;}
  if(e.target.closest&&e.target.closest("button")&&e.key===" ")return;
  if(KEYS[e.key]){clearInterval(P.iv);step(...KEYS[e.key]);e.preventDefault();}
  else if(e.key===" "||e.key==="Enter"){mine();e.preventDefault();}
});
cv.addEventListener("pointerdown",e=>{const r=cv.getBoundingClientRect(),dpr=cv.width/r.width;
  const x=Math.floor(((e.clientX-r.left)*dpr/scale+cam.x)/T),y=Math.floor(((e.clientY-r.top)*dpr/scale+cam.y)/T);goTo(x,y);cv.focus();});
document.querySelectorAll("[data-d]").forEach(b=>b.addEventListener("click",()=>{clearInterval(P.iv);step(...b.dataset.d.split(",").map(Number));}));
document.getElementById("mineBtn").onclick=mine;
document.getElementById("card").addEventListener("click",e=>{const o=e.target.closest(".opt");if(o&&cur){answer(+o.dataset.i);return;}if(lab&&!lab.o.state.includes("done"))labClick(e);});
document.getElementById("mode1").onclick=()=>newWorld(1);
document.getElementById("mode2").onclick=()=>newWorld(2);
document.getElementById("reset").onclick=()=>newWorld(S.teams);
document.getElementById("sound").onclick=e=>{soundOn=!soundOn;e.target.textContent="Sound: "+(soundOn?"on":"off");};
window.addEventListener("resize",resize);
window.claude?.hot?.snapshot?.(()=>{save();return S;});
function start(d){load(d);resize();requestAnimationFrame(frame);}
window.claude?.hot?.ready ? window.claude.hot.ready(start) : start(window.claude?.hot?.data ?? {});
</script>

</body>
</html>
