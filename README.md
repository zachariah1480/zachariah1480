// Set up game variables
let playerPos = { x: 400, y: 300 };
let playerSpeed = 3;
let arrows = [];
let arrowSpeed = 100;  // Arrow speed set to 100 blocks per second
let bowLength = 60;  // Bowstring length
let pullStrength = 0;  // Tracks bow pull
let arrowRadius = 5;
let isShooting = false;

// Draw the player, bow, and arrows
function drawGameObjects() {
  // Clear the canvas
  context.clearRect(0, 0, canvas.width, canvas.height);

  // Draw the player
  context.beginPath();
  context.arc(playerPos.x, playerPos.y, 20, 0, 2 * Math.PI);
  context.fillStyle = "blue";
  context.fill();

  // Get mouse position for aiming
  let mousePos = getMousePos(canvas, event);

  // Draw the bow
  let angle = Math.atan2(mousePos.y - playerPos.y, mousePos.x - playerPos.x);
  let bowStartX = playerPos.x + 20 * Math.cos(angle);
  let bowStartY = playerPos.y + 20 * Math.sin(angle);
  let bowEndX = playerPos.x + (bowLength + pullStrength) * Math.cos(angle);
  let bowEndY = playerPos.y + (bowLength + pullStrength) * Math.sin(angle);

  // Draw the bowstring
  context.beginPath();
  context.moveTo(playerPos.x, playerPos.y);
  context.lineTo(bowEndX, bowEndY);
  context.lineWidth = 3;
  context.strokeStyle = "brown";
  context.stroke();

  // Draw arrows
  arrows.forEach((arrow) => {
    context.beginPath();
    context.arc(arrow.pos.x, arrow.pos.y, arrowRadius, 0, 2 * Math.PI);
    context.fillStyle = "green";
    context.fill();
  });
}

// Shoot the arrow
function shootArrow() {
  let mousePos = getMousePos(canvas, event);
  let angle = Math.atan2(mousePos.y - playerPos.y, mousePos.x - playerPos.x);
  let arrow = {
    pos: { x: playerPos.x, y: playerPos.y },
    speed: arrowSpeed + pullStrength,
    angle: angle,
    distanceTraveled: 0  // New property to track distance
  };
  arrows.push(arrow);
  pullStrength = 0; // Reset pull strength after shooting
}

// Update arrows
function updateArrows() {
  arrows.forEach((arrow, index) => {
    // Calculate the distance traveled since the last frame
    let prevX = arrow.pos.x;
    let prevY = arrow.pos.y;

    let angleToPlayer = Math.atan2(playerPos.y - arrow.pos.y, playerPos.x - arrow.pos.x);
    arrow.pos.x += Math.cos(angleToPlayer) * arrow.speed;
    arrow.pos.y += Math.sin(angleToPlayer) * arrow.speed;

    // Calculate the distance between the previous and current position
    let distance = Math.sqrt(Math.pow(arrow.pos.x - prevX, 2) + Math.pow(arrow.pos.y - prevY, 2));
    arrow.distanceTraveled += distance;  // Update the total distance traveled

    // Remove arrows that travel more than 150 blocks
    if (arrow.distanceTraveled >= 150) {
      arrows.splice(index, 1);  // Remove the arrow from the array
    }
  });
}

// Mouse handling
canvas.addEventListener('mousemove', (event) => {
  let mousePos = getMousePos(canvas, event);
  let angle = Math.atan2(mousePos.y - playerPos.y, mousePos.x - playerPos.x);
  let bowEndX = playerPos.x + (bowLength + pullStrength) * Math.cos(angle);
  let bowEndY = playerPos.y + (bowLength + pullStrength) * Math.sin(angle);
  // You can add logic here to handle pulling the bow
});

canvas.addEventListener('mousedown', () => {
  isShooting = true;
});

canvas.addEventListener('mouseup', () => {
  if (isShooting) {
    shootArrow();
    isShooting = false;
  }
});

// Game loop
function gameLoop() {
  drawGameObjects();
  updateArrows();
  requestAnimationFrame(gameLoop);
}

gameLoop();
