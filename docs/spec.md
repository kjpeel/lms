# Last Man Standing (LMS) Feature Specification
 
Version: 0.4
Status: Ready for implementation handoff
Date: 2026-11-24
 
## 1. Product Overview
 
Last Man Standing (LMS) is a football prediction game platform where users join games and try to survive each official matchday by selecting one team they believe will win.
 
A player is eliminated from a game if their selected team does not win.
 
## 2. Scope
 
### In Scope (MVP)
 
- User sign-in with Google and role-based access
- Game creation and management by game skippers
- Public game list for browsing games
- Join request workflow and skipper approval
- Skipper direct add with player acceptance
- Matchday pick submission with one-pick-per-player-per-matchday
- One-team-once-per-game rule per player
- Match outcome resolution and elimination
- Auto-pick behavior for missing picks
- Pick visibility rules
- Admin management of system-wide league, matchday, fixture and result data, through web forms and CSV upload
   
### Out of Scope (MVP)
 
- Betting, payments, or prize processing
- Mobile/native app requirements
- Stack-specific technical architecture details
- Email notifications (email is collected for future notifications)
- Automated fixture and result data feeds (possible later addition)
- Sign-in providers other than Google, and username/password sign-in
   
## 3. Roles and Permissions
 
### Player (global)
 
- Participate in games
- Browse games and request to join them
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
 
- Users sign in with Google. The app never sees or stores a password.
- On first sign-in, the user chooses a `username`. Their Google account ID and `email` (from Google) are stored with it.
- `username` and `email` are both required and must each be unique across all users.
- Only the Google account ID, username and email are stored about a user.
- Email is kept for future notifications; no emails are sent in the MVP.
- Every registered user has `player` capability.
- Global `admin` role can be assigned/revoked only by admins.
- The first admin is the user whose email matches the server's configured initial admin email when they first sign in.
   
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

### Game Discovery

- All games are visible to every signed-in user.
- A player can request to join any game that hasn't started.

 
### Join Request
 
- Player submits join request.
- Skipper approves or rejects.
- If rejected, player can request again later (no cooldown/limit in MVP).
   
### Skipper Direct Add
 
- Skipper finds the player by username and sends them an invite.
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
 
Each pick is resolved as soon as an admin enters a final status (result, postponed or abandoned) for its match.

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
 
- Winner checks run once every fixture in the matchday has a final status, not as each pick is resolved.
- Standard outcome: last remaining active player wins.
- All eliminated together: if every remaining active player is eliminated in the same matchday, they all become joint winners.
- Season end: if the league has no matchdays left and 2 or more players are still active, they all become joint winners.
- Joint winner agreement:
    - If exactly 2 active players remain, either may propose a joint win and the other accepts or declines.
    - If the proposal isn't accepted before the next matchday's pick deadline, it lapses and play continues.
    - A joint win can be proposed again in any later matchday while both players remain active.
   
## 12. Admin Data Features
 
League, matchday, fixture and result data is system-wide: every game that uses a league shares the same data. Only admins can create or change it.

Admin must be able to:
 
- Create/update leagues
- Upload official matchday schedules
- Upload/update fixtures for each matchday
- Upload/update match results and status (including postponed/abandoned)
- Do each of the above through web forms (one item at a time) or CSV upload (in bulk)
   
  These updates drive pick validation and elimination outcomes.

### CSV Formats

CSV files have a header row and are uploaded for one league season. Kickoff times are ISO 8601 in UTC.

- Fixtures: `matchday,kickoff,home_team,away_team`
- Results: `matchday,home_team,away_team,status,home_goals,away_goals`, where `status` is `completed`, `postponed` or `abandoned`, and goals are blank unless `completed`

If any row in an upload is invalid, nothing is imported and the errors are reported.
   
## 13. Functional Acceptance Criteria
 
1. Given a new user signs in with Google for the first time and chooses a unique username, when submitted, then an account is created with their Google account ID, username and email.
2. Given a username already in use, or a Google account whose email is already registered, when the user signs up, then account creation is rejected.
3. Given a returning user signs in with Google, when sign-in completes, then they are signed in to their existing account.
4. Given a user's email matches the configured initial admin email, when their account is created, then they have the admin role.
5. Given a non-admin user, when they try to create or change league, matchday, fixture or result data, then the request is rejected.
6. Given an admin uploads a valid fixtures CSV, when it is processed, then the fixtures are created for that league season.
7. Given an admin uploads a CSV with any invalid row, when it is processed, then nothing is imported and the errors are reported.
8. Given a skipper creates a game, when it is created, then the skipper is a player in that game.
9. Given a signed-in user, when they view the game list, then all games are shown.
10. Given a player requests to join a game, when skipper approves, then membership is active.
11. Given a player is rejected from join request, when they request again later, then request is accepted for processing.
12. Given skipper invites a player by username, when player accepts, then player becomes a member.
13. Given game has fewer than 2 players at its starting matchday's deadline, when the deadline passes, then the game does not start and its start moves to the next matchday.
14. Given game has 2 or more players at its starting matchday's deadline, when the deadline passes, then the game starts and joining closes.
15. Given player attempts to join after game start, when request is made, then request is denied.
16. Given player selects a previously used team in same game, when submitting pick, then pick is rejected.
17. Given deadline passes without player pick, when deadline processing runs, then system auto-picks first alphabetical eligible unused team.
18. Given a player has no eligible team for a matchday, when deadline processing runs, then the player is eliminated.
19. Given an admin enters a final status for a match, when it is saved, then the picks on that match are resolved immediately.
20. Given picked team wins, when results are resolved, then player remains active.
21. Given picked team draws or loses, when results are resolved, then player is eliminated.
22. Given picked match is abandoned, when results are resolved, then player is eliminated.
23. Given picked match is postponed, when results are resolved, then player remains active and the picked team counts as used.
24. Given a matchday's pick deadline has not passed, when a player views picks, then picks are hidden.
25. Given a matchday's pick deadline has passed and auto-picks have been assigned, when a player views picks, then picks are visible.
26. Given a matchday still has fixtures without a final status, when a pick is resolved, then the game is not completed, even if only one active player remains.
27. Given one active player remains once every fixture in the matchday has a final status, when winner checks run, then game is completed with that player as winner.
28. Given every remaining active player is eliminated in the same matchday, when winner checks run, then game is completed with all of them as joint winners.
29. Given the league has no matchdays left and 2 or more players are active, when winner checks run after the final matchday, then game is completed with all of them as joint winners.
30. Given exactly two active players remain and one proposes a joint win, when the other accepts before the next matchday's pick deadline, then game is completed with both as winners.
31. Given a joint-win proposal is not accepted before the next matchday's pick deadline, when the deadline passes, then the proposal lapses and play continues.
32. Given skipper abandons a game, when abandon action is confirmed, then game status becomes abandoned with no winner.
33. Given an eliminated player attempts re-entry to same game, when request is made, then request is rejected.
     
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