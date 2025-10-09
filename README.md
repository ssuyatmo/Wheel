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
  margin-top: 330px;
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
img.preview {
  display: block;
  width: 100%;
  max-height: 150px;
  object-fit: cover;
  margin-top: 5px;
  border-radius: 8px;
}
.progress-container {
  width: 100%;
  background: #ddd;
  border-radius: 6px;
  margin-top: 5px;
  display: none;
}
.progress-bar {
  height: 8px;
  width: 0%;
  background: #4caf50;
  border-radius: 6px;
  transition: width 0.3s;
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
#uploadStatus {
  text-align: center;
  font-weight: bold;
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
    <label>📞 Kirim ke No. WhatsApp (tanpa + atau 0):</label>
    <input type="number" id="waNumber" placeholder="62xxxxxx">
    <label>📷 Foto Umum Unit:</label><input type="file" id="fotoUnit" accept="image/*" onchange="previewImage(this,'previewUnit')">
    <img id="previewUnit" class="preview" style="display:none;">
  </div>
</div>

<!-- ISI -->
<div class="container" id="mainContent">
  <div id="sections"></div>
  <div class="card">
    <h3>⚠️ Deviation Tambahan</h3>
    <textarea id="manualDeviation" rows="3" placeholder="Tambahkan deviation manual..."></textarea>
    <label>📸 Foto Deviation:</label>
    <input type="file" id="deviationImage" accept="image/*" onchange="previewImage(this,'previewDeviation')">
    <img id="previewDeviation" class="preview" style="display:none;">
  </div>

  <div class="buttons">
    <button class="main" onclick="sendToWhatsApp()">📤 Send WhatsApp</button>
  </div>

  <div id="uploadStatus"></div>
  <div class="progress-container">
    <div class="progress-bar" id="progressBar"></div>
  </div>
</div>

<!-- Firebase SDK -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.2/firebase-app.js";
import { getStorage, ref, uploadBytesResumable, getDownloadURL } from "https://www.gstatic.com/firebasejs/10.13.2/firebase-storage.js";

// 🔧 Ganti konfigurasi Firebase kamu di bawah ini
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

// === Render bagian inspeksi ===
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
      <input type="file" id="img_${id}" accept="image/*" onchange="previewImage(this,'prev_${id}')">
      <img id="prev_${id}" class="preview" style="display:none;">
      `;
    });
    html += `</div>`;
  });
  document.getElementById("sections").innerHTML = html;
}
renderSections();

// === Tombol OK/Not OK ===
window.toggleButton=function(el,val,id){
  const parent = el.parentElement;
  parent.querySelectorAll("button").forEach(b=>b.classList.remove("active"));
  el.classList.add("active");
};

// === Preview gambar ===
window.previewImage=function(input, previewId){
  const img = document.getElementById(previewId);
  if (input.files && input.files[0]) {
    const reader = new FileReader();
    reader.onload = e => { img.src = e.target.result; img.style.display = "block"; };
    reader.readAsDataURL(input.files[0]);
  } else { img.style.display = "none"; }
};

// === Upload progress ke Firebase ===
async function uploadImage(file, path){
  return new Promise((resolve, reject)=>{
    if(!file) return resolve(null);
    const storageRef = ref(storage, path);
    const uploadTask = uploadBytesResumable(storageRef, file);
    document.querySelector(".progress-container").style.display = "block";
    uploadTask.on("state_changed",
      (snapshot)=>{
        const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
        document.getElementById("progressBar").style.width = progress + "%";
        document.getElementById("uploadStatus").textContent = `Mengunggah... ${Math.round(progress)}%`;
      },
      (error)=>{
        document.getElementById("uploadStatus").textContent = "❌ Upload gagal!";
        reject(error);
      },
      ()=>{
        getDownloadURL(uploadTask.snapshot.ref).then((url)=>{
          document.getElementById("uploadStatus").textContent = "✅ Upload selesai!";
          resolve(url);
        });
      }
    );
  });
}

// === Kirim ke WhatsApp ===
window.sendToWhatsApp = async function(){
  const qaType = document.getElementById("qaType").value;
  const tgl = document.getElementById("tgl").value;
  const mekanik = document.getElementById("mekanik").value;
  const cn = document.getElementById("cn").value;
  const hm = document.getElementById("hm").value;
  const waNumber = document.getElementById("waNumber").value.trim();

  if (!waNumber) { alert("Masukkan nomor WhatsApp tujuan!"); return; }

  let msg = `*${qaType === "QA1" ? "QA-1 Pre Inspection" : "QA-7 Final Inspection"}*\n\n`;
  msg += `📅 Tanggal: ${tgl}\n👷 Mekanik: ${mekanik}\n🚗 CN Unit: ${cn}\n⌛ HM: ${hm}\n\n`;

  const fotoUnit = document.getElementById("fotoUnit").files[0];
  if(fotoUnit){
    const url = await uploadImage(fotoUnit, `unit/${cn}_${Date.now()}`);
    if(url) msg += `📸 Foto Unit: ${url}\n\n`;
  }

  for(const sec of inspectionSections){
    msg += `🧩 *${sec.name}*\n`;
    for(const [idx, item] of sec.items.entries()){
      const id = sec.name.replace(/\s+/g,'')+idx;
      const note = document.getElementById(`note_${id}`).value.trim();
      const notOk = document.querySelector(`#img_${id}`).previousElementSibling.previousElementSibling.querySelector(".notok.active");
      msg += `${item}: ${notOk ? "❌ Not OK" : "✅ OK"}\n`;
      if(note) msg += `📝 ${note}\n`;
      const file = document.getElementById(`img_${id}`).files[0];
      if(file){
        const url = await uploadImage(file, `temuan/${cn}_${id}_${Date.now()}`);
        if(url) msg += `📷 ${url}\n`;
      }
    }
    msg += `\n`;
  }

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
  window.open(`https://wa.me/${waNumber}?text=${waMsg}`, "_blank");
}
</script>
</body>
</html>
