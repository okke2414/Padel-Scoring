# Implementation Plan: Padel Scoring System

## Overview

Implement a single-file (`index.html`) client-side Padel scoring web app using HTML5, Tailwind CSS (CDN), and Vanilla JavaScript. The scoring engine is extracted into a parallel `scoring-engine.js` module so it can be tested independently with Vitest + fast-check. Implementation proceeds layer by layer: project skeleton → scoring engine → render layer → UI screens → testing.

## Tasks

- [-] 1. Project skeleton and test infrastructure
  - Create `index.html` with Tailwind CDN, screen containers (setup, match, result), and `<script>` tags importing `scoring-engine.js` and `app.js`
  - Create `scoring-engine.js` as an ES module exporting all engine functions and the initial `MatchState` factory
  - Create `app.js` that imports `scoring-engine.js` and wires DOM events
  - Initialize a `package.json` and install `vitest` and `fast-check` as dev dependencies
  - Create `vitest.config.js` configured for jsdom environment
  - Create `scoring-engine.test.js` and `app.test.js` as placeholder test files
  - _Requirements: 10.4, 10.5_

- [ ] 2. MatchState data model and core engine utilities
  - [~] 2.1 Implement the `MatchState` factory (`initMatch`) with all fields defined in the design
    - Include `players`, `sets`, `currentSet`, `gamesA`, `gamesB`, `pointsA`, `pointsB`, `gameMode`, `deuceCount`, `gamePhase`, `servingTeam`, `initialServer`, `tiebreakPointsPlayed`, `previousState`, `matchOver`, `winner`, `setsA`, `setsB`
    - _Requirements: 2.1, 2.2_
  - [~] 2.2 Write property test for initial server reflection (Property 4)
    - **Property 4: Initial server is always reflected in match state**
    - **Validates: Requirements 2.2, 2.3**
  - [~] 2.3 Implement `cloneState` (deep clone) and `resetToSetup` utilities
    - `cloneState` must produce a fully independent copy of `MatchState` including nested objects
    - _Requirements: 9.1, 9.2_

- [ ] 3. Input validation
  - [~] 3.1 Implement `validateNames(formData)` returning a `ValidationResult` with per-field errors
    - Reject fields that are empty or contain only whitespace characters
    - _Requirements: 1.3, 1.4_
  - [~] 3.2 Write property test for whitespace name rejection (Property 1)
    - **Property 1: Whitespace names are always rejected**
    - **Validates: Requirements 1.3, 1.4**
  - [~] 3.3 Write property test for name length cap (Property 3)
    - **Property 3: Player name length is capped at 30 characters**
    - **Validates: Requirements 1.6**

- [ ] 4. Normal game scoring engine
  - [~] 4.1 Implement `scorePoint(team)` for NORMAL game phase: advance `pointsX` through 0 → 1 → 2 → 3 following FIP sequence, trigger game win when at 3 and opponent < 3
    - _Requirements: 3.2, 3.3_
  - [~] 4.2 Write property test for FIP point sequence (Property 5)
    - **Property 5: Normal game point progression follows FIP sequence**
    - **Validates: Requirements 3.2**
  - [~] 4.3 Implement deuce/advantage/star-point transitions in `scorePoint`: DEUCE_1 → ADV_x_1 → DEUCE_2 → ADV_x_2 → STAR_POINT → GAME_WIN
    - Enforce maximum two deuces; `deuceCount` must never exceed 2
    - _Requirements: 3.4–3.14_
  - [~] 4.4 Write property test for FSM phase validity (Property 6)
    - **Property 6: Game phase state machine is always valid**
    - **Validates: Requirements 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10, 3.11, 3.12, 3.13, 3.14**
  - [~] 4.5 Write property test for game win resets (Property 7)
    - **Property 7: Game win always resets points and increments game score**
    - **Validates: Requirements 3.3, 3.6, 4.1**

- [ ] 5. Set and match progression
  - [~] 5.1 Implement `checkSetWin()`: detect 6-x (x ≤ 4), 7-5, and 7-6 (post-tiebreak) win conditions; reset game score and increment set score on win
    - _Requirements: 4.2, 4.3, 4.5, 4.6_
  - [~] 5.2 Write property test for set win conditions (Property 8)
    - **Property 8: Set win conditions are correctly enforced**
    - **Validates: Requirements 4.2, 4.3, 4.6**
  - [~] 5.3 Implement `checkMatchWin()`: set `matchOver = true` and assign `winner` when `setsA === 2` or `setsB === 2`
    - _Requirements: 8.1, 8.3_
  - [~] 5.4 Write property test for match-over trigger (Property 16)
    - **Property 16: Match ends exactly when Set_Score reaches 2**
    - **Validates: Requirements 8.1, 8.3**

