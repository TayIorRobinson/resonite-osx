# resonite-osx
Notes on running Resonite on macOS

## Does it work?

After a bit of effort, yes. There are issues with video players (see below), and after a while, the server just seems to give up. FrooxEngine doesn't crash, but stops responding to network requests and all users timeout disconnect.

<img width="5344" height="3054" alt="image" src="https://github.com/user-attachments/assets/ca83a830-ca1a-43a1-997b-e76df1827fe5" />


## Obtaining Resonite

To run Resonite, you will of course, require a copy of Resonite itsself. I do not have access to the headless server software (If you do, it will likely be easier to get this working, but may differ from what I'm doing here.), so I had to fashion my own. I don't feel like giving out a copy of this (as y'know, YDMS ask $10/mo to run a headless and I don't want to lower the effort barrier with a janky hack), but if you have any level of skill in C#, it shouldn't be too hard to initialize FrooxEngine, setup Userspace & call `RunUpdateLoop` repeatedly, and if you get stuck, C# decompilers will make short work of Renderite.Host.exe.

If you have a Windows machine, you can just copy the files across, but if you have Steam installed on your Mac, you can do the following

1. Check https://steamdb.info/app/2519830/depots/ for the latest Depot ID  (its the one thats not `Depot from 228980` and runs on Windows & Linux. Should be about 1.8GB)
2. Open `steam://open/console` into a browser
3. in the Steam console window enter `download_depot 2519830 DEPOT_ID`.
4. Once done, youll get a message like `Depot download complete : "/Users//Library/Application Support/Steam/Steam.AppBundle/Steam/Contents/MacOS\steamapps\content\app_2519830\depot_2519832"`


## Replacing Libraries

Resonite uses a few native libraries, that are compiled specifically for Windows x64. We'll need to replace these with macOS ARM64 variants.

FWIW: A lot of libraries are common, and may already be installed via Homebrew. If you set the envvar `DYLD_LIBRARY_PATH=/opt/homebrew/lib`, you'll be able to get around recompiling these as they'll already be present from Homebrew. You'll at least need FreeType installed from Homebrew.




**Steam Audio**: From https://github.com/ValveSoftware/steam-audio/releases/download/v4.7.0/steamaudio_4.7.0.zip grab `lib/osx/libphonon.dylib`

**Microsoft.Extensions.ObjectPool**: Grab 6.0.36 from NuGet.

Other libraries need to be compiled from source. There are prebuilt binaries in this repo.

### `// TODO: don't just dump binaries on GitHub`
 - **msdfgen**: https://github.com/TayIorRobinson/ydms-msdfgen/actions/workflows/ydms-build.yaml

### Opus

I couldn't get Opus support to work. It kept crashing with an invalid argument error. Here I'll provide a Harmony patch that simply disables error checking.

This seems to break videoplayers, but voice still works.

```csharp
[HarmonyPatch]
class NoOpusPatch
{
    static MethodBase TargetMethod()
    {
        var type = AccessTools.TypeByName("POpusCodec.Wrapper");
        return AccessTools.Method(type, "HandleStatusCode");
    }

    static bool Prefix(object __instance) {
        Console.WriteLine("POpusCodec.Wrapper#HandleStatusCode");
        return false;
    }

}
```
(To use Harmony patches, simply include Harmony as a library from NuGet and then after initialising the engine: )
```csharp
var harmony = new Harmony("guillotine.patch");
harmony.PatchAll();
```
