<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <title>QA Inspection HD785-7</title>

  <style>
    * {
      box-sizing: border-box;
      font-family: "Segoe UI", sans-serif;
    }

    body {
      margin: 0;
      background: #f7f9ef;
    }

    header {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      background: #9acd32;
      color: #fff;
      text-align: center;
      padding: 10px 0;
      font-size: 20px;
      font-weight: bold;
      z-index: 200;
      box-shadow: 0 2px 5px rgba(0,0,0,0.2);
    }

    .fixed-header {
      position: fixed;
      top: 55px; /* posisi di bawah header */
      left: 0;
      width: 100%;
      background: #d8e9a8;
      z-index: 150;
      padding: 8px 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.2);
      display: flex;
      flex-direction: column;
      gap: 5px;
    }

    .card {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      align-items: center;
      justify-content: space-between;
    }

    label {
      font-weight: 600;
    }

    input, select {
      padding: 5px;
      border-radius: 6px;
      border: 1px solid #aaa;
      flex: 1;
      min-width: 120px;
    }

    .container {
      margin-top: 250px; /* beri ruang agar tidak tertutup header */
      padding: 10px;
    }

    .card-content {
      background: #fff;
      border-radius: 10px;
      padding: 10px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      margin-bottom: 10px;
    }

    .buttons {
      text-align: center;
      margin: 20px 0;
    }

    .buttons button {
      padding: 10px 20px;
      border: none;
      border-radius: 8px;
      background: #6da34d;
      color: white;
      font-weight: bold;
      margin: 5px;
      cursor: pointer;
    }

    .buttons button:hover {
      background: #588d3d;
    }

    #previewPopup {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.7);
      display: none;
      justify-content: center;
      align-items: center;
      z-index: 500;
    }

    #previewPopup pre {
      background: #fff;
      padding: 20px;
      border-radius: 10px;
      max-height: 80vh;
      overflow-y: auto;
      width: 80%;
      font-size: 14px;
      white-space: pre-wrap;
    }

    #popupButtons {
      text-align: center;
      margin-top: 10px;
    }

    #popupButtons button {
      margin: 5px;
      padding: 8px 16px;
      border: none;
      border-radius: 6px;
      background: #9acd32;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }

    /* Responsif untuk HP */
    @media (max-width: 600px) {
      .card {
        flex-direction: column;
        align-items: flex-start;
      }

      input, select {
        width: 100%;
      }

      .fixed-header {
        top: 50px;
        padding-bottom: 8px;
      }

      .container {
        margin-top: 280px;
      }
    }

    /* Agar header tetap terlihat saat keyboard muncul (Android/iOS) */
    @supports (height: 100dvh) {
      body {
        height: 100dvh;
        overflow-y: auto;
      }
      .fixed-header {
        position: fixed;
      }
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
      <label>👷 Mekanik:</label><input type="text" id="mekanik" placeholder="Nama Mekanik">
      <label>🚗 CN Unit:</label><input type="text" id="cn" placeholder="CN Unit">
      <label>⌛ HM:</label><input type="number" id="hm" placeholder="Hour Meter">
    </div>
  </div>

  <div class="container" id="mainContent">
    <div id="sections">
      <div class="card-content">🔧 Contoh Checklist 1</div>
      <div class="card-content">🔧 Contoh Checklist 2</div>
      <div class="card-content">🔧 Contoh Checklist 3</div>
      <div class="card-content">🔧 Contoh Checklist 4</div>
      <div class="card-content">🔧 Contoh Checklist 5</div>
      <div class="card-content">🔧 Contoh Checklist 6</div>
      <div class="card-content">🔧 Contoh Checklist 7</div>
      <div class="card-content">🔧 Contoh Checklist 8</div>
      <div class="card-content">🔧 Contoh Checklist 9</div>
      <div class="card-content">🔧 Contoh Checklist 10</div>
    </div>

    <div class="buttons">
      <button onclick="showPreview()">🧾 Preview</button>
      <button onclick="sendToWhatsApp()">📤 Kirim WA</button>
    </div>
  </div>

  <div id="previewPopup">
    <div>
      <pre id="previewText"></pre>
      <div id="popupButtons">
        <button onclick="copyText()">📋 Copy</button>
        <button onclick="sendToWhatsApp()">📤 WA</button>
        <button onclick="closePopup()">❌ Tutup</button>
      </div>
    </div>
  </div>

  <script>
    function showPreview() {
      const data = `
🌿 QA Inspection Komatsu HD785-7
Jenis QA: ${document.getElementById('qaType').value}
Tanggal: ${document.getElementById('tgl').value}
Mekanik: ${document.getElementById('mekanik').value}
CN Unit: ${document.getElementById('cn').value}
HM: ${document.getElementById('hm').value}

✅ Semua checklist sudah diperiksa.
`;
      document.getElementById('previewText').innerText = data;
      document.getElementById('previewPopup').style.display = "flex";
    }

    function closePopup() {
      document.getElementById('previewPopup').style.display = "none";
    }

    function copyText() {
      const text = document.getElementById('previewText').innerText;
      navigator.clipboard.writeText(text);
      alert("✅ Teks berhasil disalin!");
    }

    function sendToWhatsApp() {
      const text = encodeURIComponent(document.getElementById('previewText').innerText);
      window.open(`https://wa.me/?text=${text}`, "_blank");
    }
  </script>
</body>
</html>
