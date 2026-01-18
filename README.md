# Sip-calculate-
Sip calculate 
<!DOCTYPE html>
<html lang="hi" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SIP Pro - ऑल-इन-वन कैलकुलेटर</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    
    <style>
        :root { --bg: #f0f4f8; --card: #ffffff; --text: #333; --primary: #1a73e8; --border: #ddd; }
        [data-theme="dark"] { --bg: #121212; --card: #1e1e1e; --text: #f0f0f0; --primary: #8ab4f8; --border: #444; }

        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: var(--text); padding: 15px; transition: 0.3s; }
        .container { background: var(--card); padding: 25px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); width: 100%; max-width: 800px; margin: auto; }
        
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        .tabs { display: flex; gap: 10px; margin-bottom: 20px; border-bottom: 1px solid var(--border); }
        .tab-btn { padding: 10px 15px; border: none; background: none; color: var(--text); cursor: pointer; font-weight: bold; opacity: 0.6; }
        .tab-btn.active { opacity: 1; border-bottom: 3px solid var(--primary); color: var(--primary); }

        .input-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin-bottom: 20px; }
        input, select { width: 100%; padding: 12px; border: 1px solid var(--border); border-radius: 10px; background: var(--card); color: var(--text); box-sizing: border-box; }
        
        .btn-group { display: flex; gap: 10px; flex-wrap: wrap; }
        button.main-btn { flex: 2; padding: 15px; background: var(--primary); color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }
        button.pdf-btn { flex: 1; background: #34a853; color: white; border: none; border-radius: 10px; cursor: pointer; display: none; }

        .result-section { display: none; margin-top: 30px; animation: fadeIn 0.5s; }
        .summary-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 15px; margin-bottom: 25px; }
        .summary-item { padding: 15px; border-radius: 12px; text-align: center; background: rgba(26, 115, 232, 0.1); }
        
        .delay-notice { padding: 15px; background: #fff5f4; border: 1px dashed #d93025; border-radius: 10px; color: #d93025; margin-bottom: 20px; }
        [data-theme="dark"] .delay-notice { background: #2d1a1a; }

        table { width: 100%; border-collapse: collapse; margin-top: 20px; font-size: 14px; }
        th, td { border: 1px solid var(--border); padding: 12px; text-align: right; }
        th { background: rgba(0,0,0,0.03); text-align: center; }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

<div class="container" id="main-app">
    <div class="header">
        <h2 style="color: var(--primary);">SIP Pro Suite</h2>
        <button onclick="toggleTheme()" style="background:none; border:1px solid var(--border); padding:8px; border-radius:50%; cursor:pointer;">🌓</button>
    </div>

    <div class="tabs">
        <button class="tab-btn active" onclick="switchTab('sip')">Standard SIP</button>
        <button class="tab-btn" onclick="switchTab('goal')">Goal Planner</button>
        <button class="tab-btn" onclick="switchTab('retirement')">Retirement</button>
    </div>

    <div class="input-grid">
        <div id="p-input">
            <label id="p-label">मासिक निवेश</label>
            <div style="display:flex; gap:5px;">
                <select id="currency" style="width:80px;" onchange="updateUI()">
                    <option value="INR" data-symbol="₹" data-rate="1">INR</option>
                    <option value="USD" data-symbol="$" data-rate="83.5">USD</option>
                    <option value="AED" data-symbol="د.إ" data-rate="22.7">AED</option>
                </select>
                <input type="number" id="p_value" value="5000">
            </div>
        </div>
        <div>
            <label>अपेक्षित रिटर्न (%)</label>
            <input type="number" id="rate" value="12">
        </div>
        <div id="duration-input">
            <label id="d-label">समय (वर्ष)</label>
            <input type="number" id="duration" value="10">
        </div>
        <div id="extra-input">
            <label id="e-label">स्टेप-अप (%)</label>
            <input type="number" id="extra_val" value="10">
        </div>
    </div>

    <div class="btn-group">
        <button class="main-btn" onclick="calculate()">गणना करें</button>
        <button id="pdfBtn" class="pdf-btn" onclick="exportPDF()">PDF रिपोर्ट</button>
    </div>

    <div id="results" class="result-section">
        <div class="summary-grid" id="summaryArea"></div>
        <div id="delayBox" class="delay-notice" style="display:none;"></div>
        <div style="height:300px; margin-bottom:30px;"><canvas id="mainChart"></canvas></div>
        <h3>विस्तृत रिपोर्ट</h3>
        <table id="reportTable"><thead><tr id="tableHead"></tr></thead><tbody id="tableBody"></tbody></table>
    </div>
</div>

<script>
    let activeTab = 'sip';
    let myChart = null;

    function toggleTheme() {
        const theme = document.documentElement.getAttribute('data-theme') === 'dark' ? 'light' : 'dark';
        document.documentElement.setAttribute('data-theme', theme);
    }

    function switchTab(tab) {
        activeTab = tab;
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        event.target.classList.add('active');
        
        // UI Labels Update
        if(tab === 'sip') {
            setLabels("मासिक निवेश", "समय (वर्ष)", "स्टेप-अप (%)", 5000, 10, 10);
        } else if(tab === 'goal') {
            setLabels("लक्ष्य राशि", "समय (वर्ष)", "महंगाई (%)", 10000000, 15, 6);
        } else {
            setLabels("मासिक पेंशन", "रिटायरमेंट तक वर्ष", "रिटायरमेंट बाद रिटर्न (%)", 100000, 25, 8);
        }
        document.getElementById('results').style.display = 'none';
    }

    function setLabels(l1, l2, l3, v1, v2, v3) {
        document.getElementById('p-label').innerText = l1;
        document.getElementById('d-label').innerText = l2;
        document.getElementById('e-label').innerText = l3;
        document.getElementById('p_value').value = v1;
        document.getElementById('duration').value = v2;
        document.getElementById('extra_val').value = v3;
    }

    function calculate() {
        const currency = document.getElementById('currency');
        const symbol = currency.options[currency.selectedIndex].getAttribute('data-symbol');
        const r = parseFloat(document.getElementById('rate').value)/100/12;
        const dur = parseInt(document.getElementById('duration').value);
        const extra = parseFloat(document.getElementById('extra_val').value);
        
        let htmlSummary = "";
        let tableHTML = "";
        let labels = [];
        let dataValues = [];

        if(activeTab === 'sip') {
            let p = parseFloat(document.getElementById('p_value').value);
            let totalInvested = 0; let fv = 0;
            document.getElementById('tableHead').innerHTML = "<th>वर्ष</th><th>मंथली निवेश</th><th>कुल निवेश</th><th>फंड वैल्यू</th>";
            
            for(let y=1; y<=dur; y++){
                let yearlyInv = 0;
                for(let m=1; m<=12; m++) { fv = (fv + p) * (1+r); yearlyInv += p; }
                totalInvested += yearlyInv;
                tableHTML += `<tr><td>${y}</td><td>${symbol}${Math.round(p).toLocaleString()}</td><td>${symbol}${Math.round(totalInvested).toLocaleString()}</td><td>${symbol}${Math.round(fv).toLocaleString()}</td></tr>`;
                labels.push("Year "+y); dataValues.push(Math.round(fv));
                p *= (1 + extra/100);
            }
            htmlSummary = `<div class="summary-item"><h3>${symbol}${Math.round(fv).toLocaleString()}</h3><p>मैच्योरिटी राशि</p></div>
                           <div class="summary-item"><h3>${symbol}${Math.round(totalInvested).toLocaleString()}</h3><p>कुल निवेश</p></div>`;
            
            // Cost of Delay
            let delayFV = 0; let n_delay = (dur-5)*12;
            if(dur > 5) {
                let p_fixed = parseFloat(document.getElementById('p_value').value);
                for(let i=1; i<=n_delay; i++) delayFV = (delayFV + p_fixed) * (1+r);
                document.getElementById('delayBox').innerHTML = `⚠️ 5 साल की देरी से आपको <b>${symbol}${Math.round(fv - delayFV).toLocaleString()}</b> का नुकसान हो सकता है!`;
                document.getElementById('delayBox').style.display = 'block';
            }
        } else if(activeTab === 'goal') {
            const target = parseFloat(document.getElementById('p_value').value);
            const n = dur * 12;
            const requiredSIP = target / (((Math.pow(1+r, n)-1)/r)*(1+r));
            htmlSummary = `<div class="summary-item"><h3>${symbol}${Math.round(requiredSIP).toLocaleString()}</h3><p>ज़रूरी मासिक SIP</p></div>`;
            document.getElementById('delayBox').style.display = 'none';
        }

        document.getElementById('summaryArea').innerHTML = htmlSummary;
        document.getElementById('tableBody').innerHTML = tableHTML;
        document.getElementById('results').style.display = 'block';
        document.getElementById('pdfBtn').style.display = 'block';
        updateChart(labels, dataValues);
        localStorage.setItem('sipData', JSON.stringify({tab: activeTab, v1: document.getElementById('p_value').value}));
    }

    function updateChart(labels, data) {
        const ctx = document.getElementById('mainChart').getContext('2d');
        if(myChart) myChart.destroy();
        myChart = new Chart(ctx, {
            type: 'line',
            data: { labels: labels, datasets: [{ label: 'Wealth Growth', data: data, borderColor: '#1a73e8', tension: 0.4, fill: true, backgroundColor: 'rgba(26, 115, 232, 0.1)' }] },
            options: { maintainAspectRatio: false }
        });
    }

    function exportPDF() {
        const element = document.getElementById('main-app');
        html2pdf().from(element).save('My_Financial_Report.pdf');
    }
</script>
</body>
</html>
<!DOCTYPE html>
<html lang="hi" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SIP Pro - ऑल-इन-वन कैलकुलेटर</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    
    <style>
        :root { --bg: #f0f4f8; --card: #ffffff; --text: #333; --primary: #1a73e8; --border: #ddd; }
        [data-theme="dark"] { --bg: #121212; --card: #1e1e1e; --text: #f0f0f0; --primary: #8ab4f8; --border: #444; }

        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: var(--text); padding: 15px; transition: 0.3s; }
        .container { background: var(--card); padding: 25px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); width: 100%; max-width: 800px; margin: auto; }
        
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        .tabs { display: flex; gap: 10px; margin-bottom: 20px; border-bottom: 1px solid var(--border); }
        .tab-btn { padding: 10px 15px; border: none; background: none; color: var(--text); cursor: pointer; font-weight: bold; opacity: 0.6; }
        .tab-btn.active { opacity: 1; border-bottom: 3px solid var(--primary); color: var(--primary); }

        .input-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin-bottom: 20px; }
        input, select { width: 100%; padding: 12px; border: 1px solid var(--border); border-radius: 10px; background: var(--card); color: var(--text); box-sizing: border-box; }
        
        .btn-group { display: flex; gap: 10px; flex-wrap: wrap; }
        button.main-btn { flex: 2; padding: 15px; background: var(--primary); color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }
        button.pdf-btn { flex: 1; background: #34a853; color: white; border: none; border-radius: 10px; cursor: pointer; display: none; }

        .result-section { display: none; margin-top: 30px; animation: fadeIn 0.5s; }
        .summary-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 15px; margin-bottom: 25px; }
        .summary-item { padding: 15px; border-radius: 12px; text-align: center; background: rgba(26, 115, 232, 0.1); }
        
        .delay-notice { padding: 15px; background: #fff5f4; border: 1px dashed #d93025; border-radius: 10px; color: #d93025; margin-bottom: 20px; }
        [data-theme="dark"] .delay-notice { background: #2d1a1a; }

        table { width: 100%; border-collapse: collapse; margin-top: 20px; font-size: 14px; }
        th, td { border: 1px solid var(--border); padding: 12px; text-align: right; }
        th { background: rgba(0,0,0,0.03); text-align: center; }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

<div class="container" id="main-app">
    <div class="header">
        <h2 style="color: var(--primary);">SIP Pro Suite</h2>
        <button onclick="toggleTheme()" style="background:none; border:1px solid var(--border); padding:8px; border-radius:50%; cursor:pointer;">🌓</button>
    </div>

    <div class="tabs">
        <button class="tab-btn active" onclick="switchTab('sip')">Standard SIP</button>
        <button class="tab-btn" onclick="switchTab('goal')">Goal Planner</button>
        <button class="tab-btn" onclick="switchTab('retirement')">Retirement</button>
    </div>

    <div class="input-grid">
        <div id="p-input">
            <label id="p-label">मासिक निवेश</label>
            <div style="display:flex; gap:5px;">
                <select id="currency" style="width:80px;" onchange="updateUI()">
                    <option value="INR" data-symbol="₹" data-rate="1">INR</option>
                    <option value="USD" data-symbol="$" data-rate="83.5">USD</option>
                    <option value="AED" data-symbol="د.إ" data-rate="22.7">AED</option>
                </select>
                <input type="number" id="p_value" value="5000">
            </div>
        </div>
        <div>
            <label>अपेक्षित रिटर्न (%)</label>
            <input type="number" id="rate" value="12">
        </div>
        <div id="duration-input">
            <label id="d-label">समय (वर्ष)</label>
            <input type="number" id="duration" value="10">
        </div>
        <div id="extra-input">
            <label id="e-label">स्टेप-अप (%)</label>
            <input type="number" id="extra_val" value="10">
        </div>
    </div>

    <div class="btn-group">
        <button class="main-btn" onclick="calculate()">गणना करें</button>
        <button id="pdfBtn" class="pdf-btn" onclick="exportPDF()">PDF रिपोर्ट</button>
    </div>

    <div id="results" class="result-section">
        <div class="summary-grid" id="summaryArea"></div>
        <div id="delayBox" class="delay-notice" style="display:none;"></div>
        <div style="height:300px; margin-bottom:30px;"><canvas id="mainChart"></canvas></div>
        <h3>विस्तृत रिपोर्ट</h3>
        <table id="reportTable"><thead><tr id="tableHead"></tr></thead><tbody id="tableBody"></tbody></table>
    </div>
</div>

<script>
    let activeTab = 'sip';
    let myChart = null;

    function toggleTheme() {
        const theme = document.documentElement.getAttribute('data-theme') === 'dark' ? 'light' : 'dark';
        document.documentElement.setAttribute('data-theme', theme);
    }

    function switchTab(tab) {
        activeTab = tab;
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        event.target.classList.add('active');
        
        // UI Labels Update
        if(tab === 'sip') {
            setLabels("मासिक निवेश", "समय (वर्ष)", "स्टेप-अप (%)", 5000, 10, 10);
        } else if(tab === 'goal') {
            setLabels("लक्ष्य राशि", "समय (वर्ष)", "महंगाई (%)", 10000000, 15, 6);
        } else {
            setLabels("मासिक पेंशन", "रिटायरमेंट तक वर्ष", "रिटायरमेंट बाद रिटर्न (%)", 100000, 25, 8);
        }
        document.getElementById('results').style.display = 'none';
    }

    function setLabels(l1, l2, l3, v1, v2, v3) {
        document.getElementById('p-label').innerText = l1;
        document.getElementById('d-label').innerText = l2;
        document.getElementById('e-label').innerText = l3;
        document.getElementById('p_value').value = v1;
        document.getElementById('duration').value = v2;
        document.getElementById('extra_val').value = v3;
    }

    function calculate() {
        const currency = document.getElementById('currency');
        const symbol = currency.options[currency.selectedIndex].getAttribute('data-symbol');
        const r = parseFloat(document.getElementById('rate').value)/100/12;
        const dur = parseInt(document.getElementById('duration').value);
        const extra = parseFloat(document.getElementById('extra_val').value);
        
        let htmlSummary = "";
        let tableHTML = "";
        let labels = [];
        let dataValues = [];

        if(activeTab === 'sip') {
            let p = parseFloat(document.getElementById('p_value').value);
            let totalInvested = 0; let fv = 0;
            document.getElementById('tableHead').innerHTML = "<th>वर्ष</th><th>मंथली निवेश</th><th>कुल निवेश</th><th>फंड वैल्यू</th>";
            
            for(let y=1; y<=dur; y++){
                let yearlyInv = 0;
                for(let m=1; m<=12; m++) { fv = (fv + p) * (1+r); yearlyInv += p; }
                totalInvested += yearlyInv;
                tableHTML += `<tr><td>${y}</td><td>${symbol}${Math.round(p).toLocaleString()}</td><td>${symbol}${Math.round(totalInvested).toLocaleString()}</td><td>${symbol}${Math.round(fv).toLocaleString()}</td></tr>`;
                labels.push("Year "+y); dataValues.push(Math.round(fv));
                p *= (1 + extra/100);
            }
            htmlSummary = `<div class="summary-item"><h3>${symbol}${Math.round(fv).toLocaleString()}</h3><p>मैच्योरिटी राशि</p></div>
                           <div class="summary-item"><h3>${symbol}${Math.round(totalInvested).toLocaleString()}</h3><p>कुल निवेश</p></div>`;
            
            // Cost of Delay
            let delayFV = 0; let n_delay = (dur-5)*12;
            if(dur > 5) {
                let p_fixed = parseFloat(document.getElementById('p_value').value);
                for(let i=1; i<=n_delay; i++) delayFV = (delayFV + p_fixed) * (1+r);
                document.getElementById('delayBox').innerHTML = `⚠️ 5 साल की देरी से आपको <b>${symbol}${Math.round(fv - delayFV).toLocaleString()}</b> का नुकसान हो सकता है!`;
                document.getElementById('delayBox').style.display = 'block';
            }
        } else if(activeTab === 'goal') {
            const target = parseFloat(document.getElementById('p_value').value);
            const n = dur * 12;
            const requiredSIP = target / (((Math.pow(1+r, n)-1)/r)*(1+r));
            htmlSummary = `<div class="summary-item"><h3>${symbol}${Math.round(requiredSIP).toLocaleString()}</h3><p>ज़रूरी मासिक SIP</p></div>`;
            document.getElementById('delayBox').style.display = 'none';
        }

        document.getElementById('summaryArea').innerHTML = htmlSummary;
        document.getElementById('tableBody').innerHTML = tableHTML;
        document.getElementById('results').style.display = 'block';
        document.getElementById('pdfBtn').style.display = 'block';
        updateChart(labels, dataValues);
        localStorage.setItem('sipData', JSON.stringify({tab: activeTab, v1: document.getElementById('p_value').value}));
    }

    function updateChart(labels, data) {
        const ctx = document.getElementById('mainChart').getContext('2d');
        if(myChart) myChart.destroy();
        myChart = new Chart(ctx, {
            type: 'line',
            data: { labels: labels, datasets: [{ label: 'Wealth Growth', data: data, borderColor: '#1a73e8', tension: 0.4, fill: true, backgroundColor: 'rgba(26, 115, 232, 0.1)' }] },
            options: { maintainAspectRatio: false }
        });
    }

    function exportPDF() {
        const element = document.getElementById('main-app');
        html2pdf().from(element).save('My_Financial_Report.pdf');
    }
</script>
</body>
</html>
