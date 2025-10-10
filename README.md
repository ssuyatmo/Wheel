<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>QA Inspection HD785-7</title>
<style>
/* === STYLE SAMA, HANYA TAMBAH SEDIKIT DI IMG === */
body {
  font-family: "Poppins", sans-serif;
  margin: 0;
  background: linear-gradient(to bottom right, #d8e9a8, #9cdba6);
  color: #222;
}
header {
  position: fixed; top: 0; left: 0; width: 100%;
  background: linear-gradient(to right, #f1d28a, #f5e4a2);
  color: #1a1a1a; text-align: center; padding: 10px 0;
  font-weight: bold; font-size: 18px; box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  z-index: 100;
}
.fixed-header { position: fixed; top: 60px; width: 100%; background: #d8e9a8; z-index: 99; padding: 8px 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.15); }
.card { background:#fff; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,0.2); padding:10px; margin-bottom:8px; }
.container { padding:10px; max-width:600px; margin:auto; margin-top:300px; }
img#photoPreview { display:block; margin:8px auto; max-width:90%; border-radius:8px; box-shadow:0 2px 6px rgba(0,0,0,0.3); }
#previewPopup { display:none; position:fixed; top:50%; left:50%; transform:translate(-50%,-50%); background:white; border-radius:10px; box-shadow:0 0 20px rgba(0,0,0,0.3); width:90%; max-width:400px; z-index:200; padding:15px; }
#previewPopup pre { max-height:60vh; overflow-y:auto; white-space:pre-wrap; font-size:13px; margin-bottom:10px; }
#previewImage { max-width:100%; border-radius:10px; display:none; }
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

    <!-- 🔹 Upload Foto Unit -->
    <label>📸 Foto Unit:</label>
    <input type="file" id="photo" accept="image/*" onchange="compressAndPreview(event)">
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
  <img id="previewImage">
  <div id="popupButtons">
    <button onclick="copyText()">📋 Copy</button>
    <button onclick="sendToWhatsApp()">📤 Send WA</button>
    <button onclick="closePopup()">❌ Close</button>
  </div>
</div>

<script>
let photoBase64 = "";

/* === 🔧 Kompres & Tampilkan Foto === */
function compressAndPreview(event) {
  const file = event.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = function(e) {
    const img = new Image();
    img.src = e.target.result;
    img.onload = function() {
      const canvas = document.createElement("canvas");
      const maxWidth = 600; // lebar maksimum
      const scale = Math.min(1, maxWidth / img.width);
      const width = img.width * scale;
      const height = img.height * scale;
      canvas.width = width;
      canvas.height = height;
      const ctx = canvas.getContext("2d");
      ctx.drawImage(img, 0, 0, width, height);
      // kualitas 0.7 untuk kompres
      photoBase64 = canvas.toDataURL("image/jpeg", 0.7);
      document.getElementById("photoPreview").src = photoBase64;
    };
  };
  reader.readAsDataURL(file);
}

/* === Bagian logika QA tetap sama, hanya potongan penting di bawah === */
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
    {label:"Common rail pressure (ON)",type:"number",unit:"MPa"},
    {label:"Power Supply (ON)",type:"number",unit:"V"}
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
      if(typeof item==='object'){
        html+=`<label>${item.label}:<input type="number" id="num_${id}" step="0.1" placeholder="(${item.unit})"></label>`;
      } else {
        html+=`<div class="label">
          <span>${item}</span>
          <div>
            <button class="ok active" onclick="toggleButton(this,'OK','${id}')">OK</button>
            <button class="notok" onclick="toggleButton(this,'Not OK','${id}')">Not OK</button>
          </div>
        </div>
        <textarea id="note_${id}" placeholder="Tulis temuan..."></textarea>`;
      }
    });
    html+="</div>";
  });
  const valid=sectionsData[qaType];
  html+=`<div class="card"><h3>${valid.title}</h3>`;
  valid.items.forEach((v,i)=>{
    const id="val"+i;
    html+=`<div class="label"><span>${v}</span><div><button class="ok active" onclick="toggleButton(this,'OK','${id}')">OK</button><button class="notok" onclick="toggleButton(this,'Not OK','${id}')">Not OK</button></div></div>`;
  });
  html+=`</div><div class="card"><h3>⚠️ Deviation</h3><textarea id="manualDeviation" rows="3"></textarea></div>`;
  document.getElementById('sections').innerHTML=html;
  updateCardColors();
}
renderSections();
document.getElementById('qaType').addEventListener('change',renderSections);

