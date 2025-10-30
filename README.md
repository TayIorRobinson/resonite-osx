# resonite-osx
Notes on running Resonite on macOS

## Does it work?

<img width="5344" height="3054" alt="image" src="https://github.com/user-attachments/assets/ca83a830-ca1a-43a1-997b-e76df1827fe5" />


## Obtaining Resonite

To run Resonite, you will of course, require a copy of Resonite itsself. I do not have access to the headless server software (If you do, it will likely be easier to get this working, but may differ from what I'm doing here.), so I had to fashion my own. I don't feel like giving out a copy of this (as y'know, YDMS ask $10/mo to run a headless and I don't want to lower the effort barrier with a janky hack), but if you have any level of skill in C#, here's the one-liner: `await new StandaloneFrooxEngineRunner(new OSXSystemInfo()).Initialize(LaunchOptions.GetLaunchOptions(args));`

If you have a Windows machine, you can just copy the files across, but if you have Steam installed on your Mac, you can do the following

1. Check https://steamdb.info/app/2519830/depots/ for the latest Depot ID  (its the one thats not `Depot from 228980` and runs on Windows & Linux. Should be about 1.8GB)
2. Open `steam://open/console` into a browser
3. in the Steam console window enter `download_depot 2519830 DEPOT_ID`.
4. Once done, youll get a message like `Depot download complete : "/Users//Library/Application Support/Steam/Steam.AppBundle/Steam/Contents/MacOS\steamapps\content\app_2519830\depot_2519832"`


## Replacing Libraries

Resonite uses a few native libraries, that are compiled specifically for Windows x64. We'll need to replace these with macOS ARM64 variants.

FWIW: A lot of libraries are common, and may already be installed via Homebrew. If you set the envvar `DYLD_LIBRARY_PATH=/opt/homebrew/lib`, you'll be able to get around recompiling these as they'll already be present from Homebrew.



 - [x] **assimp**: https://github.com/Yellow-Dog-Man/assimp/actions/workflows/ccpp.yml (build expired, so just grab from Homebrew in the meantime)
 - [x] **brotli**: https://github.com/TayIorRobinson/ydms-brotli/actions/workflows/ydms-build.yaml
 - [x] **msdfgen**: https://github.com/TayIorRobinson/ydms-msdfgen/actions/workflows/ydms-build.yaml
 - [x] **opus**: https://github.com/TayIorRobinson/ydms-opus/actions/workflows/build-ydms.yml
 - [x] **soundpipe**: https://github.com/TayIorRobinson/ydms-soundpipe/actions/workflows/build-macos.yml
 - [x] **libphonon** (Steam Audio): https://github.com/ValveSoftware/steam-audio/releases/download/v4.7.0/steamaudio_4.7.0.zip 
 - [x] **Microsoft.Extensions.ObjectPool**: Grab 6.0.36 from NuGet.
 - [x] **Discord Game SDK**: https://discord.com/developers/docs/developer-tools/game-sdk
 - [x] **Steamworks.NET**: (just needs to be rebuilt targeting arm64, or grab from this repo)
 - [ ] **FreeImage**: (you can get from Homebrew for the time being)
 - [ ] **FreeType**: (you can get from this repo for the time being)
 - [ ] **pdfium**: (i didn't bother - its not really needed)
 - [ ] **unknown unknowns**: (there are other so & dll files in the resonite folder but i dont know what they do)


### Opus

Opus relies on use of varargs, and .NET P/Invoking functions that use varargs is broken on ASi. You'll need to patch P-Opus to get around this. Or just use the precompiled version in this repo.



### Awwdio

I had weird crashing (try and spawn Resonite Essentials > Accessibility > Context Menu Mute Helper) and adding this load bearing Console.WriteLine fixed it (???????????????????)

```csharp
Console.WriteLine(Awwdio.AudioRolloffCurve.Linear);
```
