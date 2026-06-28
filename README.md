<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>6 Oylik Trading Plan & Tracker</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f4f6f9; margin: 0; padding: 20px; color: #333; }
    .container { max-width: 1250px; margin: auto; background: white; padding: 25px; border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
    h1 { color: #1e3c72; text-align: center; margin-bottom: 5px; font-size: 26px; }
    .subtitle { text-align: center; color: #666; margin-bottom: 30px; font-size: 14px; }
    
    .chart-container { width: 100%; height: 340px; margin-bottom: 40px; }
    
    table { width: 100%; border-collapse: collapse; margin-top: 20px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
    th, td { border: 1px solid #e0e0e0; padding: 11px 10px; text-align: center; font-size: 14px; }
    th { background: #1e3c72; color: white; font-weight: 600; position: sticky; top: 0; z-index: 10; }
    tr:nth-child(even) { background: #f9fbfd; }
    
    input[type="number"] { width: 65px; padding: 5px; text-align: center; border: 1px solid #ccc; border-radius: 4px; font-weight: bold; }
    select { padding: 6px 10px; border-radius: 20px; font-weight: bold; cursor: pointer; border: 1px solid #ccc; outline: none; }
    
    /* Status ranglari */
    .status-pending { background: #e0e0e0; color: #555; }
    .status-success { background: #28a745; color: white; }
    .status-warning { background: #ffc107; color: #212529; }
    .status-danger { background: #dc3545; color: white; }

    .profit-row { background-color: #e6f4ea !important; }
    .warning-row { background-color: #fffde7 !important; }
    .loss-row { background-color: #fce8e6 !important; }
    .pending-row { background-color: white; }
    
    .text-highlight { color: #0284c7; font-weight: bold; }
    .reset-btn { background: #dc3545; color: white; border: none; padding: 8px 15px; border-radius: 5px; font-weight: bold; cursor: pointer; margin-bottom: 15px; float: right; }
    .reset-btn:hover { background: #bd2130; }
  </style>
</head>
<body>

<div class="container">
  <button class="reset-btn" onclick="resetAllData()">Rejani Tozalash (Reset) 🔄</button>
  <h1>🚀 6 OY DAWAMINDA: Trading Plan & Level Tracker</h1>
  <div class="subtitle">Har bir kunning holatini va foizini o'zingiz ruchnoy belgilaysiz | O'zgarishlar brauzerda eslab qolinadi</div>
  
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
  
  function getExcelTargetPercent(day) {
    if (day === 1 || day === 2) return 8.0;
    if (day >= 3 && day <= 25) return 10.0;
    if (day >= 26 && day <= 64) return 6.0;
    if (day >= 65 && day <= 81) return 4.0;
    return 2.0;
  }

  function getExcelRiskPercent(day) {
    if (day <= 25) return 5.0;
    if (day >= 26 && day <= 64) return 3.0;
    if (day >= 65 && day <= 81) return 2.0;
    return 1.0;
  }

  // --- BRAUZER XOTIRASIDAN STATUS VA FOIZLARNI YUKLASH ---
  let userStatus = [];
  let userProfits = [];

  let savedStatus = localStorage.getItem('tr_tracker_status_150');
  let savedProfits = localStorage.getItem('tr_tracker_profits_150');

  if (savedStatus && savedProfits) {
    userStatus = JSON.parse(savedStatus);
    userProfits = JSON.parse(savedProfits);
  } else {
    for (let d = 1; d <= totalLevels; d++) {
      userStatus.push('pending'); // Boshida hamma kun "Kutilmoqda" bo'ladi
      userProfits.push(0);        // Boshida foyda 0% bo'ladi
    }
  }
  
  let balances = new Array(totalLevels).fill(0);
  balances[0] = initialBalance;

  function calculateData() {
    for (let i = 0; i < totalLevels; i++) {
      let startBal = balances[i];
      let pPercent = userProfits[i];
      
      let endBal = startBal + (startBal * (pPercent / 100));
      if (endBal < 0) endBal = 0;

      if (i < totalLevels - 1) {
        balances[i+1] = endBal;
      }
    }
    
    // Xotiraga saqlash
    localStorage.setItem('tr_tracker_status_150', JSON.stringify(userStatus));
    localStorage.setItem('tr_tracker_profits_150', JSON.stringify(userProfits));
    
    renderTable();
    updateChart();
  }

  function renderTable() {
    const tbody = document.getElementById('tableBody');
    tbody.innerHTML = '';
    
    for (let i = 0; i < totalLevels; i++) {
      let currentDay = i + 1;
      let startBal = balances[i];
      let pPercent = userProfits[i];
      let status = userStatus[i];
      
      let riskPercent = getExcelRiskPercent(currentDay);
      let riskAmount = startBal * (riskPercent / 100);
      let targetPercent = getExcelTargetPercent(currentDay);
      let standardGoal = startBal * (targetPercent / 100); 

      let endBal = startBal + (startBal * (pPercent / 100));
      if (endBal < 0) endBal = 0;
      
      let row = document.createElement('tr');
      
      // Qator rangini statusga qarab belgilash
      if (status === 'success') row.className = 'profit-row';
      else if (status === 'warning') row.className = 'warning-row';
      else if (status === 'loss') row.className = 'loss-row';
      else row.className = 'pending-row';

      // Drop-down menyu yaratish
      let selectHTML = `
        <select class="status-${status}" onchange="updateDayStatus(${i}, this.value)">
          <option value="pending" ${status === 'pending' ? 'selected' : ''}>Kutilmoqda ⏳</option>
          <option value="success" ${status === 'success' ? 'selected' : ''}>Bajarildi ✅</option>
          <option value="warning" ${status === 'warning' ? 'selected' : ''}>Kam Foyda ⚠️</option>
          <option value="loss" ${status === 'loss' ? 'selected' : ''}>Zarar (Minus) ❌</option>
        </select>
      `;

      row.innerHTML = `
        <td><b>${currentDay}</b></td>
        <td>$${startBal.toFixed(2)}</td>
        <td>$${riskAmount.toFixed(2)}</td>
        <td class="text-highlight">$${standardGoal.toFixed(2)}</td>
        <td><input type="number" step="0.1" value="${pPercent}" onchange="updateProfit(${i}, this.value)">%</td>
        <td><b>$${endBal.toFixed(2)}</b></td>
        <td>${selectHTML}</td>
      `;
      tbody.appendChild(row);
    }
  }

  function updateProfit(index, value) {
    userProfits[index] = parseFloat(value) || 0;
    calculateData();
  }

  function updateDayStatus(index, statusValue) {
    userStatus[index] = statusValue;
    let currentDay = index + 1;
    
    // Agar foydalanuvchi "Bajarildi" deb tanlasa va foiz o'zgarmagan bo'lsa, avtomatik Excel foizini yozadi
    if (statusValue === 'success' && userProfits[index] === 0) {
      userProfits[index] = getExcelTargetPercent(currentDay);
    } else if (statusValue === 'loss' && userProfits[index] >= 0) {
      userProfits[index] = -getExcelRiskPercent(currentDay); // Avtomatik minus risk foizi
    } else if (statusValue === 'pending') {
      userProfits[index] = 0;
    }
    calculateData();
  }

  function resetAllData() {
    if (confirm("Barcha kiritilgan ruchnoy ma'lumotlarni o'chirib, toza holatga keltirmoqchimisiz?")) {
      localStorage.removeItem('tr_tracker_status_150');
      localStorage.removeItem('tr_tracker_profits_150');
      userStatus = [];
      userProfits = [];
      for (let d = 1; d <= totalLevels; d++) {
        userStatus.push('pending');
        userProfits.push(0);
      }
      calculateData();
    }
  }

  const ctx = document.getElementById('planChart').getContext('2d');
  let planChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: Array.from({length: totalLevels}, (_, i) => `${i+1}`),
      datasets: [{
        label: 'Real Balans O\'sishi ($)',
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
      let cBal = balances[i] + (balances[i] * (userProfits[i]/100));
      closingBalances.push(cBal < 0 ? 0 : cBal);
    }
    planChart.data.datasets[0].data = closingBalances;
    planChart.update();
  }

  calculateData();
</script>
</body>
</html>
