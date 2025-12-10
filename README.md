
<html lang="ka">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>ermedguide — CMC Emergency მედიკამენტები (online)</title>
<style>
body{font-family:Arial,sans-serif;background:#f5f5f5;margin:0;padding:20px}
h1{text-align:center;margin-bottom:12px}
.container{max-width:980px;margin:auto;background:#fff;padding:20px;border-radius:10px;box-shadow:0 0 18px rgba(0,0,0,.06)}
input,select,textarea,button{font-family:inherit}
input,select,textarea{width:100%;padding:10px;margin:6px 0;border:1px solid #ccc;border-radius:6px;box-sizing:border-box}
textarea{min-height:80px;resize:vertical}
button{padding:10px 14px;border:none;border-radius:6px;background:#111;color:#fff;cursor:pointer;font-size:15px}
.row{display:flex;gap:12px}
.row .col{flex:1}
#search{padding:12px;font-size:16px;margin-bottom:12px}
.card{background:#fff;padding:14px;margin:12px 0;border-left:6px solid #d32f2f;border-radius:6px;box-shadow:0 2px 10px rgba(0,0,0,.04)}
.drug-name{color:#d32f2f;font-weight:700;font-size:1.08em;margin-bottom:6px}
.meta{color:#444;margin:4px 0}
.small{font-size:13px;color:#666}
.actions{margin-top:10px}
.actions button{margin-right:8px}
.delete{background:#d32f2f}
.edit{background:#1976d2}
.status{font-size:13px;color:#666;margin-bottom:6px}
@media(max-width:760px){.row{flex-direction:column}}
</style>
</head>
<body>
<h1>ermedguide — CMC Emergency მედიკამენტები (online)</h1>
<div class="container">
  <div class="status" id="status">ერთდება საბაზისო სერვისს...</div>

  <input id="search" placeholder="ძებნა: სახელით ან ჩვენებით..." />

  <h2>მედიკამენტის დამატება / რედაქტირება</h2>

  <input id="name" placeholder="მედიკამენტის სახელი (წითლად)" />
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
      <input id="siteCustom" placeholder="ჩასაწერად - custom გაკეთების ადგილი" style="display:none;margin-top:6px" />
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
      <input id="dilutionCustom" placeholder="ჩასაწერად - custom განზავება" style="display:none;margin-top:6px" />
    </div>
  </div>

  <textarea id="description" placeholder="მოკლე აღწერა (დამატებითი ინფორმაცია)"></textarea>
  <input id="side" placeholder="გვერდითი ეფექტი (side effects)" />

  <div style="margin-top:8px">
    <button id="saveBtn">შენახვა</button>
  </div>

  <h2 style="margin-top:20px">სრული სია</h2>
  <div id="list"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js/dist/supabase.min.js"></script>
<script>
const SUPABASE_URL='https://upkknbvujqwtekcutxum.supabase.co';
const SUPABASE_KEY='sb_publishable_lfg87qO215RkGlCU-NvRyA_oKlsoCS8';
const supabase=supabase.createClient(SUPABASE_URL,SUPABASE_KEY);

const statusEl=document.getElementById('status');
const searchEl=document.getElementById('search');
const nameEl=document.getElementById('name');
const indicationEl=document.getElementById('indication');
const siteEl=document.getElementById('site');
const siteCustomEl=document.getElementById('siteCustom');
const dilutionEl=document.getElementById('dilution');
const dilutionCustomEl=document.getElementById('dilutionCustom');
const descriptionEl=document.getElementById('description');
const sideEl=document.getElementById('side');
const listEl=document.getElementById('list');
const saveBtn=document.getElementById('saveBtn');

let editingId=null;

function setStatus(msg){statusEl.innerHTML=msg;}

async function fetchMeds(){
  try{
    const {data,error}=await supabase.from('meds').select('*').order('created_at',{ascending:true});
    if(error)throw error;
    renderList(data||[]);
    setStatus('სია სინქრონიზებულია ✔️');
  }catch(e){
    console.error(e); setStatus('მონაცემთა დატვირთვა ვერ მოხერხდა ❌'); 
  }
}

function renderList(items){
  const q=(searchEl.value||'').trim().toLowerCase();
  listEl.innerHTML='';
  if(!items||items.length===0){listEl.innerHTML='<p style="text-align:center;color:#777;padding:18px">ჯერ არ არის დამატებული მედიკამენტები</p>';return;}
  items.filter(m=>{if(!q)return true;const n=(m.name||'').toLowerCase();const i=(m.indication||'').toLowerCase();return n.includes(q)||i.includes(q);})
       .forEach(m=>{
        const siteText=m.site==='More'?m.site_custom||'—':m.site||'—';
        const dilutionText=m.dilution==='More'?m.dilution_custom||'—':m.dilution||'—';
        const card=document.createElement('div');
        card.className='card';
        card.innerHTML=`
          <div class="drug-name">${escapeHtml(m.name||'უსახელო')}</div>
          <div class="meta"><b>ჩვენება:</b> ${escapeHtml(m.indication||'-')}</div>
          <div class="meta"><b>გაკეთების ადგილი:</b> ${escapeHtml(siteText)}</div>
          <div class="meta"><b>განზავება:</b> ${escapeHtml(dilutionText)}</div>
          <div class="meta"><b>მოკლე აღწერა:</b> ${escapeHtml(m.description||'-')}</div>
          <div class="meta"><b>გვერდითი ეფექტი:</b> ${escapeHtml(m.side||'-')}</div>
          <div class="actions">
            <button class="edit" onclick="startEdit(${m.id})">Edit</button>
            <button class="delete" onclick="removeMed(${m.id})">Delete</button>
          </div>
        `;
        listEl.appendChild(card);
       });
}

async function saveMedication(){
  const name=(nameEl.value||'').trim();
  if(!name){alert('გთხოვ მიუთითე მედიკამენტის სახელი');return;}
  const payload={
    name,
    indication:(indicationEl.value||'').trim(),
    site:siteEl.value||'',
    site_custom:siteEl.value==='More'?(siteCustomEl.value||'').trim():'',
    dilution:dilutionEl.value||'',
    dilution_custom:dilutionEl.value==='More'?(dilutionCustomEl.value||'').trim():'',
    description:(descriptionEl.value||'').trim(),
    side:(sideEl.value||'').trim()
  };
  saveBtn.disabled=true; saveBtn.textContent=editingId?'რთ. შენახვა...':'მიმატება...';
  try{
    if(editingId){
      const {error}=await supabase.from('meds').update(payload).eq('id',editingId);
      if(error)throw error;
      editingId=null;
    }else{
      const {error}=await supabase.from('meds').insert([payload]);
      if(error)throw error;
    }
    clearForm();
  }catch(e){console.error(e);alert('შენახვისას შეცდომა');}
  finally{saveBtn.disabled=false; saveBtn.textContent='შენახვა';}
}

async function removeMed(id){
  if(!confirm('გსურთ მართლწინვე წაშლა?'))return;
  try{const {error}=await supabase.from('meds').delete().eq('id',id);if(error)throw error;}catch(e){console.error(e);alert('წაშლისას შეცდომა');}
}

async function startEdit(id){
  try{
    const {data,error}=await supabase.from('meds').select('*').eq('id',id).single();
    if(error)throw error;
    const m=data;
    nameEl.value=m.name||''; indicationEl.value=m.indication||'';
    siteEl.value=m.site||''; siteCustomEl.style.display=m.site==='More'?'block':'none';
    siteCustomEl.value=m.site_custom||''; dilutionEl.value=m.dilution||'';
    dilutionCustomEl.style.display=m.dilution==='More'?'block':'none'; dilutionCustomEl.value=m.dilution_custom||'';
    descriptionEl.value=m.description||''; sideEl.value=m.side||''; editingId=id; saveBtn.textContent='შენახვა (რედაქტირება)'; window.scrollTo({top:0,behavior:'smooth'});
  }catch(e){console.error(e);alert('რედაქტირებისას შეცდომა');}
}

function clearForm(){nameEl.value='';indicationEl.value='';siteEl.value='';siteCustomEl.value='';siteCustomEl.style.display='none';dilutionEl.value='';dilutionCustomEl.value='';dilutionCustomEl.style.display='none';descriptionEl.value='';sideEl.value='';editingId=null;saveBtn.textContent='შენახვა';}
function escapeHtml(str){if(!str)return'';return String(str).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;').replace(/'/g,'&#039;');}

async function setupRealtime(){
  try{
    supabase.from('meds').on('INSERT',()=>fetchMeds())
    .on('UPDATE',()=>fetchMeds())
    .on('DELETE',()=>fetchMeds())
    .subscribe();
  }catch(e){console.warn(e);}
}

(async function init(){await fetchMeds(); await setupRealtime();})();

saveBtn.addEventListener('click',saveMedication);
searchEl.addEventListener('input',fetchMeds);
siteEl.addEventListener('change',()=>siteCustomEl.style.display=siteEl.value==='More'?'block':'none');
dilutionEl.addEventListener('change',()=>dilutionCustomEl.style.display=dilutionEl.value==='More'?'block':'none');
window.startEdit=startEdit; window.removeMed=removeMed;
</script>
</body>
</html>
