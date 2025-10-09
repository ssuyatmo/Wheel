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
  position: fixed;
  top: 0; left: 0;
  width: 100%;
  background: linear-gradient(to right, #f1d28a, #f5e4a2);
  color: #1a1a1a;
  text-align: center;
  padding: 10px 0;
  font-weight: bold;
  font-size: 18px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  z-index: 100;
}
.fixed-header {
  position: fixed;
  top: 60px;
  left: 0;
  width: 100%;
  background: #d8e9a8;
  z-index: 99;
  padding: 8px 10px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.15);
}
.fixed-header .card {
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px;
  margin-bottom: 8px;
}
.container {
  padding: 10px;
  max-width: 600px;
  margin: auto;
  margin-top: 280px;
}
.card {
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px;
  margin-bottom: 10px;
}
.label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 4px 0;
}
.label span { flex: 1; font-size: 14px; }
.label button {
  margin-left: 5px;
  padding: 4px 10px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
}
.ok { background: #a4f5a4; }
.ok.active { background: #4caf50; color:white; }
.notok { background: #f5a4a4; }
.notok.active { background: #f44336; color:white; }
textarea, input, select {
  width: 100%;
  border-radius: 6px;
  border: 1px solid #ccc;
  resize: none;
  margin-top: 4px;
  padding: 5px;
  font-size: 13px;
}
input[type="file"] {
  margin-top: 5px;
}
button.main {
  padding: 8px 14px;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  background: #f1d28a;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}
.buttons {
  display: flex;
  justify-content: space-around;
  margin-top: 10px;
}
</style>
</head>
<body>

<header>🌿 QA Inspection Komatsu HD785-7</header>

<!-- INPUT FIXED -->
<div class="fixed-header">
  <div class="card">
    <label>📋 Jenis QA:
      <select id="qaType">
        <option value="QA1">QA-1 (Pre Inspection)</option>
        <option value="QA7">QA-7 (Final Inspection)</option>
      </select>
    </label>
    <label>📅 Tanggal:</label><input type="date" id="tgl">
    <label>👷 Mekanik:</label><input type="text" id="mekanik">
    <label>🚗 CN Unit:</label><input type="text" id="cn">
    <label>⌛ HM:</label><input type="number" id="hm">
    <label>📷 Foto Umum Unit:</label><input type="file" id="fotoUnit" accept="image/*">
  </div>
</div>

<!-- ISI -->
<div class="container" id="mainContent">
  <div id="sections"></div>
  <div class="card">
    <h3>⚠️ Deviation Tambahan</h3>
    <textarea id="manualDeviation" rows="3" placeholder="Tambahkan deviation manual..."></textarea>
    <label>📸 Foto Deviation:</label>
    <input type="file" id="deviationImage" accept="image/*">
  </div>

  <div class="buttons">
    <button class="main" onclick="sendToWhatsApp()">📤 Kirim ke WhatsApp</button>
  </div>
</div>

<!-- Firebase SDK -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.2/firebase-app.js";
import { getStorage, ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/10.13.2/firebase-storage.js";

// 🔧 Ganti konfigurasi ini dengan milikmu sendiri dari Firebase Console
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
const app = initializeApp(firebaseConfig);
const storage = getStorage(app);

const sectionsData = {
  QA1: { title: "Validasi Checklist Service", items: ["Job Card","Form QA 1","Form PPM"] },
  QA7: { title: "Kelengkapan Checklist Service", items: ["Job Card","Form QA 7","Form Repair Order"] }
};

const inspectionSections = [
  { name: "Engine Area", items: ["Belt tension","Oil leakage"] },
  { name: "Cabin Area", items: ["Power window","FM Radio"] }
];

function renderSections(){
  let html="";
  inspectionSections.forEach(sec=>{
    html += `<div class="card"><h3>${sec.name}</h3>`;
    sec.items.forEach((item,idx)=>{
      const id = sec.name.replace(/\s+/g,'')+idx;
      html += `
      <div class="label">
        <span>${item}</span>
        <div>
          <button class="ok active" onclick="toggleButton(this,'OK','${id}')">OK</button>
          <button class="notok" onclick="toggleButton(this,'Not OK','${id}')">Not OK</button>
        </div>
      </div>
      <textarea id="note_${id}" placeholder="Catatan temuan..."></textarea>
      <input type="file" id="img_${id}" accept="image/*">
      `;
    });
    html += `</div>`;
  });
  document.getElementById("sections").innerHTML = html;
}
window.toggleButton=function(el,val,id){
  const parent = el.parentElement;
  parent.querySelectorAll("button").forEach(b=>b.classList.remove("active"));
  el.classList.add("active");
}
renderSections();

async function uploadImage(file, path){
  if(!file) return null;
  const imageRef = ref(storage, path);
  await uploadBytes(imageRef, file);
  return await getDownloadURL(imageRef);
}

window.sendToWhatsApp = async function(){
  const qaType = document.getElementById("qaType").value;
  const tgl = document.getElementById("tgl").value;
  const mekanik = document.getElementById("mekanik").value;
  const cn = document.getElementById("cn").value;
  const hm = document.getElementById("hm").value;
  
  let msg = `*${qaType === "QA1" ? "QA-1 Pre Inspection" : "QA-7 Final Inspection"}*\n\n`;
  msg += `📅 Tanggal: ${tgl}\n👷 Mekanik: ${mekanik}\n🚗 CN Unit: ${cn}\n⌛ HM: ${hm}\n\n`;

  // Upload foto unit
  const fotoUnit = document.getElementById("fotoUnit").files[0];
  if(fotoUnit){
    const url = await uploadImage(fotoUnit, `unit/${cn}_${Date.now()}`);
    if(url) msg += `📸 Foto Unit: ${url}\n\n`;
  }

  // Bagian inspeksi
  for(const sec of inspectionSections){
    msg += `🧩 *${sec.name}*\n`;
    for(const [idx, item] of sec.items.entries()){
      const id = sec.name.replace(/\s+/g,'')+idx;
      const note = document.getElementById(`note_${id}`).value.trim();
      const isNotOk = document.querySelector(`#img_${id}`).previousElementSibling.previousElementSibling.querySelector(".notok.active");
      msg += `${item}: ${isNotOk ? "❌ Not OK" : "✅ OK"}\n`;
      if(note) msg += `📝 ${note}\n`;
      const file = document.getElementById(`img_${id}`).files[0];
      if(file){
        const url = await uploadImage(file, `temuan/${cn}_${id}_${Date.now()}`);
        if(url) msg += `📷 ${url}\n`;
      }
    }
    msg += `\n`;
  }

  // Deviation
  const devText = document.getElementById("manualDeviation").value.trim();
  const devImg = document.getElementById("deviationImage").files[0];
  if(devText || devImg){
    msg += `⚠️ *Deviation:*\n${devText || "Tidak ada"}\n`;
    if(devImg){
      const url = await uploadImage(devImg, `deviation/${cn}_${Date.now()}`);
      if(url) msg += `📸 ${url}\n`;
    }
  }

  const waMsg = encodeURIComponent(msg);
  window.open(`https://wa.me/?text=${waMsg}`, "_blank");
}
</script>
</body>
</html>
