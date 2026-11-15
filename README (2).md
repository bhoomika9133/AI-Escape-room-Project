# The Blackwood Lighthouse: The Keeper's Last Shift
### An AI-Powered Horror Escape Room. Student: **Bhoomika** (Master's)

An original supernatural psychological-horror escape room built with **Python**, **Streamlit** and **Google Gemini**. It comes in two versions:

| | Version 1: Interactive AI Escape Room | Version 2: Reliable AI Escape Room |
|---|---|---|
| Who decides outcomes | Gemini (LLM) | Python rules engine |
| Game state | Chat transcript only | Structured `GameState` (JSON) |
| Puzzle solutions | Inside the LLM prompt | In Python only, never sent to the LLM |
| Gemini's role | Game Master: narrates **and** rules | Narrator only: describes verified facts |
| Adversarial protection | Prompt instruction only | Security filter, validation, output filter, invariant rollback |
| Without Gemini | Keyword fallback (no checks) | Template narrator (identical rules) |

---

## 1. Storyline

A young researcher answers a distress signal from **Blackwood Lighthouse**, a station abandoned for decades.

- The entrance door slams and locks behind them.
- A radio broadcasts the voice of **Elias Crane**, the keeper who drowned in 1913 while searching for his daughter **Elsie**.
- A bell tolls, and on the thirteenth toll the keeper takes a new assistant for ever.

To escape, the player must uncover the keeper's last secret and relight the lamp.

### Rooms, objects and puzzle chain (identical answers in code, tests and report)

**Room 1, The Watch Room:** radio receiver, keeper's journal, locked wooden cabinet, broken wall clock, control panel, iron ceiling hatch, entrance door.
**Room 2, The Lantern Gallery:** great lens, signal bell, salt-stained mirror, storm door.

| # | Puzzle | How the player finds the answer | Solution | Reward |
|---|---|---|---|---|
| 1 | The Stopped Clock | Journal: "the clock stopped at 3:47 the night Elsie went into the water" | set the clock to **3:47** | Brass Key |
| 2 | The Locked Cabinet | Requires the Brass Key | use the key | Copper Fuse, Tide Chart |
| 3 | The Dead Man's Frequency | Tide chart: "She answers only on 1913 kHz" | tune the radio to **1913** | Clue: lamp sequence |
| 4 | Light Her Home | Radio voice: "Red… amber… red… white"; the panel needs the fuse | insert the fuse, press **RED AMBER RED WHITE** | Iron hatch unlocks |
| 5 | The Keeper's Question (final) | Mirror: "Speak the name of the one I wait for" plus the journal | say **ELSIE** | Escape |

**Full walkthrough (Version 2):**

1. `examine journal`
2. `set the clock to 3:47`
3. `take key`
4. `open cabinet`
5. `take fuse`
6. `take chart`
7. `read tide chart`
8. `tune radio to 1913`
9. `use fuse on panel`
10. `press red amber red white`
11. `climb the hatch`
12. `examine mirror`
13. `say Elsie`

---

## 2. Technologies
- **Python 3.10+** (tested on 3.13)
- **Streamlit** for the web UI and session state
- **Google Gemini** through the official `google-genai` SDK (free-tier Flash model; default alias `gemini-flash-latest`, configurable)
- **pytest** and Streamlit `AppTest` for automated and headless UI tests
- **Mermaid** for the architecture diagrams
- All visual and audio assets are original: hand-written SVG and NumPy-synthesised sounds (`tools/generate_assets.py`)

## 3. Version 1 features
- Horror home screen (animated flicker, original lighthouse art), story introduction, instructions and an audio bell
- One room with 6+ objects and a second room; four connected puzzles plus a final escape
- Free natural-language chat with Gemini as the **AI Game Master**
- Memory: the last 30 conversation turns are sent with every request
- Items and escape are detected from `[ITEM: …]` / `[ESCAPED]` tags in the model's reply
- Success screen and restart
- Offline keyword Game Master if Gemini is unavailable

*Known, intentional limitations* (documented in `tests/test_cases.md`):
- the solutions sit inside the prompt;
- no inventory checks;
- puzzles can be skipped;
- the model's tags are trusted blindly.

## 4. Version 2 improvements
- **Structured state** (`game_state.py`): inventory, examined objects, collected clues, revealed/used items, solved puzzles, current room, door status, attempt counters, turn/bell counter and an audit log, all stored as JSON in `st.session_state`
- **Action parser** (`game_engine.parse`): natural language becomes `Action(verb, target, item, value)`. Many phrasings work ("unlock the cabinet with the key", "turn the dial to 1913 khz", "whisper 'Elsie'")
- **Validation engine**: checks that the object is in the current room, the item is in the inventory, the prerequisite puzzles are solved, the required clue is discovered and required flags (fuse inserted) are set
- **Deterministic puzzle verification** against `game_data.PUZZLES`. Wrong answers are counted, and hints get stronger after three failures
- **Security filter**: blocks prompt injection, administrator impersonation, requests for answers and skip attempts *before* parsing, without spending a turn or touching state
- **False-claim detection**: "I already have the fuse" is checked against the real inventory
- **State integrity**: every action runs against a snapshot. If `check_invariants()` finds a contradiction (e.g. hatch open without the lamp lit), the state is rolled back
- **Gemini as narrator only**:
  - it receives verified facts, discovered clues and the inventory, never solutions;
  - the player's text is marked as untrusted;
  - its output is filtered for undiscovered secrets and false unlock/escape claims;
  - unsafe output is replaced with the deterministic narration
