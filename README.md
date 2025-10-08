<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Foto ke Teks & Kirim WA</title>
<script src="https://cdn.jsdelivr.net/npm/tesseract.js@4.1.1/dist/tesseract.min.js"></script>
<style>
  body { font-family: Arial, sans-serif; margin: 20px; }
  video, canvas { border: 1px solid #ccc; max-width: 100%; }
  #settings { margin-top: 20px; padding: 10px; border: 1px solid #ccc; }
  button { margin-top: 10px; padding: 10px; }
</style>
</head>
<body>

<h2>Aplikasi Foto ke Teks & Kirim WA</h2>

<!-- Kamera -->
<video id="video" autoplay></video>
<br>
<button id="capture">Ambil Foto</button>

<!-- Canvas hasil foto -->
<canvas id="canvas" style="display:none;"></canvas>

<!-- Hasil OCR -->
<h3>Hasil Teks</h3>
<textarea id="ocrResult" rows="6" style="width:100%;"></textarea>

<!-- Menu pengaturan sederhana -->
<div id="settings">
  <h4>Pengaturan</h4>
  <label>
    Bahasa OCR:
    <select id="ocrLang">
      <option value="ind">Bahasa Indonesia</option>
      <option value="eng">English</option>
    </select>
  </label>
</div>

<!-- Tombol kirim WA -->
<button id="sendWA">Kirim ke WhatsApp</button>

<script>
const video = document.getElementById('video');
const canvas = document.getElementById('canvas');
const captureBtn = document.getElementById('capture');
const ocrResult = document.getElementById('ocrResult');
const ocrLang = document.getElementById('ocrLang');
const sendWA = document.getElementById('sendWA');

// Akses kamera
navigator.mediaDevices.getUserMedia({ video: true })
.then(stream => {
  video.srcObject = stream;
})
.catch(err => {
  alert("Tidak bisa mengakses kamera: " + err);
});

// Ambil foto
captureBtn.addEventListener('click', () => {
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
  
  // OCR menggunakan Tesseract.js
  Tesseract.recognize(
    canvas,
    ocrLang.value,
    { logger: m => console.log(m) }
  ).then(({ data: { text } }) => {
    ocrResult.value = text;
  });
});

// Kirim ke WhatsApp
sendWA.addEventListener('click', () => {
  const text = encodeURIComponent(ocrResult.value);
  window.open(`https://wa.me/?text=${text}`, '_blank');
});
</script>

</body>
</html>
