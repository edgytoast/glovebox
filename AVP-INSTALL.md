# Installing Glovebox on Apple Vision Pro

Glovebox is by the [Glovebox contributors](https://github.com/glovebox-vr/glovebox). This repository is a copy of
theirs with this guide and a few build fixes added, so it can be listed in the
[AVP Ports Index](https://github.com/edgytoast/avp-ports-index); [Trevorbilt](https://trevorbilt.com) wrote both.
Report problems with Glovebox [on their repository](https://github.com/glovebox-vr/glovebox/issues), not here.

*Half-Life: Alyx* runs on Apple Vision Pro itself, with no PC and no streaming: Glovebox builds it on your Mac from
your own Steam copy and installs it on your headset. None of the game's files are here.

## Requirements

- **Apple Vision Pro** on visionOS 26 or later, with about 75 GB free.
- **Two PS VR2 Sense controllers.** Hand tracking isn't supported.
- **Half-Life: Alyx** in your Steam library, with Steam Guard (the Steam app on your phone, or codes).
- **A Mac with Apple silicon** and **Xcode 26** or later, with about 15 GB free.
- **An Apple Account** signed in to Xcode (Xcode › Settings › Apple Accounts). A free account works, but apps it signs stop
  opening after 7 days until you install them again.

## Game Files

Glovebox signs in to Steam with your account and downloads only what you own: about 1.7 GB to the Mac to build the
app, and the rest of the game (about 67 GB) straight to the headset, with your Steam Cloud saves. Your password goes
only to Steam and is never stored.

## Install with the Glovebox app

The easy way. The Glovebox app is a prebuilt Mac app from the Glovebox developers; the AVP Ports Index reviewed this
repository's source, not that app.

1. **[Download the Glovebox app](https://github.com/glovebox-vr/glovebox/releases/latest/download/Glovebox.dmg)** and
   open it.
2. **Follow along.** It sets up Xcode, your Apple Account, Steam and the headset, downloads your game files, builds the
   game and installs it on the headset. You only step in where Apple or Steam need a person, and it shows you where.
3. Continue with **On the headset** below.

## Or build from source

This builds the code the AVP Ports Index reviewed. The one prebuilt part is MoltenVK, Khronos's official release, which
the build checks against a pinned hash. Clone into a folder with no spaces in its path (the kit build can't take
them); if you came from the index and already cloned the commit it lists into one, skip the clone.

```bash
git clone https://github.com/edgytoast/glovebox.git ~/glovebox
cd ~/glovebox
xcrun python3 engine/glovebox.py doctor
```

`doctor` lists what the Mac and headset still need, each with its fix (for example
`xcodebuild -downloadPlatform visionOS`, pairing the headset, or turning on Developer Mode). Run it again until
everything passes.

Download the game's Mac files with Glovebox's own Steam tool, built from this checkout. `signin` asks for your password
(it goes only to Steam) and a Steam Guard approval; `fetch` takes the game build this checkout pins in
`packages/alyx/package.json`:

```bash
swift build -c release --package-path launcher
launcher/.build/release/steamctl signin YOUR_STEAM_ACCOUNT
for DEPOT in 546564 546561; do
  launcher/.build/release/steamctl fetch --app 546560 --depot $DEPOT \
    --manifest "$(xcrun python3 -c "import json; print(json.load(open('packages/alyx/package.json'))['steam']['manifests']['$DEPOT'])")" \
    --include game/bin/linuxsteamrtarm64/ --include game/hlvr/bin/linuxsteamrtarm64/ \
    --include game/core/shaders_vulkan_ --include game/hlvr/shaders_vulkan_ \
    --out "$HOME/Library/Application Support/Glovebox/alyx/linux"
done
```

Then build the runtime kit once (about 25 minutes), and the game:

```bash
brew install cmake
zsh engine/kit/build-kit.sh glovebox-kit
xcrun python3 engine/glovebox.py build
xcrun python3 engine/glovebox.py install
```

If your Xcode came from the App Store, macOS asks once for your Mac's password before signing: click **Always Allow**
with the mouse ([why](docs/SIGNING-PROMPTS.md)). For `install`, keep the headset on, unlocked and on the same Wi-Fi
as the Mac.

## On the headset

1. **Trust it, with a free Apple Account:** Settings › General › VPN & Device Management.
2. **Open Glovebox: HLA** and sign in to Steam. It checks that you own the game, downloads the rest of it and syncs
   your saves.
3. **Turn on both Sense controllers** and press **Play**.

To update or renew (free accounts every 7 days), the Glovebox app shows **Renew**; from source, run `build` and
`install` again. Your game files and saves stay on the headset.

## Known issues

- The game draws 45 frames a second and the headset shows each one twice, so close objects can look doubled at the
  edges when you move your head.
- A few busy scenes run below full speed.
- Left-handed and one-controller play don't work.

More in [docs/KNOWN-ISSUES.md](docs/KNOWN-ISSUES.md). For problems, press **Copy diagnostics** in the Glovebox app and
[open an issue on their repository](https://github.com/glovebox-vr/glovebox/issues/new).
