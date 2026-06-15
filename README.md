# Cube-dahs
Jump fast
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cube Dash</title>
    <style>
        body {
            margin: 0;
            background: #111;
            color: white;
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        canvas {
            background: linear-gradient(to bottom, #1a0033, #330066);
            border: 4px solid #00ffff;
            box-shadow: 0 0 20px #00ffff;
            border-radius: 5px;
        }
        #info {
            margin-top: 15px;
            font-size: 18px;
            text-align: center;
        }
    </style>
</head>
<body>

    <canvas id="gameCanvas" width="800" height="400"></canvas>
    <div id="info">Press <strong>SPACEBAR</strong> or <strong>TAP/CLICK</strong> to Jump</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Game variables
let score = 0;
let highScore = 0;
let gameOver = false;
let gameSpeed = 6;

// Player (Cube) object
const player = {
    x: 100,
    y: 300,
    size: 40,
    color: "#00ffff",
    gravity: 0.6,
    velocity: 0,
    jumpStrength: -12,
    isGrounded: false,
    rotation: 0
};

// Obstacles array
let obstacles = [];
const floorY = canvas.height - 60;

// Input Listeners
window.addEventListener("keydown", (e) => {
    if (e.code === "Space") {
        handleJump();
    }
});
canvas.addEventListener("touchstart", handleJump);
canvas.addEventListener("mousedown", handleJump);

function handleJump() {
    if (gameOver) {
        resetGame();
    } else if (player.isGrounded) {
        player.velocity = player.jumpStrength;
        player.isGrounded = false;
    }
}

// Spawn obstacles randomly
function spawnObstacle() {
    if (gameOver) return;
    
    // Ensure minimum distance between spikes
    if (obstacles.length === 0 || obstacles[obstacles.length - 1].x < canvas.width - 300) {
        if (Math.random() < 0.02) { 
            obstacles.push({
                x: canvas.width,
                width: 30,
                height: 40,
                color: "#ff0055"
            });
        }
    }
}

function resetGame() {
    score = 0;
    gameSpeed = 6;
    obstacles = [];
    player.y = floorY - player.size;
    player.velocity = 0;
    player.rotation = 0;
    player.isGrounded = true;
    gameOver = false;
    animate();
}

// Main Game Loop
function animate() {
    if (gameOver) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 1. Draw Floor
    ctx.fillStyle = "#222";
    ctx.fillRect(0, floorY, canvas.width, canvas.height - floorY);
    ctx.strokeStyle = "#00ffff";
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(0, floorY);
    ctx.lineTo(canvas.width, floorY);
    ctx.stroke();

    // 2. Player Physics & Movement
    player.velocity += player.gravity;
    player.y += player.velocity;

    // Floor collision
    if (player.y + player.size >= floorY) {
        player.y = floorY - player.size;
        player.velocity = 0;
        player.isGrounded = true;
        // Snap rotation to nearest 90 degrees when landing
        player.rotation = Math.round(player.rotation / 90) * 90;
    } else {
        // Rotate while in the air
        player.rotation += 4; 
    }

    // 3. Draw Player (with Rotation)
    ctx.save();
    ctx.translate(player.x + player.size / 2, player.y + player.size / 2);
    ctx.rotate((player.rotation * Math.PI) / 180);
    
    // Cube face
    ctx.fillStyle = player.color;
    ctx.fillRect(-player.size / 2, -player.size / 2, player.size, player.size);
    ctx.strokeStyle = "#fff";
    ctx.strokeRect(-player.size / 2, -player.size / 2, player.size, player.size);
    
    // Cube eyes (to give it personality)
    ctx.fillStyle = "#000";
    ctx.fillRect(5, -12, 6, 6);
    ctx.fillRect(15, -12, 6, 6);
    ctx.restore();

    // 4. Handle Obstacles (Spikes)
    spawnObstacle();

    for (let i = obstacles.length - 1; i >= 0; i--) {
        let obs = obstacles[i];
        obs.x -= gameSpeed;

        // Draw Triangle Spike
        ctx.fillStyle = obs.color;
        ctx.beginPath();
        ctx.moveTo(obs.x, floorY);
        ctx.lineTo(obs.x + obs.width / 2, floorY - obs.height);
        ctx.lineTo(obs.x + obs.width, floorY);
        ctx.closePath();
        ctx.fill();
        ctx.strokeStyle = "#fff";
        ctx.stroke();

        // Collision Detection (AABB vs Triangle approximation)
        if (
            player.x + player.size - 5 > obs.x &&
            player.x + 5 < obs.x + obs.width &&
            player.y + player.size > floorY - obs.height
        ) {
            triggerGameOver();
        }

        // Score tracking
        if (obs.x + obs.width < player.x && !obs.passed) {
            obs.passed = true;
            score++;
            if (score > highScore) highScore = score;
            // Gradually increase speed
            if (score % 5 === 0) gameSpeed += 0.5;
        }

        // Remove off-screen obstacles
        if (obs.x + obs.width < 0) {
            obstacles.splice(i, 1);
        }
    }

    // 5. UI text
    ctx.fillStyle = "#fff";
    ctx.font = "20px sans-serif";
    ctx.fillText(`Score: ${score}`, 20, 40);
    ctx.fillText(`Best: ${highScore}`, 20, 70);

    requestAnimationFrame(animate);
}

function triggerGameOver() {
    gameOver = true;
    ctx.fillStyle = "rgba(0, 0, 0, 0.8)";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = "#ff0055";
    ctx.font = "bold 40px sans-serif";
    ctx.textAlign = "center";
    ctx.fillText("GAME OVER", canvas.width / 2, canvas.height / 2 - 20);

    ctx.fillStyle = "#fff";
    ctx.font = "20px sans-serif";
    ctx.fillText("Click or Press Space to Retry", canvas.width / 2, canvas.height / 2 + 30);
    ctx.textAlign = "left"; // Reset alignment
}

// Start game initially
resetGame();
</script>
</body>
</html>
