DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BloxD.io Hack Client with Aimbot Bow and Arrows</title>
    <style>
        canvas { background-color: #f0f0f0; display: block; margin: 0 auto; }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Player settings
        const player1 = { x: 100, y: 100, width: 50, height: 50, speed: 5, angle: 0 };
        const player2 = { x: 600, y: 300, width: 50, height: 50, speed: 5, angle: 0 };

        // Arrow settings
        const arrows = [];
        const arrowSpeed = 5;
        const arrowLife = 200;  // Distance after which the arrow will disappear
        const splitDistance = 50;  // Distance to split the arrow (set to 50 blocks)
        const maxArrows = 5;  // The number of arrows to split into
        const arrowDamage = 1000;  // Damage per arrow
        const knockbackForce = 1000;  // Knockback strength for the arrow

        let isRapidFireActive = false; // Track if the rapid fire (E key) is active

        // Calculate the angle between player and mouse position
        function calculateAngleToMouse(player, mouse) {
            const dx = mouse.x - (player.x + player.width / 2);
            const dy = mouse.y - (player.y + player.height / 2);
            return Math.atan2(dy, dx);
        }

        // Create and shoot an arrow in the calculated direction
        function shootArrow() {
            const angle = calculateAngleToMouse(player1, mouse); // Player 1 aims
            const arrow = {
                x: player1.x + player1.width / 2,
                y: player1.y + player1.height / 2,
                angle: angle,
                speed: arrowSpeed,
                life: arrowLife,
                split: false,  // Flag to indicate if the arrow should split
                target: player2 // Initially targeting player 2
            };
            arrows.push(arrow);
        }

        // Update arrow positions and handle movement
        function updateArrows() {
            for (let i = 0; i < arrows.length; i++) {
                const arrow = arrows[i];
                // Move the arrow based on its angle
                const moveX = Math.cos(arrow.angle) * arrow.speed;
                const moveY = Math.sin(arrow.angle) * arrow.speed;
                arrow.x += moveX;
                arrow.y += moveY;

                // Decrease the arrow's life
                arrow.life--;

                // If arrow detects nearby players, it splits
                if (!arrow.split) {
                    const playerDistance = Math.sqrt(
                        (player1.x - player2.x) ** 2 + (player1.y - player2.y) ** 2
                    );

                    if (playerDistance <= splitDistance) {
                        // Split the arrow when players are close (now 50 blocks)
                        arrow.split = true;
                        splitArrow(arrow);
                    }
                }

                // Check for collision with players (simple collision logic)
                const distanceToPlayer1 = Math.sqrt(
                    (player1.x + player1.width / 2 - arrow.x) ** 2 +
                    (player1.y + player1.height / 2 - arrow.y) ** 2
                );
                const distanceToPlayer2 = Math.sqrt(
                    (player2.x + player2.width / 2 - arrow.x) ** 2 +
                    (player2.y + player2.height / 2 - arrow.y) ** 2
                );

                if (distanceToPlayer1 < 10 || distanceToPlayer2 < 10) {
                    arrows.splice(i, 1);
                    i--;  // Adjust the index after removal
                }

                // Remove the arrow if it has exceeded its life
                if (arrow.life <= 0) {
                    arrows.splice(i, 1);
                    i--; // Adjust index after removal
                }
            }
        }

        // Function to split the arrow into 5 and target the players separately
        function splitArrow(arrow) {
            // Split into 5 arrows, each pointing towards different players or directions
            for (let i = 0; i < maxArrows; i++) {
                const angleOffset = (Math.random() - 0.5) * Math.PI / 6; // Small random angle offset for diversity
                const newArrow = {
                    ...arrow,
                    angle: arrow.angle + angleOffset,
                    life: arrowLife, // Reset the life for each new arrow
                    target: i % 2 === 0 ? player1 : player2 // Alternate between targeting player1 and player2
                };
                arrows.push(newArrow);  // Add each new arrow to the array
            }
        }

        // Draw the player and the arrows
        function drawGame() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);  // Clear canvas

            // Draw players (blue and green squares)
            ctx.fillStyle = 'blue';
            ctx.fillRect(player1.x, player1.y, player1.width, player1.height);
            ctx.fillStyle = 'green';
            ctx.fillRect(player2.x, player2.y, player2.width, player2.height);

            // Draw the bow (represented as a line from the player to the mouse position)
            ctx.strokeStyle = 'brown';
            ctx.lineWidth = 5;
            ctx.beginPath();
            ctx.moveTo(player1.x + player1.width / 2, player1.y + player1.height / 2);
            ctx.lineTo(mouse.x, mouse.y);  // Line from player to mouse (bow)
            ctx.stroke();

            // Draw arrows (red circles)
            ctx.fillStyle = 'red';
            for (const arrow of arrows) {
                ctx.beginPath();
                ctx.arc(arrow.x, arrow.y, 5, 0, Math.PI * 2);  // Draw the arrow as a circle
                ctx.fill();
            }
        }

        // Update player movement based on keyboard input
        document.addEventListener('keydown', (event) => {
            if (event.key === 'ArrowUp') player1.y -= player1.speed;
            if (event.key === 'ArrowDown') player1.y += player1.speed;
            if (event.key === 'ArrowLeft') player1.x -= player1.speed;
            if (event.key === 'ArrowRight') player1.x += player1.speed;

            if (event.key === 'w') player2.y -= player2.speed;
            if (event.key === 's') player2.y += player2.speed;
            if (event.key === 'a') player2.x -= player2.speed;
            if (event.key === 'd') player2.x += player2.speed;

            // Activate rapid-fire when E is pressed
            if (event.key === 'e' || event.key === 'E') {
                if (isRapidFireActive) {
                    shootArrow(); // Shoot continuously if rapid fire is active
                }
            }

            // Deactivate rapid-fire when Q is pressed
            if (event.key === 'q' || event.key === 'Q') {
                isRapidFireActive = !isRapidFireActive;  // Toggle rapid fire on or off
            }
        });

        document.addEventListener('keyup', (event) => {
            if (event.key === 'e' || event.key === 'E') {
                // Stop shooting when E key is released
            }
        });

        // Mouse position tracking
        const mouse = { x: 0, y: 0 };
        canvas.addEventListener('mousemove', (event) => {
            const rect = canvas.getBoundingClientRect();
            mouse.x = event.clientX - rect.left;
            mouse.y = event.clientY - rect.top;
        });

        // Shoot an arrow when mouse is clicked and handle rapid-fire with E key
        function handleRapidFire() {
            if (isRapidFireActive) {
                shootArrow(); // Shoot continuously when the E key is pressed
            }
        }

        // Handle knockback effect when the arrow hits the player
        function applyKnockback(arrow, target) {
            const dx = target.x + target.width / 2 - arrow.x;
            const dy = target.y + target.height / 2 - arrow.y;
            const distance = Math.sqrt(dx * dx + dy * dy);
            const knockbackX = (dx / distance) * knockbackForce;
            const knockbackY = (dy / distance) * knockbackForce;

            // Apply knockback to target (player)
            target.x += knockbackX;
            target.y += knockbackY;
        }

        // Game loop to continuously update the game state
        function gameLoop() {
            updateArrows();
            handleRapidFire();
            drawGame();

            requestAnimationFrame(gameLoop);  // Repeat the loop
        }

        gameLoop();  // Start the game loop
    </script>
</body>
</html>
