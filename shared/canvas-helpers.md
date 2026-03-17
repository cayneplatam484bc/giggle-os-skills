# Canvas Drawing Patterns

Common Canvas API patterns for Giggle OS apps.

## Canvas Setup

```js
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let W, H;

function resize() {
  W = canvas.width = window.innerWidth;
  H = canvas.height = window.innerHeight;
}
window.addEventListener('resize', resize);
resize();
```

## Drawing Smooth Lines

```js
ctx.lineCap = 'round';
ctx.lineJoin = 'round';
ctx.strokeStyle = '#333';
ctx.lineWidth = 8;

ctx.beginPath();
ctx.moveTo(x1, y1);
ctx.lineTo(x2, y2);
ctx.stroke();
```

## Drawing Emoji

```js
function drawEmoji(emoji, x, y, size) {
  ctx.font = `${size}px serif`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(emoji, x, y);
}
```

## Undo Stack

```js
let undoStack = [];

function saveState() {
  if (undoStack.length > 20) undoStack.shift();
  undoStack.push(canvas.toDataURL());
}

function undo() {
  if (undoStack.length === 0) return;
  const img = new Image();
  img.onload = () => {
    ctx.clearRect(0, 0, W, H);
    ctx.drawImage(img, 0, 0);
  };
  img.src = undoStack.pop();
}
```

## Simple Particle System

```js
const particles = [];

function spawnParticle(x, y, color) {
  particles.push({
    x, y, color,
    vx: (Math.random() - 0.5) * 6,
    vy: (Math.random() - 0.5) * 6 - 3,
    life: 1.0,
    size: 4 + Math.random() * 4
  });
}

function updateParticles() {
  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i];
    p.x += p.vx;
    p.y += p.vy;
    p.vy += 0.1; // gravity
    p.life -= 0.02;
    if (p.life <= 0) particles.splice(i, 1);
  }
}

function drawParticles() {
  particles.forEach(p => {
    ctx.globalAlpha = p.life;
    ctx.fillStyle = p.color;
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
    ctx.fill();
  });
  ctx.globalAlpha = 1;
}
```

## Animation Loop (30fps target for RPi)

```js
let lastTime = 0;
const FPS = 30;
const FRAME_TIME = 1000 / FPS;

function gameLoop(timestamp) {
  requestAnimationFrame(gameLoop);
  if (timestamp - lastTime < FRAME_TIME) return;
  lastTime = timestamp;

  // Update and draw here
}
requestAnimationFrame(gameLoop);
```

## Bounding Box Collision

```js
function collides(a, b) {
  return a.x < b.x + b.w &&
         a.x + a.w > b.x &&
         a.y < b.y + b.h &&
         a.y + a.h > b.y;
}
```

## Save Canvas as Image

```js
function saveCanvas() {
  const dataURL = canvas.toDataURL('image/png');
  // Can be stored, displayed, or downloaded
  return dataURL;
}
```
