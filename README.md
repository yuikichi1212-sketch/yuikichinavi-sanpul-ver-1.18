<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculator english</title>
    <style>
        /* --- リキッドグラデーションの背景（白・シルバー系） --- */
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #e0eafc 0%, #cfdef3 50%, #e0eafc 100%);
            background-size: 400% 400%;
            animation: liquidGradient 12s ease infinite;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            overflow: hidden;
            color: #333;
        }

        @keyframes liquidGradient {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        /* --- メインのグラスコンテナ --- */
        .calculator {
            position: relative;
            width: 360px;
            background: rgba(255, 255, 255, 0.6);
            backdrop-filter: blur(30px);
            -webkit-backdrop-filter: blur(30px);
            border: 1px solid rgba(255, 255, 255, 0.8);
            border-radius: 36px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.05), inset 0 0 20px rgba(255, 255, 255, 0.5);
            padding: 28px;
            overflow: hidden;
        }

        /* --- ヘッダー領域 --- */
        .header {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20px;
        }

        .text-btn {
            background: rgba(255, 255, 255, 0.5);
            border: 1px solid rgba(255, 255, 255, 0.6);
            border-radius: 14px;
            padding: 8px 16px;
            color: #555;
            font-size: 14px;
            font-weight: 600;
            letter-spacing: 0.5px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 2px 10px rgba(0,0,0,0.02);
        }

        .text-btn:hover {
            background: rgba(255, 255, 255, 0.8);
            transform: translateY(-1px);
            color: #111;
        }

        /* --- ディスプレイ --- */
        .display {
            text-align: right;
            margin-bottom: 28px;
            padding: 10px;
            background: rgba(255, 255, 255, 0.3);
            border-radius: 16px;
            box-shadow: inset 0 2px 10px rgba(0,0,0,0.02);
        }

        .previous-operand {
            color: #888;
            font-size: 1.1rem;
            min-height: 24px;
            margin-bottom: 4px;
        }

        .current-operand {
            color: #111;
            font-size: 3.2rem;
            font-weight: 200;
            letter-spacing: -1px;
            word-wrap: break-word;
            word-break: break-all;
        }

        /* --- ボタンのグリッド --- */
        .buttons {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 12px;
        }

        button {
            border: none;
            outline: none;
            background: rgba(255, 255, 255, 0.4);
            border: 1px solid rgba(255, 255, 255, 0.5);
            border-radius: 18px;
            font-size: 1.4rem;
            color: #333;
            padding: 16px 0;
            cursor: pointer;
            transition: all 0.2s ease;
            box-shadow: 0 4px 12px rgba(0,0,0,0.03);
            font-weight: 400;
        }

        button:hover {
            background: rgba(255, 255, 255, 0.8);
            border-color: rgba(255, 255, 255, 0.9);
            transform: scale(1.03);
        }

        button:active {
            transform: scale(0.97);
        }

        .operator { 
            background: rgba(220, 230, 245, 0.6); 
            color: #0056b3; 
            font-weight: 500;
        }
        
        .function {
            background: rgba(240, 240, 240, 0.6);
            font-size: 1.1rem;
            color: #555;
        }

        .equals { 
            background: #0056b3; 
            color: #fff; 
            grid-row: span 2; 
            border: none;
            box-shadow: 0 8px 20px rgba(0, 86, 179, 0.3);
        }
        
        .equals:hover {
            background: #004494;
        }

        .zero { grid-column: span 2; border-radius: 18px; }
        
        .clear-btn { color: #d93025; font-weight: 500; }

        /* --- 履歴パネル --- */
        .history-panel {
            position: absolute;
            bottom: -100%;
            left: 0;
            width: 100%;
            height: 75%;
            background: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(40px);
            -webkit-backdrop-filter: blur(40px);
            border-top: 1px solid rgba(255, 255, 255, 0.9);
            border-radius: 36px 36px 0 0;
            transition: bottom 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            padding: 28px;
            box-sizing: border-box;
            overflow-y: auto;
            box-shadow: 0 -10px 40px rgba(0,0,0,0.05);
        }

        .history-panel.active {
            bottom: 0;
        }

        .history-title {
            color: #111;
            margin-top: 0;
            margin-bottom: 20px;
            font-weight: 500;
            font-size: 1.2rem;
            letter-spacing: 1px;
        }

        .history-item {
            color: #444;
            padding: 12px 0;
            border-bottom: 1px solid rgba(0, 0, 0, 0.05);
            font-size: 1.1rem;
            text-align: right;
            font-family: monospace;
        }
    </style>
</head>
<body>

    <div class="calculator">
        <div class="header">
            <button class="text-btn" onclick="toggleHistory()">History</button>
            <button class="text-btn" onclick="shareResult()">Share</button>
        </div>

        <div class="display">
            <div class="previous-operand" id="previousOperand"></div>
            <div class="current-operand" id="currentOperand">0</div>
        </div>

        <div class="buttons">
            <button class="function" onclick="calculateSingle('sqrt')">√</button>
            <button class="function" onclick="calculateSingle('sqr')">x²</button>
            <button class="function" onclick="calculateSingle('sin')">sin</button>
            <button class="function" onclick="calculateSingle('cos')">cos</button>

            <button class="clear-btn" onclick="clearDisplay()">AC</button>
            <button onclick="deleteNumber()">DEL</button>
            <button class="operator" onclick="appendOperator('%')">%</button>
            <button class="operator" onclick="appendOperator('/')">÷</button>

            <button onclick="appendNumber('7')">7</button>
            <button onclick="appendNumber('8')">8</button>
            <button onclick="appendNumber('9')">9</button>
            <button class="operator" onclick="appendOperator('*')">×</button>

            <button onclick="appendNumber('4')">4</button>
            <button onclick="appendNumber('5')">5</button>
            <button onclick="appendNumber('6')">6</button>
            <button class="operator" onclick="appendOperator('-')">-</button>

            <button onclick="appendNumber('1')">1</button>
            <button onclick="appendNumber('2')">2</button>
            <button onclick="appendNumber('3')">3</button>
            <button class="operator" onclick="appendOperator('+')">+</button>

            <button class="zero" onclick="appendNumber('0')">0</button>
            <button onclick="appendNumber('.')">.</button>
            <button class="equals" onclick="calculate()">=</button>
        </div>

        <div class="history-panel" id="historyPanel">
            <h3 class="history-title">Calculation History</h3>
            <div id="historyList"></div>
        </div>
    </div>

    <script>
        let currentOperand = '';
        let previousOperand = '';
        let operation = undefined;
        let history = [];

        const currentDisplay = document.getElementById('currentOperand');
        const previousDisplay = document.getElementById('previousOperand');
        const historyList = document.getElementById('historyList');
        const historyPanel = document.getElementById('historyPanel');

        function updateDisplay() {
            currentDisplay.innerText = currentOperand || '0';
            if (operation != null) {
                let opSymbol = operation;
                if(opSymbol === '*') opSymbol = '×';
                if(opSymbol === '/') opSymbol = '÷';
                previousDisplay.innerText = `${previousOperand} ${opSymbol}`;
            } else {
                previousDisplay.innerText = '';
            }
        }

        function appendNumber(number) {
            if (number === '.' && currentOperand.includes('.')) return;
            currentOperand = currentOperand.toString() + number.toString();
            updateDisplay();
        }

        function appendOperator(op) {
            if (currentOperand === '') return;
            if (previousOperand !== '') {
                calculate();
            }
            operation = op;
            previousOperand = currentOperand;
            currentOperand = '';
            updateDisplay();
        }

        function clearDisplay() {
            currentOperand = '';
            previousOperand = '';
            operation = undefined;
            updateDisplay();
        }

        function deleteNumber() {
            currentOperand = currentOperand.toString().slice(0, -1);
            updateDisplay();
        }

        // 新規追加：単項演算（即時計算）の処理
        function calculateSingle(action) {
            if (currentOperand === '') return;
            const current = parseFloat(currentOperand);
            if (isNaN(current)) return;

            let result;
            let symbolText;

            switch (action) {
                case 'sqrt':
                    result = Math.sqrt(current);
                    symbolText = `√(${current})`;
                    break;
                case 'sqr':
                    result = Math.pow(current, 2);
                    symbolText = `${current}²`;
                    break;
                case 'sin':
                    // 角度を度数法(Degree)として計算
                    result = Math.sin(current * (Math.PI / 180));
                    symbolText = `sin(${current})`;
                    // 微小な誤差を修正
                    if (Math.abs(result) < 1e-10) result = 0;
                    break;
                case 'cos':
                    result = Math.cos(current * (Math.PI / 180));
                    symbolText = `cos(${current})`;
                    if (Math.abs(result) < 1e-10) result = 0;
                    break;
                default:
                    return;
            }

            // 桁数を丸める処理（小数点以下8桁まで）
            result = Math.round(result * 100000000) / 100000000;

            const historyText = `${symbolText} = ${result}`;
            history.unshift(historyText);
            updateHistoryUI();

            currentOperand = result.toString();
            updateDisplay();
        }

        function calculate() {
            let computation;
            const prev = parseFloat(previousOperand);
            const current = parseFloat(currentOperand);
            if (isNaN(prev) || isNaN(current)) return;

            switch (operation) {
                case '+': computation = prev + current; break;
                case '-': computation = prev - current; break;
                case '*': computation = prev * current; break;
                case '/': computation = prev / current; break;
                case '%': computation = prev % current; break;
                default: return;
            }

            // 桁数を丸める処理
            computation = Math.round(computation * 100000000) / 100000000;

            let opSymbol = operation;
            if(opSymbol === '*') opSymbol = '×';
            if(opSymbol === '/') opSymbol = '÷';
            const historyText = `${prev} ${opSymbol} ${current} = ${computation}`;
            history.unshift(historyText);
            updateHistoryUI();

            currentOperand = computation.toString();
            operation = undefined;
            previousOperand = '';
            updateDisplay();
        }

        function toggleHistory() {
            historyPanel.classList.toggle('active');
        }

        function updateHistoryUI() {
            historyList.innerHTML = '';
            history.forEach(item => {
                const div = document.createElement('div');
                div.className = 'history-item';
                div.innerText = item;
                historyList.appendChild(div);
            });
        }

        async function shareResult() {
            const textToShare = currentOperand ? `Calculation Result: ${currentOperand}\n[Sent from Modern Glass Calculator]` : "Check out this modern calculator app!";
            
            if (navigator.share) {
                try {
                    await navigator.share({
                        title: 'Share Result',
                        text: textToShare,
                    });
                } catch (err) {
                    console.log('Share canceled or failed', err);
                }
            } else {
                alert('Result copied to clipboard: \n' + textToShare);
                navigator.clipboard.writeText(textToShare);
            }
        }
    </script>
</body>
</html>
