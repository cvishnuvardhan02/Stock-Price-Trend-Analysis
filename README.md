# Stock-Price-Trend-Analysis
This Stock Price Trend Analysis project visualizes stock data through dynamic charts, including line, bar, and area charts. It integrates a live stock price ticker that scrolls across the screen, showing real-time price changes. The app also features a query-based AI assistant and options to download charts as images or PDFs.

<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Stock Price Trend Analysis</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <style>
    body {
      background-color: #f0f4f8;
      transition: background-color 0.5s, color 0.5s;
    }

    .chat-box {
      max-height: 400px;
      overflow-y: auto;
    }

    .dark-mode {
      background-color: #1f2937;
      color: #f9fafb;
    }

    .dark-mode input,
    .dark-mode select,
    .dark-mode button {
      color: #111827;
    }

    .stock-ticker {
      display: flex;
      overflow-x: auto;
      white-space: nowrap;
      padding: 10px;
      background-color: #1f2937;
      color: #f9fafb;
      font-size: 18px;
      font-family: sans-serif;
      margin-top: 20px;
    }

    .stock-ticker div {
      margin-right: 50px;
      padding: 5px 10px;
      border-radius: 4px;
    }

    .stock-ticker .up {
      background-color: #4CAF50; /* Green for up */
    }

    .stock-ticker .down {
      background-color: #F44336; /* Red for down */
    }
  </style>
</head>

