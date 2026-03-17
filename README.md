# Giggle Skills

> The AI skill set powering **Giggle OS** — a creative tablet OS for kids ages 5–10, running on Raspberry Pi 5 with a 7" touchscreen.

A child describes what they want to build — in their own words, out loud or typed. The **Imagination Engine** (an AI skill loaded into any LLM) reads that description and generates a complete, working, interactive HTML app in seconds. No installs. No app store. Just imagination.

---

## Table of Contents

- [What Is This?](#what-is-this)
- [How It Works](#how-it-works)
- [The 6 Studios](#the-6-studios)
- [Using with Different LLMs](#using-with-different-llms)
  - [Claude (Anthropic)](#claude-anthropic)
  - [ChatGPT (OpenAI)](#chatgpt-openai)
  - [Gemini (Google)](#gemini-google)
  - [Local Models (Ollama, LM Studio)](#local-models-ollama-lm-studio)
  - [Open WebUI](#open-webui)
- [Project Structure](#project-structure)
- [Output Format](#output-format)
- [Design Principles](#design-principles)
- [Age Adaptation](#age-adaptation)
- [Tech Stack](#tech-stack)
- [Target Hardware](#target-hardware)
- [Extending the Skill](#extending-the-skill)
- [Examples](#examples)

---

## What Is This?

**Giggle Skills** is a collection of LLM skill definitions (system prompt files) that turn any capable AI model into a children's creative app generator.

The core idea: instead of downloading apps, kids *describe* what they want. The AI builds it live, right there on the device.

```
Kid: "I want a piano that's rainbow colored!"
AI:  → generates a complete working piano app in HTML
     → runs instantly in the built-in browser
     → kid plays it, no setup needed
```

Each "skill" is a structured `skill.md` file that instructs the LLM on:
- What kind of app to build (game, story, music, art, science, tool)
- What patterns and techniques to use
- How to adapt content for the child's age
- What constraints to follow (no external assets, touch-first, etc.)

---

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                      Giggle OS                          │
│                                                         │
│  Child speaks/types: "make me a maze game!"             │
│              │                                          │
│              ▼                                          │
│  ┌─────────────────────┐                                │
│  │   Imagination Engine │  ← skill loaded into LLM     │
│  │   (LLM + skill.md)  │                                │
│  └─────────────────────┘                                │
│              │                                          │
│              ▼                                          │
│  Generated HTML (single file, self-contained)           │
│              │                                          │
│              ▼                                          │
│  ┌─────────────────────┐                                │
│  │  iframe / webview   │  ← app runs here               │
│  │  (Chromium on RPi)  │                                │
│  └─────────────────────┘                                │
└─────────────────────────────────────────────────────────┘
```

1. The skill file (`SKILL.md` or a studio-specific `skill.md`) is loaded as the **system prompt**
2. The child's request is sent as the **user message**
3. The LLM responds with a **single complete HTML file**
4. Giggle OS renders that HTML in an iframe / webview
5. The child plays/uses it immediately

The AI output is always one self-contained HTML file — no dependencies, no CDN, no external resources. Everything is inline: CSS, JavaScript, audio (synthesized), and graphics (emoji + Canvas).

---

## The 6 Studios

### 🎮 Game Studio — Game Forge
**Skill file:** [skills/game-forge/skill.md](skills/game-forge/skill.md)

Turns a kid's game idea into a playable HTML5 game.

| Pattern | Description | Example prompt |
|---------|-------------|----------------|
| Platformer | Side-scrolling jumper | "a cat that jumps over puddles" |
| Catcher | Catch falling items | "catch the falling stars with a basket" |
| Maze | Navigate grid mazes | "a maze game with a mouse finding cheese" |
| Whack-a-Mole | Tap disappearing targets | "tap the frogs before they hide" |
| Runner | Auto-scroll, tap to jump | "a dinosaur running from meteors" |
| Memory | Flip-card matching | "a memory game with animals" |
| Pong | Paddle and ball | "pong but with a rocket ship" |
| Target | Tap random targets | "pop the balloons!" |

**Technical details:** Canvas API or DOM, `requestAnimationFrame` at 30fps, bounding-box collision, Web Audio sound effects, emoji graphics, touch drag + tap controls.

---

### 📖 Story Studio — Story Weaver
**Skill file:** [skills/story-weaver/skill.md](skills/story-weaver/skill.md)

Turns a story idea into an interactive illustrated narrative.

| Pattern | Description | Example prompt |
|---------|-------------|----------------|
| Branching | Choose-your-own-adventure | "a brave knight who fights a dragon" |
| Mad Libs | Fill-in-the-blank funny story | "a mad libs about a silly robot" |
| Comic Strip | Panel-based visual story | "a comic about a dog going to school" |
| Adventure | Room exploration + items | "explore a haunted house" |

**Technical details:** Scene objects with emoji illustrations, mood-based background colors, large readable text (24px+), big colorful choice buttons (56px min height), always happy endings.

---

### 🎵 Music Studio — Beat Lab
**Skill file:** [skills/beat-lab/skill.md](skills/beat-lab/skill.md)

Turns a music idea into a playable instrument or beat maker.

| Pattern | Description | Example prompt |
|---------|-------------|----------------|
| Piano | Touch-play melodic keys | "a rainbow piano" |
| Drum Machine | Pad-based percussion | "a drum machine" |
| Sequencer | Step-sequencer with loop | "let me make a beat" |
| Theremin | Drag for pitch + volume | "a weird space instrument" |
| Sound Board | Grid of fun sounds | "a DJ soundboard" |
| Music Box | Rotating note cylinder | "a music box I can program" |

**Technical details:** Web Audio API exclusively, synthesized sounds (no audio files), zero-latency `touchstart` trigger, ADSR envelope (attack 0.01s / decay 0.1s / sustain 0.3 / release 0.2), equal temperament tuning (A4=440Hz).

---

### 🎨 Art Studio — Canvas Magic
**Skill file:** [skills/canvas-magic/skill.md](skills/canvas-magic/skill.md)

Turns an art idea into a creative drawing tool.

| Pattern | Description | Example prompt |
|---------|-------------|----------------|
| Freehand | Finger painting | "a drawing app with lots of colors" |
| Stamp | Emoji stamp tool | "a stamp app with animals" |
| Pixel Art | Grid pixel editor | "a pixel art maker" |
| Kaleidoscope | Mirrored drawing | "a magic mirror drawing app" |
| Fireworks | Tap-to-explode particles | "fireworks I can make with my finger" |
| Color Mixer | Mix primary colors | "teach me how colors mix" |

**Technical details:** Canvas 2D API, `lineCap='round'` + `lineJoin='round'` for smooth strokes, 44px+ tap targets for color swatches, undo stack (up to 20 states), `toDataURL` for save.

---

### 🔬 Science Studio — Lab Explorer
**Skill file:** [skills/lab-explorer/skill.md](skills/lab-explorer/skill.md)

Turns a science topic into an interactive educational simulation.

| Pattern | Description | Example prompt |
|---------|-------------|----------------|
| Solar System | Orbiting planets with facts | "show me the solar system" |
| Ecosystem | Food chain / habitat | "how do animals eat each other?" |
| Weather | Particle weather sim | "make it rain and snow" |
| Body Explorer | Tap organs for facts | "show me inside the human body" |
| Chemistry | Safe reaction mixer | "what happens when I mix things?" |
| Dinosaurs | Era timeline | "show me all the dinosaurs" |
| Gravity | Drop objects on planets | "what's gravity on Mars?" |

**Technical details:** Animated Canvas simulations, tap-to-reveal info panels, age-scaled fact length (1 sentence → paragraph), real scientific data used, color-coded themes per domain.

---

### 🔧 Tinker Studio — Gadget Shop
**Skill file:** [skills/gadget-shop/skill.md](skills/gadget-shop/skill.md)

Turns a tool idea into a useful or entertaining mini-app.

| Pattern | Description | Example prompt |
|---------|-------------|----------------|
| Calculator | Colorful big-button calc | "a calculator" |
| Timer | Visual countdown + ring | "a 5-minute timer" |
| Flashcards | Create + study mode | "flashcards for my spelling test" |
| Fortune Teller | Magic 8-ball style | "a fortune teller" |
| Dice Roller | Animated configurable dice | "roll some dice" |
| Color Generator | Harmonious palette picker | "show me pretty colors" |
| Animation | Particle / bouncing toys | "bouncing balls I can add more of" |

**Technical details:** Oversized controls (48px+ tap targets), clear visual state indicators, audio feedback on every interaction, genuinely functional utilities.

---

## Using with Different LLMs

The skill files are plain markdown with a YAML frontmatter header. They work as system prompts in any LLM that supports a system role. Here's how to load them in common environments.

---

### Claude (Anthropic)

**Claude Code (this project's native environment):**

The `SKILL.md` file is already formatted as a Claude Code skill. Place it in your Claude skills directory or reference it in your project.

**Claude.ai (web):**

1. Open [claude.ai](https://claude.ai)
2. Start a new conversation
3. Click the **System prompt** or **Project instructions** field
4. Paste the full contents of `SKILL.md`
5. Send your first message: `"I want to make a piano"` or any kid's description

**Claude API:**

```python
import anthropic

with open("SKILL.md", "r") as f:
    skill = f.read()

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=8096,
    system=skill,
    messages=[
        {"role": "user", "content": "make me a rainbow piano!"}
    ]
)

html = message.content[0].text
# Write html to file or inject into iframe
```

**Claude Projects:**

1. Create a new Project in Claude.ai
2. Go to **Project Instructions**
3. Paste the contents of `SKILL.md`
4. Every conversation in this project is now Giggle-powered

---

### ChatGPT (OpenAI)

**ChatGPT Custom GPT:**

1. Go to [chat.openai.com](https://chat.openai.com) → **Explore GPTs** → **Create**
2. In the **Instructions** field, paste the contents of `SKILL.md`
3. Name it "Giggle Engine" or similar
4. Save and share — anyone can use it as a kid-app generator

**ChatGPT API:**

```python
from openai import OpenAI

with open("SKILL.md", "r") as f:
    skill = f.read()

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": skill},
        {"role": "user", "content": "make me a maze game with a mouse!"}
    ],
    max_tokens=8096
)

html = response.choices[0].message.content
```

**ChatGPT Web (manual):**

Start a new chat and paste this before your first message:

```
[Paste full SKILL.md contents here]

---

Now: make me a dinosaur jumping game!
```

---

### Gemini (Google)

**Gemini API:**

```python
import google.generativeai as genai

with open("SKILL.md", "r") as f:
    skill = f.read()

genai.configure(api_key="YOUR_API_KEY")

model = genai.GenerativeModel(
    model_name="gemini-2.0-flash",
    system_instruction=skill
)

response = model.generate_content("make me a finger painting app")
html = response.text
```

**Google AI Studio:**

1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Click **System instructions**
3. Paste `SKILL.md` contents
4. Type your kid's request in the prompt
5. Copy the generated HTML and open in browser

---

### Local Models (Ollama, LM Studio)

Works with any model that supports a system prompt. Larger models (13B+) produce better results. Recommended: `llama3.2`, `mistral-nemo`, `qwen2.5-coder`.

**Ollama (CLI):**

```bash
# Create a Modelfile
cat > Modelfile << 'EOF'
FROM llama3.2

SYSTEM """
$(cat SKILL.md)
"""
EOF

ollama create giggle-engine -f Modelfile
ollama run giggle-engine "make me a drum machine"
```

**Ollama (API):**

```python
import requests

with open("SKILL.md", "r") as f:
    skill = f.read()

response = requests.post("http://localhost:11434/api/chat", json={
    "model": "llama3.2",
    "messages": [
        {"role": "system", "content": skill},
        {"role": "user", "content": "make a solar system I can tap"}
    ],
    "stream": False
})

html = response.json()["message"]["content"]
```

**LM Studio:**

1. Open LM Studio → load any model (7B+ recommended)
2. Go to the **Chat** tab
3. Click the **System Prompt** field (top of chat)
4. Paste `SKILL.md` contents
5. Chat normally with kid prompts

**Tip for local models:** If output is truncated, split the skill into smaller studio-specific files (e.g., `skills/game-forge/skill.md`) and load only the one relevant to the current studio.

---

### Open WebUI

Open WebUI supports system prompts natively and can be self-hosted alongside Ollama.

**Method 1 — Model System Prompt:**

1. Go to **Workspace** → **Models**
2. Create a new model or duplicate an existing one
3. In **System Prompt**, paste `SKILL.md` contents
4. Save → use this model in any chat

**Method 2 — Knowledge Base:**

1. Go to **Workspace** → **Knowledge**
2. Create a collection called "Giggle Skills"
3. Upload all files in `skills/` and `shared/`
4. Attach this collection to your model
5. Now the model can reference individual studio guides

**Method 3 — Prompt Template:**

1. Go to **Workspace** → **Prompts**
2. Create a prompt named "Giggle App"
3. Content: `You are the Giggle OS Imagination Engine. {{skill_contents}} Build this app: {{input}}`
4. Use as a quick-start in any chat

---

## Project Structure

```
giggle-skills/
│
├── SKILL.md                    # Master skill — loads all 6 studios
│                               # Use this as your system prompt
│
├── skills/                     # Individual studio skills (load one at a time
│   │                           # for smaller context windows)
│   ├── game-forge/
│   │   └── skill.md            # Game Studio definition
│   ├── story-weaver/
│   │   └── skill.md            # Story Studio definition
│   ├── beat-lab/
│   │   └── skill.md            # Music Studio definition
│   ├── canvas-magic/
│   │   └── skill.md            # Art Studio definition
│   ├── lab-explorer/
│   │   └── skill.md            # Science Studio definition
│   └── gadget-shop/
│       └── skill.md            # Tinker Studio definition
│
├── templates/                  # HTML boilerplates — starting points for
│   │                           # each studio type
│   ├── base.html               # Universal boilerplate (audio, touch, viewport)
│   ├── game.html               # Canvas game: loop, score, start screen
│   ├── story.html              # Branching story: scene renderer, choices
│   ├── music.html              # Instrument: Web Audio, pads, ADSR
│   ├── art.html                # Drawing: palette, brushes, undo stack
│   ├── science.html            # Simulation: canvas, hit test, info panel
│   └── tinker.html             # Gadget: display, button grid, audio click
│
├── examples/                   # Fully working apps — open in browser to test
│   ├── game-forge/
│   │   └── dino-jumper.html    # Tap-to-jump runner, emoji graphics, scoring
│   ├── story-weaver/
│   │   └── brave-cat.html      # 6-scene branching story with multiple endings
│   ├── beat-lab/
│   │   └── simple-piano.html   # 8-key rainbow piano with Web Audio
│   ├── canvas-magic/
│   │   └── finger-paint.html   # Full painting app with palette + undo
│   ├── lab-explorer/
│   │   └── solar-system.html   # Animated orbiting planets with fact panels
│   └── gadget-shop/
│       └── dice-roller.html    # Animated dice: d6/d8/d10/d20, 1-4 dice
│
└── shared/                     # Reference docs for AI and developers
    ├── audio-helpers.md        # Web Audio: beep, note, noise, frequencies
    ├── touch-helpers.md        # Touch events, drag, hit testing, feedback
    ├── canvas-helpers.md       # Canvas: lines, emoji, particles, undo, FPS
    └── age-guidelines.md       # Content rules for ages 5-6, 7-8, 9-10
```

---

## Output Format

Every app generated by the Imagination Engine is a **single self-contained HTML file**. No exceptions.

```
✅ Allowed                      ❌ Not allowed
─────────────────────────────   ──────────────────────────────
Inline <style> tags             External CSS files
Inline <script> tags            <script src="...">
Emoji for graphics              <img src="..."> with real images
Web Audio API (synthesized)     <audio src="..."> files
Canvas 2D API                   WebGL / Three.js
CSS animations                  GSAP / other animation libs
requestAnimationFrame           setInterval for animation
Relative units (%, vw, vh)      Fixed pixel sizes
touch events                    mouse-only events
```

### Why single-file?

- **Offline first** — RPi may not have internet. Everything works offline.
- **No CORS issues** — files run in iframes without a server
- **Easy to share** — one file, drag and drop
- **Easy to save** — kids can keep their creation as one file
- **Easy to debug** — open in any browser, everything is visible

---

## Design Principles

### 1. Touch-First
Every interactive element is designed for fingers, not cursors.
- Minimum tap target: **44px × 44px** (48px preferred)
- Use `touchstart` for immediate response (not `touchend`)
- Always `preventDefault()` to stop browser scroll/zoom
- Visual feedback on every touch: scale, brightness, ripple

### 2. Self-Contained
Zero runtime dependencies. No CDN. No `npm install`. No server.
- All CSS inline in `<style>` tags
- All JS inline in `<script>` tags
- Emoji for all graphics (no image files)
- Web Audio API for all sound (no audio files)

### 3. Emoji Graphics
Using emoji as the visual layer means:
- No image assets to download or embed
- Universally recognizable for kids
- Works on all platforms/browsers
- Infinitely extensible (thousands of emoji available)
- No copyright or licensing concerns

### 4. Synthesized Audio
All sound is generated in real-time using the Web Audio API:
- No audio file loading delay
- No bandwidth usage
- Consistent latency
- Fully customizable (pitch, duration, envelope, waveform)

### 5. Age-Adaptive
The same skill generates different experiences for different ages:
- **5–6 years:** No fail states, huge targets, 1-word instructions, instant rewards
- **7–8 years:** Gentle difficulty, simple text, 2–3 choices, optional challenge
- **9–10 years:** Real challenge, paragraphs, scores, strategic depth

### 6. Always Encouraging
- No "Game Over" screens for young kids — instead "Try Again!" with a smile
- Every mistake gets gentle positive feedback
- Every story ends happily
- Score popups, particle bursts, and beeps on every success

### 7. RPi Performance
- Target 30fps (not 60fps) — achievable on RPi 5 in Chromium
- Use `requestAnimationFrame` (not `setInterval`)
- Bounding-box collision (not pixel-perfect)
- Limit particle count (max 50–100 active)
- No heavy CSS filters or blur on animated elements

---

## Age Adaptation

The LLM automatically scales all content based on the child's declared age or grade. Here's what changes:

| Feature | Age 5–6 | Age 7–8 | Age 9–10 |
|---------|---------|---------|---------|
| Tap target size | 56px+ | 48px+ | 44px+ |
| Text size | 28px+ | 24px+ | 20px+ |
| Sentence length | 5–8 words | 1–2 sentences | Full paragraphs |
| Fail states | None | Lives / retry | Full game over |
| Difficulty | Static easy | Gentle curve | Real challenge |
| Story choices | 1 per page | 2 per page | 2–3 per page |
| Science facts | 1 sentence | 2–3 sentences | Paragraph |
| Math | Count 1–10 | +, −, × | All ops, fractions |

If no age is provided, the skill defaults to age 7–8 as a safe middle ground.

---

## Tech Stack

Everything runs in-browser with zero dependencies:

| Technology | Used for | Notes |
|-----------|---------|-------|
| HTML5 Canvas | Games, art, science simulations | 2D context only |
| Web Audio API | All sound effects and music | Synthesized, no files |
| CSS Animations | UI transitions, visual feedback | Keyframes + transforms |
| CSS Grid / Flexbox | Layouts for all app types | Responsive to screen size |
| Touch Events API | All user input | `touchstart`, `touchmove`, `touchend` |
| `requestAnimationFrame` | Game loops, animations | Targets 30fps |
| Emoji | All graphics | Rendered as text, no images |
| `localStorage` | Saving scores, flashcards | Optional, when needed |

---

## Target Hardware

| Spec | Value |
|------|-------|
| Device | Raspberry Pi 5 |
| Display | 7" official touchscreen |
| Resolution | 800 × 480 px |
| Browser | Chromium (kiosk mode) |
| OS | Raspberry Pi OS (64-bit) |
| Network | Optional (all apps work offline) |
| Input | Touch only (no keyboard/mouse assumed) |

**Kiosk mode command:**
```bash
chromium-browser --kiosk --noerrdialogs --disable-infobars \
  --incognito --app=http://localhost:3000
```

**Performance targets:**
- App generation: < 10 seconds (LLM response time)
- App launch: < 500ms (HTML parse + render)
- Animation: 30fps sustained
- Touch latency: < 16ms (touchstart response)

---

## Extending the Skill

### Adding a New Studio

1. Create a new directory under `skills/`:
   ```
   skills/your-studio-name/skill.md
   ```

2. Follow the skill file format:
   ```markdown
   ---
   name: your-studio-name
   description: One-line description of what this studio builds
   ---

   # Studio Name

   You are the [Studio Name] engine inside Giggle OS. When a child asks for X, you build Y.

   ## What kids say
   - "example prompt 1"
   - "example prompt 2"

   ## Patterns
   [List the patterns this studio knows]

   ## Rules
   [List the constraints and requirements]
   ```

3. Add the studio to the main `SKILL.md` under `## The Imagination Suites`

4. Create a corresponding template in `templates/`

5. Build a working example in `examples/your-studio-name/`

### Adding New Patterns to Existing Studios

Edit the relevant `skills/<studio>/skill.md` and add the new pattern under `## Patterns`. Give it:
- A clear name
- A description of what it builds
- Example kid prompts that trigger it
- Any unique rules or constraints

### Tuning Age Ranges

Edit [shared/age-guidelines.md](shared/age-guidelines.md) and update the rules. The main `SKILL.md` references these conceptually — if you need stricter enforcement, add explicit rules to each studio skill file.

---

## Examples

All examples are ready to open in any browser. No server needed.

| File | Studio | What it demonstrates |
|------|--------|----------------------|
| [examples/game-forge/dino-jumper.html](examples/game-forge/dino-jumper.html) | Game Forge | Canvas game loop, physics, collision, score, start screen |
| [examples/story-weaver/brave-cat.html](examples/story-weaver/brave-cat.html) | Story Weaver | Scene graph, branching narrative, mood backgrounds, 6 endings |
| [examples/beat-lab/simple-piano.html](examples/beat-lab/simple-piano.html) | Beat Lab | Web Audio oscillator, ADSR envelope, touch keys, rainbow colors |
| [examples/canvas-magic/finger-paint.html](examples/canvas-magic/finger-paint.html) | Canvas Magic | Touch drawing, color palette, brush sizes, undo stack |
| [examples/lab-explorer/solar-system.html](examples/lab-explorer/solar-system.html) | Lab Explorer | Animated Canvas, orbit math, tap-to-reveal fact panels |
| [examples/gadget-shop/dice-roller.html](examples/gadget-shop/dice-roller.html) | Gadget Shop | Animated dice roll, multiple die types, synthesized sound |

To test locally:
```bash
# Any of these work
open examples/game-forge/dino-jumper.html        # macOS
xdg-open examples/game-forge/dino-jumper.html    # Linux
start examples/game-forge/dino-jumper.html       # Windows

# Or serve all examples
python3 -m http.server 8080
# then open http://localhost:8080/examples/
```

---

## Quick Start

The fastest way to try the Imagination Engine:

1. Copy the contents of `SKILL.md`
2. Open any LLM chat (Claude, ChatGPT, Gemini, etc.)
3. Paste the skill content as the system prompt
4. Type: **"make me a rainbow piano"**
5. Copy the generated HTML into a `.html` file
6. Open in browser
7. Play!

---

*Built for curious kids, powered by AI, runs on a Raspberry Pi.*
