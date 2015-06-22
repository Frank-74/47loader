# Introduction #

47loader provides an optional module called "instascreen".  This module:
  1. loads pixmap data directly into the video RAM;
  1. then uses "[resume](Resume.md)" to load the attribute data into high memory;
  1. then uses LDIR to blit the attributes into the video RAM, resulting in a loading screen that pops up in an instant.
You can then use "resume" to load your game without a gap; the overall effect is similar to the Speedlock loading screens.

# Using "instascreen" in your code #

## Bringing in the module ##
The "instascreen" module is in a separate source file, `47loader_instascreen.asm`, that you must include.  It defines an entry point, `loader_instascreen`; this is the routine to call.

## Symbols that you must define ##

Because "instascreen" uses "resume" to load the attributes, you must define `LOADER_RESUME`.

"Instascreen" is configured using three symbols, one of which is mandatory.

  1. `LOADER_INSTASCREEN_ATTR_ADDRESS`: set this to an address in uncontended memory at which the attributes are to be loaded.  768 bytes are required starting at the specified address.  This setting is mandatory.
  1. `LOADER_INSTASCREEN_FILL_COLOUR`: by default, "instascreen" sets the screen attributes to black on black before loading the pixmap.  If you would prefer a different colours, set this symbol to the number of the colour that you want.
  1. `LOADER_INSTASCREEN_FILL_BRIGHT`: define this symbol if you want the colour defined by `LOADER_INSTASCREEN_FILL_COLOUR` to be bright.

## Simple example ##

This example uses "instascreen" to set the screen to bright blue, then loads a screen using 768 bytes starting from 49152 as temporary space for the attributes.  After the screen has loaded, "resume" is used to load the game with no perceptible gap.

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1
        ;; use 768 bytes starting at 49152 as scratch space for attrs
LOADER_INSTASCREEN_ATTR_ADDRESS:equ 49152
        ;; fill the screen bright blue while loading
LOADER_INSTASCREEN_FILL_COLOUR:equ 1
LOADER_INSTASCREEN_FILL_BRIGHT:equ 1
        ;; required by both "instascreen" and our own code
LOADER_RESUME:equ 1

        ;; place the loader into uncontended memory
        org     32768

        ;; use "instascreen" to load the screen
        call    loader_instascreen
        ;; carry set means success; jump to error handler
        ;; if the load failed
        jr      nc,error_exit

        ;; load 8144 bytes at address 49152
        ld      ix,49152
        ld      de,8144
        ;; use loader_resume to load the next block with no
        ;; gap
        call    loader_resume

        ;; jump to the code we just loaded if the load succeeded.
        ;; If it failed, fall through to the error handler
        jp      c,49152

error_exit:
        include "47loader_error_handler.asm"

        include "47loader.asm"
        include "47loader_instascreen.asm"
```

# Making a tape file #

47loader-tzx has a special option that must be used for writing screens for use with "instascreen".  It stores the pixmap and attributes as separate blocks and inserts sufficient padding pulses to leave enough time for the attributes to be copied from uncontended memory into the screen.

These commands would produce a tape file suitable for use with the above example:
```
47loader-tzx -output mygame.tzx -instascreen mygame.scr
47loader-tzx -output mygame.tzx -pilot resume mygame.bin
```

The file used with the -instascreen option ("mygame.scr" in the above example) must be 6912 bytes long.