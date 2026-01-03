<!DOCTYPE html>
<html>
<head>
<title>Setup-Assistent</title>
<style>
html,body{
  margin:0;height:100%;
  display:flex;justify-content:center;align-items:center;
  font-family:Segoe UI;background:linear-gradient(135deg,#4a90e2,#50e3c2)
}
#container{width:90%;max-width:700px}
.card{
  background:#fff;padding:24px;border-radius:12px;
  box-shadow:0 8px 20px rgba(0,0,0,.2);text-align:center
}
button{
  padding:12px 20px;border:0;border-radius:8px;
  background:#4a90e2;color:#fff;font-size:16px;cursor:pointer
}
.notice{
  margin-top:12px;padding:12px;border-radius:8px;
  background:#fff3cd;border:1px solid #ffeeba;color:#856404
}
</style>
</head>
<body>

<div id="container"></div>

<script>
const kiosk = new URLSearchParams(location.search).get("kiosk")==="1";

const websites=[
 {name:"Microsoft",url:"https://myaccount.microsoft.com/"},
 {name:"SharePoint",url:"https://ohvar.sharepoint.com/sites/oberlin-berufsbildungswerk"},
 {name:"OneDrive",url:"https://ohvar-my.sharepoint.com/"},
 {name:"Outlook",url:"https://outlook.office.com/mail/"},
 {name:"WebUntis",url:"https://s200440.webuntis.com/WebUntis/?school=s200440#/basic/login"}
];

let tabs=[];

/* ---------- Schutz im Kiosk-Modus ---------- */
if(kiosk){
  window.onbeforeunload=e=>e.returnValue="";
  document.addEventListener("keydown",e=>{
    if(e.key==="F5"||e.ctrlKey||e.altKey)e.preventDefault();
  });
}

/* ---------- UI ---------- */
document.getElementById("container").innerHTML=`
<div class="card">
  <p><b>Setup wird vorbereitet…</b></p>
  <button onclick="start()">Start</button>
</div>`;

/* ---------- Start ---------- */
function start(){
  websites.forEach(w=>{
    const t=window.open(w.url,"_blank");
    if(t)tabs.push(t);
  });
  setTimeout(closeTabs,3000);
}

/* ---------- Tabs schließen ---------- */
function closeTabs(){
  let tries=0;
  const i=setInterval(()=>{
    tries++;
    tabs.forEach(t=>{try{t.close()}catch{}});
    tabs=tabs.filter(t=>t&&!t.closed);

    if(tabs.length===0||tries>=20){
      clearInterval(i);
      finish();
    }
  },200);
}

/* ---------- Abschluss ---------- */
function finish(){
  document.getElementById("container").innerHTML=`
  <div class="card">
    <h3>✅ Setup abgeschlossen</h3>
    <div id="fallback" class="notice" style="display:none">
      Bitte dieses Tab manuell schließen.
    </div>
  </div>`;

  /* 🧠 Erkennung ob window.close erlaubt */
  setTimeout(()=>{
    const test=window.open("","_self");
    try{
      window.close();
      setTimeout(()=>{
        if(!window.closed)
          document.getElementById("fallback").style.display="block";
      },500);
    }catch{
      document.getElementById("fallback").style.display="block";
    }
  },800);
}
</script>
</body>
</html>
