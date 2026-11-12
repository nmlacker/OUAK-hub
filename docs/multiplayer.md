# OUAK Multiplayer

Play *Once Upon a KATAMARI* together over the mods' own connection. It needs **OUAKModAPI.dll**, and every player needs the same OUAK Multiplayer version (protocol v16). It never uses the game's online service, matchmaking or lobby: everyone plays their own game, and the mod shares what's needed between them.

Its page is the **Online** tab of the mod window (Right Shift), with its own tabs along the top: **Connect** (or **Session** while you're in one: players, session settings, votes), **Lobby** (the host's rules), **Ranking** (how players are placed), **Ghosts** (how other players show on your screen), **Voice** and **Chat** (your quick chat lines).

It stands alone: it needs only the API, not the OUAK Mod Menu.

It also has a **Katamari Ball** tab (moved here from the Mod Menu, which is single-player only): see [Katamari Ball offline](#katamari-ball-offline).

On screen in a session: the vote popup and the Results screen (standings; Tab or the D-pad for the after-action report) are drawn with Dear ImGui and only exist while they're showing. The player bar, name tags, chat bubbles, the chat box and the start / countdown / Sudden Death messages are drawn with Unity IMGUI, which is cheaper for things that are on screen all the time.

## Joining

* One player clicks **Host**; the others **Join** with the host's IP address and UDP port (default 27886). A password is optional; up to 8 players.
* Over the internet the host needs the UDP port forwarded, or everyone on one LAN or a VPN (Tailscale, ZeroTier). Windows may ask to let the game through the firewall the first time you host.
* **Test on this PC** hosts on 127.0.0.1 with a loopback test bot.

## Who's who: SteamIDs

* Each player is identified by their **Steam account**. Without Steam running, a random id made once per install is used instead.
* With Steam, a joining player's game sends a Steam session ticket. The host's game asks Steam to check it (`ISteamUser::BeginAuthSession`): Steam signed it for that account and for this game.
* The player list shows each player as **Steam verified**, **no Steam** or **test bot**.
* The host keys players by identity, so nobody can be in the session twice:
  * a player who rejoins (new port, router change, restarted game) gets their old place back;
  * an older connection of theirs is told it was replaced;
  * someone claiming a verified player's account without Steam's word for it is turned away;
  * nobody can join as the host's own account.
* Clients never list themselves, or the same person twice, even if a stale copy shows up in the player list.
* Host option: **Only let in players Steam verifies** (test bots always join).
* Your name defaults to your Steam name until you type one.
* Only the host sees full identities. Players' SteamIDs are shared within the session (for other mods: `Multiplayer.Players`).

## The host is in charge

When the host restarts the stage, everyone restarts. When the host goes back to the Plaza, everyone goes back (to the host's plaza, if they own it). In a session, the pause menu's **Retry** and **back to the Plaza** open a choice instead of the game's Yes/No:

| Who | Retry | Back to the Plaza |
| --- | --- | --- |
| Host | **Force** (everyone restarts now, a new run together), **Vote**, **Only me** | **Force** (everyone goes back), **Vote** |
| Everyone else | **Vote**, **Only me** | **Vote**, **Only me** (leave the race alone: disqualified) |

Choose with Up/Down or the D-pad, Enter or A to pick, Esc or B to cancel (back to the pause menu). Mouse clicks work too.

* **Leaving the race alone** (Only me, back to the Plaza) before finishing **disqualifies** that player.
  * Their result is **DQ**, listed last, and everyone sees a chat notice.
  * The race goes on without them, and nothing they do can interrupt it: the host refuses their vote requests (stage picks, random stage, restart, kick...) until the race's results are final.
  * The host's next stage brings them back.
  * A player whose game ends up in a hub some other way mid-race is disqualified the same way.
* **Votes** pass with more than half of all players. A passed vote is carried out like the host's Force.
* **One vote per player per stage**, for each kind (restart, Plaza). Once a restart vote passes, nobody can call another restart vote on that stage. After a vote fails, the next one of that kind can start **30 s** later.
* Restarts and reloads of the same stage keep these limits. Going to another stage, or through a hub, resets them.
* The Online tab shows what you can still call on the current stage. The host enforces the rules and tells a player why a request was refused.
* The host going back to the Plaza any other way (the result screen, a hotkey, the Levels tab) also brings everyone along.
* A Retry from the result screen starts the stage again for everyone.
* Players with "Join the stages the host starts" off aren't pulled along.
* The mod's own sessions (**Host** / **Join** on the Online tab) are only for the single-player stages. Multiplayer maps (Katamari Ball, Race, Target) are never voted on, pulled into, synced or shown as ghosts by them: versus modes are for the game's own networking.

## Shared play

* **Ghosts**: each player's katamari, cousin and rolled-up objects show in the other games:
  * name tags, speech bubbles and a player bar with portraits;
  * the cousin mirrors the player's real animation (emotes, rolling, held poses);
  * ghost display settings: distance, how many to show, parts;
  * a ghost's katamari is drawn at its real size: in story stages the game's core ball is much smaller than the katamari (its rolled-up objects make up the rest), and a ghost only gets the objects that are shared with it, so its core ball is scaled up to the katamari's real radius (it sits on the ground instead of floating at its centre's height);
  * while the game changes scene (its own scene loads, e.g. the ship to a scroll page, or a plaza to a stage), every other player's ghost and cousin is switched off until the new scene has settled (1.5 s), so the scene's own setup never finds another player's character model.
* **Shared pickups** (host setting): what anyone rolls up vanishes for everyone on that map. Turning it off puts the objects back.
* **Stages together**:
  * the host's stage pulls everyone in, held at the start until all are ready, then 3-2-1-GO (race start or strict start);
  * three timers for every player and the moment each one reaches the stage's win condition (see [Timers and ranking](#timers-and-ranking));
  * results and standings are shared, placed by the host's ranking system, with an after-action report;
  * **Sudden Death** (host setting, on by default) only happens at the stage's end, only for the slowest: once the stage has ended for the first player the game's own way (their time ran out, or the goal ended a stage that ends on it; leaving or being cut doesn't count), anyone still playing (they paused, restarted or started late) must reach their next growth size within the time set (30 s by default), again and again, or their run ends (CUT). **Off**, everyone plays the stage out like single-player: a run ends when that player's own time runs out, when the goal ends the stage, or when they leave it (DQ), and the results wait for everyone;
  * match logs go to `UserData/OUAKModMenu/Matches`.
* **Free roam**: walk the plazas and the ship together. Picking a stage from a fan or the Levels tab starts a vote (or "go now" for the host), and everyone gathers at that fan's stage screen first.
* **You in the hubs**: in a plaza you're always the plaza's own walking character (its CharacterManager), asked again every time; right after a plaza loads (after a match especially) the mod waits for it instead of taking another walking character, which could be one of the plaza's cousins standing around (the others then saw "you" at that cousin's spot).
* **Your cousin as you walk**: in the ship and the plazas, others see the cousin you're walking as (the hub's `ModelLoader.PlayerNow`), even when you swapped to a cousin in a plaza without changing the save's choice; a change shows for the others within about 3 s (logged as "Your look changed").
* **Teleport** (player bar): on the ship or in a plaza, point at another player's slot ("click: teleport" shows) and click it, then choose **Teleport** to be put where they stand. If they're in another hub, **Go there** takes you to their plaza or the ship first (a plaza changes your era, like the game's map jump) and puts you next to them once you're both there. Players in a stage can't be teleported to. It moves your walking character the way the game stands you at a fan (`CharacterMove.SetCharacterPosition`).
* **Lobby settings** (host): Sudden Death (on / off, time per growth size), which freebies are allowed, the fly-over vote, ready time, the lost-vote cooldown.
  * The freebie list also has the API's **reimplemented freebies**: Shoes, Crown, Mushroom and their Specials, which do nothing in story stages in the game itself. While you're in a session with others, exactly the ticked ones work, for everyone (the API's `StoryFreebies.SetItem`); "no freebies" turns them all off. Your own setting (another mod may have switched some on or off) comes back when you leave the session. The host's list travels in the existing settings packet, so the protocol is unchanged.
* **Chat** (T) and **voice chat**: push-to-talk (B) or open mic, off by default.
* **Quick chat** for controllers: **D-pad down** opens your quick chat lines at the left of the screen; D-pad up/down choose, **A** sends, **B** (or D-pad left) closes; a mouse click or the keys 1-9 work too, and it closes by itself after 10 s. Up to 9 lines, edited, reordered and reset on the Online tab's **Chat** page (saved under `[OUAKModMenu_Online]` `QuickChatMessages`, `|`-separated). They go out as normal chat messages (the host's rate limit and the speech bubbles apply). While the list is open the game ignores your controls. It doesn't open while the chat box, the mod window, a prompt or the pause menu is open.
* **Votes**: skip the fly-over, play a stage, a random stage, remove a player, restart, back to the Plaza.

## Timers and ranking

Every player's slot on the player bar shows three timers during a shared run, and the Results screen shows them all at the end:
* **IGT** (in-game time): how long they actually played the run, counted frame by frame from the shared GO while the stage is in play. It leaves out the pause menu, the game's one-time tips, the clear demo, game over and a restart's wait. A frozen stage clock (the Timer freebie) doesn't stop it, and a restart of their own keeps adding to it.
* **RTA** (real time attack): real time from the shared GO to the end of their run, pauses and all, on the host's clock.
* **Stage time**: the game's own clock (time left, or time so far on time-attack stages).
* **Win condition**: the moment they meet the stage's goal ("20 food items", "15 cm"...), the game's own `MakeStageCleared`, recorded as IGT, RTA and the stage clock ("win: IGT 0:52.3 · RTA 0:53.0" on the slot; "Win condition (IGT / RTA)" on the Results). Once past it, the slot also shows the rank badge their stage HUD shows and their live place.

**Ranking systems** (the Online tab's **Ranking** page): a ranking system is a title and a list of measures in order. Players are compared on the first measure, and the next one only decides between players equal on everything before it. Players equal on every measure share the place (1, 1, 3). Leaving the race (DQ) always comes last. RTA counts to a tenth of a second (it crosses the network), IGT and the stage clock to a hundredth.
* **Standard** (preset, the default): reached the win condition › stage score › time to the win condition (RTA) › time to the win condition (IGT) › diameter. So a slower player with a better score beats a faster one with a worse score, and the time decides between equal scores, RTA first, IGT when the RTAs are equal to the tenth.
* **Speedrun** (preset): reached the win condition › time to it (RTA) › time to it (IGT). Nothing else counts.
* **Your own**: press **New** to copy the selected system under a new title, then add measures, turn one round (the other end first), move them up and down, or remove them. Up to 16 systems of up to 12 measures, saved as `[OUAKModMenu_Online]` `RankingCustom` (and the chosen one as `RankingSystem`).
* The measures: reached the win condition; **stage score**; the game's rank badge; time to the win condition (RTA, IGT or the stage clock); run time to the end (RTA or IGT); diameter; objects rolled up; freebies used; dashes.
* **Stage score** is the stage's own measure, the number its rating compares with the stage's five targets (`MainGameManager.CheckClearRate`, traced in the game's code): the diameter on size stages; the count of target objects on "N items" stages (`StageMonoManager.currentTypeMono`); the clear time on time stages (lower is better); how close to the asked size or price on **Just Right** stages (`|_targetNumber - _katamariSize|` or `_currentPrice`, lower is better); the value on price stages; the Pharaoh's favour, the watered desert and the ducks' missions on those stages. The stage's rank badge counts first, then the exact number. A few unique stages (the cow and bear, sweetness, cleaning) only have their rank badge.
* The **host's** system is the session's: it travels with the lobby settings, places everyone live and on the Results, and the Results show its title and measures under the table, with each player's stage score, so nobody has to guess how they were ranked. A player who joined sees the host's system on the Ranking page. Match logs record it too.

## Versus online (the game's own networking, mod players only)

Katamari Ball, Race and Target online use the **game's own online** (the ship's terminal: create, search or join a room), not the mod's connection. On PC that is Steam: rooms are Steam lobbies (lobby data `roomName`, `roomParam`, ...), and the match runs over Steam's networking. The mod's own **Host** / **Join** is only for the single-player stages.

With OUAK Multiplayer installed, versus is limited to other OUAK Multiplayer players, through Steam's lobby servers and the game's own join step (no server of the mod's own):
* A room you create gets the Steam lobby data `ouak_mod = ouak-versus-1`, and you mark yourself in any lobby you enter (lobby member data, same key).
* Room searches ask Steam only for lobbies with that mark, so you only find, and random-match into, mod rooms.
* As the host, a player in your lobby without the mark is refused: the game's join check (`MultiplayController.Update` asks the transport's `IsUserBlocked`) says no, and the game sends its own "join denied". Your first join request waits 2 s after entering a lobby, so the host already sees your mark (the game re-sends it every 2 s anyway, up to 10 times).
* If you end up in a room without the mark (for example a friend's invite to a game without the mod), the join fails through the game's own error message.
* Players without the mod can still see mod rooms in their own room list, but can't get in.
* The Online tab's **Versus** page shows the lobby you're in and who in it runs the mod.

## Katamari Ball offline

The **Katamari Ball** tab is the game's Sports Katamari (Katamari Ball) against three CPU princes, the game's own training mode. Nothing here changes online play with other people.

* **Start a match** on any of the seven Sports Katamari maps (Training, Battle, Edo, Gold Rush, Skull Island, Pyramid, Ice Cave), the way the ship's terminal does (`MultiplayGameManager.StartTraining`).
* **Race** and **Target**, the game's unfinished online rounds, played on the API's fixes for them (part of the API, acting only on those maps). In Target the API also fixes the CPU princes: in the game they never roll anything up (no object is set up for them); with the fix they roll up what their katamari touches.
* **Multiplayer Room**, the game's online lobby. On its own it has no room to show, so the button opens the ship's multiplayer menu.
* **King Chances** (on by default, `[OUAKModMenu_Match] KingChances`). Every match the game draws 4 different King Chances out of 6. It has their messages and its tip says the King acts "depending on your time remaining", but it never starts one. With this on, the match's 4 go off on time: one in each quarter of the time limit, evenly spaced, at 1/8, 3/8, 5/8 and 7/8 of it. With no time limit, the map's own limit is used. What each one does is the API's (`KingChances`): new objects, new freebies, a King Chest for every prince, confiscated freebies, the bonus area opened, unlimited dash for 60 s. With the custom rules leaving no freebies, a King Chest or freebie drop becomes an object drop. A second object drop shows the game's own "... Nothing happened!".
* **Target points** (on by default, `[OUAKModMenu_Match] TargetPoints`): in the Target round every origami crane rolled up gives 10 points, for you and the CPU princes (off: 0 points). The cranes are counted by the game's own target counter, and the HUD's points and the results show the points (the API's `TargetRound.PointsPerTarget`).
* **Custom rules** for offline Sports Katamari matches (`[OUAKModMenu_Match]`, the old Match tab settings carry over):
  * Time limit: game default, 1 to 15 minutes, or none (`MainGameManager._startTime`).
  * CPU princes: Normal, Easy and Hard (half or double their catch-up growth, `MultiplaySetting.dicBattleCpuAddCoreSize`), or Idle (`MultiplayCPU.SetAIWaitTime`).
  * Freebies: off (King Boxes and item boxes give nothing), or two lists you edit, through the API's `BattleItemPools`:
    * **King Boxes** (the King's chests): which rewards they may give. The game's own are a Buddy, Washtub, Mushroom, Magnet, Rocket or a Special one; Hand, Spiderweb, Bubble, Shoes and Crown can be added, and so can **Smoke, Hoop, Dice and Special Dice**, which have no King Box reward of their own: the API hands them straight to your freebie slot with the roulette (`BattleItemPools.SetKingBox(rewards, freebies)`), and a CPU prince that opens such a box gets one of the ticked rewards instead (or nothing). Buttons: Game's own, All, None.
    * **Item boxes**: which freebies they may give. The game's own are Washtub, Spiderweb, Bubble, Mushroom, Hand, Shoes, Crown and Specials; Magnet, Rocket, Smoke, Hoop and both Dice can be added.
    * A reward or freebie outside a list is swapped for one in it; an empty list gives nothing (a King Box is still taken off the map). The results' buddy count follows what was really given. Sunglasses has no battle code, so it can't be in either list; Ice axe, Sonar and Timer do nothing in battles.
    * The lists are saved as `KingBoxPool` / `ItemBoxPool` (`none` = nothing) and `KingBoxFreebies` (the King Box freebies). The first time, the first two are made from the old per-freebie list (`DisabledFreebies`).

Test: `UserData/OUAKMultiplayer/trainingtest.flag` holding internal map names (e.g. `Stage_ONBattle1`, `Stage_ONTarget1`) runs a match test, one map per launch: backup, the last-used slot, the map, PASS once the round runs, the time cut short, results, quit. Options in the same file:
* `kcquarter`: the round is cut to 70 s and the four timed King Chances are measured (when fired against when due, when shown, the message label, a picture each).
* `kcall`: fires all six types in turn and logs what each changed.
* `kingchance`: the King Box chain (counts, rewards, displays).
* `rules:off`, `rules:some`, `rules:none` or `rules:kbfree`: the custom rules for that launch only, never saved (every freebie off; King Boxes only Washtub or Rocket and item boxes only Crown; custom rules off; King Boxes only Smoke, Hoop and both Dice). With `kingchance` your prince rolls up a King Box, and every freebie a King Box gives is logged (`a King Box gave ... to you / a CPU`).

`Stage_ONTarget1` also surveys the map, rolls up a crane and logs everyone's points and crane count. Look for `[TrainingTest]`, `[KingChance]` and `[TargetProbe]` in `MelonLoader/Latest.log`.

## Reporting problems

Every session is logged by the API (on by default; off on the API's Mods page or with `[OUAKModAPI] SessionLogs = false`): `UserData/OUAKModAPI/Logs/Sessions/<date_time>/`, with `all.log`, `errors.log`, `OUAK_Multiplayer.log` (this mod's lines, sorted by tag, e.g. `[OUAK Multiplayer/Online]`) and `Game.log` (the game's own errors, scene loads, crashes). A session that crashed is marked so at the next start. After a playtest with a problem, that folder from each player's PC tells what happened.

## For other mods

`OUAKModAPI.Multiplayer` is implemented here:
* `InSession`, `IsHost`, `Players`;
* `Send(channel, data)` / `Received`: your mod's own messages, relayed by the host (`Msg.ModData`).

## Protocol v16 (what changed from v15)

* `State`: the player's IGT, IGT / RTA / stage clock at the win condition (-1 not yet), the HUD's rank badge, the stage's rating and the stage score (kind and number).
* `Result` and `Standings`: IGT, IGT and RTA at the win condition, the stage score (kind and number).
* `Settings`: the host's ranking system (title, then each measure: key, high bit = turned round), after the lobby settings.

## Protocol v15 (what changed from v14)

* `Hello`: the player's identity (kind + SteamID or install id) and Steam session ticket.
* `Roster`: every player's identity and verified flag.
* `VoteDenied`: carries the reason.
* New messages: `HostPlaza` (29, the host sends everyone to its plaza) and `ModData` (30).
* `VoteKind` gains `Restart` (5) and `ReturnToPlaza` (6).

## Test

`UserData/OUAKModMenu/onlinetest.flag` containing `run Stage_Big1` runs the full online test with the loopback bot, then quits. It needs a save; it loads the last-used slot after backing up the saves. Look for `[OnlineTest]` in `MelonLoader/Latest.log`.

It covers:
* the roster and run handshakes (including a stale copy of you under another id);
* ship, plaza and fan-pick flows, chat, quick chat (the list shown with your controls held, line 1 sent and answered), bubbles, voice (made-up voices, no microphone);
* teleport: in the plaza, the player-bar prompt for the bot and its Teleport button, then your distance to the bot's spot;
* back in the plaza after the match: you're the plaza's own walking character and the bot's cousin shows;
* the scene change guard (ghosts switched off on the way from the ship to the plaza), a cousin swap in the plaza (you walk as another cousin: your look changes, the bot copying you is rebuilt as it, then back), and the bot's ghost katamari drawn at its real size;
* the shared start, race checks, results, the sharing toggle and the vote cooldown;
* the run timers: IGT counting and never ahead of RTA, then the katamari grown past the stage's goal and the win-condition moment recorded (IGT, RTA, the stage clock equal to the game's Size Time) and mirrored by the bot; a picture of the player bar (`hud_timers`);
* ranking: live places by Standard (both reached the goal, so the bigger katamari is 1st; the bot sends its own size and the rating it earns), a custom system sent through the Settings bytes and read back, and with it turned round (biggest last) the smaller one first; the Ranking page drawn (`menu_ranking`);
* host authority: the Force / Vote / Only me choice, the vote rules, a passed restart vote, a refused second one, Force restart, Force back to the Plaza.
