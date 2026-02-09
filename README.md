# Infinite Summer Session (Beskonechnyi Letnik)

A story-driven Ren'Py visual novel where the player survives a high-pressure exam week through choices, relationships, and subject-specific minigames.

## Table of Contents
- [Project Overview](#project-overview)
- [Core Features](#core-features)
- [Story Flow](#story-flow)
- [Exam Systems](#exam-systems)
- [Scoring, GPA, and Endings](#scoring-gpa-and-endings)
- [Audio and Visual Design](#audio-and-visual-design)
- [Project Structure](#project-structure)
- [Controls](#controls)
- [How to Run](#how-to-run)
- [Build and Distribution Notes](#build-and-distribution-notes)
- [Generated Files and Git Hygiene](#generated-files-and-git-hygiene)
- [Developer Notes](#developer-notes)

## Project Overview
This project is a complete Ren'Py game with:
- A 5-day narrative arc (`day1` to `day5`).
- 6 exams represented by unique gameplay systems.
- Relationship flags that influence exam difficulty and outcomes.
- A final transcript screen with GPA calculation and branching endings.

Entry point:
- `game/script.rpy` -> `label start` -> `jump day1`

Main game title is configured in:
- `game/options.rpy` (`config.name`)

## Core Features
- Branching dialogue and social choice consequences.
- Character relationship tracking (`ayan`, `diana`, `timur`, `alina`).
- Subject-specific exam gameplay:
  - Programming: robot pathing puzzle
  - Psychology: drag-and-drop interpretation + therapy choice
  - Cryptography: lockpicking
  - Math Analysis: timed 2048 variant
  - English: timed response challenge
  - Networks: pipe-routing network puzzle
- Dynamic exam soundtrack switching by subject.
- Automatic switch back to location music after exam completion.
- Final GPA + multiple ending routes.

## Story Flow
Primary story files:
- `game/story/day1.rpy`
- `game/story/day2.rpy`
- `game/story/day3.rpy`
- `game/story/day4.rpy`
- `game/story/day5.rpy`

High-level progression:
1. Day 1
   - Intro, player naming, first social choices.
   - Exams: Programming, Psychology.
2. Day 2
   - Preparation day with branch selection (library / OS / dorm).
   - Sets helper flags for later exams.
3. Day 3
   - Exams: Cryptography, Math Analysis.
4. Day 4
   - Group interaction day and social alignment choices.
5. Day 5
   - Exams: English, Networks.
   - Results screen, GPA calculation, ending branch.

## Exam Systems
Central exam dispatcher:
- `game/mechanics/exams_logic.rpy`
- `run_exam(subject, resume_music=None)`

It does:
- Plays subject-specific exam music.
- Runs the corresponding minigame/label.
- Stops exam music (`fadeout=0.5`).
- Restores location music when `resume_music` is passed.
- Clamps and stores score in `store.exam_grades[subject]`.

### 1) Programming Exam
File:
- `game/mechanics/minigame_programming.rpy`

Gameplay:
- Robot command programming on grid maps (`S` start, `G` goal, `#` obstacle).
- Commands: `Forward`, `Turn Left`, `Turn Right`.
- Three levels with increasing complexity.
- Optional prefill hint from `helped_ayan`.

Scoring:
- Level 1 pass: +30
- Level 2 pass: +35
- Level 3 pass: +35
- Total max: 100

### 2) Psychology Exam
File:
- `game/mechanics/minigame_psychology.rpy`

Gameplay:
- Drag emotion labels to target aspects.
- Multi-scenario interpretation rounds.
- Final therapy-choice round.
- Optional prefilled hint path if `trusted_diana` is set.

Scoring:
- Scenario-based scoring function `_score_psycho_session`.
- Total max: 100 (30 + 30 + 20 + 20 structure).

### 3) Cryptography Exam
Files:
- `game/mechanics/exams_logic.rpy` (exam phase logic)
- `game/mechanics/minigame_lockpicking.rpy` (lockpicking displayable)

Gameplay:
- Open 5 locks with limited lockpicks.
- Difficulty increases each lock.
- Extra lockpick granted if `bonded_timur` is set.

Scoring:
- 20 points per successful lock.
- Total max: 100.

### 4) Math Analysis Exam
File:
- `game/mechanics/minigame_math.rpy`

Gameplay:
- Timed 2048 variant (`90s`) with target tile `256`.
- Keyboard and on-screen directional controls.
- Optional helper setup if `studied_matan` is set (better initial conditions).

Scoring:
- Based on highest tile reached via mapping up to 256.
- 256 => 100 points.

### 5) English Exam
File:
- `game/mechanics/minigame_english.rpy`

Gameplay:
- Timed reaction/answer rounds.
- Countdown bar and timeouts.
- Optional bonus time if `practiced_alina` is set.

Scoring:
- 3 rounds with full and partial credit.
- Total max: 100.

### 6) Networks Exam
File:
- `game/mechanics/minigame_networks.rpy`

Gameplay:
- Rotate pipe tiles to establish valid START -> END network path.
- Three levels, each with timer and rotation penalty effects.
- Network validation requires true bidirectional tile connection logic.

Scoring:
- `score_for_level(level_idx, rotations, elapsed)` with penalties.
- Clamped final total to 0..100.

### Generic Exam UI
File:
- `game/mechanics/screens_exams.rpy`

Reusable screens include:
- click-error
- input
- ordering
- multiselect
- choice
- adjuster

## Scoring, GPA, and Endings
State and grading logic:
- `game/variables.rpy`

Important data:
- `exam_grades` dict with all 6 subjects.
- Relationship counters.
- Helper flags used by minigames.

GPA functions:
- `get_letter_grade(score)`
- `get_gpa_point(score)`
- `calculate_gpa()`

Defensive normalization:
- `normalize_score(value)` handles non-numeric score shapes before GPA conversion.

Final result and endings:
- `game/story/day5.rpy`
- Endings selected by GPA + total relationship points.

## Audio and Visual Design
### Character and Background Images
Defined in:
- `game/characters.rpy`

Background aliases include:
- `bg room`, `bg hall`, `bg hallway`, `bg cafeteria`, `bg classroom`, `bg os`, `bg cafe`, `bg library`

### Music System
Story/location tracks:
- `game/story/day1.rpy` ... `game/story/day5.rpy`

Exam tracks:
- `game/mechanics/exams_logic.rpy`

Soundtrack files:
- `game/audio/Soundtracks/Room.mp3`
- `game/audio/Soundtracks/hall.mp3`
- `game/audio/Soundtracks/Cafeteria.mp3`
- `game/audio/Soundtracks/Ending.mp3`
- `game/audio/Soundtracks/Programming.mp3`
- `game/audio/Soundtracks/Psychology.mp3`
- `game/audio/Soundtracks/Crypto.mp3`
- `game/audio/Soundtracks/Math.mp3`
- `game/audio/Soundtracks/English.mp3`
- `game/audio/Soundtracks/Network.mp3`

Audio defaults:
- `game/options.rpy`: `preferences.music_volume = 0.5`

### Main Menu Customization
Files:
- `game/gui.rpy`
- `game/screens.rpy`

Implemented:
- Custom background: `images/Main.png`
- Dark translucent overlay frame
- Adjusted title placement
- Styled navigation button text and outlines

## Project Structure
```text
FinalProject/
  game/
    audio/
      Soundtracks/
    images/
    mechanics/
      exams_logic.rpy
      screens_exams.rpy
      minigame_programming.rpy
      minigame_psychology.rpy
      minigame_lockpicking.rpy
      minigame_math.rpy
      minigame_english.rpy
      minigame_networks.rpy
    story/
      day1.rpy
      day2.rpy
      day3.rpy
      day4.rpy
      day5.rpy
    script.rpy
    variables.rpy
    characters.rpy
    gui.rpy
    screens.rpy
    screens_extra.rpy
    options.rpy
```

## Controls
General VN:
- Mouse clicks / standard Ren'Py UI navigation.
- Save/Load/History/Skip from standard menu systems.

Minigame-specific:
- Programming: click command buttons, run/reset style flow.
- Psychology: drag and drop emotion tags.
- Lockpicking: mouse interactions with lock mechanism.
- Math 2048: arrow keys (`Left/Right/Up/Down`) and on-screen arrow buttons.
- English: timed option selection.
- Networks: click tiles to rotate, then validate/send ping.

## How to Run
Prerequisites:
- Ren'Py 8.x (project has been run on 8.5.x).

Run steps:
1. Open Ren'Py Launcher.
2. Add/open the `FinalProject` directory.
3. Launch the project.

## Build and Distribution Notes
Project metadata:
- `project.json`

Configured package targets:
- `win`
- `pc`

Build helper config:
- `game/options.rpy` build classification rules and save directory setup.

## Generated Files and Git Hygiene
Current `.gitignore` already excludes:
- `game/saves/`
- `game/cache/`
- compiled files (`*.rpyc`, `*.rpymc`)
- logs and trace files (`log.txt`, `errors.txt`, `traceback.txt`, etc.)

Recommended repository policy:
- Commit source `.rpy`, assets, and config files.
- Do not commit saves/cache/runtime logs.

## Developer Notes
### Where to add a new exam
1. Add subject key in `game/variables.rpy` -> `exam_grades`.
2. Add branch in `run_exam` inside `game/mechanics/exams_logic.rpy`.
3. Implement minigame and/or label in `game/mechanics/`.
4. Display score row in final results UI (`game/story/day5.rpy`).

### Relationship-based helper hooks currently used
- `helped_ayan` -> programming assist
- `trusted_diana` -> psychology assist
- `bonded_timur` -> extra lockpick in cryptography
- `practiced_alina` -> extra time in English
- `studied_matan` -> stronger starting state in math
