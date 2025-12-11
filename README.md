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
  textarea{min-height:70px;resize:vertical}
  button{padding:10px 14px;border:none;border-radius:6px;background:#111;color:#fff;cursor:pointer;font-size:15px}
  .row{display:flex;gap:10px}
  .row .col{flex:1}
  #search{padding:12px;font-size:16px;margin-bottom:12px}
  .card{background:#fff;padding:14px;margin:12px 0;border-left:6px solid #d32f2f;border-radius:6px;box-shadow:0 2px 8px rgba(0,0,0,.06)}
  .drug-name{color:#d32f2f;font-weight:bold;font-size:1.1em}
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
        <input id="dilutionCustom" placeholder="ჩაწერე custom განზავება" style="display:none;margin-top:6px" />
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

<script>
  // =================================================================
  // 1. FIREBASE CONFIGURATION AND INITIALIZATION
  // =================================================================

  // For Firebase JS SDK v7.20.0 and later, measurementId is optional
  const firebaseConfig = {
    apiKey: "AIzaSyCmsg-1RrZDma6xgVLGyd-VPev1tv8eVks",
    authDomain: "ermedguide.firebaseapp.com",
    projectId: "ermedguide",
    storageBucket: "ermedguide.firebasestorage.app",
    messagingSenderId: "369723574752",
    appId: "1:369723574752:web:5fd96d3eb5bf94ab1f8385",
    measurementId: "G-814WWYWSDC"
  };

  // Firebase SDK-ების ჩართვა (v8.10.0-ის CDN ვერსიები)
  // რადგან HTML-ში პირდაპირ ჩასმა გსურთ, გამოიყენება ძველი, მაგრამ მარტივი ვერსია.
  // ლოგიკურად, ეს სკრიპტები უნდა იყოს აქ:
  document.write('<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>');
  document.write('<script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>');

  // ინიციალიზაცია
  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
  const MEDICATION_COLLECTION = 'medications'; // კოლექციის სახელი Firestore-ში

  // =================================================================
  // 2. GLOBAL VARIABLES AND ELEMENTS
  // =================================================================

  let medications = []; // შეინახავს მონაცემებს Firestore-დან
  let editId = null; // Firestore Document ID რედაქტირებისთვის

  const searchEl = document.getElementById('search');
  const nameEl = document.getElementById('name');
  const indicationEl = document.getElementById('indication');
  const siteEl = document.getElementById('site');
  const siteCustomEl = document.getElementById('siteCustom');
  const dilutionEl = document.getElementById('dilution');
  const dilutionCustomEl = document.getElementById('dilutionCustom');
  const descriptionEl = document.getElementById('description');
  const sideEl = document.getElementById('side');
  const listEl = document.getElementById('list');
  const saveBtn = document.getElementById('saveBtn');

  // =================================================================
  // 3. CORE FUNCTIONS (CRUD using Firestore)
  // =================================================================

  function display(medsList){
    const q = searchEl.value.trim().toLowerCase();
    listEl.innerHTML = '';

    if(medsList.length === 0){
      listEl.innerHTML = '<p style="text-align:center;color:#777;padding:18px">ჯერ არ არის დამატებული მედიკამენტები</p>';
      return;
    }

    medsList
      .filter(m => {
        if(!q) return true;
        return (
          (m.name && m.name.toLowerCase().includes(q)) ||
          (m.indication && m.indication.toLowerCase().includes(q))
        );
      })
      .forEach(m => {
        const siteText = (m.site === 'More' ? (m.siteCustom || '—') : (m.site || '—'));
        const dilutionText = (m.dilution === 'More' ? (m.dilutionCustom || '—') : (m.dilution || '—'));

        const card = document.createElement('div');
        card.className = 'card';
        card.innerHTML = `
          <div class="drug-name">${escapeHtml(m.name || 'უსახელო')}</div>
          <div><b>ჩვენება:</b> ${escapeHtml(m.indication || '-')}</div>
          <div><b>გაკეთების ადგილი:</b> ${escapeHtml(siteText)}</div>
          <div><b>განზავება:</b> ${escapeHtml(dilutionText)}</div>
          <div><b>მოკლე აღწერა:</b> ${escapeHtml(m.description || '-')}</div>
          <div><b>გვერდითი ეფექტი:</b> ${escapeHtml(m.side || '-')}</div>
          <div class="actions">
            <button class="edit" onclick="editMed('${m.id}')">Edit</button>
            <button class="delete" onclick="deleteMed('${m.id}')">Delete</button>
          </div>
        `;
        listEl.appendChild(card);
      });
  }

  async function saveMedication(){
    const name = nameEl.value.trim();
    if(!name){
      alert('გთხოვ, მიუთითე მედიკამენტის სახელი.');
      return;
    }

    const medData = {
      name,
      indication: indicationEl.value.trim(),
      site: siteEl.value,
      siteCustom: siteEl.value === 'More' ? siteCustomEl.value.trim() : '',
      dilution: dilutionEl.value,
      dilutionCustom: dilutionEl.value === 'More' ? dilutionCustomEl.value.trim() : '',
      description: descriptionEl.value.trim(),
      side: sideEl.value.trim(),
      // სორტირებისთვის დამატება:
      updatedAt: firebase.firestore.FieldValue.serverTimestamp() 
    };

    try {
      if(editId === null){
        // ADD - ახალი ჩანაწერის დამატება
        await db.collection(MEDICATION_COLLECTION).add(medData);
        alert('მედიკამენტი წარმატებით დაემატა!');
      } else {
        // UPDATE - არსებულის რედაქტირება
        await db.collection(MEDICATION_COLLECTION).doc(editId).set(medData, { merge: true });
        alert('მედიკამენტი წარმატებით განახლდა!');
      }
    } catch (e) {
      console.error("Error saving document: ", e);
      alert('შეცდომა მონაცემების შენახვისას.');
    }

    clearForm();
  }

  async function deleteMed(id){
    if(confirm('გსურთ მართლწინვე წაშლა?')){
      try {
        await db.collection(MEDICATION_COLLECTION).doc(id).delete();
        alert('მედიკამენტი წარმატებით წაიშალა!');
      } catch (e) {
        console.error("Error removing document: ", e);
        alert('შეცდომა წაშლისას.');
      }
      if(editId === id){
        editId = null;
        clearForm();
      }
    }
  }

  function editMed(id){
    // მოძებნე მედიკამენტი ადგილობრივ სიაში (რომელიც onSnapshot-მა ჩატვირთა)
    const m = medications.find(med => med.id === id);
    if (!m) return;

    nameEl.value = m.name || '';
    indicationEl.value = m.indication || '';
    siteEl.value = m.site || '';
    siteCustomEl.style.display = m.site === 'More' ? 'block' : 'none';
    siteCustomEl.value = m.siteCustom || '';
    dilutionEl.value = m.dilution || '';
    dilutionCustomEl.style.display = m.dilution === 'More' ? 'block' : 'none';
    dilutionCustomEl.value = m.dilutionCustom || '';
    descriptionEl.value = m.description || '';
    sideEl.value = m.side || '';
    
    editId = id; // დააყენე document ID რედაქტირებისთვის
    saveBtn.textContent = 'შენახვა (რედაქტირება)';
    window.scrollTo({top:0,behavior:'smooth'});
  }

  function clearForm(){
    nameEl.value = '';
    indicationEl.value = '';
    siteEl.value = '';
    siteCustomEl.value = '';
    siteCustomEl.style.display = 'none';
    dilutionEl.value = '';
    dilutionCustomEl.value = '';
    dilutionCustomEl.style.display = 'none';
    descriptionEl.value = '';
    sideEl.value = '';
    editId = null;
    saveBtn.textContent = 'შენახვა';
  }

  function escapeHtml(str){
    if(!str) return '';
    return String(str)
      .replace(/&/g,'&amp;')
      .replace(/</g,'&lt;')
      .replace(/>/g,'&gt;')
      .replace(/"/g,'&quot;')
      .replace(/'/g,'&#039;');
  }
  
  // =================================================================
  // 4. REAL-TIME LISTENER (onSnapshot)
  // =================================================================

  // ეს არის ყველაზე მნიშვნელოვანი ნაწილი: ის ავტომატურად განაახლებს
  // 'medications' მასივს და იძახებს 'display' ფუნქციას ნებისმიერი ცვლილებისას.
  db.collection(MEDICATION_COLLECTION)
    .orderBy('updatedAt', 'desc') // დაალაგებს ბოლოს განახლებულის მიხედვით
    .onSnapshot(snapshot => {
      medications = snapshot.docs.map(doc => ({
        id: doc.id, // ვინახავთ Firestore-ის Document ID-ს რედაქტირების/წაშლისთვის
        ...doc.data()
      }));
      display(medications); // განაახლე ინტერფეისი
    }, error => {
      console.error("Error getting real-time documents: ", error);
      listEl.innerHTML = `<p style="text-align:center;color:red;padding:18px">შეცდომა Firebase-თან კავშირისას: ${error.message}</p>`;
    });


  // =================================================================
  // 5. EVENT LISTENERS
  // =================================================================

  saveBtn.addEventListener('click', saveMedication);
  searchEl.addEventListener('input', () => display(medications)); // ძებნა იფილტრება ადგილობრივად
  siteEl.addEventListener('change', ()=> siteCustomEl.style.display = siteEl.value==='More'?'block':'none');
  dilutionEl.addEventListener('change', ()=> dilutionCustomEl.style.display = dilutionEl.value==='More'?'block':'none');

  // გლობალური ფუნქციები (საჭიროა HTML-ში onclick-ისთვის)
  window.editMed = editMed;
  window.deleteMed = deleteMed;

</script>
</body>
</html>
