# Touch Event Handling Patterns

Touch-first patterns for Giggle OS apps running on RPi 7" touchscreen (800x480).

## Core Principles

- Always use `touchstart` for immediate response (not `touchend`)
- Always `e.preventDefault()` on touch events to prevent scrolling/zooming
- Use `{ passive: false }` with touch event listeners
- Minimum tap target size: 44px (48px preferred)
- Provide visual feedback on every touch

## Prevent Default Behaviors

```js
// Global: prevent scroll, zoom, pull-to-refresh
document.addEventListener('touchmove', e => e.preventDefault(), { passive: false });
```

```html
<!-- In meta viewport -->
<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
```

```css
/* In CSS */
html, body {
  touch-action: none;
  -webkit-user-select: none;
  user-select: none;
  overflow: hidden;
}
```

## Tap Detection

```js
element.addEventListener('touchstart', e => {
  e.preventDefault();
  const x = e.touches[0].clientX;
  const y = e.touches[0].clientY;
  handleTap(x, y);
}, { passive: false });
```

## Drag / Move

```js
let dragging = false;

element.addEventListener('touchstart', e => {
  e.preventDefault();
  dragging = true;
}, { passive: false });

element.addEventListener('touchmove', e => {
  e.preventDefault();
  if (!dragging) return;
  const x = e.touches[0].clientX;
  const y = e.touches[0].clientY;
  handleDrag(x, y);
}, { passive: false });

element.addEventListener('touchend', () => { dragging = false; });
element.addEventListener('touchcancel', () => { dragging = false; });
```

## Canvas Drawing (Smooth Lines)

```js
let drawing = false, lastX, lastY;

canvas.addEventListener('touchstart', e => {
  e.preventDefault();
  drawing = true;
  lastX = e.touches[0].clientX;
  lastY = e.touches[0].clientY;
}, { passive: false });

canvas.addEventListener('touchmove', e => {
  e.preventDefault();
  if (!drawing) return;
  const x = e.touches[0].clientX;
  const y = e.touches[0].clientY;
  ctx.beginPath();
  ctx.moveTo(lastX, lastY);
  ctx.lineTo(x, y);
  ctx.stroke();
  lastX = x;
  lastY = y;
}, { passive: false });

canvas.addEventListener('touchend', () => { drawing = false; });
```

## Hit Testing (Circle)

```js
function hitTest(touchX, touchY, objX, objY, radius) {
  const dx = touchX - objX;
  const dy = touchY - objY;
  return dx * dx + dy * dy < radius * radius;
}
```

## Hit Testing (Rectangle / Bounding Box)

```js
function hitRect(tx, ty, rx, ry, rw, rh) {
  return tx >= rx && tx <= rx + rw && ty >= ry && ty <= ry + rh;
}
```

## Visual Feedback on Touch

```css
.btn:active {
  transform: scale(0.93);
  filter: brightness(1.2);
}
```

```js
// Programmatic feedback
element.addEventListener('touchstart', e => {
  e.preventDefault();
  element.classList.add('active');
  // ... action
}, { passive: false });

element.addEventListener('touchend', () => {
  element.classList.remove('active');
});
```
