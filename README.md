# Calibration-tool
رسم منحنی کالیبراسیون و محاسبه R2
<!DOCTYPE html>
<html lang="fa">
<head>
  <meta charset="UTF-8">
  <title>منحنی کالیبراسیون</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/regression@2.0.1/dist/regression.min.js"></script>
  <style>
    body { font-family: sans-serif; direction: rtl; background-color: #f4f6f8; padding: 20px; text-align: center; }
    input, button { padding: 10px; margin: 5px; font-size: 16px; border: 1px solid #ccc; border-radius: 6px; }
    button { background-color: #007bff; color: white; cursor: pointer; }
    button:hover { background-color: #0056b3; }
    #chart-container { width: 100%; max-width: 600px; margin: 30px auto; }
  </style>
</head>
<body>
  <h2>🔬 رسم منحنی کالیبراسیون و محاسبه R²</h2>

  <div>
    <input type="text" id="concentration" placeholder="غلظت (ppm)">
    <input type="text" id="absorbance" placeholder="جذب (Abs)">
    <button onclick="addPoint()">➕ افزودن</button>
    <button onclick="resetChart()">🗑 حذف</button>
  </div>

  <div id="chart-container">
    <canvas id="calibrationChart"></canvas>
  </div>

  <p id="equationOutput"></p>

  <script>
    const points = [];
    const ctx = document.getElementById("calibrationChart").getContext("2d");

    const chart = new Chart(ctx, {
      type: "scatter",
      data: {
        datasets: [{
          label: "نقاط آزمایش",
          data: [],
          backgroundColor: "#ff6384"
        }]
      },
      options: {
        scales: {
          x: { title: { display: true, text: "غلظت (ppm)" } },
          y: { title: { display: true, text: "جذب (Abs)" } }
        }
      }
    });

    function addPoint() {
      const conc = parseFloat(document.getElementById("concentration").value);
      const abs = parseFloat(document.getElementById("absorbance").value);
      if (!isNaN(conc) && !isNaN(abs)) {
        points.push([conc, abs]);
        document.getElementById("concentration").value = "";
        document.getElementById("absorbance").value = "";
        updateChart();
      }
    }

    function resetChart() {
      points.length = 0;
      chart.data.datasets[0].data = [];
      if (chart.data.datasets[1]) chart.data.datasets.pop();
      chart.update();
      document.getElementById("equationOutput").innerText = "";
    }

    function updateChart() {
      chart.data.datasets[0].data = points.map(p => ({ x: p[0], y: p[1] }));

      if (points.length >= 2) {
        const result = regression.linear(points);
        const line = [
          { x: Math.min(...points.map(p => p[0])), y: result.predict(Math.min(...points.map(p => p[0])))[1] },
          { x: Math.max(...points.map(p => p[0])), y: result.predict(Math.max(...points.map(p => p[0])))[1] }
        ];

        const lineDataset = {
          label: "خط کالیبراسیون",
          data: line,
          type: "line",
          borderColor: "#28a745",
          borderWidth: 2,
          fill: false
        };

        if (chart.data.datasets.length > 1) {
          chart.data.datasets[1] = lineDataset;
        } else {
          chart.data.datasets.push(lineDataset);
        }

        chart.update();

        document.getElementById("equationOutput").innerText =
          y = ${result.equation[0].toFixed(4)}x + ${result.equation[1].toFixed(4)} | R² = ${result.r2.toFixed(4)};
      }

      chart.update();
    }
  </script>
</body>
</html>
