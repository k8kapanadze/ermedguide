
<html lang="ka">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>CMC Emergency – მედიკამენტები</title>
<style>
  body{font-family:Arial, sans-serif;background:#f5f5f5;margin:0;padding:20px}
  h1{text-align:center;margin-bottom:12px}
  .container{max-width:900px;margin:auto;background:#fff;padding:20px;border-radius:10px;box-shadow:0 0 15px rgba(0,0,0,.08)}
  input,select,textarea{width:100%;padding:10px;margin:6px 0;border:1px solid #ccc;border-radius:6px;box-sizing:border-box}
  button{padding:10px 14px;border:none;border-radius:6px;background:#111;color:#fff;cursor:pointer;font-size:15px}
  .row{display:flex;gap:10px}
  .row .col{flex:1}
  #search{padding:12px;font-size:16px;margin-bottom:12px}
  .card{background:#fff;padding:14px;margin:12px 0;border-left:6px solid #d32f2f;border-radius:6px;box-shadow:0 2px 8px rgba(0,0,0,.06)}
  .card b{display:inline-block;margin-bottom:6px}
  .card .actions{margin-top:10px}
  .card .actions button{margin-right:8px}
  .delete{background:#d32f2f}
  .edit{background:#1976d2}
  .small{font-size:13px;color:#666}
  @media(max-width:700px){.row{flex-direction:column}}
</style>
</head>
<body>
  <h1>CMC Emergency – მედიკამენტების სია</h1>

  <div class="container">
    <input id="search" placeholder="ძებნა: სახელით ან ჩვენებით..." />

    <h2>მედიკამენტის დამატება / რედაქტირება</h2>

    <input id="name" placeholder="მედიკამენტის სახელი" />
    <input id="indication" placeholder="ჩვენება (indication)" />

    <div class="row">
      <div class="col">
        <label class="small">გაკეთების ადგილი</label>
        <select id="site">
          <option value="">აირჩიე</option>
          <option value="IV">IV</option>
          <option value="IM">IM</option>
          <option value="SubQ">SubQ</option>
          <option value="Oral">Oral</option>
          <option value="Inhalation">Inhalation</option>
          <option value="More">More (custom)</option>
        </select>
        <input id="siteCustom" placeholder="ჩაწერე custom გაკეთების ადგილი" style="display:none;margin-top:6px" />
      </div>

      <div class="col">
        <label class="small">განზავება</label>
        <select id="dilution">
          <option value="">აირჩიე</option>
          <option value="არა">არა</option>
          <option value="5 ml">5 ml</option>
          <option value="10 ml">10 ml</option>
          <option value="20 ml">20 ml</option>
          <option value="NaCl">NaCl</option>
          <option value="More">More (custom)</option>
        </select>
        <input id="dilutionCustom" placeholder="ჩაწერე custom განზავება (მაგ. 2 ml, D5W...)" style="display:none;margin-top:6px" />
      </div>
    </div>

    <input id="side" placeholder="გვერდითი ეფექტი (side effects)" />

    <div style="margin-top:8px">
      <button id="saveBtn">შენახვა</button>
    </div>

    <h2 style="margin-top:20px">სრული სია</h2>
    <div id="list"></div>
  </div>

<script>
  // მონაცემები
  let meds = JSON.parse(localStorage.getItem('meds') || '[]');
  let editIndex = null;

  // ელემენტები
  const searchEl = document.getElementById('search');
  const nameEl = document.getElementById('name');
  const indicationEl = document.getElementById('indication');
  const siteEl = document.getElementById('site');
  const siteCustomEl = document.getElementById('siteCustom');
  const dilutionEl = document.getElementById('dilution');
  const dilutionCustomEl = document.getElementById('dilutionCustom');
  const sideEl = document.getElementById('side');
  const listEl = document.getElementById('list');
  const saveBtn = document.getElementById('saveBtn');

  // დაფა
  function display(){
    const q = searchEl.value.trim().toLowerCase();
    listEl.innerHTML = '';

    if(meds.length === 0){
      listEl.innerHTML = '<p style="text-align:center;color:#777;padding:18px">ჯერ არ არის დამატებული მედიკამენტები</p>';
      return;
    }

    meds
      .map((m, i) => ({...m, _i:i}))
      .filter(m => {
        if(!q) return true;
        return (
          (m.name && m.name.toLowerCase().includes(q)) ||
          (m.indication && m.indication.toLowerCase().includes(q)) ||
          (m.site && (m.site + ' ' + (m.siteCustom||'')).toLowerCase().includes(q)) ||
          (m.dilution && (m.dilution + ' ' + (m.dilutionCustom||'')).toLowerCase().includes(q))
        );
      })
      .forEach(m => {
        const siteText = (m.site === 'More' ? (m.siteCustom || '—') : (m.site || '—'));
        const dilutionText = (m.dilution === 'More' ? (m.dilutionCustom || '—') : (m.dilution || '—'));

        const card = document.createElement('div');
        card.className = 'card';
        card.innerHTML = `
          <b style="font-size:1.05em">${escapeHtml(m.name || 'უსახელო')}</b>
          <div style="margin-top:8px"><b>ჩვენება:</b> ${escapeHtml(m.indication || '-')}</div>
          <div><b>გაკეთების ადგილი:</b> ${escapeHtml(siteText)}</div>
          <div><b>განზავება:</b> ${escapeHtml(dilutionText)}</div>
          <div><b>გვერდითი ეფექტი:</b> ${escapeHtml(m.side || '-')}</div>
          <div class="actions">
            <button class="edit" onclick="editMed(${m._i})">Edit</button>
            <button class="delete" onclick="deleteMed(${m._i})">Delete</button>
          </div>
        `;
        listEl.appendChild(card);
      });
  }

  // შენახვა (ახალი ან რედაქტირებული)
  function saveMedication(){
    const name = nameEl.value.trim();
    if(!name){
      alert('გთხოვ, მიუთითე მედიკამენტის სახელი.');
      return;
    }

    const med = {
      name: name,
      indication: indicationEl.value.trim(),
      site: siteEl.value,
      siteCustom: siteEl.value === 'More' ? siteCustomEl.value.trim() : '',
      dilution: dilutionEl.value,
      dilutionCustom: dilutionEl.value === 'More' ? dilutionCustomEl.value.trim() : '',
      side: sideEl.value.trim()
    };

    if(editIndex === null){
      meds.push(med);
    } else {
      meds[editIndex] = med;
      editIndex = null;
      saveBtn.textContent = 'შენახვა';
    }

    localStorage.setItem('meds', JSON.stringify(meds));
    clearForm();
    display();
  }

  // წაშლა
  function deleteMed(i){
    if(confirm('გსურთ მართლწინვე წაშლა?')){
      meds.splice(i,1);
      localStorage.setItem('meds', JSON.stringify(meds));
      // თუ ვამუშავებდი ამ ელემენტს, წავშალე რედაქტირება
      if(editIndex === i) { editIndex = null; clearForm(); saveBtn.textContent = 'შენახვა'; }
      display();
    }
  }

  // რედაქტირება
  function editMed(i){
    const m = meds[i];
    nameEl.value = m.name || '';
    indicationEl.value = m.indication || '';
    siteEl.value = m.site || '';
    if(m.site === 'More'){
      siteCustomEl.style.display = 'block';
      siteCustomEl.value = m.siteCustom || '';
    } else {
      siteCustomEl.style.display = 'none';
      siteCustomEl.value = '';
    }
    dilutionEl.value = m.dilution || '';
    if(m.dilution === 'More'){
      dilutionCustomEl.style.display = 'block';
      dilutionCustomEl.value = m.dilutionCustom || '';
    } else {
      dilutionCustomEl.style.display = 'none';
      dilutionCustomEl.value = '';
    }
    sideEl.value = m.side || '';
    editIndex = i;
    saveBtn.textContent = 'შენახვა (რედაქტირება)';
    window.scrollTo({top:0,behavior:'smooth'});
  }

  // ფორმის გასუფთავება
  function clearForm(){
    nameEl.value = '';
    indicationEl.value = '';
    siteEl.value = '';
    siteCustomEl.value = '';
    siteCustomEl.style.display = 'none';
    dilutionEl.value = '';
    dilutionCustomEl.value = '';
    dilutionCustomEl.style.display = 'none';
    sideEl.value = '';
    editIndex = null;
    saveBtn.textContent = 'შენახვა';
  }

  // ევენთები
  saveBtn.addEventListener('click', saveMedication);
  searchEl.addEventListener('input', display);

  siteEl.addEventListener('change', () => {
    if(siteEl.value === 'More') siteCustomEl.style.display = 'block';
    else siteCustomEl.style.display = 'none';
  });
  dilutionEl.addEventListener('change', () => {
    if(dilutionEl.value === 'More') dilutionCustomEl.style.display = 'block';
    else dilutionCustomEl.style.display = 'none';
  });

  // უსაფრთხო HTML გამოსვლა
  function escapeHtml(str){
    if(!str) return '';
    return String(str)
      .replace(/&/g, '&amp;')
      .replace(/</g,'&lt;')
      .replace(/>/g,'&gt;')
      .replace(/"/g,'&quot;')
      .replace(/'/g,'&#039;');
  }

  // expose to global for onclick attributes
  window.editMed = editMed;
  window.deleteMed = deleteMed;

  // საწყისი ჩვენება
  display();

</script>
</body>
</html>
