# resonite-osx
Notes on running Resonite on macOS

## Does it work?

After a bit of effort, yes. There are issues with video players (see below), and after a while, the server just seems to give up. (If you try spawning Resonite Essentials > Accessibility > Mute Helper Context Menu). It sems to be related to the repeated

```
Assembly Awwdio is not a data model assembly for this world, cannot decode type: Awwdio.AudioRolloffCurve
Failed to decode type: [ProtoFluxBindings]FrooxEngine.ProtoFlux.Runtimes.Execution.Nodes.ValueInput<[Awwdio]Awwdio.AudioRolloffCurve>
Unsupported worker type, couldn't load: Type: , Typename: [ProtoFluxBindings]FrooxEngine.ProtoFlux.Runtimes.Execution.Nodes.ValueInput<[Awwdio]Awwdio.AudioRolloffCurve>, LegacyTypes: False
```
in the server logs as the clients crash out with
```
❌00:28:13.181 (FPS: 60):	Exception in running sync loop:

System.ArgumentException: Invalid data model type: Awwdio.AudioRolloffCurve
   at FrooxEngine.TypeManager.EncodeType(BinaryWriter writer, Type type, Boolean isDeconstructing) in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Controllers\TypeManager.cs:line 204
   at FrooxEngine.TypeManager.EncodeType(BinaryWriter writer, Type type, Boolean isDeconstructing) in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Controllers\TypeManager.cs:line 197
   at FrooxEngine.SyncBagBase`2.InternalEncodeDelta(BinaryWriter writer, BinaryMessageBatch outboundMessage) in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Sync Members\SyncBagBase.cs:line 408
   at FrooxEngine.SyncElement.EncodeDelta(BinaryWriter writer, BinaryMessageBatch outboundMessage) in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Base Classes\SyncElement.cs:line 370
   at FrooxEngine.SyncController.CollectDeltaMessages() in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Controllers\SyncController.cs:line 79
   at FrooxEngine.SessionSyncManager.GenerateDeltaBatchAndSend() in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Network Session\SessionSyncManager.cs:line 450
   at FrooxEngine.SessionSyncManager.SyncLoop() in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Network Session\SessionSyncManager.cs:line 289
   at FrooxEngine.SessionSyncManager.SyncLoopRunner() in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Network Session\SessionSyncManager.cs:line 241

   at System.Environment.get_StackTrace()
   at Elements.Core.UniLog.Error(String message, Boolean stackTrace) in D:\Workspace\Everion\FrooxEngine\Elements.Core\UniLog.cs:line 78
   at FrooxEngine.SessionSyncManager.SyncLoopRunner() in D:\Workspace\Everion\FrooxEngine\FrooxEngine\Data Model\Network Session\SessionSyncManager.cs:line 241
   at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state)

```

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



 - **assimp**: https://github.com/Yellow-Dog-Man/assimp/actions/workflows/ccpp.yml
 - **brotli**: https://github.com/TayIorRobinson/ydms-brotli/actions/workflows/ydms-build.yaml
 - **msdfgen**: https://github.com/TayIorRobinson/ydms-msdfgen/actions/workflows/ydms-build.yaml
 - **opus**: https://github.com/TayIorRobinson/ydms-opus/actions/workflows/build-ydms.yml
 - **soundpipe**: https://github.com/TayIorRobinson/ydms-soundpipe/actions/workflows/build-macos.yml
 - **libphonon** (Steam Audio): https://github.com/ValveSoftware/steam-audio/releases/download/v4.7.0/steamaudio_4.7.0.zip 
 - **Microsoft.Extensions.ObjectPool**: Grab 6.0.36 from NuGet.
 - **Discord Game SDK**: https://discord.com/developers/docs/developer-tools/game-sdk

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

### Example csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <ResonitePath>$(SolutionDir)\Resonite</ResonitePath>
    </PropertyGroup>
    
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net9.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
        <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
        <AppendRuntimeIdentifierToOutputPath>false</AppendRuntimeIdentifierToOutputPath>
        <OutputPath>$(ResonitePath)</OutputPath>
    </PropertyGroup>
    

    <ItemGroup>
        <Reference Include="Elements.Assets">
            <HintPath>$(ResonitePath)\Elements.Core.dll</HintPath>
        </Reference>

        <Reference Include="Elements.Core">
            <HintPath>$(ResonitePath)\Elements.Core.dll</HintPath>
        </Reference>

        <Reference Include="FrooxEngine">
            <HintPath>$(ResonitePath)\FrooxEngine.dll</HintPath>
        </Reference>

        <Reference Include="FrooxEngine.Store">
          <HintPath>ban\Debug\net9.0\FrooxEngine.Store.dll</HintPath>
        </Reference>

        <Reference Include="POpusCodec">
          <HintPath>ban\Debug\net9.0\POpusCodec.dll</HintPath>
        </Reference>

        <Reference Include="ProtoFlux.Nodes.Core">
            <HintPath>$(ResonitePath)\ProtoFlux.Nodes.Core.dll</HintPath>
        </Reference>

        <Reference Include="ProtoFlux.Nodes.FrooxEngine">
            <HintPath>$(ResonitePath)\ProtoFlux.Nodes.FrooxEngine.dll</HintPath>
        </Reference>

        <Reference Include="ProtoFluxBindings">
            <HintPath>$(ResonitePath)\ProtoFluxBindings.dll</HintPath>
        </Reference>

        <Reference Include="Rug.Osc">
            <HintPath>$(ResonitePath)\Rug.Osc.dll</HintPath>
        </Reference>

        <Reference Include="SkyFrost.Base">
            <HintPath>$(ResonitePath)\SkyFrost.Base.dll</HintPath>
        </Reference>

        <Reference Include="SkyFrost.Base.Models">
            <HintPath>$(ResonitePath)\SkyFrost.Base.Models.dll</HintPath>
        </Reference>
    </ItemGroup>

    <ItemGroup>
      <Folder Include="libraries\" />
    </ItemGroup>

    <ItemGroup>
        <None Update="libraries\discord_game_sdk.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>discord_game_sdk.dylib</TargetPath>
        </None>

        <None Update="libraries\FreeImage.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>FreeImage.dylib</TargetPath>
        </None>
        <None Update="libraries\libbrolib.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>brolib_x64.dylib</TargetPath>
        </None>
        <None Update="libraries\libfreetype6.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>libfreetype6.dylib</TargetPath>
        </None>
        <None Update="libraries\libmsdfgen.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>libmsdfgen.dylib</TargetPath>
        </None>
        <None Update="libraries\libopus.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>opus.dylib</TargetPath>
        </None>
        <None Update="libraries\libphonon.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>libphonon.dylib</TargetPath>
        </None>
        <None Update="libraries\libsoundpipe.dylib">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>libsoundpipe.dylib</TargetPath>
        </None>
        <None Update="libraries\Steamworks.NET.dll">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
            <TargetPath>Steamworks.NET.dll</TargetPath>
        </None>

        
      
    </ItemGroup>

    <ItemGroup>
      <PackageReference Include="Lib.Harmony" Version="2.4.1" />
      <PackageReference Include="Microsoft.Extensions.ObjectPool" Version="6.0.36" />
    </ItemGroup>

</Project>

```
