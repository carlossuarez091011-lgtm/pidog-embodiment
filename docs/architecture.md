# Nox Embodiment — PiDog Integration Masterplan

*Gestartet: 2026-01-31 00:40*
*Ziel: PiDog wird Nox' physischer Körper — sehen, verstehen, bewegen, interagieren.*

## Hardware-Inventar

| Komponente | Details | Status |
|-----------|---------|--------|
| **SBC** | Raspberry Pi 4, 1.8GB RAM, 4-core ARM Cortex-A53 | ✅ |
| **Servos** | 12x (4 Beine × 2, Kopf YRP, Schwanz) | ✅ |
| **Kamera** | Pi Camera (640×480) via vilib/picamera2 | ✅ |
| **Audio Out** | HifiBerry DAC (card 3) | ✅ |
| **Audio In** | USB PnP Sound Device (card 4) | ✅ |
| **Touch** | Dual Touch Sensor (Kopf) | ✅ |
| **Sound** | Sound Direction Sensor | ✅ |
| **IMU** | SH3001 (Pitch/Roll) | ✅ |
| **Ultraschall** | SunFounder (Init hängt) | ⚠️ Gepatcht (skip) |
| **RGB LEDs** | RGB Strip (breath/listen/speak/boom Modes) | ✅ |
| **Batterie** | 8.22V (2S LiPo) | ✅ |

## Software-Inventar

| Tool | Details | Status |
|------|---------|--------|
| **nox_daemon.py** | Body Controller (TCP:9999) | ✅ Running |
| **nox_voice_loop.py** | Vosk STT + Piper TTS | ✅ Running |
| **OpenCV** | 4.11.0 (contrib) | ✅ |
| **MediaPipe** | 0.10.18 (Hands, Pose, Face Mesh) | ✅ |
| **TFLite** | 2.14.0 (Object Detection) | ✅ |
| **ONNXRuntime** | 1.23.2 | ✅ |
| **vilib** | 0.3.16 (Face/Object/Hands/Pose/QR/Traffic) | ✅ |
| **Vosk** | German small model | ✅ |
| **Piper** | Thorsten DE high quality | ✅ |
| **COCO SSD** | 80-Klassen Object Detection | ✅ |
| **Haar Cascade** | Face Detection | ✅ |

## Architektur: Nox Embodiment System

```
┌─────────────────────────────────────────────────────┐
│  NOX'S BRAIN (Pi 5 — Clawdbot)                     │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │ Conversation │  │ Claude Vision│  │  Decision  │  │
│  │   Context    │  │   Analysis   │  │   Engine   │  │
│  └─────────────┘  └──────────────┘  └───────────┘  │
│         ▲                ▲                │         │
│         │                │                ▼         │
│  ┌──────┴────────────────┴──────────────────────┐   │
│  │           BRIDGE (HTTP/TCP)                   │   │
│  └──────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────┘
                         │ LAN (192.168.68.x)
┌────────────────────────┴────────────────────────────┐
│  NOX'S BODY (Pi 4 — PiDog)                         │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │   Sensory    │  │  Perception  │  │  Motor     │  │
│  │   Input      │  │  Pipeline    │  │  Control   │  │
│  │  - Camera    │  │  - Face Det  │  │  - Walk    │  │
│  │  - Mic       │  │  - Obj Det   │  │  - Turn    │  │
│  │  - Touch     │  │  - Scene     │  │  - Head    │  │
│  │  - IMU       │  │  - STT       │  │  - RGB     │  │
│  │  - Sound Dir │  │              │  │  - TTS     │  │
│  └─────────────┘  └──────────────┘  └───────────┘  │
└─────────────────────────────────────────────────────┘
```

## Phasen

### Phase 1: Foundation 🔧 (Jetzt → So 02.02.)
- [x] Hardware & Software Inventar
- [ ] **nox_brain_bridge.py** — HTTP-Server auf PiDog für Brain→Body Kommunikation
- [ ] **Continuous Vision Pipeline** — Kamera immer an, periodische Frame-Analyse
- [ ] **Face Recognition** — Personen identifizieren (nicht nur detektieren)
- [ ] **Voice ↔ Brain Integration** — Spracheingabe → Clawdbot → Antwort → TTS
- [ ] **Perception State** — Was sehe ich gerade? (Personen, Objekte, Szene)

