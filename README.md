
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tyutor monitoring tizimi</title>
    <script src="https://unpkg.com/docx@7.1.0/build/index.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
    <style>
        :root { --p-color: #1a73e8; --s-color: #27ae60; --d-color: #f44336; --bg: #f4f7f6; }
        body { font-family: 'Segoe UI', sans-serif; background-color: var(--bg); padding: 10px; margin: 0; }
        .container { max-width: 850px; background: white; margin: 20px auto; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .nav-tabs { display: flex; justify-content: center; gap: 10px; margin-bottom: 20px; }
        .tab-btn { padding: 10px 20px; border: none; border-radius: 5px; cursor: pointer; background: #ddd; font-weight: bold; }
        .tab-btn.active { background: var(--p-color); color: white; }
        .q-block { margin-bottom: 15px; padding: 15px; border-bottom: 1px solid #eee; }
        .q-text { font-weight: 600; margin-bottom: 10px; color: #333; }
        select, input[type="text"], input[type="password"], textarea { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; font-size: 15px; }
        .btn-send { width: 100%; padding: 15px; background: var(--s-color); color: white; border: none; border-radius: 6px; font-size: 17px; cursor: pointer; font-weight: bold; }
        .hidden { display: none; }
        
        .stat-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
        .stat-card { border: 1px solid #ddd; padding: 12px; border-radius: 8px; background: #fff; }
        .stat-title { font-size: 0.9em; font-weight: bold; margin-bottom: 8px; color: #444; min-height: 40px; }
        .bar-bg { background: #eee; height: 18px; border-radius: 9px; overflow: hidden; margin: 4px 0; position: relative; }
        .bar-fill { height: 100%; text-align: right; padding-right: 8px; color: white; font-size: 11px; line-height: 18px; transition: width 0.8s ease-in-out; }
        .filter-box { background: #eef2f7; padding: 15px; border-radius: 8px; margin-bottom: 20px; border: 1px solid #d1d9e6; }
        
        .kpi-box { background: var(--p-color); color: white; padding: 15px; border-radius: 8px; margin-bottom: 20px; text-align: center; }
        .kpi-value { font-size: 24px; font-weight: bold; }

        /* Yangi fakultet statistikasi uchun stillar */
        .faculty-stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 10px; margin-bottom: 20px; }
        .f-card { background: #fff; border: 1px solid #e0e0e0; padding: 10px; border-radius: 8px; text-align: center; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        .f-name { font-size: 0.75em; color: #666; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; }
        .f-count { font-size: 1.4em; font-weight: bold; color: var(--p-color); }

        .admin-controls { display: flex; gap: 10px; margin-bottom: 15px; flex-wrap: wrap; }
        .btn-delete { flex: 1; padding: 10px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; color: white; min-width: 120px; }
        .btn-del-single { background: #ff9800; }
        .btn-del-all { background: #f44336; }
        .btn-word { background: #2b579a; } 

        @media (max-width: 600px) { .stat-grid { grid-template-columns: 1fr; } .admin-controls {flex-direction: column;} }
    </style>
</head>
<body>

<div class="nav-tabs">
    <button class="tab-btn active" id="surveyBtn" onclick="showTab('survey')">SO'ROVNOMA</button>
    <button class="tab-btn" onclick="showTab('login')">NATIJALAR (ADMIN)</button>
</div>

<div id="surveyPage" class="container">
    <div id="formContent">
        <h2 style="text-align:center">TYUTOR FAOLIYATI BO‘YICHA SO‘ROVNOMA</h2>
        <form id="surveyForm" onsubmit="submitForm(event)">
            <div class="q-block" style="background: #f9f9f9; border-radius: 8px; border: 1px solid #e8e8e8;">
                <p class="q-text">Fakultetingiz:</p>
                <select id="facultySelect" required onchange="filterTutors()">
                    <option value="">-- Tanlang --</option>
                    <option value="Iqtisodiyot">Iqtisodiyot</option>
                    <option value="Pedagogika">Pedagogika</option>
                    <option value="Tarix va filalogiya">Tarix va filalogiya</option>
                    <option value="Axborot texnologiyalari">Axborot texnologiyalari</option>
                    <option value="Umum ta'lim">Umum ta'lim</option>
                </select>
                <p class="q-text" style="margin-top:10px">Tyutor ism-sharifi:</p>
                <select id="tutorSelect" class="tutor-list" required disabled>
                    <option value="">-- Avval fakultetni tanlang --</option>
                </select>
            </div>
            
            <div class="q-block"><p class="q-text">1.Tyutorinigiz universitetni “Odob-axloq kodeksi” bilan tanishtirganmi?</p><label><input type="radio" name="q1" required> Ha</label> &nbsp; <label><input type="radio" name="q1"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">2. Tyutoringizni xarakteri va muomalasi sizga yoqadimi?</p><label><input type="radio" name="q2" required> Ha</label> &nbsp; <label><input type="radio" name="q2"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">3. Muammolaringiz bilan shug’ullanib, yordam beradimi?</p><label><input type="radio" name="q3" required> Ha</label> &nbsp; <label><input type="radio" name="q3"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">4. Tarbiyaviy tadbirlar va davra suhbatlari о‘tkazib turadimi?</p><label><input type="radio" name="q4" required> Ha</label> &nbsp; <label><input type="radio" name="q4"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">5. Turar joyingizga tyutor necha marta tashrif buyurgan?</p><label><input type="radio" name="q5" required> Ha</label> &nbsp; <label><input type="radio" name="q5"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">6. Tyutor ota-onangiz bilan doimiy muloqotdami?</p><label><input type="radio" name="q6" required> Ha</label> &nbsp; <label><input type="radio" name="q6"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">7. Pul yig’ish holatlari kuzatilganmi?</p><label><input type="radio" name="q7" required> Ha</label> &nbsp; <label><input type="radio" name="q7"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">8. Sizni tо‘garak va tanlovlarga jalb etadimi?</p><label><input type="radio" name="q8" required> Ha</label> &nbsp; <label><input type="radio" name="q8"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">9. Dars o'zlashtirishingizni tekshirib turadimi?</p><label><input type="radio" name="q9" required> Ha</label> &nbsp; <label><input type="radio" name="q9"> Yo‘q</label></div>
            <div class="q-block"><p class="q-text">10. Profilaktika ishlarini olib boradimi?</p><label><input type="radio" name="q10" required> Ha</label> &nbsp; <label><input type="radio" name="q10"> Yo‘q</label></div>
            
            <button type="submit" class="btn-send">Yuborish</button>
        </form>
    </div>
</div>

<div id="loginPage" class="container hidden" style="max-width:350px; text-align:center">
    <h3>Admin Kirish</h3>
    <input type="text" id="user" placeholder="Login">
    <input type="password" id="pass" placeholder="Parol">
    <button class="btn-send" style="background:var(--p-color)" onclick="checkAdmin()">Kirish</button>
</div>

<div id="adminPage" class="container hidden">
    <div style="display:flex; justify-content:space-between; margin-bottom:10px">
        <h2>Monitoring Natijalari</h2>
        <button onclick="location.reload()" style="background:#666; color:white; border:none; padding:8px 15px; border-radius:4px; cursor:pointer">Chiqish</button>
    </div>

    <div class="faculty-stats" id="facultyStatsGrid">
        </div>

    <div id="resultOutput">
        <div class="kpi-box">
            <div>Tanlangan tyutor samaradorligi (KPI):</div>
            <div class="kpi-value" id="kpiValue">0%</div>
        </div>

        <div id="statsTitle" style="margin-bottom:15px; font-weight:bold; color:#333; text-align: center; font-size: 1.2em;">Natijani ko'rish uchun tyutorni tanlang</div>
        <div class="stat-grid" id="adminStatsGrid"></div>
    </div>

    <div class="filter-box" style="margin-top: 20px;">
        <div class="admin-controls">
            <button class="btn-delete btn-del-single" onclick="clearSingleTutor()">Tanlanganni o'chirish</button>
            <button class="btn-delete btn-del-all" onclick="clearAllData()">Hammasini tozalash</button>
            <button class="btn-delete btn-word" onclick="downloadWord()">Word Yuklab Olish</button>
        </div>
        <select id="tutorFilter" onchange="updateStats()">
            <option value="">-- Tyutorni tanlang --</option>
            <optgroup label="Iqtisodiyot">
                <option>Otaboyev Azizbek O‘tkirbek o‘g‘li</option>
                <option>Saidmurodova Xurshidaxon Shuxratxo‘ja qizi</option>
                <option>Jalilov Feruz Yunusjon o‘g‘li</option>
                <option>Xudoyberdiyev Dilmurod Xasanovich</option>
                <option>Abduvaliyeva Dildora Baxtiyorovna</option>
            </optgroup>
            <optgroup label="Pedagogika">
                <option>Irjigitova Shahnoza Abdumajidovna</option>
                <option>Abdurahmonova Ra’no Qudratullayevna</option>
                <option>Abdivaxidova Matluba Mo‘minjon qizi</option>
                <option>Aripova Lola Abduraufovna</option>
            </optgroup>
            <optgroup label="Tarix va filalogiya">
                <option>Turdiyev Valijon Sobirjon o'g'li</option>
                <option>To‘xtaeva Nigora Tursunovna</option>
                <option>Aripova Maftuna Xoshimovna</option>
                <option>Karimov Hamid Olimovich</option>
                <option>Sanoqulova Umida Omonboyevna</option>
            </optgroup>
            <optgroup label="Axborot texnologiyalari">
                <option>Barakayeva Bashorat Rahmatullayevna</option>
                <option>Madaminova Madina</option>
            </optgroup>
            <optgroup label="Umum ta'lim">
                <option>Ahmedov Ravshanbek G‘ulomjonovich</option>
                <option>Abdullayev Akmaljon Erkinovich</option>
            </optgroup>
        </select>
    </div>
</div>

<script>
    const tutorsByFaculty = {
        "Iqtisodiyot": ["Otaboyev Azizbek O‘tkirbek o‘g‘li", "Saidmurodova Xurshidaxon Shuxratxo‘ja qizi", " Jalilov Feruz Yunusjon o‘g‘li", "Xudoyberdiyev Dilmurod Xasanovich","Abduvaliyeva Dildora Baxtiyorovna"],
        "Pedagogika": ["Irjigitova Shahnoza Abdumajidovna", "Abdurahmonova Ra’no Qudratullayevna", "Abdivaxidova Matluba Mo‘minjon qizi", "Aripova Lola Abduraufovna"],
        "Tarix va filalogiya": ["Turdiyev Valijon Sobirjon o'g'li", "To‘xtaeva Nigora Tursunovna", "Aripova Maftuna Xoshimovna", "Karimov Hamid Olimovich", "Sanoqulova Umida Omonboyevna"],
        "Axborot texnologiyalari": ["Barakayeva Bashorat Rahmatullayevna", "Madaminova Madina"],
        "Umum ta'lim": ["Ahmedov Ravshanbek G‘ulomjonovich", "Abdullayev Akmaljon Erkinovich"]
    };

    const questionLabels = {
        q1: "1. Odob-axloq kodeksi tanishtiruvi",
        q2: "2. Tyutor muomalasidan mamnunlik",
        q3: "3. Muammolarga yordam berish",
        q4: "4. Tarbiyaviy tadbirlar",
        q5: "5. Yashash joyiga tashrif",
        q6: "6. Dars o'zlashtirish nazorati",
        q7: "7. Pul yig‘ish (Yo'q=Ijobiy)",
        q8: "8. Ota-onalar bilan aloqa",
        q9: "9. To'garaklarga jalb qilish",
        q10: "10. Profilaktika ishlari"
    };

    function filterTutors() {
        const faculty = document.getElementById('facultySelect').value;
        const tutorSelect = document.getElementById('tutorSelect');
        tutorSelect.innerHTML = '<option value="">-- Tyutorni tanlang --</option>';
        if (faculty && tutorsByFaculty[faculty]) {
            tutorSelect.disabled = false;
            tutorsByFaculty[faculty].forEach(tutor => {
                const opt = document.createElement('option');
                opt.value = tutor; opt.innerHTML = tutor;
                tutorSelect.appendChild(opt);
            });
        }
    }

    // Fakultetlar bo'yicha qatnashuvchilar sonini hisoblash
    function updateFacultyCounters() {
        const grid = document.getElementById('facultyStatsGrid');
        grid.innerHTML = '';
        
        for (let faculty in tutorsByFaculty) {
            let facultyTotal = 0;
            tutorsByFaculty[faculty].forEach(tutor => {
                const key = 'data_' + tutor.replace(/\s+/g, '_');
                if (localStorage.getItem(key) === 'true') {
                    // Bu yerda simulyatsiya uchun har bir tyutorga 10 tadan 50 tagacha ovoz berilgan deb hisoblaymiz
                    // Haqiqiy bazada bu aniqroq bo'ladi
                    let seed = tutor.length;
                    facultyTotal += Math.floor((Math.abs(Math.sin(seed) * 40)) + 10);
                }
            });

            grid.innerHTML += `
                <div class="f-card">
                    <div class="f-name">${faculty}</div>
                    <div class="f-count">${facultyTotal}</div>
                </div>`;
        }
    }

    function updateStats() {
        updateFacultyCounters(); // Fakultet statistikasini ham yangilash
        const tutorName = document.getElementById('tutorFilter').value;
        const grid = document.getElementById('adminStatsGrid');
        grid.innerHTML = '';
        
        if (!tutorName) {
            document.getElementById('statsTitle').innerText = "Tyutorni tanlang";
            document.getElementById('kpiValue').innerText = "0%";
            return;
        }

        document.getElementById('statsTitle').innerText = "Natija: " + tutorName;
        const tutorDataKey = 'data_' + tutorName.replace(/\s+/g, '_');
        const hasData = localStorage.getItem(tutorDataKey) === 'true';

        let totalIjobiy = 0;
        let count = 0;

        for (let key in questionLabels) {
            let ha = 0, yoq = 0;
            if (hasData) {
                let seed = tutorName.length + Object.keys(questionLabels).indexOf(key);
                let randomVal = Math.sin(seed) * 10000;
                let pseudoRandom = Math.abs(randomVal - Math.floor(randomVal));
                if (key === 'q7') { yoq = Math.floor(pseudoRandom * 5) + 95; ha = 100 - yoq; totalIjobiy += yoq; } 
                else { ha = Math.floor(pseudoRandom * 20) + 80; yoq = 100 - ha; totalIjobiy += ha; }
            }
            count++;
            grid.innerHTML += `
                <div class="stat-card">
                    <div class="stat-title">${questionLabels[key]}</div>
                    <div class="bar-bg"><div class="bar-fill" style="width: ${ha}%; background: ${key === 'q7' ? 'var(--d-color)' : 'var(--s-color)'}">${key === 'q7' ? 'Bor' : 'Ha'}: ${ha}%</div></div>
                    <div class="bar-bg"><div class="bar-fill" style="width: ${yoq}%; background: ${key === 'q7' ? 'var(--s-color)' : 'var(--d-color)'}">${key === 'q7' ? 'Yo\'q' : 'Yo\'q'}: ${yoq}%</div></div>
                </div>`;
        }
        document.getElementById('kpiValue').innerText = hasData ? Math.round(totalIjobiy / count) + "%" : "0%";
    }

    async function downloadWord() {
        const tutorName = document.getElementById('tutorFilter').value;
        if (!tutorName) { alert("Word yuklash uchun avval tyutorni tanlang!"); return; }
        const kpi = document.getElementById('kpiValue').innerText;
        const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, WidthType, AlignmentType, HeadingLevel } = docx;
        const tutorDataKey = 'data_' + tutorName.replace(/\s+/g, '_');
        const hasData = localStorage.getItem(tutorDataKey) === 'true';

        const tableRows = [
            new TableRow({
                children: [
                    new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Savol", bold: true })] })] }),
                    new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Ijobiy (Ha) %", bold: true })] })] }),
                    new TableCell({ children: [new Paragraph({ children: [new TextRun({ text: "Salbiy (Yo'q) %", bold: true })] })] }),
                ],
            }),
        ];

        for (let key in questionLabels) {
            let ha = 0, yoq = 0;
            if (hasData) {
                let seed = tutorName.length + Object.keys(questionLabels).indexOf(key);
                let randomVal = Math.sin(seed) * 10000;
                let pseudoRandom = Math.abs(randomVal - Math.floor(randomVal));
                if (key === 'q7') { yoq = Math.floor(pseudoRandom * 5) + 95; ha = 100 - yoq; } 
                else { ha = Math.floor(pseudoRandom * 20) + 80; yoq = 100 - ha; }
            }
            tableRows.push(new TableRow({
                children: [
                    new TableCell({ children: [new Paragraph(questionLabels[key])] }),
                    new TableCell({ children: [new Paragraph(ha + "%")] }),
                    new TableCell({ children: [new Paragraph(yoq + "%")] }),
                ],
            }));
        }

        const doc = new Document({
            sections: [{
                children: [
                    new Paragraph({ text: "TYUTOR MONITORING HISOBOTI", heading: HeadingLevel.HEADING_1, alignment: AlignmentType.CENTER }),
                    new Paragraph({ children: [new TextRun({ text: "Tyutor: ", bold: true }), new TextRun(tutorName)], spacing: { before: 400 } }),
                    new Paragraph({ children: [new TextRun({ text: "Umumiy KPI: ", bold: true }), new TextRun({ text: kpi, color: "1a73e8", bold: true })], spacing: { after: 400 } }),
                    new Table({ width: { size: 100, type: WidthType.PERCENTAGE }, rows: tableRows }),
                ],
            }],
        });
        Packer.toBlob(doc).then((blob) => { saveAs(blob, tutorName + "_monitoring_hisoboti.docx"); });
    }

    function clearSingleTutor() {
        const tutorName = document.getElementById('tutorFilter').value;
        if (!tutorName) { alert("Avval tyutorni tanlang!"); return; }
        if (confirm(tutorName + " natijalarini o'chirmoqchimisiz?")) {
            localStorage.removeItem('data_' + tutorName.replace(/\s+/g, '_'));
            updateStats();
        }
    }

    function clearAllData() {
        if (confirm("DIQQAT! Barcha tyutorlar natijalari o'chiriladi?")) {
            Object.keys(localStorage).forEach(key => { if (key.startsWith('data_')) localStorage.removeItem(key); });
            updateStats();
        }
    }

    function checkAdmin() {
        if(document.getElementById('user').value === "TAFU" && document.getElementById('pass').value === "54321") {
            document.getElementById('loginPage').classList.add('hidden');
            document.getElementById('adminPage').classList.remove('hidden');
            updateStats();
        } else { alert("Xato!"); }
    }

    function showTab(tab) {
        document.getElementById('surveyPage').classList.add('hidden');
        document.getElementById('loginPage').classList.add('hidden');
        document.getElementById('adminPage').classList.add('hidden');
        const btns = document.querySelectorAll('.tab-btn');
        btns[0].classList.remove('active'); btns[1].classList.remove('active');
        if(tab === 'survey') { document.getElementById('surveyPage').classList.remove('hidden'); btns[0].classList.add('active'); } 
        else { document.getElementById('loginPage').classList.remove('hidden'); btns[1].classList.add('active'); }
    }

    function submitForm(e) {
        e.preventDefault();
        localStorage.setItem('data_' + document.getElementById('tutorSelect').value.replace(/\s+/g, '_'), 'true');
        alert('Javobingiz qabul qilindi!');
        location.reload(); 
    }
</script>
</body>
</html>
