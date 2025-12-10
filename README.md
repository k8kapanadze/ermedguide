<!DOCTYPE html>
<html lang="ka">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CMC Emergency – მედიკამენტები</title>

<style>
body{
    font-family: Arial,sans-serif;
    background:#f5f5f5;
    margin:0;
    padding:20px;
}
h1{text-align:center;}
.container{
    max-width:800px;
    margin:auto;
    background:white;
    padding:20px;
    border-radius:10px;
    box-shadow:0 0 15px rgba(0,0,0,0.1);
}
input,select{
    width:100%;
    padding:10px;
    margin:6px 0;
    border-radius:5px;
    border:1px solid #ccc;
}
button{
    width:100%;
    padding:12px;
    margin-top:10px;
    background:black;
    color:white;
    border:none;
    border-radius:5px;
    cursor:pointer;
}
.card{
    background:white;
    padding:15px;
    margin:12px 0;
    border-left:6px solid #d32f2f;
    box-shadow:0 2px 8px rgba(0,0,0,0.1);
}
.card button{
    width:auto;
    margin:5px 5px 0 0;
    padding:6px 12px;
    font-size:14px;
}
.delete{background:#d32f2f;}
.edit{background:#1976d2;}
</style>
</head>

<body>

<h1>CMC Emergency – მედიკამენტების სია</h1>

<div class="container">

<input type="text" id="search" placeholder="ძებნა სახელით ან ჩვენებით...">

<h2>მედიკამენტის დამატება / რედაქტირება</h2>

<input id="name" placeholder="მედიკამენტის სახელი">
<input id="indication" placeholder="ჩვენება">

<select id="site">
    <option value="">გაკეთების ადგილი</option>
    <option>IV</option>
    <option>IM</option>
    <option>SubQ</option>
    <option>Oral</option>
    <option>Inhalation</option>
</select>

<select id="dilution">
    <option value="">განზავება</option>
    <option>No</option>
    <option>5 ml</option>
    <option>10 ml</option>
    <option>20 ml</option>
    <option>More</option>
</select>

<input id="side" placeholder="გვერდითი ეფექტი">

<button onclick="saveMedication()">შენახვა</button>

<h2>სრული სია</h2>
<div id="list"></div>

</div>

<script>
let meds = JSON.parse(localStorage.getItem("meds")) || [];
let editIndex = null;

function display(){
    const list = document.getElementById("list");
    const s = document.getElementById("search").value.toLowerCase();
    list.innerHTML = "";

    meds
    .filter(m =>
        m.name.toLowerCase().includes(s) ||
        m.indication.toLowerCase().includes(s)
    )
    .forEach((m,i)=>{
        list.innerHTML += `
        <div class="card">
            <b>${m.name}</b><br><br>
            <b>ჩვენება:</b> ${m.indication}<br>
            <b>გაკეთების ადგილი:</b> ${m.site}<br>
            <b>განზავება:</b> ${m.dilution}<br>
            <b>გვერდითი ეფექტი:</b> ${m.side}<br><br>
            <button class="edit" onclick="editMed(${i})">Edit</button>
            <button class="delete" onclick="deleteMed(${i})">Delete</button>
        </div>`;
    });
}

function saveMedication(){
    const med = {
        name: name.value.trim(),
        indication: indication.value.trim(),
        site: site.value,
        dilution: dilution.value,
        side: side.value.trim()
    };

    if(!med.name){
        alert("მედიკამენტის სახელი აუცილებელია");
        return;
    }

    if(editIndex === null){
        meds.push(med);
    } else {
        meds[editIndex] = med;
        editIndex = null;
    }

    localStorage.setItem("meds", JSON.stringify(meds));
    clearForm();
    display();
}

function deleteMed(index){
    if(confirm("გსურს წაშლა?")){
        meds.splice(index,1);
        localStorage.setItem("meds", JSON.stringify(meds));
        display();
    }
}

function editMed(index){
    const m = meds[index];
    name.value = m.name;
    indication.value = m.indication;
    site.value = m.site;
    dilution.value = m.dilution;
    side.value = m.side;
    editIndex = index;
}

function clearForm(){
    name.value="";
    indication.value="";
    site.value="";
    dilution.value="";
    side.value="";
}

search.addEventListener("input",display);
display();
</script>

</body>
</html>
