# mame2003-plus - AtGames Legends Ultimate Mod

Current release is v1.1.0.  It can be downloaded from [here](https://github.com/gm2552/mame2003-plus-libretro/releases/download/1.1.0/mame2003_plus_libretro_alu_1.1.0.zip).

## Acknowledgements 
Port of MAME 0.139 for libretro,.

Includes mods specific to the AtGames Legends Ultimate gaming console.  Forked from original source found at https://github.com/libretro/mame2003-libretro

## Rationale

Generally speaking, the goals of this modified core can certainly be accomplished by other means (however, some cannot).  The ALU (AtGame Ultimate Legends) machine packages games using the squashfs format with and extension of .UCE.  Inside that UCE are resources required to launch the emulator and load those resources.  The emulator is launched via an exec script, and that script can be modified to do almost anything you would like.  However, most users generating UCE files do not have tooling (or know how to use tools) to modify the finer details of the UCE.

The philosophy of this core is to use the ROM Zip as a universal packaging medium that contains all the resources and logic to load the required resources of a game.  You could just as easily package these resources in other locations in the UCE and use the exec script to relocate them appropriately, but again most users do not have the necessary skill set to build such a custom UCE.  Using the ROM Zip as the packager, a modified ROM Zip can be created using somewhat intuitive tooling that is available on almost any platform such as WinZip, WinRar, and P7Zip.  This allows generally available tooling like the [AtGame AddOn Tool](https://github.com/FalkensMaze1983/ultimate_addon/tree/master/AddOn_tool) to still be used to build the UCE without custimation to the UCE packaging tool.

## ROM Zips

ROM zips are defined here as the zip files that contain the necessary ROM image files to run a game.  If you have ever used MAME, you are probably VERY familiar with ROM sets and the ZIP files in those set used to lanch games.  Those zips files are the ROM Zips.  ROM zips should not be confused with other zip files used in MAME that may the same name as the ROM Zips.  Examples are the `samples` zip files that generally hold .wav files for game sound effects.

In many of the mods below, additional files and folders are added to the ROM zips.  The tools and methods to add files to the ROM Zips are out scope of this document, but the processes can be easily accoplished using generally available software like WinZip, WinRar, and P7Zip. 

## Mods

### Config Saves

Games settings which include controler configured, high scores, and nvram files are persisted into the save partition of the UCE file.  These settings will persist between power cycling the console.

### Samples Support

Samples are supported by adding a game's samples zip file into the ROM Zip in a folder named `samples` (the directory name is case sensitive).  For example, take the game Zaxxon whose ROM Zip file name is `zaxxon.zip` and whose samples file name is also named `zaxxon.zip` (typically samples zip files have the same name as the ROM Zip).  In ROM zip file, you would create a new folder named samples and add the samples `zaxxon.zip` to that folder.  Below is an example of the contents of the ROM Zip:

```
Zaxxon.zip
   - zaxxon.1
   - zaxxon.2
   .
   .
   .
   - zaxxon.u98
   - samples
     - zaxxon.zip
```

### Prebuilt NVRAM Support

Generally games create and modify .nv (nvram) files as needed without issue.  However some games require special sequences to initial these files correctly.  For example, running Simpsons Bowling creates nv file, but a user must go through a series of save states and game resets (the F2 and F3 sequense) to properlly initialize the file so the game loads successfully.  Executing these types of sequense may be very difficult if not impossibly on the Legends console.  However, if you have created a working .nv file using an emulator like Mame or advmame, you can add that .nv file to the ROM zip and have the core explicitly use that .nv file (as opposed to the core creating a new .nv file from scratch).

Similar to adding samples to the ROM zip, custom .nv files are added to the ROM Zip in a folder named `nvram` (case sensitive).  Using the Simpsons Bolwing example above, you would add the pre-created simpbowl.nv file to the ROM zip into the `nvram` folder.  Below is an example of the ROM Zip layout:

```
simpbowl.zip
   - 999a01.73
   - simpbowl.25c
   - zaxxon.u98
   - nvram
     - simpbowl.nv
```

### CHD Support

Some games required an external image file that contains required content for the game to load and run.  CHD files are supported similarly to samples by placing the game's chd file into the ROM Zip in a folder named `chd` (case sensitive).  The core extracts the CHD file and places it into the console /tmp folder and the core loads the CHD content from that directory.  

*NOTE:* CHD files can be very large (sometimes 1 or many gigabytes).  There is limited storage on the console, so it very possible that games with CHD files larger that a couple of hundred megabytes may not load due to insufficient storage.  To be a good neighbor, the core deletes the CHD file from the /tmp directory when a game exists.  Below is an example of the ROM Zip layout of the Simpsons Bowling game using an embedded CHD file:

```
simpbowl.zip
   - 999a01.73
   - simpbowl.25c
   - zaxxon.u98
   - nvram
     - simpbowl.nv
   - chd
     - simpbowl.chd
```

### HiScore.Dat

The hiscore.dat database is needed by some games to properly save high score data. Normally the core will create a default hiscore.dat upon first launch so nothing is needed on the user's part.  However, if you want to use a custom hiscore.dat, such a file can be included in the zip file, either at the root directory or inside a subfolder.  Again this is rarely needed.

Below is an example:

```
Zaxxon.zip
   - zaxxon.1
   .
   .
   - hiscore.dat
```

*NOTE:* Some instructions for making UCE packages talk about creating an empty hiscore.dat in the roms directory.  This approach is not compatible with this core.  Having a hiscore.dat in the roms directory actually prevents samples/nvram etc. packaged inside the zip file from working.

### Cheats

Some games will allow for configured cheats.  To enable cheats, you will need to include the cheat.dat file into your zip.  The cheat.dat file can be placed at the root directory of the zip, or inside a subfolder.  Below is an example:

```
1941.zip
   - 41_09.rom
   .
   .
   - cheat.dat
```

The cheat.dat file has to be in the "old" format as used by MAME prior to version 0.126.  Below is an example:

```
; [ 1941 - Counter Attack (World) ]
:1941:00000000:FF0D59:00000009:FFFFFFFF:Infinite Credits
:1941:00000000:FF9A64:00000002:FFFFFFFF:Invincibility - 1st Side Fighter
:1941:00010000:FF9AE4:00000002:FFFFFFFF:Invincibility - 1st Side Fighter (2/2)
:1941:00000000:FF9964:00000002:FFFFFFFF:Invincibility - 2nd Side Fighter
:1941:00010000:FF99E4:00000002:FFFFFFFF:Invincibility - 2nd Side Fighter (2/2)
.
.
```

Once inside game, go to Adv Config and enable cheats. Game will reset to make the selected cheats take effect.

