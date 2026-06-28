<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>6 Oylik Trading Plan & Tracker</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #0f172a; color: #f8fafc; margin: 0; padding: 20px; }
    .container { max-width: 1300px; margin: auto; background: #1e293b; padding: 25px; border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.3); }
    h1 { color: #38bdf8; text-align: center; margin-bottom: 5px; font-size: 28px; }
    .subtitle { text-align: center; color: #94a3b8; margin-bottom: 30px; font-size: 14px; }
    
    /* Grafik qismi */
    .chart-container { width: 100%; height: 350px; margin-bottom: 40px; background: #0f172a; padding: 15px; border-radius: 8px; box-shadow: inset 0 2px 8px rgba(0,0,0,0.5); }
    
    /* Jadval dizayni */
    table { width: 100%; border-collapse: collapse; margin-top: 20px; background: #1e293b; overflow: hidden; border-radius: 8px; }
    th, td { border: 1px solid #334155; padding: 12px; text-align: center; }
    th { background: #0284c7; color: white; font-weight: 600; font-size: 14px; text-transform: uppercase; letter-spacing: 0.5px; }
    
    /* Rangli holatlar va qatorlar */
    .row-success { background-color: rgba(34, 197, 94, 0.15) !important; }
    .row-danger { background-color: rgba(239, 68, 68, 0.15) !important; }
    .row-normal { background-color: #1e293b; }
    tr:nth-child(even).row-normal { background-color: #1e293b; opacity: 0.9; }
    
    /* Tugmalar va interfeys elementlari */
    select { background: #0f172a; color: #f8fafc; border: 1px solid #475569; padding: 6px 10px; border-radius: 6px; font-weight: bold; cursor: pointer; outline: none; }
    select:focus { border-color: #38bdf8; }
    
    .status-badge { padding: 4px 8px; border-radius: 6px; font-size: 12px; font-weight: bold; }
    .badge-pending { background: #475569; color: #cbd5e1; }
    .badge-success { background: #22c55e; color: white; }
    .badge-danger { background: #ef4444; color: white; }
    
    input[type="number"] { background: #0f172a; color: #f8fafc; border: 1px solid #475569; padding: 5px; width: 70px; text-align: center; border-radius: 4px; font-weight: bold; }
    
    .text-highlight { color: #38bdf8; font-weight: bold; }
    .text-profit { color: #4ade80; font-weight: bold; }
    .text-loss { color: #f87171; font-weight: bold; }
  </style>
</head>
<body>

<div class="container">
  <h1>📈 6 OY DAWAMINDA: Trading Plan & Level Tracker</h1>
  <div class="subtitle">Boshlang'ich balans: $100 | Har bir darajada xavf (Risk): 5% | Maqsadlar Excel formulasiga asosan dinamik hisoblanadi</div>
  
  <div class="chart-container">
    <canvas id="planChart"></canvas>
  </div>

  <table id="planTable">
    <thead>
      <tr>
        <th>Level</th>
        <th>Boshlang'ich Balans ($)</th>
        <th>Risk (%)</th>
        <th>Risk ($)</th>
        <th>Foyda Maqsadi ($)</th>
        <th>Pips Target</th>
        <th>Tavsiya Lot</th>
        <th>Amaldagi Natija (%)</th>
        <th>Yopilish Balansi ($)</th>
        <th>Daraja Statusi</th>
      </tr>
    </thead>
    <tbody id="tableBody">
      <!-- JavaScript qatorlarni avtomatik to'ldiradi -->
    </tbody>
  </table>
</div>

<script>
  const totalLevels = 40; // Rejadagi jami darajalar soni
  let currentStatus = new Array(totalLevels).fill('pending'); // pending, success, loss
  let userAdjustments = new Array(totalLevels).fill(0); // Qo'lda kiritilgan maxsus foizlar (agar standartdan farq qilsa)
  let initialBalance = 100.00;

  let computedBalances = [];

  function calculatePlan() {
    computedBalances = [];
    let activeBalance = initialBalance;

    for (let i = 0; i < totalLevels; i++) {
      let levelStart = activeBalance;
      let riskPercent = 5; // Standart 5% risk
      let riskAmount = levelStart * (riskPercent / 100);
      
      // Standart maqsad: darajaga nisbatan o'sish sur'ati
      // Excelingizdagi 1-qator qoidasi (100$ -> 8$ profit -> 108$ goal)
      let standardProfitGoal = i === 0 ? 8.00 : levelStart * 0.08; 
      if (i === 2) standardProfitGoal = levelStart * 0.10; // Excel maxsus moslashuvi
      
      let pips = standardProfitGoal / 2;
      let lot = (riskAmount / pips) / 10;
      if(lot < 0.01) lot = 0.01;

      // Amaldagi natijani hisoblash
      let finalResultPercent = 0;
      if (currentStatus[i] === 'success') {
        // Agar muvaffaqiyatli bo'lsa, Excel bo'yicha balans o'sadi
        finalResultPercent = (standardProfitGoal / levelStart) * 100;
      } else if (currentStatus[i] === 'loss') {
        // Agar daraja zarar bilan yopilsa, foydalanuvchi kiritgan minus foiz yoki standart -5% risk olinadi
        finalResultPercent = userAdjustments[i] < 0 ? userAdjustments[i] : -riskPercent;
      } else {
        // Kutilmoqda rejimi - kelajak reja sifatida standart o'sishni ko'rsatib turadi
        finalResultPercent = (standardProfitGoal / levelStart) * 100;
      }

      let levelEnd = levelStart + (levelStart * (finalResultPercent / 100));
      if (levelEnd < 0) levelEnd = 0;

      computedBalances.push({
        level: i + 1,
        start: levelStart,
        riskP: riskPercent,
        riskA: riskAmount,
        goal: standardProfitGoal,
        pips: pips,
        lot: lot,
        actualP: currentStatus[i] === 'pending' ? 0 : finalResultPercent,
        end: levelEnd
      });

      // Zanjir davom etadi: keyingi daraja shu kunning yopilishidan boshlanadi
      activeBalance = levelEnd;
    }

    renderTable();
    updateChart();
  }

  function renderTable() {
    const tbody = document.getElementById('tableBody');
    tbody.innerHTML = '';

    for (let i = 0; i < totalLevels; i++) {
      let d = computedBalances[i];
      let row = document.createElement('tr');
      
      // Ranglar va status ko'rinishi
      let statusSelectHTML = `
        <select onchange="updateStatus(${i}, this.value)">
          <option value="pending" ${currentStatus[i] === 'pending' ? 'selected' : ''}>Kutilmoqda ⏳</option>
          <option value="success" ${currentStatus[i] === 'success' ? 'selected' : ''}>Bajarildi ✅</option>
          <option value="loss" ${currentStatus[i] === 'loss' ? 'selected' : ''}>Zarar (Minus) ❌</option>
        </select>
      `;

      if (currentStatus[i] === 'success') {
        row.className = 'row-success';
      } else if (currentStatus[i] === 'loss') {
        row.className = 'row-danger';
      } else {
        row.className = 'row-normal';
      }

      // Agar zarar bo'lsa, qo'lda minus foiz kiritish maydoni ochiladi
      let actualResultHTML = '';
      if (currentStatus[i] === 'loss') {
        let val = userAdjustments[i] !== 0 ? userAdjustments[i] : -5;
        actualResultHTML = `<input type="number" step="0.1" value="${val}" onchange="updateCustomPercent(${i}, this.value)">%`;
      } else if (currentStatus[i] === 'success') {
        actualResultHTML = `<span class="text-profit">+${d.actualP.toFixed(2)}%</span>`;
      } else {
        actualResultHTML = `<span style="color:#64748b;">0.00%</span>`;
      }

      row.innerHTML = `
        <td><b>${d.level}</b></td>
        <td>$${d.start.toFixed(2)}</td>
        <td>${d.riskP}%</td>
        <td>$${d.riskA.toFixed(2)}</td>
        <td class="text-highlight">$${d.goal.toFixed(2)}</td>
        <td>${d.pips.toFixed(2)}</td>
        <td style="color:#f59e0b; font-weight:bold;">${d.lot.toFixed(4)}</td>
        <td>${actualResultHTML}</td>
        <td><b>$${d.end.toFixed(2)}</b></td>
        <td>${statusSelectHTML}</td>
      `;
      tbody.appendChild(row);
    }
  }

  function updateStatus(index, value) {
    currentStatus[index] = value;
    if (value !== 'loss') userAdjustments[index] = 0; // Reset custom percent if not loss
    calculatePlan();
  }

  function updateCustomPercent(index, value) {
    let val = parseFloat(value) || 0;
    // Agarda foydalanuvchi minus belgisini qo'ymasa, avtomatik minus qilamiz
    if (val > 0) val = -val;
    userAdjustments[index] = val;
    calculatePlan();
  }

  // Chart.js Grafik sozlamalari (To'q qora trading uslubida)
  const ctx = document.getElementById('planChart').getContext('2d');
  let planChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: Array.from({length: totalLevels}, (_, i) => `Lvl ${i+1}`),
      datasets: [{
        label: 'Balans O\'sish Trayektoriyasi ($)',
        data: [],
        borderColor: '#38bdf8',
        backgroundColor: 'rgba(56, 189, 248, 0.08)',
        borderWidth: 2.5,
        fill: true,
        tension: 0.2,
        pointBackgroundColor: '#38bdf8'
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: { labels: { color: '#f8fafc' } }
      },
      scales: {
        x: { grid: { color: '#334155' }, ticks: { color: '#94a3b8' } },
        y: { grid: { color: '#334155' }, ticks: { color: '#94a3b8' }, beginAtZero: false }
      }
    }
  });

  function updateChart() {
    planChart.data.datasets[0].data = computedBalances.map(d => d.end);
    planChart.update();
  }

  // Ilk hisob-kitobni ishga tushirish
  calculatePlan();
</script>
</body>
</html>
