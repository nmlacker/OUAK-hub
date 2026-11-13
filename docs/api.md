# OUAK mods and the OUAK Modding API

MelonLoader 0.7 (IL2CPP) mods for *Once Upon a KATAMARI*, as separate mods. The released ones come together as the **OUAK Modpack** (`OUAK_Mods.zip` of each [release](https://github.com/nmlacker/OUAK-Mods/releases)), and each also as its own zip; the website lists them: https://nmlacker.github.io/OUAK-hub/mods.html

| Mod | DLL (in `Mods/`) | What it is |
| --- | --- | --- |
| **OUAK Modding API** | `OUAKModAPI.dll` + `UserLibs/ImGui.NET.dll` + `UserLibs/cimgui.dll` | The shared base every mod below uses, and that your own mods can use: the mod window and windows of your own (Dear ImGui), hotkeys, game events, stages and game modes, saves and backups, screenshots, logging, asset replacement and hot mods. It is also a **patch for the game**: it fixes broken or unused parts (Race and Target modes, Target points and CPU princes, King Chances, Shoes / Crown / Mushroom in story stages, the Random freebie) so mods can use them. Each fix only acts where it belongs, which only happens through mods; on its own the API changes nothing in the game except the mouse-pointer fix. **Required by the other mods.** |
| **OUAK Mod Menu** | `OUAKModMenu.dll` | Single-player cheats and tools for speedrunning, TASing and messing around: freebies (with the API's story-freebie patches), the stage clock, stages and modes, hotkeys, unlocks, DLC locking, save backups. See [OUAKModMenu/README.md](OUAKModMenu/README.md). |
| **OUAK Multiplayer** | `OUAKMultiplayer.dll` | Online play over the mods' own connection, with the host in charge of restarts and returns to the Plaza, and the **Katamari Ball** tab: offline matches against CPU princes with custom rules, King Chances, Race and Target (with the API's patches). See [OUAKMultiplayer/README.md](OUAKMultiplayer/README.md). |
| **OUAK Dash Bar** | `OUAKDashBar.dll` | Shows how much Prince Dash you have left on the game's own unused dash gauge, and counts down until it comes back; rainbow during unlimited dash. In the modpack since 0.1.1. See [OUAKDashBar/README.md](OUAKDashBar/README.md). |

Install the API plus any of the mods: each mod needs only the API, not the others. Every player in an online session needs the same OUAK Multiplayer protocol (v16). OUAK Song Selector (`OUAKSongSelector/`) isn't released yet.

Press **Right Shift** in game for the mod window (rebind it in the Keys tab). Every installed mod adds its tabs to this one window, listed down its left side. It's drawn with Dear ImGui: drag the title bar to move it, the corner to resize it; a controller moves through it (D-pad or left stick, A presses, B backs out) while it has the game's controls. The API's own tab, **Mods**, lists:
* the installed mods and hot mods (reload / unload);
* asset replacement (reload, list names, export textures);
* the game patches and whether each is in place;
* the game modes, each with a Start button;
* screenshots, Steam status and the window settings.

## Building

```
dotnet build OUAK.sln -c Release
```

Each project copies its DLL to `Mods/`; a running game keeps its DLLs locked, so close it first. Shared settings and references live in `Directory.Build.props`:
* `-p:GameDir=...` builds against another game folder;
* `-p:NoInstall=true` builds without copying.

Every project references ImGui.NET 1.91.6.1 (NuGet); the API's build copies `ImGui.NET.dll` and the native `cimgui.dll` to `UserLibs/`, where MelonLoader loads them. `Samples/ImGuiTest` is a test window of its own (`ModWindows`, F8) with ImGui's full demo; `UserData/OUAKModAPI/imguitest.flag` runs its self-test.

The samples (`Samples/ExampleMod`, `Samples/ExampleHotMod`) don't install by default. Build one with `-p:NoInstall=false` to install it: ExampleMod goes to `Mods/`, ExampleHotMod to the hot mod folder.

---

# Writing a mod with the OUAK Modding API

Start from `Samples/ExampleMod`:
1. Copy the folder.
2. Rename the project and the `MelonInfo`.
3. Build.

Your mod is an ordinary MelonLoader mod that references `OUAKModAPI.dll`. It loads after the API because of the reference plus `[assembly: MelonAdditionalDependencies("OUAKModAPI")]`.

```csharp
using ImGuiNET;
using OUAKModAPI;

public sealed class MyMod : MelonMod
{
    static readonly ModLog Log = ModLog.For("My Mod");

    public override void OnInitializeMelon()
    {
        var cat = ApiCore.Category("MyMod", "My Mod");                       // MelonPreferences category
        ModMenu.AddTab("mymod", "My Mod", DrawPage);                         // a page in the mod window
        Hotkeys.Register(cat, "Boost", "Give a random freebie", "Ctrl+B",
            () => StageControl.GiveFreebie(Il2CppApp.KatamariSin.eInstageItemType.RandomItem), "My Mod");
        GameEvents.StagePlaying += gm => Log.Info($"Go! Size {Katamari.SizeText}");
    }

    void DrawPage()                                                          // plain Dear ImGui, every frame
    {
        ImUi.Title("My Mod");
        ImGui.Text($"Size: {Katamari.SizeText}");
        if (ImUi.Button("Finish the stage", Katamari.Exists))
            ModTasks.Run(StageControl.FinishStage);                          // game actions run from Update, not OnGUI
    }
}
```

### Windows of your own

The mod window is one shared window; a mod can also open its own Dear ImGui windows with `ModWindows.Add`. Your callback makes the `ImGui.Begin` / `ImGui.End` calls, so it picks the title, size and flags:

```csharp
ModWindow win = null;
win = ModWindows.Add("mymod.stats", () =>
{
    if (ImGui.Begin("My stats", ref win.Open)) ImGui.Text($"Size: {Katamari.SizeText}");
    ImGui.End();
});
win.Open = true;
```

* A window with `TakesControl` (the default) shows the mouse pointer, locks the game's controls and takes the controller while it's open, like the mod window. Turn it off for a HUD panel (draw it with `ImGuiWindowFlags.NoInputs`).
* All windows share one ImGui context: the same font, colours (`ImTheme`) and scale (`ImUi.Scale`, the window scale setting).
* `ImUi` has a few helpers in that look: `Title`, `Header`, `Hint`, `Button` / `Primary` / `Danger` (with an enabled flag), `Buttons` (a row), `Segments`, `ChoiceGrid` (a grid of choices with icons), `Image` and `TextureId` for Unity textures, `Field`, `Input` (a text box bound to a `TextField`), `Tip`.
* A callback that throws is closed and logged; ImGui closes whatever it left open. This build of cimgui keeps its asserts: misusing the ImGui API (for example popping more than you pushed) stops the game on a message box, so keep pushes and pops balanced.
* Nothing ImGui runs while no ImGui window is up.

## What the API offers

Everything is in the `OUAKModAPI` namespace, and the XML docs ship with the DLL.

| Area | Classes | What for |
| --- | --- | --- |
| Running things | `ModTasks` | Queue game actions for the next Update (`Run`, `RunWhen`, `Later`), the status line, on-screen `Toast`s. Actions return the message to show; throwing `InvalidOperationException` shows a "can't do that now" message. |
| Logging | `ModLog`, `SessionLog` | `ModLog.For("My Mod")` (levels `Debug` / `Info` / `Warn` / `Error` / `Exception`; `.In("Net")` for a category: "[Net] ..." in the console, `[My Mod/Net]` in the session logs): to the MelonLoader console, `UserData/OUAKModAPI/Logs/<name>.log` and the session logs. `Debug` lines show in the console when "Show mods' debug messages" is on. **Session logs** (on by default; users turn them off on the API's page of the mod window, or `[OUAKModAPI] SessionLogs = false` in MelonPreferences.cfg; `SessionLogsKeep`, default 30 sessions): every game session gets `UserData/OUAKModAPI/Logs/Sessions/<yyyy-MM-dd_HH-mm-ss>/` with `all.log` (everything in order), `errors.log` (warnings, errors, crashes) and one file per source: every mod's lines (also plain `MelonLogger` ones; a leading "[Tag] " becomes the category), MelonLoader's, the game's own errors and exceptions (`Game/Unity ...`), scene loads (`Game/Scene`), game events (`Game/Events`: stage loaded / playing / ended / left, plaza, ship) and unhandled .NET exceptions (`Game/Crash`). Lines are flushed at once; a session that didn't end normally is marked in its `all.log` at the next start; a warning / error repeated more than 20x is only counted. `SessionLog.Write(source, category, level, text)` writes one line directly. MelonLoader's own Logs folder isn't reliable: it deletes by file name and its names aren't zero-padded, so it keeps old logs and deletes new ones. |
| Game state | `GameRefs`, `Katamari` | The game's managers (`GameRefs.Main`, `Save`, `Global`...), `InStage`, `InGameSession`, `IsMultiplay`, DLC ownership. The katamari: size (tenths of a mm, `SizeText`), position, radius, clock (`Elapsed`, `TimeLeft`, settable `TimeLimit`), freebie, `Playing`, `Cleared`. |
| Events | `GameEvents` | `Update`, `LateUpdate`, `SceneLoaded`, `StageLoaded`, `StagePlaying`, `StageRestarted`, `StageEnded(cleared)`, `StageLeft`, `PlazaEntered`, `ShipEntered`, `PauseChanged`, `Quit`. A handler that throws is logged once and never stops the others. |
| Stages | `StageCatalog`, `SceneLoader`, `StageControl` | Every map with the game's own names (`StageCatalog.Entries`). `SceneLoader` loads them the way the game does: stages, multiplayer maps, plazas, modes. `SceneLoader.Interceptors` lets a mod take a load over (the online session uses it for votes). `StageControl` gives and clears freebies, completes the clock, finishes, restarts or leaves the stage, and holds the clock (`HoldClock`). |
| Freebies | `FreebieRules` | Which freebies actually do something, and where (measured in game). Story stages: Drink, Magnet, Pickel, Sonar, Timer, their Specials (same effect, twice as long; Special Magnet also pulls from further) and Random. Battles: the game's battle table (Tarai, Spider, Bubble, Mushroom, Hand, Shoes, Crown, their Specials, Magnet, Drink as the Rocket) and Random. Hoop, Smoke, Dice, Special Dice and Sunglasses do nothing in the game itself; the API's `UnusedFreebies` makes Smoke and Hoop work in battles and both Dice everywhere (Sunglasses stay as they are). `StageControl.GiveFreebie` refuses one that does nothing in the current stage. A mod that gives a freebie an effect calls `FreebieRules.Register`. |
| Pause menu | `PauseMenu` | Take over the pause menu's Retry / back-to-Plaza (`Interceptors`), then carry it out the game's way (`Confirm`) or `Resume`. |
| Game modes | `GameMode`, `GameModes` | Your own way to play: `Start()` loads a map, then lifecycle callbacks run while it's active. Built in: **Race** and **Target** (`GameModes.RaceMode` / `TargetMode`), the game's unfinished online rounds against CPU princes, made playable by the API's patches. |
| Patches | `GamePatches`, `KingChances`, `TargetRound`, `StoryFreebies`, `RandomFreebie`, `BattleItemPools`, `UnusedFreebies`, `ItemBoxes` | The fixed features of the game, ready to use: King Chances, the Target mode's points and crane counter, battle freebies in story stages, King Box and item box pools, Smoke / Hoop / Dice, story freebie boxes. See "Patches for broken and unused features" below. |
| Freebie models | `FreebieModels` | The one standard way to show a freebie in 3D: `Create(item, parent, size)` uses a model a mod registered (`Register`), else the game's own (story box models, loaded game meshes), else a card with the freebie's HUD icon. Item boxes use it. |
| Input | `Hotkeys`, `KeyBinding`, `GameInput`, `Pad`, `TextInput` / `TextField` | Rebindable hotkeys saved in MelonPreferences (listed in the Keys tab). `GameInput.BlockThisFrame` locks the game's controls while your UI is up. `Pad` reads the gamepad directly. `TextInput` is a text box for IMGUI overlays (GUI.TextField is stripped from this game); `IsTyping` is also true while an ImGui text box has the keyboard. `CursorControl`: nothing moves the desktop mouse pointer any more (the game used to park it in the bottom-left corner); it shows while you use the mouse and hides on controller input or after 3 s without mouse input (`[OUAKModAPI] ControlMousePointer` turns this off). |
| UI | `ModMenu`, `ModWindows`, `ImUi`, `ImTheme`, `Prompt`, `ConfirmDialog`, `ModGui`, `Theme` | Dear ImGui (ImGui.NET): tabs in the shared mod window, windows of your own, small helpers and the shared colours. `Prompt` is a choice popup for keyboard, controller and mouse. `ConfirmDialog` is the "I understand" gate that every save-changing action must use. `ModGui` overlays and `Theme` styles are for plain IMGUI drawing over the game (name tags, HUD), under the ImGui windows. |
| Saves | `BackupManager` | Back up every save file with a note, list and restore backups. Don't write saves without a backup and `ConfirmDialog`. |
| Pictures | `FrameCapture`, `Screenshots`, `Png` | The game's own frame (with HUD and overlays) to PNG, never the desktop. `Screenshots.Take()` (Ctrl+F12) saves to `UserData/OUAKModAPI/Screenshots`. `FrameCapture.SaveTexture` saves any texture, and `Png` encodes and decodes PNGs (this build has no ImageConversion). |
| Assets | `AssetReplacer`, `Wav`, `Bundles` | Replace textures (PNG) and sounds (WAV) by name, live. Mods can add their own folders or replace from code. `Bundles` loads Unity asset bundles (built with Unity 6000.0.30f1; async only in this build). |
| Hooks | `HookSet` | Harmony prefixes and postfixes on game methods, owned by your mod and removable together (`UnpatchAll`). A prefix that returns false replaces the method. |
| Hot mods | `HotMod`, `HotMods` | Mods that reload without restarting the game. See below. |
| Steam | `Steam` | Your SteamID and persona name, Steam session tickets, and checking another player's ticket (`Verify`), all through the game's own Steam library. |
| Online | `Multiplayer` | When OUAK Multiplayer is installed: who's in the session (`Players`, with SteamIDs and verification), and `Send` / `Received` on your own channel to keep your feature in sync between players. |

## Patches for broken and unused features

The API is two things in one: the modding toolkit above, and a patch for the game. The game ships with parts that never run or don't work: the online Race and Target rounds can't be played offline, the Target round never scores and its CPU princes never roll anything up, King Chances are never started, Shoes, Crown and Mushroom do nothing in story stages, and Smoke, Hoop and Dice were never finished. The API fixes them and makes them work.

Every patch is part of the API: it's installed when the API starts and only acts in its own context (crane points only in the Target mode, story-freebie effects only when a battle freebie is used in a story stage, King Chance effects only when a mod fires one...). Those contexts only exist through mods, so the unmodded game plays exactly as before. Mods don't switch patches on or off; they use the features. The Mods tab lists the patches and whether each is in place (`GamePatches.All`, `Ready`, `Error`).

| Patch | What's broken in the game | What the API does, and where |
| --- | --- | --- |
| `GamePatches.Race` | Offline training always runs the Big Battle rule, which the Race map can't run under (it bogs down and hangs); the game's per-map battle tables have no Race entry, so `MainGameManager.InitMultiPlay` throws halfway (KeyNotFoundException) and skips the rest of its setup. | On the Race map offline: the race rule, the course loaded before the CPU racers spawn, the restart points the map lacks, no hard-point arrow, the missing table entries (`MapTables`, Battle1's values), and a HUD safety net (`HudSafety`: if a map lacks a HUD part, the rest of the HUD still updates). Any other offline map always gets the Big Battle rule back. Start it with `GameModes.Start("race")` (`GameModes.RaceMode`). |
| `GamePatches.Target` | Same for the Target ("collect") map. | On the Target map offline: the collect rule, restart points, no arrow or hard point. `GameModes.Start("target")`. |
| `GamePatches.TargetPoints` | The collect rule never scores: only the results' end bonuses count. | In the Target mode only (the Target map under the collect rule): `TargetRound.PointsPerTarget` (10) points per crane rolled up, through the game's own point path (`MainGameCore.AddHardPointNum` -> `UserStatus.hardPointNum`, what the HUD's "pt" labels and the results show). What counts is the game's own: `MainGameManager.CheckMonoTargetType` (the origami cranes) and `GetMonoTargetCount` (a bundle counts each crane). `TargetRound.Count(core)` reads the game's per-prince crane counter, the big number the HUD shows instead of the diameter. An object scores once per prince, when it really goes onto that prince's katamari. Set `PointsPerTarget = 0` for none. |
| `GamePatches.TargetCpus` | The CPU princes never roll anything up in the Target round (measured: none in 30 s), and `MainGameCore.MultiGetMono` turns a CPU away from any object not flagged `isMultiplaySync`, a flag no object on the map has. | In the Target mode only: each CPU prince rolls up what its katamari touches (within `TargetRound.CpuReach` radii), through `MultiGetMono` inside that CPU's own update, so its crane counter, size and points follow the game's own path. |
| `GamePatches.KingChances` | Each battle draws 4 King Chances and the game has their texts and handler, but nothing ever starts one, and 3 of the 6 types only show their message. | When a mod fires one: `KingChances.Fire(type)` / `FireNext()` request it the game's way (the King's message, then the effect). King Chest gives every prince a King Box reward, Confiscation takes every prince's freebies (the one held, including yours in the HUD slot, and the ones in use: Crown power, Rocket, Magnet, unlimited dash, Smoke, Hoop; `KingChances.Confiscate(mgr, core)`), Unlimited dash gives everyone the game's 60 s unlimited dash (which only works with `GamePatches.BattleDash`). A King Chance fired while another is being shown waits for it (`KingChances.Queued`) instead of being lost. `KingChances.AddFilter(owner, f)` lets mods swap types side by side; `KingChances.Nothing` is the game's "... Nothing happened!". When to fire them is the mod's choice (the game's tip: "depending on your time remaining"). |
| `GamePatches.StoryFreebies` | In story stages Shoes, Crown, Mushroom and the Special Crown / Mushroom are used up with only a cosmetic effect (the Crown's glow even stays on until a restart): their code only runs in battles. | When a mod gives you one in a story stage: Shoes 5 s of unlimited dash, Crown / Special Crown 1.5x speed for 5 / 10 s (the user's choice; glow ends with it), Mushroom / Special Mushroom grow the katamari at once (2.5x / 5x a tenth of the stage's start size, the share battle maps use). Shoes and Crown keep their icon and countdown on the HUD like the story freebies do; both Mushrooms stay on the HUD for the battle's `Game.ONLINE_ITEM_TIME`, so the game's Mushroom effect plays (in story stages the game ends the freebie at once and `EffectManager.UpdateItemEffect` removes the effect it just started). `StoryFreebies.SetItem(item, on)` switches single ones (e.g. a host's choice). |
| `GamePatches.RandomFreebie` | Random can hand out freebies that do nothing (Hoop, Smoke, Dice, Special Dice in battles; battle freebies in Philosopher Hunt). | Only when the game's own draw is one that does nothing where you are: it draws again from the same group (normal / Special) among working ones. Story stages are never touched (their draws all work). |
| `GamePatches.BattleItemPools` | (A tool more than a fix.) King Boxes only ever give Buddy, Washtub, Mushroom, Magnet and Rocket (and Specials); item boxes only Washtub, Spiderweb, Bubble, Mushroom, Hand, Shoes, Crown (and Specials). | When a mod sets them, in battles: `BattleItemPools.SetKingBox(rewards)` / `SetItemBox(freebies)` choose what each may give, from everything the battle code can use (`KingBoxRewards`, `ItemBoxItems`: e.g. Hand or Shoes from a King Box; Smoke, Hoop and both Dice from item boxes); an empty pool gives nothing (the box is still taken); `Reset()` goes back to the game's own draw. King Boxes can also give the freebies with no reward type (`SetKingBox(rewards, freebies)`, `KingBoxFreebies`: Smoke, Hoop, Dice, Special Dice): they go to your slot with the roulette; CPU princes, who can't use them, get a reward of the pool instead. |
| `GamePatches.BattleDash` | Unlimited dash never works in battles: the Shoes and the King Chance set the prince's flag (`MainGameCore.SetMultiInfinityDash`), but the dash timer (`ExecuteOujiDashTimerSingle`, which battles use too) only reads the debug switch `GlobalManager._debugInifiniteDash`; and the host's item switch (`HostGameProcess`) has no case for Shoes at all, so they only show a countdown (TCRF: "no longer functions"). | Unlimited dash (battle Shoes and the King Chance; story Shoes through `StoryFreebies`), the user's choice "Both" (2026-11-09): **nonstop**, a running dash keeps going until you brake (both sticks back), hit a wall or climb, or the unlimited dash ends, which stops it (the game's own `ExtendOujiDash` every frame: the dash timer back to 1.5 s, what the dash input does while you keep wiggling); **free**, no stamina at all, no 0.5 start cost (`ExecuteOujiDash` adds it even under the game's switch), never out of dash, so you can dash again at once. Even the game's own switch only stopped the drain during a dash and every dash still ended after 1.5 s, which made 5 s of unlimited dash look like a normal one. Shoes start the game's unlimited dash for `BattleDash.ShoesTime` (5 s, the user's choice; the game's `bigBattleShoesTime` is 10), which the host ends. `BattleDash.Unlimited(core)` tells whether a prince has it; `NonstopFrames`, `FreeStarts`, `StoppedAtEnd` count what it did. Measured with real stick dashes (tokens `dashstory`, `dashbattle`): see API-CHANGES. This changes the game's own Katamari Ball (it's a bug fix). |
| `GamePatches.BattleFreebies` | Hand steals the other prince's King Box reward (`MainGameCore.StealItem` / `StolenItem`) but the stolen freebie never reaches your freebie slot, so it can't be used; a freebie stolen from you stays in your slot, unusable. The HUD's countdown ring (`GetItemEffectTimeRatio`) has no case for Spiderweb, Bubble, Hand, Shoes, Crown, Hoop, Smoke, Dice or the Specials after SP_Timer, so it stays empty. | In battles: while Hand is in your slot, an icon above every other prince shows the freebie it holds (`BattleFreebies.HandReveal`); using it takes from the nearest one who holds something (the game: a random one; CPUs keep that); the stolen freebie goes to your slot (with the roulette); one stolen from you leaves it; the ring shows the countdown's share of the time the game set. Crown / Special Crown last 5 / 10 s (`BattleFreebies.CrownTime` / `CrownPlusTime`, the user's choice; the game's are 15 / 30): the HUD countdown, the host's Crown power, the prince's own Crown and the King Box reward's text ("Fast & invincible (5 sec.)", "Unlimited dash (5 sec.)") all follow. Measured: Hand -> Crown in your slot, ready to use; Spiderweb ring 0.90 at 0.5 s, Crown 0.88 at 2 s. Also the game's own Katamari Ball. |
| `GamePatches.UnusedFreebies` | Smoke, Hoop, Dice and Special Dice have no effect. In story stages they're used up with a look only (Smoke even leaves you see-through until a restart); in battles they can't be used at all (no King Box reward type). The battle code still has pieces of them: the host never rolls up a katamari that is see-through (`UserStatus.timeTransparent`), invincible or has Crown power, Crown power knocks the other katamari away, and the never-called `StartLocalCoreTransparent` plus `MultiplaySetting.bigBattleSmokeBombTime` (10 s) / `bigBattleSmokeBombTransparentRate` (0.4) are an unfinished Smoke bomb. | When a mod gives you one. Smoke: battles = the game's Smoke bomb, see-through for 10 s and no katamari can roll you up; story = the ghost look for 10 s, then back to normal. Hoop (the user's choice, from the game's leftovers): battles = you drop through the rainbow ring and come out at a random spawn point, through the game's never-used random warp (`MainGameCore.SetRandomSpawnWarp` / `WarpRandomSpawnPoint`), with the ring at both ends (the second after the game's `HoopWaitTime`, 0.3 s; a warp that lands under 5 units away is redone); story = the ring only (no spawn points). Dice / Special Dice: roll a random freebie with the game's roulette (`SetInventoryItem(item, isRandom: true)`), normal ones / Specials, never a Dice, Random or Sunglasses, only ones that work where you are (`UnusedFreebies.DiceChoices`). |
| `GamePatches.ItemBoxes` | (A tool.) A story freebie box holds a mask of the five story freebies (`MainGameItemBox.ContainItem`) and a model of its freebie placed in the stage; nothing can change either. | When a mod sets a rule, in story stages: `ItemBoxes.SetEnabled(item, on)` (a box left with nothing disappears), `SetSwap(from, to)`, `SetBox(box, items)`, `Reset()`. A changed box gives its new freebie (the game's own steps: "get" animation, model onto the katamari, slot; several = the roulette) and shows the new freebie's model (`FreebieModels`). |

Measured in game with `UserData/OUAKModAPI/patchtest.flag` (see Test flags).

Not patched: Sunglasses (only a King Box reward with no effect and no text; kept as it is). Smoke, Hoop and Dice have no description in the game's texts either; what they do now is the design the user gave, built from the battle code's own pieces where they exist (above). Hoop's only code of its own is visual: the rainbow ring (effect particle #40, the "Ef_Hulahoop_Mass" effect), `MainGameManager.RequestHoopEffect` and `EffectManager.WaitAndCallHoopEffectCor` (both unused). The HUD's countdown ring also stayed empty for every freebie whose countdown the API runs (`MainGameManager.GetItemEffectTimeRatio` only knows the story freebies' times); the API fills it.

### Using the patches in your own mod

```csharp
// A battle: fire the match's next King Chance (only does something in a battle; the King shows it first)
if (KingChances.InOfflineMatch && !KingChances.Pending) KingChances.FireNext();

// Your match rules: no freebie King Chances, King Boxes that give Hand or Shoes, item boxes that give nothing
using var f = KingChances.AddFilter("My Mod", t => t == MultiplaySetting.KingChanceType.itemDrop ? KingChances.Nothing : t);
BattleItemPools.SetKingBox(new[] { MultiplaySetting.KingBoxRewardType.hand, MultiplaySetting.KingBoxRewardType.shoes });
BattleItemPools.SetItemBox(Array.Empty<eInstageItemType>());

// The Target mode: start it, then read the cranes and points
GameModes.Start("target");
GameEvents.Update += () => { if (TargetRound.InRound) Log.Info($"cranes {TargetRound.Count(Katamari.Core)}"); };

// A story stage: give a battle freebie that the API makes work there
StageControl.GiveFreebie(eInstageItemType.Shoes);

// Story freebie boxes: Magnet boxes give (and show) Shoes, no Timer from boxes at all
ItemBoxes.SetSwap(eInstageItemType.Magnet, eInstageItemType.Shoes);
ItemBoxes.SetEnabled(eInstageItemType.Timer, false);

// Your own model for a freebie, used everywhere the API shows one
FreebieModels.Register(eInstageItemType.Smoke, mySmokeModelTemplate);
```

## Hot mods

A hot mod is a class deriving from `HotMod` in a DLL in `UserData/OUAKModAPI/HotMods/`. The API loads it at startup and loads it again every time the DLL changes, so you can rebuild while the game runs. To try it:
1. Build `Samples/ExampleHotMod` with `dotnet build -c Release -p:NoInstall=false`. It builds straight into that folder.
2. Change its text and build again: its "Hot mod" tab updates without restarting the game.

```csharp
public sealed class MyHotMod : HotMod
{
    public override void OnLoad()
    {
        AddTab("My hot mod", () => ImGui.Text("v2!"));
        AddHotkey("Hi", "Say hi", "None", () => "Hi!");
        Hooks.Postfix(typeof(MainGameManager), "RestartStage", typeof(MyHotMod), nameof(AfterRestart));
        GameEvents.StagePlaying += OnPlaying;
        Track(() => GameEvents.StagePlaying -= OnPlaying);   // undone on unload
    }
    static void AfterRestart() { }
    void OnPlaying(MainGameManager gm) => Log.Info("go");
}
```

Everything added through `AddTab`, `AddWindow`, `AddHotkey`, `AddOverlay`, `AddGameMode`, `Hooks` and `Track` is removed before the new build loads. Override `OnUpdate`, `OnGUI` and `OnSceneLoaded` for per-frame work.

Limits:
* A hot mod can't register new Il2Cpp types (ClassInjector).
* An old build's code stays in memory, unused, until the game closes.

## Replacing textures and sounds

* Put `Textures/<texture name>.png` or `Audio/<clip name>.wav` in `UserData/OUAKModAPI/Assets/`. A mod can add its own folder with the same layout: `AssetReplacer.AddFolder`.
* Saving a file again swaps it live.
* A texture's own pixels are changed, so everything that uses it changes: rebuilt in place when readable, copied over on the graphics card for the game's compressed (DXT1/DXT5) textures, otherwise swapped where it's used.
* A picture of another size is scaled to the original's size.
* Each texture is tried once per file version. Scans happen only after a scene loads and when a file changes; with no replacement files, nothing runs at all.
* Sounds are swapped wherever an AudioSource plays them: music, loops, voices. One-shot effects that never sit in an AudioSource can't be swapped.
* A removed file's original comes back at the next scene load.
* To find names:
  * **List names** (Mods tab) writes `Assets/names.txt` with every loaded texture and sound.
  * **Export textures** saves the current scene's textures as PNGs to edit.

## Rules that keep the game from crashing

* **Never call a Unity method this build stripped.** MelonLoader rebuilds stripped methods around raw engine calls, and any engine error inside one crashes the whole game with nothing in the log. Before using a Unity API the game doesn't use itself, check it in a Cpp2IL dump. Known stripped members include:
  * `ImageConversion`, `GUILayout`, `GUI.Window`, `GUI.TextField`, `Texture.isReadable`;
  * `Graphics.Blit(src, dst)` (use the 4-argument one);
  * `AudioSource.timeSamples`, `AudioSettings.dspTime`;
  * every `Microphone` call that takes a device name.
* **Keep hooks small and catch everything inside them.** An exception thrown inside a hooked game method unwinds through native code.
* **Run game actions from Update** (`ModTasks.Run`), never from OnGUI.
* **Saves:** only through `ConfirmDialog` ("I understand") with a backup first. Never load stages before a save is loaded (`GameRefs.InGameSession`): the default slot could be overwritten.
* **DLC:** never make DLC available that Steam doesn't report as owned.
* **Online:** never use the game's own online service (PlayFab rooms, matchmaking). OUAK Multiplayer has its own connection, and `Multiplayer.Send` goes through it.

## Test flags

Create the file before launching the game. Each test logs to `MelonLoader/Latest.log`; most quit the game when done.

| Flag | Test |
| --- | --- |
| `UserData/OUAKModAPI/apitest.flag` | The API at the title screen: PNG and WAV, a prompt, a screenshot, live texture replacement, hot mod reload (with ExampleHotMod installed), tabs from every mod. Writes `[ApiTest] PASS/FAIL`. |
| `UserData/OUAKModAPI/patchtest.flag` | One patch in game, then quits: `target` (Target round: counter, points, HUD, results, CPU princes, then a Pyramid match without crane points), `kingchance` (every King Chance type in a battle, then the King Box and item box pools), `race` (Race round start to results), `freebies` (Shoes, Crown and both Mushrooms in a story stage, with pictures; hides UnityExplorer's windows), `battlemushroom` (both Mushrooms used in a battle), `unused` (Smoke, Hoop, both Dice and the freebie box rules in a story stage), `unusedbattle` (Smoke and Hoop against a bigger CPU prince, both Dice, in a battle), `hud` (the HUD of Drink, Magnet, Shoes and Crown compared), `texts` (game texts about freebies). Makes a save backup and loads the last-used slot (`slot N` picks one). Writes `[PatchTest] OK/FAIL` lines and `PASS`. |
| `UserData/OUAKModMenu/selftest.flag` | The mod menu: every tab of the window, the confirmation dialog, a report of what the mods see. |
| `UserData/OUAKMultiplayer/trainingtest.flag` | A Katamari Ball map (Battle, Race, Target...) end to end. See the multiplayer README. |
| `UserData/OUAKModMenu/onlinetest.flag` (`run Stage_Big1`) | The full online test with a loopback bot. See the multiplayer README. |
