<!DOCTYPE html>
<html>
<head>
<title>Setup-Assistent</title>
<style>
html, body {
  margin:0; padding:0;
  width:100vw; height:100vh;
  display:flex; justify-content:center; align-items:center;
  font-family:'Segoe UI', Tahoma, sans-serif;
  background:linear-gradient(135deg,#4a90e2,#50e3c2);
}
#container { width:90%; max-width:700px; }

.card {
  background:#fff;
  border-radius:12px;
  padding:24px;
  box-shadow:0 8px 20px rgba(0,0,0,.2);
  text-align:center;
}

button {
  margin:10px 5px;
  padding:12px 20px;
  font-size:16px;
  border:none;
  border-radius:8px;
  cursor:pointer;
  background:#4a90e2;
  color:white;
}
button:hover { background:#357ABD; }

.progress { font-weight:bold; font-size:18px; margin-bottom:10px; }

.site-grid {
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(140px,1fr));
  gap:16px;
  margin-top:12px;
}

.site-card {
  border:1px solid #ddd;
  border-radius:8px;
  padding:10px;
  background:#f9f9f9;
}

.site-card img {
  width:48px;
  height:48px;
  object-fit:contain;
}

.domain { font-size:12px; color:#555; }

.notice {
  margin-top:12px;
  background:#fff3cd;
  border:1px solid #ffeeba;
  border-radius:8px;
  padding:12px;
  color:#856404;
}
</style>
</head>
<body>

<div id="container"></div>

<script>
const websites = [
  {name:"Microsoft-Account", url:"https://myaccount.microsoft.com/"},
  {name:"SharePoint", url:"https://ohvar.sharepoint.com/sites/oberlin-berufsbildungswerk"},
  {name:"OneDrive", url:"https://ohvar-my.sharepoint.com/"},
  {name:"Outlook", url:"https://outlook.office.com/mail/"},
  {name:"WebUntis", url:"https://s200440.webuntis.com/WebUntis/?school=s200440#/basic/login"}
];

let openedTabs = [];

const urls = websites.map(w => {
  const domain = new URL(w.url).hostname;
  return {
    ...w,
    domain,
    icon:`https://www.google.com/s2/favicons?sz=64&domain=${domain}`
  };
});

init();

/* ---------- INIT ---------- */
function init() {
  showStart();
}

/* ---------- UI ---------- */
function showStart() {
  document.getElementById("container").innerHTML = `
    <div class="card">
      <p class="progress">Schritt 1 von 2</p>
      <button onclick="testPopups()">Websites testen</button>
      <p>Folgende Webseiten werden geöffnet:</p>
      <div class="site-grid">
        ${urls.map(u=>`
          <div class="site-card">
            <img src="${u.icon}">
            <div><b>${u.name}</b></div>
            <div class="domain">${u.domain}</div>
          </div>
        `).join("")}
      </div>
    </div>
  `;
}

/* ---------- Tabs ---------- */
function openTab(url) {
  const w = window.open(url,'_blank');
  if (w) openedTabs.push(w);
}

/* ---------- Popup-Test ---------- */
function testPopups() {
  openedTabs = [];
  urls.forEach(u => openTab(u.url));

  setTimeout(() => {
    closeAllTabs();
  }, 3000);
}

/* ---------- Tabs schließen (mehrere Versuche) ---------- */
function closeAllTabs() {
  let attempts = 0;

  const interval = setInterval(() => {
    attempts++;

    openedTabs.forEach(tab => {
      if (!tab || tab.closed) return;
      try { tab.close(); } catch(e){}
    });

    openedTabs = openedTabs.filter(t => t && !t.closed);

    if (openedTabs.length === 0 || attempts >= 20) {
      clearInterval(interval);
      showDone();
    }
  }, 200);
}

/* ---------- Fertig ---------- */
function showDone() {
  document.getElementById("container").innerHTML = `
    <div class="card">
      <p class="progress">✅ Setup abgeschlossen</p>
      <div class="notice">
        Das Fenster schließt sich automatisch.
      </div>
    </div>
  `;

  // Setup-Seite schließen (nur erlaubt, weil per window.open geöffnet)
  setTimeout(() => window.close(), 1500);
}
</script>

</body>
</html>
