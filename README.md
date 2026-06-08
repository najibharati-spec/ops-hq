<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>OPS HQ — Connexion</title>
  <link rel="stylesheet" href="css/style.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@2.44.0/tabler-icons.min.css" />
</head>
<body class="login-body">

  <div class="login-wrapper">
    <div class="login-card">

      <div class="login-logo">
        <span class="logo-text">OPS HQ</span>
        <span class="logo-sub">Field Management System</span>
      </div>

      <div class="login-form">
        <div class="form-group">
          <label>Identifiant</label>
          <div class="input-icon">
            <i class="ti ti-user"></i>
            <input type="text" id="loginEmail" placeholder="votre.nom@opshq.ma" autocomplete="username" />
          </div>
        </div>
        <div class="form-group">
          <label>Mot de passe</label>
          <div class="input-icon">
            <i class="ti ti-lock"></i>
            <input type="password" id="loginPassword" placeholder="••••••••" autocomplete="current-password" />
            <i class="ti ti-eye toggle-pw" id="togglePw"></i>
          </div>
        </div>

        <div id="loginError" class="error-msg" style="display:none;"></div>

        <button class="btn-login" id="loginBtn" onclick="handleLogin()">
          <span id="loginBtnText">Se connecter</span>
          <i class="ti ti-arrow-right"></i>
        </button>
      </div>

      <div class="login-footer">
        <span>Système réservé aux membres autorisés</span>
      </div>

    </div>
  </div>

  <!-- Firebase -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
    import { getAuth, signInWithEmailAndPassword, onAuthStateChanged }
      from "https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js";
    import { getFirestore, doc, getDoc }
      from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

    // ============================================================
    //  REMPLACE PAR TA CONFIG FIREBASE ICI
    // ============================================================
    const firebaseConfig = {
      apiKey:            "VOTRE_API_KEY",
      authDomain:        "VOTRE_PROJECT.firebaseapp.com",
      projectId:         "VOTRE_PROJECT_ID",
      storageBucket:     "VOTRE_PROJECT.appspot.com",
      messagingSenderId: "VOTRE_SENDER_ID",
      appId:             "VOTRE_APP_ID"
    };
    // ============================================================

    const app  = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db   = getFirestore(app);

    // Si déjà connecté → rediriger
    onAuthStateChanged(auth, async (user) => {
      if (user) {
        const snap = await getDoc(doc(db, "users", user.uid));
        if (snap.exists()) {
          const role = snap.data().role;
          if (role === "admin") {
            window.location.href = "pages/dashboard.html";
          } else {
            window.location.href = "pages/agent.html";
          }
        }
      }
    });

    window.handleLogin = async function () {
      const email    = document.getElementById("loginEmail").value.trim();
      const password = document.getElementById("loginPassword").value;
      const btn      = document.getElementById("loginBtn");
      const errDiv   = document.getElementById("loginError");
      const btnText  = document.getElementById("loginBtnText");

      errDiv.style.display = "none";
      btn.disabled = true;
      btnText.textContent  = "Connexion...";

      try {
        const cred = await signInWithEmailAndPassword(auth, email, password);
        const snap = await getDoc(doc(db, "users", cred.user.uid));
        if (!snap.exists()) throw new Error("Compte introuvable.");
        const role = snap.data().role;
        window.location.href = role === "admin"
          ? "pages/dashboard.html"
          : "pages/agent.html";
      } catch (e) {
        errDiv.textContent     = "Identifiant ou mot de passe incorrect.";
        errDiv.style.display   = "block";
        btn.disabled           = false;
        btnText.textContent    = "Se connecter";
      }
    };

    // Toggle password visibility
    document.getElementById("togglePw").addEventListener("click", () => {
      const inp = document.getElementById("loginPassword");
      const ico = document.getElementById("togglePw");
      if (inp.type === "password") {
        inp.type = "text";
        ico.classList.replace("ti-eye", "ti-eye-off");
      } else {
        inp.type = "password";
        ico.classList.replace("ti-eye-off", "ti-eye");
      }
    });

    // Enter key
    document.getElementById("loginPassword").addEventListener("keydown", (e) => {
      if (e.key === "Enter") window.handleLogin();
    });
  </script>
</body>
</html>
