# ruleta-ck
ruleta 
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Ruleta La Casa del Kine</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@500;700;800&display=swap" rel="stylesheet">
<style>
:root{
  --magenta:#B5127F; --ciruela:#4D0035; --naranja:#FF4F1A; --rosa:#FF7AF5; --azul:#2D9CFF; --azul-oscuro:#1B7FE0;
  color-scheme:dark; box-sizing:border-box;
  padding-top:env(safe-area-inset-top,0px); padding-bottom:env(safe-area-inset-bottom,0px);
}
*,*::before,*::after{box-sizing:inherit}
html,body{height:100%;margin:0}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
body{
  font-family:Poppins,system-ui,-apple-system,"Segoe UI",sans-serif; color:#fff;
  background:linear-gradient(160deg,var(--magenta) 0%,#8A0A63 45%,var(--ciruela) 100%);
  display:flex;flex-direction:column;align-items:center;justify-content:center;gap:14px;
  overflow:hidden;-webkit-user-select:none;user-select:none;touch-action:manipulation;
}
header{text-align:center;padding:0 16px}
header p{margin:0;font-weight:500;font-size:clamp(14px,2.2vw,20px);opacity:.9}
header h1{margin:2px 0 0;font-weight:800;font-size:clamp(26px,5vw,46px);line-height:1.1}
.stage{position:relative;width:min(92vw,calc(100vh - 210px));width:min(92vw,calc(100svh - 210px));aspect-ratio:1}
#wheel{width:100%;height:100%;display:block;filter:drop-shadow(0 14px 28px rgba(0,0,0,.45));will-change:transform}
.pointer{position:absolute;left:50%;top:-2.5%;width:0;height:0;transform:translateX(-50%);z-index:2;
  border-left:clamp(14px,3.2vw,26px) solid transparent;border-right:clamp(14px,3.2vw,26px) solid transparent;
  border-top:clamp(30px,7vw,56px) solid #fff;filter:drop-shadow(0 4px 6px rgba(0,0,0,.5))}
.hub{position:absolute;left:50%;top:50%;width:26%;aspect-ratio:1;transform:translate(-50%,-50%);z-index:3;
  border-radius:50%;border:clamp(4px,1vw,8px) solid var(--rosa);background:var(--naranja);color:#fff;
  font:800 clamp(16px,3.6vw,30px)/1 Poppins,system-ui,sans-serif;cursor:pointer;
  box-shadow:6px 6px 0 var(--azul),0 10px 24px rgba(0,0,0,.4);transition:transform .15s}
.hub:active{transform:translate(-50%,-50%) scale(.94)}
.hub:disabled{filter:saturate(.5);cursor:default}
.hub:focus-visible,.close:focus-visible,.reveal:focus-visible{outline:4px solid #fff;outline-offset:3px}
.stats{font-size:13px;font-weight:500;opacity:.75;margin:0}
.overlay{position:fixed;inset:0;background:rgba(40,0,28,.72);display:none;align-items:center;justify-content:center;padding:20px;z-index:10}
.overlay.on{display:flex}
.card{width:min(560px,100%);background:#fff;color:var(--ciruela);border-radius:28px;padding:28px 26px;text-align:center;
  border-bottom:10px solid var(--c,var(--naranja));box-shadow:8px 8px 0 var(--azul);animation:pop .28s ease-out}
.card h2{margin:0 0 10px;font-weight:800;font-size:clamp(28px,5vw,42px);line-height:1.1;color:var(--c,var(--naranja))}
.card p{margin:0 0 18px;font-size:clamp(17px,2.6vw,22px);line-height:1.4;font-weight:500}
.card .q{font-weight:700;font-size:clamp(20px,3vw,26px)}
.answer{display:none;font-weight:800;color:var(--magenta)}
.answer.on{display:block}
.btns{display:flex;gap:12px;justify-content:center;flex-wrap:wrap}
.close,.reveal{font:800 clamp(16px,2.4vw,20px) Poppins,system-ui,sans-serif;border:0;border-radius:999px;padding:14px 30px;cursor:pointer;color:#fff;background:var(--naranja);box-shadow:4px 4px 0 var(--azul)}
.reveal{background:var(--magenta)}
@keyframes pop{from{transform:scale(.88);opacity:0}to{transform:none;opacity:1}}
@media (prefers-reduced-motion:reduce){.card{animation:none}}
</style>
</head>
<body>
<header>
  <p>La Casa del Kine</p>
  <h1>Girá la ruleta y ganá</h1>
</header>

<div class="stage">
  <div class="pointer" aria-hidden="true"></div>
  <canvas id="wheel" width="1000" height="1000" role="img" aria-label="Ruleta de premios"></canvas>
  <button class="hub" id="spin" type="button" aria-label="Girar la ruleta">GIRAR</button>
</div>
<p class="stats" id="stats">Giros: 0 | Premios: 0</p>

<div class="overlay" id="ov" role="dialog" aria-modal="true" aria-live="polite">
  <div class="card" id="card">
    <h2 id="t"></h2>
    <p id="d"></p>
    <p class="answer" id="a"></p>
    <div class="btns">
      <button class="reveal" id="rv" type="button" hidden>Ver respuesta</button>
      <button class="close" id="cl" type="button">Seguir jugando</button>
    </div>
  </div>
</div>

<script>
/* ====== CONFIGURACIÓN: editá acá textos, preguntas y desafíos ====== */
const SEGS = [
  {k:'premio',   l:['¡PREMIO!'],                          c:'#FF4F1A', fs:46},
  {k:'seguir',   l:['SEGUÍ','PARTICIPANDO'],              c:'#B5127F', fs:34},
  {k:'pregunta', l:['RESPONDÉ UNA','PREGUNTA','Y GANÁ'],  c:'#1B7FE0', fs:34},
  {k:'premio2',  l:['PREMIO','DE NUEVO'],                 c:'#FF4F1A', fs:40},
  {k:'desafio',  l:['DESAFÍO'],                           c:'#B5127F', fs:44},
  {k:'seguir2',  l:['SEGUÍ','PARTICIPANDO','DE NUEVO'],   c:'#1B7FE0', fs:34}
];
const QUESTIONS = [
  ['¿Cuántas vértebras tiene la columna cervical?','Siete (C1 a C7).'],
  ['¿Qué nervio se comprime en el síndrome del túnel carpiano?','El nervio mediano.'],
  ['¿Cuál es el hueso más largo del cuerpo humano?','El fémur.'],
  ['¿Qué articulación une el húmero con la escápula?','La articulación glenohumeral.'],
  ['¿Qué músculo es el principal flexor del codo?','El bíceps braquial (junto con el braquial).'],
  ['¿Cómo se llama el vendaje con cinta elástica adhesiva aplicada sobre la piel?','Kinesiotaping o vendaje neuromuscular.']
];
const CHALLENGES = [
  'Mantené el equilibrio sobre un pie durante 20 segundos.',
  'Hacé 10 sentadillas sin parar.',
  'Nombrá 5 músculos del miembro inferior en 15 segundos.',
  'Sostené la posición de plancha durante 20 segundos.',
  'Tocá la punta de tus pies sin doblar las rodillas.'
];
const RESULTS = {
  premio:  {t:'¡Ganaste un premio!', d:'Acercate al stand de La Casa del Kine para retirarlo.', c:'#FF4F1A'},
  premio2: {t:'¡Otro premio!',       d:'Premio de nuevo. Acercate al stand de La Casa del Kine para retirarlo.', c:'#FF4F1A'},
  seguir:  {t:'Seguí participando',  d:'Esta vez no hubo premio. ¡Girá otra vez!', c:'#B5127F'},
  seguir2: {t:'Seguí participando',  d:'Casi. Probá de nuevo, la próxima puede ser premio.', c:'#1B7FE0'},
  pregunta:{t:'Respondé y ganá',     c:'#1B7FE0'},
  desafio: {t:'Desafío',             c:'#B5127F'}
};
/* ================================================================ */

const cv = document.getElementById('wheel'), ctx = cv.getContext('2d');
const N = SEGS.length, A = 2*Math.PI/N;
const REDUCE = matchMedia('(prefers-reduced-motion: reduce)').matches;

function draw(){
  ctx.clearRect(0,0,1000,1000); ctx.save(); ctx.translate(500,500);
  ctx.fillStyle='#fff'; ctx.beginPath(); ctx.arc(0,0,496,0,7); ctx.fill();
  ctx.fillStyle='#4D0035'; ctx.beginPath(); ctx.arc(0,0,482,0,7); ctx.fill();
  for(let i=0;i<24;i++){
    const a=i*Math.PI/12; ctx.fillStyle = i%2 ? '#FF7AF5' : '#fff';
    ctx.beginPath(); ctx.arc(Math.cos(a)*463,Math.sin(a)*463,9,0,7); ctx.fill();
  }
  SEGS.forEach((s,i)=>{
    const a0=-Math.PI/2+i*A, a1=a0+A;
    ctx.fillStyle=s.c; ctx.beginPath(); ctx.moveTo(0,0); ctx.arc(0,0,442,a0,a1); ctx.closePath(); ctx.fill();
    ctx.strokeStyle='#fff'; ctx.lineWidth=5; ctx.stroke();
    ctx.save(); ctx.rotate(a0+A/2); ctx.fillStyle='#fff'; ctx.textAlign='right'; ctx.textBaseline='middle';
    ctx.font='800 '+s.fs+'px Poppins, system-ui, sans-serif';
    ctx.shadowColor='rgba(0,0,0,.35)'; ctx.shadowBlur=6;
    const lh=s.fs*1.1;
    s.l.forEach((ln,j)=>ctx.fillText(ln,415,(j-(s.l.length-1)/2)*lh));
    ctx.restore();
  });
  ctx.restore();
}
draw();
if(document.fonts && document.fonts.load){ document.fonts.load('800 34px Poppins').then(draw).catch(()=>{}); }

let rot=0, busy=false, pending=null, finished=true, spins=0, prizes=0, qi=0, ci=0;
const $=id=>document.getElementById(id);
const rand = n => { try{ const b=new Uint32Array(1); crypto.getRandomValues(b); return b[0]/4294967296*n; }catch(e){ return Math.random()*n; } };
const order = arr => arr.map((_,i)=>i).sort(()=>rand(1)-.5);
let qOrder=order(QUESTIONS), cOrder=order(CHALLENGES);

function spin(){
  if(busy) return; busy=true; finished=false; $('spin').disabled=true;
  const i=Math.floor(rand(N)); pending=i;
  const jitter=(rand(1)-.5)*A*.7*180/Math.PI;
  const target=-(i+.5)*A*180/Math.PI + jitter;
  const cur=((rot%360)+360)%360;
  rot += (((target-cur)%360)+360)%360 + 360*(5+Math.floor(rand(3)));
  const dur = REDUCE ? 1 : 5.5;
  cv.style.transition='transform '+dur+'s cubic-bezier(.12,.7,.12,1)';
  cv.style.transform='rotate('+rot+'deg)';
  setTimeout(finish, dur*1000+400);
}
function finish(){
  if(finished) return; finished=true;
  const s=SEGS[pending], r=RESULTS[s.k];
  spins++; if(s.k==='premio'||s.k==='premio2') prizes++;
  $('stats').textContent='Giros: '+spins+' | Premios: '+prizes;
  let d=r.d||'', ans='';
  if(s.k==='pregunta'){ const q=QUESTIONS[qOrder[qi++%QUESTIONS.length]]; d=q[0]; ans=q[1]; }
  if(s.k==='desafio'){ d=CHALLENGES[cOrder[ci++%CHALLENGES.length]]; }
  $('card').style.setProperty('--c',r.c);
  $('t').textContent=r.t; $('d').textContent=d; $('d').className = s.k==='pregunta' ? 'q' : '';
  $('a').textContent=ans; $('a').classList.remove('on');
  $('rv').hidden = !ans; $('rv').textContent='Ver respuesta';
  $('ov').classList.add('on'); $('cl').focus();
}
cv.addEventListener('transitionend', finish);
$('spin').addEventListener('click', spin);
cv.addEventListener('click', spin);
$('rv').addEventListener('click', ()=>{ $('a').classList.toggle('on'); $('rv').textContent = $('a').classList.contains('on') ? 'Ocultar respuesta' : 'Ver respuesta'; });
$('cl').addEventListener('click', ()=>{ $('ov').classList.remove('on'); busy=false; $('spin').disabled=false; });
</script>
</body>
</html>