- [~] 6. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 7. Tiebreak scoring
  - [~] 7.1 Implement tiebreak activation: when game score reaches 6-6 in sets 1 or 2, set `gameMode = 'TIEBREAK'` and reset point counters
    - _Requirements: 4.4, 5.1_
  - [~] 7.2 Write property test for tiebreak activation at 6-6 (Property 9)
    - **Property 9: Tiebreak activates at exactly 6-6**
    - **Validates: Requirements 4.4, 5.1**
  - [~] 7.3 Implement tiebreak `scorePoint` logic: raw integer increments, win at ≥ 7 with 2-point lead (or any lead after 6-6)
    - _Requirements: 5.2, 5.3, 5.4_
  - [~] 7.4 Write property test for tiebreak win conditions (Property 10)
    - **Property 10: Tiebreak win requires correct lead**
    - **Validates: Requirements 5.2, 5.3, 5.4**
  - [~] 7.5 Implement `rotateTiebreakService()`: change serve after point 1, then every 2 points thereafter
    - _Requirements: 5.5_
  - [~] 7.6 Write property test for tiebreak service rotation (Property 11)
    - **Property 11: Tiebreak service rotation is correct**
    - **Validates: Requirements 5.5, 6.6**

- [ ] 8. Super Tiebreak (third set)
  - [~] 8.1 Implement Super Tiebreak activation: when set score reaches 1-1, set `gameMode = 'SUPER_TIEBREAK'` instead of starting a normal third set
    - _Requirements: 6.1, 6.2_
  - [~] 8.2 Write property test for Super Tiebreak activation at 1-1 (Property 12)
    - **Property 12: Super Tiebreak activates at 1-1 in sets**
    - **Validates: Requirements 6.1**
  - [~] 8.3 Implement Super Tiebreak `scorePoint` logic: raw integer increments, win at ≥ 10 with 2-point lead (or any lead after 9-9); apply same service rotation as tiebreak
    - _Requirements: 6.3, 6.4, 6.5, 6.6_
  - [~] 8.4 Write property test for Super Tiebreak win conditions (Property 13)
    - **Property 13: Super Tiebreak win requires correct lead**
    - **Validates: Requirements 6.3, 6.4, 6.5**

- [ ] 9. Service rotation for normal games
  - [~] 9.1 Implement post-game serve transfer: after a normal game ends, `servingTeam` switches to the opposing team
    - _Requirements: 7.1_
  - [~] 9.2 Write property test for service alternation after each game (Property 14)
    - **Property 14: Service alternates after every normal game**
    - **Validates: Requirements 7.1**
  - [~] 9.3 Implement new-set service continuation: at the start of each new set, assign serve to the team that received in the last game of the previous set
    - _Requirements: 7.2_
  - [~] 9.4 Write property test for set service continuation (Property 15)
    - **Property 15: Set service continuation rule is respected**
    - **Validates: Requirements 7.2**

- [ ] 10. Undo functionality
  - [~] 10.1 Implement `undoLastPoint()`: restore `MatchState` from `previousState` snapshot; disable Undo when `previousState === null`
    - Ensure `cloneState` is called before every `scorePoint` mutation to capture the snapshot
    - _Requirements: 9.1, 9.2, 9.3, 9.4_
  - [~] 10.2 Write property test for undo round-trip (Property 17)
    - **Property 17: Undo restores the complete prior state**
    - **Validates: Requirements 9.1, 9.2**

- [~] 11. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 12. Helper display utilities
  - [~] 12.1 Implement `getPointLabel(points, phase, team)`: map internal `pointsX` values to FIP display labels (0, 15, 30, 40, Deuce, Adv, ★ Star Point) and raw integers for tiebreak/super tiebreak modes
    - _Requirements: 3.13, 5.1, 6.2_
  - [~] 12.2 Implement `getGameModeLabel(mode)`: return "Normal", "Tiebreak", or "Super Tiebreak" string for badge display
    - _Requirements: 10.3_

