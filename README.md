<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Camera & Firebase OCR & WA</title>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://cdn.jsdelivr.net/npm/tesseract.js@4.1.1/dist/tesseract.min.js"></script>
<style>
body{font-family:Arial;margin:10px;background:#f9f9f9;color:#222;}
video,img{width:100%;border-radius:8px;max-height:50vh;object-fit:cover;}
textarea{width:100%;height:120px;margin-top:8px;border-radius:8px;border:1px solid #ddd;padding:8px;}
button,input,select{padding:10px;margin-top:8px;border-radius:8px;border:1px solid #ccc;width:100%;}
#progressContainer{width:100%;background:#eee;border-radius:8px;overflow:hidden;height:18px;display:none;margin-top:8px;}
#progressBar{width:0%;height:100%;background:#4caf50;color:#fff;text-align:center;font-size:12px;line-height:18px;}
</style>
</head>
<body>

<h2>Camera & Firebase OCR & WA</h2>
<video id="video" autoplay playsinline></video>
<input type="file" id="uploadPhoto" accept="image/*">
<button id="capture">📸 Ambil Foto & OCR</button>
<img id="photoPreview" alt="Preview" style="display:none"/>
<div id="progressContainer"><div id="progressBar">0%</div></div>
<textarea id="ocrResult" placeholder="Hasil OCR akan muncul disini..."></textarea>
<input type="text" id="ocrLang" value="eng" placeholder="Bahasa OCR (eng/ind/jpn)">
<button id="sendWA" style="background:#25D366;color:white;">💬 Kirim ke WhatsApp</button>

<script>
// Firebase Config
const firebaseConfig = {
  apiKey: "API_KEY",
  authDomain: "PROJECT_ID.firebaseapp.com",
  projectId: "PROJECT_ID",
  storageBucket: "PROJECT_ID.appspot.com",
  messagingSenderId: "SENDER_ID",
  appId: "APP_ID"
};
firebase.initializeApp(firebaseConfig);
firebase.auth().signInAnonymously().catch(console.error);
const storage = firebase.storage();

// Kamera
const video = document.getElementById('video');
navigator.mediaDevices.getUserMedia({video:{facingMode:'environment'}})
.then(stream=>video.srcObject=stream)
.catch(e=>alert('Kamera error: '+e.message));

const canvas = document.createElement('canvas');
const photoPreview = document.getElementById('photoPreview');
const ocrResult = document.getElementById('ocrResult');
const progressContainer = document.getElementById('progressContainer');
const progressBar = document.getElementById('progressBar');
const ocrLangInput = document.getElementById('ocrLang');

// Ambil foto & OCR
document.getElementById('capture').addEventListener('click', async ()=>{
  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;
  canvas.getContext('2d').drawImage(video,0,0);
  const dataUrl = canvas.toDataURL('image/png');
  photoPreview.src = dataUrl;
  photoPreview.style.display='block';
  await uploadAndOCR(dataUrl);
});

// Upload foto
document.getElementById('uploadPhoto').addEventListener('change', async (e)=>{
  if(!e.target.files[0]) return;
  const file = e.target.files[0];
  const reader = new FileReader();
  reader.onload = async ()=> {
    const dataUrl = reader.result;
    photoPreview.src = dataUrl;
    photoPreview.style.display='block';
    await uploadAndOCR(dataUrl);
  };
  reader.readAsDataURL(file);
});

// Upload ke Firebase + OCR
async function uploadAndOCR(dataUrl){
  progressContainer.style.display='block';
  progressBar.style.width='0%';
  progressBar.innerText='0%';
  ocrResult.value='Sedang memproses...';

  try{
    // Convert dataURL ke blob
    const blob = await (await fetch(dataUrl)).blob();
    const filename = 'foto_'+Date.now()+'.png';
    const ref = storage.ref().child(filename);
    await ref.put(blob);

    // OCR
    const lang = ocrLangInput.value || 'eng';
    const worker = Tesseract.createWorker({
      logger:m=>{
        if(m.status==='recognizing text'||m.status==='loading tesseract core'){
          const p = Math.round(m.progress*100);
          progressBar.style.width = p+'%';
          progressBar.innerText = p+'%';
        }
      }
    });
    await worker.load();
    await worker.loadLanguage(lang);
    await worker.initialize(lang);
    const { data:{ text } } = await worker.recognize(blob);
    ocrResult.value = text;
    progressBar.style.width='100%';
    progressBar.innerText='Selesai';
    await worker.terminate();
  }catch(e){alert('Error: '+e.message);}
}

// Kirim WA
document.getElementById('sendWA').addEventListener('click', ()=>{
  const text = encodeURIComponent(ocrResult.value || '');
  window.open(`https://wa.me/?text=${text}`,'_blank');
});
</script>

</body>
</html>
