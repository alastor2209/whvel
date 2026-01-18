<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Airport Randomizer</title>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;600&family=Share+Tech+Mono&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-color: #0b1d2a;
            --board-color: #223344;
            --text-gold: #ffcc00;
            --text-white: #e0e0e0;
            --accent: #d32f2f;
        }

        body {
            font-family: 'Kanit', sans-serif;
            background-color: var(--bg-color);
            color: var(--text-white);
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            overflow-x: hidden;
        }

        /* Header Style */
        header {
            width: 100%;
            background: #000;
            padding: 15px 0;
            text-align: center;
            border-bottom: 4px solid var(--text-gold);
            box-shadow: 0 5px 15px rgba(0,0,0,0.5);
        }

        h1 {
            margin: 0;
            font-family: 'Share Tech Mono', monospace;
            color: var(--text-gold);
            font-size: 2.5rem;
            letter-spacing: 2px;
            text-transform: uppercase;
        }

        /* Container */
        .container {
            width: 90%;
            max-width: 800px;
            margin-top: 30px;
            text-align: center;
        }

        /* Input Section */
        #setup-area {
            background: var(--board-color);
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            margin-bottom: 20px;
        }

        textarea {
            width: 100%;
            height: 150px;
            background: #111;
            color: #fff;
            border: 1px solid #444;
            padding: 10px;
            font-family: 'Kanit', sans-serif;
            font-size: 1rem;
            resize: vertical;
            border-radius: 5px;
        }

        .hint {
            font-size: 0.8rem;
            color: #aaa;
            margin-top: 5px;
            text-align: left;
        }

        /* Display Board (The Result) */
        .flight-board {
            background: #000;
            padding: 20px;
            border-radius: 10px;
            border: 2px solid #444;
            min-height: 200px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            position: relative;
            margin-bottom: 20px;
        }

        .flight-info {
            font-family: 'Share Tech Mono', monospace;
            width: 100%;
            display: flex;
            justify-content: space-between;
            border-bottom: 1px dashed #444;
            padding: 10px 0;
            color: #888;
            font-size: 1.2rem;
        }

        #result-display {
            font-family: 'Share Tech Mono', monospace;
            font-size: 3.5rem;
            color: var(--text-gold);
            text-shadow: 0 0 10px rgba(255, 204, 0, 0.5);
            margin: 30px 0;
            letter-spacing: 2px;
            min-height: 80px;
        }

        .status-badge {
            background: var(--accent);
            color: white;
            padding: 5px 15px;
            border-radius: 5px;
            font-size: 1rem;
            display: none;
            animation: blink 1s infinite;
        }

        /* Controls */
        .btn {
            background: var(--text-gold);
            color: #000;
            border: none;
            padding: 12px 30px;
            font-size: 1.2rem;
            font-weight: bold;
            cursor: pointer;
            border-radius: 5px;
            transition: 0.3s;
            text-transform: uppercase;
            font-family: 'Share Tech Mono', monospace;
            margin: 10px;
        }

        .btn:hover {
            background: #e6b800;
            transform: scale(1.05);
        }

        .btn-small {
            font-size: 1rem;
            padding: 8px 20px;
            background: #555;
            color: white;
        }

        /* Waiting List */
        .queue-list {
            margin-top: 20px;
            color: #aaa;
            font-size: 0.9rem;
            background: rgba(0,0,0,0.3);
            padding: 10px;
            border-radius: 5px;
        }
        
        #passenger-count {
            color: var(--text-gold);
            font-weight: bold;
        }

        @keyframes blink {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }

        /* Hide Setup when running */
        .hidden {
            display: none;
        }
    </style>