- [ ] 13. Setup screen UI
  - [~] 13.1 Build the Setup screen HTML and JavaScript: four name inputs grouped by team, `maxlength="30"` on each input, inline error `<span>` elements, "Start Match" button
    - _Requirements: 1.1, 1.2, 1.6_
  - [~] 13.2 Wire form submission to `validateNames`; show inline errors on invalid fields, prevent navigation on failure
    - _Requirements: 1.3, 1.4_
  - [~] 13.3 After valid name submission, reveal the server-selection radio group (Team A / Team B) and a "Begin Match" button that calls `initMatch` and transitions to the Match screen
    - _Requirements: 2.1, 2.2_
  - [~] 13.4 Write unit tests for setup screen validation behaviour
    - Test: four fields present grouped by team
    - Test: all-whitespace names blocked, valid names accepted
    - Test: server-selection UI appears after valid names
    - Test: match screen shown after server selection
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 2.1_

- [ ] 14. Match screen and scoreboard render layer
  - [~] 14.1 Implement `render()`: update team name labels ("P1 / P2" format), set score columns, game score, point score, game mode badge, and serving indicator in the DOM after every state change
    - _Requirements: 10.1, 10.2, 10.3_
  - [~] 14.2 Write property test for team name display format (Property 2)
    - **Property 2: Team names are displayed in "Player1 / Player2" format**
    - **Validates: Requirements 1.5**
  - [~] 14.3 Write property test for scoreboard rendering completeness (Property 18)
    - **Property 18: Scoreboard always renders all required fields**
    - **Validates: Requirements 10.1, 10.3**
  - [~] 14.4 Wire Team A and Team B score buttons to `scorePoint('A')` / `scorePoint('B')` then `render()`; disable buttons when `matchOver === true`
    - _Requirements: 3.1, 8.3_
  - [~] 14.5 Wire Undo button to `undoLastPoint()` then `render()`; disable when `previousState === null`
    - _Requirements: 9.1, 9.3_
  - [~] 14.6 Add serving indicator display next to the serving team's row; update in `render()`
    - _Requirements: 7.3, 10.2_
  - [~] 14.7 Write unit tests for match screen DOM behaviour
    - Test: score buttons present and enabled at match start
    - Test: Undo button disabled at match start
    - Test: game mode badge shows "Tiebreak" during tiebreak
    - Test: game mode badge shows "Super Tiebreak" during super tiebreak
    - Test: score buttons disabled after match completion
    - _Requirements: 3.1, 8.3, 9.3, 10.3_

- [ ] 15. Result screen
  - [~] 15.1 Implement the Result screen: display winning team names, final set scores for each completed set, and a "New Match" button
    - _Requirements: 8.2_
  - [~] 15.2 Wire "New Match" button to `resetToSetup()` and navigate back to the Setup screen with a blank state
    - _Requirements: 8.4_
  - [~] 15.3 Write unit tests for result screen
    - Test: correct winner name shown
    - Test: final set scores displayed
    - Test: "New Match" button returns to blank setup screen
    - _Requirements: 8.2, 8.4_

- [ ] 16. Responsiveness and Tailwind styling
  - [~] 16.1 Apply Tailwind utility classes to all three screens; verify layout renders correctly at 320 px and 1920 px widths with no horizontal scrolling and no custom CSS files
    - _Requirements: 10.4, 10.5_

- [~] 17. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP delivery
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation throughout development
- Property tests (P1–P18) validate universal correctness properties using fast-check with ≥ 100 iterations each
- Unit tests validate specific scenarios and DOM behaviour using Vitest + jsdom
- The scoring engine (`scoring-engine.js`) must remain free of DOM imports so it can be tested in Node.js
- Tag format for property tests: `// Feature: padel-scoring-system, Property <N>: <short description>`

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["2.1", "2.3"] },
    { "id": 1, "tasks": ["2.2", "3.1"] },
    { "id": 2, "tasks": ["3.2", "3.3", "4.1"] },
    { "id": 3, "tasks": ["4.2", "4.3"] },
    { "id": 4, "tasks": ["4.4", "4.5", "5.1"] },
    { "id": 5, "tasks": ["5.2", "5.3", "7.1"] },
    { "id": 6, "tasks": ["5.4", "7.2", "7.3", "9.1"] },
    { "id": 7, "tasks": ["7.4", "7.5", "8.1", "9.2", "9.3", "10.1"] },
    { "id": 8, "tasks": ["7.6", "8.2", "8.3", "9.4", "10.2"] },
    { "id": 9, "tasks": ["8.4", "12.1", "12.2"] },
    { "id": 10, "tasks": ["13.1", "13.2"] },
    { "id": 11, "tasks": ["13.3", "14.1"] },
    { "id": 12, "tasks": ["13.4", "14.2", "14.3", "14.4", "14.5", "14.6"] },
    { "id": 13, "tasks": ["14.7", "15.1"] },
    { "id": 14, "tasks": ["15.2", "15.3", "16.1"] }
  ]
}
```