function toggleButton(el,val,id){
  const p=el.parentElement;
  p.querySelectorAll('button').forEach(b=>b.classList.remove('active'));
  el.classList.add('active');
  updateCardColors();
}
function updateCardColors(){
  document.querySelectorAll('.card').forEach(c=>{
    c.style.background=c.querySelector('.notok.active')?'#f8d7da':'#fff';
  });
}

/* === Kirim & Preview === */
function generateText(){
  const qaType=document.getElementById('qaType').value;
  let txt=`*${qaType==='QA1'?'QA-1 Pre Inspection':'QA-7 Final Inspection'}*\n\n`;
  txt+=`📅 Tanggal: ${tgl.value}\n👷 Mekanik: ${mekanik.value}\n🚗 CN: ${cn.value}\n⌛ HM: ${hm.value}\n\n`;

  let devs=[];
  inspectionSections.forEach(sec=>{
    txt+=`🧩 *${sec.name}*\n`;
    sec.items.forEach((it,idx)=>{
      const id=sec.name.replace(/\s+/g,'')+idx;
      if(typeof it==='object'){
        const val=document.getElementById(`num_${id}`).value;
        txt+=`${it.label}: ${val?val+' '+it.unit:'-'}\n`;
      } else {
        const ok=document.querySelector(`#note_${id}`).previousElementSibling.querySelector('.ok.active');
        const notok=document.querySelector(`#note_${id}`).previousElementSibling.querySelector('.notok.active');
        const st=ok?'✅ OK':notok?'❌ Not OK':'❔';
        txt+=`${it}: ${st}\n`;
        const note=document.getElementById(`note_${id}`).value.trim();
        if(note) devs.push(`⚠️ ${note}`);
      }
    });
    txt+="\n";
  });
  const valid=sectionsData[qaType];
  txt+=`📝 *${valid.title}*\n`;
  valid.items.forEach((v,i)=>{
    const id="val"+i;
    const ok=document.querySelector(`[onclick*="${id}"].ok.active`);
    const notok=document.querySelector(`[onclick*="${id}"].notok.active`);
    txt+=`${v}: ${ok?'✅ OK':notok?'❌ Not OK':'❔'}\n`;
  });
  const manual=document.getElementById('manualDeviation').value.trim();
  if(manual) manual.split('\n').forEach(d=>devs.push(`⚠️ ${d.trim()}`));
  txt+=`\n⚠️ *Deviation:*\n${devs.length?devs.join('\n'):'Tidak ada'}\n`;
  if(photoBase64) txt+=`\n📸 *Foto Unit:* (terlampir di bawah)\n`;
  return txt;
}

function showPreview(){
  document.getElementById('previewText').textContent=generateText();
  const img=document.getElementById('previewImage');
  if(photoBase64){img.src=photoBase64; img.style.display='block';} else img.style.display='none';
  document.getElementById('previewPopup').style.display='block';
}
function sendToWhatsApp(){
  const msg=encodeURIComponent(generateText());
  if(photoBase64){
    const link=encodeURIComponent(photoBase64);
    window.open(`https://wa.me/?text=${msg}%0A${link}`,'_blank');
  } else window.open(`https://wa.me/?text=${msg}`,'_blank');
}
function copyText(){
  navigator.clipboard.writeText(generateText());
  alert('✅ Disalin!');
}
function closePopup(){document.getElementById('previewPopup').style.display='none';}
</script>
</body>
</html>
