# Automated Card Dealer & Interactive Game Management System

> **Senior Design — Project Plan (Draft v0.1)**  
> **Status:** Early planning; requirements, technologies, and design choices are subject to change.  
> **Repository:** [Smart-Card-Dealer](https://github.com/Amit-B8/Smart-Card-Dealer)

## 1. Project Overview

Traditional card games require players to shuffle and distribute cards, keep track of turns, enforce rules, and calculate scores. This project proposes a programmable tabletop card dealer connected to a phone-accessible game management application.

The device will distribute physical playing cards to players, while the application will coordinate shared game sessions, validate actions, track turns, and calculate scores. The goal is to demonstrate **reliable physical dealing integrated with a reusable, real-time multiplayer software system**.

The initial prototype will use a **manually shuffled standard 52-card deck**. Automatic shuffling and camera-based card recognition are potential future extensions, not core requirements.

## 2. Project Goals

1. Build a tabletop mechanism that aims toward player positions and dispenses one card at a time.
2. Detect and handle common dispensing faults, including jams and missed cards.
3. Implement a configurable software game engine with clear game states, legal actions, turn handling, and scoring.
4. Provide a mobile-friendly interface that lets multiple players join and interact with one game session.
5. Synchronize the embedded device, backend, and player dashboards; recover safely from interruptions.
6. Evaluate dealing reliability and demonstrate a complete game with multiple players.

## 3. Proposed Minimum Viable Prototype (MVP)

| Area | Initial target |
| --- | --- |
| Players | Support 2–4 player positions around a table. |
| Card supply | Standard deck, shuffled and loaded manually. |
| Dealing mechanism | Motorized single-card dispenser and rotating aiming base. |
| Sensors | Detect successful dispensing and report relevant faults; investigate how to identify double-deals reliably. |
| Embedded controller | Control motors, read sensors, and communicate device status. |
| Multiplayer application | Game selection, join code or QR code, player actions, scores, and live device status. |
| Game engine | Blackjack and Crazy Eights, plus a configurable dealing mode. |
| Card identities | Allow manual entry/confirmation when the game needs to know card rank or suit. |
| Data | Track current session state and persist completed game history. |
| Recovery | Pause safely on faults or disconnections and support controlled resumption. |

**Important boundary:** The dispenser can automate physical card delivery without knowing which card was dealt. Gameplay and scoring that depend on card identities will need a defined manual entry/confirmation workflow until card recognition is implemented.

### Stretch Goals (Not Required for MVP)

- Automated deck shuffling.
- Camera-based card recognition.
- Additional card games beyond the initial modes.
- More advanced statistics, analytics, or player profiles.
- Custom PCB or more sophisticated enclosure, if not needed for a reliable prototype.

## 4. Intended User Workflow

1. **Select:** A host creates a session, chooses a game, selects the number of players, and configures available rules.
2. **Join:** Players scan a QR code or enter a session code using their phones.
3. **Deal:** The backend issues a dealing sequence; the device rotates, dispenses cards, and reports completion or faults.
4. **Play / Track:** Players submit actions (for example, *hit* in Blackjack); the engine validates turns and synchronizes state across devices.
5. **Score / Repeat:** The game engine calculates outcomes using the known/entered card identities, displays results, and saves the session history.

If the dispenser jams or loses communication, the system should stop issuing additional deals until the current operation is reconciled.

## 5. Proposed System Architecture

| Subsystem | Responsibilities | Candidate technologies |
| --- | --- | --- |
| Embedded controller | Motor control, position management, sensor readings, local fault state, hardware acknowledgments. | ESP32; embedded C/C++ |
| Mechanical hardware | Single-card feed, rotating base, card path, mounts, enclosure. | Motors/servos/steppers, 3D-printed parts |
| Backend / game engine | Authoritative sessions, rules, turn validation, scoring, device commands, recovery, persistence. | Python/FastAPI **or** Node.js |
| Player interface | Join/host session, configure games, submit actions, view scores and dealer status. | React + TypeScript, mobile-friendly web UI |
| Communication | Device command/acknowledgment and live multiplayer state updates. | REST + WebSockets; device transport TBD |
| Database | Session records, game history, and any required configuration. | Database choice TBD |

**Working principle:** The backend should own the authoritative game state. The embedded controller should independently enforce safe physical operations and report confirmed outcomes; a command request should not be treated as a successful deal until the device acknowledges it.

### Likely Backend State Transitions

`LOBBY → INITIAL_DEAL → PLAYER_TURN(S) → SCORING → COMPLETED`

Hardware operations may temporarily enter states such as `WAITING_FOR_DEVICE`, `DEALING`, or `PAUSED_FAULT`. Exact state names and transition rules will be defined during architecture design.

## 6. Game Modes

### Blackjack

- Host configures a session and player count.
- Dealer distributes initial cards according to the selected rules.
- Players request actions such as **hit** or **stand** on their turn.
- Card ranks are entered or confirmed as needed to calculate hand values.
- The engine validates legal actions, determines outcomes, and updates scores.
- **Open design question:** How to physically handle and display the house/dealer hand, and which Blackjack rule variations to support.

### Crazy Eights

- Dealer distributes the configured starting hand to each player.
- Application tracks turn order, played cards, draw requests, and any required suit selection.
- Players identify played/drawn cards manually when necessary.
- Engine checks legal plays, win conditions, and scoring using agreed rules.
- **Open design question:** How players place/discard physical cards while keeping the software state accurate.

### Configurable Dealing Mode

- Host selects player count, number of cards per player, and dealing sequence.
- Device deals cards without requiring a complete game-rule engine.
- Useful for hardware tests and games not yet implemented in software.

## 7. Major Engineering Challenges

| Challenge | Why it matters | Initial approach |
| --- | --- | --- |
| Single-card reliability | Double-deals, missed cards, and jams corrupt the physical game state. | Prototype feed mechanism early; add sensing, mechanical guides, and repeatable tests. |
| Accurate aiming | A card must reach the intended player position. | Calibrate angular positions; test repeatability at 2–4 positions. |
| Reusable game rules | Different games require distinct actions, scoring, and turn transitions. | Separate common session/state logic from game-specific rule modules. |
| Real-time synchronization | Several phones and the dealer may observe or change the same session. | Make backend authoritative; broadcast state changes and validate actions server-side. |
| Recovery from partial deals | Retrying commands blindly could dispense extra cards. | Track command IDs and acknowledgments; reconcile incomplete operations before retrying. |
| Manual card identification | No camera recognition is planned for the MVP. | Provide a simple, explicit entry/confirmation flow and test its usability. |

## 8. Testing and Demonstration Plan

### Physical Dealer Tests

- Conduct **at least 100 dispensing trials** and record successful single-card deals, missed deals, double-deals, and jams.
- Test aiming and distribution at each supported player position.
- Verify that sensor feedback matches actual dispensing outcomes.
- Verify that a jam stops unsafe operations and can be cleared before resuming.
- Agree on a numeric pass threshold for dealing accuracy before final evaluation.

### Software and Integration Tests

- Test game-state transitions, legal/illegal player actions, and scoring rules.
- Verify that multiple phones see consistent updates within a session.
- Test rejoining after a phone disconnects without losing game state.
- Test device disconnection and partial-deal recovery without unintended extra cards.
- Verify that completed game results are saved and can be retrieved.

### Proposed Live Demo

1. Four players join a Blackjack session using their phones.
2. The device distributes initial cards toward the correct positions.
3. Players enter/confirm card identities and request actions such as *hit*.
4. The device responds to additional deal commands; dashboards update in real time.
5. The engine calculates scores/outcomes and saves the completed game.
6. A controlled jam or disconnection demonstrates fault detection and safe recovery.

## 9. Development Phases (Dates and Owners TBD)

| Phase | Focus | Suggested exit deliverable |
| --- | --- | --- |
| 1. Requirements and research | Confirm MVP, game-rule variants, budget, parts, and responsibilities. | Agreed requirements and initial system diagram. |
| 2. Hardware proof of concept | Build/test card feed, rotation, sensors, and controller firmware. | Repeatable one-card dispensing with basic fault reporting. |
| 3. Software foundation | Implement session API, game-engine interfaces, and basic phone UI. | Multiple clients can join and play a simulated game. |
| 4. End-to-end integration | Connect backend commands and embedded acknowledgments. | Software-triggered physical dealing tied to session state. |
| 5. Reliability and recovery | Test scoring, synchronization, mechanical faults, and reconnects. | Documented test results and fixes. |
| 6. Final demo and documentation | Prepare demonstration, architecture docs, and evaluation results. | Working prototype and senior design presentation. |

## 10. Decisions to Make as a Team

- [ ] Confirm the required MVP and which features are optional.
- [ ] Select exact Blackjack and Crazy Eights rule variants.
- [ ] Define the card-identity entry workflow and who performs it.
- [ ] Decide how the Blackjack house/dealer hand is physically dealt.
- [ ] Choose the backend language/framework and database.
- [ ] Choose the device-to-backend transport and network topology.
- [ ] Select motors, drivers, sensors, and initial mechanical feed design.
- [ ] Set power, safety, budget, size, and noise constraints.
- [ ] Define pass/fail thresholds for dealing and aiming reliability.
- [ ] Assign subsystem owners and milestone dates.

## 11. Related Work to Review

- **Cornell Automatic Card Shuffler & Dealer (2022):** Mechanical shuffling/dealing with a graphical interface.
- **ESE5160 Gambler Automated Card Dealer (2025):** IoT-oriented dealer with motors, PCB, and mobile/web controls.
- **Dealtron MAX6:** Commercial card distribution for preset games.

Our intended distinction is the combination of programmable physical distribution, a reusable rule/scoring engine, synchronized multiplayer interaction, game history, and recovery from hardware faults.

> Add verified source links, detailed comparisons, and citations before including related work in a formal report.

## 12. Working Notes

This document is a **living plan**, not a finalized specification. Update it as the team confirms requirements and completes prototypes. Keep the README as the concise public project overview; use this file for engineering decisions, milestones, and scope changes.
