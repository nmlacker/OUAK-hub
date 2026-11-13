# OUAK Dash Bar

A MelonLoader mod for *Once Upon a KATAMARI* that shows how much **Prince Dash** you have left, on the game's own
dash bar, which the game never shows. It also counts down how long it is until the dash starts coming back.

* **Story stages and multiplayer modes:** Katamari Ball battles, the Race and Target modes, and versus online. The OUAK Multiplayer mod's own online sessions play story stages, so they're covered too. Never on the ship, the Plaza or in menus.
* **The game's own bar:** the dash gauge at the bottom centre of the screen, with its dash icon. It's grey and yellow normally, and turns rainbow while you have unlimited dash.
* **Always on top:** it's drawn on an overlay canvas of its own, after the 3D scene and the game's other HUD layers. The bar's frame art is semi-transparent, so busy scenery still shows *through* an empty bar.
* **Out of the way:** the bottom centre is shared with the game's stage-goal counter (e.g. "13 / 100"), its freebie slot and the prince's picture, and in an OUAK Multiplayer session with the player bar. The bar uses one of three places: the game's own; straight up, above a player bar or the goal counter; or beside the freebie slot, smaller if needed. It doesn't move for pop-ups that come and go.
* **Always on in a stage:** in a story stage it shows the whole time you're in it, also while the King speaks (his opening talk and his remarks). It hides only in the pause menu, in cutscenes and once the stage has ended. In a match it shows only while the round runs: not during the countdown before it, nor after the round ends.

It works on its own: it needs only the **OUAK Modding API** (`OUAKModAPI.dll`), not the Mod Menu or OUAK Multiplayer. Install `OUAKDashBar.dll` to `Mods/`.

## What it shows

