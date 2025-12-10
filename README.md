<!DOCTYPE html>
<html lang="ka">
<head>
    <meta charset="UTF-8">
    <title>CMC Emergency - მედიკამენტების სია</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f4f6f8;
            margin: 0;
            padding: 20px;
        }

        h1 {
            text-align: center;
        }

        .form-container {
            background: white;
            padding: 20px;
            max-width: 700px;
            margin: 0 auto 30px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        input, select, textarea, button {
            width: 100%;
            padding: 10px;
            margin-top: 8px;
            margin-bottom: 15px;
            border-radius: 6px;
            border: 1px solid #ccc;
            font-size: 14px;
        }

        button {
            background: #1976d2;
            color: white;
            border: none;
            cursor: pointer;
            font-size: 16px;
        }

        button:hover {
            background: #125ba1;
        }

        .cards {
            max-width: 1100px;
            margin: auto;
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 20px;
        }

        .card {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 8px rgba(0,0,0,0.1);
        }

        .card h3 {
            margin-top: 0;
            color: #1976d2;
        }

        .card p {
            margin: 6px 0;
        }

        .delete-btn {
            background: crimson;
            margin-top: 10px;
        }
    </style>
</head>
<body>

<h1>CMC Emergency - მედიკამენტების ბაზა</h1>

<div class="form-container">
    <h2>მედიკამენტის დამატება</h2>

    <input id="name" placeholder="მედიკამენტის დასახელება">

    <textarea id="indication" placeholder="ჩვენება"></textarea>

    <select id="route">
        <option value="">გაკეთების ადგილი</option>
        <option>IV</option>
        <option>IM</option>
        <option>subQ</option>
        <option>oral</option>
        <option>inhalation</option>
    </select>

    <select id="dilution">
        <option value="">განზავება</option>
        <option>no</option>
        <option>5 ml</option>
        <option>10 ml</option>
        <option>20 ml</option>
        <option>more</option>
    </select>

    <textarea id="sideEffects" placeholder="გვერდითი ეფექტი"></textarea>

    <button onclick="addMedicine()">დამატება</button>
</div>

<div class="cards" id="cards"></div>

<script>
    let medicines = JSON.parse(localStorage.getItem("medicines")) || [];

    function addMedicine() {
        const name = document.getElementById("name").value;
        const indication = document.getElementById("indication").value;
        const route = document.getElementById("route").value;
        const dilution = document.getElementById("dilution").value;
        const sideEffects = document.getElementById("sideEffects").value;

        if (!name || !indication || !route || !dilution || !sideEffects) {
            alert("შეავსე ყველა ველი!");
            return;
        }

        const medicine = {
            id: Date.now(),
            name,
            indication,
            route,
            dilution,
            sideEffects
        };

        medicines.push(medicine);
        localStorage.setItem("medicines", JSON.stringify(medicines));
        renderMedicines();
        clearForm();
    }

    function renderMedicines() {
        const cards = document.getElementById("cards");
        cards.innerHTML = "";

        medicines.forEach(med => {
            const div = document.createElement("div");
            div.className = "card";

            div.innerHTML = `
                <h3>${med.name}</h3>
                <p><strong>ჩვენება:</strong> ${med.indication}</p>
                <p><strong>გაკეთების ადგილი:</strong> ${med.route}</p>
                <p><strong>განზავება:</strong> ${med.dilution}</p>
                <p><strong>გვერდითი ეფექტი:</strong> ${med.sideEffects}</p>
                <button class="delete-btn" onclick="deleteMedicine(${med.id})">წაშლა</button>
            `;

            cards.appendChild(div);
        });
    }

    function deleteMedicine(id) {
        medicines = medicines.filter(m => m.id !== id);
        localStorage.setItem("medicines", JSON.stringify(medicines));
        renderMedicines();
    }

    function clearForm() {
        document.getElementById("name").value = "";
        document.getElementById("indication").value = "";
        document.getElementById("route").value = "";
        document.getElementById("dilution").value = "";
        document.getElementById("sideEffects").value = "";
    }

    renderMedicines();
</script>

</body>
</html>
