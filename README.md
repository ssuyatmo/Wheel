<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Checklist HD785-7</title>
<style>
body {
  font-family: "Poppins", sans-serif;
  margin: 0;
  background: linear-gradient(to bottom right, #d8e9a8, #9cdba6);
  color: #222;
}

header {
  position: fixed;
  top: 0;
  left: 0;
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

/* === BAGIAN KONTEN UTAMA === */
.container {
  padding: 10px;
  max-width: 600px;
  margin: auto;
  transition: margin-top 0.3s ease;
}

.card {
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  padding: 10px;
  margin-bottom: 10px;
  transition: background 0.3s;
}

h3 {
  background: #cfe8cf;
  padding: 6px;
  border-radius: 6px;
  font-size: 15px;
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

textarea, input[type="text"], input[type="number"], input[type="date"], select {
  width: 100%;
  border-radius: 6px;
  border: 1px solid #ccc;
  resize: none;
  margin-top: 4px;
  padding: 5px;
  font-size: 13px;
}

.buttons {
  display: flex;
  justify-content: space-around;
  margin-top: 10px;
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

/* === POPUP TEXTAREA MELAYANG === */
#popupTemuan {
  display: none;
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0,0,0,0.4);
  backdrop-filter: blur(3px);
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.popup-box {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.3);
  width: 90%;
  max-width: 400px;
  padding: 15px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.popup-box h4 {
  margin: 0;
  text-align: center;
  font-size: 16px;
  background: #d8e9a8;
  padding: 6px;
  border-radius: 8px;
}

.popup-box textarea {
  height: 120px;
  border-radius: 8px;
  border: 1px solid #ccc;
  padding: 8px;
  font-size: 14px;
}

.popup-box .actions {
  display: flex;
  justify-content: space-around;
}

.popup-box button {
  padding: 6px 14px;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  background: #f1d28a;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

</style>
</head>
<body>

<header>Checklist Harian HD785-7</header>

<div class="fixed-header">
  <div class="card">
    <label>Operator</label>
    <input type="text" placeholder="Nama Operator">
  </div>
</div>

<div class="container">
  <div class="card">
    <h3>Oil Level</h3>
    <div class="label">
      <span>Engine Oil</span>
      <button class="ok">OK</button>
      <button class="notok">NOT OK</button>
    </div>
    <div class="label">
      <span>Hydraulic Oil</span>
      <button class="ok">OK</button>
      <button class="notok">NOT OK</button>
    </div>

    <button class="main" onclick="openPopup()">Input Temuan</button>
  </div>
</div>

<!-- === POPUP MELAYANG UNTUK TEMUAN === -->
<div id="popupTemuan">
  <div class="popup-box">
    <h4>Catat Temuan</h4>
    <textarea id="inputTemuan" placeholder="Tuliskan temuan di sini..."></textarea>
    <div class="actions">
      <button onclick="saveTemuan()">Simpan</button>
      <button onclick="closePopup()">Batal</button>
    </div>
  </div>
</div>

<script>
// === Atur margin container otomatis ===
function adjustContainerMargin() {
  const header = document.querySelector('header');
  const fixed = document.querySelector('.fixed-header');
  const container = document.querySelector('.container');
  if (header && fixed && container) {
    const totalHeight = header.offsetHeight + fixed.offsetHeight + 20;
    container.style.marginTop = totalHeight + 'px';
  }
}
window.addEventListener('load', adjustContainerMargin);
window.addEventListener('resize', adjustContainerMargin);

// === Fungsi Popup Temuan ===
function openPopup() {
  document.getElementById('popupTemuan').style.display = 'flex';
}
function closePopup() {
  document.getElementById('popupTemuan').style.display = 'none';
}
function saveTemuan() {
  const teks = document.getElementById('inputTemuan').value.trim();
  if (teks !== "") alert("Temuan tersimpan:\n" + teks);
  closePopup();
}

// Tombol OK/NOT OK toggle
document.querySelectorAll('.ok, .notok').forEach(btn=>{
  btn.addEventListener('click',()=>{
    const group = btn.parentElement.querySelectorAll('.ok, .notok');
    group.forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
  });
});
</script>
</body>
</html>
