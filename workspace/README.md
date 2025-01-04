# MinUI CodeBase Analysis

## General
At the highest level, MinUI (much like OnionOS) is not an OS, but rather a UI/configuration platform laid on top of the device's stock OS. This comes with some interesting limitations and benefits. 

- The cool part is that boring things like input, bluetooth/wifi, display drivers come free, we don't have to implement any of those.
- The annoying part is that we're locked into existing architecture/drivers. In the event of something like the RGXX where the manufacturer hasn't extracted all the device's power out of it at launch, we can't throw our own drivers at it.

## Code Prerequisites
It is recommended (but not entirely necessary) to do all development in an x86_64 environment such as common AMD/Intel CPUs as this project relies on some bespoke Docker containers that are only built for one platform. This can be overcome on an M1 Mac by changing some of the Docker run calls to explicitly use the `--platform linux/arm64/v8` Docker flag. But there will be other minor build issues that need to be addressed. For the easiest experience, avoid working on Apple Silicon.

Some CMake library along with zip are required to build/package the project:

```sh
# Install necessary dependencies.
sudo apt update
sudo apt install cmake gcc zip
```

There are some hiccups with the Makefile that make it difficult to build without modification. For starters, the tidy command ignores your PLATFORM flag and assumes you've built for the rg35xxplus and rg40xxcube.


## Build Guide  
Ideally things could be built with a simple 'make' command from the root directory. However, there is a bit of complexity in the build pipelines.

### Make Target Overview

#### name
- Outputs the build target to the command line.

#### setup 
- Removes `build` directory
- Prepares `releases` and `build` directories 
- Creates txt files containing build/readme information required for packaging

#### build
- Builds the toolchain for the specified `$PLATFORM`

#### system
- Makes the specified `$PLATFORM` workspace makefile
- Populates the associated `.build/SYSTEM` directory with build files

#### cores
- Copies libretro (RetroArch) cores from the specified `$PLATFORM` workspace into the associated `.build/SYSTEM` directory

##### *Note: The `trimuismart` and `m17` `$PLATFORM`s have a dependency on `miyoomini` (pico8 core). This means builds will fail if `miyoomini` isn't already built. Consider adding that core directly to the m17 and trimuismart builds.*

#### common
- Calls *build*, *system* and *cores* targets to make files common for all installations

Building for a fresh install should be done in this order: 
```sh

# Start with a clean build directory
make clean 
# Build for tg3040 platform
# Prepares build directories
make setup PLATFORM=tg3040 

# Builds core system files
make common PLATFORM=tg3040 

# Builds device-specific files 
make special PLATFORM=tg3040 

# Cleans up build artifacts
make tidy PLATFORM=tg3040 

# Creates release zips  
make package PLATFORM=tg3040  
```
 ##### *note: Some minor modifications had to be made to the  `tg3040` `makefile.copy` along with the main `makefile` in order to get this working as both were dependent on `miyoomini` and `rg35xx` builds running before it.*
## Code Overview
As mentioned above, we're dealing with a UI layer on top of stock OS here. The code structure is as follows:

```sh
MinUI/
├─ build/
├─ github/
├─ toolchains/
├─ workspace/
makefile
makefile.toolchain
```

### Makefile & Makefile.toolchain
As with any C project, these files lay out the instructions for building the project. The `makefile.toolchain` file is used to clone/build/run the current toolchains container and define environment variables that distinguish the paths of the host machine and toolchain container's workspace.

### Build
This is where your compiled binaries will end up after you've built the project.

### Github
Files necessary to generate/host the readme screenshots.

### Toolchains
The dependent Docker build toolchains for specific devices. They set up a similar Linux environment to that of the host device (install the same libraries) and drop you into a host shell to run your test applications from. For example, `tg3040-toolchain` directs you to Trimui's own [toolchain_sdk_smartpro](https://github.com/trimui/toolchain_sdk_smartpro/releases). Some other devices have required the community to make their own "SDK"/toolchains such as the [RG35XX toolchain](https://github.com/shauninman/union-rg35xxplus-toolchain).

### Workspace
This is where the magic happens, all UI/device specific code is located in this directory.

## Build Overview
The general release of MinUI contains:

