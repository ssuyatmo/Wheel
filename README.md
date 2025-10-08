<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Foto ke Teks Mobile & WA</title>
<script src="https://cdn.jsdelivr.net/npm/tesseract.js@4.1.1/dist/tesseract.min.js"></script>
<style>
  body { font-family: Arial, sans-serif; margin: 10px; background-color: #f9f9f9; color: #333; }
  body.dark { background-color: #222; color: #f0f0f0; }
  h2 { font-size: 1.5em; }
  video, canvas, img { border: 1px solid #ccc; max-width: 100%; margin-top: 10px; border-radius: 8px; }
  #settings { margin-top: 15px; padding: 10px; border: 1px solid #ccc; border-radius: 8px; }
  button { margin-top: 10px; padding: 10px; width: 100%; font-size: 1em; border-radius: 5px; cursor: pointer; }
  textarea { width: 100%; margin-top: 10px; padding: 8px; border-radius: 5px; resize: vertical; }
  label { display: block; margin-top: 10px; }
  #progressContainer { width: 100%; background-color: #ddd; border-radius: 5px; margin-top: 10px; }
  #progressBar { width: 0%; height: 20px; background-color: #4caf50; border-radius: 5px; text-align: center; color: white; }
</style>
</head>
<body>

<h2>Aplikasi Foto ke Teks Mobile</h2>

<video id="video" autoplay playsinline></video>
<br>
<button id="capture">Ambil Foto</button>
<button id="downloadPhoto">Download Foto</button>
<img id="photoPreview" style="display:none;" alt="Preview Foto">

<h3>Progres OCR</h3>
<div id="progressContainer">
  <div id="progressBar">0%</div>
</div>

<h3>Hasil Teks</h3>
<textarea id="ocrResult" rows="6"></textarea>

<div id="settings">
  <h4>Pengaturan</h4>
  <label>
    Bahasa OCR:
    <select id="ocrLang">
      <option value="ind">Bahasa Indonesia</option>
      <option value="eng">English</option>
      <option value="jpn">Japanese</option>
    </select>
  </label>
  
  <label>
    Tema Aplikasi:
    <select id="theme">
      <option value="light">Terang</option>
      <option value="dark">Gelap</option>
    </select>
  </label>
  
  <label>
    Ukuran Font Hasil Teks:
    <input type="number" id="fontSize" value="16" min="10" max="36"> px
  </label>
</div>

<button id="sendWA">Kirim ke WhatsApp</button>

<script>
const video = document.getElementById('video');
const canvas = document.createElement('canvas');
const captureBtn = document.getElementById('capture');
const downloadBtn = document.getElementById('downloadPhoto');
const photoPreview = document.getElementById('photoPreview');
const ocrResult = document.getElementById('ocrResult');
const ocrLang = document.getElementById('ocrLang');
const sendWA = document.getElementById('sendWA');
const themeSelect = document.getElementById('theme');
const fontSizeInput = document.getElementById('fontSize');
const progressBar = document.getElementById('progressBar');

// Akses kamera (mobile-friendly)
navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } })
.then(stream => video.srcObject = stream)
.catch(err => alert("Tidak bisa mengakses kamera: " + err));

// Ambil foto dan OCR
captureBtn.addEventListener('click', () => {
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
  
  // Preview foto
  photoPreview.src = canvas.toDataURL('image/png');
  photoPreview.style.display = 'block';
  
  // Reset progress & teks
  progressBar.style.width = '0%';
  progressBar.innerText = '0%';
  ocrResult.value = "Sedang memproses OCR...";
  
  // OCR dengan progres
  Tesseract.recognize(
    canvas,
    ocrLang.value,
    {
      logger: m => {
        if(m.status === 'recognizing text') {
          let progress = Math.floor(m.progress * 100);
          progressBar.style.width = progress + '%';
          progressBar.innerText = progress + '%';
        }
      }
    }
  ).then(({ data: { text } }) => {
    ocrResult.value = text;
    ocrResult.style.fontSize = fontSizeInput.value + 'px';
    progressBar.style.width = '100%';
    progressBar.innerText = 'Selesai';
  });
});

// Download foto
downloadBtn.addEventListener('click', () => {
  const link = document.createElement('a');
  link.href = canvas.toDataURL('image/png');
  link.download = 'foto_ocr.png';
  link.click();
});

// Kirim ke WhatsApp
sendWA.addEventListener('click', () => {
  const text = encodeURIComponent(ocrResult.value);
  window.open(`https://wa.me/?text=${text}`, '_blank');
});

// Tema aplikasi
themeSelect.addEventListener('change', () => {
  document.body.className = themeSelect.value === 'dark' ? 'dark' : '';
});

// Update ukuran font teks
fontSizeInput.addEventListener('change', () => {
  ocrResult.style.fontSize = fontSizeInput.value + 'px';
});
</script>

</body>
</html>
