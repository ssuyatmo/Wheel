<!DOCTYPE html>  <html lang="id">  
<head>  
  <meta charset="UTF-8">  
  <meta name="viewport" content="width=device-width, initial-scale=1.0">  
  <title>Validasi Checklist HD785-7</title>  
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">  
  <style>  
    body {  
      font-family: 'Segoe UI', sans-serif;  
      background: linear-gradient(rgba(255,255,255,0.7), rgba(255,255,255,0.7)),  
        url('2a.-Banner-Portrait-4-alat-yg-wajib-dimiliki-di-tambang.jpg') center/cover no-repeat fixed;  
      color: #000;  
      min-height: 100vh;  
    }  
    .container {  
      background-color: rgba(255,255,255,0.95);  
      padding: 2rem;  
      border-radius: 15px;  
      margin-top: 2rem;  
      box-shadow: 0 8px 24px rgba(0,0,0,0.3);  
    }  
    .tab-pane { display: none; }  
    .tab-pane.active { display: block; }  
    footer {  
      text-align: center;  
      margin-top: 2rem;  
      color: #333;  
    }  
    .nav-link { cursor: pointer; }  
    .riwayat-img {  
      max-width: 100%;  
      height: auto;  
      margin-bottom: 10px;  
      cursor: zoom-in;  
    }  
  </style>  
</head>  
<body>  
  <div class="container">  
    <ul class="nav nav-tabs mb-3">  
      <li class="nav-item"><a class="nav-link" onclick="ubahTab('login')">Login</a></li>  
      <li class="nav-item"><a class="nav-link" onclick="ubahTab('checklist')">Checklist</a></li>  
      <li class="nav-item"><a class="nav-link" onclick="ubahTab('riwayat')">Riwayat</a></li>  
      <li class="nav-item"><a class="nav-link" onclick="ubahTab('sandi')">Ganti Sandi</a></li>  
      <li class="nav-item"><a class="nav-link" onclick="ubahTab('logout')">Logout</a></li>  
    </ul>  <div class="tab-pane active" id="login">  
    <h3>Login</h3>  
    <p class="text-warning"><strong></strong></p>  
    <input type="password" id="loginPass" class="form-control mb-2" placeholder="Masukan Sandi">  
    <button class="btn btn-primary" onclick="doLogin()">Login</button>  
  </div>  

  <div class="tab-pane" id="checklist">  
    <h3>Validasi Checklist</h3>  
    <input type="text" class="form-control mb-2" placeholder="Kode Unit" id="unit">  
    <input type="text" class="form-control mb-2" placeholder="HM Unit" id="hm">  
    <input type="text" class="form-control mb-2" placeholder="Nama Mekanik" id="mekanik">  
    <input type="text" class="form-control mb-2" placeholder="Nama Leader" id="leader">  
    <textarea class="form-control mb-2" placeholder="Catatan" id="catatan"></textarea>  
    <label class="form-label">Unggah Foto (maks. 5):</label>  
    <input type="file" class="form-control mb-2" id="gambar1">  
    <input type="file" class="form-control mb-2" id="gambar2">  
    <input type="file" class="form-control mb-2" id="gambar3">  
    <input type="file" class="form-control mb-2" id="gambar4">  
    <input type="file" class="form-control mb-2" id="gambar5">  
    <button class="btn btn-success" onclick="simpanChecklist()">Simpan Checklist</button>  
  </div>  

  <div class="tab-pane" id="riwayat">  
    <h3>Riwayat Checklist</h3>  
    <input type="text" class="form-control mb-2" id="cari" placeholder="Cari berdasarkan nama unit..." oninput="tampilkanRiwayat()">  
    <div id="listRiwayat"></div>  
  </div>  

  <div class="tab-pane" id="sandi">  
    <h3>Ganti Sandi</h3>  
    <input type="password" class="form-control mb-2" id="sandiBaru" placeholder="Sandi Baru">  
    <button class="btn btn-warning" onclick="gantiSandi()">Ganti</button>  
  </div>  
  <div class="tab-pane" id="logout">  
    <h3>Logout</h3>  
    <button class="btn btn-danger" onclick="logout()">Logout</button>  
  </div>  