```
MinUI-<version>-0-base
# ------------------------------------------------------------------------ #
# - Generally speaking, this contains the contents of build/BASE once
# ‘make’, ‘make special’ and ‘make package' are called.
# ------------------------------------------------------------------------ #
├── Bios # - Empty Bios folders for users to fill in - created from
│   ├── FC
│   ├── GB
│   ├── GBA
│   ├── GBC
│   ├── MD
│   ├── PS
│   └── SFC
├── MinUI.zip  # - Core OS files used for all platforms
│   ├── .system
# Entire build/SYSTEM folder after ‘make tidy’ is run (commits.txt and
# version.txt created).
│   └── .tmp_update
# Entire build/BOOT/common folder. These files come from
# workspace/<device>/install
├── README.txt
├── Roms # - Empty Roms folders for users to fill in - created from
│   ├── Game Boy (GB)
│   ├── Game Boy Advance (GBA)
│   ├── Game Boy Color (GBC)
│   ├── Nintendo Entertainment System (FC)
│   ├── Sega Genesis (MD)
│   ├── Sony PlayStation (PS)
│   └── Super Nintendo Entertainment System (SFC)
├── Saves # - Empty Roms folders for users to fill in - created from
│   ├── FC
│   ├── GB
│   ├── GBA
│   ├── GBC
│   ├── MD
│   ├── PS
│   └── SFC
├── em_ui.sh
# Special device specific install scripts/files folders generated from
# ‘make special’
├── gkdpixel
├── miyoo
├── miyoo354
├── rg35xx
├── rg35xxplus
└── trimui
# ------------------------------------------------------------------------ #
```

The extras release of MinUI contains:

```
MinUI-<version>-0-extras
# ------------------------------------------------------------------------ #
Generally speaking, this contains the contents of build/EXTRAS once 'make'
and 'make package' are called. These are mostly(only?) additional/experimental
emulators (and their associated bios/roms/saves folders), device specific apps.
# ------------------------------------------------------------------------ #
├── Bios
│   ├── GG
│   ├── MGBA
│   ├── NGPC
│   ├── P8
│   ├── PCE
│   ├── PKM
│   ├── SGB
│   ├── SMS
│   └── VB
├── Emus
│   ├── gkdpixel
│   ├── m17
│   ├── magicmini
│   ├── miyoomini
│   ├── my282
│   ├── rg35xx
│   ├── rg35xxplus
│   ├── rgb30
│   ├── tg3040
│   ├── tg5040
│   └── trimuismart
├── README.txt
├── Roms
│   ├── Game Boy Advance (MGBA)
│   ├── Neo Geo Pocket Color (NGPC)
│   ├── Pico-8 (P8)
│   ├── Pokémon mini (PKM)
│   ├── Sega Game Gear (GG)
│   ├── Sega Master System (SMS)
│   ├── Super Game Boy (SGB)
│   ├── Super Nintendo Entertainment System (SUPA)
│   ├── TurboGrafx-16 (PCE)
│   └── Virtual Boy (VB)
├── Saves
│   ├── GG
│   ├── MGBA
│   ├── NGPC
│   ├── P8
│   ├── PCE
│   ├── PKM
│   ├── SGB
│   ├── SMS
│   ├── SUPA
│   └── VB
└── Tools
    ├── gkdpixel
    ├── m17
    ├── magicmini
    ├── miyoomini
    ├── my282
    ├── rg35xx
    ├── rg35xxplus
    ├── rgb30
    ├── tg3040
    │   ├── Bootlogo.pak
    │   │   ├── bootlogo.bmp
    │   │   ├── launch.sh
    │   │   └── original.bmp
    │   ├── Clock.pak
    │   │   ├── clock.elf
    │   │   └── launch.sh
    │   ├── Input.pak
    │   │   ├── launch.sh
    │   │   └── minput.elf
    │   └── Remove Loading.pak
    │       ├── done.png
    │       └── launch.sh
    ├── tg5040
    └── trimuismart
```

## TrimUI Brick a.k.a tg3040
SD Card seems to be mounted to `/mnt/SDCARD`.

### BUILD
Makefile requires Docker to be running, uses container to build.

### Skeleton
- **BASE**: Holds file structure information (bios/roms/saves hierarchy, etc).
- **BOOT**:
  - **Common**: Holds OS booting bash script.
    - `updater.sh`: Checks `proc/cpuinfo` to check what device/platform is running and sets a corresponding `$PLATFORM` local var. Runs corresponding platform script in `/mnt/SDCARD/.tmp_update/`. This is how it remains multiplat I assume.

*Note: Looking at the code, it looks like the Miyoo mini (plus?) and trimuismart use the same CPU architecture denoted by “sun8i”. However, the Miyoo uses an ARM Cortex-A7 dual core, and the trimui smart pro uses an Allwinner A133P (A133 Plus) 1.8GHz.*

### Workspace
- **Trimuismart**
  - **Install**
    - `Boot.sh`: Called from `skeleton/boot/common/updater.sh`. Sends performance information to a CPU path. Uses `MinUI.zip` to update the system if the zip is present.

### Random Findings

#### Colors/Themes
All colors (and other system defaults) are defined in `workspace/all/common/defines.h` and used throughout `workspace/all/minui/minui.c`. Simple modifications to these would allow for quick theme changes.

#### Thumbnail Support
The system supports thumbnails for files and folders. A thumbnail for a file or folder named `NAME.EXT` needs a corresponding `/.res/NAME.EXT.png` that is no bigger than the platform's `FIXED_HEIGHT x FIXED_HEIGHT`.