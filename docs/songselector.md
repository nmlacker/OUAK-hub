# OUAK Song Selector

Playlists for *Once Upon a KATAMARI*: mix the game's own songs with your own music files (MP3, M4A, FLAC, WMA, WAV...), and choose where each playlist plays: everywhere, every story stage, the Plaza, or single stages. Everything is managed from the **Songs** tab of the mod window (**Right Shift**).

Built on the OUAK Modding API (required). It only changes which music plays; nothing is written to your save.

## Install

1. Install the OUAK Modding API (`Mods/OUAKModAPI.dll` plus its `UserLibs`).
2. Copy `OUAKSongSelector.dll` to `Mods/`.
3. Put your music in `UserData/OUAKSongSelector/Music/` (created on the first start; subfolders are fine).

## The Songs tab

**Now playing** shows where you are (a story stage or the Plaza), which playlist plays there and why, and the song with its position. **Previous**, **Next** and **Stop playlist here** control it. "Stop playlist here" gives the place its own music back until you leave.

**Playlists**
* **New playlist**, **Duplicate**, **Delete** (asks first). Click a playlist to edit it; type a new name and press Enter to rename it.
* **Shuffle**, and what happens after the last song:
  * **Repeat all**: start over (a shuffled list is shuffled again);
  * **Repeat one**: the same song again;
  * **Play once**: after the last song, the game's own music comes back.
* The song list: **Play** (plays that song here now), **Up** / **Down**, **X** (remove). Songs that can't play are greyed out with the reason.
* **Add game songs**: the game's music with its real titles and singers, grouped like the game's own music player (base game, then the DLC music packs), instrumental versions included. Type in the filter box to search.
* **Add your files**: the music files in your Music folder.
* **Play here now** (in a stage or the Plaza): plays that playlist here until you leave.

