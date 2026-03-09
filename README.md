# DISCO MIND
![screenshot](https://github.com/user-attachments/assets/7d1b94c7-51eb-4a13-93e9-11c9df44002e)

> *"You type a thought. 24 personalities living in your head react to it simultaneously."*

A locally-running AI application where the 24 psychological skill voices from **Disco Elysium** respond to your thoughts in real time. Built on Ollama, Piper TTS (migrating to XTTS v2 with cloned voices), and a single HTML file. No cloud. No API costs. No data leaving your machine.

![Disco Mind UI](screenshot.png)

---

## What It Is

In Disco Elysium, your character's psychology is represented as 24 distinct internal voices - Logic, Electrochemistry, Inland Empire, Half Light, Volition, and 20 others - each with its own personality, agenda, and way of seeing the world. They argue. They clash. They sometimes tell you to do terrible things.

Disco Mind takes that system and turns it into a reflection tool. You type anything - a thought, a feeling, a situation - and the skills respond. Not as a unified AI assistant. As themselves.

---

## How It Works

```
Your input
    │
    ▼
Conductor (Ollama)
    │  reads your input, picks 4-5 relevant skills
    ▼
Skills fire in parallel (Ollama)
    │  each with its own system prompt and persona
    ▼
Responses rendered in the UI
    │  with portraits, skill names, category colors
    ▼
TTS (Piper or XTTS)
    │  each skill spoken aloud in Lenval Johnson's voice
    ▼
You
```

**The conductor** makes one Ollama call to decide which skills are relevant to your input. It uses an explicit trigger guide so visceral skills (Electrochemistry, Half Light) fire on the right inputs.

**The skills** all call Ollama in parallel, each with a deeply individual system prompt. They don't know what the other skills are saying. The clash is structural.

**TTS** is handled by a local Python server. Currently Piper with per-skill voice parameter tuning. Migrating to XTTS v2 with reference clips extracted directly from the game's audio files - 3-5 clips per skill, each shaped by Lenval's actual in-game delivery of that character.

---

## Stack

| Component | Technology |
|-----------|-----------|
| UI | Single HTML file, vanilla JS |
| LLM | Ollama (local) - recommended: `qwen2.5:14b` |
| TTS (current) | Piper TTS via Python server |
| TTS (target) | XTTS v2 (coqui-tts) with cloned voices |
| Voice reference | Extracted from Disco Elysium game audio |
| Portraits | Embedded base64 in HTML |

---

## Requirements

- Windows 10/11 (developed on Windows)
- [Ollama](https://ollama.com) installed and running
- Python 3.11 (specifically - not 3.12+, not 3.14)
- GPU with 8GB+ VRAM recommended (16GB for `qwen2.5:14b`)
- Disco Elysium (for voice extraction - Epic or Steam)

---

## Setup

### 1. Ollama

Kill the system tray icon if it's running, then start it with CORS enabled:

```powershell
$env:OLLAMA_ORIGINS="*"; ollama serve
```

Pull the recommended model:

```powershell
ollama pull qwen2.5:14b
```

### 2. Python dependencies

```powershell
py -3.11 -m pip install coqui-tts
py -3.11 -m pip install torch torchaudio --index-url https://download.pytorch.org/whl/cu124
```

### 3. Voice extraction (XTTS path)

Extract reference audio from the game using UABE (Unity Asset Bundle Extractor):

- Game audio lives at:
  `C:\Program Files\Epic Games\DiscoElysium\Disco Elysium_Data\StreamingAssets\aa\StandaloneWindows64`
- Files are organized by skill and speaker
- Export 3-5 clean WAV clips per skill (8-15 seconds each, no background music)
- Place into `disco-voices/<skill_name>/` folder structure

### 4. TTS server

**Piper (current):**
```powershell
cd ~/piper-voices
python piper_server.py
```

**XTTS (in progress):**
```powershell
py -3.11 xtts_server.py
```

### 5. Open the app

Open `disco_mind.html` in Chrome or Firefox. Set the model to `qwen2.5:14b` in the config bar.

---

## The 24 Skills

### INTELLECT
| Skill | Character |
|-------|-----------|
| Logic | Surgical prosecutor. Stress-tests everything. Faintly contemptuous of irrationality. |
| Encyclopedia | Overeducated archive. Delivers full lectures when one sentence was needed. |
| Rhetoric | Internal debate champion. Every conversation is ideological combat. |
| Drama | Theatrical diagnostician. Detects performance in everyone, including you. |
| Conceptualization | Tortured art critic. Sees meaning in everything, sometimes correctly. |
| Visual Calculus | Forensic replay engine. Reconstructs events frame by frame. |

### PSYCHE
| Skill | Character |
|-------|-----------|
| Volition | The last sane adult. Holds on. Asks you to hold on too. |
| Inland Empire | Mystical dream engine. Objects speak. Places remember. |
| Empathy | Reads people like storms. Sharp and exhausted by it. |
| Authority | Primal urge to own the room. Weakness disgusts it. |
| Esprit de Corps | Quiet solidarity. Believes in the bond built through real pressure. |
| Suggestion | Smooth manipulator. Prefers influence through precision over force. |

### PHYSIQUE
| Skill | Character |
|-------|-----------|
| Endurance | Stubborn biological engine. Get up. You've survived worse. |
| Pain Threshold | Genuinely desensitized. Almost comfortable with damage. |
| Physical Instrument | Raw muscle. Stop analyzing. Do something. |
| Electrochemistry | Feral pleasure demon. Do the drugs. All of them. |
| Shivers | The city itself. Four hundred years pressing on this moment. |
| Half Light | Pure animal aggression. ON at all times. The whole world is a wooden house. |

### MOTORICS
| Skill | Character |
|-------|-----------|
| Hand/Eye Coordination | Precision. Clean execution. Nothing more. |
| Perception | Notices everything. Cannot stop noticing. |
| Reaction Speed | Lives in the gap between stimulus and conscious thought. |
| Savoir Faire | Coolness, style, and contempt for clumsiness. |
| Interfacing | Understands machines. Applies it to everything else. |
| Composure | The ice-cold mask. Tired in a very specific way. |

---

## Known Limitations

- LLMs with strong RLHF training (Qwen, Llama) will sanitize responses to certain inputs regardless of persona instructions. This is a model-level behavior, not a prompt failure. Uncensored model variants (dolphin series) reduce this but don't fully eliminate it.
- XTTS generation takes 5-10 seconds per clip. This is the cost of quality.
- The 14B model occasionally drifts out of character on long context. This improves significantly at 32B but requires 24GB+ VRAM for acceptable speed.

---

## Roadmap

- [ ] XTTS v2 integration with per-skill cloned voice references
- [ ] Voice extraction pipeline documented fully
- [ ] Response streaming (partial text appears as it generates)
- [ ] Personal context layer - skills that know *your* specific patterns
- [ ] Export session as text/document
- [ ] Packaging for easier distribution (Docker or installer)

---

## Credits

- **Disco Elysium** - ZA/UM. All skill concepts, names, and voice identity belong to them.
- **Lenval Johnson** - narrator and voice of all 24 skills in the original game.
- **Ollama** - local LLM inference.
- **Coqui TTS / XTTS v2** - voice synthesis.
- **Piper TTS** - current TTS backend.

---

*This is a fan project. Not affiliated with ZA/UM.*
