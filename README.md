<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Daily Meal Tracker</title>
  <style>
    body {
      font-family: "Segoe UI", Tahoma, sans-serif;
      margin: 0;
      background: #f4f6f9;
      color: #333;
    }
    h1 {
      text-align: center;
      padding: 20px;
      margin: 0;
      background: #0078d7;
      color: white;
      font-weight: 600;
    }
    .section {
      margin: 20px auto;
      padding: 20px;
      background: #fff;
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.05);
      max-width: 800px;
    }
    .section h2 {
      margin-top: 0;
      font-size: 20px;
      color: #0078d7;
      border-bottom: 2px solid #eee;
      padding-bottom: 8px;
    }
    label {
      display: block;
      margin: 10px 0;
      font-weight: 500;
    }
    input[type="number"], input[type="date"] {
      padding: 8px;
      margin-top: 5px;
      width: 200px;
      border: 1px solid #ccc;
      border-radius: 5px;
      font-size: 14px;
    }
    input[type="checkbox"] {
      margin-right: 8px;
    }
    button {
      padding: 10px 16px;
      background: #0078d7;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 14px;
      margin-top: 10px;
      transition: background 0.3s ease;
    }
    button:hover {
      background: #005fa3;
    }
    .delete-btn {
      background: #d9534f;
      font-size: 13px;
      padding: 6px 12px;
    }
    .delete-btn:hover {
      background: #b52b27;
    }
    #summary {
      margin: 20px auto;
      font-size: 20px;
      font-weight: bold;
      text-align: center;
      color: #444;
    }
    table {
      width: 90%;
      margin: 20px auto;
      border-collapse: collapse;
      background: #fff;
      border-radius: 10px;
      overflow: hidden;
      box-shadow: 0 4px 10px rgba(0,0,0,0.05);
    }
    th, td {
      border: 1px solid #eee;
      padding: 12px;
      text-align: center;
      font-size: 14px;
    }
    th {
      background: #0078d7;
      color: white;
      font-weight: 600;
    }
    tr:nth-child(even) {
      background: #f9f9f9;
    }
  </style>
</head>
<body>
  <h1>Daily Meal Expense Tracker</h1>

  <div class="section">
    <h2>Default Prices</h2>
    <label>Breakfast: <input type="number" id="defaultBreakfast"></label>
    <label>Lunch: <input type="number" id="defaultLunch"></label>
    <label>Dinner: <input type="number" id="defaultDinner"></label>
    <button onclick="saveDefaults()">Save Defaults</button>
  </div>

  <div class="section">
    <h2>Select Date</h2>
    <input type="date" id="mealDate">
  </div>

  <div class="section">
    <h2>Meals Consumed</h2>
    <label><input type="checkbox" id="breakfastCheck"> Breakfast</label>
    <label><input type="checkbox" id="lunchCheck"> Lunch</label>
    <label><input type="checkbox" id="dinnerCheck"> Dinner</label>
    <button onclick="saveDay()">Save Day</button>
  </div>

  <div id="summary">Total Expense: Rs 0</div>

  <table id="mealTable">
    <thead>
      <tr>
        <th>Date</th>
        <th>Breakfast</th>
        <th>Lunch</th>
        <th>Dinner</th>
        <th>Total (Rs)</th>
        <th>Delete</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <div class="section">
    <h2>Calculate Total Between Dates</h2>
    <label>Start Date: <input type="date" id="startDate"></label>
    <label>End Date: <input type="date" id="endDate"></label>
    <button onclick="calculateRangeTotal()">Calculate</button>
    <p id="rangeSummary"></p>
  </div>

  <script>
    let defaults = JSON.parse(localStorage.getItem("defaults")) || {Breakfast:0, Lunch:0, Dinner:0};
    let days = JSON.parse(localStorage.getItem("days")) || [];

    function formatDate(isoDate) {
      let [year, month, day] = isoDate.split("-");
      return `${day}-${month}-${year.slice(2)}`;
    }

    function saveDefaults() {
      defaults.Breakfast = parseFloat(document.getElementById("defaultBreakfast").value) || 0;
      defaults.Lunch = parseFloat(document.getElementById("defaultLunch").value) || 0;
      defaults.Dinner = parseFloat(document.getElementById("defaultDinner").value) || 0;
      localStorage.setItem("defaults", JSON.stringify(defaults));
      alert("Defaults saved!");
    }

    function saveDay() {
      let date = document.getElementById("mealDate").value || new Date().toISOString().split('T')[0];
      let breakfast = document.getElementById("breakfastCheck").checked;
      let lunch = document.getElementById("lunchCheck").checked;
      let dinner = document.getElementById("dinnerCheck").checked;

      let total = 0;
      if (breakfast) total += defaults.Breakfast;
      if (lunch) total += defaults.Lunch;
      if (dinner) total += defaults.Dinner;

      days.push({ date, breakfast, lunch, dinner, total });
      localStorage.setItem("days", JSON.stringify(days));
      renderTable();
    }

    function renderTable() {
      let tbody = document.getElementById("mealTable").getElementsByTagName("tbody")[0];
      tbody.innerHTML = "";
      let grandTotal = 0;

      days.forEach((day, index) => {
        let row = tbody.insertRow();
        row.insertCell(0).innerText = formatDate(day.date);
        row.insertCell(1).innerText = day.breakfast ? "✅" : "❌";
        row.insertCell(2).innerText = day.lunch ? "✅" : "❌";
        row.insertCell(3).innerText = day.dinner ? "✅" : "❌";
        row.insertCell(4).innerText = "Rs " + day.total.toFixed(2);

        let deleteCell = row.insertCell(5);
        let deleteBtn = document.createElement("button");
        deleteBtn.innerText = "Delete";
        deleteBtn.className = "delete-btn";
        deleteBtn.onclick = function() {
          days.splice(index, 1);
          localStorage.setItem("days", JSON.stringify(days));
          renderTable();
        };
        deleteCell.appendChild(deleteBtn);

        grandTotal += day.total;
      });

      document.getElementById("summary").innerText = "Total Expense: Rs " + grandTotal.toFixed(2);
    }

    function calculateRangeTotal() {
      let start = document.getElementById("startDate").value;
      let end = document.getElementById("endDate").value;

      if (!start || !end) {
        document.getElementById("rangeSummary").innerText = "Please select both start and end dates.";
        return;
      }

      let totalRange = 0;
      let filtered = days.filter(day => day.date >= start && day.date <= end);
      filtered.forEach(day => totalRange += day.total);

      document.getElementById("rangeSummary").innerText =
        `Total from ${formatDate(start)} to ${formatDate(end)}: Rs ${totalRange.toFixed(2)}`;
    }

    document.getElementById("defaultBreakfast").value = defaults.Breakfast;
    document.getElementById("defaultLunch").value = defaults.Lunch;
    document.getElementById("defaultDinner").value = defaults.Dinner;

    renderTable();
  </script>
</body>
</html>
