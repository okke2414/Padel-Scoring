# Requirements Document

## Introduction

A web-based Padel scoring system built with HTML5, Tailwind CSS, and Vanilla JavaScript. The application allows two teams to track a full Padel match according to the FIP (Federación Internacional de Pádel) official rulebook, including the 2026 amendment that limits deuce to a maximum of two occurrences per game, after which a Star Point decides the game. Each team is composed of two players. The system manages point-by-point scoring through games, sets, and the full match, enforcing FIP rules for deuce, advantage, star point, tiebreaks, and the super tiebreak (third set).

## Glossary

- **App**: The web-based Padel scoring system
- **Match**: A full Padel contest consisting of a best-of-three sets format
- **Set**: A collection of games won by one team; first team to reach 6 games (with a 2-game lead, or via tiebreak at 6–6) wins the set
- **Game**: A single scoring unit within a set; won by reaching 40 and scoring once more, subject to deuce/advantage rules
- **Tiebreak**: A special game played at 6–6 within sets 1 or 2; first team to reach 7 points with a 2-point lead wins the set
- **Super_Tiebreak**: A deciding game played instead of a full third set; first team to reach 10 points with a 2-point lead wins the match
- **Point**: The smallest scoring unit; values are 0, 15, 30, 40, Deuce, and Advantage
- **Deuce**: The state of a game when both teams have reached 40; under 2026 FIP rules a maximum of two deuces are allowed per game
- **Deuce_1**: The first deuce state; reached when both teams are at 40–40 for the first time
- **Deuce_2**: The second and final deuce state; reached when the score returns to 40–40 after Advantage_1 was not converted
- **Advantage**: The state after one team scores from a Deuce state; if the Advantage team scores again they win the game; if the other team scores the game advances to the next deuce or Star Point
- **Star_Point**: The decisive point state reached after Advantage_2 is not converted (2026 FIP rule); the next team to score wins the game regardless of which team it is; displayed as "★ Star Point" on the Scoreboard
- **Team**: A pair of two players competing together on one side of the court
- **Player**: An individual participant; each Team has exactly two Players
- **Scoreboard**: The visual display of the current match score
- **Server**: The player currently serving; service alternates per game and rotates per set
- **Set_Score**: The count of sets won by each team in the current match
- **Game_Score**: The count of games won by each team in the current set
- **Point_Score**: The current point tally within the current game (0, 15, 30, 40, Deuce, Advantage)

## Requirements

### Requirement 1: Player Name Input

**User Story:** As a match organizer, I want to enter the names of all four players before the match starts, so that the scoreboard displays personalized team labels throughout the match.

#### Acceptance Criteria

1. THE App SHALL present a setup screen before the match begins that contains four text input fields, one for each player.
2. THE App SHALL group the four input fields into two teams: Team A (Player 1 and Player 2) and Team B (Player 3 and Player 4).
3. WHEN the organizer submits the setup form, THE App SHALL validate that all four name fields contain at least one non-whitespace character.
4. IF any name field is empty or contains only whitespace, THEN THE App SHALL display an inline validation error for each invalid field and prevent the match from starting.
5. WHEN valid names are submitted, THE App SHALL display each team on the Scoreboard using the format "Player1 / Player2" for that team.
6. THE App SHALL limit each player name to a maximum of 30 characters.

---

### Requirement 2: Match Initialization

**User Story:** As a match organizer, I want to designate which team serves first, so that the match begins with the correct serving order per FIP rules.

#### Acceptance Criteria

1. WHEN the setup form is submitted with valid names, THE App SHALL present a selection to choose which team serves first.
2. THE App SHALL record the selected serving team as the initial Server for the first game of the first set.
3. WHEN the first point of the match is scored, THE App SHALL reflect the current server on the Scoreboard.

---

### Requirement 3: Point Scoring

**User Story:** As a scorekeeper, I want to award a point to a team with a single action, so that the Point_Score advances correctly according to FIP rules.

#### Acceptance Criteria

1. THE App SHALL display two clearly labelled score buttons, one for Team A and one for Team B, throughout the match.
2. WHEN a score button is pressed for a team during a normal game (not a Tiebreak or Super_Tiebreak), THE App SHALL advance that team's Point_Score according to the FIP tennis-style sequence: 0 → 15 → 30 → 40.
3. WHEN a team's Point_Score is 40 and the opposing team's Point_Score is below 40, THE App SHALL award the game to the team that scored and reset the Point_Score to 0–0.
4. WHEN both teams have a Point_Score of 40, THE App SHALL set the game state to Deuce 1 (first deuce).
5. WHILE the game state is Deuce 1, WHEN a team scores, THE App SHALL set that team's state to Advantage 1.
6. WHILE the game state is Advantage 1, WHEN the Advantage team scores, THE App SHALL award the game to the Advantage team and reset the Point_Score to 0–0.
7. WHILE the game state is Advantage 1, WHEN the non-Advantage team scores, THE App SHALL advance the game state to Deuce 2 (second deuce).
8. WHILE the game state is Deuce 2, WHEN a team scores, THE App SHALL set that team's state to Advantage 2.
9. WHILE the game state is Advantage 2, WHEN the Advantage team scores, THE App SHALL award the game to the Advantage team and reset the Point_Score to 0–0.
10. WHILE the game state is Advantage 2, WHEN the non-Advantage team scores, THE App SHALL advance the game state to Star Point.
11. WHEN the game state is Star Point, THE App SHALL display a clear "★ Star Point" indicator on the Scoreboard to signal that the next point wins the game regardless of which team scores it.
12. WHEN the game state is Star Point and any team scores, THE App SHALL award the game to the team that scored the Star Point and reset the Point_Score to 0–0.
13. THE App SHALL display the Point_Score using the FIP labels: 0, 15, 30, 40, Deuce, and Advantage (with an indicator for which team holds Advantage), and SHALL display "★ Star Point" when the game reaches the Star Point state.
14. THE App SHALL track the deuce count internally (Deuce 1, Deuce 2) to enforce the maximum two-deuce rule introduced by FIP in 2026.