</head>
<body>

    <header>
        <h1>DEPARTURE RANDOMIZER</h1>
    </header>

    <div class="container">
        
        <div id="setup-area">
            <h3>ลงทะเบียนผู้โดยสาร (Passenger List)</h3>
            <textarea id="name-input" placeholder="วางรายชื่อที่นี่ (1 บรรทัดต่อ 1 ชื่อ)..."></textarea>
            <div class="hint">* ใส่รายชื่อแล้วกดปุ่ม "LOAD DATA" เพื่อเริ่มระบบ</div>
            <button class="btn" onclick="loadData()">LOAD DATA</button>
        </div>

        <div id="randomizer-area" class="hidden">
            <div class="flight-board">
                <div class="flight-info">
                    <span>FLIGHT: <span id="flight-num">TG-001</span></span>
                    <span>GATE: <span id="gate-num">RANDOM</span></span>
                </div>
                
                <div id="result-display">WAITING...</div>
                
                <div id="status-text" class="status-badge">BOARDING NOW</div>
            </div>

            <button class="btn" id="action-btn" onclick="randomizeName()">NEXT PASSENGER</button>
            <button class="btn btn-small" onclick="resetSystem()">RESET</button>

            <div class="queue-list">
                ผู้โดยสารที่เหลือ (Remaining): <span id="passenger-count">0</span> คน
            </div>
        </div>

    </div>

    <script>
        let poolNames = [];
        let lockedNames = {};
        let currentTurn = 1;
        let totalCount = 0;
        let isAnimating = false;

        function loadData() {
            const input = document.getElementById('name-input').value;
            if (!input.trim()) {
                alert("กรุณาใส่รายชื่อก่อนครับ");
                return;
            }

            poolNames = [];
            lockedNames = {};
            currentTurn = 1;
            
            const lines = input.split('\n');
            
            lines.forEach(line => {
                let name = line.trim();
                if (name) {
                    if (name.includes('///')) {
                        const parts = name.split('///');
                        const personName = parts[0].trim();
                        const lockOrder = parseInt(parts[1].trim());

                        if (!isNaN(lockOrder)) {
                            lockedNames[lockOrder] = personName;
                        } else {
                            poolNames.push(personName);
                        }
                    } else {
                        poolNames.push(name);
                    }
                }
            });

            totalCount = poolNames.length + Object.keys(lockedNames).length;

            document.getElementById('passenger-count').innerText = totalCount;
            document.getElementById('setup-area').classList.add('hidden');
            document.getElementById('randomizer-area').classList.remove('hidden');
            document.getElementById('result-display').innerText = "READY";
            document.getElementById('status-text').style.display = "none";
        }

        function randomizeName() {
            if (isAnimating) return;
            
            if (poolNames.length === 0 && !lockedNames[currentTurn]) {
                document.getElementById('result-display').innerText = "CLOSED";
                document.getElementById('status-text').innerText = "FLIGHT DEPARTED";
                document.getElementById('status-text').style.background = "gray";
                document.getElementById('action-btn').disabled = true;
                return;
            }

            isAnimating = true;
            const display = document.getElementById('result-display');
            const status = document.getElementById('status-text');
            
            status.style.display = "none";
            document.getElementById('gate-num').innerText = Math.floor(Math.random() * 99) + 1;

            let counter = 0;
            const tempInterval = setInterval(() => {
                if (poolNames.length > 0) {
                    display.innerText = poolNames[Math.floor(Math.random() * poolNames.length)];
                } else {
                    display.innerText = "LOADING...";
                }
                counter++;
                
                if (counter > 20) {
                    clearInterval(tempInterval);
                    finalizeResult();
                }
            }, 50);
        }

        function finalizeResult() {
            const display = document.getElementById('result-display');
            const status = document.getElementById('status-text');
            let selectedName = "";

            if (lockedNames[currentTurn]) {
                selectedName = lockedNames[currentTurn];
            } else {
                if (poolNames.length > 0) {
                    const randomIndex = Math.floor(Math.random() * poolNames.length);
                    selectedName = poolNames[randomIndex];
                    poolNames.splice(randomIndex, 1);
                } else {
                    selectedName = "---";
                }
            }
