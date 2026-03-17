# Web Audio API Patterns

Common audio patterns for Giggle OS apps. All sound is generated programmatically — no external audio files.

## Audio Context (Lazy Init)

```js
let audioCtx;
function getAudioCtx() {
  if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  return audioCtx;
}
```

Always init on first user interaction (touchstart/click) to comply with autoplay policies.

## Simple Beep

```js
function playBeep(freq = 440, duration = 0.1, type = 'sine') {
  const ctx = getAudioCtx();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = type; // 'sine', 'square', 'sawtooth', 'triangle'
  osc.frequency.value = freq;
  gain.gain.setValueAtTime(0.3, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);
  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.start();
  osc.stop(ctx.currentTime + duration);
}
```

## Musical Note with Envelope (ADSR)

```js
function playNote(freq, duration = 0.5, type = 'sine') {
  const ctx = getAudioCtx();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = type;
  osc.frequency.value = freq;

  const now = ctx.currentTime;
  gain.gain.setValueAtTime(0, now);
  gain.gain.linearRampToValueAtTime(0.4, now + 0.01);  // Attack
  gain.gain.linearRampToValueAtTime(0.3, now + 0.1);   // Decay → Sustain
  gain.gain.linearRampToValueAtTime(0, now + duration); // Release

  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.start(now);
  osc.stop(now + duration);
}
```

## Noise Burst (Percussion)

```js
function playNoise(duration = 0.1) {
  const ctx = getAudioCtx();
  const bufferSize = ctx.sampleRate * duration;
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
  const data = buffer.getChannelData(0);
  for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;

  const source = ctx.createBufferSource();
  const gain = ctx.createGain();
  source.buffer = buffer;
  gain.gain.setValueAtTime(0.3, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);
  source.connect(gain);
  gain.connect(ctx.destination);
  source.start();
}
```

## Equal Temperament Frequencies

```js
// semitone 0 = A4 (440Hz)
function noteFreq(semitone) {
  return 440 * Math.pow(2, semitone / 12);
}

// C4=261.63, D4=293.66, E4=329.63, F4=349.23
// G4=392.00, A4=440.00, B4=493.88, C5=523.25
```

## Common Sound Effects

| Sound       | Freq   | Duration | Type     |
|-------------|--------|----------|----------|
| Click       | 800Hz  | 0.05s    | sine     |
| Jump        | 523Hz  | 0.10s    | sine     |
| Collect     | 880Hz  | 0.05s    | sine     |
| Error       | 200Hz  | 0.30s    | square   |
| Win         | 660Hz  | 0.20s    | triangle |
| Pop         | 600Hz  | 0.06s    | sine     |
