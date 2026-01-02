<!DOCTYPE html>
<html>
<head>
<title>Setup-Assistent</title>
<style>
html, body {
  margin:0;
  padding:0;
  width:100vw;
  height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg,#4a90e2,#50e3c2);
}
#container {
  width:90%;
  max-width:700px;
}
.card {
  background:#fff;
  border-radius:12px;
  padding:24px;
  box-shadow:0 8px 20px rgba(0,0,0,0.2);
  text-align:center;
  margin-bottom:10px;
  transition: transform 0.3s, background 0.3s;
}
.card.active {
  transform: scale(1.05);
  background:#e0f7ff;
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
  transition:0.2s;
}
button:hover {
  background:#357ABD;
}
.progress {
  font-weight:bold;
  font-size:18px;
  margin-bottom:12px;
}
p { margin:10px 0; font-size:16px; }

/* -------- Website Grid -------- */
.site-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 16px;
  margin-top:12px;
}
.site-card {
  text-align:center;
  border:1px solid #ddd;
  border-radius:8px;
  padding:8px;
  background:#f9f9f9;
}
.site-card img {
  width:48px;
  height:48px;
  object-fit: contain;
  margin-bottom:4px;
}
.site-card .name {
  font-weight:bold;
  font-size:14px;
}
.site-card .domain {
  font-size:12px;
  color:#555;
}
</style>
</head>
<body>

<div id="container"></div>

<script>
const websites = [
  {name:"Microsoft-Account", url:"https://myaccount.microsoft.com/", icon:"https://upload.wikimedia.org/wikipedia/commons/thumb/4/44/Microsoft_logo.svg/960px-Microsoft_logo.svg.png"},
  {name:"SharePoint", url:"https://ohvar.sharepoint.com/sites/oberlin-berufsbildungswerk", icon:"https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRdovwLKgna5FZfrlGnpIUQIxaO1DRB-osA7g&s"},
  {name:"Onedrive", url:"https://ohvar-my.sharepoint.com/", icon:"https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/Microsoft_OneDrive_Icon_%282025_-_present%29.svg/2560px-Microsoft_OneDrive_Icon_%282025_-_present%29.svg.png"},
  {name:"Onedrive", url:"https://ohvar-my.sharepoint.com/", icon:"https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/Microsoft_OneDrive_Icon_%282025_-_present%29.svg/2560px-Microsoft_OneDrive_Icon_%282025_-_present%29.svg.png"},
  {name:"Outlook", url:"https://outlook.office.com/mail/", icon:"https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQzyBs_Uv8dfK-99-G8hsyojYmxDBKgwKO6Xg&s"},
  {name:"WebUntis", url:"https://s200440.webuntis.com/WebUntis/?school=s200440#/basic/login", icon:"https://www.untis.at/fileadmin/_processed_/8/3/csm_app_icon_9a229b3c41.jpg"}
];

let firstTab = null;
let openedTabs = [];

// Domain extrahieren, Icon wählen
const urls = websites.map(w => {
  const domain = new URL(w.url).hostname;
  const icon = w.icon || `https://www.google.com/s2/favicons?sz=64&domain=${domain}`;
  return {...w, domain, icon};
});

localStorage.clear();
init();

/* ---------- INIT ---------- */
function init() {
  const step = localStorage.getItem("step") || "start";
  if (step === "done") showDone();
  else if (step === "firstDone") showContinue();
  else showStart();
}

/* ---------- UI ---------- */
function showStart() {
  const siteHtml = urls.map(u => `
    <div class="site-card">
      <div class="name">${u.name}</div>
      <img src="${u.icon}" alt="${u.name}">
      <div class="domain">${u.domain}</div>
    </div>
  `).join('');

  document.getElementById("container").innerHTML = `
    <div class="card">
      <p class="progress">Schritt 1 von 2</p>
      <button onclick="testPopups()">Websites testen</button>
      <p>Status: <span id="status">❓</span></p>
      <p>Folgende Webseiten werden geöffnet:</p>
      <div class="site-grid">${siteHtml}</div>
    </div>
  `;
}

