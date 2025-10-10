<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Inspection HD785-7</title>
<style>
/* === GAYA SAMA SEPERTI SEBELUMNYA === */
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
  margin-top: 300px;
}
.card {
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px;
  margin-bottom: 10px;
}
h3 { background: #cfe8cf; padding: 6px; border-radius: 6px; font-size: 15px; }
.label { display: flex; justify-content: space-between; align-items: center; margin: 4px 0; }
.label span { flex: 1; font-size: 14px; }
.label button {
  margin-left: 5px; padding: 4px 10px;
  border: none; border-radius: 6px; cursor: pointer; font-weight: bold;
}
.ok { background: #a4f5a4; }
.ok.active { background: #4caf50; color:white; }
.notok { background: #f5a4a4; }
.notok.active { background: #f44336; color:white; }
textarea, input, select {
  width: 100%; border-radius: 6px; border: 1px solid #ccc;
  resize: none; margin-top: 4px; padding: 5px; font-size: 13px;
}
.buttons { display: flex; justify-content: space-around; margin-top: 10px; }
button.main {
  padding: 8px 14px; border: none; border-radius: 8px; font-weight: bold;
  cursor: pointer; background: #f1d28a; box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}
#previewPopup {
  display: none; position: fixed; top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  background: white; border-radius: 10px; box-shadow: 0 0 20px rgba(0,0,0,0.3);
  width: 90%; max-width: 400px; z-index: 200; padding: 15px;
}
#previewPopup pre {
  max-height: 60vh; overflow-y: auto; white-space: pre-wrap;
  font-size: 13px; margin-bottom: 10px;
}
#popupButtons { display: flex; justify-content: space-around; }
img#photoPreview {
  display: block; margin: 8px auto; max-width: 100%; border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
}
</style>
</head>
<body>

<header>🌿 QA Inspection Komatsu HD785-7</header>

<div class="fixed-header">
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

    <!-- 🔹 Tambahan upload foto unit -->
    <label>📸 Foto Unit:</label>
    <input type="file" id="photo" accept="image/*" onchange="previewPhoto(event)">
    <img id="photoPreview" alt="Preview Foto Unit">
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
  <pre id="previewText"></pre>
  <img id="previewImage" style="max-width:100%;border-radius:10px;display:none;">
  <div id="popupButtons">
    <button onclick="copyText()">📋 Copy</button>
    <button onclick="sendToWhatsApp()">📤 Send WA</button>
    <button onclick="closePopup()">❌ Close</button>
  </div>
</div>

<script>
let photoBase64 = "";

function previewPhoto(event) {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = function(e) {
      photoBase64 = e.target.result;
      document.getElementById('photoPreview').src = photoBase64;
    };
    reader.readAsDataURL(file);
  }
}

/* === RENDER SECTIONS & LOGIKA === */
const sectionsData = {
  QA1:{title:"Validasi & Kelengkapan Checklist Service",items:[
    "Job Card & Cover Checklist","Form Observasi Redo PS","Form QA 1 & QA 7","Form PPM",
    "Form Check List A","Form Check List B","Form Check List C","Form Backlog",
    "Form FUI","Form Repair Order","Form Combine Maintenance","Form Service Activity Report"
  ]},
  QA7:{title:"Kelengkapan Pengisian Checklist Service",items:[
    "Job Card & Cover Checklist","Form Observasi Redo PS","Form QA 1 & QA 7","Form PPM",
    "Form Check List A","Form Check List B","Form Check List C","Form Backlog",
    "Form FUI","Form Repair Order","Form Combine Maintenance","Form Service Activity Report"
  ]}
};

const inspectionSections = [
  { name:"Oil Level", items:["Engine oil level","Transmission oil level","Hydraulic oil level"] },
  { name:"Engine Area", items:[
    "Belt tension","Engine oil leakage","Common Rail Connector","Injector Tube",
    {label:"Common rail pressure (ON)", type:"number", unit:"MPa"},
    {label:"Power Supply (ON)", type:"number", unit:"V"}
  ]},
  { name:"Cabin Area", items:["FM Radio","Fatigue Warning","Power Window"] },
  { name:"Frame Area", items:["Operator seat","Hand Rail"] },
  { name:"Suspension Pressure (on monitor panel)", items:[
    {label:"FL",type:"number",unit:"MPa"},
    {label:"FR",type:"number",unit:"MPa"},
    {label:"RL",type:"number",unit:"MPa"},
    {label:"RR",type:"number",unit:"MPa"}
  ]},
  { name:"Tyre Condition", items:["Tyre Condition"] }
];

function renderSections(){
  const qaType=document.getElementById('qaType').value;
  let html="";
  inspectionSections.forEach(sec=>{
    html+=`<div class="card"><h3>${sec.name}</h3>`;
    sec.items.forEach((item,idx)=>{
      const id=sec.name.replace(/\s+/g,'')+idx;
      if(typeof item==='object'&&item.type==='number'){
        html+=`<label>${item.label}:<input type="number" id="num_${id}" step="0.1" placeholder="Masukkan nilai (${item.unit})..."></label>`;
      } else {
        html+=`
        <div class="label">
          <span>${item}</span>
          <div>
            <button class="ok active" onclick="toggleButton(this,'OK','${id}')">OK</button>
            <button class="notok" onclick="toggleButton(this,'Not OK','${id}')">Not OK</button>
          </div>
        </div>
        <textarea id="note_${id}" placeholder="Tulis temuan jika ada..."></textarea>`;
      }
    });
    html+=`</div>`;
  });

  const validasi=sectionsData[qaType];
  html+=`<div class="card"><h3>${validasi.title}</h3>`;
  validasi.items.forEach((v,i)=>{
    const id="val"+i;
    html+=`
      <div class="label">
        <span>${v}</span>
        <div>
          <button class="ok active" onclick="toggleButton(this,'OK','${id}')">OK</button>
          <button class="notok" onclick="toggleButton(this,'Not OK','${id}')">Not OK</button>
        </div>
      </div>`;
  });
  html+=`</div><div class="card"><h3>⚠️ Deviation</h3><textarea id="manualDeviation" rows="3" placeholder="Tambahkan deviation manual..."></textarea></div>`;
  document.getElementById('sections').innerHTML=html;
  updateCardColors();
}
renderSections();
document.getElementById('qaType').addEventListener('change', renderSections);

function toggleButton(el,val,id){
  const parent=el.parentElement;
  parent.querySelectorAll('button').forEach(b=>b.classList.remove('active'));
  el.classList.add('active');
  el.dataset.value=val;
  updateCardColors();
}
function updateCardColors(){
  document.querySelectorAll('.card').forEach(c=>{
    const notOk=c.querySelector('.notok.active');
    c.style.background=notOk?'#f8d7da':'#fff';
  });
}

/* === FUNGSI PREVIEW & WHATSAPP === */
function generateText(){
  const qaType=document.getElementById('qaType').value;
  let text=`*${qaType==='QA1'?'QA-1 Pre Inspection':'QA-7 Final Inspection'}*\n\n`;
  text+=`📅 Tanggal : ${tgl.value}\n👷 Mekanik : ${mekanik.value}\n🚗 CN : ${cn.value}\n⌛ HM : ${hm.value}\n\n`;

  let deviations=[];
  inspectionSections.forEach(sec=>{
    text+=`🧩 *${sec.name}*\n`;
    sec.items.forEach((item,idx)=>{
      const id=sec.name.replace(/\s+/g,'')+idx;
      if(typeof item==='object'){
        const val=document.getElementById(`num_${id}`).value;
        text+=`${item.label} : ${val?val+' '+item.unit:'-'}\n`;
      } else {
        const ok=document.querySelector(`#note_${id}`).previousElementSibling.querySelector('.ok.active');
        const notok=document.querySelector(`#note_${id}`).previousElementSibling.querySelector('.notok.active');
        const status=ok?'✅ OK':notok?'❌ Not OK':'❔';
        text+=`${item} : ${status}\n`;
        const note=document.getElementById(`note_${id}`).value.trim();
        if(note) deviations.push(`⚠️ ${note}`);
      }
    });
    text+=`\n`;
  });

  const qaSection=sectionsData[qaType];
  text+=`📝 *${qaSection.title}*\n`;
  qaSection.items.forEach((v,i)=>{
    const id="val"+i;
    const ok=document.querySelector(`[onclick*="${id}"].ok.active`);
    const notok=document.querySelector(`[onclick*="${id}"].notok.active`);
    text+=`${v} : ${ok?'✅ OK':notok?'❌ Not OK':'❔'}\n`;
  });

  const manual=document.getElementById('manualDeviation').value.trim();
  if(manual) manual.split('\n').forEach(d=>deviations.push(`⚠️ ${d.trim()}`));

  text+=`\n⚠️ *Deviation:*\n${deviations.length?deviations.join('\n'):'Tidak ada'}\n`;
  
  if(photoBase64){
    text+=`\n📸 *Foto Unit:* (terlampir di bawah)\n`;
  }

  return text;
}

function showPreview(){
  document.getElementById('previewText').textContent=generateText();
  const img=document.getElementById('previewImage');
  if(photoBase64){
    img.src=photoBase64;
    img.style.display='block';
  } else img.style.display='none';
  document.getElementById('previewPopup').style.display='block';
}

function sendToWhatsApp(){
  const msg=encodeURIComponent(generateText());
  if(photoBase64){
    // WhatsApp tidak mendukung pengiriman langsung gambar base64, tampilkan link data URI saja
    const photoLink=encodeURIComponent(photoBase64);
    window.open(`https://wa.me/?text=${msg}%0A${photoLink}`,'_blank');
  } else {
    window.open(`https://wa.me/?text=${msg}`,'_blank');
  }
}

function copyText(){
  navigator.clipboard.writeText(generateText());
  alert("✅ Teks disalin!");
}

function closePopup(){
  document.getElementById('previewPopup').style.display='none';
}
</script>
</body>
</html>
