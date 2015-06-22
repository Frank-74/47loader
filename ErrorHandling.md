# Introduction #

By default, 47loader:
  * detects BREAK/SPACE and aborts the load if it is being pressed;
  * returns if a load error is detected.

You can change both these behaviours.

# Ignoring BREAK/SPACE #

Define the symbol `LOADER_IGNORE_BREAK` in your code. That's all there is to it!

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1
        ;; do not abort the load if BREAK/SPACE is pressed
LOADER_IGNORE_BREAK:equ 1

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
        ;; if failed, drop into error handler.  The only possible failure
        ;; is a tape error; the loader will not have returned on BREAK/SPACE
        include "47loader_error_handler.asm"

        include "47loader.asm"
```

# Disabling return to BASIC on error #

Instead of returning to BASIC when a tape error occurs, it is possible for 47loader to reboot the Spectrum via USR 0 instead.  You might like to use this feature if it is not safe to return to BASIC, for example if your game overwrites the BASIC environment.

To use the feature, define `LOADER_DIE_ON_ERROR`. Defining this symbol also disables BREAK/SPACE detection, so you don't need to define `LOADER_IGNORE_BREAK` as well.

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1
        ;; ignore BREAK/SPACE and reset via USR 0 on error
LOADER_DIE_ON_ERROR:equ 1

        ;; place the loader into uncontended memory
        org     32768

        ;; load a screen directly into the video RAM
        ld      ix,16384
        ld      de,6912
        call    loader_entry
        ;; no need for any error handler; if the load failed, the
        ;; loader reset the system

        ;; load 8144 bytes at address 49152
        ld      ix,49152
        ld      de,8144
        call    loader_entry

        ;; jump to the code we just loaded
        jp      49152

        include "47loader.asm"
```