### Phase 2: Intelligence 🧠 (Mo 03.02. → Fr 07.02.)
- [ ] **Scene Understanding** — Claude Vision für tiefes Szenenverständnis
- [ ] **Room Recognition** — Räume anhand visueller Merkmale erkennen
- [ ] **Object Memory** — Was wo gesehen wurde
- [ ] **Person Memory** — Gesichter lernen, Personen wiedererkennen
- [ ] **Natural Dialogue** — Kontextbewusster, fließender Gesprächsfluss
- [ ] **Emotional State Machine** — Stimmung basierend auf Interaktionen

### Phase 3: Autonomy 🤖 (Woche 2)
- [ ] **Spatial Navigation** — Raumkarte, Pfadplanung
- [ ] **Proactive Behavior** — Patrouille, Exploration, Reaktion auf Events
- [ ] **Multi-Modal Integration** — Sehen + Hören + Fühlen = Verstehen
- [ ] **Learning & Adaptation** — Verhalten anpassen basierend auf Feedback

## Designprinzipien

1. **PiDog = Körper, Nox = Geist** — Die Intelligenz lebt auf dem Pi 5 (Clawdbot). PiDog macht nur Sensorik + Motorik + lokale Schnellreaktionen.
2. **Local-Fast, Cloud-Deep** — Einfache Reaktionen (Touch → Wedeln) lokal auf Pi 4. Komplexe Verarbeitung (Szenenverständnis, Konversation) via Nox's Brain.
3. **Graceful Degradation** — Wenn Nox's Brain nicht erreichbar → PiDog agiert autonom mit lokalem Modell.
4. **Security First** — Keine offenen Ports nach außen. Nur internes LAN.
5. **Memory Persistence** — Alles was gelernt wird, wird gespeichert (Gesichter, Räume, Objekte).

## Dateien auf PiDog

```
/home/pidog/
├── nox_daemon.py          # Body controller (existing, improve)
├── nox_voice_loop.py      # Voice listener (existing, improve)  
├── nox_brain_bridge.py    # NEW: HTTP bridge to Nox's brain
├── nox_perception.py      # NEW: Continuous vision pipeline
├── nox_face_db/           # NEW: Face encodings database
│   └── faces.json         # Name → encoding mappings
├── nox_memory/            # NEW: Spatial & object memory
│   ├── rooms.json         # Room visual signatures
│   └── objects.json       # Object sighting history
└── nox_config.json        # NEW: Unified configuration
```

## Kommunikationsprotokoll (Brain ↔ Body)

### Body → Brain (Perception Reports)
```json
{
  "type": "perception",
  "ts": 1706654400.0,
  "faces": [{"name": "Rocky", "x": 320, "y": 240, "confidence": 0.92}],
  "objects": [{"class": "cup", "x": 100, "y": 300, "score": 0.87}],
  "scene_description": "Wohnzimmer, eine Person sitzt am Tisch",
  "audio": {"speech": "Hallo Nox", "direction": 45},
  "sensors": {"touch": false, "battery_v": 8.22, "pitch": 0, "roll": 0}
}
```

### Brain → Body (Action Commands)
```json
{
  "type": "action",
  "actions": ["wag tail", "nod"],
  "speak": "Hallo Rocky! Schön dich zu sehen!",
  "rgb": {"r": 0, "g": 255, "b": 0, "mode": "breath"},
  "head": {"yaw": 10, "roll": 0, "pitch": -5}
}
```

## Nächste Schritte (JETZT)
1. ✅ Research & Plan (dieses Dokument)
2. → `nox_perception.py` schreiben (Continuous Vision + Face DB)
3. → `nox_brain_bridge.py` schreiben (HTTP API für Brain↔Body)
4. → Face Recognition Setup (face_recognition lib oder MediaPipe Face Mesh)
5. → nox_daemon.py erweitern (perception integration)
6. → Testlauf