<body class="font-sans transition-all duration-500">
  <div class="max-w-5xl mx-auto p-6">
    <h1 class="text-4xl font-bold mb-4 text-center text-blue-700">📈 Stock Price Trend Analysis</h1>
    <div class="mb-4 flex flex-wrap gap-2">
      <input id="stockInput" type="text" placeholder="Enter Stock Ticker (e.g. AAPL)" class="border p-2 w-64 rounded">
      <button onclick="fetchStockData()" class="bg-blue-600 text-white p-2 rounded hover:bg-blue-800 transition">Analyze</button>
      <select id="chartType" class="border p-2 rounded">
        <option value="line">Line Chart</option>
        <option value="bar">Bar Chart</option>
        <option value="area">Area Chart</option>
        <option value="radar">Radar Chart</option>
      </select>
      <button onclick="downloadImage()" class="bg-yellow-600 text-white p-2 rounded hover:bg-yellow-800 transition">Download Image</button>
      <button onclick="downloadPDF()" class="bg-purple-600 text-white p-2 rounded hover:bg-purple-800 transition">Export PDF</button>
      <button onclick="toggleDarkMode()" class="bg-gray-800 text-white p-2 rounded hover:bg-gray-900 transition">Toggle Dark Mode</button>
    </div>

    <canvas id="stockChart" class="mb-8"></canvas>

    <!-- Scrolling Stock Price Ticker -->
    <div class="stock-ticker" id="stockTicker"></div>

    <h2 class="text-2xl font-semibold mb-4">💬 Ask your Stock-related Queries</h2>
    <div class="border p-4 rounded bg-white chat-box mb-4 text-black" id="chatBox"></div>
    <div class="flex">
      <input id="userInput" type="text" placeholder="Type your question..." class="border p-2 flex-grow rounded-l">
      <button onclick="handleQuery()" class="bg-green-600 text-white p-2 rounded-r hover:bg-green-800 transition">Send</button>
    </div>

    <footer class="mt-10 text-center text-gray-500">&copy; 2025 Stock Trend AI</footer>
  </div>

  <script>
    let chart;
    let lastPrices = [];

    // Function to simulate stock data fetching
    async function fetchStockData() {
      const ticker = document.getElementById('stockInput').value.toUpperCase();
      if (!ticker) return alert('Please enter a stock ticker!');

      const type = document.getElementById('chartType').value;

      const labels = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri'];
      const prices = labels.map(() => (Math.random() * 100 + 100).toFixed(2));

      lastPrices = prices.map(Number);

      if (chart) chart.destroy();
      const ctx = document.getElementById('stockChart').getContext('2d');

      const commonOptions = {
        responsive: true,
        animation: { duration: 1500, easing: 'easeOutQuart' }
      };

      // Create chart based on selected type
      if (type === 'line') {
        chart = new Chart(ctx, {
          type: 'line',
          data: {
            labels: labels,
            datasets: [{
              label: `${ticker} Price`,
              data: prices,
              borderColor: 'rgb(75, 192, 192)',
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              fill: false,
              tension: 0.3
            }]
          },
          options: commonOptions
        });
      } else if (type === 'bar') {
        chart = new Chart(ctx, {
          type: 'bar',
          data: {
            labels: labels,
            datasets: [{
              label: `${ticker} Price`,
              data: prices,
              backgroundColor: 'rgba(54, 162, 235, 0.6)',
            }]
          },
          options: commonOptions
        });
      } else if (type === 'area') {
        chart = new Chart(ctx, {
          type: 'line',
          data: {
            labels: labels,
            datasets: [{
              label: `${ticker} Price`,
              data: prices,
              borderColor: 'rgb(153, 102, 255)',
              backgroundColor: 'rgba(153, 102, 255, 0.4)',
              fill: true,
              tension: 0.3
            }]
          },
          options: commonOptions
        });
      } else if (type === 'radar') {
        chart = new Chart(ctx, {
          type: 'radar',
          data: {
            labels: labels,
            datasets: [{
              label: `${ticker} Price`,
              data: prices,
              backgroundColor: 'rgba(255, 206, 86, 0.2)',
              borderColor: 'rgba(255, 206, 86, 1)',
              borderWidth: 1
            }]
          },
          options: commonOptions
        });
      }

      // Start updating the stock price ticker
      updateTicker(prices);
    }

    // Function to generate stock ticker with ups and downs
    function updateTicker(prices) {
      const tickerElement = document.getElementById('stockTicker');
      let prevPrice = parseFloat(prices[0]);
      prices.forEach(price => {
        const priceChange = (Math.random() > 0.5 ? 'up' : 'down');
        const priceElement = document.createElement('div');
        priceElement.classList.add(priceChange);
        priceElement.textContent = `Price: $${price} (${priceChange === 'up' ? '⬆️' : '⬇️'})`;
        tickerElement.appendChild(priceElement);
      });

      // Loop the ticker indefinitely
      setTimeout(() => {
        tickerElement.scrollLeft += 100;
        if (tickerElement.scrollLeft >= tickerElement.scrollWidth) {
          tickerElement.scrollLeft = 0;
        }
      }, 200);
    }

    // Function to handle user queries
    function handleQuery() {
      const userInput = document.getElementById('userInput').value;
      if (!userInput) return;
      const chatBox = document.getElementById('chatBox');
      chatBox.innerHTML += `<div class='mb-2'><strong>You:</strong> ${userInput}</div>`;
      const response = generateResponse(userInput);
      chatBox.innerHTML += `<div class='mb-2 text-blue-700'><strong>AI:</strong> ${response}</div>`;
      chatBox.scrollTop = chatBox.scrollHeight;
      document.getElementById('userInput').value = '';
    }

    // Function to generate responses for stock queries
    function generateResponse(input) {
      input = input.toLowerCase();
      if (!lastPrices.length) return "Please analyze a stock first.";
      if (input.includes('best time')) return "Historically, investing during market dips yields better long-term gains.";
      if (input.includes('moving average')) {
        let avg = (lastPrices.reduce((a, b) => a + b, 0) / lastPrices.length).toFixed(2);
        return `The moving average over the last 5 days is $${avg}.`;
      }
      if (input.includes('latest price')) return `The latest price is $${lastPrices[lastPrices.length - 1]}.`;
      if (input.includes('highest')) return `The highest price was $${Math.max(...lastPrices)}.`;
      if (input.includes('lowest')) return `The lowest price was $${Math.min(...lastPrices)}.`;
      if (input.includes('average')) {
        let avg = (lastPrices.reduce((a, b) => a + b, 0) / lastPrices.length).toFixed(2);
        return `The average price is $${avg}.`;
      }
      return "I couldn't find an answer to your query.";
    }

    // Function to download the chart as an image
    function downloadImage() {
      html2canvas(document.getElementById('stockChart')).then(canvas => {
        const link = document.createElement('a');
        link.download = 'stock-trend.png';
        link.href = canvas.toDataURL();
        link.click();
      });
    }

    // Function to download chart as a PDF
    function downloadPDF() {
      const { jsPDF } = window.jspdf;
      const pdf = new jsPDF();
      pdf.addImage(document.getElementById('stockChart'), 'PNG', 10, 10, 180, 100);
      pdf.save('stock-trend.pdf');
    }

    // Function to toggle dark mode
    function toggleDarkMode() {
      document.body.classList.toggle('dark-mode');
    }
  </script>
</body>

</html>
