<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SNG Runner Gold Edition</title>
    <style>
        body { margin: 0; background: #222; display: flex; justify-content: center; align-items: center; height: 100vh; font-family: 'Courier New', Courier, monospace; overflow: hidden; }
        canvas { background: #555; border: 2px solid #fff; max-width: 100%; max-height: 100%; image-rendering: pixelated; touch-action: none; }
    </style>
</head>
<body>

<canvas id="game"></canvas>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

canvas.width = 800;
canvas.height = 400;

// === ЗАГРУЗКА ОБНОВЛЕННЫХ СПРАЙТОВ (ССЫЛКИ ИЗ ИНТЕРНЕТА) ===
const imgPlayer = new Image();
imgPlayer.src = 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/25.png'; // Заглушка, позже будет реальный Скотт
// ВАЖНО: Ниже я вручную рисую пиксели по фото, т.к. найти прямые ссылки сложно.

const imgTrash = new Image();
// Вручную рисую пиксельную мусорку ниже

const imgCup = new Image();
// Вручную рисую кубок ниже

// === СОСТОЯНИЕ ИГРЫ ===
let score = 0;
let highScore = parseInt(localStorage.getItem('sng_score')) || 0;
let hasCup = localStorage.getItem('sng_cup') === 'true';
let gameState = 'MENU'; // MENU, PLAYING, WIN, GAMEOVER
let gameSpeed = 5;
let frame = 0;

// === ИГРОК (ОБНОВЛЕННЫЙ ПИКСЕЛЬНЫЙ АВАТАР СКОТТА) ===
const player = {
    x: 80, y: 340 - 50, w: 40, h: 50, dy: 0, gravity: 0.8, jump: -15, grounded: false,
    draw() {
        // Рисую аватар по фото (светло-голубой, пиксельный, светящийся)
        ctx.fillStyle = '#b3f0ff'; // Светло-голубой
        ctx.fillRect(this.x, this.y, this.w, this.h);
        
        // Внутренние "пиксели" для текстуры
        ctx.fillStyle = '#fff';
        ctx.fillRect(this.x + 8, this.y + 8, 8, 8); // Левый глаз
        ctx.fillRect(this.x + 24, this.y + 8, 8, 8); // Правый глаз
        ctx.fillRect(this.x + 16, this.y + 24, 8, 16); // Тело
        
        // Эффект свечения (как на фото)
        ctx.strokeStyle = '#00e6e6';
        ctx.lineWidth = 2;
        ctx.strokeRect(this.x, this.y, this.w, this.h);
    }
};

let obstacles = [];
let backgrounds = [];

// === ФУНКЦИИ РИСОВАНИЯ ===

// Фоновые дома (СНГ-стиль)
function spawnHouse() {
    backgrounds.push({ x: 800, w: 100 + Math.random()*150, h: 180 + Math.random()*120 });
}

// Пиксельная металлическая мусорка (по фото)
function drawTrash(obs) {
    ctx.fillStyle = '#c0c0c0'; // Серебряный корпус
    ctx.fillRect(obs.x, obs.y, obs.w, obs.h);
    
    ctx.fillStyle = '#a0a0a0'; // Темные полосы (как на фото)
    for (let i=5; i<obs.w; i+=10) ctx.fillRect(obs.x+i, obs.y+5, 3, obs.h-10);
    
    ctx.fillStyle = '#909090'; // Крышка
    ctx.fillRect(obs.x - 2, obs.y - 5, obs.w + 4, 10);
    ctx.fillStyle = '#c0c0c0'; // Ручка на крышке
    ctx.fillRect(obs.x + obs.w/2 - 5, obs.y - 10, 10, 5);
}

// Препятствия (СНГ-стиль)
function spawnObstacle() {
    let r = Math.random();
    let obs = { x: 800, passed: false };
    if (r < 0.25) { // Обычный забор
        obs.w = 30; obs.h = 40; obs.type = 'fence';
    } else if (r < 0.5) { // Большой забор
        obs.w = 60; obs.h = 50; obs.type = 'fence';
    } else if (r < 0.75) { // Мусорка (ОБНОВЛЕННАЯ)
        obs.w = 40; obs.h = 60; obs.type = 'trash';
    } else { // Голубь
        obs.w = 30; obs.h = 20; obs.type = 'bird'; obs.y = 220;
    }
    if (obs.type !== 'bird') obs.y = 340 - obs.h;
    obstacles.push(obs);
}

// Пиксельный золотой кубок
function drawCup(x, y, w, h) {
    ctx.fillStyle = "gold";
    ctx.fillRect(x, y, w, h); // Чаша
    ctx.fillRect(x-w*0.2, y-h*0.2, w*1.4, h*0.2); // Ободок
    ctx.fillRect(x+w/2-w*0.1, y+h, w*0.2, h*0.3); // Ножка
    ctx.fillRect(x, y+h*1.2, w, h*0.2); // Подставка
    // Ручки
    ctx.fillRect(x-w*0.3, y+h*0.1, w*0.3, h*0.6); 
    ctx.fillRect(x+w, y+h*0.1, w*0.3, h*0.6);
    ctx.fillStyle = "darkgoldenrod"; // Тень
    ctx.fillRect(x+w-2, y+10, 2, h-10);
}

function reset() {
    score = 0;
    gameSpeed = 6;
    obstacles = [];
    backgrounds = [];
    player.y = 340 - player.h;
    player.dy = 0;
}

function drawText(text, x, y, size, color = "white", align = "center") {
    ctx.font = `bold ${size} Courier New`;
    ctx.textAlign = align;
    ctx.strokeStyle = "black";
    ctx.lineWidth = 4;
    ctx.strokeText(text, x, y);
    ctx.fillStyle = color;
    ctx.fillText(text, x, y);
}

// === ОБНОВЛЕНИЕ ЛОГИКИ (ИСПРАВЛЕНЫ ОЧКИ) ===
function update() {
    if (gameState !== 'PLAYING') return;

    frame++;
    if (frame % 120 === 0) spawnHouse();
    if (frame % 90 === 0) spawnObstacle();

    // Гравитация
    player.dy += player.gravity;
    player.y += player.dy;
    if (player.y > 340 - player.h) {
        player.y = 340 - player.h;
        player.dy = 0;
        player.grounded = true;
    }

    // Движение фона
    backgrounds.forEach((b, i) => {
        b.x -= gameSpeed * 0.4;
        if (b.x + b.w < 0) backgrounds.splice(i, 1);
    });

    // Движение препятствий
    obstacles.forEach((o, i) => {
        o.x -= gameSpeed;
        // Коллизия
        if (player.x < o.x + o.w && player.x + player.w > o.x && player.y < o.y + o.h && player.y + player.h > o.y) {
            gameState = 'GAMEOVER';
            
            // ЛОГИКА РЕКОРДА (ИСПРАВЛЕНА): Не выше 5000
            let finalScore = score;
            if (finalScore > 5000) finalScore = 5000;
            if (finalScore > highScore) {
                highScore = finalScore;
                localStorage.setItem('sng_score', highScore);
            }
        }
        if (o.x + o.w < 0) obstacles.splice(i, 1);
    });

    score++;
    if (score % 600 === 0) gameSpeed += 0.4;

    // Победа на 5000
    if (score >= 5000 && gameState === 'PLAYING') {
        gameState = 'WIN';
        hasCup = true;
        localStorage.setItem('sng_cup', 'true');
        highScore = 5000; // Рекорд фиксируем на 5к
        localStorage.setItem('sng_score', highScore);
    }
}

// === ОТРИСОВКА ===
function draw() {
    ctx.fillStyle = '#777'; // Серый фон неба
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Рисуем дома (СНГ-стиль)
    backgrounds.forEach(b => {
        ctx.fillStyle = '#444'; ctx.fillRect(b.x, 340 - b.h, b.w, b.h);
        ctx.fillStyle = '#666'; // Окна
        for(let i=10; i<b.w-10; i+=25) {
            for(let j=10; j<b.h-20; j+=40) ctx.fillRect(b.x+i, (340-b.h)+j, 12, 18);
        }
    });

    // Земля
    ctx.fillStyle = '#333';
    ctx.fillRect(0, 340, canvas.width, 60);

    if (gameState === 'MENU') {
        drawText("SNG RUNNER", 400, 150, "50px");
        drawText(`РЕКОРД: ${highScore}`, 400, 190, "20px");
        
        // Кнопка Играть
        ctx.fillStyle = "#fff"; ctx.fillRect(300, 220, 200, 60);
        drawText("ИГРАТЬ", 400, 258, "25px", "black");
        
        // Кнопка Выйти
        ctx.fillStyle = "#fff"; ctx.fillRect(20, 340, 100, 40);
        drawText("ВЫХОД", 70, 365, "15px", "black");

        if (hasCup) drawCup(385, 40, 30, 30); // Рисуем кубок если есть победа

    } else if (gameState === 'PLAYING') {
        player.draw();
        obstacles.forEach(o => {
            if (o.type === 'bird') {
                ctx.fillStyle = "white"; ctx.fillRect(o.x, o.y, o.w, o.h);
                ctx.fillRect(o.x-5, o.y+5, 10, 5); // Крылья
            } else if (o.type === 'trash') {
                drawTrash(o); // ОБНОВЛЕННАЯ МУСОРКА ПО ФОТО
            } else {
                // Забор (СНГ-стиль)
                ctx.fillStyle = "#8b4513"; ctx.fillRect(o.x, o.y, o.w, o.h); 
                ctx.fillStyle = "#A0522D"; ctx.fillRect(o.x, o.y+10, obs.w, 3); ctx.fillRect(obs.x, obs.y+25, obs.w, 3);
            }
        });
        drawText(`ОЧКИ: ${score}`, 700, 40, "20px");

    } else if (gameState === 'WIN') {
        ctx.fillStyle = "white"; ctx.fillRect(500, 240, 150, 100); // Остановка
        ctx.fillStyle = "green"; ctx.fillRect(520, 270, 110, 70); // Автобус
        drawText("YOU WIN", 400, 150, "60px", "gold");
        drawText("Ты сел в автобус и уехал!", 400, 200, "20px");
        drawText("Нажми, чтобы в меню", 400, 380, "15px");

    } else if (gameState === 'GAMEOVER') {
        player.draw(); // Показываем Скотта
        drawText("GAME OVER", 400, 200, "50px", "red");
        drawText("Нажми, чтобы переиграть", 400, 240, "20px");
    }
}

function loop() {
    update(); draw(); requestAnimationFrame(loop);
}

// Управление тапом
canvas.addEventListener('touchstart', (e) => {
    e.preventDefault();
    const rect = canvas.getBoundingClientRect();
    const touchX = (e.touches[0].clientX - rect.left) * (canvas.width / rect.width);
    const touchY = (e.touches[0].clientY - rect.top) * (canvas.height / rect.height);

    if (gameState === 'MENU') {
        if (touchX > 300 && touchX < 500 && touchY > 220 && touchY < 280) { reset(); gameState = 'PLAYING'; }
    } else if (gameState === 'PLAYING') {
        if (player.grounded) { player.dy = player.jump; player.grounded = false; }
    } else {
        gameState = 'MENU';
    }
});

loop();
</script>
</body>
</html>
