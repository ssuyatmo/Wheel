<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Inspection HD785-7</title>
<style>
body {
  font-family: "Poppins", sans-serif;
  margin: 0;
  background: linear-gradient(to bottom right, #d8e9a8, #9cdba6);
  color: #222;
}
header {
  position: fixed; top: 0; left: 0; width: 100%;
  background: linear-gradient(to right, #f1d28a, #f5e4a2);
  color: #1a1a1a; text-align: center;
  padding: 10px 0; font-weight: bold; font-size: 18px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2); z-index: 100;
}
.fixed-header {
  position: fixed; top: 60px; left: 0; width: 100%;
  background: #d8e9a8; z-index: 99;
  padding: 8px 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.15);
}
.fixed-header .card {
  background: #fff; border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px; margin-bottom: 8px;
}
label { display: block; margin-bottom: 6px; font-weight: 500; }
.container { padding: 10px; max-width: 600px; margin: auto; margin-top: 320px; }
.card {
  background: #fff; border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px; margin-bottom: 10px;
}
h3 { background: #cfe8cf; padding: 6px; border-radius: 6px; font-size: 15px; }
.label { display: flex; justify-content: space-between; align-items: center; margin: 4px 0; }
.label span { flex: 1; font-size: 14px; }
.ok, .notok { padding: 4px 10px; border: none; border-radius: 6px; cursor: pointer; font-weight: bold; }
.ok { background: #a4f5a4; } .ok.active { background: #4caf50; color:white; }
.notok { background: #f5a4a4; } .notok.active { background: #f44336; color:white; }
textarea, input[type="text"], input[type="number"], input[type="date"], select {
  width: 100%; border-radius: 6px; border: 1px solid #ccc;
  resize: none; margin-top: 4px; padding: 5px; font-size: 13px;
}
.buttons { display: flex; justify-content: space-around; margin-top: 10px; }
button.main {
  padding: 8px 14px; border: none; border-radius: 8px;
  font-weight: bold; cursor: pointer; background: #f1d28a;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}
#previewPopup {
  display: none; position: fixed; top: 50%; left: 50%;
  transform: translate(-50%, -50%); background: white;
  border-radius: 10px; box-shadow: 0 0 20px rgba(0,0,0,0.3);
  width: 90%; max-width: 400px; z-index: 200; padding: 15px;
}
#previewPopup pre { max-height: 60vh; overflow-y: auto; white-space: pre-wrap; font-size: 13px; margin-bottom: 10px; }
#popupButtons { display: flex; justify-content: space-around; }
#photoPreview { width: 100%; border-radius: 10px; margin-top: 8px; display:none; }
@media (max-width: 480px) {
  .container { margin-top: 300px; }
}
</style>
</head>
<body>

<header>🌿 QA Inspection Komatsu HD785-7</header>

<div class="fixed-header">
  <div class="card">
    <label>📷 Upload Foto Unit:</label>
    <input type="file" id="photoInput" accept="image/*" onchange="previewPhoto(event)">
    <img id="photoPreview" alt="Preview Foto">
  </div>

  <div class="card">
    <label>Pilih Jenis QA:
      <select id="qaType">
        <option value="QA1">QA-1 (Pre Inspection)</option>
        <option value="QA7">QA-7 (Final Inspection)</option>
      </select>
    </label>
  </div>

  <div class="card">
    <label>📅 Tanggal:</label><input type="date" id="tgl">
    <label>👷 Mekanik:</label><input type="text" id="mekanik">
    <label>🚗 CN Unit:</label><input type="text" id="cn">
    <label>⌛ HM:</label><input type="number" id="hm">
  </div>
</div>

<div class="container" id="mainContent">
  <div id="sections"></div>
  <div class="buttons">
    <button class="main" onclick="showPreview()">🧾 Preview</button>
    <button class="main" onclick="sendToWhatsApp()">📤 Send WA</button>
  </div>
</div>

<div id="previewPopup">
  <img id="popupImage" style="width:100%;border-radius:10px;margin-bottom:8px;display:none;">
  <pre id="previewText"></pre>
  <div id="popupButtons">
    <button onclick="copyText()">📋 Copy</button>
    <button onclick="sendToWhatsApp()">📤 Send WA</button>
    <button onclick="closePopup()">❌ Close</button>
  </div>
</div>

<script>
let uploadedPhotoBase64 = "";

function previewPhoto(event){
  const file = event.target.files[0];
  if(!file) return;
  const reader = new FileReader();
  reader.onload = function(e){
    uploadedPhotoBase64 = e.target.result;
    const preview = document.getElementById('photoPreview');
    preview.src = e.target.result;
    preview.style.display = 'block';
  };
  reader.readAsDataURL(file);
}

const inspectionSections = [
  { name: "Oil Level", items: ["Engine oil level","Transmission oil level","Hydraulic oil level"] },
  { name: "Engine Area", items: ["Belt tension","Engine oil leakage"] }
];
const sectionsData = {
  QA1:{title:"Validasi Checklist QA-1",items:["Job Card","Form QA","Form Check List A"]},
  QA7:{title:"Validasi Checklist QA-7",items:["Job Card","Form QA","Form Check List B"]}
};

function renderSections(){
  const qaType=document.getElementById('qaType').value;
  let html="";
  inspectionSections.forEach(sec=>{
    html+=`<div class="card"><h3>${sec.name}</h3>`;
    sec.items.forEach((item,idx)=>{
      const id=sec.name.replace(/\s+/g,'')+idx;
      html+=`<div class="label"><span>${item}</span>
        <div><button class="ok active" onclick="toggleButton(this,'OK','${id}')">OK</button>
        <button class="notok" onclick="toggleButton(this,'Not OK','${id}')">Not OK</button></div></div>`;
    });
    html+="</div>";
  });
  document.getElementById('sections').innerHTML=html;
}
renderSections();

function toggleButton(el,val,id){
  const parent=el.parentElement;
  parent.querySelectorAll('button').forEach(b=>b.classList.remove('active'));
  el.classList.add('active');
}

function showPreview(){
  document.getElementById('previewPopup').style.display='block';
  const popupImg=document.getElementById('popupImage');
  if(uploadedPhotoBase64){
    popupImg.src=uploadedPhotoBase64;
    popupImg.style.display='block';
  }
  document.getElementById('previewText').textContent=generateText();
}

function generateText(){
  let text=`*QA Inspection*\n📅 ${tgl.value}\n👷 ${mekanik.value}\n🚗 ${cn.value}\n⌛ ${hm.value}\n\n`;
  inspectionSections.forEach(sec=>{
    text+=`🧩 *${sec.name}*\n`;
    sec.items.forEach((item,idx)=>{
      const id=sec.name.replace(/\s+/g,'')+idx;
      const ok=document.querySelector(`[onclick*="${id}"].ok.active`);
      const notok=document.querySelector(`[onclick*="${id}"].notok.active`);
      text+=`${item}: ${ok?'✅ OK':notok?'❌ Not OK':'❔'}\n`;
    });
    text+='\n';
  });
  return text;
}

function sendToWhatsApp(){
  const msg=encodeURIComponent(generateText());
  if(uploadedPhotoBase64){
    alert("📸 Gambar tidak bisa dikirim langsung via link WA.\nSilakan kirim manual bersama teks:\n\n1️⃣ Salin teks\n2️⃣ Kirim foto + teks ke grup WA.");
  }
  window.open(`https://wa.me/?text=${msg}`,'_blank');
}

function copyText(){
  navigator.clipboard.writeText(document.getElementById('previewText').textContent);
  alert("✅ Teks disalin!");
}
function closePopup(){ document.getElementById('previewPopup').style.display='none'; }
</script>

</body>
</html>
