<html lang="es">
<head>
<meta charset="utf-8" />
<title>Asistencia por código (QR / Barras) — Unificado</title>
<meta name="viewport" content="width=device-width,initial-scale=1" />
<meta name="theme-color" content="#2563eb" />
<style>
:root { --accent:#2563eb; --muted:#777; --card:#fff; --radius:10px; }
*{box-sizing:border-box;}
body { font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial; max-width:1200px; margin:18px auto; padding:12px; color:#111; background:#f6f7fb; }
header, main, footer { display:block; }
header { display:flex; justify-content:space-between; align-items:center; gap:12px; background:#fff; padding:12px 16px; border-radius:12px; box-shadow:0 4px 12px rgba(0,0,0,0.1); margin-bottom:16px; }
.layout { display:grid; grid-template-columns: 1fr 360px; gap:18px; margin-top:14px; align-items:start; }
.card { border-radius:var(--radius); padding:12px; background:var(--card); box-shadow:0 6px 18px rgba(16,24,40,0.06); border:1px solid rgba(0,0,0,0.03); }
input[type=text], select { padding:10px; font-size:16px; width:100%; box-sizing:border-box; border:1px solid #ddd; border-radius:8px; }
table { width:100%; border-collapse: collapse; margin-top:10px; font-size:14px; }
th, td { padding:8px; border-bottom:1px solid #f0f0f0; text-align:left; }
th { font-weight:600; color:#333; }
footer { margin-top:18px; color:#666; font-size:13px; }
.counts { display:flex; flex-direction:column; gap:8px; max-height:420px; overflow:auto; }
.count-item { display:flex; justify-content:space-between; align-items:center; padding:8px; border-radius:8px; border:1px solid #f1f1f1; background:#fbfbff; }
.danger { background:#fff0f0; border-color:#ffd6d6; }
.modal { display:none; position:fixed; inset:0; background:rgba(0,0,0,0.55); justify-content:center; align-items:center; z-index:1000; }
.modal-content { background:#fff; padding:16px; border-radius:12px; width:95%; max-width:720px; box-shadow:0 6px 30px rgba(0,0,0,0.25); text-align:center; }
#reader { width:100%; height:360px; }
@media (max-width:900px){ .layout{ grid-template-columns:1fr; } #sidebar{ order:2 } }
</style>
<script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>

<header>
  <div>
    <h1>Registro de asistencia — Código de barras / QR</h1>
    <div class="small">Funciona en móvil y en PC. Los datos se guardan localmente.</div>
  </div>
</header>

<main class="layout">

<section id="panel-registro" class="card">
  <div style="display:flex; justify-content:space-between; align-items:center;">
    <div>
      <div class="small">Escribe o escanea el código.</div>
      <h2>Ingresar código</h2>
    </div>
    <div style="display:flex; gap:8px;">
      <button id="btn-export">Exportar CSV</button>
      <button id="btn-clear" class="danger">Borrar historial</button>
    </div>
  </div>
  <form id="scanForm" onsubmit="return false;" style="margin-top:10px; display:flex; gap:8px;">
    <input id="codeInput" type="text" placeholder="Escribe o escanea el código y presiona Enter" autocomplete="off" />
    <button type="button" id="btnCamera">📷 Escanear</button>
  </form>
  <div style="margin-top:10px; overflow:auto; max-height:420px;">
    <table id="recentTable">
      <thead><tr><th>#</th><th>Nombre</th><th>Código</th><th>Fecha</th><th>Hora</th></tr></thead>
      <tbody></tbody>
    </table>
  </div>
</section>

<aside class="card" id="sidebar">
  <h3 style="margin:0">Conteo por persona</h3>
  <div class="counts" id="countsList" style="margin-top:10px;"></div>
  <div id="totalCount" class="small"></div>
</aside>

<section id="panel-admin" class="card" style="grid-column:1/-1;">
  <h2>Administración</h2>
  <div style="display:flex; gap:8px; margin-bottom:10px;">
    <input id="formCode" type="text" placeholder="Código" />
    <input id="formName" type="text" placeholder="Nombre completo" />
    <button id="btnAddPerson">Agregar / Actualizar</button>
  </div>
  <table id="peopleTable" style="width:100%; margin-bottom:16px;">
    <thead><tr><th>Código</th><th>Nombre</th><th>Acciones</th></tr></thead>
    <tbody></tbody>
  </table>
</section>
</main>

<footer><div class="small">Todo se guarda en localStorage.</div></footer>

<div id="cameraModal" class="modal">
  <div class="modal-content">
    <h3>Escanear código</h3>
    <div style="display:flex; gap:8px; margin-bottom:8px;">
      <select id="cameraSelect" style="flex:1"></select>
      <button id="btnStartCam">Iniciar</button>
      <button id="btnCloseCam">Cerrar</button>
    </div>
    <div id="reader"></div>
  </div>
</div>

<script>
const KEY_PEOPLE='attendance_people';
const KEY_SCANS='attendance_scans';

const load=(k,d)=>JSON.parse(localStorage.getItem(k)||d);
const save=(k,v)=>localStorage.setItem(k,JSON.stringify(v));

let people=load(KEY_PEOPLE,"{}"), scans=load(KEY_SCANS,"[]");

const codeInput=document.getElementById('codeInput');
const recentTbody=document.querySelector('#recentTable tbody');
const countsList=document.getElementById('countsList');
const totalCountEl=document.getElementById('totalCount');
const formCode=document.getElementById('formCode');
const formName=document.getElementById('formName');
const btnAddPerson=document.getElementById('btnAddPerson');
const peopleTbody=document.querySelector('#peopleTable tbody');
const btnExport=document.getElementById('btn-export');
const btnClear=document.getElementById('btn-clear');

function renderRecent(){ 
  recentTbody.innerHTML=''; 
  const last=scans.slice().reverse(); 
  last.forEach((r,i)=>{ 
    const d=r.ts.split('T'); 
    recentTbody.innerHTML+=`<tr><td>${last.length-i}</td><td>${people[r.code]||'?'}</td><td>${r.code}</td><td>${d[0]}</td><td>${d[1].slice(0,8)}</td></tr>`; 
  }); 
}

function renderCounts(){ 
  countsList.innerHTML=''; 
  const map={}; 
  scans.forEach(s=>map[s.code]=(map[s.code]||0)+1); 
  Object.keys(map).forEach(c=>{ 
    countsList.innerHTML+=`<div class="count-item"><div><b>${people[c]||'?'}</b><div class="small">Código: ${c}</div></div><div class="small">Veces: ${map[c]}</div></div>`; 
  }); 
  totalCountEl.textContent=scans.length+" registros"; 
}

function renderPeople(){ 
  peopleTbody.innerHTML=''; 
  Object.keys(people).forEach(c=>{
    peopleTbody.innerHTML+=`<tr><td>${c}</td><td>${people[c]}</td><td><button onclick="editPerson('${c}')">Editar</button> <button onclick="delPerson('${c}')">Eliminar</button></td></tr>`;
  });
}

function editPerson(c){ formCode.value=c; formName.value=people[c]; }
function delPerson(c){ delete people[c]; save(KEY_PEOPLE,people); renderPeople(); }

btnAddPerson.onclick=()=>{
  const c=formCode.value.trim(), n=formName.value.trim();
  if(!c||!n){ alert("Completa código y nombre"); return; }
  people[c]=n;
  save(KEY_PEOPLE,people);
  formCode.value=formName.value='';
  renderPeople();
};

codeInput.addEventListener('keypress', e=>{
  if(e.key==='Enter'){
    const code=codeInput.value.trim();
    if(!code) return;
    if(!people[code]){ alert("Usuario no registrado"); return; }
    scans.push({code, ts:new Date().toISOString()});
    save(KEY_SCANS,scans);
    codeInput.value='';
    renderRecent();
    renderCounts();
  }
});

btnClear.onclick=()=>{ if(confirm("Borrar todo el historial?")){ scans=[]; save(KEY_SCANS,scans); renderRecent(); renderCounts(); } };

btnExport.onclick=()=>{ 
  let csv="Nombre,Código,Fecha,Hora,Veces\n";
  const map={};
  scans.forEach(s=>{
    if(!map[s.code]) map[s.code]={code:s.code, name:people[s.code]||'?', ts:s.ts, count:0};
    map[s.code].count++;
    map[s.code].ts = s.ts; // guardamos última vez
  });
  Object.values(map).forEach(item=>{
    const d=item.ts.split('T');
    csv+=`"${item.name}","${item.code}","${d[0]}","${d[1].slice(0,8)}","${item.count}"\n`;
  });
  const blob=new Blob([csv],{type:'text/csv'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a'); a.href=url; a.download="asistencia.csv"; a.click(); URL.revokeObjectURL(url);
};

renderPeople();
renderRecent();
renderCounts();

const cameraModal=document.getElementById('cameraModal');
const readerEl=document.getElementById('reader');
const cameraSelect=document.getElementById('cameraSelect');
const btnStartCam=document.getElementById('btnStartCam');
const btnCloseCam=document.getElementById('btnCloseCam');
let html5QrcodeScanner;

document.getElementById('btnCamera').onclick=()=>{
  cameraModal.style.display='flex';
  Html5Qrcode.getCameras().then(devices=>{
    cameraSelect.innerHTML='';
    devices.forEach(cam=>{
      const opt=document.createElement('option'); opt.value=cam.id; opt.textContent=cam.label; cameraSelect.appendChild(opt);
    });
  });
};

btnStartCam.onclick = () => {
  if(html5QrcodeScanner) html5QrcodeScanner.stop();
  html5QrcodeScanner = new Html5Qrcode("reader");

  html5QrcodeScanner.start(
    cameraSelect.value,
    { fps: 10, qrbox: 250 },
    decoded => {
      if(!people[decoded]){
        alert("Usuario no registrado");
        return; // cámara sigue abierta
      }
      // si existe, registrar y cerrar cámara
      scans.push({ code: decoded, ts: new Date().toISOString() });
      save(KEY_SCANS, scans);
      renderRecent();
      renderCounts();
      html5QrcodeScanner.stop().then(()=> cameraModal.style.display='none');
    }
  );
};

btnCloseCam.onclick=()=>{
  cameraModal.style.display='none';
  if(html5QrcodeScanner) html5QrcodeScanner.stop();
};
</script>

</body>
</html>
