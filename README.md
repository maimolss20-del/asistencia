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
  </style>
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>
  <!-- LOGIN PANTALLA -->
  <div id="loginScreen">
    <div id="loginBox">
      <h2>Iniciar sesión</h2>
      <input id="loginUser" type="text" placeholder="Teléfono / Usuario">
      <input id="loginPass" type="password" placeholder="Contraseña">
      <button id="btnDoLogin">Ingresar</button>
      <button id="btnShowRegister" class="secondary">Registrar nuevo admin</button>

      <div id="registerBox" style="display:none; margin-top:12px; text-align:left;">
        <h3>Registro de admin</h3>
        <input id="regUser" type="text" placeholder="Teléfono / Usuario">
        <input id="regPass" type="password" placeholder="Contraseña">
        <input id="regName" type="text" placeholder="Nombre completo">
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
    </nav>
  </header>

  <main class="layout">
    <!-- Aquí va TODO el contenido de tu página (igual al que me pasaste) -->
  </main>

  <footer>
    <div class="small">Todo se guarda localmente en tu navegador (localStorage).</div>
  </footer>

  <script>
    // --- Claves localStorage
    const KEY_ADMINS = 'attendance_admins_v2';
    const KEY_SESSION = 'attendance_session_v2';

    function loadAdmins(){ return JSON.parse(localStorage.getItem(KEY_ADMINS) || '{}'); }
    function saveAdmins(a){ localStorage.setItem(KEY_ADMINS, JSON.stringify(a)); }
    function getSession(){ return JSON.parse(localStorage.getItem(KEY_SESSION) || 'null'); }
    function setSession(s){ localStorage.setItem(KEY_SESSION, JSON.stringify(s)); }
    function clearSession(){ localStorage.removeItem(KEY_SESSION); }

    let admins = loadAdmins();

    // Crear admin por defecto si no existe ninguno
    if(Object.keys(admins).length===0){ 
      admins['3000000000'] = { password:'admin123', name:'Admin' }; 
      saveAdmins(admins);
    }

    const loginScreen=document.getElementById('loginScreen');
    const log