**Where they play**: the most specific choice wins.
1. A **single stage** ("Add a choice for this stage"; "This stage" picks the one you're in). A single stage can also be set to *the game's own music*, which overrides everything else there.
2. **All story stages** / **The Plaza**.
3. **Everywhere**.
4. Nothing chosen: the game's own music, including the song you pick in a fan's talk and the jukebox's shuffle.

A change takes effect at once in the place you're in.

**Your music folder**: the folder's path, **Rescan the folder**, and files in formats this mod can't read. Rescan is rarely needed: the folder is checked by itself (see "Keeping up with your files").

**Volume**: your files follow the game's music and master volume (Options), times "Your files' volume". With the game's music volume at 0, your files are silent too.

**Options**
* your files' volume, on top of the game's BGM and master volume, which apply to them as well;
* show the song's name when a playlist song starts;
* allow base-game songs your save hasn't unlocked yet.

**Hotkeys** (Keys tab, all unbound by default):
* **Play / pause the playlist**:
  * pause holds your place, including through the pause menu and the Timer freebie;
  * Play goes on from there;
  * with nothing playing, Play starts the playlist chosen for where you are.
* **Next song** / **Previous song**: Previous from more than 3 s into a song starts that song again.
* **Loop the playlist on / off**: switches between "Repeat all" and "Play once".
* **Loop this song on / off**: switches between "Repeat one" and "Repeat all".
* **Shuffle on / off**.

Loop and shuffle change the playlist that's playing and are saved with it. Playing a song (Next, Previous, Play here now) ends a pause.

## Which songs can play

* **Game songs**: the songs of the game's own music player (118 entries in this version). Base-game songs need to be unlocked in the save you're playing, like in the game's music player; the option above lifts that. DLC songs need their DLC pack, as reported by the game.
* **Your files**: the game plays its music with CRI ADX2, which can't read your files, so they play through Unity's audio next to it. Two decoders feed it, and neither needs anything installed or shipped:
  * **WAV** is read by the mod (PCM 8/16/24/32-bit or 32-bit float, any sample rate).
  * **Everything else** is decoded by Windows itself (Media Foundation, part of Windows 10/11). The mod lists exactly the types this Windows can open; the Songs tab shows them under "Your music folder". On a standard Windows 10 that's MP3, M4A / AAC / ALAC, FLAC, WMA, MKA, WebM / WEBA (Opus) and a few more. A WAV the mod can't read (e.g. ADPCM) also goes to Windows.
  * **OGG / Opus (.ogg, .opus)** aren't in a standard Windows 10. Microsoft's free **Web Media Extensions** (Microsoft Store) adds them, and the mod picks them up automatically. Otherwise convert them, e.g. `ffmpeg -i song.ogg song.flac`. Windows "N" editions need Microsoft's Media Feature Pack for any of the Windows-decoded types.
  * Files of a type this Windows can't open are listed under "Your music folder".

## How it plays

* **Story stages**: the playlist's song replaces the one the game would load when the stage starts and at every retry, so it starts, pauses (pause menu, King's messages) and stops (results) exactly like the game's music. Your files follow the game's music the same way: the game's own song keeps running silently underneath and your file pauses, resumes and fades out with it.
* **The Timer freebie**: while it stops the clock, the game holds its music (with a CRI action cue, not a plain pause). Your file stops in the same frame, stays stopped for as long as the Timer runs (also when you open and close the pause menu meanwhile) and goes on from the same place when the clock runs again.
* **The Plaza**: the playlist replaces the era's theme, the way the game's own jukebox does. When the game plays something else there (a fan's stage screen, the jukebox), the playlist pauses; when the theme would come back, the playlist goes on from where it was.
* **Song ends**: the game's songs are endless loops. The mod reads each song's play-through length from the game's sound files (`StreamingAssets/Sound/*.acb`) and moves on when one play-through is over. Your files move on when they end.
* The game's song-name banner in a stage names the playlist's song (your file's name for one of yours). A toast also names each song as it starts (Options).
* Each playlist remembers its place while the game runs: going from the Plaza into a stage continues with the next song.
* The game's jukebox shuffle can't change songs under a playlist in a stage.
* Multiplayer and Katamari Ball matches are never touched.

### Limits

* Your files don't get the game's own music effects, which the game applies to its songs through CRI action cues (`BGM_ActionTrack`: pitch-up, slow play, count-down, extra layers). The game's songs in a playlist keep them.
* How loud your files are next to the game's songs wasn't measured: adjust "Your files' volume" to taste.
* A file is decoded into memory before it plays (about 20 MB per stereo minute at 44.1 kHz; a 4-minute MP3 took 0.8 s, a FLAC 0.5 s, off the game's main thread). The mod keeps at most two loaded and starts loading the playlist's next file early. Songs longer than 20 minutes are cut there.
* Other places (title screen, ship, results, menus) keep the game's music.

## Keeping up with your files

The mod asks Windows to tell it when the Music folder (any subfolder) or `playlists.json` changes, so nothing is read while nothing changes. Half a second after the last change (copying a file fires many), the folder is listed once, off the game's main thread, and the song list updated.
* **New, changed, renamed or removed music files** show up in the Songs tab by themselves; they can be added and played without Rescan.
* **`playlists.json` changed outside the game** (another editor, a copied file) is read again, and the new playlists and choices apply at once.
* **A file that doesn't read** (half saved, a typo) is ignored: the playlists in use stay, the Songs tab says what's wrong, and the next change is tried again.
* If Windows can't watch the folder, it's checked every 30 seconds instead.

## Files

* `UserData/OUAKSongSelector/playlists.json`: your playlists and where they play, saved on every change. Game songs are stored as `game:N` (the game's song number; +1000 = the instrumental version), your files as `file:relative/path.mp3`.
* `UserData/OUAKSongSelector/Music/`: your music files.
* `UserData/OUAKModAPI/Logs/Song Selector.log`: what the mod did.

## Building

```
dotnet build OUAKSongSelector/OUAKSongSelector.csproj -c Release -p:BuildProjectReferences=false
```

It installs to `Mods/` (close the game first); `-p:NoInstall=true` builds without copying.

## Test

`UserData/OUAKSongSelector/songtest.flag` runs the in-game test once, then quits the game (log lines start with `[SongTest]`):
* `play` (the default): two test WAVs (plus an MP3, a FLAC and an OGG if `UserData/OUAKSongSelector/testfiles/` holds `B.mp3`, `E.flac`, `F.ogg`) and two playlists, which are saved and read back; then the Plaza and *As Large As Possible 1*. It checks song ends, Next / Previous, the jukebox taking over, the pause menu, repeat one, a settings change, a retry and a locked song being skipped. It also captures the Songs tab to `UserData/OUAKSongSelector/TestShots`. Your `playlists.json` is put back afterwards, and the test files are deleted.
* `timer`: your file plays in a stage while the Timer freebie is used, the pause menu is opened and closed during it, and the freebie ends. It checks, frame by frame (`timertest.csv`), that the file stops with the game's music, holds its place, and goes on from there.
* `controls`:
  * Pause / Play with your file and with a game song, also across the pause menu;
  * the loop, loop-one and shuffle switches;
  * a file added to the Music folder and removed again;
  * `playlists.json` changed outside the game, then broken.
* `gallery` (read-only): opens the game's music gallery in the Plaza the way its menu does, writes its whole object tree to `gallery_dump.txt`, captures pictures, and closes it.
* `overlap`: a playlist changes songs again and again (Next, Previous, song ends, game song ↔ your file) in the Plaza and a stage, while CRI's own counters (streaming voices and sequences in use) check that only one song plays at a time.
* `survey`: logs how the game plays its music: the song tables to `survey.txt`, every music call, the CRI channels, and Unity's audio output.
* `slot N` picks a save slot (default: the last one used). Save writes are blocked while a test runs.
* `loud` plays your files at a fixed test level whatever the game's music volume is, so their level can be measured. The game's settings aren't touched.

Tests that change the settings keep your `playlists.json` aside, in memory and as `playlists.before-test.json`, and put it back when they finish. A copy left by a test that didn't finish is put back by the next test. The `gallery` and `survey` tests don't touch it.
