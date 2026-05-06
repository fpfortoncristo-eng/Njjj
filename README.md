<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GameZone Arcade</title>
    <style>
        :root {
            --bg-color: #0b061f;
            --card-flappy: linear-gradient(135deg, #4e54ff, #9146ff);
            --card-snake: linear-gradient(135deg, #00a884, #116b5a);
            --card-space: linear-gradient(135deg, #4b138d, #1f1147);
        }

        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: white;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* Menú Principal */
        #menu {
            width: 100%;
            max-width: 500px;
            padding: 20px;
            box-sizing: border-box;
        }

        .header {
            text-align: center;
            margin-bottom: 30px;
        }

        .header h1 {
            font-size: 3rem;
            margin: 0;
            font-weight: 900;
        }

        .game-card {
            border-radius: 25px;
            padding: 25px;
            margin-bottom: 20px;
            position: relative;
            cursor: pointer;
            transition: transform 0.2s;
            overflow: hidden;
        }

        .game-card:active { transform: scale(0.98); }

        .flappy { background: var(--card-flappy); }
        .snake { background: var(--card-snake); }
        .space { background: var(--card-space); }

        .difficulty {
            position: absolute;
            top: 20px;
            right: 20px;
            background: rgba(255, 255, 255, 0.2);
            padding: 5px 15px;
            border-radius: 15px;
            font-size: 0.8rem;
            text-transform: uppercase;
        }

        /* Contenedor de Juegos */
        #game-container {
            display: none;
            width: 100vw;
            height: 100vh;
            background: #000;
            position: fixed;
            top: 0;
            left: 0;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        canvas {
            background: #111;
            border: 2px solid #333;
            max-width: 90%;
            max-height: 70%;
        }

        .controls {
            margin-top: 20px;
            display: flex;
            gap: 10px;
        }

        button {
            padding: 15px 25px;
            border-radius: 10px;
            border: none;
            font-weight: bold;
            cursor: pointer;
        }

        .btn-exit { background: #ff4e4e; color: white; }
        .btn-whatsapp { background: #25d366; color: white; }
    </style>
</head>
<body>

    <div id="menu">
        <div class="header">
            <p>⚡ ARCADE</p>
            <h1>GameZone</h1>
            <p>Selecciona un juego y compite</p>
        </div>

        <div class="game-card flappy" onclick="startGame('flappy')">
            <span class="difficulty">Difícil</span>
            <h2>Flappy Bird</h2>
            <p>Esquiva los tubos volando.</p>
            <small>Toca para volar</small>
        </div>

        <div class="game-card snake" onclick="startGame('snake')">
            <span class="difficulty">Medio</span>
            <h2>Snake</h2>
            <p>Come manzanas y crece.</p>
            <small>Desliza para mover</small>
        </div>

        <div class="game-card space" onclick="startGame('space')">
            <span class="difficulty">Fácil</span>
            <h2>Space Invaders</h2>
            <p>Elimina las oleadas de aliens.</p>
            <small>Toca lados para mover / Centro dispara</small>
        </div>
    </div>

    <div id="game-container">
        <h2 id="game-title">Juego</h2>
        <canvas id="gameCanvas"></canvas>
        <div id="score-display">Puntaje: 0</div>
        
        <div class="controls">
            <button class="btn-exit" onclick="exitGame()">Salir</button>
            <button class="btn-whatsapp" onclick="shareScore()">Compartir en WhatsApp</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        let currentScore = 0;
        let currentGame = null;
        let gameInterval = null;

        // Ajustar canvas para móvil
        canvas.width = 320;
        canvas.height = 480;

        function startGame(gameType) {
            document.getElementById('menu').style.display = 'none';
            document.getElementById('game-container').style.display = 'flex';
            currentGame = gameType;
            currentScore = 0;
            
            if(gameInterval) clearInterval(gameInterval);

            if(gameType === 'snake') initSnake();
            else if(gameType === 'flappy') initFlappy();
            else if(gameType === 'space') initSpace();
        }

        function exitGame() {
            location.reload(); // Forma más rápida de limpiar los estados de los juegos
        }

        function shareScore() {
            const text = `¡Mira mi puntaje en GameZone! Hice ${currentScore} puntos en ${currentGame}. ¿Puedes superarme?`;
            const url = `https://wa.me/?text=${encodeURIComponent(text)}`;
            window.open(url, '_blank');
        }

        // --- LÓGICA SIMPLIFICADA DE SNAKE ---
        function initSnake() {
            let snake = [{x: 10, y: 10}];
            let food = {x: 15, y: 15};
            let dx = 1, dy = 0;
            
            gameInterval = setInterval(() => {
                const head = {x: snake[0].x + dx, y: snake[0].y + dy};
                snake.unshift(head);

                if(head.x === food.x && head.y === food.y) {
                    currentScore += 10;
                    food = {x: Math.floor(Math.random()*20), y: Math.floor(Math.random()*30)};
                } else {
                    snake.pop();
                }

                // Dibujar
                ctx.fillStyle = "black"; ctx.fillRect(0,0,320,480);
                ctx.fillStyle = "lime";
                snake.forEach(p => ctx.fillRect(p.x*16, p.y*16, 14, 14));
                ctx.fillStyle = "red";
                ctx.fillRect(food.x*16, food.y*16, 14, 14);
                
                document.getElementById('score-display').innerText = "Puntaje: " + currentScore;
            }, 150);

            // Control táctil básico
            canvas.onclick = () => { if(dx === 1) { dx=0; dy=1; } else if(dy === 1) { dx=-1; dy=0; } else if(dx === -1) { dx=0; dy=-1; } else { dx=1; dy=0; } };
        }

        // --- LÓGICA SIMPLIFICADA FLAPPY ---
        function initFlappy() {
            let birdY = 200, velocity = 0;
            gameInterval = setInterval(() => {
                velocity += 0.5;
                birdY += velocity;
                ctx.fillStyle = "black"; ctx.fillRect(0,0,320,480);
                ctx.fillStyle = "yellow";
                ctx.fillRect(50, birdY, 20, 20);
                if(birdY > 480) birdY = 200;
                currentScore++;
                document.getElementById('score-display').innerText = "Puntaje: " + currentScore;
            }, 30);
            canvas.onclick = () => { velocity = -8; };
        }

        // --- LÓGICA SIMPLIFICADA SPACE INVADERS ---
        function initSpace() {
            let playerX = 150;
            gameInterval = setInterval(() => {
                ctx.fillStyle = "black"; ctx.fillRect(0,0,320,480);
                ctx.fillStyle = "#9146ff";
                ctx.fillRect(playerX, 440, 30, 20);
                currentScore += 1;
                document.getElementById('score-display').innerText = "Puntaje: " + currentScore;
            }, 50);
            canvas.onclick = (e) => {
                let rect = canvas.getBoundingClientRect();
                let x = e.clientX - rect.left;
                if(x < 160) playerX -= 20; else playerX += 20;
            };
        }
    </script>
</body>
</html>
