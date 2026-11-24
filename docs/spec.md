# Last Man Standing (LMS) Feature Specification
 
Version: 0.2
Status: Ready for implementation handoff
Date: 2026-11-23
 
## 1. Product Overview
 
Last Man Standing (LMS) is a football prediction game platform where users join games and try to survive each official matchday by selecting one team they believe will win.
 
A player is eliminated from a game if their selected team does not win.
 
## 2. Scope
 
### In Scope (MVP)
 
- User self-registration and role-based access
- Game creation and management by game skippers
- Join request workflow and skipper approval
- Skipper direct add with player acceptance
- Matchday pick submission with one-pick-per-player-per-matchday
- One-team-once-per-game rule per player
- Match outcome resolution and elimination
- Auto-pick behavior for missing picks/repicks
- Pick visibility rules
- Admin upload/management of leagues, official matchdays, fixtures, and results
   
### Out of Scope (MVP)
 
- Betting, payments, or prize processing
- Mobile/native app requirements
- Stack-specific technical architecture details
   
## 3. Roles and Permissions
 
### Player (global)
 
- Participate in games
- Request to join games
- Accept or decline skipper direct-add invitations
- Submit picks while active in a game
   
### Skipper (game-scoped)
 
- Create a game
- Configure leagues for that game at creation
- Approve or reject join requests
- Directly invite/add players to the game (player must accept)
- Abandon a game
   
  Skipper permissions apply only to that specific game.
   
### Admin (global)
 
- Administer platform users and global roles
- Upload/manage official matchdays, fixtures, and results
- Manage league metadata and seasonal data required by game logic
   
  Admin does not override skipper pick decisions or alter locked picks.
   
## 4. User and Role Management
 
- Users self-register.
- `username` and `email` are both required.
- `username` and `email` must each be unique across all users.
- Users authenticate using username and/or email (implementation-defined), but both identities are stored and unique.
- Every registered user has `player` capability.
- Global `admin` role can be assigned/revoked only by admins.
   
## 5. Game Lifecycle
 
### Game Creation
 
- A skipper creates a game and selects one or more leagues.
- Minimum required players for a game is 2.
   
### Game Start Rule
 
- A game starts when all players in that game have completed their first pick.
   
### Late Join Rule
 
- Players cannot join a game after it has started.
   
### Game End States
 
- `completed`: winner determined by elimination rules.
- `abandoned`: skipper manually abandons game; no winner is recorded.
   
## 6. Join and Membership Flows
 
### Join Request
 
- Player submits join request.
- Skipper approves or rejects.
- If rejected, player can request again later (no cooldown/limit in MVP).
   
### Skipper Direct Add
 
- Skipper sends add/invite to player.
- Player must explicitly accept before becoming a member.
   
### Re-entry
 
- Eliminated players cannot re-enter the same game.
   
## 7. Pick Rules
 
- One pick per active player per official matchday.
- Pick must be a team with a fixture in that matchday in one of the game's configured leagues.
- A player cannot pick the same team more than once within the same game.
- Pick deadline is the first kickoff of that official matchday.
- Skipper cannot edit/override picks after deadline.
   
## 8. Missing Pick and Auto-Pick Rules
 
If an active player has not submitted a pick by deadline:
 
- System auto-assigns the first alphabetically ordered eligible team
- Eligibility for auto-pick:
    - team has a fixture in the relevant matchday
    - team has not already been picked by that player in that game
   
## 9. Match Result Interpretation
 
For a resolved pick:
 
- Team wins -> player survives
- Team draws -> player eliminated
- Team loses -> player eliminated
- Match abandoned -> treated as did not win -> player eliminated
- Match postponed -> player must repick
   
## 10. Postponed Match Repick Rules
 
- If a selected match is postponed, the player remains pending and must submit a replacement pick.
- Repick uses the same eligibility constraints as standard picks.
- Repick deadline behavior follows missing-pick policy: if no repick by deadline, auto-pick applies using alphabetical eligible team logic.
   
## 11. Pick Visibility Rules
 
- Picks are hidden until all picks for that gameweek are finalized.
- This includes auto-picks and postponed-match repicks.
- Once all picks are finalized for the gameweek, picks become visible to players in that game.
   
## 12. Winner Rules
 
- Standard outcome: last remaining active player wins.
- Joint winner exception:
    - If exactly 2 active players remain, both may explicitly agree to be joint winners.
    - Agreement must be explicit from both players.
   
## 13. Admin Data Features
 
Admin must be able to:
 
- Create/update leagues
- Upload official matchday schedules
- Upload/update fixtures for each matchday
- Upload/update match results and status (including postponed/abandoned)
   
  These updates drive pick validation and elimination outcomes.
   
## 14. Functional Acceptance Criteria
 
1. Given a new user self-registers with unique username and email, when submitted, then account is created.
2. Given a duplicate username or duplicate email, when submitted, then registration is rejected.
3. Given a player requests to join a game, when skipper approves, then membership is active.
4. Given a player is rejected from join request, when they request again later, then request is accepted for processing.
5. Given skipper directly invites a player, when player accepts, then player becomes a member.
6. Given game has fewer than 2 players, when start conditions are evaluated, then game cannot start.
7. Given all game players have made first picks, when condition is met, then game starts.
8. Given player attempts to join after game start, when request is made, then request is denied.
9. Given player selects a previously used team in same game, when submitting pick, then pick is rejected.
10. Given deadline passes without player pick, when deadline processing runs, then system auto-picks first alphabetical eligible unused team.
11. Given picked team wins, when results are resolved, then player remains active.
12. Given picked team draws or loses, when results are resolved, then player is eliminated.
13. Given picked match is abandoned, when results are resolved, then player is eliminated.
14. Given picked match is postponed, when resolution occurs, then player must repick.
15. Given postponed repick is missing at deadline, when deadline processing runs, then system auto-picks first alphabetical eligible unused team.
16. Given not all gameweek picks are finalized, when a player views picks, then picks are hidden.
17. Given all gameweek picks are finalized, when a player views picks, then picks are visible.
18. Given one active player remains, when elimination update completes, then game is completed with that player as winner.
19. Given exactly two active players remain and both submit joint-win agreement, when processed, then game is completed with both as winners.
20. Given skipper abandons a game, when abandon action is confirmed, then game status becomes abandoned with no winner.
21. Given an eliminated player attempts re-entry to same game, when request is made, then request is rejected.
     
## 15. Handoff Prompt for Claude Code
 
Use this feature specification as the source of truth.
 
Implementation constraints:
 
1. Implement only what is in scope.
2. Treat each acceptance criterion as a required test scenario.
3. If any behavior is ambiguous, stop and ask clarifying questions.
4. Do not add hidden fallback behavior beyond explicit rules.
5. Enforce role boundaries exactly: global roles vs game-scoped skipper permissions.
    
   Expected output from Claude Code:
    
- Source code changes
- Tests mapped to acceptance criteria
- List of assumptions and any unresolved ambiguities
   
   
   
## The End