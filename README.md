# shubham
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>StudyBuddy – Your Math & Science Tutor</title>
<style>
/* ===== COLORS (light + dark). Edit these to restyle the page ===== */
:root{--bg1:#e8f1ff;--bg2:#fff3e0;--panel:#ffffff;--text:#1b2540;--muted:#5d6b8a;--user:#003893;--userText:#fff;--bot:#eef2fb;--accent:#dc143c;--line:#d5def0}
:root[data-theme="dark"]{--bg1:#0c1230;--bg2:#2a1535;--panel:#161d3d;--text:#e9edff;--muted:#9aa7cf;--user:#3d6bff;--userText:#fff;--bot:#212a52;--accent:#ff5c7a;--line:#2c366a}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg1:#0c1230;--bg2:#2a1535;--panel:#161d3d;--text:#e9edff;--muted:#9aa7cf;--user:#3d6bff;--userText:#fff;--bot:#212a52;--accent:#ff5c7a;--line:#2c366a}}
*{box-sizing:border-box}
html,body{height:100%;margin:0}
html{padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
body{font-family:"Trebuchet MS","Segoe UI",system-ui,sans-serif;color:var(--text);background:linear-gradient(135deg,var(--bg1),var(--bg2));display:flex;flex-direction:row;gap:14px;padding:14px;overflow:hidden}
#stage{flex:1 1 42%;position:relative;border-radius:22px;overflow:hidden;min-width:0}
#stage canvas{display:block;width:100%;height:100%}
#title{position:absolute;left:18px;top:14px;font-size:1.5rem;font-weight:800;letter-spacing:.3px}
#title small{display:block;font-size:.8rem;font-weight:400;color:var(--muted)}
#credit{position:absolute;left:0;right:0;bottom:10px;text-align:center;font-size:.85rem;color:var(--muted)}
#theme{position:absolute;right:12px;top:12px;border:1px solid var(--line);background:var(--panel);color:var(--text);border-radius:20px;padding:6px 12px;cursor:pointer;font:inherit;font-size:.85rem}
#chat{flex:1 1 58%;display:flex;flex-direction:column;background:var(--panel);border-radius:22px;border:1px solid var(--line);min-width:0;min-height:0}
#log{flex:1;overflow-y:auto;padding:16px;display:flex;flex-direction:column;gap:10px;scroll-behavior:smooth}
.msg{max-width:88%;padding:10px 14px;border-radius:16px;line-height:1.55;font-size:1rem;word-wrap:break-word}
.msg.bot{background:var(--bot);align-self:flex-start;border-bottom-left-radius:4px}
.msg.user{background:var(--user);color:var(--userText);align-self:flex-end;border-bottom-right-radius:4px}
.msg sup,.msg sub{font-size:.7em}
.msg .step{display:block;margin:4px 0}
#think{display:none;padding:0 18px 6px;color:var(--muted);font-size:.9rem}
#think.on{display:block}
#think i{display:inline-block;width:6px;height:6px;border-radius:50%;background:var(--accent);margin-right:3px;animation:dot 1s infinite}
#think i:nth-child(2){animation-delay:.2s}#think i:nth-child(3){animation-delay:.4s}
@keyframes dot{0%,100%{opacity:.2}50%{opacity:1}}
#quick{display:flex;gap:8px;flex-wrap:wrap;padding:0 14px 8px}
#quick button{border:1px solid var(--line);background:transparent;color:var(--text);border-radius:18px;padding:6px 12px;cursor:pointer;font:inherit;font-size:.85rem}
#quick button:hover{border-color:var(--accent)}
#form{display:flex;gap:8px;padding:10px 14px 14px;border-top:1px solid var(--line)}
#box{flex:1;border:1px solid var(--line);background:var(--bg1);color:var(--text);border-radius:14px;padding:11px 14px;font:inherit;font-size:1rem;min-width:0}
#send{border:0;background:var(--accent);color:#fff;border-radius:14px;padding:0 20px;font:inherit;font-weight:700;cursor:pointer}
#send:disabled{opacity:.5}
button:focus-visible,#box:focus-visible{outline:3px solid var(--accent);outline-offset:2px}
/* Phones: robot on top, chat below */
@media (max-width:760px){body{flex-direction:column;padding:8px;gap:8px}#stage{flex:0 0 32%}#chat{flex:1 1 68%}#title{font-size:1.15rem}}
</style>
</head>
<body>
<div id="stage">
  <div id="title">StudyBuddy<small>Free math &amp; science help</small></div>
  <button id="theme" aria-label="Switch light or dark mode">Dark / Light</button>
  <div id="credit"></div>
</div>
<section id="chat">
  <div id="log" aria-live="polite"></div>
  <div id="think"><i></i><i></i><i></i> StudyBuddy is thinking...</div>
  <div id="quick">
    <button data-q="I need help with a math problem.">Math help</button>
    <button data-q="I need help with a science question (physics, chemistry or biology).">Science help</button>
    <button data-q="Give me a practice question for my grade. Ask me my grade first if you don't know it.">Give me a practice question</button>
  </div>
  <div id="form">
    <input id="box" type="text" placeholder="Type your question here..." autocomplete="off">
    <button id="send">Send</button>
  </div>
</section>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
/* =====================================================
   1. SETTINGS – change these to customize StudyBuddy
   ===================================================== */
const TUTOR_NAME   = "StudyBuddy";
const CREATOR_NAME = "Shubham Acharya";      // <-- your name, shown on the page and used by the tutor
const WORKER_URL   = "";   // <-- paste your Cloudflare Worker link here, e.g. https://studybuddy.yourname.workers.dev

// Robot colors (any hex color)
const ROBOT_BODY   = 0xf4f7ff;   // main shell
const ROBOT_ACCENT = 0xdc143c;   // arms, ears, belly light
const ROBOT_FACE   = 0x0d1533;   // dark face screen
const ROBOT_EYES   = 0x39e6ff;   // glowing eyes
const ROBOT_ANTENNA= 0xffc233;   // antenna light

const GREETING = "Namaste! I am " + TUTOR_NAME + ". Ask me any school math, physics, chemistry or biology question and I will solve it step by step. 😊";

// The tutor's rules (system prompt) now live in worker.js, so strangers cannot change them.

/* =====================================================
   2. CHAT LOGIC
   ===================================================== */
const log = document.getElementById("log"), box = document.getElementById("box");
const sendBtn = document.getElementById("send"), thinkEl = document.getElementById("think");
let history = [];          // full conversation sent to the API every time
let mood = "idle";         // idle | thinking | talking  (read by the 3D animation)
let happyUntil = 0;        // time (ms) until the happy bounce ends

// Turn plain AI text into safe HTML with powers, subscripts, roots and steps
function formatMath(text){
  let s = text.replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;");
  s = s.replace(/\*\*(.+?)\*\*/g,"<b>$1</b>");
  s = s.replace(/sqrt\(([^()]+)\)/g,"√($1)");
  s = s.replace(/\^(\{[^}]+\}|-?[A-Za-z0-9]+)/g,(m,p)=>"<sup>"+p.replace(/[{}]/g,"")+"</sup>");
  s = s.replace(/_(\{[^}]+\}|[A-Za-z0-9]+)/g,(m,p)=>"<sub>"+p.replace(/[{}]/g,"")+"</sub>");
  s = s.replace(/\((\d+)\)\/\((\d+)\)/g,"<sup>$1</sup>⁄<sub>$2</sub>");
  return s.split("\n").map(l=>/^\s*(Step \d+|Key idea)/i.test(l)?'<span class="step">'+l+"</span>":l).join("<br>");
}
function addMsg(who, html){
  const d = document.createElement("div");
  d.className = "msg " + who; d.innerHTML = html;
  log.appendChild(d); log.scrollTop = log.scrollHeight; return d;
}
// Show the reply gradually while the robot "talks"
function typeReply(el, text){
  return new Promise(res=>{
    let i = 0; const step = Math.max(3, Math.ceil(text.length/150)); mood = "talking";
    (function tick(){
      i = Math.min(text.length, i + step);
      el.innerHTML = formatMath(text.slice(0,i)); log.scrollTop = log.scrollHeight;
      if(i < text.length) setTimeout(tick, 20); else { mood = "idle"; res(); }
    })();
  });
}
async function ask(text){
  text = text.trim(); if(!text) return;
  addMsg("user", formatMath(text)); box.value = ""; sendBtn.disabled = true;
  history.push({role:"user", content:text});
  mood = "thinking"; thinkEl.classList.add("on");
  try{
    if(!WORKER_URL) throw new Error("NO_AI");
    // Send the conversation to YOUR Worker; the Worker adds the secret key and calls the AI
    const r = await fetch(WORKER_URL,{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({messages:history})});
    if(!r.ok) throw new Error("HTTP " + r.status);
    let reply = (await r.json()).reply;
    history.push({role:"assistant", content:reply});
    const correct = reply.trim().startsWith("[CORRECT]");
    reply = reply.replace("[CORRECT]","").trim();
    thinkEl.classList.remove("on");
    if(correct) happyUntil = performance.now() + 1600;
    await typeReply(addMsg("bot",""), reply);
  }catch(err){
    history.pop();   // remove the failed question so the history stays valid
    thinkEl.classList.remove("on"); mood = "idle";
    addMsg("bot", err.message==="NO_AI"
      ? "Please paste your Worker link into the <b>WORKER_URL</b> setting at the top of the code, then reload."
      : "Sorry, I could not reach my brain right now. Please check your internet, then try again. 🙏");
  }
  sendBtn.disabled = false; box.focus();
}
sendBtn.onclick = ()=>ask(box.value);
box.addEventListener("keydown",e=>{ if(e.key==="Enter") ask(box.value); });
document.querySelectorAll("#quick button").forEach(b=>b.onclick=()=>ask(b.dataset.q));
document.getElementById("theme").onclick = ()=>{
  const root = document.documentElement;
  const dark = root.dataset.theme ? root.dataset.theme==="dark" : matchMedia("(prefers-color-scheme:dark)").matches;
  root.dataset.theme = dark ? "light" : "dark";
};
document.getElementById("credit").textContent = "Created by " + CREATOR_NAME + ", Nepal";
addMsg("bot", formatMath(GREETING));

/* =====================================================
   3. 3D ROBOT (Three.js)
   ===================================================== */
const stage = document.getElementById("stage");
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(40,1,0.1,100); camera.position.set(0,0.4,9);
const renderer = new THREE.WebGLRenderer({antialias:true, alpha:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,2)); stage.appendChild(renderer.domElement);

// Soft lighting
scene.add(new THREE.HemisphereLight(0xffffff,0x8899cc,0.9));
const key = new THREE.DirectionalLight(0xffffff,0.8); key.position.set(3,5,6); scene.add(key);
const rim = new THREE.PointLight(0xff8aa5,0.6,20); rim.position.set(-4,2,3); scene.add(rim);

const mat = (c,o={})=>new THREE.MeshStandardMaterial(Object.assign({color:c,roughness:0.35,metalness:0.1},o));
const glow = c=>new THREE.MeshStandardMaterial({color:c,emissive:c,emissiveIntensity:1.2});
function mesh(geo,m,x=0,y=0,z=0,parent=robot){ const o=new THREE.Mesh(geo,m); o.position.set(x,y,z); parent.add(o); return o; }

const robot = new THREE.Group(); scene.add(robot);
// Body, belly light, arms
const body = mesh(new THREE.SphereGeometry(1,32,24),mat(ROBOT_BODY),0,-1.7,0); body.scale.set(0.95,1.05,0.85);
mesh(new THREE.CylinderGeometry(0.25,0.25,0.06,24),glow(ROBOT_ACCENT),0,-1.65,0.82).rotation.x = Math.PI/2;
const armL = mesh(new THREE.SphereGeometry(0.28,20,16),mat(ROBOT_ACCENT),-1.15,-1.5,0);
const armR = mesh(new THREE.SphereGeometry(0.28,20,16),mat(ROBOT_ACCENT), 1.15,-1.5,0);
mesh(new THREE.CylinderGeometry(0.3,0.4,0.25,24),mat(0x9aa7cf),0,-0.7,0);   // neck
// Head group (rotates to follow the mouse)
const head = new THREE.Group(); head.position.y = 0.35; robot.add(head);
const skull = mesh(new THREE.SphereGeometry(1.35,40,32),mat(ROBOT_BODY),0,0,0,head); skull.scale.set(1.1,0.95,1);
const face = mesh(new THREE.SphereGeometry(1,32,24),mat(ROBOT_FACE,{roughness:0.15,metalness:0.4}),0,0,0.55,head); face.scale.set(1.15,0.78,0.85);
const eyeL = mesh(new THREE.SphereGeometry(0.2,20,16),glow(ROBOT_EYES),-0.5,0.18,1.28,head);
const eyeR = mesh(new THREE.SphereGeometry(0.2,20,16),glow(ROBOT_EYES), 0.5,0.18,1.28,head);
const mouth = mesh(new THREE.BoxGeometry(0.5,0.08,0.06),glow(ROBOT_EYES),0,-0.35,1.3,head);
mesh(new THREE.CylinderGeometry(0.2,0.2,0.3,20),mat(ROBOT_ACCENT),-1.5,0,0,head).rotation.z = Math.PI/2;
mesh(new THREE.CylinderGeometry(0.2,0.2,0.3,20),mat(ROBOT_ACCENT), 1.5,0,0,head).rotation.z = Math.PI/2;
mesh(new THREE.CylinderGeometry(0.05,0.05,0.7,12),mat(0x9aa7cf),0,1.65,0,head);
const bulb = mesh(new THREE.SphereGeometry(0.17,20,16),glow(ROBOT_ANTENNA),0,2.1,0,head);

// Floating symbols drawn on canvases (no image files needed)
const symbols = [];
["π","+","∑","H₂O","⚛","√","x²","DNA"].forEach((t,i)=>{
  const c = document.createElement("canvas"); c.width = c.height = 128;
  const g = c.getContext("2d"); g.font = "bold 62px Georgia,serif"; g.textAlign = "center"; g.textBaseline = "middle";
  g.fillStyle = i%2 ? "#ff7a93" : "#6c8cff"; g.fillText(t,64,64);
  const s = new THREE.Sprite(new THREE.SpriteMaterial({map:new THREE.CanvasTexture(c),transparent:true,opacity:0.75}));
  s.scale.set(1.1,1.1,1); s.userData = {a:i/8*Math.PI*2, r:3.4+(i%3)*0.5, y:(i%4-1.5)*1.3, sp:0.15+0.05*(i%3)};
  scene.add(s); symbols.push(s);
});

// Mouse / touch position, -1 to 1
const ptr = {x:0,y:0};
addEventListener("pointermove",e=>{ ptr.x = e.clientX/innerWidth*2-1; ptr.y = -(e.clientY/innerHeight*2-1); });

function resize(){
  const w = stage.clientWidth, h = stage.clientHeight; renderer.setSize(w,h); camera.aspect = w/h;
  camera.position.z = w/h < 0.9 ? 11 : (w/h < 1.4 ? 9.5 : 8.5); camera.updateProjectionMatrix();
}
addEventListener("resize",resize); resize();

/* =====================================================
   4. ANIMATION LOOP – idle, thinking, talking, happy
   ===================================================== */
let nextBlink = 2, blinkT = -1;
const clock = new THREE.Clock();
(function animate(){
  requestAnimationFrame(animate);
  const t = clock.getElapsedTime(), now = performance.now();
  const happy = now < happyUntil;

  // Floating (idle) + happy bounce
  robot.position.y = Math.sin(t*1.6)*0.12 + (happy ? Math.abs(Math.sin(t*10))*0.7 : 0);
  armL.position.y = armR.position.y = -1.5 + (happy ? Math.sin(t*14)*0.25 : Math.sin(t*1.6)*0.05);

  // Head follows pointer, with slow idle sway and a tilt while thinking
  const tx = -ptr.y*0.35 + Math.sin(t*0.6)*0.05, ty = ptr.x*0.5 + Math.sin(t*0.4)*0.08;
  const tilt = mood==="thinking" ? 0.3*Math.sin(t*2) + 0.15 : 0;
  head.rotation.x += (tx-head.rotation.x)*0.08;
  head.rotation.y += (ty-head.rotation.y)*0.08;
  head.rotation.z += (tilt-head.rotation.z)*0.08;

  // Antenna pulses while thinking
  const pulse = mood==="thinking" ? 1+0.5*Math.sin(t*8) : 1;
  bulb.scale.setScalar(pulse); bulb.material.emissiveIntensity = mood==="thinking" ? 1.2+Math.sin(t*8) : 1;

  // Eye blink every few seconds
  if(blinkT < 0 && t > nextBlink){ blinkT = t; nextBlink = t + 2 + Math.random()*3; }
  let eyeY = 1;
  if(blinkT >= 0){ const p = (t-blinkT)/0.18; eyeY = p<1 ? Math.abs(1-p*2)*0.9+0.1 : (blinkT=-1,1); }
  eyeL.scale.y = eyeR.scale.y = happy ? 0.45 : eyeY;   // happy = squinty smile eyes

  // Mouth moves while talking
  mouth.scale.y = mood==="talking" ? 1+Math.abs(Math.sin(t*14))*5 : (happy ? 3 : 1);
  mouth.scale.x = happy ? 1.5 : 1;

  symbols.forEach(s=>{ const d = s.userData, a = d.a + t*d.sp;
    s.position.set(Math.cos(a)*d.r, d.y + Math.sin(t+d.a)*0.25, Math.sin(a)*1.5 - 1.5); });
  renderer.render(scene,camera);
})();
</script>
</body>
</html>
