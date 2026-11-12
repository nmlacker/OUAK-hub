# OUAK Mod Menu

MelonLoader (IL2CPP) mod for *Once Upon a KATAMARI*, built on the **OUAK Modding API** (`OUAKModAPI.dll`, required). See [../README.md](../README.md) for the API and the other mods.

**A single-player menu**: for speedrunning, TASing and messing around with the story side of the game. The one exception (your request, 2026-11-12): the Freebies tab also works in Katamari Ball matches. Otherwise it doesn't change KatamariBall (Sports Katamari) or anything multiplayer: since 2026-11-06 the Match tab, the multiplayer maps and the Multiplayer Room are **OUAK Multiplayer**'s. The menu works on its own with just the API (measured: self-test with OUAK Multiplayer removed).

Press **Right Shift** in game to open the mod window (rebind it on the Keys tab). The menu's pages are tabs in the API's shared window, next to the tabs other mods add: **Online** from OUAK Multiplayer, and the API's own **Mods** tab. While the window is open the game ignores your keyboard and controllers (`BlockGameInputWhileOpen`), and a controller moves through the window instead. Since 1.4.0 the pages are drawn with Dear ImGui through the API: choices are grids you click (double-click a freebie to give it right away), settings are tick boxes and drop-downs.

Since 1.3.0, these live in the API:
* the window, the hotkey system, the "I understand" dialog and backups;
* map loading and the stage actions;

This mod is the pages and the cheats built on them. Settings keep their old MelonPreferences categories.

