<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <title>Asistencia por código (QR / Barras)</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    body { font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial; max-width:1100px; margin:18px auto; padding:12px; color:#111; }
    header { display:flex; justify-content:space-between; align-items:center; gap:12px; }
    h1 { margin:0; font-size:20px; }
    nav { display:flex; gap:10px; }
    button { padding:8px 12px; border-radius:8px; border:1px solid #ddd; background:#f7f7f7; cursor:pointer; }
    button.primary { background:#2563eb; color:white; border-color:#1e40af; }
    .container { margin-top:14px; display:grid; grid-template-columns: 1fr 360px; gap:18px; }
    .card { border:1px solid #e6e6e6; border-radius:10px; padding:12px; background:white; box-shadow: 0 1px 2px rgba(0,0,0,0.03); }
    input[type=text], input[type=password] { padding:10px; font-size:16px; width:100%; box-sizing:border-box; border:1px solid #ddd; border-radius:8px; }
    table { width:100%; border-collapse: collapse; margin-top:10px; font-size:14px; }
    th, td { padding:8px; border-bottom:1px solid #f0f0f0; text-align:left; }
    th { font-weight:600; color:#333; }
    .small { font-size:13px; color:#555; }
    .counts { display:flex; flex-direction:column; gap:8px; max-height:420px; overflow:auto; }
    .count-item { display:flex; justify-content:space-between; align-items:center; padding:8px; border-radius:8px; border:1px solid #f1f1f1; background:#fbfbff; }
    .actions { display:flex; gap:8px; align-items:center; }
    .muted { color:#777; font-size:13px; }
    .top-actions { display:flex; gap:8px; align-items:center; flex-wrap:wrap; }
    .danger { background:#fff0f0; border-color:#ffd6d6; }
    footer { margin-top:18px; color:#666; font-size:13px; }

    /* --- Modal cámara --- */
    .modal {
      display:none; position:fixed; top:0; left:0; width:100%; height:100%;
      background:rgba(0,0,0,0.55); justify-content:center; align-items:center; z-index:1000;
    }
    .modal-content {
      background:#fff; padding:16px; border-radius:12px; width:95%; max-width:420px;
      box-shadow:0 4px 20px rgba(0,0,0,0.25); text-align:center;
    }
    #reader { width:100%; height:280px; }
  </style>
  <!-- Librería para QR y códigos de barras -->
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>
  <header>
    <h1>Registro de asistencia — Código de barras / QR</h1>
    <nav>
      <button id="tab-registro" class="primary">Registro</button>
      <button id="tab-admin">Administración</button>
    </nav>
  </header>

  <main class="container">
    <!-- Panel principal -->
    <section id="panel-registro" class="card">
      <div style="display:flex; justify-content:space-between; align-items:center; gap:12px;">
        <div>
          <div class="small">Puedes escribir manualmente o escanear con cámara.</div>
          <h2 style="margin:8px 0 4px 0">Ingresar código</h2>
        </div>
        <div class="top-actions">
          <button id="btn-export">Exportar CSV</button>
          <button id="btn-clear" class="danger">Borrar historial</button>
        </div>
      </div>

      <form id="scanForm" onsubmit="return false;" style="margin-top:10px; display:flex; gap:8px;">
        <input id="codeInput" type="text" placeholder="Escribe o escanea el código y presiona Enter" autocomplete="off" />
        <button type="button" id="btnCamera">📷 Escanear</button>
      </form>

      <div style="display:flex; gap:12px; margin-top:12px; align-items:center;">
        <div class="small">Últimos escaneos</div>
        <div class="muted">· se registran todas las veces que se repita el código</div>
      </div>

      <div style="margin-top:10px; overflow:auto; max-height:360px;">
        <table id="recentTable">
          <thead>
            <tr><th>#</th><th>Nombre</th><th>Código</th><th>Fecha / Hora</th></tr>
          </thead>
          <tbody></tbody>
        </table>
      </div>
    </section>

    <!-- Sidebar -->
    <aside class="card" id="sidebar">
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <h3 style="margin:0">Conteo por persona</h3>
        <div class="small" id="totalCount">0 registros</div>
      </div>
      <div class="counts" id="countsList" style="margin-top:10px;"></div>
      <hr style="margin:12px 0; border:none; border-top:1px solid #f0f0f0;">
      <div>
        <h4 style="margin:0">Administración rápida</h4>
        <div style="margin-top:10px; display:grid; gap:8px;">
          <input id="quickCode" type="text" placeholder="Código" />
          <input id="quickName" type="text" placeholder="Nombre" />
          <div style="display:flex; gap:8px;">
            <button id="btnQuickAdd">Agregar / Actualizar</button>
            <button id="btnQuickDelete">Eliminar</button>
          </div>
        </div>
      </div>
    </aside>

    <!-- Panel admin -->
    <section id="panel-admin" class="card" style="display:none; grid-column: 1 / -1;">
      <h2>Administración</h2>
      <div id="loginBox">
        <input id="adminPass" type="password" placeholder="Contraseña admin (admin123)" />
        <button id="btn-login">Ingresar</button>
      </div>
      <div id="adminContent" style="display:none; margin-top:14px;">
        <input id="formCode" type="text" placeholder="Código" />
        <input id="formName" type="text" placeholder="Nombre completo" />
        <button id="btnAddPerson">Agregar / Actualizar</button>
        <table id="peopleTable" style="margin-top:12px; width:100%;">
          <thead><tr><th>Código</th><th>Nombre</th><th>Acciones</th></tr></thead>
          <tbody></tbody>
        </table>
      </div>
    </section>
  </main>

  <footer>
    <div class="small">Todo se guarda localmente en tu navegador (localStorage).</div>
  </footer>

  <!-- Modal cámara -->
  <div id="cameraModal" class="modal">
    <div class="modal-content">
      <h3>Escanear código</h3>
      <div id="reader"></div>
      <button id="btnCloseCam" style="margin-top:10px;">Cerrar</button>
    </div>
  </div>

  <script>
    // --- Claves localStorage
    const KEY_PEOPLE = 'attendance_people_v1';
    const KEY_SCANS = 'attendance_scans_v1';
    const KEY_PASS =  'attendance_admin_pass_v1';

    function loadPeople(){ return JSON.parse(localStorage.getItem(KEY_PEOPLE) || '{}'); }
    function savePeople(p){ localStorage.setItem(KEY_PEOPLE, JSON.stringify(p)); }
    function loadScans(){ return JSON.parse(localStorage.getItem(KEY_SCANS) || '[]'); }
    function saveScans(s){ localStorage.setItem(KEY_SCANS, JSON.stringify(s)); }
    function getAdminPass(){ return localStorage.getItem(KEY_PASS) || 'admin123'; }
    function setAdminPass(p){ localStorage.setItem(KEY_PASS, p); }

    let people = loadPeople(), scans = loadScans();

    const codeInput=document.getElementById('codeInput'), recentTbody=document.querySelector('#recentTable tbody');
    const countsList=document.getElementById('countsList'), totalCountEl=document.getElementById('totalCount');
    const tabRegistro=document.getElementById('tab-registro'), tabAdmin=document.getElementById('tab-admin');
    const panelRegistro=document.getElementById('panel-registro'), panelAdmin=document.getElementById('panel-admin');
    const btnExport=document.getElementById('btn-export'), btnClear=document.getElementById('btn-clear');

    const formCode=document.getElementById('formCode'), formName=document.getElementById('formName');
    const btnAddPerson=document.getElementById('btnAddPerson'), peopleTbody=document.querySelector('#peopleTable tbody');
    const adminPassInput=document.getElementById('adminPass'), btnLogin=document.getElementById('btn-login');
    const adminContent=document.getElementById('adminContent'), loginBox=document.getElementById('loginBox');

    // --- Render
    function renderRecent(){
      recentTbody.innerHTML='';
      const last=scans.slice().reverse();
      for(let i=0;i<Math.min(last.length,200);i++){
        const r=last[i];
        const tr=document.createElement('tr');
        tr.innerHTML=`<td>${last.length-i}</td><td>${people[r.code]||'Desconocido'}</td><td>${r.code}</td><td>${r.ts}</td>`;
        recentTbody.appendChild(tr);
      }
    }
    function renderCounts(){
      countsList.innerHTML='';
      const map={}; for(const s of scans){ map[s.code]=(map[s.code]||0)+1; }
      Object.keys(map).forEach(c=>{
        const div=document.createElement('div'); div.className='count-item';
        div.innerHTML=`<div><strong>${people[c]||'Desconocido'}</strong><div class="small">Código: ${c}</div></div>
        <div class="small">Veces: ${map[c]}</div>`;
        countsList.appendChild(div);
      });
      totalCountEl.textContent=scans.length+' registros';
    }
    function renderPeople(){
      peopleTbody.innerHTML='';
      Object.keys(people).forEach(c=>{
        const tr=document.createElement('tr');
        tr.innerHTML=`<td>${c}</td><td>${people[c]}</td><td><button onclick="editPerson('${c}')">Editar</button><button onclick="delPerson('${c}')">Eliminar</button></td>`;
        peopleTbody.appendChild(tr);
      });
    }
    function renderAll(){ people=loadPeople(); scans=loadScans(); renderRecent(); renderCounts(); renderPeople(); }

    // --- Escaneo manual
    codeInput.addEventListener('keydown',e=>{
      if(e.key==='Enter'){ e.preventDefault(); if(!codeInput.value.trim())return;
        addScan(codeInput.value.trim()); codeInput.value='';
      }
    });
    function addScan(code){
      const ts=new Date().toISOString().replace('T',' ').slice(0,19);
      scans.push({code,ts}); saveScans(scans); renderAll();
    }

    // --- Admin
    btnLogin.addEventListener('click',()=>{ 
      if(adminPassInput.value===getAdminPass()){ 
        loginBox.style.display='none'; adminContent.style.display='block'; 
      } else alert('Contraseña incorrecta'); 
    });
    btnAddPerson.addEventListener('click',()=>{ 
      if(formCode.value&&formName.value){ 
        people[formCode.value]=formName.value; 
        savePeople(people); renderAll(); formCode.value=''; formName.value=''; 
      } 
    });
    window.editPerson=(c)=>{ formCode.value=c; formName.value=people[c]; };
    window.delPerson=(c)=>{ if(confirm('Eliminar '+people[c]+'?')){ delete people[c]; savePeople(people); renderAll(); } };

    // --- Exportar y borrar
    btnExport.addEventListener('click',()=>{
      let csv='Código,Nombre,FechaHora\n';
      scans.forEach(s=>{ csv+=`${s.code},${people[s.code]||''},${s.ts}\n`; });
      downloadText(csv,'asistencia.csv','text/csv');
    });
    btnClear.addEventListener('click',()=>{ if(confirm('Borrar historial?')){ scans=[]; saveScans(scans); renderAll(); } });
    function downloadText(text,filename,mime){ const a=document.createElement('a'); a.href=URL.createObjectURL(new Blob([text],{type:mime})); a.download=filename; a.click(); }

    // --- Tabs
    tabRegistro.addEventListener('click',()=>{ panelRegistro.style.display='block'; panelAdmin.style.display='none'; tabRegistro.classList.add('primary'); tabAdmin.classList.remove('primary'); });
    tabAdmin.addEventListener('click',()=>{ panelRegistro.style.display='none'; panelAdmin.style.display='block'; tabAdmin.classList.add('primary'); tabRegistro.classList.remove('primary'); });

    // --- Modal cámara
    const btnCamera=document.getElementById('btnCamera'), camModal=document.getElementById('cameraModal'), btnCloseCam=document.getElementById('btnCloseCam');
    let html5QrCode=null;
    btnCamera.addEventListener('click',()=>{
      camModal.style.display='flex';
      html5QrCode=new Html5Qrcode("reader");
      html5QrCode.start({facingMode:"environment"},{fps:10,qrbox:{width:250,height:150}},(decodedText)=>{
        addScan(decodedText);
        camModal.style.display='none'; html5QrCode.stop(); html5QrCode=null;
      });
    });
    btnCloseCam.addEventListener('click',()=>{ 
      camModal.style.display='none'; 
      if(html5QrCode){ html5QrCode.stop(); html5QrCode=null; } 
    });

    // --- Inicializar
    (function initDefault(){
      if(Object.keys(loadPeople()).length===0){ 
        savePeople({"7772234":"Maicol Sanchez","1234567":"Daiyelin","9876543":"Carlos Perez"}); 
      }
      renderAll();
    })();
  </script>
</body>
</html>
