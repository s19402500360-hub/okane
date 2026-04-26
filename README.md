app_code = """
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Click Tycoon - お金稼ぎシミュレーター</title>
    <style>
        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            background-color: #f0f2f5;
            color: #333;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        h1 {
            color: #2c3e50;
            margin-bottom: 10px;
        }
        .balance-container {
            background: white;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            text-align: center;
            margin-bottom: 30px;
            width: 80%;
            max-width: 400px;
        }
        .balance {
            font-size: 3em;
            font-weight: bold;
            color: #27ae60;
        }
        .per-sec {
            font-size: 1.2em;
            color: #7f8c8d;
        }
        .work-btn {
            background: linear-gradient(135deg, #3498db, #2980b9);
            color: white;
            border: none;
            border-radius: 50%;
            width: 150px;
            height: 150px;
            font-size: 1.5em;
            cursor: pointer;
            box-shadow: 0 6px 0 #1f618d;
            transition: all 0.1s;
            margin-bottom: 40px;
            outline: none;
            user-select: none;
        }
        .work-btn:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #1f618d;
        }
        .shop {
            background: white;
            width: 80%;
            max-width: 600px;
            border-radius: 15px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            padding: 20px;
        }
        .shop h2 {
            margin-top: 0;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
        }
        .item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 15px;
            border-bottom: 1px solid #eee;
        }
        .item:last-child {
            border-bottom: none;
        }
        .item-info {
            display: flex;
            flex-direction: column;
        }
        .item-name {
            font-weight: bold;
            font-size: 1.1em;
        }
        .item-desc {
            font-size: 0.9em;
            color: #7f8c8d;
        }
        .buy-btn {
            background-color: #2ecc71;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            transition: background 0.2s;
        }
        .buy-btn:hover {
            background-color: #27ae60;
        }
        .buy-btn:disabled {
            background-color: #bdc3c7;
            cursor: not-allowed;
        }
        .reset-btn {
            margin-top: 20px;
            background: #e74c3c;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 0.8em;
        }
        .floating-text {
            position: absolute;
            font-weight: bold;
            color: #27ae60;
            pointer-events: none;
            animation: floatUp 1s ease-out forwards;
        }
        @keyframes floatUp {
            0% { opacity: 1; transform: translateY(0); }
            100% { opacity: 0; transform: translateY(-50px); }
        }
    </style>
</head>
<body>
    <h1>💰 Click Tycoon</h1>
    <div class="balance-container">
        <div>現在の資産</div>
        <div class="balance" id="balance">¥0</div>
        <div class="per-sec">秒間収益: <span id="per-sec">¥0</span>/秒</div>
    </div>
    <button class="work-btn" id="work-btn" onclick="clickWork(event)">
        働く<br>Click!
    </button>
    <div class="shop">
        <h2>🛒 投資ショップ</h2>
        <div id="shop-items"></div>
    </div>
    <button class="reset-btn" onclick="resetGame()">データをリセット</button>
    <script>
        let state = {
            balance: 0, clickPower: 1, autoIncome: 0,
            items: [
                { id: 1, name: "コーヒー", desc: "集中力が上がる (+¥1/クリック)", price: 50, type: "click", value: 1, count: 0 },
                { id: 2, name: "効率PC", desc: "作業効率が上がる (+¥5/クリック)", price: 300, type: "click", value: 5, count: 0 },
                { id: 3, name: "アルバイト", desc: "自動で稼いでくれる (+¥2/秒)", price: 100, type: "auto", value: 2, count: 0 },
                { id: 4, name: "フリーランス", desc: "スキルで稼ぐ (+¥10/秒)", price: 500, type: "auto", value: 10, count: 0 },
                { id: 5, name: "会社設立", desc: "従業員を雇って稼ぐ (+¥50/秒)", price: 2000, type: "auto", value: 50, count: 0 },
                { id: 6, name: "投資ファンド", desc: "お金がお金を呼ぶ (+¥200/秒)", price: 10000, type: "auto", value: 200, count: 0 }
            ]
        };
        function loadGame() { 
            const saved = localStorage.getItem('clickTycoonSave'); 
            if (saved) { Object.assign(state, JSON.parse(saved)); } 
            updateUI(); renderShop(); 
        }
        function saveGame() { localStorage.setItem('clickTycoonSave', JSON.stringify(state)); }
        function updateUI() {
            document.getElementById('balance').innerText = '¥' + Math.floor(state.balance).toLocaleString();
            document.getElementById('per-sec').innerText = '¥' + state.autoIncome.toLocaleString();
            state.items.forEach(item => {
                const btn = document.getElementById('btn-' + item.id);
                if (btn) { btn.disabled = state.balance < item.price; }
            });
        }
        function clickWork(event) {
            state.balance += state.clickPower;
            const floatText = document.createElement('div');
            floatText.className = 'floating-text';
            floatText.innerText = '+' + state.clickPower + '円';
            floatText.style.left = event.clientX + 'px';
            floatText.style.top = event.clientY + 'px';
            document.body.appendChild(floatText);
            setTimeout(() => floatText.remove(), 1000);
            updateUI(); saveGame();
        }
        function buyItem(id) {
            const item = state.items.find(i => i.id === id);
            if (item && state.balance >= item.price) {
                state.balance -= item.price; item.count++;
                if (item.type === 'click') state.clickPower += item.value;
                else state.autoIncome += item.value;
                item.price = Math.floor(item.price * 1.2);
                updateUI(); renderShop(); saveGame();
            }
        }
        function renderShop() {
            const container = document.getElementById('shop-items');
            container.innerHTML = state.items.map(item => `
                <div class="item">
                    <div class="item-info">
                        <span class="item-name">${item.name} (所持: ${item.count})</span>
                        <span class="item-desc">${item.desc}</span>
                    </div>
                    <button id="btn-${item.id}" class="buy-btn" onclick="buyItem(${item.id})">購入 (${item.price.toLocaleString()}円)</button>
                </div>`).join('');
        }
        setInterval(() => { if (state.autoIncome > 0) { state.balance += state.autoIncome; updateUI(); saveGame(); } }, 1000);
        function resetGame() { if(confirm("リセットしますか？")) { localStorage.removeItem('clickTycoonSave'); location.reload(); } }
        loadGame();
    </script>
</body>
</html>
"""

with open('money_app.html', 'w') as f:
    f.write(app_code)
print("money_app.html has been saved via Python.")