| Tab | What it does |
| --- | --- |
| Freebies | Works in story stages and (since 2026-11-12, your request) in **Katamari Ball matches**: there the top grid holds the freebies that work in battles, the rest are listed as "does nothing in battles", and give / clear / freeze use the game's battle way (the roulette spins, then the freebie lands in your battle slot; measured: Crown -> crown power, frozen Shoes back after their effect, clear empties the battle slot). Spawn-with-a-freebie stays story-only. In story stages: every freebie, in two grids: the ones that do something in story stages, then the **cosmetic-only** ones (Tarai, Spider, Bubble, Hand, Smoke, Hoop, Dice, Sunglasses and their Specials: battle freebies that act on other princes, and ones the game never finished). Those are given anyway (`StageControl.GiveFreebie(item, evenIfItDoesNothing: true)`) and used up with just their visual effect. Hover a freebie to see what it does (the API's `ChoiceGrid` tooltips): the game's own description (its freebie tips, in the game's language; Drink is the game's "Rocket", Pickel its "Ice axe") and its measured effect in story stages. Give any freebie, clear it, **freeze** it so it comes back after the effect ends, or **auto-give** a chosen freebie every time a stage starts, restarts or reloads. Working in story stages: Drink, Magnet, Pickel, Sonar, Timer and their Specials (twice as long), plus **Shoes** (unlimited dash for 10 s), **Crown** (1.5x speed for 15 s; Special 30 s, as in battles) and **Mushroom** (enlarges the katamari at once by 2.5x, Special 5x, a tenth of the stage's start size: e.g. +12.5 mm in As Large As Possible 1). Shoes/Crown/Mushroom and the Random fix are the API's patches, always part of the API and acting only in story stages; the menu just gives the freebies. |
| Item boxes | What story stages' freebie boxes give. A box holds Drink, Magnet, Pickel, Sonar or Timer: switch one off (a box left with nothing disappears until you allow it again) or swap it for any freebie; the box then gives the new one and shows its model (the API's `ItemBoxes` and `FreebieModels`: the game's own model where it has one, else the freebie's icon on a card). Saved under `[OUAKModMenu]` (`ItemBoxOff`, `ItemBoxSwaps`) and applied in every story stage. In a stage, each box is listed with what it gives, and one box can be set to something else for that visit (not saved). |
| Stage | Freeze the stage timer, complete the timer (time's up now), finish the stage (instant clear), instant restart, full reload, return to the Plaza. |
| Levels | Every single-player map under the name the game shows, grouped by era, with a search box: all story stages (same path the King uses) and the twelve era plazas. |
| Modes | Jump to Title, Save Select, Plaza, Prince's Ship, Ending Game, Cosmos, Collection, Memorial, Achievements, Character Customize, dev Stage Launcher. |
| Keys | Bind a key (or Ctrl/Alt/Shift + key) to any action of any mod: open the window, timer/freebie toggles, give/clear freebies, complete timer, finish/restart/reload stage, return to the Plaza, screenshots (API), and whatever other mods register. |
| Unlocks | Unlock everything on the loaded save slot / relock it back to its pre-unlock progress. |
| DLC | Lock (hide) DLC you own so the game treats it as not installed, and unlock it again. |
| Backups | Make a backup with a note, browse backups (with per-slot details), restore one. |
| Online | Comes from **OUAK Multiplayer** (a separate mod since 1.3.0): see [../OUAKMultiplayer/README.md](../OUAKMultiplayer/README.md). |

## Maps

* **Names** come from the game: the King's mission list (`SelectHiroba/MissionItemData` → stage name table), then the stage's own Japanese tag matched against the text tables, then a readable form of the internal name.
* **Plaza maps** set the era (`GlobalSaveData._stageIndex`) and reload the Plaza, like the in-game map jump. Story events for an era you haven't reached may play out of order.

## Cosmos

* **Cosmos** is entered the way the ship does it: fade out, stop the music, set the ship spawn point for the way back, then load. If you're elsewhere the mod goes to the ship first.

## Online

Online play moved to its own mod, **OUAK Multiplayer**: see [../OUAKMultiplayer/README.md](../OUAKMultiplayer/README.md).

## Hotkeys and auto freebie

* Bindings are saved under `[OUAKModMenu_Keys]` in `UserData/MelonPreferences.cfg` (`Key_<Action> = "Ctrl+F5"`, `"None"` to unbind). Two actions sharing a key are flagged on the Keys tab.
* Auto freebie (`AutoFreebie`, `AutoFreebieItem` under `[OUAKModMenu]`) arms when a new stage loads or the stage clock resets / a restart finishes, then gives the item once you have control.

## "I understand" prompt

Anything that edits, moves or restores save data (Unlock, Relock, Restore backup, DLC lock/unlock) opens a dialog where you type **I understand** (capitals don't matter) before it runs. The dialog also takes an optional note, which is stored with the automatic backup made first.

## Backups

* Saved to `UserData/OUAKModMenu/Backup/<yyyy-MM-dd_HH-mm-ss>_<reason>/`:
  * `SaveGames/…` – copies of every save file (all three slots + system data),
  * `info.txt` – readable summary: time, reason, your note, game version, and for each slot the last-saved time, stages cleared, crowns, cousins, presents, collection objects, stardust and completion; plus which DLC was owned/locked,
  * `backup.json` – the same in machine-readable form,
  * `LoadedSlot.bin` – a live snapshot of the slot that was loaded (when made from inside a save).
* **Restore** copies a backup's save files back (current ones are backed up first) and makes the game re-read them. It only runs on the Title screen; use *Go to Title* on the Backups tab first. Slots that didn't exist in the backup are left alone.
* By hand: quit the game and copy a backup's `SaveGames` contents to `%LOCALAPPDATA%\Bandai Namco Entertainment\Once Upon A KATAMARI\Saved\SaveGames`.

## Unlock / relock

* Use it from the Plaza (not inside a stage).
* The first unlock on a slot records its backup as the slot's pre-unlock state (`UserData/OUAKModMenu/unlock_state.json`); Relock writes that state back through the game's own save converter, keeps your current options, saves, and reloads the Plaza.
* DLC content is only unlocked for DLC that is owned on Steam and not locked.

## DLC lock

The DLC tab lists every DLC pack the game knows about, by its Steam name: *Rolling Live Highlights* (gates the AA stages and their music), *Cousin Customization: Face*, and the four *Katamari Damacy Series* music packs.

* **Lock** hides DLC you own: the game treats it as not installed, so a brand-new game starts with no DLC prompts, stages, music or presents. Locks are remembered between launches.
* **Unlock** makes owned DLC visible again. On a save that never saw it, the game then offers it like newly bought DLC, so you can switch back at any time or start a new game with it.
* Change locks from the Title screen (or the Plaza), then **restart the game**: DLC music is only registered when the game starts.
* The mod never makes DLC available that Steam doesn't report as owned.
* The *bonus for owning other KATAMARI games* can be locked the same way. Owning Katamari Damacy REROLL or We Love Katamari REROLL+ unlocks one thing: the King as a playable character (cousin #58, set in `TableCustomData.OldGameDataSetModel`), announced by a one-time prompt. Locking it stops new saves from getting him; saves that already have him keep him. "Unlock everything" only grants the King when the game has detected one of those games and the bonus isn't locked.
* How it works: the game keeps DLC availability in `DlcBase.dictDlcAvailable`, filled by `DlcSteam.GetDownLoadContent()` from Steam. A postfix on that method (and a per-frame check) clears locked packs with the game's own `DlcManager.SetAvailable(i, false)`; unlocking repeats the game's Steam ownership check for that pack only.

## Building

```
dotnet build -c Release
```

Run it here, or `dotnet build ../OUAK.sln -c Release` for every mod. This builds the API too (`OUAKModAPI.dll`). References come from the game's `MelonLoader/` folder (see `../Directory.Build.props`), and the build copies the DLLs into `Mods/`.
Creating `UserData/OUAKModMenu/dlcdump.flag` before launch writes `UserData/OUAKModMenu/dlc_items.txt` at the title screen: every DLC song, costume piece, cousin and quest, read from the game's own tables (read-only).
Creating `UserData/OUAKModMenu/selftest.flag` before launch runs a self-test at the title screen (logged to `MelonLoader/Latest.log`); it makes one backup copy but never writes to the saves.
Creating `UserData/OUAKModMenu/selftest.flag` also shows one freebie's hover text without the mouse (the test never moves the pointer) and logs every freebie's hover text. `textdump.flag` (comma-separated words; a word ending in `_` is an id prefix, e.g. `TIPS_`) writes the game's matching texts to `UserData/OUAKModMenu/texts.txt` at the title screen and quits (read-only).
The multiplayer-map test (`trainingtest.flag`) and the King Box / King Chance / Target probes moved out with the KatamariBall features (kept in `Moved/`, not built). The King Box fix was measured before the move: with every freebie off, 5 King Boxes gave nothing; with only Basin, Rocket and Crown allowed, all 5 gave Rocket, and the buddy counts stayed true.

Creating `UserData/OUAKModMenu/itemtest.flag` measures every freebie in As Large As Possible 1 (after a backup): the game's battle item tables, each freebie's effect time and strength after the game's own activation, the rebuilt Shoes and Crown, the Random draw and the refusals. `freebietest.flag` checks the spawn freebie and the restart (`slot 3` in the file picks a save slot; it closes one-time game tips that open during the restart, which pause game time until closed).

The online test (`onlinetest.flag`) is OUAK Multiplayer's: see its README.