function showContinue() {
  document.getElementById("container").innerHTML = `
    <div class="card">
      <p class="progress">Schritt 2 von 2</p>
      <p>Du bist angemeldet.</p>
      <button onclick="openRest()">Weiter</button>
      <button onclick="reset()">Setup neu starten</button>
    </div>
  `;
}

function showDone() {
  document.getElementById("container").innerHTML = `
    <div class="card">
      <p class="progress">✅ Setup abgeschlossen</p>
      <button onclick="reset()">Neu starten</button>
    </div>
  `;
}

/* ---------- Tab-Logik ---------- */
function openTab(url) {
  const w = window.open(url,'_blank');
  if(w) openedTabs.push(w);
  return w;
}

/* ---------- Tabs schließen mit Versuch + Meldung erst danach ---------- */
function closeAllTabs() {
  const manualCloseTabs = [];
  let attempts = 0;

  const interval = setInterval(() => {
    attempts++;
    let remaining = [];

    openedTabs.forEach(tab => {
      if (!tab || tab.closed) return;
      try {
        tab.close();
        if (!tab.closed) manualCloseTabs.push(tab);
      } catch(e) {
        manualCloseTabs.push(tab);
      }
    });

    openedTabs = openedTabs.filter(tab => tab && !tab.closed);

    // Stop nach max 20 Versuchen (4 Sekunden), dann Meldung
    if (openedTabs.length === 0 || attempts >= 20) {
      clearInterval(interval);
      if (manualCloseTabs.length > 0) {
        alert("Einige Tabs (z.B. Outlook) konnten nicht automatisch geschlossen werden. Bitte schließe sie manuell.");
      }
    }

  }, 200);
}

/* ---------- Pop-up Test ---------- */
function testPopups() {
  openedTabs = [];
  let opened = 0;

  for (const u of urls) {
    const w = openTab(u.url);
    if (w) opened++;
  }

  setTimeout(() => {
    closeAllTabs();
    document.getElementById("status").textContent = opened === urls.length ? "✅" : "❌";
    
    document.getElementById("container").innerHTML = `
      <div class="card">
        <p>Wurden alle Tabs geöffnet?</p>
        <button onclick="prepareFirst()">Ja</button>
        <button onclick="showHelp()">Nein</button>
        <button onclick="testPopups()">Erneut testen</button>
      </div>
    `;
  },3000);
}

/* ---------- Schritt 1 (Anmeldung) ---------- */
function prepareFirst() {
  document.getElementById("container").innerHTML = `
    <div class="card">
      Wenn du dich <b>angemeldet</b> hast, komm zurück.<br><br>
      <button onclick="openFirst()">OK</button>
    </div>
  `;
}

function openFirst() {
  alert("Wenn du dich angemeldet hast, dann komm zurück und ich öffne die restlichen Webseiten.");
  firstTab = openTab(urls[0].url);
  if(firstTab) firstTab.focus();
  localStorage.setItem("step","firstDone");
  showContinue();
}

/* ---------- Schritt 2 (Rest öffnen) ---------- */
function openRest() {
  const container = document.getElementById("container");
  const card = container.querySelector(".card");

  for(let i=1; i<urls.length; i++) {
    setTimeout(()=> {
      card.classList.add("active");
      openTab(urls[i].url);
      setTimeout(()=>card.classList.remove("active"),500);
    }, 200*i);
  }

  localStorage.setItem("step","done");
  setTimeout(showDone, 200*urls.length + 500);
}

/* ---------- Hilfe ---------- */
function showHelp() {
  document.getElementById("container").innerHTML = `
    <div class="card">
      Öffne Browser-Einstellungen → Datenschutz & Sicherheit →<br>
      <b>Pop-ups und Weiterleitungen erlauben</b>.<br>
      Tabs mit OAuth-Login (z.B. Outlook) müssen manuell geschlossen werden.
    </div>
  `;
}

/* ---------- Reset ---------- */
function reset() {
  localStorage.clear();
  firstTab = null;
  openedTabs = [];
  init();
}
</script>

</body>
</html>
