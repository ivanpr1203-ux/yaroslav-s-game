[index.html](https://github.com/user-attachments/files/31438921/index.html)
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ігровий Портал — Неонові Ігри</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            background-color: #0b0b10;
            color: #ffffff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }
        header {
            width: 100%;
            padding: 20px;
            background: #12121c;
            text-align: center;
            box-shadow: 0 2px 10px rgba(0, 255, 204, 0.2);
        }
        header h1 {
            color: #00ffcc;
            font-size: 28px;
            text-transform: uppercase;
            letter-spacing: 2px;
        }
        nav {
            margin: 20px 0;
            display: flex;
            gap: 15px;
        }
        .btn-game {
            background: #1a1a26;
            color: #00ffcc;
            border: 2px solid #00ffcc;
            padding: 12px 24px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.3s ease;
        }
        .btn-game:hover, .btn-game.active {
            background: #00ffcc;
            color: #0b0b10;
            box-shadow: 0 0 15px rgba(0, 255, 204, 0.6);
        }
        #game-wrapper {
            position: relative;
            background: #12121c;
            border-radius: 12px;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.8);
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        canvas {
            background: #000000;
            border-radius: 6px;
            box-shadow: 0 0 15px rgba(255, 0, 128, 0.2);
        }
        .controls-info {
            margin-top: 15px;
            font-size: 14px;
            color: #aaa;
            text-align: center;
        }
    </style>
</head>
<body>

    <header>
        <h1>🎮 Ігровий Портал</h1>
    </header>

    <nav>
        <button class="btn-game active" onclick="switchGame('block-blast')">Neon Block Blast</button>
        <button class="btn-game" onclick="switchGame('tetris')">Тетрис</button>
    </nav>

    <div id="game-wrapper">
        <canvas id="mainCanvas" width="360" height="600"></canvas>
        <div class="controls-info" id="controlsText">Оберіть ігру для початку</div>
    </div>

    <script>
        const canvas = document.getElementById('mainCanvas');
        const ctx = canvas.getContext('2d');
        const controlsText = document.getElementById('controlsText');

        let currentGame = 'block-blast';

        function switchGame(gameType) {
            currentGame = gameType;
            document.querySelectorAll('.btn-game').forEach(btn => btn.classList.remove('active'));
            
            if (gameType === 'block-blast') {
                event.target.classList.add('active');
                controlsText.innerText = 'Керування: Перетягуйте неонові блоки на поле мишкою або тачем.';
                renderBlockBlast();
            } else if (gameType === 'tetris') {
                event.target.classList.add('active');
                controlsText.innerText = 'Керування: Стрілки ← → для руху, ↑ поворот, ↓ прискорення, Пробіл — швидке падіння.';
                renderTetris();
            }
        }

        function renderBlockBlast() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#12121c';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#00ffcc';
            ctx.font = '22px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('Neon Block Blast', canvas.width / 2, canvas.height / 2 - 20);
            ctx.fillStyle = '#888';
            ctx.font = '14px Arial';
            ctx.fillText('Гра готова до запуску на сайті', canvas.width / 2, canvas.height / 2 + 20);
        }

        function renderTetris() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#05050a';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#ff0080';
            ctx.font = '22px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('Класичний Тетрис', canvas.width / 2, canvas.height / 2 - 20);
            ctx.fillStyle = '#888';
            ctx.font = '14px Arial';
            ctx.fillText('Гра готова до запуску на сайті', canvas.width / 2, canvas.height / 2 + 20);
        }

        // Запуск за замовчуванням
        renderBlockBlast();
    </script>
</body>
</html>
