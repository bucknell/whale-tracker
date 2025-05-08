<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Whale Tracker (DeBank)</title>
  <style>
    body { font-family: sans-serif; padding: 20px; background: #f0f0f0; }
    button { font-size: 16px; padding: 10px 20px; margin-right: 10px; }
    pre { background: #fff; padding: 15px; border: 1px solid #ccc; margin-top: 20px; white-space: pre-wrap; }
  </style>
</head>
<body>
  <h1>🦈 Whale Tracker (via DeBank API)</h1>
  <button onclick="trackWhales()">Track Wallets</button>
  <button onclick="exportCSV()">Export to CSV</button>
  <pre id="output">Click to fetch wallet data...</pre>

  <script>
    const wallets = [
      '0x742d35Cc6634C0532925a3b844Bc454e4438f44e',
      '0xDC76CD25977E0a5Ae17155770273aD58648900D3'
    ];

    let csvData = [['Wallet', 'Symbol', 'Amount', 'USD Value']];

    async function fetchDeBank(address) {
      const url = `https://openapi.debank.com/v1/user/token_list?id=${address}&is_all=true`;
      const res = await fetch(url);
      if (!res.ok) throw new Error('API error');
      return await res.json();
    }

    async function trackWhales() {
      const out = document.getElementById('output');
      out.textContent = 'Loading data...\n';
      csvData = [['Wallet', 'Symbol', 'Amount', 'USD Value']];

      for (const wallet of wallets) {
        out.textContent += `\n📊 Wallet: ${wallet}\n`;
        try {
          const tokens = await fetchDeBank(wallet);
          tokens.forEach(t => {
            const usd = (t.price * t.amount).toFixed(2);
            const line = `→ ${t.symbol.padEnd(8)} $${usd} | ${t.amount.toFixed(4)} tokens`;
            out.textContent += line + '\n';
            csvData.push([wallet, t.symbol, t.amount.toFixed(4), usd]);
          });
        } catch (e) {
          out.textContent += `❌ Error: ${e.message}\n`;
        }
      }
    }

    function exportCSV() {
      const rows = csvData.map(row => row.join(','));
      const blob = new Blob([rows.join('\n')], { type: 'text/csv' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'whale_snapshot.csv';
      a.click();
      URL.revokeObjectURL(url);
    }
  </script>
</body>
</html>
