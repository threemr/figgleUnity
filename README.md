```
$$$$$$$$\$$\                  $$\                 $$$$$$\                         $$\   $$\         $$\  $$\              
$$  _____\__|                 $$ |               $$  __$$\                        $$ |  $$ |        \__| $$ |             
$$ |     $$\ $$$$$$\  $$$$$$\ $$ |$$$$$$\        $$ /  \__$$$$$$\  $$$$$$\        $$ |  $$ $$$$$$$\ $$\$$$$$$\  $$\   $$\ 
$$$$$\   $$ $$  __$$\$$  __$$\$$ $$  __$$\       $$$$\   $$  __$$\$$  __$$\       $$ |  $$ $$  __$$\$$ \_$$  _| $$ |  $$ |
$$  __|  $$ $$ /  $$ $$ /  $$ $$ $$$$$$$$ |      $$  _|  $$ /  $$ $$ |  \__|      $$ |  $$ $$ |  $$ $$ | $$ |   $$ |  $$ |
$$ |     $$ $$ |  $$ $$ |  $$ $$ $$   ____|      $$ |    $$ |  $$ $$ |            $$ |  $$ $$ |  $$ $$ | $$ |$$\$$ |  $$ |
$$ |     $$ \$$$$$$$ \$$$$$$$ $$ \$$$$$$$\       $$ |    \$$$$$$  $$ |            \$$$$$$  $$ |  $$ $$ | \$$$$  \$$$$$$$ |
\__|     \__|\____$$ |\____$$ \__|\_______|      \__|     \______/\__|             \______/\__|  \__\__|  \____/ \____$$ |
            $$\   $$ $$\   $$ |                                                                                 $$\   $$ |
            \$$$$$$  \$$$$$$  |                                                                                 \$$$$$$  |
             \______/ \______/                                                                                   \______/ 
                                                                                                                          
```
_(Using font: "Big Money-nw")_

# ASCII banner generation for .NET, modified to work with Unity 2018.3.10f1

## Basic Usage.

```c#
using Figgle;
// ...

Debug.Log(
    FiggleFonts.Standard.Render("Hello, World!"));

// Note the pattern is: FiggleFonts.[font name].Render(string s);

```

Produces...