---

### Requirement 4: Game and Set Progression

**User Story:** As a scorekeeper, I want the set and match score to update automatically when a game is won, so that I do not have to manually track set scores.

#### Acceptance Criteria

1. WHEN a team wins a game, THE App SHALL increment that team's Game_Score for the current set by 1.
2. WHEN a team's Game_Score reaches 6 and the opposing team's Game_Score is 4 or fewer, THE App SHALL award the set to that team.
3. WHEN a team's Game_Score reaches 7 and the opposing team's Game_Score is 5, THE App SHALL award the set to that team (7–5 set).
4. WHEN both teams' Game_Scores reach 6, THE App SHALL start a Tiebreak game for sets 1 and 2.
5. WHEN a team wins the Tiebreak game, THE App SHALL record the set score as 7–6 for the winning team and start the next set.
6. WHEN a team wins a set, THE App SHALL increment that team's Set_Score by 1 and reset the Game_Score to 0–0.

---

### Requirement 5: Tiebreak Scoring

**User Story:** As a scorekeeper, I want the app to switch to tiebreak scoring at 6–6 in a set, so that the correct FIP tiebreak rules are applied automatically.

#### Acceptance Criteria

1. WHEN a Tiebreak begins, THE App SHALL display the Point_Score as integers starting from 0, replacing the standard 0/15/30/40 labels.
2. WHEN a team's tiebreak point count reaches 7 and the opposing team's count is 5 or fewer, THE App SHALL award the Tiebreak to that team.
3. WHEN both teams' tiebreak point counts reach 6, THE App SHALL continue the Tiebreak until one team leads by 2 points.
4. WHEN a team leads by 2 points after both teams have reached 6 tiebreak points, THE App SHALL award the Tiebreak to that team.
5. WHEN the Tiebreak is active, THE App SHALL change the serving side after the first point and then after every 2 points thereafter, following FIP tiebreak service rotation rules.

---

### Requirement 6: Super Tiebreak (Third Set)

**User Story:** As a scorekeeper, I want the app to start a Super Tiebreak when the match reaches one set each, so that the deciding set follows FIP rules.

#### Acceptance Criteria

1. WHEN each team has won one set, THE App SHALL start a Super_Tiebreak instead of a third full set.
2. WHEN a Super_Tiebreak begins, THE App SHALL display the Point_Score as integers starting from 0.
3. WHEN a team's Super_Tiebreak point count reaches 10 and the opposing team's count is 8 or fewer, THE App SHALL award the Super_Tiebreak and the Match to that team.
4. WHEN both teams' Super_Tiebreak point counts reach 9, THE App SHALL continue until one team leads by 2 points.
5. WHEN a team leads by 2 points after both teams have reached 9 Super_Tiebreak points, THE App SHALL award the Super_Tiebreak and the Match to that team.
6. WHEN the Super_Tiebreak is active, THE App SHALL apply the same service rotation as a standard Tiebreak (change after first point, then every 2 points).

---

### Requirement 7: Service Rotation

**User Story:** As a scorekeeper, I want the app to track and display which team is serving, so that players know whose turn it is to serve without manual tracking.

#### Acceptance Criteria

1. WHEN a game ends (excluding Tiebreak and Super_Tiebreak), THE App SHALL transfer the serve to the opposing team for the next game.
2. WHEN a new set begins, THE App SHALL assign the first serve to the team that received serve in the last game of the previous set, following FIP service continuation rules.
3. THE App SHALL display a visual indicator on the Scoreboard next to the currently serving team.

---

### Requirement 8: Match Completion

**User Story:** As a scorekeeper, I want the app to detect when the match is over and display the winner, so that the result is clearly communicated.

#### Acceptance Criteria

1. WHEN a team's Set_Score reaches 2, THE App SHALL declare that team the match winner.
2. WHEN the match is won, THE App SHALL display a match-result screen showing the winning team's names and the final set scores.
3. WHEN the match is won, THE App SHALL disable the score buttons to prevent further scoring.
4. THE App SHALL provide a "New Match" button on the match-result screen that resets all scores and navigates back to the setup screen.

---

### Requirement 9: Score Correction (Undo)

**User Story:** As a scorekeeper, I want to undo the last scored point, so that I can correct accidental input without restarting the match.

#### Acceptance Criteria

1. THE App SHALL provide an Undo button that reverses the most recently recorded point.
2. WHEN the Undo button is pressed, THE App SHALL restore the complete match state (Point_Score, Game_Score, Set_Score, serving team, and game mode) to what it was before the last point was scored.
3. WHEN no points have been scored in the current match, THE App SHALL disable the Undo button.
4. THE App SHALL support only a single level of undo (one point at a time).

---

### Requirement 10: Scoreboard Display

**User Story:** As a spectator or player, I want to see a clear, real-time scoreboard, so that I can follow the match score at a glance.

#### Acceptance Criteria

1. THE Scoreboard SHALL display the team names, Set_Score, Game_Score, and Point_Score simultaneously and update after every point.
2. THE Scoreboard SHALL clearly differentiate the serving team from the receiving team using a visual indicator.
3. THE Scoreboard SHALL display the current game mode (Normal, Tiebreak, or Super_Tiebreak) so scorekeepers are always aware of the active scoring rules.
4. THE App SHALL be responsive and render correctly on screens from 320px to 1920px wide without horizontal scrolling.
5. THE App SHALL apply Tailwind CSS utility classes for all styling with no custom CSS files.
