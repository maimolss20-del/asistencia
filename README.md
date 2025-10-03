<html lang="es">
<head>
  <meta charset="utf-8" />
  <title>Asistencia por código (QR / Barras) — Unificado</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <meta name="theme-color" content="#2563eb" />
  <style>
    :root{ --accent:#2563eb; --muted:#777; --card:#fff; --radius:10px; }
    *{box-sizing:border-box}
    body { font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial; max-width:1200px; margin:18px auto; padding:12px; color:#111; background:#f6f7fb; }
    header, main, footer { display:none; } /* ocultos hasta login */

    /* --- LOGIN --- */
    #loginScreen { position:fixed; inset:0; background:#f0f2f9; display:flex; justify-content:center; align-items:center; }
    #loginBox { background:white; padding:24px; border-radius:12px; box-shadow:0 8px 24px rgba(0,0,0,0.2); width:95%; max-width:400px; text-align:center; }
    #loginBox input { width:100%; padding:10px; margin:6px 0; border:1px solid #ccc; border-radius:8px; }
    #loginBox button { margin-top:8px; padding:10px; width:100%; border:none; border-radius:8px; background:var(--accent); color:white; font-weight:600; cursor:pointer; }
    #loginBox button.secondary { background:#eee; color:#333; }
    .small { font-size:13px; color:#666; margin-top:10px; }

    header { display:flex; justify-content:space-between; align-items:center; gap:12px; background:#fff; padding:12px 16px; border-radius:12px; box-shadow:0 4px 12px rgba(0,0,0,0.1); margin-bottom:16px; }
    nav { display:flex; gap:10px; }
    nav button { padding:8px 14px; border:none; border-radius:8px; background:#eee; cursor:pointer; }
    nav button.primary { background:var(--accent); color:white; }
    #btnLogout { background:#dc2626; color:white; }

    .layout { display:grid; grid-template-columns: 1fr 360px; gap:18px; margin-top:14px; align-items:start; }
    .card { border-radius:var(--radius); padding:12px; background:var(--card); box-shadow: 0 6px 18px rgba(16,24,40,0.06); border:1px solid rgba(0,0,0,0.03); }
    input[type=text], input[type=password], select { padding:10px; font-size:16px; width:100%; box-sizing:border-box; border:1px solid #ddd; border-radius:8px; }
    table { width:100%; border-collapse: collapse; margin-top:10px; font-size:14px; }
    th, td { padding:8px; border-bottom:1px solid #f0f0f0; text-align:left; }
    th { font-weight:600; color:#333; }
    footer { margin-top:18px; color:#666; font-size:13px; }
    .counts { display:flex; flex-direction:column; gap:8px; max-height:420px; overflow:auto; }
    .count-item { display:flex; justify-content:space-between; align-items:center; padding:8px; border-radius:8px; border:1px solid #f1f1f1; background:#fbfbff; }
    .danger { background:#fff0f0; border-color:#ffd6d6; }
    /* modal cámara */
    .modal { display:none; position:fixed; inset:0; background:rgba(0,0,0,0.55); justify-content:center; align-items:center; z-index:1000; }
    .modal-content { background:#fff; padding:16px; border-radius:12px; width:95%; max-width:720px; box-shadow:0 6px 30px rgba(0,0,0,0.25); text-align:center; }
    #reader { width:100%; height:360px; }
    @media (max-width:900px){ .layout{ grid-template-columns:1fr; } #sidebar{ order:2 } }
  </style>
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>
  <!-- LOGIN -->
  <div id="loginScreen">
    <div id="loginBox">
      <h2>Iniciar sesión</h2>
      <input id="loginUser" type="text" placeholder="Usuario">
      <input id="loginPass" type="password" placeholder="Contraseña">
      <button id="btnDoLogin">Ingresar</button>
      <button id="btnShowRegister" class="secondary">Registrar nuevo admin</button>
      <div id="registerBox" style="display:none; margin-top:12px; text-align:left;">
        <h3>Registro de admin</h3>
        <input id="regUser" type="text" placeholder="Usuario">
        <input id="regPass" type="password" placeholder="Contraseña">
        <button id="btnDoRegister">Registrar</button>
      </div>
    </div>
  </div>

  <!-- APP -->
  <header>
    <div>
      <h1>Registro de asistencia — Código de barras / QR</h1>
      <div class="small">Funciona en móvil y en PC. Los datos se guardan localmente.</div>
    </div>
    <nav>
      <button id="tab-registro" class="primary">Registro</button>
      <button id="tab-admin">Administración</button>
      <button id="btnLogout">Cerrar sesión</button>
    </nav>
  </header>

  <main class="layout">
    <!-- Panel registro -->
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

    <!-- Sidebar -->
    <aside class="card" id="sidebar">
      <h3 style="margin:0">Conteo por persona</h3>
      <div class="counts" id="countsList" style="margin-top:10px;"></div>
      <div id="totalCount" class="small"></div>
    </aside>

    <!-- Panel admin -->
    <section id="panel-admin" class="card" style="display:none; grid-column:1/-1;">
      <h2>Administración</h2>
      <div style="display:flex; gap:8px; margin-bottom:10px;">
        <input id="formCode" type="text" placeholder="Código" />
        <input id="formName" type="text" placeholder="Nombre completo" />
        <button id="btnAddPerson">Agregar / Actualizar</button>
      </div>
      <table id="peopleTable" style="width:100%;">
        <thead><tr><th>Código</th><th>Nombre</th><th>Acciones</th></tr></thead>
        <tbody></tbody>
      </table>
    </section>
  </main>

  <footer><div class="small">Todo se guarda en localStorage.</div></footer>

  <!-- Modal cámara -->
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
    // --- Claves ---
    const KEY_PEOPLE='attendance_people';
    const KEY_SCANS='attendance_scans';
    const KEY_ADMINS='attendance_admins';
    const KEY_SESSION='attendance_session';

    // helpers
    const load=(k,d)=>JSON.parse(localStorage.getItem(k)||d);
    const save=(k,v)=>localStorage.setItem(k,JSON.stringify(v));
    let people=load(KEY_PEOPLE,"{}"), scans=load(KEY_SCANS,"[]"), admins=load(KEY_ADMINS,"{}");

    // default admin
    if(Object.keys(admins).length===0){ admins["admin"]={password:"admin123"}; save(KEY_ADMINS,admins); }

    // --- LOGIN ---
    const loginScreen=document.getElementById('loginScreen');
    const loginUser=document.getElementById('loginUser');
    const loginPass=document.getElementById('loginPass');
    const btnDoLogin=document.getElementById('btnDoLogin');
    const btnShowRegister=document.getElementById('btnShowRegister');
    const registerBox=document.getElementById('registerBox');
    const regUser=document.getElementById('regUser');
    const regPass=document.getElementById('regPass');
    const btnDoRegister=document.getElementById('btnDoRegister');
    const btnLogout=document.getElementById('btnLogout');

    function setSession(s){ save(KEY_SESSION,s); }
    function getSession(){ return load(KEY_SESSION,"null"); }
    function clearSession(){ localStorage.removeItem(KEY_SESSION); }

    if(getSession()) showApp();

    btnDoLogin.onclick=()=>{ 
      const u=loginUser.value.trim(), p=loginPass.value;
      if(admins[u]&&admins[u].password===p){ setSession({user:u}); showApp(); }
      else alert("Usuario o contraseña incorrectos");
    };
    btnShowRegister.onclick=()=>{ registerBox.style.display=registerBox.style.display==='none'?'block':'none'; };
    btnDoRegister.onclick=()=>{ 
      const u=regUser.value.trim(),p=regPass.value;
      if(!u||!p) return alert("Completa usuario y contraseña");
      if(admins[u]) return alert("Usuario ya existe");
      admins[u]={password:p}; save(KEY_ADMINS,admins);
      alert("Administrador registrado"); registerBox.style.display='none'; regUser.value=regPass.value='';
    };
    btnLogout.onclick=()=>{ clearSession(); location.reload(); };

    function showApp(){
      loginScreen.style.display='none';
      document.querySelector('header').style.display='flex';
      document.querySelector('main').style.display='grid';
      document.querySelector('footer').style.display='block';
      renderAll();
    }

    // --- Registro
    const codeInput=document.getElementById('codeInput'), recentTbody=document.querySelector('#recentTable tbody');
    const countsList=document.getElementById('countsList'), totalCountEl=document.getElementById('totalCount');
    const tabRegistro=document.getElementById('tab-registro'), tabAdmin=document.getElementById('tab-admin');
    const panelRegistro=document.getElementById('panel-registro'), panelAdmin=document.getElementById('panel-admin');
    const btnExport=document.getElementById('btn-export'), btnClear=document.getElementById('btn-clear');
    const formCode=document.getElementById('formCode'), formName=document.getElementById('formName'), btnAddPerson=document.getElementById('btnAddPerson'), peopleTbody=document.querySelector('#peopleTable tbody');

    function renderRecent(){ recentTbody.innerHTML=''; const last=scans.slice().reverse(); last.forEach((r,i)=>{ const d=r.ts.split('T'); recentTbody.innerHTML+=`<tr><td>${last.length-i}</td><td>${people[r.code]||'?'}</td><td>${r.code}</td><td>${d[0]}</td><td>${d[1].slice(0,8)}</td></tr>`; }); }
    function renderCounts(){ countsList.innerHTML=''; const map={}; scans.forEach(s=>map[s.code]=(map[s.code]||0)+1); Object.keys(map).forEach(c=>{ countsList.innerHTML+=`<div class="count-item"><div><b>${people[c]||'?'}</b><div class="small">Código: ${c}</div></div><div class="small">Veces: ${map[c]}</div></div>`; }); totalCountEl.textContent=scans.length+" registros"; }
    function renderPeople(){ peopleTbody.innerHTML=''; Object.keys(people).forEach(c=>{ peopleTbody.innerHTML+=`<tr><td>${c}</td><td>${people[c]}</td><td><button onclick="editPerson('${c}')">Editar</button> <button onclick="delPerson('${c}')">Eliminar</button></td></tr>`; }); }
    function renderAll(){ people=load(KEY_PEOPLE,"{}"); scans=load(KEY_SCANS,"[]"); admins=load(KEY_ADMINS,"{}"); renderRecent(); renderCounts(); renderPeople(); }

    codeInput.addEventListener('keydown',e=>{ if(e.key==='Enter'){ e.preventDefault(); if(codeInput.value.trim()){ addScan(codeInput.value.trim()); codeInput.value=''; } } });
    function addScan(code){ scans.push({code,ts:new Date().toISOString()}); save(KEY_SCANS,scans); renderAll(); }

    btnAddPerson.onclick=()=>{ if(formCode.value&&formName.value){ people[formCode.value]=formName.value; save(KEY_PEOPLE,people); renderAll(); formCode.value=formName.value=''; } };
    window.editPerson=(c)=>{ formCode.value=c; formName.value=people[c]; };
    window.delPerson=(c)=>{ if(confirm("Eliminar "+people[c]+"?")){ delete people[c]; save(KEY_PEOPLE,people); renderAll(); } };

    btnExport.onclick=()=>{ let csv='Código,Nombre,Fecha,Hora\n'; scans.forEach(s=>{ const d=s.ts.split('T'); csv+=`${s.code},"${people[s.code]||''}",${d[0]},${d[1].slice(0,8)}\n`; }); download(csv,'asistencia.csv'); };
    btnClear.onclick=()=>{ if(confirm("Borrar historial?")){ scans=[]; save(KEY_SCANS,scans); renderAll(); } };
    function download(text,fn){ const a=document.createElement('a'); a.href=URL.createObjectURL(new Blob([text],{type:'text/csv'})); a.download=fn; a.click(); }

    tabRegistro.onclick=()=>{ panelRegistro.style.display='block'; panelAdmin.style.display='none'; tabRegistro.classList.add('primary'); tabAdmin.classList.remove('primary'); };
    tabAdmin.onclick=()=>{ panelRegistro.style.display='none'; panelAdmin.style.display='block'; tabAdmin.classList.add('primary'); tabRegistro.classList.remove('primary'); };

    // --- Cámara
    const btnCamera=document.getElementById('btnCamera'), camModal=document.getElementById('cameraModal'), btnCloseCam=document.getElementById('btnCloseCam');
    const cameraSelect=document.getElementById('cameraSelect'), btnStartCam=document.getElementById('btnStartCam');
    let html5QrCode=null;
    btnCamera.onclick=async ()=>{ const cams=await Html5Qrcode.getCameras().catch(()=>[]); if(!cams.length) return alert("No hay cámara"); cameraSelect.innerHTML=cams.map(c=>`<option value="${c.id}">${c.label||c.id}</option>`).join(''); camModal.style.display='flex'; };
    btnStartCam.onclick=()=>{ const id=cameraSelect.value; if(!id) return; if(html5QrCode) html5QrCode.stop(); html5QrCode=new Html5Qrcode("reader"); html5QrCode.start(id,{fps:10,qrbox:{width:320,height:200}},txt=>{ addScan(txt); alert("Código: "+txt); closeScanner(); camModal.style.display='none'; }); };
    btnCloseCam.onclick=()=>{ closeScanner(); camModal.style.display='none'; };
    function closeScanner(){ if(html5QrCode){ html5QrCode.stop().catch(()=>{}); html5QrCode.clear(); html5QrCode=null; } }

    // inicializar
    (function(){ if(Object.keys(people).length===0){ save(KEY_PEOPLE,{"7772234":"Maicol Sanchez","1234567":"Daiyelin"}); } renderAll(); })();
  </script>
</body>
</html>
