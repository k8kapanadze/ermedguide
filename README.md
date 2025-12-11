<html lang="ka">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>CMC Emergency – მედიკამენტები</title>

<!-- FIREBASE SCRIPTS (უნდა იყოს ყველაზე ზემოთ!) -->
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js"></script>

<script>
  const firebaseConfig = {
    apiKey: "AIzaSyCmsg-1RrZDma6xgVLGyd-VPev1tv8eVks",
    authDomain: "ermedguide.firebaseapp.com",
    projectId: "ermedguide",
    storageBucket: "ermedguide.firebasestorage.app",
    messagingSenderId: "369723574752",
    appId: "1:369723574752:web:5fd96d3eb5bf94ab1f8385",
    measurementId: "G-814WWYWSDC"
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
</script>

<style>
  body{font-family:Arial, sans-serif;background:#f5f5f5;margin:0;padding:20px}
  .container{max-width:900px;margin:auto;background:#fff;padding:20px;border-radius:10px}
  .row{display:flex;gap:10px}
  input,select,textarea{width:100%;padding:10px;margin:6px 0}
  .card{background:#fff;padding:14px;margin:12px 0;border-left:6px solid #d32f2f;border-radius:6px}
  .delete{background:#d32f2f;color:#fff}
  .edit{background:#1976d2;color:#fff}
</style>
</head>
<body>

<h1>CMC Emergency – მედიკამენტების სია</h1>

<div class="container">
  <input id="search" placeholder="ძებნა..." />

  <h2>მედიკამენტის დამატება</h2>

  <input id="name" placeholder="მედიკამენტის სახელი" />
  <input id="indication" placeholder="ჩვენება" />

  <textarea id="description" placeholder="მოკლე აღწერა"></textarea>

  <input id="side" placeholder="გვერდითი ეფექტი" />

  <button id="saveBtn">შენახვა</button>

  <h2>სია</h2>
  <div id="list"></div>
</div>

<script>
  let meds = JSON.parse(localStorage.getItem("meds") || "[]");
  let editIndex = null;

  function display() {
    const listEl = document.getElementById("list");
    const q = document.getElementById("search").value.toLowerCase();
    listEl.innerHTML = "";

    meds
      .map((m, i) => ({ ...m, _i: i }))
      .filter(m => m.name.toLowerCase().includes(q))
      .forEach(m => {
        const div = document.createElement("div");
        div.className = "card";
        div.innerHTML = `
          <div><b>${m.name}</b></div>
          <div>ჩვენება: ${m.indication}</div>
          <div>აღწერა: ${m.description}</div>
          <div>გვერდითი ეფექტი: ${m.side}</div>
          <button class="edit" onclick="editMed(${m._i})">Edit</button>
          <button class="delete" onclick="deleteMed(${m._i})">Delete</button>
        `;
        listEl.appendChild(div);
      });
  }

  function saveMedication() {
    const name = nameEl.value.trim();
    if (!name) {
      alert("შეიყვანე მედიკამენტის სახელი");
      return;
    }

    const med = {
      name,
      indication: indicationEl.value,
      description: descriptionEl.value,
      side: sideEl.value
    };

    if (editIndex === null) {
      meds.push(med);
    } else {
      meds[editIndex] = med;
      editIndex = null;
    }

    localStorage.setItem("meds", JSON.stringify(meds));
    clearForm();
    display();
  }

  function deleteMed(i) {
    meds.splice(i, 1);
    localStorage.setItem("meds", JSON.stringify(meds));
    display();
  }

  function editMed(i) {
    const m = meds[i];
    nameEl.value = m.name;
    indicationEl.value = m.indication;
    descriptionEl.value = m.description;
    sideEl.value = m.side;
    editIndex = i;
  }

  function clearForm() {
    nameEl.value = "";
    indicationEl.value = "";
    descriptionEl.value = "";
    sideEl.value = "";
    editIndex = null;
  }

  saveBtn.onclick = saveMedication;
  search.oninput = display;

  display();
</script>

</body>
</html>
