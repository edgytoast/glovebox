# Installing Glovebox on Apple Vision Pro

Glovebox plays your own Steam copy of *Half-Life: Alyx* on Apple Vision Pro, on the headset itself: no PC and no streaming. Steam's copy of the game includes a native 64-bit ARM Linux build. [Klepton](https://github.com/shinyquagsire23/Klepton), by shinyquagsire23, converts its libraries into signed Vision Pro code on your Mac and runs the game against Apple's frameworks: Vulkan through [MoltenVK](https://github.com/KhronosGroup/MoltenVK), the displays through Compositor Services, and the PS VR2 Sense controllers through Game Controller and ARKit. The game renders both eyes itself, with head tracking from ARKit. [How it works](docs/HOW-IT-WORKS.md) has the whole story.

## What you need

- Apple Vision Pro on visionOS 26 or later, with about 75 GB free. The README says it's tested on M5 and M2 headsets.
- Two PlayStation VR2 Sense controllers. The game needs their buttons, sticks and triggers; hand tracking isn't supported.
- Half-Life: Alyx in your own Steam library. Signing in uses Steam Guard: approve it in the Steam app on your phone, or enter a code.
- A Mac with Apple silicon and Xcode 26 or later, with at least 15 GB free. The Glovebox app installs Xcode's visionOS support if it's missing; from Terminal, `doctor` names the command (`xcodebuild -downloadPlatform visionOS`).
- An Apple Account signed in to Xcode (Xcode › Settings › Apple Accounts). A free account works: Glovebox builds with its Personal Team, and Apple gives free teams the Increased Memory Limit capability the game needs (it uses about 6 to 8 GB). An install lasts 7 days with a free account, a year with the paid Apple Developer Program.
- The headset paired with your Mac (on the headset, Settings › General › Remote Devices) and Developer Mode on (Settings › Privacy & Security).

## Your game files

Nothing from Half-Life: Alyx is in this repository or its releases, and Glovebox never hosts or shares game files ([Legal](docs/LEGAL.md)). Every file comes from your own Steam account to your own devices, and Steam refuses the download to an account that doesn't own the game.

- **On the Mac:** Glovebox signs in to Steam in its own screen and downloads about 1.7 GB: the game's ARM64 Linux libraries and its Vulkan shader packs (`steam.mac_download` in `packages/alyx/package.json`). They go to `~/Library/Application Support/Glovebox/alyx/linux/game/`. The build converts the libraries and rebuilds the game's graphics pipeline list from your shader files; this repository carries only a recipe for it, with no shader code.
- **On the headset:** the game app, **Glovebox: HLA**, signs in to Steam and downloads the rest of the game, about 67 GB, straight from Steam's servers, with your Steam Cloud saves.
- Your Steam password goes only to Steam, encrypted with Steam's key, and is never stored. Glovebox keeps only Steam's sign-in token, in your Keychain.
- Glovebox installs the game build it was tested with (`steam.build` in `packages/alyx/package.json`), so a Steam update can't break your install. Newer game versions come as Glovebox updates once they're tested.

## Install

There's no prebuilt Vision Pro app: visionOS runs only code that is signed into the app, so the game app is built on your Mac from your own copy and signed with your own Apple account. The [releases](https://github.com/glovebox-vr/glovebox/releases) have a prebuilt **Mac** app, Glovebox, that does all of it. It contains no game files.

1. [Download the Glovebox app](https://github.com/glovebox-vr/glovebox/releases/latest/download/Glovebox.dmg) and open it.
2. Follow along. It checks the Mac, sets up Xcode, your Apple account, Steam and the headset, downloads your game files, builds the game app (about 2 minutes) and installs it on the headset. You step in only where Apple or Steam need a person, and it shows you where.
3. Put on your headset and open **Glovebox: HLA**. The first launch is described under [Install on Apple Vision Pro](#install-on-apple-vision-pro).

The Mac app is signed with a Developer ID and notarized, updates itself from this repository's releases (Sparkle), and stays in the menu bar to offer game updates and renewals.

## Build from source

The Mac app runs the same engine you can run from Terminal, `engine/glovebox.py`. From the root of a checkout of this repository:

1. **Check this Mac.** `xcrun python3 engine/glovebox.py doctor` lists what this Mac and headset still need, each with the command or setting that fixes it. Run its fixes (visionOS support: `xcodebuild -downloadPlatform visionOS`) before you build the kit.

2. **The runtime kit.** `build` links a prebuilt, open-source runtime: Klepton, its Half-Life: Alyx add-on with LuaJIT, MoltenVK, SDL 3, sdl2-compat and `klepton-ld`. The Mac app carries it. From a checkout, build it once with `engine/kit/build-kit.sh`, which needs cmake on top of Xcode. It fetches SDL 3, sdl2-compat, LuaJIT, Fossilize and MoltenVK at pinned versions and writes the kit to `glovebox-kit/` at the repository root, where `build` finds it by itself (`GLOVEBOX_KIT` can point at another kit folder or .zip):

   ```sh
   brew install cmake
   zsh engine/kit/build-kit.sh glovebox-kit
   ```

3. **Your game files.** The Terminal commands don't sign in to Steam. `build` reads the game's files from `~/Library/Application Support/Glovebox/alyx/linux/game/`, where the Mac app's download puts them, or from `GLOVEBOX_GAME` if you already have the game's Steam depot `game/` folder on disk:

   ```sh
   export GLOVEBOX_GAME="/path/to/game"   # only when the files aren't in the folder above
   ```

   It checks that `bin/linuxsteamrtarm64/`, `hlvr/bin/linuxsteamrtarm64/`, `core/shaders_vulkan_dir.vpk` and `hlvr/shaders_vulkan_dir.vpk` are there.

4. **Check, build and install:**

   ```sh
   xcrun python3 engine/glovebox.py doctor    # what this Mac and headset still need
   xcrun python3 engine/glovebox.py build     # the game app, from your own game files, signed for your team
   xcrun python3 engine/glovebox.py install   # onto your paired Vision Pro
   ```

`doctor` lists each check (Mac, space, Xcode, visionOS support, Apple account, headset) with the action that fixes it. `build` then:

- downloads Debian's arm64 runtime libraries (libstdc++, OpenSSL, GLib and others), checked against pinned hashes (`packages/alyx/runtime-debs.json`);
- rebuilds the pipeline list from your shader files and translates every game library with `klepton-ld`;
- generates the Xcode project and builds it in Release, signed for your team with automatic provisioning. It registers your paired headset with your team, so pair it before you build.

The app's bundle identifier is `com.glovebox.alyx.t` followed by your `TEAMID` in lowercase, so there's nothing to change for your own team. The built app is `~/Library/Application Support/Glovebox/alyx/KleptonAlyx.app`.

If your Xcode came from the App Store, macOS asks once for your Mac's password before the build may sign with your key. Type it and click **Always Allow** with the mouse: Enter picks Allow, which covers a single signature. [docs/SIGNING-PROMPTS.md](docs/SIGNING-PROMPTS.md) explains why.

Optional settings for `build`, as environment variables:

- `GLOVEBOX_TEAM=TEAMID`: the team to sign for (default: a paid team in Xcode's settings, else the free one)
- `GLOVEBOX_FREE_TEAM=0`: refuse a free Personal Team
- `GLOVEBOX_WORK`: where everything goes (default `~/Library/Application Support/Glovebox/alyx`)
- `GLOVEBOX_SHARE_STUTTERS=1` or `0`: your answer about stutter reports (see Notes; unset, the headset asks)

## Install on Apple Vision Pro

`install` copies `KleptonAlyx.app` to the first paired headset with Developer Mode on, with `devicectl`. Keep the headset on and unlocked, on the same Wi-Fi as the Mac. The Mac app does this step for you.

1. With a free Apple Account, trust the developer once after the first install: on the headset, open Settings › General › VPN & Device Management, tap your Apple Account, then Trust.
2. Open **Glovebox: HLA**. Sign in to Steam in its own screen and approve the sign-in in the Steam app, or enter a code. It checks that you own the game, downloads the rest of it and syncs your Steam Cloud saves. With the headset off, the download carries on in the background, more slowly; open Glovebox: HLA again to finish it.
3. Turn on both Sense controllers. Play waits until both are on, and visionOS asks for permission to track them when you press Play.

## Notes

- **Renewing:** an install stops opening after 7 days with a free Apple Account, or after a year with a paid one. Glovebox on the Mac shows **Renew** two days before a free install runs out (a month before a paid one). Renewing builds and installs it again and keeps your game files and saves. From Terminal, run `build` and `install` again.
- **One team:** each Apple team gets its own copy of the app on the headset, because the bundle identifier includes the team, so switching teams downloads the game again.
- **Saves:** Glovebox syncs your Steam Cloud saves the way the Steam client does, with a backup before it replaces anything. A save made on the headset goes up the next time you open Glovebox: HLA, because quitting the game closes the app. Your PC's game settings (teleport movement, say) come down from Steam Cloud too; change them in the game's options on the headset.
- **Known issues** ([full list](docs/KNOWN-ISSUES.md)): the game draws 45 frames a second and the headset shows each one twice, so close objects can look doubled at the edges when you move your head; a few busy scenes run below full speed; left-handed and one-controller modes don't work, so play with two controllers, right-handed.
- **Stutter reports** are opt-in. Glovebox asks once, on the Mac during the install or on the headset during the download. With a yes, the headset sends a report to `glovebox-reports.vercel.app` each time the app opens: fingerprints of the graphics pipelines it compiled (no shader code), the log of slow pipeline compiles, the "Preparing graphics" timings, the headset model, the visionOS, Glovebox and kit versions, and an anonymous install id ([diagnostics/README.md](diagnostics/README.md)).
- **Problems:** press **Copy diagnostics** in the Mac app and [open an issue](https://github.com/glovebox-vr/glovebox/issues/new). The diagnostics leave out passwords and game files.
- Glovebox is an open-source fan project, not affiliated with or endorsed by Valve or Apple ([Legal](docs/LEGAL.md)).
