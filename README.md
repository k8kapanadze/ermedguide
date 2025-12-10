<!DOCTYPE html>
<html lang="ka">
<head>
    <meta charset="UTF-8">
    <title>CMC Emergency მედიკამენტები</title>
    <style>
        body {
            font-family: sans-serif;
            color: black;
        }
        .med {
            border: 1px solid #ccc;
            padding: 10px;
            margin: 10px 0;
            background-color: #f9f9f9;
        }
        .med h3 {
            color: maroon; /* შინდისფერი (burgundy/maroon) */
            margin-top: 0;
        }
        input, textarea, select {
            width: 100%;
            max-width: 400px;
            margin-bottom: 10px;
        }
        button {
            padding: 10px;
            background-color: #007bff;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>
    <h1>CMC Emergency განყოფილება - მედიკამენტების სია</h1>
    
    <input type="text" id="search" placeholder="ძებნა ჩვენებით ან წამლის სახელწოდებით" oninput="searchMeds()">
    
    <div id="medList"></div>
    
    <h2>ახალი მედიკამენტის დამატება</h2>
    <form id="addForm">
        <label for="name">წამლის სახელი:</label><br>
        <input type="text" id="name" required><br>
        
        <label for="indication">ჩვენება:</label><br>
        <textarea id="indication" required></textarea><br>
        
        <label for="route">გაკეთების ადგილი:</label><br>
        <select id="route" required>
            <option value="IV">IV</option>
            <option value="IM">IM</option>
            <option value="subQ">subQ</option>
            <option value="oral">oral</option>
            <option value="inhalation">inhalation</option>
        </select><br>
        
        <label for="dilution">განზავება:</label><br>
        <select id="dilution" required>
            <option value="no">no</option>
            <option value="5ml">5ml</option>
            <option value="10ml">10ml</option>
            <option value="20ml">20ml</option>
            <option value="more">more</option>
        </select><br>
        
        <label for="sideEffects">გვერდითი ეფექტი:</label><br>
        <textarea id="sideEffects" required></textarea><br>
        
        <button type="button" onclick="addMed()">დამატება</button>
    </form>
    
    <script>
        let meds = JSON.parse(localStorage.getItem('meds')) || [];
        
        function renderMeds(filter = '') {
            const list = document.getElementById('medList');
            list.innerHTML = '';
            const filteredMeds = meds.filter(med => 
                med.name.toLowerCase().includes(filter.toLowerCase()) || 
                med.indication.toLowerCase().includes(filter.toLowerCase())
            );
            filteredMeds.forEach(med => {
                const div = document.createElement('div');
                div.classList.add('med');
                div.innerHTML = `
                    <h3>${med.name}</h3>
                    <p><strong>ჩვენება:</strong> ${med.indication}</p>
                    <p><strong>გაკეთების ადგილი:</strong> ${med.route}</p>
                    <p><strong>განზავება:</strong> ${med.dilution}</p>
                    <p><strong>გვერდითი ეფექტი:</strong> ${med.sideEffects}</p>
                `;
                list.appendChild(div);
            });
        }
        
        function addMed() {
            const name = document.getElementById('name').value;
            const indication = document.getElementById('indication').value;
            const route = document.getElementById('route').value;
            const dilution = document.getElementById('dilution').value;
            const sideEffects = document.getElementById('sideEffects').value;
            
            if (name && indication && route && dilution && sideEffects) {
                meds.push({ name, indication, route, dilution, sideEffects });
                localStorage.setItem('meds', JSON.stringify(meds));
                renderMeds();
                document.getElementById('addForm').reset();
            } else {
                alert('გთხოვთ, შეავსოთ ყველა ველი!');
            }
        }
        
        function searchMeds() {
            const filter = document.getElementById('search').value;
            renderMeds(filter);
        }
        
        window.onload = () => renderMeds();
    </script>
</body>
</html>
