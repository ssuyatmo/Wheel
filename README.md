<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>QA-1 Pre Inspection HD785-7</title>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-storage-compat.js"></script>
  <style>
    body { font-family: Arial; padding: 20px; background: #f5f5f5; }
    h2 { text-align: center; }
    label { display: block; margin-top: 10px; font-weight: bold; }
    select, input, textarea { width: 100%; padding: 6px; margin-top: 4px; border-radius: 6px; border: 1px solid #ccc; }
    button { width: 100%; padding: 10px; margin-top: 15px; border: none; background: #007bff; color: #fff; font-weight: bold; border-radius: 6px; cursor: pointer; }
    button:hover { background: #0056b3; }
    .checklist { background: #fff; padding: 10px; border-radius: 8px; margin-top: 10px; }
    .section-title { font-weight: bold; background: #ddd; padding: 6px; border-radius: 4px; }
  </style>
</head>
<body>

<h2>📋 QA-1 PRE INSPECTION</h2>

<form id="inspectionForm">
  <label>Tanggal:</label>
  <input type="date" id="tanggal" required>

  <label>Nama Mekanik:</label>
  <input type="text" id="mekanik" placeholder="Masukkan nama mekanik" required>

  <label>CN Unit:</label>
  <input type="text" id="cn" placeholder="Misal: HD785-7-1032" required>

  <label>Hour Meter (HM):</label>
  <input type="number" id="hm" required>

  <div class="checklist">
    <div class="section-title">Oil Level</div>
    <label>Engine Oil:</label>
    <select id="engineOil"><option>✅ OK</option><option>⚠️ Not OK</option></select>
    <label>Transmission Oil:</label>
    <select id="transOil"><option>✅ OK</option><option>⚠️ Not OK</option></select>
    <label>Hydraulic Oil:</label>
    <select id="hydOil"><option>✅ OK</option><option>⚠️ Not OK</option></select>
  </div>

  <div class="checklist">
    <div class="section-title">Engine Area</div>
    <label>Belt Tension:</label>
    <select id="belt"><option>✅ OK</option><option>⚠️ Not OK</option></select>
    <label>Oil Leakage:</label>
    <select id="leak"><option>✅ OK</option><option>⚠️ Not OK</option></select>
  </div>

  <div class="checklist">
    <div class="section-title">Cabin Area</div>
    <label>Seat Belt:</label>
    <select id="seatBelt"><option>✅ OK</option><option>⚠️ Not OK</option></select>
    <label>Wiper Function:</label>
    <select id="wiper"><option>✅ OK</option><option>⚠️ Not OK</option></select>
  </div>

  <div class="checklist">
    <div class="section-title">Safety Item</div>
    <label>Fire Extinguisher:</label>
    <select id="fire"><option>✅ OK</option><option>⚠️ Not OK</option></select>
    <label>Horn & Lamp:</label>
    <select id="lamp"><option>✅ OK</option><option>⚠️ Not OK</option></select>
  </div>

  <div class="checklist">
    <div class="section-title">Fuel System</div>
    <label>Fuel Line:</label>
    <select id="fuelLine"><option>✅ OK</option><option>⚠️ Not OK</option></select>
    <label>Fuel Leakage:</label>
    <select id="fuelLeak"><option>✅ OK</option><option>⚠️ Not OK</option></select>
  </div>

  <div class="checklist">
    <div class="section-title">Hydraulic System</div>
    <label>Hose Condition:</label>
    <select id="hose"><option>✅ OK</option><option>⚠️ Not OK</option></select>
    <label>Fitting Tightness:</label>
    <select id="fitting"><option>✅ OK</option><option>⚠️ Not OK</option></select>
  </div>

  <label>Upload Foto Unit:</label>
  <input type="file" id="photo" accept="image/*" required>

  <button type="button" onclick="uploadData()">Kirim ke WhatsApp</button>
</form>

<script>
  // 🔧 Konfigurasi Firebase kamu isi di sini:
  const firebaseConfig = {
    apiKey: "ISI_API_KEY_KAMU",
    authDomain: "ISI_AUTH_DOMAIN_KAMU",
    projectId: "ISI_PROJECT_ID_KAMU",
    storageBucket: "ISI_STORAGE_BUCKET_KAMU",
    messagingSenderId: "ISI_MSG_SENDER_ID",
    appId: "ISI_APP_ID"
  };

  firebase.initializeApp(firebaseConfig);
  const storage = firebase.storage();

  async function uploadData() {
    const file = document.getElementById("photo").files[0];
    if (!file) return alert("Harap pilih foto terlebih dahulu!");

    const ref = storage.ref('inspection_photos/' + file.name);
    await ref.put(file);
    const photoURL = await ref.getDownloadURL();

    const tanggal = document.getElementById("tanggal").value;
    const mekanik = document.getElementById("mekanik").value;
    const cn = document.getElementById("cn").value;
    const hm = document.getElementById("hm").value;

    const msg = `📋 QA-1 PRE INSPECTION
Tgl: ${tanggal}
Mekanik: ${mekanik}
CN: ${cn}
HM: ${hm}

Oil Level:
${engineOil.value}
${transOil.value}
${hydOil.value}

Engine Area:
${belt.value}
${leak.value}

Cabin Area:
${seatBelt.value}
${wiper.value}

Safety Item:
${fire.value}
${lamp.value}

Fuel System:
${fuelLine.value}
${fuelLeak.value}

Hydraulic System:
${hose.value}
${fitting.value}

Bukti Foto:
${photoURL}

Selesai ✅`;

    const waUrl = `https://wa.me/?text=${encodeURIComponent(msg)}`;
    window.open(waUrl, '_blank');
  }
</script>

</body>
</html>
