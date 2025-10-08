<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Health & Lifestyle Insights Dashboard</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 20px;
            min-height: 100vh;
        }
        .dashboard-container { max-width: 1600px; margin: 0 auto; }
        .header {
            background: white;
            padding: 35px;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            margin-bottom: 30px;
        }
        .header h1 { color: #667eea; font-size: 38px; margin-bottom: 10px; font-weight: 800; }
        .header p { color: #64748b; font-size: 18px; }
        .info-box {
            background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
            padding: 20px;
            border-radius: 15px;
            margin-bottom: 25px;
            border-left: 5px solid #f59e0b;
        }
        .info-box h3 { color: #92400e; margin-bottom: 10px; }
        .info-box p { color: #78350f; font-size: 14px; line-height: 1.6; }
        .info-box a { color: #2563eb; text-decoration: underline; }
        .filters {
            background: white;
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            margin-bottom: 30px;
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
            gap: 20px;
        }
        .filter-group label {
            display: block;
            font-weight: 700;
            color: #667eea;
            margin-bottom: 8px;
            font-size: 14px;
        }
        .filter-group select {
            width: 100%;
            padding: 12px;
            border: 2px solid #e0e7ff;
            border-radius: 10px;
            font-size: 14px;
            background: white;
            cursor: pointer;
            transition: all 0.3s;
            font-weight: 600;
        }
        .filter-group select:hover {
            border-color: #667eea;
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.2);
        }
        .kpi-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 25px;
            margin-bottom: 30px;
        }
        .kpi-card {
            background: white;
            padding: 28px;
            border-radius: 18px;
            box-shadow: 0 15px 40px rgba(0,0,0,0.2);
            transition: all 0.3s;
            position: relative;
            overflow: hidden;
        }
        .kpi-card::before {
            content: '';
            position: absolute;
            left: 0;
            top: 0;
            bottom: 0;
            width: 6px;
        }
        .kpi-card.green::before { background: linear-gradient(180deg, #10b981, #059669); }
        .kpi-card.yellow::before { background: linear-gradient(180deg, #f59e0b, #d97706); }
        .kpi-card.red::before { background: linear-gradient(180deg, #ef4444, #dc2626); }
        .kpi-card.purple::before { background: linear-gradient(180deg, #8b5cf6, #7c3aed); }
        .kpi-card:hover { transform: translateY(-8px); box-shadow: 0 25px 50px rgba(0,0,0,0.3); }
        .kpi-title {
            font-size: 15px;
            color: #64748b;
            font-weight: 700;
            margin-bottom: 12px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        .kpi-value { font-size: 48px; font-weight: 900; margin-bottom: 10px; }
        .kpi-card.green .kpi-value { color: #10b981; }
        .kpi-card.yellow .kpi-value { color: #f59e0b; }
        .kpi-card.red .kpi-value { color: #ef4444; }
        .kpi-card.purple .kpi-value { color: #8b5cf6; }
        .kpi-insight {
            font-size: 13px;
            color: #64748b;
            margin-top: 12px;
            font-weight: 600;
            line-height: 1.5;
        }
        .chart-section {
            background: white;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 15px 40px rgba(0,0,0,0.2);
            margin-bottom: 30px;
        }
        .chart-section.full-width { grid-column: 1 / -1; }
        .chart-title {
            font-size: 22px;
            font-weight: 800;
            color: #667eea;
            margin-bottom: 25px;
            padding-bottom: 15px;
            border-bottom: 3px solid #e0e7ff;
        }
        .chart-container { position: relative; height: 350px; }
        .chart-container.line-chart { height: 400px; }
        .chart-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 30px;
            margin-bottom: 30px;
        }
        .chart-insight {
            margin-top: 20px;
            padding: 15px;
            background: linear-gradient(135deg, #dbeafe 0%, #e0e7ff 100%);
            border-radius: 12px;
            font-size: 14px;
            color: #334155;
            font-weight: 600;
            border-left: 4px solid #667eea;
        }
        .loading {
            text-align: center;
            padding: 80px;
            color: white;
            font-size: 26px;
            font-weight: 700;
            background: rgba(255,255,255,0.15);
            border-radius: 20px;
        }
        @media (max-width: 1024px) { .chart-grid { grid-template-columns: 1fr; } }
        @media (max-width: 768px) { .kpi-grid { grid-template-columns: 1fr; } }
    </style>
</head>
<body>
    <div class="dashboard-container">
        <div class="header">
            <h1>🏥 Health & Lifestyle Insights Dashboard</h1>
            <p>Comprehensive population well-being analysis and public health trends</p>
        </div>

        <div class="info-box">
            <h3>📊 Data Source</h3>
            <p>This dashboard uses data from: <a href="https://docs.google.com/spreadsheets/d/1pOjJwGCSrYQrnk_zULqGaItwwhNufqqAhLD2czyewbw/edit?gid=1682477492#gid=1682477492" target="_blank">Google Sheets Health Dataset</a></p>
            <p><strong>Note:</strong> Download the sheet as CSV and upload it below to visualize the data.</p>
        </div>

        <div class="filters">
            <div class="filter-group">
                <label>📁 Upload CSV File</label>
                <input type="file" id="csvFileInput" accept=".csv" style="width:100%;padding:12px;border:2px solid #e0e7ff;border-radius:10px;cursor:pointer;" />
            </div>
            <div class="filter-group">
                <label>🌍 Region</label>
                <select id="regionFilter"><option value="All">All Regions</option></select>
            </div>
            <div class="filter-group">
                <label>👥 Age Group</label>
                <select id="ageFilter">
                    <option value="All">All Ages</option>
                    <option value="18-30">18-30</option>
                    <option value="31-45">31-45</option>
                    <option value="46-60">46-60</option>
                    <option value="60+">60+</option>
                </select>
            </div>
            <div class="filter-group">
                <label>⚧ Gender</label>
                <select id="genderFilter"><option value="All">All Genders</option></select>
            </div>
            <div class="filter-group">
                <label>💼 Employment</label>
                <select id="employmentFilter"><option value="All">All Employment</option></select>
            </div>
        </div>

        <div id="loading" class="loading">📊 Upload CSV from Google Sheets to start...</div>

        <div id="dashboard" style="display: none;">
            <div class="kpi-grid">
                <div class="kpi-card" id="smokingCard">
                    <div class="kpi-title">🚬 Smoking Rate</div>
                    <div class="kpi-value" id="smokingValue">0%</div>
                    <div class="kpi-insight" id="smokingInsight"></div>
                </div>
                <div class="kpi-card" id="activityCard">
                    <div class="kpi-title">🏃 Physical Activity</div>
                    <div class="kpi-value" id="activityValue">0%</div>
                    <div class="kpi-insight" id="activityInsight"></div>
                </div>
                <div class="kpi-card" id="chronicCard">
                    <div class="kpi-title">❤️ Chronic Conditions</div>
                    <div class="kpi-value" id="chronicValue">0%</div>
                    <div class="kpi-insight" id="chronicInsight"></div>
                </div>
                <div class="kpi-card" id="mentalCard">
                    <div class="kpi-title">🧠 Mental Illness</div>
                    <div class="kpi-value" id="mentalValue">0%</div>
                    <div class="kpi-insight" id="mentalInsight"></div>
                </div>
                <div class="kpi-card purple">
                    <div class="kpi-title">👨‍👩‍👧‍👦 Avg Children</div>
                    <div class="kpi-value" id="childrenValue">0.0</div>
                    <div class="kpi-insight">Family size indicator</div>
                </div>
                <div class="kpi-card" id="depressionCard">
                    <div class="kpi-title">😔 Depression Risk</div>
                    <div class="kpi-value" id="depressionValue">0%</div>
                    <div class="kpi-insight" id="depressionInsight"></div>
                </div>
            </div>

            <div class="chart-section full-width">
                <div class="chart-title">📈 Mental Illness Trend by Age</div>
                <div class="chart-container line-chart">
                    <canvas id="mentalTrendChart"></canvas>
                </div>
                <div class="chart-insight" id="mentalTrendInsight"></div>
            </div>

            <div class="chart-grid">
                <div class="chart-section">
                    <div class="chart-title">💰 Income by Employment</div>
                    <div class="chart-container"><canvas id="incomeChart"></canvas></div>
                    <div class="chart-insight" id="incomeInsight"></div>
                </div>
                <div class="chart-section">
                    <div class="chart-title">😴 Sleep Quality</div>
                    <div class="chart-container"><canvas id="sleepChart"></canvas></div>
                    <div class="chart-insight" id="sleepInsight"></div>
                </div>
                <div class="chart-section">
                    <div class="chart-title">🎓 Education Levels</div>
                    <div class="chart-container"><canvas id="educationChart"></canvas></div>
                    <div class="chart-insight" id="educationInsight"></div>
                </div>
                <div class="chart-section">
                    <div class="chart-title">🏋️ Activity by Age</div>
                    <div class="chart-container"><canvas id="activityAgeChart"></canvas></div>
                    <div class="chart-insight" id="activityAgeInsight"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        var allData = [];
        var charts = {};

        document.getElementById('csvFileInput').addEventListener('change', function (e) {
            var file = e.target.files[0];
            if (!file) return;
            document.getElementById('loading').textContent = '⏳ Loading...';
            Papa.parse(file, {
                header: true,
                dynamicTyping: true,
                skipEmptyLines: true,
                complete: function(r) { allData = r.data; initDashboard(); },
                error: function() { document.getElementById('loading').textContent = '❌ Error'; }
            });
        });

        function initDashboard() {
            if (!allData || allData.length === 0) return;
            var regions = [], genders = [], employments = [];
            for (var i = 0; i < allData.length; i++) {
                var d = allData[i];
                if (d.Region && regions.indexOf(d.Region) === -1) regions.push(d.Region);
                if (d.Gender && genders.indexOf(d.Gender) === -1) genders.push(d.Gender);
                if (d['Employment Status'] && employments.indexOf(d['Employment Status']) === -1) employments.push(d['Employment Status']);
            }
            addOpts('regionFilter', regions);
            addOpts('genderFilter', genders);
            addOpts('employmentFilter', employments);
            document.getElementById('regionFilter').addEventListener('change', update);
            document.getElementById('ageFilter').addEventListener('change', update);
            document.getElementById('genderFilter').addEventListener('change', update);
            document.getElementById('employmentFilter').addEventListener('change', update);
            update();
            document.getElementById('loading').style.display = 'none';
            document.getElementById('dashboard').style.display = 'block';
        }

        function addOpts(id, opts) {
            var sel = document.getElementById(id);
            for (var i = 0; i < opts.length; i++) {
                var opt = document.createElement('option');
                opt.value = opts[i];
                opt.textContent = opts[i];
                sel.appendChild(opt);
            }
        }

        function getFiltered() {
            var r = document.getElementById('regionFilter').value;
            var ag = document.getElementById('ageFilter').value;
            var g = document.getElementById('genderFilter').value;
            var e = document.getElementById('employmentFilter').value;
            var filtered = [];
            for (var i = 0; i < allData.length; i++) {
                var d = allData[i];
                if (r !== 'All' && d.Region !== r) continue;
                if (g !== 'All' && d.Gender !== g) continue;
                if (e !== 'All' && d['Employment Status'] !== e) continue;
                if (ag !== 'All') {
                    var age = d.Age;
                    if (ag === '18-30' && (age < 18 || age > 30)) continue;
                    if (ag === '31-45' && (age < 31 || age > 45)) continue;
                    if (ag === '46-60' && (age < 46 || age > 60)) continue;
                    if (ag === '60+' && age < 60) continue;
                }
                filtered.push(d);
            }
            return filtered;
        }

        function update() {
            var data = getFiltered();
            if (data.length === 0) {
                document.getElementById('loading').style.display = 'block';
                document.getElementById('loading').textContent = '🔍 No matches';
                document.getElementById('dashboard').style.display = 'none';
                return;
            }
            document.getElementById('loading').style.display = 'none';
            document.getElementById('dashboard').style.display = 'block';
            updateKPIs(data);
            updateCharts(data);
        }

        function updateKPIs(data) {
            var total = data.length;
            var smokers = 0, active = 0, chronic = 0, mental = 0, depression = 0, totalKids = 0;
            for (var i = 0; i < data.length; i++) {
                if (data[i]['Smoking Status'] === 'Yes' || data[i]['Smoking Status'] === 'Smoker') smokers++;
                var act = data[i]['Physical Activity Level'];
                if (act === 'Active' || act === 'Moderate') active++;
                if (data[i]['Chronic Medical Conditions'] === 'Yes') chronic++;
                if (data[i]['Mental Illness History'] === 'Yes') mental++;
                if (data[i]['History of Depression'] === 'Yes' || data[i]['Family History of Depression'] === 'Yes') depression++;
                totalKids += data[i]['Number of Children'] || 0;
            }
            var sr = ((smokers / total) * 100).toFixed(1);
            document.getElementById('smokingValue').textContent = sr + '%';
            setColor('smokingCard', sr, 30, 15, false);
            document.getElementById('smokingInsight').textContent = sr > 30 ? 'HIGH RISK' : sr > 15 ? 'MODERATE' : 'LOW RISK';

            var ar = ((active / total) * 100).toFixed(1);
            document.getElementById('activityValue').textContent = ar + '%';
            setColor('activityCard', ar, 60, 40, true);
            document.getElementById('activityInsight').textContent = ar > 60 ? 'EXCELLENT' : ar > 40 ? 'MODERATE' : 'LOW';

            var cr = ((chronic / total) * 100).toFixed(1);
            document.getElementById('chronicValue').textContent = cr + '%';
            setColor('chronicCard', cr, 30, 15, false);
            document.getElementById('chronicInsight').textContent = cr > 30 ? 'HIGH' : cr > 15 ? 'MODERATE' : 'LOW';

            var mr = ((mental / total) * 100).toFixed(1);
            document.getElementById('mentalValue').textContent = mr + '%';
            setColor('mentalCard', mr, 25, 15, false);
            document.getElementById('mentalInsight').textContent = mr > 25 ? 'HIGH' : mr > 15 ? 'MODERATE' : 'LOW';

            document.getElementById('childrenValue').textContent = (totalKids / total).toFixed(2);

            var dr = ((depression / total) * 100).toFixed(1);
            document.getElementById('depressionValue').textContent = dr + '%';
            setColor('depressionCard', dr, 25, 15, false);
            document.getElementById('depressionInsight').textContent = dr > 25 ? 'ELEVATED' : 'Normal range';
        }

        function setColor(id, val, high, med, rev) {
            var card = document.getElementById(id);
            card.className = 'kpi-card';
            var v = parseFloat(val);
            if (rev) {
                if (v > high) card.className = 'kpi-card green';
                else if (v > med) card.className = 'kpi-card yellow';
                else card.className = 'kpi-card red';
            } else {
                if (v > high) card.className = 'kpi-card red';
                else if (v > med) card.className = 'kpi-card yellow';
                else card.className = 'kpi-card green';
            }
        }

        function updateCharts(data) {
            for (var k in charts) if (charts[k]) charts[k].destroy();
            charts = {};

            var incomeByEmp = {};
            for (var i = 0; i < data.length; i++) {
                var emp = data[i]['Employment Status'] || 'Unknown';
                var inc = data[i].Income || 0;
                if (!incomeByEmp[emp]) incomeByEmp[emp] = { total: 0, count: 0 };
                incomeByEmp[emp].total += inc;
                incomeByEmp[emp].count++;
            }
            var incData = [];
            for (var emp in incomeByEmp) incData.push({ emp: emp, avg: Math.round(incomeByEmp[emp].total / incomeByEmp[emp].count) });
            incData.sort(function(a, b) { return b.avg - a.avg; });

            charts.income = new Chart(document.getElementById('incomeChart'), {
                type: 'bar',
                data: {
                    labels: incData.map(function(d) { return d.emp; }),
                    datasets: [{ data: incData.map(function(d) { return d.avg; }), backgroundColor: ['#667eea', '#764ba2', '#f093fb', '#4facfe', '#00f2fe'], borderRadius: 8 }]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { beginAtZero: true } } }
            });
            document.getElementById('incomeInsight').textContent = 'Income varies by employment status';

            var sleepCounts = { 'Good': 0, 'Fair': 0, 'Poor': 0 };
            for (var i = 0; i < data.length; i++) {
                var s = data[i]['Sleep Quality'];
                if (s && sleepCounts.hasOwnProperty(s)) sleepCounts[s]++;
            }
            charts.sleep = new Chart(document.getElementById('sleepChart'), {
                type: 'doughnut',
                data: { labels: Object.keys(sleepCounts), datasets: [{ data: Object.values(sleepCounts), backgroundColor: ['#10b981', '#f59e0b', '#ef4444'], borderWidth: 4 }] },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom' } } }
            });
            document.getElementById('sleepInsight').textContent = 'Sleep quality distribution';

            var eduCounts = {};
            for (var i = 0; i < data.length; i++) {
                var edu = data[i]['Education Level'] || 'Unknown';
                eduCounts[edu] = (eduCounts[edu] || 0) + 1;
            }
            var eduData = [];
            for (var edu in eduCounts) eduData.push({ edu: edu, count: eduCounts[edu] });

            charts.education = new Chart(document.getElementById('educationChart'), {
                type: 'bar',
                data: {
                    labels: eduData.map(function(d) { return d.edu; }),
                    datasets: [{ data: eduData.map(function(d) { return d.count; }), backgroundColor: ['#8b5cf6', '#ec4899', '#f59e0b', '#10b981', '#3b82f6'], borderRadius: 8 }]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { beginAtZero: true } } }
            });
            document.getElementById('educationInsight').textContent = 'Education distribution';

            var ageGroups = ['18-30', '31-45', '46-60', '60+'];
            var actByAge = [], menByAge = [];
            for (var j = 0; j < ageGroups.length; j++) {
                var grp = ageGroups[j];
                var parts = grp.indexOf('+') !== -1 ? [60, 999] : grp.split('-').map(Number);
                var filtered = [];
                for (var i = 0; i < data.length; i++) if (data[i].Age >= parts[0] && data[i].Age <= parts[1]) filtered.push(data[i]);
                var actCnt = 0, menCnt = 0;
                for (var i = 0; i < filtered.length; i++) {
                    var a = filtered[i]['Physical Activity Level'];
                    if (a === 'Active' || a === 'Moderate') actCnt++;
                    if (filtered[i]['Mental Illness History'] === 'Yes') menCnt++;
                }
                actByAge.push({ grp: grp, pct: filtered.length > 0 ? ((actCnt / filtered.length) * 100).toFixed(1) : 0 });
                menByAge.push({ grp: grp, pct: filtered.length > 0 ? ((menCnt / filtered.length) * 100).toFixed(1) : 0 });
            }

            charts.activityAge = new Chart(document.getElementById('activityAgeChart'), {
                type: 'bar',
                data: {
                    labels: actByAge.map(function(d) { return d.grp; }),
                    datasets: [{ data: actByAge.map(function(d) { return d.pct; }), backgroundColor: ['#10b981', '#14b8a6', '#06b6d4', '#0ea5e9'], borderRadius: 8 }]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { beginAtZero: true, max: 100 } } }
            });
            document.getElementById('activityAgeInsight').textContent = 'Activity declines with age';

            charts.mentalTrend = new Chart(document.getElementById('mentalTrendChart'), {
                type: 'line',
                data: {
                    labels: menByAge.map(function(d) { return d.grp; }),
                    datasets: [{ label: 'Mental Illness %', data: menByAge.map(function(d) { return d.pct; }), borderColor: '#ef4444', backgroundColor: 'rgba(239,68,68,0.1)', borderWidth: 3, fill: true, tension: 0.4 }]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: true } }, scales: { y: { beginAtZero: true, max: 100 } } }
            });
            document.getElementById('mentalTrendInsight').textContent = 'Mental illness trends across age groups';
        }
    </script>
</body>
</html>
