# Introduction #

47loader permits blocks to be glued together with tiny pilots.  You can use this feature to load a screen, then move straight into loading the game without a perceptible gap.

This feature is called "resume".

# How to use "resume" in your code #

You need to define a symbol, `LOADER_RESUME`. This adds a new entry point, `loader_resume`, that permits code to call the loader without locking onto a pilot tone: the load resumes at the syncing phase.

Here is an example of how to load a screen, then use "resume" to load the game:

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1
        ;; this symbol adds the loader_resume entry point
LOADER_RESUME:equ 1

        ;; place the loader into uncontended memory
        org     32768

        ;; load a screen directly into the video RAM
        ld      ix,16384
        ld      de,6912
        call    loader_entry
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
```

# Making a tape file #

Here are the 47loader-tzx commands that make a tape file that will work with the above example:

```
47loader-tzx -output mygame.tzx -pause 0 mygame.scr
47loader-tzx -output mygame.tzx -pilot resume mygame.bin
```

The first command adds the screen to the tape file with no pause afterwards.  The second command adds the game that will be loaded with `loader_resume`. The length of the pilot tone is made as short as possible.

When using "resume", you **must** write the preceding block using -pause 0. If `loader_resume` does not find a pulse immediately, it raises a loading error.