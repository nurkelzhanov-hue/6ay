<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>6 Oylik Trading Plan & Tracker</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f4f6f9; margin: 0; padding: 20px; color: #333; }
    .container { max-width: 1200px; margin: auto; background: white; padding: 25px; border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
    h1 { color: #1e3c72; text-align: center; margin-bottom: 5px; font-size: 26px; }
    .subtitle { text-align: center; color: #666; margin-bottom: 30px; font-size: 14px; }
    
    .chart-container { width: 100%; height: 340px; margin-bottom: 40px; }
    
    table { width: 100%; border-collapse: collapse; margin-top: 20px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
    th, td { border: 1px solid #e0e0e0; padding: 11px 10px; text-align: center; font-size: 14px; }
    th { background: #1e3c72; color: white; font-weight: 600; position: sticky; top: 0; z-index: 10; }
    tr:nth-child(even) { background: #f9fbfd; }
    
    input[type="number"] { width: 65px; padding: 5px; text-align: center; border: 1px solid #ccc; border-radius: 4px; font-weight: bold; }
    
    .status-badge { padding: 5px 12px; border-radius: 20px; font-size: 12px; font-weight: bold; display: inline-block; }
    .status-success { background-color: #28a745; color: white; }
    .status-warning { background-color: #ffc107; color: #212529; }
    .status-danger { background-color: #dc3545; color: white; }

    .profit-row { background-color: #e6f4ea !important; }
    .warning-row { background-color: #fffde7 !important; }
    .loss-row { background-color: #fce8e6 !important; }
    
    .text-highlight { color: #0284c7; font-weight: bold; }
    .text-profit { color: #137333; font-weight: bold; }
  </style>
</head>
<body>

<div class="container">
  <h1>🚀 6 OY DAWAMINDA: Trading Plan & Level Tracker</h1>
  <div class="subtitle">Boshlang'ich balans: $100 | Jami: 150 Savdo Kuni | Risk va Maqsadlar 100% Excel andozasida</div>
  
  <div class="chart-container">
    <canvas id="planChart"></canvas>
  </div>

  <table id="planTable">
    <thead>
      <tr>
        <th>Kun / Level</th>
        <th>Boshlang'ich Balans ($)</th>
        <th>Xavf ($)</th>
        <th>Foyda Maqsadi ($)</th>
        <th>Kunlik Foyda / Zarar (%)</th>
        <th>Yopilish Balansi ($)</th>
        <th>Kunlik Status (Holat)</th>
      </tr>
    </thead>
    <tbody id="tableBody">
      </tbody>
  </table>
</div>

<script>
  const totalLevels = 150;
  let initialBalance = 100.00;
  
  // Excel bo'yicha maqsad foizlari
  function getExcelTargetPercent(day) {
    if (day === 1 || day === 2) return 8.0;
    if (day >= 3 && day <= 25) return 10.0;
    if (day >= 26 && day <= 64) return 6.0;
    if (day >= 65 && day <= 81) return 4.0;
    return 2.0;
  }

  // Excel bo'yicha risk foizlari
  function getExcelRiskPercent(day) {
    if (day <= 25) return 5.0;
    if (day >= 26 && day <= 64) return 3.0;
    if (day >= 65 && day <= 81) return 2.0;
    return 1.0;
  }

  let profits = [];
  for (let d = 1; d <= totalLevels; d++) {
    profits.push(getExcelTargetPercent(d));
  }
  
  let balances = new Array(totalLevels).fill(0);
  balances[0] = initialBalance;

  function calculateData() {
    for (let i = 0; i < totalLevels; i++) {
      let startBal = balances[i];
      let pPercent = profits[i];
      
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
      let currentDay = i + 1;
      let startBal = balances[i];
      let pPercent = profits[i];
      
      let riskPercent = getExcelRiskPercent(currentDay);
      let riskAmount = startBal * (riskPercent / 100);
      let targetPercent = getExcelTargetPercent(currentDay);
      
      let standardGoal = startBal * (targetPercent / 100); 

      let endBal = startBal + (startBal * (pPercent / 100));
      if (endBal < 0) endBal = 0;
      
      let row = document.createElement('tr');
      let statusHTML = '';
      
      if (pPercent >= targetPercent) {
        row.className = 'profit-row';
        statusHTML = '<span class="status-badge status-success">Maqsad Bajarildi ✅</span>';
      } else if (pPercent > 0 && pPercent < targetPercent) {
        row.className = 'warning-row';
        statusHTML = '<span class="status-badge status-warning">Kam Foyda ⚠️</span>';
      } else if (pPercent === 0) {
        statusHTML = '<span class="status-badge" style="background:#e0e0e0; color:#555;">Kutilmoqda ⏳</span>';
      } else {
        row.className = 'loss-row';
        statusHTML = '<span class="status-badge status-danger">Zarar Ko\'rildi ❌</span>';
      }

      row.innerHTML = `
        <td><b>${currentDay}</b></td>
        <td>$${startBal.toFixed(2)}</td>
        <td>$${riskAmount.toFixed(2)}</td>
        <td class="text-highlight">$${standardGoal.toFixed(2)}</td>
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

  const ctx = document.getElementById('planChart').getContext('2d');
  let planChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: Array.from({length: totalLevels}, (_, i) => `${i+1}`),
      datasets: [{
        label: '6 Oylik Trayektoriya ($)',
        data: [],
        borderColor: '#1e3c72',
        backgroundColor: 'rgba(30, 60, 114, 0.04)',
        borderWidth: 2,
        fill: true,
        tension: 0.1,
        pointRadius: 1
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