![The bar in each state, from the in-game test's frame captures](https://raw.githubusercontent.com/nmlacker/OUAK-Mods/main/OUAKDashBar/docs/dashbar_states.png)

![The bar in a Katamari Ball battle, Race and Target](https://raw.githubusercontent.com/nmlacker/OUAK-Mods/main/OUAKDashBar/docs/dashbar_multiplayer.png)

| What happens | The bar | The countdown (right of the bar) |
| --- | --- | --- |
| Full stamina | Full, yellow | none |
| Dashing | Goes down as the dash uses stamina | Seconds until the dash ends, which is when the stamina starts coming back. If this dash will use the stamina up, it counts down to the end of the 8 s wait instead. |
| Regenerating | Fills back up slowly | none: it's already coming back |
| Out of dash | Empty | Red, counts down the 8 s wait. Then the stamina is full again all at once. The dash icon fills up from the bottom over the wait. |
| Unlimited dash: Shoes, or in battles the Unlimited dash King Chance | Full, in the game's rainbow look: with the OUAK Modding API, unlimited dash is free (no start cost, no drain). | Seconds of unlimited dash left |

## How the game's dash works

Traced in the game code (`MainGameCore.ExecuteOujiDashTimerSingle`, `ExecuteOujiDash`) and measured in game:

* You have **3 s** of dash stamina (`MaxOujiDashTime`).
* Every dash start costs **0.5 s** (`AddTimePerDashActive`): a full dash, and also the short push you get from a single L2 tap.
* A full dash runs **1.5 s** and uses stamina for as long as it runs.
* When a dash isn't running, stamina comes back at **0.1 s per second**, starting on the first frame after the dash ends. There is no delay before it starts. From empty that takes 30 s.
* If you use more than all of it, you're **out of dash**. You can't dash for **8 s** (`OUJI_DASH_EXHAUST_TIME`), then the stamina is full again all at once.
* **Battles use the same rules**, measured in a Katamari Ball battle: max 3, start cost 0.5, the same 0.1 per second regeneration and 8 s wait. `CoreMultiMove` runs the same `ExecuteOujiDashTimerSingle`. `CanOujiDash` also checks the unused cool timer there, which nothing ever sets above 0. The battle core's `MultiDashCoolTime` is 0.001, against 15 in story stages.
* **Unlimited dash in battles** (Shoes, and the Unlimited dash King Chance) is the prince's own flag, which the OUAK Modding API's `BattleDash` patch makes work. The time left is `UserStatus.infinityDashTime`, which the match's host counts down: 5 s for Shoes, 60 s for the King Chance.
* **Shoes** (the OUAK Modding API makes them work in story stages) refill the stamina and give 5 s of unlimited dash.
  * Since the API build of 2026-11-09, unlimited dash in story stages and battles is **free and nonstop**: a dash start costs nothing, the stamina stays full, and a running dash goes on until you brake, hit a wall or start climbing, or the unlimited dash ends. The mod reads it through `BattleDash.Unlimited(core)` and shows it in the rainbow "Max" look the game's battle code uses for unlimited dash (`ResetOujiDashMultiCoolTime`).
  * The game's own unlimited dash (its debug switch `GlobalManager._debugInifiniteDash`, which a mod's cheat may turn on) only stops the drain while dashing: each start still costs 0.5 and a dash still ends after 1.5 s. The bar shows that too, in the rainbow look, with the real stamina.

## The game's unused dash bar

`MainGameUIManager.MultiDashUI` is an `App.KatamariSin.UIMultiDashGauge`, the object `MainGameUICanvas/LAYER00/MultiDash`. It's in every story stage's HUD, switched off, and no game code ever switches it on. Its parts:

* **Base:** frame `DashGauge`, bar `Gauge` (`DashGaugeBar`, a yellow horizontal fill), icon `DashIcon` (`DashIconMask`, a vertical fill).
* **BaseMax:** the same parts in the rainbow look (`GaugeMax`, `DashIconMax`, both white).

The battle HUD has the same object, also switched off: measured in a Katamari Ball battle, it stays off. `SetGaugeValue(v)` sets both bars' fill, and `SetDisplayMax(b)` swaps between the normal and rainbow looks. Only the battle code's dash cool-time functions call them:
* `ExecuteOujiDashTimerMulti`, which nothing calls;
* `ResetOujiDashMultiCoolTime`, which sets the bar full in the Max look for battle Shoes and the unlimited-dash King Chance;
* `ForceOujiDashMultiCoolTime`.

So it was a bar for a dash that recharged after a cool time (`MultiDashCoolTime`, 15 s) and was dropped. This mod switches the object on and drives it with the game's own methods. The countdown is a TextMeshPro label in the stage clock's font.

In battles the game's code still writes to the gauge when unlimited dash starts and ends: full in the Max look, then empty. The mod compares the gauge's real state with the stamina every frame, so the bar is right again on the next frame (measured).

**Always on top:** the HUD canvas `LAYER00` is drawn in camera space by the game's `UICamera`.
* The mod moves the gauge to an overlay canvas of its own, `OUAKDashBar_OnTop`. It sits under `MainGameUICanvas`, so it goes away with the stage's HUD.
* That canvas uses the HUD's own scaler settings, kept in sync, so the bar stays exactly where the game put it but is drawn over the 3D scene.
* Its sort order is −100, so other overlay screens stay above it.
* The King's messages are on the overlay canvas `Canvas(Clone)` (sort order 100), above the bar's (−100), so his window covers the bar, not the other way round. Measured: his text at (1280, 737) and face at (828, 787) on a 2560 × 1440 screen, far above the bar.
* The gauge goes back into `LAYER00` when the mod lets go of it.

**Out of the way:** every frame it's shown, the bar keeps clear of:
* the game's own HUD at the bottom of the screen: whatever each HUD piece (each child of `LAYER00` / `LAYER01`) actually draws there, the union of its visible images and texts, rescanned 10 times a second. Examples: the freebie slot (`ItemCanvas`), the stage-goal counter (`ClearTargetUI`), the sub-challenge counter;
* the prince's picture at the bottom right (`MainGameUIManager.OujiBackLoader._myRect`, from 28% across it: measured, that's where the prince and his backdrop start);
* other mods' HUD, which they report through the OUAK Modding API's `HudSpace`. OUAK Multiplayer claims its player bar as `OUAK Multiplayer/PlayerBar`, every frame it's drawn.

It skips:
* pieces off screen: in a stage without a count goal, the goal counter is parked just below the screen;
* pop-ups that come and go while you roll: the last-caught object's name (`MonoNameDisplay`), the song title (`BGM`), and the markers pointing at other princes (`SonarGuide`, which sit at the screen edge whenever one is behind you; measured in battles and Target);
* pieces still moving (they count after holding still for 0.6 s): measured, the freebie slot slides up from below the screen through the bar's place when you get a freebie.

It uses one of three places, the first that's free, keeping 8 px (at 1440p) clear of everything:
1. the game's own place (bottom centre);
2. straight up, just above whatever is at or under it (a player bar, the goal counter);
3. beside the freebie slot, on its right at the slot's height. It's drawn smaller (down to 55%) if that's what it takes to fit before the prince's picture.

If none is free, it uses whichever covers the least. When its place gets taken it moves at once; it moves back to an earlier place only once that place has stayed free for 3 frames and 0.5 s, so it doesn't jump around. Only the mod's own overlay canvas moves and scales; the game's HUD canvas is never touched.
* The wait also covers a claim that lapses for a frame. A claim lapses 0.25 s after its owner last renewed it, so one long frame can make it look gone. Measured: saving a frame capture took 0.45–0.55 s, and the bar dipped and came back before the wait was added.
* Only the mod's own overlay canvas moves; the game's HUD canvas is never touched.
* With an older API that has no `HudSpace`, it still keeps clear of the game's own counter and freebie slot.