```
  _   _      _ _         __        __         _     _ _
 | | | | ___| | | ___    \ \      / /__  _ __| | __| | |
 | |_| |/ _ \ | |/ _ \    \ \ /\ / / _ \| '__| |/ _` | |
 |  _  |  __/ | | (_) |    \ V  V / (_) | |  | | (_| |_|
 |_| |_|\___|_|_|\___( )    \_/\_/ \___/|_|  |_|\__,_(_)
                     |/
```

The library bundles 265 [FIGlet](http://www.figlet.org/) [fonts](http://www.jave.de/figlet/fonts.html). You can add your own if that's not enough! 

## Installation

### Background

_Disclaimer:_ this is a work-around meant for our specific use-case and with that particular version of Unity in mind. It works and it was tested for our purposes, but this is NOT meant to be a full replacement for the Figgle library. **#TODO:** unit-test the changes with the tests in the original Figgle the package.

Unity does not work too well with NuGet packages, because it regenerates the Packages folder automatically. There are [workarounds](https://www.what-could-possibly-go-wrong.com/unity-and-nuget/), but an advantage of having a pure C# package available in source code is that we can integrate it directly into our Unity project. The caveat is that Unity uses a [specific version](https://docs.unity3d.com/2018.3/Documentation/Manual/CSharpCompiler.html) of C#/.NET, which may or may not be fully compatible with the version used to build the package. (See how to [update the .NET version](https://docs.microsoft.com/en-us/visualstudio/gamedev/unity/unity-scripting-upgrade) in Unity 2017.1 or above; 2018.2+ recommended.) This is the case with the standard version of [Figgle](https://www.nuget.org/packages/Figgle/): although its documentation says it targets .NET Standard 1.3, the compiler used by **Unity 2018.3.10f1** complained about certain constructs, even though Visual Studio considered them perfectly valid.

This fork reformatted a few aspects (mostly stylistic) in the library to a version that Unity accepted, which generally meant to re-write a few lines with "an older style" of C#. 
* The only expected "potentially unsafe" change was the removal of [`Nullable<T>`](https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2012/2cf62fcy(v=vs.110)) checks on optional parameters (e.g. `myFunction(MyType? optParam = null)`. Note the question mark `?` after the name of the type.).

* A couple of [local static methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions) had to be promoted into a higher scope within the class. Unity complained about the use of `static`, which is likely because [it requires C# 8.0 and later](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#local-function-syntax). Without analyzing the code too deeply, we didn't think it was a good idea to remove the `static` designation, but moving the methods to make them regular `private static` methods in the class didn't seem to cause undesirable effects.

* The use of the [`using statement`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-statement) to handle the font database (a zip file). The original code uses a C# 8 flavor which eliminates the need of code blocks and braces to delimit the scope of that `using` statement. This was replaced with the older style with braces.

* The original code assumes the font database is bundled within a DLL assembly, and handles it like so. Since we are using the source code directly, we don't have that DLL. Instead, the zip file is moved to a location recognized by Unity (and left untouched when preparing its own bundle; more on that below), and the stream is handled directly as part of the local filesystem.

### How to install

1) Download or clone this repository. The only folder that is needed for Unity is **the Figgle folder**. Copy this folder from where you cloned it, into your **Assets** folder. Although it is not mandatory, if you follow the convention of creating an **Assets/Scripts** folder, that's where you should copy the Figgle folder. For the rest of the instructions, we will assume you copied the folder in there and its path will be: **Assets/Scripts/Figgle**

2) Create an **[Assets/StreamingAssets](https://docs.unity3d.com/2018.3/Documentation/Manual/StreamingAssets.html)** folder. The name is case-sensitive. It is also important to use this folder to make sure what we put in there is preserved in the final bundle. From Unity's documentation:
   > Any files placed in a folder called StreamingAssets (case-sensitive) in a Unity project _will be copied verbatim_ to a particular folder on the target machine. You can retrieve the folder using the `Application.streamingAssetsPath` property. 

3) **Move** the font database file **Assets/Scripts/Figgle/Fonts.zip** into **Assets/StreamingAssets/Figgle.zip**.

4) Enjoy! Remember to add `using Figgle;` on your Unity scripts, to access this library's namespace.

### Tips and known issues.

* If you want to quickly visualize how a certain text looks like without having to write multiple tests in C#, try using the excellent online tool [Text to ASCII Art Generator (TAAG)](https://www.patorjk.com/software/taag), which gives you a button to "test all" fonts at once.

* Since the console provided within Unity does not use monospaced fonts, the results in there won't look correct. However, note that Debug.Log and its variants will also write to the [log files](https://docs.unity3d.com/2018.3/Documentation/Manual/LogFiles.html). That's one example of where Figgle excels; instead of getting lost in a sea of text, we used Figgle to visually separate sections of the text written to the logs. For instance, this is an excerpt of the start of a build report: 
```bash
 ____  _____  ___  ____     ____  ____  _____  ___  ____  ___  ___
(  _ \(  _  )/ __)(_  _)___(  _ \(  _ \(  _  )/ __)( ___)/ __)/ __)
 )___/ )(_)( \__ \  )( (___))___/ )   / )(_)(( (__  )__) \__ \\__ \
(__)  (_____)(___/ (__)    (__)  (_)\_)(_____)\___)(____)(___/(___/

<size=25><color=navy>Post-process</color></size> (Start: 1/25/2021 4:21:54 AM, CB order: 9999)
PreBuildConfigurator stage 'Post-process' for target 'iOS' at path '[project path]'
<Post-build> Found 1 GameObjects in the scene.
Object <b>'Main Camera'</b>, (id: 0xFFFF8EE0, -28960):  Active status (self: True, hierachy: True)

-------------------------------------------------------------------------------
Build Report
Uncompressed usage by category (Percentages based on user generated assets only):
Textures               183.9 mb  88.2%
Meshes                 18.1 mb   8.7%
Animations             0.0 kb    0.0%
Sounds                 358.2 kb  0.2%
Shaders                1.0 mb    0.5%
Other Assets           614.1 kb  0.3%
Levels                 322.8 kb  0.2%
Scripts                445.7 kb  0.2%
Included DLLs          3.7 mb    1.8%
File headers           33.9 kb   0.0%
Total User Assets      208.5 mb  100.0%
Complete build size    1065.5 mb
...

```
* As a compromise for the Unity console, note you can also [use some rich text](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/StyledText.html). You can see that on the example above. This renders colored, bigger text within the console (for the same purposes of visual separation). 

* Other uses. As long as you make sure you **`use a monospaced font`**, the results from Figgle will look alright anywhere else in your application.


## Other examples

Font `FiggleFonts.Graffiti`:

```
 ____ ___      .__  __                                 __           ._.
|    |   \____ |__|/  |_ ___.__. _______  ____   ____ |  | __  _____| |
|    |   /    \|  \   __<   |  | \_  __ \/  _ \_/ ___\|  |/ / /  ___/ |
|    |  /   |  \  ||  |  \___  |  |  | \(  <_> )  \___|    <  \___ \ \|
|______/|___|  /__||__|  / ____|  |__|   \____/ \___  >__|_ \/____  >__
             \/          \/                         \/     \/     \/ \/
```

Font `FiggleFonts.ThreePoint`:

```
| | _ ._|_    _ _  _|  _|
|_|| || |\/  | (_)(_|<_\.
         /               
```

Font `FiggleFonts.Ogre`:

```
             _ _                          _           _ 
 /\ /\ _ __ (_) |_ _   _   _ __ ___   ___| | _____   / \
/ / \ \ '_ \| | __| | | | | '__/ _ \ / __| |/ / __| /  /
\ \_/ / | | | | |_| |_| | | | | (_) | (__|   <\__ \/\_/ 
 \___/|_| |_|_|\__|\__, | |_|  \___/ \___|_|\_\___/\/   
                   |___/                                
```

Font `FiggleFonts.Rectangles`:

```
                                            __ 
 _____     _ _                      _      |  |
|  |  |___|_| |_ _ _    ___ ___ ___| |_ ___|  |
|  |  |   | |  _| | |  |  _| . |  _| '_|_ -|__|
|_____|_|_|_|_| |_  |  |_| |___|___|_,_|___|__|
                |___|                          
```

Font `FiggleFonts.Slant`:

```
   __  __      _ __                           __        __
  / / / /___  (_) /___  __   _________  _____/ /_______/ /
 / / / / __ \/ / __/ / / /  / ___/ __ \/ ___/ //_/ ___/ / 
/ /_/ / / / / / /_/ /_/ /  / /  / /_/ / /__/ ,< (__  )_/  
\____/_/ /_/_/\__/\__, /  /_/   \____/\___/_/|_/____(_)   
                 /____/                                   
```
