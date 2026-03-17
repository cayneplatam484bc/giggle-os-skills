# Giggle Skills

The AI skill set powering **Giggle OS** — a creative tablet OS for kids ages 5-10, running on Raspberry Pi 5 with a 7" touchscreen.

A child describes what they want to build. The Imagination Engine turns it into a working, interactive, self-contained HTML app.

## Studios

| Studio | Skill | Description |
|--------|-------|-------------|
| 🎮 Game Studio | `game-forge` | Interactive HTML5 games — platformers, catchers, mazes, runners |
| 📖 Story Studio | `story-weaver` | Branching stories, mad libs, comic strips, adventures |
| 🎵 Music Studio | `beat-lab` | Piano, drum machine, sequencer, theremin, music box |
| 🎨 Art Studio | `canvas-magic` | Drawing, stamps, pixel art, kaleidoscope, fireworks |
| 🔬 Science Studio | `lab-explorer` | Solar system, ecosystems, weather, body explorer |
| 🔧 Tinker Studio | `gadget-shop` | Calculator, timer, dice roller, flashcards, animations |

## Project Structure

```
giggle-skills/
├── SKILL.md                 # Master skill definition
├── skills/                  # Individual studio skills
│   ├── game-forge/
│   ├── story-weaver/
│   ├── beat-lab/
│   ├── canvas-magic/
│   ├── lab-explorer/
│   └── gadget-shop/
├── templates/               # HTML starter templates
│   ├── base.html            # Universal boilerplate
│   ├── game.html
│   ├── story.html
│   ├── music.html
│   ├── art.html
│   ├── science.html
│   └── tinker.html
├── examples/                # Working example apps
│   ├── game-forge/          # Dino Jumper
│   ├── story-weaver/        # The Brave Cat
│   ├── beat-lab/            # Rainbow Piano
│   ├── canvas-magic/        # Finger Paint
│   ├── lab-explorer/        # Solar System
│   └── gadget-shop/         # Dice Roller
└── shared/                  # Common patterns & guides
    ├── audio-helpers.md     # Web Audio API patterns
    ├── touch-helpers.md     # Touch event handling
    ├── canvas-helpers.md    # Canvas drawing patterns
    └── age-guidelines.md    # Age-appropriate content
```

## Design Principles

- **Self-contained** — Every app is a single HTML file, no external dependencies
- **Touch-first** — Built for small fingers on a 7" touchscreen
- **Emoji graphics** — No image assets, all visuals from emoji + CSS + Canvas
- **Synthesized audio** — Web Audio API for all sounds, no audio files
- **Age-adaptive** — Content adjusts for 5-6, 7-8, and 9-10 year olds
- **Always encouraging** — No fail states for young kids, positive feedback always

## Target Hardware

- Raspberry Pi 5
- 7" touchscreen (800x480)
- Chromium browser in kiosk mode
- Target: 30fps for smooth animation
