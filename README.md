<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة X - O الذكية</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 0;
            color: #fff;
        }
        .container {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);
            text-align: center;
            max-width: 400px;
            width: 90%;
        }
        h1 {
            margin-bottom: 10px;
            font-size: 2.2rem;
        }
        .mode-selector {
            margin-bottom: 20px;
        }
        .mode-btn {
            background: rgba(255, 255, 255, 0.2);
            border: none;
            color: white;
            padding: 8px 16px;
            margin: 0 5px;
            border-radius: 20px;
            cursor: pointer;
            transition: 0.3s;
        }
        .mode-btn.active {
            background: #ff4757;
            font-weight: bold;
        }
        .status {
            font-size: 1.3rem;
            margin-bottom: 20px;
            height: 35px;
            font-weight: bold;
            color: #ffeaa7;
        }
        .board {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-bottom: 20px;
        }
        .cell {
            width: 100%;
            height: 90px;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 12px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 2.5rem;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s ease;
        }
        .cell:hover {
            background: rgba(255, 255, 255, 0.35);
            transform: scale(1.02);
        }
        .cell.x {
            color: #ff6b6b;
        }
        .cell.o {
            color: #1dd1a1;
        }
        .reset-btn {
            background: #2ed573;
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 25px;
            cursor: pointer;
            box-shadow: 0 4px 15px rgba(46, 213, 115, 0.4);
            transition: 0.3s;
        }
        .reset-btn:hover {
            background: #26af5f;
            transform: translateY(-2px);
        }
        .scoreboard {
            display: flex;
            justify-content: space-around;
            margin-bottom: 20px;
            background: rgba(0, 0, 0, 0.2);
            padding: 10px;
            border-radius: 10px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🎮 لعبة X - O</h1>
    
    <div class="mode-selector">
        <button class="mode-btn active" id="btn2p" onclick="setMode('2p')">لاعبين 👥</button>
        <button class="mode-btn" id="btnAi" onclick="setMode('ai')">ضد الكمبيوتر 💻</button>
    </div>

    <div class="scoreboard">
        <div>لاعب (X): <span id="scoreX">0</span></div>
        <div>خصم (O): <span id="scoreO">0</span></div>
    </div>

    <div class="status" id="status">دور اللاعب: X</div>

    <div class="board" id="board">
        <div class="cell" onclick="makeMove(0)"></div>
        <div class="cell" onclick="makeMove(1)"></div>
        <div class="cell" onclick="makeMove(2)"></div>
        <div class="cell" onclick="makeMove(3)"></div>
        <div class="cell" onclick="makeMove(4)"></div>
        <div class="cell" onclick="makeMove(5)"></div>
        <div class="cell" onclick="makeMove(6)"></div>
        <div class="cell" onclick="makeMove(7)"></div>
        <div class="cell" onclick="makeMove(8)"></div>
    </div>

    <button class="reset-btn" onclick="resetGame()">إعادة اللعب 🔄</button>
</div>

<script>
    let boardState = ["", "", "", "", "", "", "", "", ""];
    let currentPlayer = "X";
    let gameActive = true;
    let gameMode = "2p";
    let scoreX = 0;
    let scoreO = 0;

    const winningConditions = [
        [0, 1, 2], [3, 4, 5], [6, 7, 8],
        [0, 3, 6], [1, 4, 7], [2, 5, 8],
        [0, 4, 8], [2, 4, 6]
    ];

    function setMode(mode) {
        gameMode = mode;
        document.getElementById('btn2p').classList.toggle('active', mode === '2p');
        document.getElementById('btnAi').classList.toggle('active', mode === 'ai');
        resetGame();
    }

    function makeMove(index) {
        if (boardState[index] !== "" || !gameActive) return;

        applyMove(index, currentPlayer);

        if (checkWin(boardState, currentPlayer)) {
            if (currentPlayer === "X") {
                document.getElementById('status').innerText = "كسبت افسيو 🎉";
                scoreX++;
            } else {
                document.getElementById('status').innerText = "فاز الخصم (O) 😞";
                scoreO++;
            }
            updateScore();
            gameActive = false;
            return;
        }

        if (!boardState.includes("")) {
            document.getElementById('status').innerText = "🤝 تعادل!";
            gameActive = false;
            return;
        }

        currentPlayer = currentPlayer === "X" ? "O" : "X";
        document.getElementById('status').innerText = `دور اللاعب: ${currentPlayer}`;

        if (gameMode === "ai" && currentPlayer === "O" && gameActive) {
            setTimeout(aiSmartMove, 400);
        }
    }

    function applyMove(index, player) {
        boardState[index] = player;
        let cell = document.getElementsByClassName('cell')[index];
        cell.innerText = player;
        cell.classList.add(player.toLowerCase());
    }

    function aiSmartMove() {
        let bestMove = findBestMove('O');
        if (bestMove === -1) bestMove = findBestMove('X');
        if (bestMove === -1 && boardState[4] === "") bestMove = 4;
        if (bestMove === -1) {
            let emptyIndices = boardState.map((val, idx) => val === "" ? idx : null).filter(val => val !== null);
            bestMove = emptyIndices[Math.floor(Math.random() * emptyIndices.length)];
        }

        if (bestMove !== undefined) {
            makeMove(bestMove);
        }
    }

    function findBestMove(player) {
        for (let condition of winningConditions) {
            let [a, b, c] = condition;
            let vals = [boardState[a], boardState[b], boardState[c]];
            if (vals.filter(v => v === player).length === 2 && vals.includes("")) {
                if (boardState[a] === "") return a;
                if (boardState[b] === "") return b;
                if (boardState[c] === "") return c;
            }
        }
        return -1;
    }

    function checkWin(board, player) {
        return winningConditions.some(condition => {
            return condition.every(index => board[index] === player);
        });
    }

    function updateScore() {
        document.getElementById('scoreX').innerText = scoreX;
        document.getElementById('scoreO').innerText = scoreO;
    }

    function resetGame() {
        boardState = ["", "", "", "", "", "", "", "", ""];
        currentPlayer = "X";
        gameActive = true;
        document.getElementById('status').innerText = `دور اللاعب: X`;
        let cells = document.getElementsByClassName('cell');
        Array.from(cells).forEach(cell => {
            cell.innerText = "";
            cell.classList.remove('x', 'o');
        });
    }
</script>

</body>
</html>