## Settings

In the mod window (**Right Shift**), on the **Dash Bar** tab, also saved in `UserData/MelonPreferences.cfg` under `[OUAKDashBar]`:
* show the bar;
* also show it in multiplayer modes (on by default);
* show the countdown;
* count down during a dash;
* hide the bar while the stamina is full.

The tab also shows the live state.

## Building

```
dotnet build OUAKDashBar/OUAKDashBar.csproj -c Release -p:BuildProjectReferences=false
```

The build installs to `Mods/`, or doesn't install while the game is running.

## Test

`UserData/OUAKDashBar/dashtest.flag` runs the in-game test once, then quits the game. It:
1. loads the last-used slot, with every save write blocked while it runs;
2. plays As Large As Possible 1, dashing with real presses injected into the game's pad buffer: the button that does the dash (L2) job, after the API's controller rebinding (`PadBindings.ButtonFor`);
3. checks each state and takes frame captures to `UserData/OUAKDashBar/TestShots`;
4. counts pixels in its own pictures, to prove the bar and the countdown are actually drawn;
5. writes per-frame values to `UserData/OUAKDashBar/dashtest.csv` and `[DashBarTest] OK/FAIL` lines to the log.

Flag words:
* `watch`: press nothing, only watch another test, and check the bar against other mods' real HUD claims (pictures `watch_N`);
* `probe`: only dump the game's gauge and show it at a few values, with the mod's own drawing paused;
* where: `battle` (Katamari Ball, Battle (Jurassic), against three CPU princes), `race`, `target` (the API's modes), or none for the story stage;
* `slot N`.

In a match the test makes the round long enough for its checks, then ends it. Battles add three checks:
* pausing with the real Start button;
* the game's own writes to the gauge;
* the round's end and the results screen.

### Measured with it

Last runs (2026-11-12, with the layout around the game's HUD and the bar always on in stages), each passing all its checks:

| Where | Checks |
| --- | --- |
| Story stage: As Large As Possible 1, 5 cm (the player-bar, crowded-bottom and King checks included) | 49 |
| Katamari Ball battle: Battle (Jurassic), against three CPU princes | 36 |
| Race | 19 |
| Target | 19 |

Race and Target run a shorter list: the bar on screen and on top, a dash, regeneration, the every-frame checks, and the round's end. Their start areas are cramped: every dash there ran into a wall within 0.5–0.6 s, so the test samples the dash earlier.

| What | Result |
| --- | --- |
| The dash timer (`ExecuteOujiDashTimerSingle`) | Runs once per frame, with `Game.GAME_DELTATIME` equal to `Time.deltaTime` on every frame (over 7,000 frames), so the game's dash seconds are game-time seconds. |
| Max, start cost | 3.0, 0.5. A full dash starts at 0.5 used. |
| The bar | Equals 1 − used / 3 to within 0.0005 on every frame it was on screen (15,171 frames in the last run). |
| Countdown while dashing | Equals the game's dash timer, rounded to 0.1. |
| Regeneration | 0.1000 per second over 3 s, starting the frame after the dash ends (1.527 → 1.526). |
| Out of dash | The wait was 8.000–8.006 s in every run, then the stamina was full at once. |
| Countdown while out of dash | Equals 8 − the time since running out to within 0.001 s on all 1,267 frames of one wait. Sampled against the clock each second: off by 0.008–0.042 s at most. |
| Prediction during the dash that ran out | Shown 0.15 s into that dash: "8.616", "8.462", "8.453", "8.885", "8.888". The stamina came back 8.613, 8.459, 8.454, 8.880 and 8.888 s later. |
| The game's dash bug | Hit by chance in four runs: a full dash started at 2.80–2.85 used jumps over the max at its start (3.30–3.35). The bar showed it as out of dash, with the countdown following the game's own wait timer: within 0.008 s of the clock, refill after 8.006 s. |
| The countdown is drawn | Its box is right of the bar, at (1645, 73) at 2560 × 1440. Pixels in that box: white 1,467 while dashing ("0.8") and 1,061 under Shoes ("4.3"); red 1,267 while out of dash ("3.6"); none with no countdown. |
| Shoes | Rainbow look, Shoes' seconds left. With the API's free unlimited dash the stamina stayed at 0 used through a dash start and the dash (bar full). Back to the normal look when they end. |
| Pause menu | The bar hides with the HUD (`PauseObjs.isOpend`) and comes back on resume. In a battle the real Start button opens the same pause menu, and the bar hides there too. |
| Battles: the game's gauge | Switched off; the game didn't turn it on in the first 4 s of a round. The player's core is `MainGameManager.GetCore()`: core 0, not a CPU. |
| Battles: same dash rules | Max 3, start cost 0.5, regeneration 0.1000 per second, an 8.009 s wait; the countdown shown during the dash that ran out was off by 0.000 s. |
| Battles: Shoes | The API's `BattleDash` flag on, 4.96 s left (`UserStatus.infinityDashTime`). The countdown shows it; rainbow look; the stamina stayed at 0 used through a dash start and the dash (bar full). |
| Battles: the game writes to the gauge | After the game's own `ResetOujiDashMultiCoolTime` (bar full, Max look) or `ForceOujiDashMultiCoolTime` (bar empty), the bar shows the real stamina again on the next frame. |
| Battles: round end | Hidden at `RoundFinishState`. On the results the game has destroyed the HUD, so the mod lets go of the gauge; no bar in the picture. |
| Every frame (all four modes) | Bar within 0.0005 of the stamina; a countdown on every frame of a dash or out-of-dash wait; the rainbow look on exactly the unlimited-dash frames. |

Not measured:
* the story stages' opening, cutscenes and results screen, which hide the bar through the same checks (`DemoObjs`, `Katamari.Playing`, `StartMessageDone`);
* other stages and battle maps, which use the same code; only the maps named above were tested;
* **versus online** (the game's own online rooms): it can't be tested here, because the user's rules forbid the game's official online service. It runs the same code (`IsMultiplay`, your own core), so it should behave like the offline battle. That is inferred, not measured;
* the Unlimited dash King Chance in battles: it uses the same flag and time as Shoes (60 s), not run in the tests;
* hiding while `MultiplayGameManager.isGamePause` is set, which was never seen set.

**Out of other mods' way**, measured in the story stage (2560 × 1440). The test claimed stand-ins with the player bar's exact layout and drew them in blue; they match the real claim, below.

| Claimed space | Its top | The bar's frame bottom | Bar pixels inside the claim |
| --- | --- | --- | --- |
| 2 players, one row | 125.3 px up | 133.3 px (moved up 68.3 px) | 0 |
| 8 players, two rows of four | 240 px up | 248 px (moved up 182.9 px) | 0 |
| 8 players, one row | 125.3 px up | 133.3 px (moved up 68.3 px) | 0 |
| A space beside the bar (not under it) | 380 px up | 65.1 px (not moved) | 0 |
| Released | none | 65.1 px, the game's own place | |

The countdown stayed beside the frame in every case, and the bar was drawn in full (15,667–15,703 bar-yellow pixels).

**The real player bar:** in OUAK Multiplayer's online test (2 players: you and the loopback bot), the player bar's claim was (977.33, 1314.67) 605.33 × 106.67, the same rectangle as the 2-player stand-in.
* The Dash Bar's frame bottom was at 132.9–133.3 px in 3 pictures, 8 px above the player bar.
* The bar was drawn in full: 19,162–21,656 bar-yellow pixels in its frame.
* There were 0 dips over the whole test: it never dropped home and came back while the player bar stayed.

This check is the test's `watch` flag word: it presses nothing, it only watches another test. Here it ran alongside OUAK Multiplayer's online test (`UserData/OUAKModMenu/onlinetest.flag` with `run Stage_Big1`).

![The Dash Bar above the real player bar, and above a two-row stand-in](https://raw.githubusercontent.com/nmlacker/OUAK-Mods/main/OUAKDashBar/docs/dashbar_playerbar.png)

![A crowded bottom centre: the goal counter, the freebie slot in use and a 1-player bar; the dash bar moves beside the slot](https://raw.githubusercontent.com/nmlacker/OUAK-Mods/main/OUAKDashBar/docs/dashbar_layout.png)

![The King speaking mid-stage: the dash bar stays](https://raw.githubusercontent.com/nmlacker/OUAK-Mods/main/OUAKDashBar/docs/dashbar_king.png)

**Always on top**, measured:
* The bar is on the overlay canvas at exactly the screen place the game's own canvas gives it: frame at (938.67, 65.07), 682.67 × 136.53 px at 2560 × 1440, before and after the move.
* A test object with the prince's own material, 0.05 in front of the camera over the bar's place, leaves the bar whole: 15,914 bar-yellow pixels with the object, 15,918 without.
* For comparison, back in the game's canvas the bar wasn't covered either (15,913): the HUD's `UICamera` is a separate camera 6 units away. One earlier picture looked as if the katamari covered the bar. It was the empty, semi-transparent bar with the scenery showing through it.
* The bar no longer hides during the game's message windows (the user's choice, 2026-11-12).
