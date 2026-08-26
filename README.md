<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Тетрис Классический</title>
    <style>
        body {
            background: #111;
            color: #fff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
        }

        h1 {
            margin: 0 0 10px 0;
            font-size: 2.5em;
            letter-spacing: 2px;
            text-shadow: 0 0 10px #00f0ff;
        }

        #game-container {
            display: flex;
            background: #1a1a1a;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 240, 255, 0.3);
            border: 2px solid #333;
        }

        canvas {
            border: 4px solid #fff;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
            background: #111;
        }

        #sidebar {
            margin-left: 20px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            min-width: 120px;
        }

        .info-box {
            background: #222;
            padding: 15px;
            border-radius: 5px;
            margin-bottom: 15px;
            border: 1px solid #444;
            text-align: center;
        }

        .info-box h2 {
            margin: 0 0 5px 0;
            font-size: 0.9em;
            color: #888;
            text-transform: uppercase;
        }

        .info-box div {
            font-size: 1.6em;
            font-weight: bold;
            color: #00f0ff;
        }

        #controls-hint {
            font-size: 0.85em;
            color: #aaa;
            line-height: 1.4;
        }

        #start-btn {
            padding: 12px;
            background: #00f0ff;
            border: none;
            color: #000;
            font-size: 1em;
            font-weight: bold;
            border-radius: 5px;
            cursor: pointer;
            transition: 0.2s;
            text-transform: uppercase;
        }

        #start-btn:hover {
            background: #ff007f;
            color: #fff;
            box-shadow: 0 0 15px #ff007f;
        }
    </style>
</head>
<body>

    <h1>ТЕТРИС</h1>

    <div id="game-container">
        <canvas id="tetris" width="240" height="400"></canvas>
        
        <div id="sidebar">
            <div>
                <div class="info-box">
                    <h2>Счёт</h2>
                    <div id="score">0</div>
                </div>
                <div class="info-box">
                    <h2>Линии</h2>
                    <div id="lines">0</div>
                </div>
            </div>

            <button id="start-btn">Старт / Музыка</button>

            <div id="controls-hint" class="info-box">
                <h2>Управление</h2>
                ← / → : Влево/Вправо<br>
                ↑ : Поворот<br>
                ↓ : Ускорить<br>
                Пробел: Упасть
            </div>
        </div>
    </div>

<script>
// --- ЗВУКОВОЙ ДВИЖОК (Web Audio API) ---
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
let musicInterval;
let isMusicPlaying = false;

// Мелодия "Коробейники" (нота, длительность в долях)
const melody = [
    ['E5', 2], ['B4', 1], ['C5', 1], ['D5', 2], ['C5', 1], ['B4', 1],
    ['A4', 2], ['A4', 1], ['C5', 1], ['E5', 2], ['D5', 1], ['C5', 1],
    ['B4', 3], ['C5', 1], ['D5', 2], ['E5', 2],
    ['C5', 2], ['A4', 2], ['A4', 2], ['space', 2],
    
    ['D5', 3], ['F5', 1], ['A5', 2], ['G5', 1], ['F5', 1],
    ['E5', 3], ['C5', 1], ['E5', 2], ['D5', 1], ['C5', 1],
    ['B4', 2], ['B4', 1], ['C5', 1], ['D5', 2], ['E5', 2],
    ['C5', 2], ['A4', 2], ['A4', 2], ['space', 2]
];

const noteFreqs = {
    'A4': 440.00, 'B4': 493.88, 'C5': 523.25, 'D5': 587.33,
    'E5': 659.25, 'F5': 698.46, 'G5': 783.99, 'A5': 880.00
};

