# Last Man Standing (LMS) Feature Specification
 
Version: 0.3
Status: Ready for implementation handoff
Date: 2026-11-24
 
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
- Auto-pick behavior for missing picks
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
- Choose the game's league and starting matchday at creation
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
 
- A skipper creates a game and selects exactly one league and a starting matchday.
- The game's official matchdays are the rounds of its league.
- The skipper automatically joins the game as a player and counts toward the minimum number of players. An eliminated skipper keeps their skipper permissions.
- Minimum required players for a game is 2.
   
### Game Start Rule
 
- A game starts at the pick deadline of its starting matchday.
- At that deadline, joining closes and any member without a pick is auto-picked (see section 8).
- If the game has fewer than 2 players at that deadline, its start moves to the next matchday.
   
### Late Join Rule
 
- Players cannot join a game after it has started.
   
### Game End States
 
- `completed`: winner or joint winners determined by the winner rules (section 11).
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
- Pick must be a team with a fixture in that matchday in the game's league.
- A player cannot pick the same team more than once within the same game.
- Pick deadline is the first kickoff of that official matchday.
- Skipper cannot edit/override picks after deadline.
- A player with no eligible team for a matchday is eliminated at the deadline.
   
## 8. Missing Pick and Auto-Pick Rules
 
If an active player has not submitted a pick by deadline:
 
- System auto-assigns the first alphabetically ordered eligible team
- Eligibility for auto-pick:
    - team has a fixture in the relevant matchday
    - team has not already been picked by that player in that game
- If the player has no eligible team, no auto-pick is made and the player is eliminated.
   
## 9. Match Result Interpretation
 
For a resolved pick:
 
- Team wins -> player survives
- Team draws -> player eliminated
- Team loses -> player eliminated
- Match abandoned -> treated as did not win -> player eliminated
- Match postponed -> player survives; the picked team still counts as used
   
## 10. Pick Visibility Rules
 
- Picks for a matchday are hidden until its pick deadline has passed and auto-picks have been assigned.
- Picks then become visible to players in that game.
   
## 11. Winner Rules
 
- Standard outcome: last remaining active player wins.
- All eliminated together: if every remaining active player is eliminated in the same matchday, they all become joint winners.
- Joint winner agreement:
    - If exactly 2 active players remain, either may propose a joint win and the other accepts or declines.
    - If the proposal isn't accepted before the next matchday's pick deadline, it lapses and play continues.
    - A joint win can be proposed again in any later matchday while both players remain active.
   
## 12. Admin Data Features
 
Admin must be able to:
 
- Create/update leagues
- Upload official matchday schedules
- Upload/update fixtures for each matchday
- Upload/update match results and status (including postponed/abandoned)
   
  These updates drive pick validation and elimination outcomes.
   
## 13. Functional Acceptance Criteria
 
1. Given a new user self-registers with unique username and email, when submitted, then account is created.
2. Given a duplicate username or duplicate email, when submitted, then registration is rejected.
3. Given a skipper creates a game, when it is created, then the skipper is a player in that game.
4. Given a player requests to join a game, when skipper approves, then membership is active.
5. Given a player is rejected from join request, when they request again later, then request is accepted for processing.
6. Given skipper directly invites a player, when player accepts, then player becomes a member.
7. Given game has fewer than 2 players at its starting matchday's deadline, when the deadline passes, then the game does not start and its start moves to the next matchday.
8. Given game has 2 or more players at its starting matchday's deadline, when the deadline passes, then the game starts and joining closes.
9. Given player attempts to join after game start, when request is made, then request is denied.
10. Given player selects a previously used team in same game, when submitting pick, then pick is rejected.
11. Given deadline passes without player pick, when deadline processing runs, then system auto-picks first alphabetical eligible unused team.
12. Given a player has no eligible team for a matchday, when deadline processing runs, then the player is eliminated.
13. Given picked team wins, when results are resolved, then player remains active.
14. Given picked team draws or loses, when results are resolved, then player is eliminated.
15. Given picked match is abandoned, when results are resolved, then player is eliminated.
16. Given picked match is postponed, when results are resolved, then player remains active and the picked team counts as used.
17. Given a matchday's pick deadline has not passed, when a player views picks, then picks are hidden.
18. Given a matchday's pick deadline has passed and auto-picks have been assigned, when a player views picks, then picks are visible.
19. Given one active player remains, when elimination update completes, then game is completed with that player as winner.
20. Given every remaining active player is eliminated in the same matchday, when results are resolved, then game is completed with all of them as joint winners.
21. Given exactly two active players remain and one proposes a joint win, when the other accepts before the next matchday's pick deadline, then game is completed with both as winners.
22. Given a joint-win proposal is not accepted before the next matchday's pick deadline, when the deadline passes, then the proposal lapses and play continues.
23. Given skipper abandons a game, when abandon action is confirmed, then game status becomes abandoned with no winner.
24. Given an eliminated player attempts re-entry to same game, when request is made, then request is rejected.
     
## 14. Handoff Prompt for Claude Code
 
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