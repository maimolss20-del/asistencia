<html lang="es">
<head>
  <meta charset="utf-8" />
  <title>Asistencia por código (QR / Barras) — v2</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <meta name="theme-color" content="#2563eb" />
  <style>
    :root{ --accent:#2563eb; --muted:#777; --card:#fff; --radius:10px; }
    *{box-sizing:border-box}
    body { font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial; max-width:1200px; margin:18px auto; padding:12px; color:#111; background:#f6f7fb; }
    header { display:flex; justify-content:space-between; align-items:center; gap:12px; }
    h1 { margin:0; font-size:20px; }
    nav { display:flex; gap:10px; }
    button { padding:8px 12px; border-radius:8px; border:1px solid #ddd; background:#f7f7f7; cursor:pointer; }
    button.primary { background:var(--accent); color:white; border-color:#1e40af; }
    .layout { display:grid; grid-template-columns: 1fr 360px; gap:18px; margin-top:14px; align-items:start; }
    .card { border-radius:var(--radius); padding:12px; background:var(--card); box-shadow: 0 6px 18px rgba(16,24,40,0.06); border:1px solid rgba(0,0,0,0.03); }
    input[type=text], input[type=password], select { padding:10px; font-size:16px; width:100%; box-sizing:border-box; border:1px solid #ddd; border-radius:8px; }
    table { width:100%; border-collapse: collapse; margin-top:10px; font-size:14px; }
    th, td { padding:8px; border-bottom:1px solid #f0f0f0; text-align:left; }
    th { font-weight:600; color:#333; }
    .small { font-size:13px; color:#555; }
    .counts { display:flex; flex-direction:column; gap:8px; max-height:420px; overflow:auto; }
    .count-item { display:flex; justify-content:space-between; align-items:center; padding:8px; border-radius:8px; border:1px solid #f1f1f1; background:#fbfbff; }
    .actions { display:flex; gap:8px; align-items:center; }
    .muted { color:var(--muted); font-size:13px; }
    .top-actions { display:flex; gap:8px; align-items:center; flex-wrap:wrap; }
    .danger { background:#fff0f0; border-color:#ffd6d6; }
    footer { margin-top:18px; color:#666; font-size:13px; }
    /* modal camera */
    .modal { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.55); justify-content:center; align-items:center; z-index:1000; }
    .modal-content { background:#fff; padding:16px; border-radius:12px; width:95%; max-width:720px; box-shadow:0 6px 30px rgba(0,0,0,0.25); text-align:center; }
    #reader { width:100%; height:360px; }
    .row{display:flex;gap:8px}
    .col{flex:1}
    /* responsive */
    @media (max-width:900px){
      .layout{ grid-template-columns: 1fr; }
      #sidebar{ order:2 }
    }
  </style>
  <!-- Librería para QR y códigos de barras -->
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>
  <header>
    <div>
      <h1>Registro de asistencia — Código de barras / QR</h1>
      <div class="small">Funciona en móvil y en PC. Los datos se guardan localmente.</div>
    </div>
    <nav>
      <button id="tab-registro" class="primary">Registro</button>
      <button id="tab-admin">Administración</button>
    </nav>
  </header>

  <main class="layout">
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

      <div style="margin-top:10px; overflow:auto; max-height:420px;">
        <table id="recentTable">
          <thead>
            <tr><th>#</th><th>Nombre</th><th>Código</th><th>Fecha</th><th>Hora</th></tr>
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
      <div id="authBox">
        <div style="display:flex; gap:8px; align-items:center"> 
          <input id="adminPhone" type="text" placeholder="Teléfono / Usuario" />
          <input id="adminPass" type="password" placeholder="Contraseña" />
          <button id="btn-login">Ingresar</button>
          <button id="btn-logout" style="display:none;">Cerrar sesión</button>
        </div>
        <div style="margin-top:10px; display:flex; gap:8px; align-items:center">
          <input id="adminName" type="text" placeholder="Nombre del admin (para registro)" />
          <button id="btn-register">Registrar admin</button>
        </div>
      </div>

      <div id="adminContent" style="display:none; margin-top:14px;">
        <div style="display:flex; gap:8px;">
          <input id="formCode" type="text" placeholder="Código" />
          <input id="formName" type="text" placeholder="Nombre completo" />
          <button id="btnAddPerson">Agregar / Actualizar</button>
        </div>

        <table id="peopleTable" style="margin-top:12px; width:100%; ">
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
      <div style="display:flex; gap:8px; margin-bottom:8px; align-items:center;">
        <select id="cameraSelect" style="flex:1"></select>
        <button id="btnStartCam">Iniciar</button>
        <button id="btnCloseCam">Cerrar</button>
      </div>
      <div id="reader"></div>
    </div>
  </div>

  <script>
    // --- Claves localStorage
    const KEY_PEOPLE = 'attendance_people_v2';
    const KEY_SCANS = 'attendance_scans_v2';
    const KEY_ADMINS = 'attendance_admins_v2';
    const KEY_SESSION = 'attendance_session_v2';

    function loadPeople(){ return JSON.parse(localStorage.getItem(KEY_PEOPLE) || '{}'); }
    function savePeople(p){ localStorage.setItem(KEY_PEOPLE, JSON.stringify(p)); }
    function loadScans(){ return JSON.parse(localStorage.getItem(KEY_SCANS) || '[]'); }
    function saveScans(s){ localStorage.setItem(KEY_SCANS, JSON.stringify(s)); }
    function loadAdmins(){ return JSON.parse(localStorage.getItem(KEY_ADMINS) || '{}'); }
    function saveAdmins(a){ localStorage.setItem(KEY_ADMINS, JSON.stringify(a)); }
    function getSession(){ return JSON.parse(localStorage.getItem(KEY_SESSION) || 'null'); }
    function setSession(s){ localStorage.setItem(KEY_SESSION, JSON.stringify(s)); }
    function clearSession(){ localStorage.removeItem(KEY_SESSION); }

    let people = loadPeople(), scans = loadScans(), admins = loadAdmins();

    // --- DOM
    const codeInput=document.getElementById('codeInput'), recentTbody=document.querySelector('#recentTable tbody');
    const countsList=document.getElementById('countsList'), totalCountEl=document.getElementById('totalCount');
    const tabRegistro=document.getElementById('tab-registro'), tabAdmin=document.getElementById('tab-admin');
    const panelRegistro=document.getElementById('panel-registro'), panelAdmin=document.getElementById('panel-admin');
    const btnExport=document.getElementById('btn-export'), btnClear=document.getElementById('btn-clear');

    const formCode=document.getElementById('formCode'), formName=document.getElementById('formName');
    const btnAddPerson=document.getElementById('btnAddPerson'), peopleTbody=document.querySelector('#peopleTable tbody');
    const adminPhone=document.getElementById('adminPhone'), adminPassInput=document.getElementById('adminPass'), btnLogin=document.getElementById('btn-login');
    const adminContent=document.getElementById('adminContent'), btnLogout=document.getElementById('btn-logout');
    const btnRegister=document.getElementById('btn-register'), adminName=document.getElementById('adminName');

    // --- Render
    function renderRecent(){
      recentTbody.innerHTML='';
      const last=scans.slice().reverse();
      for(let i=0;i<last.length;i++){
        const r=last[i];
        const dateStr = r.ts.split('T')[0];
        const timeStr = r.ts.split('T')[1].slice(0,8);
        const tr=document.createElement('tr');
        tr.innerHTML=`<td>${last.length-i}</td><td>${people[r.code]||'Desconocido'}</td><td>${r.code}</td><td>${dateStr}</td><td>${timeStr}</td>`;
        recentTbody.appendChild(tr);
      }
    }
    function computeCounts(){ const map={}; for(const s of scans){ map[s.code]=(map[s.code]||0)+1; } return map; }
    function renderCounts(){
      countsList.innerHTML='';
      const map=computeCounts();
      Object.keys(map).sort((a,b)=>map[b]-map[a]).forEach(c=>{
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
        tr.innerHTML=`<td>${c}</td><td>${people[c]}</td><td><button onclick="editPerson('${c}')">Editar</button> <button onclick="delPerson('${c}')">Eliminar</button></td>`;
        peopleTbody.appendChild(tr);
      });
    }
    function renderAll(){ people=loadPeople(); scans=loadScans(); admins=loadAdmins(); renderRecent(); renderCounts(); renderPeople(); updateAuthState(); }

    // --- Escaneo manual
    codeInput.addEventListener('keydown',e=>{
      if(e.key==='Enter'){ e.preventDefault(); if(!codeInput.value.trim())return;
        addScan(codeInput.value.trim()); codeInput.value='';
      }
    });
    function addScan(code){
      const ts=new Date().toISOString();
      scans.push({code,ts}); saveScans(scans); renderAll();
    }

    // --- Admin auth (multi-admin) ---
    btnRegister.addEventListener('click',()=>{
      const phone=adminPhone.value.trim(); const pass=adminPassInput.value;
      const name=adminName.value.trim(); if(!phone||!pass||!name){ alert('Completa teléfono, contraseña y nombre para registrar.'); return; }
      admins[phone]={ password:pass, name }; saveAdmins(admins); adminName.value=''; adminPassInput.value=''; adminPhone.value=''; renderAll(); alert('Administrador registrado');
    });
    btnLogin.addEventListener('click',()=>{
      const phone=adminPhone.value.trim(); const pass=adminPassInput.value;
      if(!phone||!pass){ alert('Ingresa teléfono y contraseña'); return; }
      const a=admins[phone]; if(a && a.password===pass){ setSession({phone,name:a.name}); updateAuthState(); alert('Sesión iniciada como '+a.name); } else { alert('Usuario o contraseña incorrectos'); }
    });
    btnLogout.addEventListener('click',()=>{ clearSession(); updateAuthState(); alert('Sesión cerrada'); });

    function updateAuthState(){ const session=getSession(); if(session){ loginShow(true,session); } else { loginShow(false); } }
    function loginShow(isLogged,sessionData){ if(isLogged){ adminContent.style.display='block'; btnLogout.style.display='inline-block'; btnLogin.style.display='none'; btnRegister.style.display='none'; adminPhone.style.display='none'; adminPassInput.style.display='none'; adminName.style.display='none'; } else { adminContent.style.display='none'; btnLogout.style.display='none'; btnLogin.style.display='inline-block'; btnRegister.style.display='inline-block'; adminPhone.style.display='inline-block'; adminPassInput.style.display='inline-block'; adminName.style.display='inline-block'; } }

    // --- Admin actions (only when logged) ---
    btnAddPerson.addEventListener('click',()=>{
      const session=getSession(); if(!session){ alert('Debes iniciar sesión como administrador para modificar personas'); return; }
      if(formCode.value && formName.value){ people[formCode.value]=formName.value; savePeople(people); renderAll(); formCode.value=''; formName.value=''; }
    });
    window.editPerson=(c)=>{ formCode.value=c; formName.value=people[c]; };
    window.delPerson=(c)=>{ const session=getSession(); if(!session){ alert('Debes iniciar sesión como administrador para eliminar'); return; } if(confirm('Eliminar '+people[c]+'?')){ delete people[c]; savePeople(people); renderAll(); } };

    // --- Exportar y borrar ---
    btnExport.addEventListener('click',()=>{
      // CSV con: Codigo,Nombre,Fecha,Hora,Repeticiones (total para ese código)
      const map=computeCounts();
      let csv='Código,Nombre,Fecha,Hora,Repeticiones\n';
      for(const s of scans){
        const dateStr = s.ts.split('T')[0];
        const timeStr = s.ts.split('T')[1].slice(0,8);
        csv+=`${s.code},"${(people[s.code]||'')}",${dateStr},${timeStr},${map[s.code]}\n`;
      }
      downloadText(csv,'asistencia.csv','text/csv');
    });
    btnClear.addEventListener('click',()=>{ if(confirm('Borrar historial?')){ scans=[]; saveScans(scans); renderAll(); } });
    function downloadText(text,filename,mime){ const a=document.createElement('a'); a.href=URL.createObjectURL(new Blob([text],{type:mime})); a.download=filename; a.click(); }

    // --- Tabs ---
    tabRegistro.addEventListener('click',()=>{ panelRegistro.style.display='block'; panelAdmin.style.display='none'; tabRegistro.classList.add('primary'); tabAdmin.classList.remove('primary'); });
    tabAdmin.addEventListener('click',()=>{ panelRegistro.style.display='none'; panelAdmin.style.display='block'; tabAdmin.classList.add('primary'); tabRegistro.classList.remove('primary'); });

    // --- Cámara (compatible PC y móvil) ---
    const btnCamera=document.getElementById('btnCamera'), camModal=document.getElementById('cameraModal'), btnCloseCam=document.getElementById('btnCloseCam');
    const cameraSelect=document.getElementById('cameraSelect'), btnStartCam=document.getElementById('btnStartCam');
    let html5QrCode=null; let currentCamId=null;

    async function hasCameraAccess(){ try{ const cams = await Html5Qrcode.getCameras(); return cams && cams.length>0; }catch(e){ return false; } }

    btnCamera.addEventListener('click',async ()=>{
      const cams = await Html5Qrcode.getCameras().catch(()=>[]);
      cameraSelect.innerHTML='';
      if(cams.length===0){ alert('No se detectó cámara en este dispositivo'); return; }
      cams.forEach(c=>{ const opt=document.createElement('option'); opt.value=c.id; opt.textContent=c.label||c.id; cameraSelect.appendChild(opt); });
      camModal.style.display='flex';
    });

    btnStartCam.addEventListener('click',()=>{
      const camId=cameraSelect.value; if(!camId) return; startScanner(camId);
    });

    btnCloseCam.addEventListener('click',()=>{ closeScanner(); camModal.style.display='none'; });

    function startScanner(cameraId){
      if(html5QrCode){ html5QrCode.stop().catch(()=>{}); html5QrCode=null; }
      html5QrCode = new Html5Qrcode("reader");
      html5QrCode.start(
        cameraId,
        { fps: 10, qrbox: { width: 320, height: 200 } },
        decodedText => { addScan(decodedText); alert('Código capturado: '+decodedText); closeScanner(); camModal.style.display='none'; },
        errorMessage => { /* ignore per-frame errors */ }
      ).catch(err=>{ alert('No se pudo iniciar la cámara: '+err); });
    }
    function closeScanner(){ if(html5QrCode){ html5QrCode.stop().catch(()=>{}); html5QrCode.clear(); html5QrCode=null; } }

    // --- Inicializar datos por defecto ---
    (function initDefault(){
      if(Object.keys(loadPeople()).length===0){ 
        savePeople({"7772234":"Maicol Sanchez","1234567":"Daiyelin","9876543":"Carlos Perez"}); 
      }
      // crear admin por defecto si no hay ninguno
      if(Object.keys(loadAdmins()).length===0){ const defaultAdmin={ '3000000000':{ password:'admin123', name:'Admin' } }; saveAdmins(defaultAdmin); }
      renderAll();
    })();

    // cerrar scanner si se navega fuera
    window.addEventListener('beforeunload',()=>{ closeScanner(); });
  </script>
</body>
</html>