function playTone(freq, duration, type = 'square', volume = 0.1) {
    if (!freq) return;
    const osc = audioCtx.createOscillator();
    const gainNode = audioCtx.createGain();
    
    osc.type = type;
    osc.frequency.value = freq;
    
    gainNode.gain.setValueAtTime(volume, audioCtx.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(0.00001, audioCtx.currentTime + duration);
    
    osc.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    
    osc.start();
    osc.stop(audioCtx.currentTime + duration);
}

function startMusic() {
    if (isMusicPlaying) return;
    isMusicPlaying = true;
    let noteIndex = 0;
    const tempo = 150; // Скорость музыки

    function playNextNote() {
        if (!isMusicPlaying) return;
        const [note, duration] = melody[noteIndex];
        const seconds = duration * (60 / tempo) * 0.5;
        
        if (note !== 'space') {
            playTone(noteFreqs[note], seconds, 'square', 0.04);
        }
        
        noteIndex = (noteIndex + 1) % melody.length;
        musicInterval = setTimeout(playNextNote, seconds * 1000);
    }
    playNextNote();
}

function stopMusic() {
    isMusicPlaying = false;
    clearTimeout(musicInterval);
}

function playRowSound() {
    playTone(880, 0.1, 'triangle', 0.2);
    setTimeout(() => playTone(1320, 0.15, 'triangle', 0.2), 80);
}

function playDropSound() {
    playTone(200, 0.05, 'sawtooth', 0.1);
}

// --- ЛОГИКА ИГРЫ ТЕТРИС ---
const canvas = document.getElementById('tetris');
const context = canvas.getContext('2d');
context.scale(20, 20); // Масштабируем пиксели под сетку игрового поля

let score = 0;
let linesCleared = 0;

// Цвета для фигур с неоновым свечением
const colors = [
    null,
    '#00f0ff', // I - Голубой
    '#ffc700', // O - Желтый
    '#a000ff', // T - Пурпурный
    '#00ff00', // S - Зеленый
    '#ff0000', // Z - Красный
    '#0000ff', // J - Синий
    '#ff7f00'  // L - Оранжевый
];

// Матрицы фигур
function createPiece(type) {
    if (type === 'I') return [[0,1,0,0],[0,1,0,0],[0,1,0,0],[0,1,0,0]];
    else if (type === 'O') return [[2,2],[2,2]];
    else if (type === 'T') return [[0,3,0],[3,3,3],[0,0,0]];
    else if (type === 'S') return [[0,4,4],[4,4,0],[0,0,0]];
    else if (type === 'Z') return [[5,5,0],[0,5,5],[0,0,0]];
    else if (type === 'J') return [[0,6,0],[0,6,0],[6,6,0]];
    else if (type === 'L') return [[0,7,0],[0,7,0],[0,7,7]];
}

// Создание пустого поля (12x20)
function createMatrix(w, h) {
    const matrix = [];
    while (h--) matrix.push(new Array(w).fill(0));
    return matrix;
}

const arena = createMatrix(12, 20);
const player = {
    pos: {x: 0, y: 0},
    matrix: null,
    score: 0
};

// Проверка столкновений
function collide(arena, player) {
    const [m, o] = [player.matrix, player.pos];
    for (let y = 0; y < m.length; ++y) {
        for (let x = 0; x < m[y].length; ++x) {
            if (m[y][x] !== 0 && (arena[y + o.y] && arena[y + o.y][x + o.x]) !== 0) {
                return true;
            }
        }
    }
    return false;
}

// Закрепление фигуры на поле
function merge(arena, player) {
    player.matrix.forEach((row, y) => {
        row.forEach((value, x) => {
            if (value !== 0) {
                arena[y + player.pos.y][x + player.pos.x] = value;
            }
        });
    });
}

// Отрисовка игрового поля и текстур блоков
function draw() {
    context.fillStyle = '#111';
    context.fillRect(0, 0, canvas.width, canvas.height);

    // Сетка на фоне
    context.strokeStyle = '#222';
    context.lineWidth = 0.05;
    for (let i = 0; i < 12; i++) {
        context.beginPath(); context.moveTo(i, 0); context.lineTo(i, 20); context.stroke();
    }
    for (let j = 0; j < 20; j++) {
        context.beginPath(); context.moveTo(0, j); context.lineTo(12, j); context.stroke();
    }

    drawMatrix(arena, {x: 0, y: 0});
    drawMatrix(player.matrix, player.pos);
}

// Отрисовка конкретной матрицы с дизайном "под текстуру"
function drawMatrix(matrix, offset) {
    matrix.forEach((row, y) => {
        row.forEach((value, x) => {
            if (value !== 0) {
                context.fillStyle = colors[value];
                context.fillRect(x + offset.x, y + offset.y, 1, 1);
                
                // Внутренняя текстура (эффект объема/блика)
                context.fillStyle = 'rgba(255, 255, 255, 0.3)';
                context.fillRect(x + offset.x + 0.1, y + offset.y + 0.1, 0.8, 0.15);
                context.fillRect(x + offset.x + 0.1, y + offset.y + 0.2, 0.15, 0.7);
                
                // Темная рамка для глубины
                context.fillStyle = 'rgba(0, 0, 0, 0.3)';
                context.fillRect(x + offset.x + 0.85, y + offset.y + 0.1, 0.1, 0.85);
                context.fillRect(x + offset.x + 0.1, y + offset.y + 0.85, 0.85, 0.1);
            }
        });
    });
}

// Очистка заполненных линий
function arenaSweep() {
    let rowCount = 1;
    outer: for (let y = arena.length - 1; y > 0; --y) {
        for (let x = 0; x < arena[y].length; ++x) {
            if (arena[y][x] === 0) {
                continue outer;
            }
        }
        const row = arena.splice(y, 1)[0].fill(0);
        arena.unshift(row);
        ++y;

        score += rowCount * 100;
        linesCleared++;
        rowCount *= 2;
        playRowSound();
    }
    updateScore();
}

// Падение фигуры каждую секунду
let dropCounter = 0;
let dropInterval = 1000;
let lastTime = 0;

function playerDrop() {
    player.pos.y++;
    if (collide(arena, player)) {
        player.pos.y--;
        merge(arena, player);
        playerReset();
        arenaSweep();
    }
    dropCounter = 0;
}

function playerMove(dir) {
    player.pos.x += dir;
    if (collide(arena, player)) {
        player.pos.x -= dir;
    }
}

// Генерация новой случайной фигуры
function playerReset() {
    const pieces = 'ILJOTSZ';
    player.matrix = createPiece(pieces[pieces.length * Math.random() | 0]);
    player.pos.y = 0;
    player.pos.x = (arena[0].length / 2 | 0) - (player.matrix[0].length / 2 | 0);
    
    // Проверка на конец игры
    if (collide(arena, player)) {
        arena.forEach(row => row.fill(0));
        score = 0;
        linesCleared = 0;
        updateScore();
        playTone(150, 0.8, 'sawtooth', 0.3); // Звук проигрыша
    }
}

// Поворот фигуры
function playerRotate() {
    const pos = player.pos.x;
    let offset = 1;
    rotate(player.matrix);
    while (collide(arena, player)) {
        player.pos.x += offset;
        offset = -(offset + (offset > 0 ? 1 : -1));
        if (offset > player.matrix[0].length) {
            rotate(player.matrix);
            player.pos.x = pos;
            return;
        }
    }
    playTone(400, 0.03, 'sine', 0.1); // Звук поворота
}

function rotate(matrix) {
    for (let y = 0; y < matrix.length; ++y) {
        for (let x = 0; x < y; ++x) {
            [matrix[x][y], matrix[y][x]] = [matrix[y][x], matrix[x][y]];
        }
    }
    matrix.forEach(row => row.reverse());
}

function updateScore() {
    document.getElementById('score').innerText = score;
    document.getElementById('lines').innerText = linesCleared;
}

// Основной игровой цикл
function update(time = 0) {
    const deltaTime = time - lastTime;
    lastTime = time;

    dropCounter += deltaTime;
    if (dropCounter > dropInterval) {
        playerDrop();
    }

    draw();
    requestAnimationFrame(update);
}

// Обработка клавиш управления
document.addEventListener('keydown', event => {
    if (event.keyCode === 37) { // Влево
        playerMove(-1);
    } else if (event.keyCode === 39) { // Вправо
        playerMove(1);
    } else if (event.keyCode === 40) { // Вниз (Ускорение)
        playerDrop();
    } else if (event.keyCode === 38) { // Вверх (Поворот)
        playerRotate();
    } else if (event.keyCode === 32) { // Пробел (Мгновенное падение)
        while(!collide(arena, player)) {
            player.pos.y++;
        }
        player.pos.y--;
        merge(arena, player);
        playDropSound();
        playerReset();
        arenaSweep();
    }
});

// Запуск аудиоконтекста и старт игры
document.getElementById('start-btn').addEventListener('click', () => {
    audioCtx.resume();
    if (isMusicPlaying) {
        stopMusic();
    } else {
        startMusic();
    }
    if (!player.matrix) {
        playerReset();
        update();
    }
});
</script>
</body>
</html>
