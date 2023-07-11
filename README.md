# Makedml
A simple build system for System Shock 2 mods.

# Features

- Automatic creation of DML headers and fingerprints
- Automatic merging of different feature folders, allowing for different mod features to be added/removed as needed
- Ignoring/removing features from builds by prepending feature folder or file names with #
- Ability to create different versions of mods, additionally with a comon set of features for all versions
- Automatically zip mod versions ready for distribution using 7-zip

# How to use

MakeDML is a command line program that works in one of 3 ways, based on command line arguments. Run with no arguments to see a rundown of syntax, or see the Command Line Examples section

## Feature Mode (-f)

This mode is the simplest of the available modes.

Lets say we have a sample mod with the following folder structure:

```
- My System Shock 2 Mod
  - src <- source files for the mod
    - feature1
      - gamesys.dml
    - feature2
      - gamesys.dml
      - medsci1.mis.dml
  - out <- output folder for the project
```

The structure of each subfolder of ```src``` will be copied into ```out``` in sequence.
Files that exist in multiple folders will be concatenated together with a new line.

This will create a "combined" mod in the ```out``` folder.

```out``` will contain ```out/gamesys.dml``` (containing the contents of both ```gamesys.dml``` files from feature1 and feature2 and a ```DML1``` header)
as well as ```out/medsci1.mis.dml```, which will contain a ```DML1``` header, as well as a [DML Fingerprint](https://www.systemshock.org/index.php?topic=6821.0) for medsci1.mis.
Additionally, if specified on the command line, ```medsci1.mis.dml``` may also contain DML fingerprints for secmod or SCP.
See the ```headers``` folder to add fingerprints for other mods or fan missions.

## Version Mode (-v)

Each subfolder of ```src``` will be made independently into a separate version of the project,
as if feature mode was run for that subfolder.

Lets say we have a sample mod with the following folder structure:

```
- My System Shock 2 Mod
  - src <- source files for the mod
    - version1
      - feature1
        - gamesys.dml
      - feature2
        - gamesys.dml
        - medsci1.mis.dml
    - version2
      - feature1
        - sq_scripts
          - scripts.nut
        - gamesys.dml
      - feature2
        - gamesys.dml
        - medsci1.mis.dml
  - out <- output folder for all built versions, with each version appearing in a subfolder
```

This will make 2 versions, ```out/version1``` and ```out/version2```, which will both have been built as if feature mode was run for each version.

## Zip Mode (-z or -Z, then -f or -v)

This is designed to be run after running either Feature mode or Version mode for a mod.

Running -z with -f will create a zip for the mod at ```zips/<modname>.7z```

Running -z with -v will create a zip for each version of the mod at ```zips/<modname> - <version name>.7z```

Running -Z with -v will create a "big zip", which will create a single zip for the mod at ```zips/<modname>.7z``` with each version being added as a subfolder

```<modname>``` is specified in the command line, ```<version name>``` is based on the names of the folders for each version.

# Ignoring Features, Versions and Files

Prepending any filename in the ```src``` folder with ```#``` will exclude it from builds

Prepending any feature or version folder in the ```src``` folder with ```#``` will exclude it from builds

# Special Versions

if a version is named ```$common```, it will be included in all other versions when built. This allows having some common files that will exist in every version

if a version is named ```$core```, it will be included in all other versions when built, and will additionally be built as it's own version.
This allows creating a "basic" version of a mod with only core features, plus additional other versions containing the core features as well as their own features

Any version with a name prepended with ```$``` will not have the contents of the ```$common``` or ```$core``` versions added to it. This can be used to make special "addon" files that work with all versions of a mod.

When building zip files, the ```<version name>``` used for the ```$core``` version can be specified on the command line (see the command line examples section)

If more specialised version functionality is required (such as having features available in certain versions but not others), this is not supported, but can be done using symbolic links or junctions.

# Command Line Examples

For ease of use, it's recommended to create a ```make.cmd``` file in the root of your project to enable easy building. The following ```make.cmd``` examples assume makedml is installed to the ```build``` folder within the project.

This will make all features in the ```src``` folder in the ```out``` folder, adding fingerprints for vanilla and SCP to all mission DMLs
```
call build/makedml.cmd -f "%~dp0\src" "%~dp0\out" "vanilla" "scp"
```

This will make all features in all versions in the ```src``` folder in the ```out``` folder, with each version being added as a subfolder, adding fingerprints for vanilla and SCP to all mission DMLs
```
call build/makedml.cmd -v "%~dp0\src" "%~dp0\out" "vanilla" "scp"
```

This will zip the "feature mode" build in the ```out``` folder into a zip called ```"Randomiser 1.0RC.7z"```
```
call build/makedml.cmd -z -f "%~dp0\out" "Randomiser 1.0RC"
```

This will zip the "version mode" build in the ```out``` folder into zips called ```"zips/Randomiser 1.0RC - <version name>.7z"```, and the ```$core``` version will be in a zip named ```"zips/Randomiser 1.0RC - Lite.7z"```
```
call build/makedml.cmd -z -v "%~dp0\out" "Randomiser 1.0RC" "Lite"
```

This will zip the "version mode" build in the ```out``` folder into a big zip called ```"Randomiser 1.0RC.7z"```, and the ```$core``` version will be in a subfolder named ```"Lite"```
```
call build/makedml.cmd -Z -v "%~dp0\out" "Randomiser 1.0RC" "Lite"
```
