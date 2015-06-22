Here are some simple ways of using 47loader.

47loader source must be assembled using Pasmo 0.6.0.  The source is available from the Pasmo website; alternatively, [here's a zipfile containing a pre-built executable for Windows](https://skydrive.live.com/redir?resid=786750D7B39FEA33!238&authkey=!AIyjOk1MHYh-UTo).

# The simplest example #

This code uses 47loader to load a screen at 16384, then some code at 49152, then executes the loaded code.  If an error occurs or BREAK/SPACE is pressed, the standard ROM error handlers are used.

47loader has several configuration settings, but the only required setting is to choose one of the BorderThemes.

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1

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

Save this into the same directory as the 47loader source files, assemble it with [Pasmo 0.6.0](http://pasmo.speccy.org/) and place it into a TZX file.  Pasmo's --tzx output format is one way of doing this.  Obviously, you will need to change the address and length of the game code block, and perhaps assemble the loader at a different address.

To add the 47loader blocks to the output file, use the [47loader-tzx tool](https://code.google.com/p/47loader/source/browse/#svn%2Ftrunk%2Ftools).  You will need both 47loader-tzx.exe and 47loader-util.dll.

If your TZX file is called "mygame.tzx", the 6192-byte screen is called "mygame.scr" and the code is in a file called "mygame\_code.bin", the commands to use are:

```
47loader-tzx -output mygame.tzx mygame.scr
47loader-tzx -output mygame.tzx mygame.bin
```

For aesthetic reasons, you might like the game block to have a much shorter pilot tone.  Try:

```
47loader-tzx -output mygame.tzx -pause 100 mygame.scr
47loader-tzx -output mygame.tzx -pilot short mygame.bin
```

This variation reduces the pause after the screen to a tenth of a second, and reduces the length of the second block's pilot tone to approximately three-tenths of a second.

Or, how about:

```
47loader-tzx -output mygame.tzx -pilot click -pause 100 mygame.scr
47loader-tzx -output mygame.tzx -pilot click4 mygame.bin
```

This variation gives a clicking pilot tone.  The screen's pilot tone has eight clicks, the default.  The game's tone has four clicks.  You can have as many clicks as you want, within reason; just place the number immediately after the `click` parameter, without any leading spaces.

# A backwards-loading screen #

In this example, we will load the screen backwards.  The attributes appear first, then the pixmap loads from the bottom of the screen.  After that, the game loads conventionally.  The code is only a little bit more complicated:

```
        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1
        ;; the first block will be loaded backwards
LOADER_BACKWARDS:equ 1
        ;; we will also be loading a block forwards, so we need to
        ;; include code to reverse the direction of the load
LOADER_CHANGE_DIRECTION:equ 1

        org     32768

        ;; we are loading the screen backwards, so we point IX at
        ;; the _last_ byte, not the first
        ld      ix,23295
        ld      de,6912
        call    loader

        ;; we want to load the next block forwards, so we need to
        ;; tell the loader to do that
        call    loader_change_direction

        ;; now we load the 8144-byte game starting at address 49152
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

You must also reverse the screen when adding it to the TZX file.  47loader-tzx can do it for you.  To create the 47loader blocks for this example, do:
```
47loader-tzx -output mygame.tzx -reverse mygame.scr
47loader-tzx -output mygame.tzx mygame.bin
```

Or, if you like the really short pilot tone for the second block from the first example, you can of course combine that with the reversed screen:

```
47loader-tzx -output mygame.tzx -pause 100 -reverse mygame.scr
47loader-tzx -output mygame.tzx -pilot short mygame.bin
```