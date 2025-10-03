<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <title>Asistencia por código (QR / Barras) — v3</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <meta name="theme-color" content="#2563eb" />
  <style>
    :root{ --accent:#2563eb; --muted:#777; --card:#fff; --radius:10px; }
    *{box-sizing:border-box}
    body { font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial; max-width:1200px; margin:18px auto; padding:12px; color:#111; background:#f6f7fb; }
    header, main, footer { display:none; } /* se oculta hasta login */

    /* --- LOGIN SCREEN --- */
    #loginScreen { 
      position:fixed; top:0; left:0; width:100%; height:100%; 
      background:#f0f2f9; display:flex; justify-content:center; align-items:center; 
    }
    #loginBox { 
      background:white; padding:24px; border-radius:12px; box-shadow:0 8px 24px rgba(0,0,0,0.2); 
      width:95%; max-width:400px; text-align:center;
    }
    #loginBox h2 { margin-top:0; }
    #loginBox input { width:100%; padding:10px; margin:6px 0; border:1px solid #ccc; border-radius:8px; }
    #loginBox button { margin-top:8px; padding:10px; width:100%; border:none; border-radius:8px; background:var(--accent); color:white; font-weight:600; cursor:pointer; }
    #loginBox button.secondary { background:#eee; color:#333; }
    .small { font-size:13px; color:#666; margin-top:10px; }

    header { display:flex; justify-content:space-between; align-items:center; background:#fff; padding:12px 16px; border-radius:12px; box-shadow:0 4px 12px rgba(0,0,0,0.1); margin-bottom:16px; }
    nav button { margin-left:8px; padding:8px 14px; border:none; border-radius:8px; background:#eee; cursor:pointer; }
    nav button.primary { background:var(--accent); color:white; }
    #btnLogout { background:#dc2626; color:white; }
    main { display:grid; gap:16px; }
  </style>
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>
  <!-- LOGIN PANTALLA -->
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

  <!-- CONTENIDO ORIGINAL -->
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
    <!-- Aquí va TODO el contenido de tu página (igual al que ya tienes en tu versión anterior) -->
    <section>
      <h2>Escáner QR/Barras</h2>
      <div id="reader" style="width:100%; max-width:400px;"></div>
    </section>
  </main>

  <footer>
    <div class="small">Todo se guarda localmente en tu navegador (localStorage).</div>
  </footer>

  <script>
    // --- Claves localStorage
    const KEY_ADMINS = 'attendance_admins_v3';
    const KEY_SESSION = 'attendance_session_v3';

    function loadAdmins(){ return JSON.parse(localStorage.getItem(KEY_ADMINS) || '{}'); }
    function saveAdmins(a){ localStorage.setItem(KEY_ADMINS, JSON.stringify(a)); }
    function getSession(){ return JSON.parse(localStorage.getItem(KEY_SESSION) || 'null'); }
    function setSession(s){ localStorage.setItem(KEY_SESSION, JSON.stringify(s)); }
    function clearSession(){ localStorage.removeItem(KEY_SESSION); }

    let admins = loadAdmins();

    // Crear admin por defecto si no existe ninguno
    if(Object.keys(admins).length===0){ 
      admins['admin'] = { password:'admin123' }; 
      saveAdmins(admins);
    }

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

    // Mostrar contenido si ya hay sesión
    if(getSession()){ showApp(); }

    btnDoLogin.addEventListener('click',()=>{
      const u=loginUser.value.trim(); const p=loginPass.value;
      if(!u||!p){ alert("Completa los campos"); return; }
      const a=admins[u];
      if(a && a.password===p){ setSession({user:u}); showApp(); }
      else alert("Usuario o contraseña incorrectos");
    });

    btnShowRegister.addEventListener('click',()=>{ 
      registerBox.style.display = registerBox.style.display==='none'?'block':'none'; 
    });

    btnDoRegister.addEventListener('click',()=>{
      const u=regUser.value.trim(), p=regPass.value;
      if(!u||!p){ alert("Completa usuario y contraseña"); return; }
      if(admins[u]){ alert("Ese usuario ya existe"); return; }
      admins[u]={password:p}; saveAdmins(admins);
      alert("Administrador registrado. Ahora puedes iniciar sesión.");
      registerBox.style.display='none'; regUser.value=regPass.value='';
    });

    btnLogout.addEventListener('click',()=>{
      clearSession();
      document.querySelector('header').style.display='none';
      document.querySelector('main').style.display='none';
      document.querySelector('footer').style.display='none';
      loginScreen.style.display='flex';
      loginUser.value=loginPass.value="";
    });

    function showApp(){
      loginScreen.style.display='none';
      document.querySelector('header').style.display='flex';
      document.querySelector('main').style.display='grid';
      document.querySelector('footer').style.display='block';
    }
  </script>
</body>
</html>
