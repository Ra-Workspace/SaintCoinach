# Saint Coinach

A .NET library written in C# for extracting game assets and reading game assets from **Final Fantasy XIV: A Realm Reborn**, now with support for including the Libra Eorzea database.

## Functionality 
### Fully implemented

* Extraction of files from the game's SqPack files based on their friendly name (or by Int32 identifiers, if preferred).
* Conversion of the game's textures to `System.Drawing.Image` objects.
* Parsing and reading from the game's data files (`*.exh` and `*.exd`).
* Decoding of OGG files stored in the game's pack files (some of the `*.scd`).
* OO-representation of the most important game data.
* Self-updating of the mapping between game data and their OO-representation, in case things move around inside the game's files betwen patches.

### Partially implemented

* Decoding of the string format used by the game. Will return a good string for most queries, but more advanced things like conditional texts are not supported.
* Parsing of model data works for *most* models but is far from complete.
* Inclusion and parsing of data from the Libra Eorzea application.

### To-do

* Support for audio formats other than OGG.


## Usage

### Set-up

**Note:** When building an application using this library make sure to include a copy of [`SaintCoinach/SaintCoinach.History.zip`](/SaintCoinach/SaintCoinach.History.zip) in the application's directory. This should be done automatically if the project is included in the solution and referenced from there.

All important data is exposed by the class `SaintCoinach.ARealmReversed`, so setting up access to it is fairly straightforward.

The following is an example using the game's default installation path and English as default language:

```C#
const string GameDirectory = @"C:\Program Files (x86)\SquareEnix\FINAL FANTASY XIV - A Realm Reborn";
var realm = new SaintCoinach.ARealmReversed(GameDirectory, SaintCoinach.Ex.Language.English);
```

It's that simple. It is recommended, however, to check if the game has been updated since the last time, which is accomplished like this, in this example including the detection of data changes:

```C#
if (!realm.IsCurrentVersion) {
    const bool IncludeDataChanges = true;
    var updateReport = realm.Update(IncludeDataChanges);
}
```

`ARealmReversed.Update()` can also take one additional parameter of type `IProgress<UpdateProgress>` to which progress is reported. The returned `UpdateReport` contains a list of changes that were detected during the update.

### Accessing data

Game files can be access directly through `ARealmReversed.Packs`, game data can be accessed through `ARealmReversed.GameData`.

#### Game Data (`XivCollection`)

Specific collections can be retrieved using the `GetSheet<T>()` method.
**Note:** This only works for objects whose game data files have the same name as the class. This applies for most classes directly inside the `SaintCoinach.Xiv` namespace, so there should be no need to worry about it in most cases.

Special cases are exposed as properties:

* `ENpcs`: This collections contains objects of type `ENpc` that include data of both `ENpcBase` and `ENpcResident`.
* `EquipSlots`: There is no actual data for specific equipment slots in the game data, but having access to them makes things more convenient, so they're available here.
* `Items`: This collection combines both `EventItem` and `Item`.
* `Shops`: This collection contains all types of shops.

The following is a simple example that outputs the name and colour of all `Stain` objects to the console:

```C#
var stains = realm.GameData.GetSheet<SaintCoinach.Xiv.Stain>();

foreach(var stain in stains) {
    Console.WriteLine("#{0}: {1} is {2}", stain.Key, stain.Name, stain.Color);
}
```

## Notes

### State of documentation

Only `SaintCoinach` contains documentation, and even that only in some places (anything directly in the `SaintCoinach` namespace as well as some things in `SaintCoinach.Xiv.*`), everything else is virtually void of documentation.

There should, however, be enough documentation available to know how to use the library for game data, but figuring out the internal workings might prove difficult.

### SaintCoinach.Cmd

The project `SaintCoinach.Cmd` is a very basic console application that can be used to extract various assets.
The following commands are currently supported:

* `lang`: Displays or changes the language used for data files. Valid arguments are `Japanese`, `English`, `German`, `French`. If no argument is supplied the currently used language is shown.
* `raw`: Exports a file from the game assets without any conversions. The argument should be the friendly name of the file.
* `image`: Exports a file from the game assets as a PNG-image. The argument should be the friendly name of the image file.
* `ui`: Exports one or multiple UI icons as PNG-images. The argument can either be the number of a single UI icon, or the first and last number for a range of icons seperated by a space. Valid numbers are in the interval [0, 999999].
* `exd`: Exports all or a specified number of game data sheets as CSV-files. Arguments can either be empty to export all files, or a list of sheet names seperated by whitespace.
* `bgm`: Exports all sound files referenced in the BGM sheet as OGG-files.
* `3d`: Displays 3D objects. Very experimental, further information is going to be omitted here.



### SaintCoinach.Graphics.ViewerEngine

The viewer engine is in its first stages, and I didn't really know much 3D-graphics before starting it, so while it can be used expect strange or completely wrong results. It should also really be updated to use flat shading because different material properties are not available and the lighting makes things look weird.

If that does not scare you away see `SaintCoinach.Cmd` for examples on how to use it, and make sure to copy the project's `Effects/HLSL` directory as well as the `sharpdx_direct3d11_effects_*.dll` libraries to the application directory. The files should get copied automatically if the library project is included in the solution, though.