</div>

  </div>    <footer>  
    &copy; 2025 Kelengkapan & Validasi Checklist HD785-7 By: Suyatmo(Nrp:61140012)/IOU(Nim:049026401)  
  </footer>    <script>  
    let dataChecklist = JSON.parse(localStorage.getItem('checklist')) || [];  
    let sandi = localStorage.getItem('sandi') || 'admin';  
    let sudahLogin = false;  
  
    function doLogin() {  
      const pass = document.getElementById('loginPass').value;  
      if (pass === sandi) {  
        sudahLogin = true;  
        ubahTab('checklist');  
      } else {  
        alert('Sandi salah!');  
      }  
    }  
  
    function simpanChecklist() {  
      const unit = document.getElementById('unit').value;  
      const hm = document.getElementById('hm').value;  
      const tanggal = new Date().toLocaleString();  
      const mekanik = document.getElementById('mekanik').value;  
      const leader = document.getElementById('leader').value;  
      const catatan = document.getElementById('catatan').value;  
      const gambarInputIds = ['gambar1','gambar2','gambar3','gambar4','gambar5'];  
      const gambarFiles = gambarInputIds.map(id => document.getElementById(id).files[0]).filter(file => file);  
      if (!unit || !hm || !mekanik || !leader || !catatan || gambarFiles.length === 0) {  
        alert('Lengkapi semua data! Minimal 1 gambar.');  
        return;  
      }  
      const readerPromises = gambarFiles.map(file => new Promise(resolve => {  
        const reader = new FileReader();  
        reader.onload = e => resolve(e.target.result);  
        reader.readAsDataURL(file);  
      }));  
      Promise.all(readerPromises).then(gambarBase64 => {  
        dataChecklist.push({ unit, hm, tanggal, mekanik, leader, catatan, gambar: gambarBase64 });  
        localStorage.setItem('checklist', JSON.stringify(dataChecklist));  
        alert('Checklist disimpan!');  
        document.querySelectorAll('#checklist input, #checklist textarea').forEach(el => el.value = '');  
        tampilkanRiwayat();  
      });  
    }  
  
    function tampilkanRiwayat() {  
      const cari = document.getElementById('cari')?.value?.toLowerCase() || '';  
      const hasil = dataChecklist.map((d, index) => ({...d, index})).filter(d => d.unit.toLowerCase().includes(cari));  
      document.getElementById('listRiwayat').innerHTML = hasil.map(d => `  
        <div class='card bg-light text-dark mb-3'>  
          <div class='card-body'>  
            <h5>${d.unit}</h5>  
            <p>Tanggal: ${d.tanggal}, HM: ${d.hm}</p>  
            <p>Mekanik: ${d.mekanik}, Leader: ${d.leader}</p>  
            <p>Catatan: ${d.catatan}</p>  
            ${d.gambar.map(src => `<div><a href='${src}' download><img src='${src}' class='riwayat-img' onclick='zoomImage(this)'></a></div>`).join('')}  
            <br><button class='btn btn-sm btn-danger mt-2' onclick='hapusRiwayat(${d.index})'>Hapus</button>  
          </div>  
        </div>  
      `).join('');  
    }  
  
    function hapusRiwayat(index) {  
      if (confirm('Yakin ingin menghapus riwayat ini?')) {  
        dataChecklist.splice(index, 1);  
        localStorage.setItem('checklist', JSON.stringify(dataChecklist));  
        tampilkanRiwayat();  
      }  
    }  
  
    function gantiSandi() {  
      const baru = document.getElementById('sandiBaru').value;  
      if (!baru) return alert('Sandi tidak boleh kosong!');  
      localStorage.setItem('sandi', baru);  
      sandi = baru;  
      alert('Sandi berhasil diganti!');  
    }  
  
    function logout() {  
      sudahLogin = false;  
      ubahTab('login');  
    }  
  
    function ubahTab(id) {  
      if (!sudahLogin && id !== 'login') {  
        alert('Harap login terlebih dahulu');  
        id = 'login';  
      }  
      document.querySelectorAll('.tab-pane').forEach(p => p.classList.remove('active'));  
      document.getElementById(id).classList.add('active');  
    }  
  
    function zoomImage(img) {  
      const modal = window.open('', '_blank');  
      modal.document.write(`<img src='${img.src}' style='width:100%'><br><button onclick='window.close()'>Kembali</button>`);  
    }  
  
    tampilkanRiwayat();  
  </script>    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script> 
  <!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-storage.js"></script>

