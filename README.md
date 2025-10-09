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
  text-align: center; padding: 10px 0; font-weight: bold;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2); z-index: 100;
}
.fixed-header {
  position: fixed; top: 60px; left: 0; width: 100%;
  background: #d8e9a8; z-index: 99; padding: 8px 10px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.15);
}
.fixed-header .card {
  background: #fff; border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px; margin-bottom: 8px;
}
.container { padding: 10px; max-width: 600px; margin: auto; margin-top: 310px; }
.card {
  background: #fff; border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px; margin-bottom: 10px;
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
textarea, input[type="text"], input[type="number"], input[type="date"], select {
  width: 100%; border-radius: 6px; border: 1px solid #ccc;
  margin-top: 4px; padding: 5px; font-size: 13px;
}
img.preview {
  width: 100px; height: 80px; object-fit: cover;
  border-radius: 8px; margin-top: 6px; display: none;
}
.buttons { display: flex; justify-content: space-around; margin-top: 10px; }
button.main {
  padding: 8px 14px; border: none; border-radius: 8px;
  font-weight: bold; cursor: pointer;
  background: #f1d28a; box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}
#previewPopup {
  display: none; position: fixed; top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  background: white; border-radius: 10px;
  box-shadow: 0 0 20px rgba(0,0,0,0.3);
  width: 90%; max-width: 400px; z-index: 200; padding: 15px;
}
#previewPopup pre {
  max-height: 60vh; overflow-y: auto; white-space: pre-wrap;
  font-size: 13px; margin-bottom: 10px;
}
#popupButtons { display: flex; justify-content: space-around; }
.previewImages {
  display: flex; flex-wrap: wrap; gap: 6px; justify-content: center;
}
.previewImages img {
  width: 90px; height: 80px; border-radius: 8px; object-fit: cover;
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
  </div>
  <div class="card">
    <label><input type="checkbox" id="allowImage" checked> ✅ Izinkan unggah foto/gambar</label>
  </div>
</div>

<div class="container" id="mainContent">
  <div id="sections"></div>

  <div class="card" id="deviationSection">
    <h3>⚠️ Deviation (Tambahkan jika ada)</h3>
    <div id="deviationList"></div>
    <button class="main" onclick="addDeviation()">➕ Tambah Deviation</button>
  </div>

  <div class="buttons">
    <button class="main" onclick="showPreview()">🧾 Preview</button>
    <button class="main" onclick="sendToWhatsApp()">📤 Kirim ke WhatsApp</button>
  </div>
</div>

<!-- POPUP PREVIEW -->
<div id="previewPopup">
  <pre id="previewText"></pre>
  <div class="previewImages" id="allPreviewImages"></div>
  <div id="popupButtons">
    <button onclick="copyText()">📋 Copy</button>
    <button onclick="closePopup()">❌ Close</button>
  </div>
</div>

<script>
const inspectionSections = [
  { name: "Oil Level", items: ["Engine oil level","Transmission oil level","Hydraulic oil level"] },
  { name: "Engine Area", items: ["Belt tension","Engine oil leakage","Common Rail Connector","Injector Tube"] },
  { name: "Cabin Area", items: ["FM Radio","Fatigue Warning","Power Window"] },
  { name: "Frame Area", items: ["Operator seat","Hand Rail"] },
  { name: "Tyre Condition", items: ["Tyre Condition"] }
];

let deviationCount = 0;

function renderSections(){
  let html = "";
  inspectionSections.forEach(sec=>{
    html += `<div class="card"><h3>${sec.name}</h3>`;
    sec.items.forEach((item, idx)=>{
      const id = sec.name.replace(/\s+/g,'')+idx;
      html += `
        <div class="label">
          <span>${item}</span>
          <div>
            <button class="ok active" onclick="toggleButton(this,'OK','${id}')">OK</button>
            <button class="notok" onclick="toggleButton(this,'Not OK','${id}')">Not OK</button>
          </div>
        </div>
        <textarea id="note_${id}" placeholder="Tulis temuan jika ada..."></textarea>
        <input type="file" accept="image/*" id="img_${id}" style="display:none" onchange="previewImage('${id}', event)">
        <img id="preview_${id}" class="preview">
      `;
    });
    html += `</div>`;
  });
  document.getElementById('sections').innerHTML = html;
}
renderSections();

function toggleButton(el,val,id){
  const parent = el.parentElement;
  parent.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
  el.classList.add('active');
  const allowImg = document.getElementById('allowImage').checked;
  const imgInput = document.getElementById(`img_${id}`);
  const previewImg = document.getElementById(`preview_${id}`);
  if(val === 'Not OK' && allowImg){
    imgInput.style.display = 'block';
  } else {
    imgInput.style.display = 'none';
    previewImg.style.display = 'none';
  }
}
function previewImage(id, event){
  const file = event.target.files[0];
  if(file){
    const reader = new FileReader();
    reader.onload = e=>{
      const img = document.getElementById(`preview_${id}`);
      img.src = e.target.result;
      img.style.display = 'block';
    };
    reader.readAsDataURL(file);
  }
}

/* === Tambah Deviation === */
function addDeviation(){
  deviationCount++;
  const id = "dev"+deviationCount;
  const div = document.createElement('div');
  div.className = "card";
  const allowImg = document.getElementById('allowImage').checked;
  div.innerHTML = `
    <label>Deviation ${deviationCount}</label>
    <textarea id="note_${id}" placeholder="Tuliskan deviation..."></textarea>
    ${allowImg ? `<input type="file" accept="image/*" id="img_${id}" onchange="previewImage('${id}', event)">` : ''}
    <img id="preview_${id}" class="preview">
  `;
  document.getElementById('deviationList').appendChild(div);
}

/* === Preview dan WA === */
function showPreview(){
  const popup = document.getElementById('previewPopup');
  document.getElementById('previewText').textContent = generateText();
  const allImages = document.querySelectorAll('img.preview');
  const imgContainer = document.getElementById('allPreviewImages');
  imgContainer.innerHTML = "";
  allImages.forEach(img=>{
    if(img.src && img.style.display==="block"){
      const imgClone = document.createElement('img');
      imgClone.src = img.src;
      imgContainer.appendChild(imgClone);
    }
  });
  popup.style.display = 'block';
}

function generateText(){
  let text=`*QA Inspection HD785-7*\n\n📅 ${tgl.value}\n👷 ${mekanik.value}\n🚗 CN: ${cn.value}\n⌛ HM: ${hm.value}\n\n`;
  inspectionSections.forEach(sec=>{
    text+=`🧩 *${sec.name}*\n`;
    sec.items.forEach((item,idx)=>{
      const id=sec.name.replace(/\s+/g,'')+idx;
      const ok=document.querySelector(`[onclick*="${id}"].ok.active`);
      const notok=document.querySelector(`[onclick*="${id}"].notok.active`);
      const status=ok?'✅ OK':notok?'❌ Not OK':'❔';
      const note=document.getElementById(`note_${id}`).value.trim();
      text+=`${item} : ${status}${note?` (${note})`:''}\n`;
    });
    text+=`\n`;
  });
  text += `⚠️ *Deviation Manual:*\n`;
  for(let i=1;i<=deviationCount;i++){
    const id="dev"+i;
    const note = document.getElementById(`note_${id}`);
    if(note && note.value.trim()!==""){
      text += `• ${note.value.trim()}\n`;
    }
  }
  return text;
}

function sendToWhatsApp(){
  const msg = encodeURIComponent(generateText());
  window.open(`https://wa.me/?text=${msg}`, '_blank');
}
function copyText(){
  navigator.clipboard.writeText(document.getElementById('previewText').textContent);
  alert('✅ Teks disalin!');
}
function closePopup(){ document.getElementById('previewPopup').style.display='none'; }
</script>
</body>
</html>
