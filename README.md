<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Inspection QA - HD785</title>
<style>
  :root{
    --bg1:#f0f6ff; --bg2:#e8fff4; --card:#ffffff;
    --accent:#2c8c6a; --muted:#666;
  }
  body{
    margin:0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial;
    background: linear-gradient(135deg,var(--bg1),var(--bg2));
    padding:18px;
  }
  .card{
    background:var(--card);
    border-radius:14px;
    box-shadow: 0 8px 24px rgba(30,60,80,0.12);
    padding:18px; max-width:980px; margin:0 auto 18px;
  }
  h1{margin:0 0 8px; font-size:20px; color:var(--accent); text-align:center}
  label{display:block; font-weight:700; color:#234; margin-top:10px}
  input[type=text], input[type=date], textarea, select{
    width:100%; padding:10px; border-radius:8px; border:1px solid #d2d8dd; box-sizing:border-box;
    margin-top:6px; background:#fff;
  }
  textarea{min-height:80px; resize:vertical}
  .grid{
    display:grid; grid-template-columns: repeat(auto-fit,minmax(220px,1fr)); gap:10px; margin-top:8px;
  }
  .checkbox-item{
    display:flex; align-items:center; justify-content:space-between;
    gap:12px; padding:8px 10px; border-radius:8px; background:linear-gradient(180deg,#fff,#fbfbfb);
    border:1px solid #e6e6e6; box-shadow: 0 3px 8px rgba(20,30,40,0.04);
  }
  .checkbox-item label{margin:0; font-weight:600; color:#1f2b2a}
  .checkbox-item input[type=checkbox]{transform:scale(1.18)}
  .inline-row{display:flex; gap:8px; align-items:center; flex-wrap:wrap}
  .inline-row input[type=text]{flex:1; min-width:90px}
  .actions{display:flex; gap:10px; margin-top:14px; flex-wrap:wrap}
  .btn{padding:12px 14px; border-radius:10px; border:none; cursor:pointer; font-weight:800}
  .btn-preview{background:#f6c84c;color:#082; box-shadow:0 4px 10px rgba(246,200,76,0.18)}
  .btn-wa{background:#25d366;color:#fff}
  .btn-copy{background:#3498db;color:#fff}
  .output{white-space:pre-wrap; font-family:Menlo,monospace; background:#0f1720; color:#f8fafc; padding:14px; border-radius:10px; margin-top:14px; min-height:120px}
  .small{font-size:13px; color:var(--muted)}
  .preview-images {display:flex; flex-wrap:wrap; gap:8px; margin-top:8px;}
  .preview-images img{width:100px; height:100px; object-fit:cover; border-radius:8px; border:1px solid #ccc}
  @media (max-width:520px){
    .inline-row{flex-direction:column; align-items:stretch}
  }
</style>
</head>
<body>

<div class="card">
  <h1>Inspection Report — HD Series</h1>

  <!-- Bagian form tetap sama sampai bagian Deviation -->
  <!-- ... (semua form input tetap sama seperti versi kamu) ... -->

  <label style="margin-top:12px">*Deviation* (masukkan tiap baris baru untuk tiap temuan)</label>
  <textarea id="deviation" placeholder="contoh: canopy atas cabin crack 200mm&#10;gasket coupling oil cooler transmission leak"></textarea>

  <!-- 🔽 Tambahan: Upload foto deviasi -->
  <label>📸 Foto Deviasi (bisa lebih dari satu)</label>
  <input type="file" id="deviationImages" accept="image/*" multiple>
  <div id="previewContainer" class="preview-images"></div>

  <div class="actions">
    <button class="btn btn-preview" onclick="previewText()">🔎 Preview</button>
    <button class="btn btn-wa" onclick="sendWA()">📤 Send to WhatsApp</button>
    <button class="btn btn-copy" onclick="copyText()">📋 Copy</button>
  </div>

  <div class="output" id="output">Preview akan muncul di sini — klik Preview.</div>
  <div class="small" style="margin-top:8px">
    Catatan: "Send to WhatsApp" membuka WhatsApp dengan teks sudah terisi; tekan Send di aplikasi WhatsApp untuk kirim.
  </div>
</div>

<script>
  // default date = today
  (function(){ const d=new Date(); document.getElementById('tanggal').value=d.toISOString().split('T')[0]; })();

  // tyre radio show/hide
  tyreGood.onchange=()=>tyreFindDiv.style.display='none';
  tyreBad.onchange=()=>tyreFindDiv.style.display='block';

  function getCheck(id){return document.getElementById(id)?.checked?'✅':'❌';}

  function formatDeviationLines(raw){
    if(!raw)return'None';
    const lines=raw.split(/\r?\n/).map(l=>l.trim()).filter(l=>l.length>0);
    if(!lines.length)return'None';
    return lines.map(l=>'⚠️ '+l).join('\\n');
  }

  // 🔽 Preview gambar deviasi
  const inputImg=document.getElementById('deviationImages');
  const preview=document.getElementById('previewContainer');
  let selectedImages=[];

  inputImg.addEventListener('change',()=>{
    preview.innerHTML='';
    selectedImages=Array.from(inputImg.files);
    selectedImages.forEach(file=>{
      const img=document.createElement('img');
      img.src=URL.createObjectURL(file);
      preview.appendChild(img);
    });
  });

  function generateText(){
    const qa=qaType.value, date=tanggal.value, mekanik=mekanik.value||'-';
    const cn=cn.value||'-', hm=hm.value||'-';
    const engineOil=getCheck('engineOil'), transOil=getCheck('transOil'), hydOil=getCheck('hydOil');
    const belt=getCheck('belt'), oilLeak=getCheck('oilLeak'), crc=getCheck('crc'), injector=getCheck('injector');
    const radio=getCheck('radio'), fatigue=getCheck('fatigue'), pw=getCheck('pw');
    const power=powerSupply.value||'-', crp=crp.value||'-';
    const seat=getCheck('seat'), rail=getCheck('rail');
    const fl=fl.value||'-', fr=fr.value||'-', rl=rl.value||'-', rr=rr.value||'-';
    const tyre=document.querySelector('input[name="tyre"]:checked')?.value||'Good';
    const tyreFinding=tyreFinding.value||'';
    const jobCard=getCheck('jobCard'), redo=getCheck('redo'), qaForm=getCheck('qaForm');
    const ppm=getCheck('ppm'), a=getCheck('a'), b=getCheck('b'), c=getCheck('c');
    const backlog=getCheck('backlog'), fui=getCheck('fui'), ro=getCheck('ro');
    const combine=getCheck('combine'), sar=getCheck('sar');
    const deviationRaw=deviation.value, deviation=formatDeviationLines(deviationRaw);
    const suspensionLine=`FL : ${fl}   FR : ${fr}   RL : ${rl}   RR : ${rr}`;

    // 🔽 Tambahan: jumlah foto deviasi
    const imgCount=selectedImages.length;
    const imgInfo=imgCount>0?`📷 ${imgCount} foto deviasi terlampir (tidak terkirim via WA, hanya lokal preview).`:'Tidak ada foto deviasi.';

    const text=
`*${qa}*

📅 Tgl : ${date}
👷 Mekanik : ${mekanik}

🚗 CN : ${cn}
⌛ HM : ${hm}

🩸 *Oil Level* 🩸
Engine oil level : ${engineOil}
Transmission oil level : ${transOil}
Hydraulic oil level : ${hydOil}

⚙ *Engine Area* ⚙
Belt tension : ${belt}
Engine oil leakage : ${oilLeak}
Common Rail Connector : ${crc}
Injector Tube : ${injector}

🚗 *Cabin Area* 🚗
FM Radio : ${radio}
Fatigue Warning : ${fatigue}
Power Window : ${pw}
⚡ Power Supply : ${power}
💧 Common Rail Pressure (ON) : ${crp}

🚗 *Frame Area* 🚗
Operator seat : ${seat}
Hand Rail : ${rail}

💧 *Pressure Suspension (Panel)* 💧
${suspensionLine}

🛞 *Tyre condition :*
Tyre : ${tyre}${tyre==='Bad' && tyreFinding?' - '+tyreFinding:''}

📝 Validasi & Kelengkapan Service :
- Job Card & Cover Checklist ${jobCard}
- Form Observasi Redo PS ${redo}
- Form QA 1 & QA 7 ${qaForm}
- Form PPM ${ppm}
- Form Check List A ${a}
- Form Check List B ${b}
- Form Check List C ${c}
- Form Backlog ${backlog}
- Form FUI ${fui}
- Form Repair Order ${ro}
- Form Combine Maintenance ${combine}
- Form Service Activity Report ${sar}

*Deviation :*
${deviation}

${imgInfo}`;

    return text;
  }

  function previewText(){
    output.innerText=generateText();
    output.scrollIntoView({behavior:'smooth'});
  }
  function sendWA(){
    window.open('https://wa.me/?text='+encodeURIComponent(generateText()),'_blank');
  }
  function copyText(){
    navigator.clipboard.writeText(generateText()).then(()=>alert('✅ Teks berhasil disalin.'));
  }
</script>

</body>
</html>
