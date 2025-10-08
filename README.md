<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Camera OCR & WhatsApp</title>
<script src="https://cdn.jsdelivr.net/npm/tesseract.js@4.1.1/dist/tesseract.min.js"></script>
<style>
:root{
  --bg:#f9f9f9;
  --fg:#222;
  --card:#fff;
  --accent:#4caf50;
  --accent-dark:#1976d2;
  --accent-purple:#6a1b9a;
}
body{
  font-family: Arial, Helvetica, sans-serif;
  margin:0;
  padding:16px;
  background:var(--bg);
  color:var(--fg);
}
body.dark{
  --bg:#222;
  --fg:#f0f0f0;
  background:var(--bg);
  color:var(--fg);
}
header{display:flex;align-items:center;gap:12px;margin-bottom:8px}
h1{font-size:1.2rem;margin:0}
.card{background:var(--card);padding:12px;border-radius:10px;box-shadow:0 1px 4px rgba(0,0,0,0.06);margin-top:12px}
video, img{width:100%;border-radius:8px;max-height:50vh;object-fit:cover}
button{display:block;width:100%;padding:12px;border:0;border-radius:8px;background:var(--accent);color:#fff;font-weight:600;margin-top:8px;cursor:pointer;font-size:1em}
textarea{width:100%;height:160px;padding:8px;border-radius:8px;border:1px solid #ddd;resize:vertical}
.row{display:flex;gap:8px;margin-top:8px}
.row > *{flex:1}
#progressContainer{width:100%;background:#eee;border-radius:8px;overflow:hidden;margin-top:8px;height:18px;display:none}
#progressBar{width:0%;height:100%;background:var(--accent);color:#fff;text-align:center;line-height:18px;font-size:12px}
footer{font-size:12px;color:#666;margin-top:12px;text-align:center}
@media(min-width:720px){
  main{max-width:720px;margin:0 auto}
}
select, input[type=number]{padding:8px;border-radius:8px;border:1px solid #ddd}
</style>
</head>
<body>

<main>
<header>
<h1>Camera OCR & WhatsApp</h1>
</header>

<section class="card">
  <video id="video" autoplay playsinline></video>
  <div class="row">
    <button id="capture">📸 Ambil Foto & OCR</button>
    <button id="switchCam" style="background:var(--accent-dark)">🔁 Ganti Kamera</button>
  </div>

  <img id="photoPreview" alt="Preview Foto" style="display:none"/>

  <div id="progressContainer">
    <div id="progressBar">0%</div>
  </div>

  <h3>Hasil Teks</h3>
  <textarea id="ocrResult" placeholder="Hasil OCR akan muncul disini..."></textarea>

  <div class="row">
    <select id="ocrLang" title="Pilih bahasa OCR">
      <option value="ind">Bahasa Indonesia (ind)</option>
      <option value="eng" selected>English (eng)</option>
      <option value="jpn">Japanese (jpn)</option>
    </select>
    <input id="fontSize" type="number" min="10" max="36" value="16"/>
  </div>

  <div class="row">
    <button id="downloadPhoto" style="background:var(--accent-purple)">⬇️ Download Foto</button>
    <button id="sendWA" style="background:#25D366">💬 Kirim ke WhatsApp</button>
  </div>

  <div class="row">
    <select id="theme">
      <option value="light">Tema Terang</option>
      <option value="dark">Tema Gelap</option>
    </select>
  </div>
</section>

<footer class="card">
<strong>Petunjuk:</strong>
<ol>
  <li>Izinkan akses kamera di browser.</li>
  <li>Pilih bahasa OCR & tekan "Ambil Foto & OCR".</li>
  <li>Setelah selesai, teks dapat diedit & dikirim ke WhatsApp.</li>
</ol>
<div>Built with Tesseract.js — semua proses OCR dilakukan di browser (offline).</div>
</footer>
</main>

<script>
let video = document.getElementById('video');
let canvas = document.createElement('canvas');
let captureBtn = document.getElementById('capture');
let downloadBtn = document.getElementById('downloadPhoto');
let photoPreview = document.getElementById('photoPreview');
let ocrResult = document.getElementById('ocrResult');
let ocrLang = document.getElementById('ocrLang');
let sendWA = document.getElementById('sendWA');
let progressContainer = document.getElementById('progressContainer');
let progressBar = document.getElementById('progressBar');
let switchCamBtn = document.getElementById('switchCam');
let fontSizeInput = document.getElementById('fontSize');
let themeSelect = document.getElementById('theme');

let stream = null;
let useFront = false;

async function startCamera(){
  if(stream) stream.getTracks().forEach(t=>t.stop());
  try{
    stream = await navigator.mediaDevices.getUserMedia({video:{facingMode:useFront?'user':'environment'}, audio:false});
    video.srcObject = stream;
  }catch(err){
    alert("Gagal mengakses kamera: "+err.message);
  }
}
startCamera();

switchCamBtn.addEventListener('click', async ()=>{
  useFront = !useFront;
  await startCamera();
});

captureBtn.addEventListener('click', async ()=>{
  if(!video.videoWidth){ alert('Kamera belum siap.'); return;}
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  let ctx = canvas.getContext('2d');
  ctx.drawImage(video,0,0,canvas.width,canvas.height);
  photoPreview.src = canvas.toDataURL('image/png');
  photoPreview.style.display='block';

  progressContainer.style.display='block';
  progressBar.style.width='0%';
  progressBar.innerText='0%';
  ocrResult.value='Sedang memproses OCR...';

  try{
    const lang = ocrLang.value || 'eng';
    const worker = Tesseract.createWorker({
      logger:m=>{
        if(m.status==='recognizing text'||m.status==='loading tesseract core'){
          let p=Math.round(m.progress*100);
          progressBar.style.width=p+'%';
          progressBar.innerText=p+'%';
        }
      }
    });
    await worker.load();
    await worker.loadLanguage(lang);
    await worker.initialize(lang);
    const {data:{text}} = await worker.recognize(canvas);
    ocrResult.value=text;
    ocrResult.style.fontSize=fontSizeInput.value+'px';
    await worker.terminate();
    progressBar.style.width='100%';
    progressBar.innerText='Selesai';
  }catch(e){
    console.error(e);
    alert('OCR Error: '+e.message);
    progressBar.style.width='0%';
    progressBar.innerText='Error';
  }
});

downloadBtn.addEventListener('click', ()=>{
  if(!canvas.width){ alert('Belum ada foto untuk didownload.'); return;}
  let link = document.createElement('a');
  link.href=canvas.toDataURL('image/png');
  link.download='foto_ocr.png';
  link.click();
});

sendWA.addEventListener('click', ()=>{
  let text = encodeURIComponent(ocrResult.value||'');
  window.open(`https://wa.me/?text=${text}`,'_blank');
});

fontSizeInput.addEventListener('change', ()=>{
  ocrResult.style.fontSize=fontSizeInput.value+'px';
});

themeSelect.addEventListener('change', ()=>{
  document.body.className = themeSelect.value==='dark'?'dark':'';
});
</script>

</body>
</html>
