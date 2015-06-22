# Introduction #

47loader supports several different timings that you can select at assemble time. You can thus customize the loader for the speed and loading sound that you want.

# Available timings #

| **Name** | **Pilot pulse length** | **Zero pulse length** | **One pulse length** | **Percentage of ROM loader speed** | **Remarks** |
|:---------|:-----------------------|:----------------------|:---------------------|:-----------------------------------|:------------|
| FAST     | 1710T                  | 475T                  | 950T                 | ~185%                              |             |
| EAGER    | 1710T                  | 509T                  | 1018T                | ~170%                              |             |
| STANDARD | 1710T                  | 543T                  | 1086T                | ~160%                              | The default speed if none is selected. |
| CAUTIOUS | 1710T                  | 611T                  | 1222T                | ~145%                              |             |
| CONSERVATIVE | 1710T                  | 679T                  | 1358T                | ~130%                              |             |
| SPEEDLOCK7 | 1710T                  | 713T                  | 1426T                | ~125%                              | Almost identical to the speed of the slower Speedlock loaders. |
| ROM      | 2168T                  | 855T                  | 1710T                | 100%                               | The same timings as used by the ROM loader. |

# How to choose a timing #

You must specify the timing in two places:
  1. by defining a symbol in your code;
  1. as an option to 47loader-tzx when you create the TZX file.

## Assemble-time symbols ##

Define a symbol `LOADER_SPEED_*`, where `*` is one of the names in the above table.  Here is a simple example using the fastest-supported timings:

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1
        ;; this is the symbol specifying the timing
LOADER_SPEED_FAST:equ 1

        ;; place the loader into uncontended memory
        org     32768

        ;; load a screen directly into the video RAM
        ld      ix,16384
        ld      de,6912
        call    loader

        ;; load 8144 bytes at address 49152
        ld      ix,49152
        ld      de,8144
        call    loader

        ;; jump to the code we just loaded
        jp      49152

loader:
        ;; call the loader proper
        call    loader_entry
        ;; carry set means success
        ret     c
        ;; if failed, drop into error handler
        include "47loader_error_handler.asm"

        include "47loader.asm"
```

## 47loader-tzx's -speed option ##

The -speed option takes one of the timing names as a parameter.  The timing name is case-insensitive.  To create a tape file suitable for use with the above example, do:

```
47loader-tzx -speed fast -output mygame.tzx mygame.scr
47loader-tzx -speed fast -output mygame.tzx mygame.bin
```

# Notes about ROM timings #

If you use the ROM timings, you will notice that the assembled 47loader code increases in size somewhat.  This is because ROM timings are supported _in addition_ to turbo timings: extra code is generated to detect which timings to use from the frequency of the pilot tone.  The assemble-time option `LOADER_SUPPORT_ROM_TIMINGS` may be set in addition to any of the turbo timing options to produce a loader that supports both ROM timings and the turbo speed of your choice.  Here is a simple example specifying a loader that supports both ROM and FAST speeds:

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1
        ;; this is the symbol specifying the timings
LOADER_SPEED_FAST:equ 1
        ;; additionally, support ROM timings
LOADER_SUPPORT_ROM_TIMINGS:equ 1

        ;; place the loader into uncontended memory
        org     32768

        ;; load a screen directly into the video RAM
        ld      ix,16384
        ld      de,6912
        call    loader

        ;; load 8144 bytes at address 49152
        ld      ix,49152
        ld      de,8144
        call    loader

        ;; jump to the code we just loaded
        jp      49152

loader:
        ;; call the loader proper
        call    loader_entry
        ;; carry set means success
        ret     c
        ;; if failed, drop into error handler
        include "47loader_error_handler.asm"

        include "47loader.asm"
```

With a loader created like that, you can do:

```
47loader-tzx -speed fast -output mygame-turbo.tzx mygame.scr
47loader-tzx -speed fast -output mygame-turbo.tzx mygame.bin
47loader-tzx -speed rom -output mygame-slow.tzx mygame.scr
47loader-tzx -speed rom -output mygame-slow.tzx mygame.bin
```

Now you have two TZX files created from the same source.  You could distribute both of them together; anyone wanting to make a tape to load into a real Spectrum but who has trouble with recording reliable turbo tapes can use the slow TZX instead.