<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>6 Oylik Trading Plan & Tracker</title>
  <!-- Grafik chizish uchun Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f4f6f9; margin: 0; padding: 20px; color: #333; }
    .container { max-width: 1300px; margin: auto; background: white; padding: 25px; border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
    h1 { color: #1e3c72; text-align: center; margin-bottom: 5px; font-size: 26px; }
    .subtitle { text-align: center; color: #666; margin-bottom: 30px; font-size: 14px; }
    
    /* Grafik qismi */
    .chart-container { width: 100%; height: 320px; margin-bottom: 40px; }
    
    /* Jadval dizayni (Och rangli interfeys) */
    table { width: 100%; border-collapse: collapse; margin-top: 20px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
    th, td { border: 1px solid #e0e0e0; padding: 12px; text-align: center; }
    th { background: #1e3c72; color: white; font-weight: 600; font-size: 14px; }
    tr:nth-child(even) { background: #f9fbfd; }
    
    /* Foiz kiritish maydoni (Input) */
    input[type="number"] { width: 70px; padding: 6px; text-align: center; border: 1px solid #ccc; border-radius: 4px; font-weight: bold; }
    
    /* Dinamik Rangli Status Tugmalari */
    .status-badge { padding: 6px 15px; border-radius: 20px; font-size: 13px; font-weight: bold; display: inline-block; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
    .status-success { background-color: #28a745; color: white; }
    .status-warning { background-color: #ffc107; color: #212529; }
    .status-danger { background-color: #dc3545; color: white; }

    /* Qator ranglari */
    .profit-row { background-color: #e6f4ea !important; }
    .warning-row { background-color: #fffde7 !important; }
    .loss-row { background-color: #fce8e6 !important; }
    
    .text-highlight { color: #0284c7; font-weight: bold; }
    .text-profit { color: #137333; font-weight: bold; }
    .text-loss { color: #c5221f; font-weight: bold; }
  </style>
</head>
<body>

<div class="container">
  <h1>🚀 6 OY DAWAMINDA: Trading Plan & Level Tracker</h1>
  <div class="subtitle">Boshlang'ich balans: $100 | Har bir darajada xavf (Risk): 5% | Kunlik foyda foizini qo'lda o'zgartirishingiz mumkin</div>
  
  <div class="chart-container">
    <canvas id="planChart"></canvas>
  </div>

  <table id="planTable">
    <thead>
      <tr>
        <th>Kun / Level</th>
        <th>Boshlang'ich Balans ($)</th>
        <th>Xavf (%)</th>
        <th>Xavf ($)</th>
        <th>Foyda Maqsadi ($)</th>
        <th>Pips Maqsadi</th>
        <th>Tavsiya Lot</th>
        <th>Kunlik Foyda / Zarar (%)</th>
        <th>Yopilish Balansi ($)</th>
        <th>Kunlik Status (Holat)</th>
      </tr>
    </thead>
    <tbody id="tableBody">
      <!-- JavaScript qatorlarni avtomatik yaratadi -->
    </tbody>
  </table>
</div>

<script>
  const totalLevels = 40; // Rejadagi jami kunlar/bosqichlar soni
  let initialBalance = 100.00;
  
  // Standart boshlang'ich qiymat: har kuni +8% maqsad (Excel formulasiga ko'ra)
  let profits = new Array(totalLevels).fill(8); 
  let balances = new Array(totalLevels).fill(0);
  balances[0] = initialBalance;

  function calculateData() {
    for (let i = 0; i < totalLevels; i++) {
      let startBal = balances[i];
      let pPercent = profits[i];
      
      // Foiz hisob-kitoblari
      let endBal = startBal + (startBal * (pPercent / 100));
      if (endBal < 0) endBal = 0;

      if (i < totalLevels - 1) {
        balances[i+1] = endBal;
      }
    }
    renderTable();
    updateChart();
  }

  function renderTable() {
    const tbody = document.getElementById('tableBody');
    tbody.innerHTML = '';
    
    for (let i = 0; i < totalLevels; i++) {
      let startBal = balances[i];
      let pPercent = profits[i];
      
      // Excel formulalariga asoslangan ustunlar:
      let riskPercent = 5; 
      let riskAmount = startBal * (riskPercent / 100);
      let standardGoal = startBal * 0.08; 
      let pips = standardGoal / 2;
      let lot = startBal / 10000; // Excel: STARTING BALANCE / 10000

      let endBal = startBal + (startBal * (pPercent / 100));
      if (endBal < 0) endBal = 0;
      
      let row = document.createElement('tr');
      let statusHTML = '';
      
      // Foiz ko'rsatkichiga qarab dinamik status va qator ranglari
      if (pPercent >= 8) {
        row.className = 'profit-row';
        statusHTML = '<span class="status-badge status-success">Maqsad Bajarildi ✅</span>';
      } else if (pPercent > 0 && pPercent < 8) {
        row.className = 'warning-row';
        statusHTML = '<span class="status-badge status-warning">Kam Foyda ⚠️</span>';
      } else if (pPercent === 0) {
        statusHTML = '<span class="status-badge" style="background:#e0e0e0; color:#555;">Savdo Bo\'lmadi</span>';
      } else {
        row.className = 'loss-row';
        statusHTML = '<span class="status-badge status-danger">Zarar Ko\'rildi ❌</span>';
      }

      row.innerHTML = `
        <td><b>${i+1}</b></td>
        <td>$${startBal.toFixed(2)}</td>
        <td>${riskPercent}%</td>
        <td>$${riskAmount.toFixed(2)}</td>
        <td class="text-highlight">$${standardGoal.toFixed(2)}</td>
        <td>${pips.toFixed(2)}</td>
        <td style="color:#e67e22; font-weight:bold;">${lot.toFixed(4)}</td>
        <td><input type="number" step="0.1" value="${pPercent}" onchange="updateProfit(${i}, this.value)">%</td>
        <td><b>$${endBal.toFixed(2)}</b></td>
        <td>${statusHTML}</td>
      `;
      tbody.appendChild(row);
    }
  }

  function updateProfit(index, value) {
    profits[index] = parseFloat(value) || 0;
    calculateData();
  }

  // Och rangli chiroyli grafik sozlamalari
  const ctx = document.getElementById('planChart').getContext('2d');
  let planChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: Array.from({length: totalLevels}, (_, i) => `${i+1}-kun`),
      datasets: [{
        label: 'Balans ($)',
        data: [],
        borderColor: '#1e3c72',
        backgroundColor: 'rgba(30, 60, 114, 0.05)',
        borderWidth: 2.5,
        fill: true,
        tension: 0.2
      }]
    },
    options: { 
      responsive: true, 
      maintainAspectRatio: false,
      scales: { y: { beginAtZero: false } }
    }
  });

  function updateChart() {
    let closingBalances = [];
    for(let i=0; i<totalLevels; i++) {
      let cBal = balances[i] + (balances[i] * (profits[i]/100));
      closingBalances.push(cBal < 0 ? 0 : cBal);
    }
    planChart.data.datasets[0].data = closingBalances;
    planChart.update();
  }

  calculateData();
</script>
</body>
</html>