- **Time pressure**: the bell tolls every 5 in-world actions, and the 13th toll ends the game (game-over screen)
- **UI**:
  - inventory, clue (parchment notes), examined, puzzle and door panels;
  - a six-milestone progress indicator and bell counter;
  - context-aware quick-action buttons and a context-aware `hint` command;
  - a JSON state viewer in the sidebar for demonstrations;
  - sounds for the bell and radio

---

## 5. Project structure
```
Bhoomika_AI_Escape_Room/
├── README.md                 ← this file
├── requirements.txt
├── .env.example              ← copy to .env and add your key
├── .gitignore
├── run_colab.md              ← optional Google Colab instructions
├── .streamlit/
│   ├── config.toml           ← dark theme
│   └── secrets.toml.example  ← alternative key storage
├── version_1/
│   ├── app.py                ← Streamlit UI (V1)
│   ├── game_master.py        ← Gemini Game Master + offline fallback
│   ├── game_data.py          ← story text + rules/answers for the prompt
│   ├── theme.py              ← CSS + asset helpers
│   └── assets/               ← lighthouse.svg, sigil.svg, bell.wav, radio_static.wav
├── version_2/
│   ├── app.py                ← Streamlit UI (V2) – displays state, never mutates it
│   ├── game_engine.py        ← security filter, parser, validation, puzzle checks
│   ├── game_state.py         ← GameState dataclass, JSON, invariants, rollback
│   ├── game_master.py        ← Gemini narrator, output filter, template fallback
│   ├── game_data.py          ← authoritative world data and solutions
│   ├── theme.py
│   └── assets/
├── tests/
│   ├── conftest.py           ← loads each version's modules in isolation
│   ├── test_version_1.py     ← documents V1 limitations (+ UI smoke test)
│   ├── test_version_2.py     ← V2 reliability & adversarial tests (+ full UI game)
│   ├── test_cases.md         ← test-case table with actual results
│   └── scenario_outputs.md   ← real V1 vs V2 transcripts for each case
├── diagrams/
│   ├── version_1_architecture.mmd / .png / .svg
│   └── version_2_architecture.mmd / .png / .svg
├── documentation/
│   ├── final_report.md
│   └── final_report.pdf
└── tools/
    ├── generate_assets.py    ← regenerates all original art/sounds
    ├── run_test_scenarios.py ← regenerates tests/scenario_outputs.md
    └── build_report_pdf.py   ← builds final_report.pdf from the Markdown
```

## 6. Installation
```bash
cd Bhoomika_AI_Escape_Room
python -m venv .venv
# Windows: .venv\Scripts\activate     macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

## 7. Configuring the Gemini API key safely
1. Create a free key at **Google AI Studio** (https://aistudio.google.com/apikey).
2. Choose **one** of these options:
   - **.env file (recommended locally):** `cp .env.example .env`, then set `GEMINI_API_KEY=...` in `.env`
   - **Environment variable:** `export GEMINI_API_KEY=...` (macOS/Linux) or `$env:GEMINI_API_KEY="..."` (PowerShell)
   - **Streamlit secrets:** copy `.streamlit/secrets.toml.example` to `.streamlit/secrets.toml`
3. Optional: `GEMINI_MODEL=<model name>` to choose a specific model if Google renames its free models.

The key is never hardcoded or displayed. `.env` and `secrets.toml` are in `.gitignore`.

**No key? No problem.** Both games detect this and switch to offline mode automatically. The sidebar badge shows *Gemini* or *Offline*. Version 2's rules are identical in both modes; only the narration style changes.

## 8. Running the games (from the project root)
**Version 1:**
```bash
streamlit run version_1/app.py
```
**Version 2:**
```bash
streamlit run version_2/app.py
```
Each opens at http://localhost:8501. To run both at once, add `--server.port 8502` to the second command.

## 9. Running the tests
```bash
python -m pytest tests -v                 # all tests (V1 + V2 + headless UI tests)
python -m pytest tests/test_version_2.py  # only Version 2
python tools/run_test_scenarios.py        # regenerate the V1-vs-V2 transcripts
```
Result at submission: **39 passed, 2 skipped**. The skipped tests are the live-Gemini tests, which run only when `GEMINI_API_KEY` is set.

## 10. How to play
1. Click **Enter the Lighthouse**, read the story, then click **Start Game**.
2. Type actions in plain English (or use the quick-action buttons in V2): *examine*, *take*, *open*, *use X on Y*, *set the clock to …*, *tune the radio to …*, *press colours*, *climb*, *say a word*.
3. Type `hint` when stuck and `inventory` to check your pockets (these do not cost time).
4. In V2, watch the bell counter. Every 5 actions it tolls, and the 13th toll ends the game.
5. Escape to see the success screen. **Restart** is always available.

## 11. Notes on honesty and limitations
- The build environment had no Gemini key, so live Gemini output was **not** tested. All test results come from offline mode and simulated model outputs (see `tests/test_cases.md`).
- Google renames free-tier models from time to time. Both versions try several model names and fall back to offline mode if none responds.
- The V2 security filter is pattern-based. It is a first line of defence, not the only one: even if an attack phrase slips through, the engine still validates every action and the narrator cannot change state.
