This dirrectory has been udated to include Version 3.4 RomWBW

# Firmware
This directory contains firmware source and binaries for the board. As of now, it only contains changes to RomWBW.

## RomWBW changes
These changes apply to RomWBW 3.4.0 (the current `live` branch at time of cloning). The config **.asm** should be placed in `Source/HBIOS/Config`. A prebuilt ROM image is provided. To build this image under Windows, run `BuildShared`, then `BuildROM RCZ80 pd1024` (both from `Source`).