<script>
  // GANTI DENGAN KONFIGURASI FIREBASE KAMU
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT_ID.firebaseio.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };
  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();
  const storage = firebase.storage();

  let sandi = localStorage.getItem('sandi') || 'admin';
  let sudahLogin = localStorage.getItem('sudahLogin') === 'true';

  function doLogin() {
    const pass = document.getElementById('loginPass').value;
    if (pass === sandi) {
      sudahLogin = true;
      localStorage.setItem('sudahLogin', 'true');
      ubahTab('checklist');
    } else {
      alert('Sandi salah!');
    }
  }

  function simpanChecklist() {
    const unit = document.getElementById('unit').value;
    const hm = document.getElementById('hm').value;
    const tanggal = new Date().toLocaleString();
    const mekanik = document.getElementById('mekanik').value;
    const leader = document.getElementById('leader').value;
    const catatan = document.getElementById('catatan').value;
    const fileInputs = ['gambar1', 'gambar2', 'gambar3', 'gambar4', 'gambar5'];
    const files = fileInputs.map(id => document.getElementById(id).files[0]).filter(Boolean);

    if (!unit || !hm || !mekanik || !leader || !catatan || files.length === 0) {
      alert("Lengkapi semua data dan unggah minimal 1 gambar!");
      return;
    }

    const checklistId = db.ref().child('checklists').push().key;
    const uploadPromises = files.map(file =>
      storage.ref(`checklist/${checklistId}/${file.name}`).put(file)
        .then(snap => snap.ref.getDownloadURL())
    );

    Promise.all(uploadPromises).then(urls => {
      const data = { unit, hm, tanggal, mekanik, leader, catatan, gambar: urls };
      db.ref(`checklists/${checklistId}`).set(data).then(() => {
        alert("Checklist disimpan!");
        fileInputs.forEach(id => document.getElementById(id).value = '');
        ['unit', 'hm', 'mekanik', 'leader', 'catatan'].forEach(id => document.getElementById(id).value = '');
        tampilkanRiwayat();
      });
    }).catch(err => alert('Gagal unggah gambar: ' + err));
  }

  function tampilkanRiwayat() {
    const cari = document.getElementById('cari')?.value?.toLowerCase() || '';
    db.ref('checklists').once('value').then(snapshot => {
      const data = snapshot.val() || {};
      const hasil = Object.entries(data).filter(([_, d]) => d.unit.toLowerCase().includes(cari));
      document.getElementById('listRiwayat').innerHTML = hasil.map(([id, d]) => `
        <div class="card bg-light mb-3">
          <div class="card-body">
            <h5>${d.unit}</h5>
            <p>Tanggal: ${d.tanggal}, HM: ${d.hm}</p>
            <p>Mekanik: ${d.mekanik}, Leader: ${d.leader}</p>
            <p>Catatan: ${d.catatan}</p>
            ${d.gambar.map(src => `<div><a href="${src}" download><img src="${src}" class="riwayat-img" onclick="zoomImage(this)"></a></div>`).join('')}
            <button class="btn btn-sm btn-danger mt-2" onclick="hapusRiwayat('${id}')">Hapus</button>
          </div>
        </div>
      `).join('');
    });
  }

  function hapusRiwayat(id) {
    if (confirm('Hapus checklist ini?')) {
      db.ref('checklists/' + id).remove().then(tampilkanRiwayat);
    }
  }

  function gantiSandi() {
    const baru = document.getElementById('sandiBaru').value;
    if (!baru) return alert('Sandi tidak boleh kosong!');
    sandi = baru;
    localStorage.setItem('sandi', baru);
    alert('Sandi berhasil diganti!');
  }

  function logout() {
    localStorage.removeItem('sudahLogin');
    sudahLogin = false;
    ubahTab('login');
  }

  function ubahTab(id) {
    if (!sudahLogin && id !== 'login') {
      alert('Harap login dahulu');
      id = 'login';
    }
    document.querySelectorAll('.tab-pane').forEach(p => p.classList.remove('active'));
    document.querySelectorAll('.nav-link').forEach(n => n.classList.remove('active'));
    document.getElementById(id).classList.add('active');
    document.querySelector(`.nav-link[onclick="ubahTab('${id}')"]`)?.classList.add('active');
    if (id === 'riwayat') tampilkanRiwayat();
  }

  function zoomImage(img) {
    const win = window.open('', '_blank');
    win.document.write(`<img src="${img.src}" style="width:100%"><br><button onclick="window.close()">Kembali</button>`);
  }

  window.onload = () => ubahTab(sudahLogin ? 'checklist' : 'login');
</script> </body>  
</html>
