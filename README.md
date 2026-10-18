<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dino Chrome Runner</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
        }

        body {
            background-color: #f7f7f7;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            color: #535353;
        }

        .container {
            width: 100%;
            max-width: 700px;
            padding: 10px;
            text-align: center;
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
            padding: 0 10px;
            font-weight: bold;
            font-size: 18px;
        }

        #gameCanvas {
            background-color: #ffffff;
            border: 2px solid #535353;
            border-radius: 6px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
            width: 100%;
            height: auto;
            touch-action: manipulation;
        }

        .controls-hint {
            margin-top: 12px;
            font-size: 14px;
            color: #757575;
        }
    </style>
</head>
<body>

    <div class="container">
        <div class="header">
            <div>DINO RUN</div>
            <div>HI: <span id="highScore">00000</span> | SCORE: <span id="score">00000</span></div>
        </div>

        <canvas id="gameCanvas" width="600" height="200"></canvas>

        <div class="controls-hint">
            Presiona <b>ESPACIO</b> / <b>FLECHA ARRIBA</b> o <b>TOCA LA PANTALLA</b> para saltar.
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const highScoreElement = document.getElementById('highScore');

        // Variables del Juego
        let gameSpeed = 5;
        let score = 0;
        let highScore = localStorage.getItem('dino_high_score') || 0;
        let gameOver = false;
        let gameStarted = false;
        let frameCount = 0;

        highScoreElement.textContent = String(highScore).padStart(5, '0');

        // Dinosaurio
        const dino = {
            x: 50,
            y: 140,
            width: 40,
            height: 44,
            dy: 0,
            gravity: 0.6,
            jumpForce: -11,
            groundY: 140,
            isJumping: false,
            legFrame: 0
        };

        // Cactus
        let obstacles = [];
        const cactusTypes = [
            { width: 18, height: 38 },
            { width: 32, height: 38 },
            { width: 45, height: 40 }
        ];

        // Nubes de fondo
        let clouds = [
            { x: 200, y: 30, speed: 0.5 },
            { x: 450, y: 50, speed: 0.7 }
        ];

        // Controles
        function handleInput() {
            if (!gameStarted) {
                gameStarted = true;
                requestAnimationFrame(updateGame);
                return;
            }

            if (gameOver) {
                restartGame();
                return;
            }

            if (!dino.isJumping) {
                dino.dy = dino.jumpForce;
                dino.isJumping = true;
            }
        }

        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space' || e.code === 'ArrowUp') {
                e.preventDefault();
                handleInput();
            }
        });

        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            handleInput();
        });

        canvas.addEventListener('mousedown', () => {
            handleInput();
        });

        function spawnObstacle() {
            const type = cactusTypes[Math.floor(Math.random() * cactusTypes.length)];
            obstacles.push({
                x: canvas.width + 20,
                y: 184 - type.height,
                width: type.width,
                height: type.height
            });
        }

        function restartGame() {
            gameSpeed = 5;
            score = 0;
            gameOver = false;
            obstacles = [];
            dino.y = dino.groundY;
            dino.dy = 0;
            dino.isJumping = false;
            requestAnimationFrame(updateGame);
        }

        // Dibujo Pixel-Art del Dinosaurio
        function drawDino() {
            ctx.fillStyle = '#535353';
            
            // Cuerpo y Cabeza
            ctx.fillRect(dino.x + 12, dino.y, 22, 14); // Cabeza
            ctx.fillRect(dino.x + 28, dino.y + 4, 8, 4);  // Hocico
            ctx.fillRect(dino.x + 12, dino.y + 14, 18, 18); // Torso
            ctx.fillRect(dino.x + 4, dino.y + 18, 10, 10);  // Cola
            ctx.fillRect(dino.x + 22, dino.y + 18, 8, 4);   // Brazo

            // Ojo
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(dino.x + 18, dino.y + 2, 4, 4);

            // Piernas (Animación de carrera)
            ctx.fillStyle = '#535353';
            if (dino.isJumping) {
                ctx.fillRect(dino.x + 14, dino.y + 32, 4, 10);
                ctx.fillRect(dino.x + 24, dino.y + 32, 4, 6);
            } else {
                if (Math.floor(frameCount / 6) % 2 === 0) {
                    ctx.fillRect(dino.x + 14, dino.y + 32, 4, 12);
                    ctx.fillRect(dino.x + 24, dino.y + 32, 4, 6);
                } else {
                    ctx.fillRect(dino.x + 14, dino.y + 32, 4, 6);
                    ctx.fillRect(dino.x + 24, dino.y + 32, 4, 12);
                }
            }
        }

        // Dibujo de Cactus
        function drawCactus(obs) {
            ctx.fillStyle = '#535353';
            // Tallo central
            ctx.fillRect(obs.x + obs.width * 0.35, obs.y, obs.width * 0.3, obs.height);
            // Brazo izquierdo
            ctx.fillRect(obs.x, obs.y + obs.height * 0.3, obs.width * 0.35, obs.height * 0.2);
            ctx.fillRect(obs.x, obs.y + obs.height * 0.15, obs.width * 0.15, obs.height * 0.2);
            // Brazo derecho
            ctx.fillRect(obs.x + obs.width * 0.65, obs.y + obs.height * 0.4, obs.width * 0.35, obs.height * 0.2);
            ctx.fillRect(obs.x + obs.width * 0.85, obs.y + obs.height * 0.25, obs.width * 0.15, obs.height * 0.2);
        }

        // Bucle Principal del Juego
        function updateGame() {
            if (gameOver) return;

            frameCount++;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Suelo
            ctx.fillStyle = '#535353';
            ctx.fillRect(0, 182, canvas.width, 2);

            // Nubes
            clouds.forEach(cloud => {
                cloud.x -= cloud.speed;
                if (cloud.x < -40) cloud.x = canvas.width + 40;
                ctx.fillStyle = '#d3d3d3';
                ctx.fillRect(cloud.x, cloud.y, 30, 10);
                ctx.fillRect(cloud.x + 5, cloud.y - 5, 20, 15);
            });

            // Físicas del Dinosaurio
            dino.dy += dino.gravity;
            dino.y += dino.dy;

            if (dino.y >= dino.groundY) {
                dino.y = dino.groundY;
                dino.dy = 0;
                dino.isJumping = false;
            }

            drawDino();

            // Lógica de Cactus
            if (obstacles.length === 0 || (canvas.width - obstacles[obstacles.length - 1].x) > (180 + Math.random() * 200)) {
                spawnObstacle();
            }

            for (let i = obstacles.length - 1; i >= 0; i--) {
                let obs = obstacles[i];
                obs.x -= gameSpeed;

                drawCactus(obs);

                // Detección de Colisión (Hitbox con tolerancia)
                const margin = 5;
                if (
                    dino.x + margin < obs.x + obs.width &&
                    dino.x + dino.width - margin > obs.x &&
                    dino.y + margin < obs.y + obs.height &&
                    dino.y + dino.height - margin > obs.y
                ) {
                    gameOver = true;
                }

                // Eliminar obstáculos fuera de pantalla
                if (obs.x + obs.width < 0) {
                    obstacles.splice(i, 1);
                }
            }

            // Puntuación
            score++;
            scoreElement.textContent = String(Math.floor(score / 5)).padStart(5, '0');

            if (score / 5 > highScore) {
                highScore = Math.floor(score / 5);
                localStorage.setItem('dino_high_score', highScore);
                highScoreElement.textContent = String(highScore).padStart(5, '0');
            }

            // Incrementar Dificultad
            if (score % 500 === 0) {
                gameSpeed += 0.5;
            }

            if (gameOver) {
                ctx.fillStyle = '#535353';
                ctx.font = '20px monospace';
                ctx.fillText('G A M E   O V E R', canvas.width / 2 - 100, 90);
                ctx.font = '14px monospace';
                ctx.fillText('Presiona Espacio o Toca para reiniciar', canvas.width / 2 - 150, 120);
            } else {
                requestAnimationFrame(updateGame);
            }
        }

        // Pantalla Inicial
        ctx.fillStyle = '#535353';
        ctx.font = '16px monospace';
        ctx.fillText('PRESIONA ESPACIO O TOCA PARA JUGAR', 130, 100);
        ctx.fillRect(0, 182, canvas.width, 2);
        drawDino();
    </script>
</body>
</html>
