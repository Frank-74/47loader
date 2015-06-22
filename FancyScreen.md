# Introduction #

The 47loader-tzx tool can produce screens that load in unusual ways.  Such screens can be loaded by 47loader using an optional module, "dynamic". This module reads a table of block addresses, lengths and directions, then uses "resume" to load them in sequence.

# Creating a tape file #

You can use 47loader-tzx's `-fancyscreen` option to specify a fancy screen load.  The option takes an argument specifying the desired loading effect.

## Bidirectional screen loads ##

Define `LOADER_CHANGE_DIRECTION` when using these effects.

| **`-fancyscreen` argument** | **Effect** | **Loader direction after the load is complete** |
|:----------------------------|:-----------|:------------------------------------------------|
| `bidi_acp`                  | Attributes that start at the top and bottom and converge in the centre, then a bidirectional pixmap load. | Backwards                                       |
| `bidi_adp`                  | Attributes that start at the centre and diverge from top to bottom, then a bidirectional pixmap load. | Backwards                                       |
| `bidi_ap`                   | Bidirectional screen load, attributes first. | Backwards                                       |
| `bidi_as2p`                 | Attributes that load in blocks of 2x2 squares, then a bidirectional pixmap load. | Backwards                                       |
| `bidi_as4p`                 | Attributes that load in blocks of 4x4 squares, then a bidirectional pixmap load. | Backwards                                       |
| `bidi_as8p`                 | Attributes that load in blocks of 8x8 squares, then a bidirectional pixmap load. | Backwards                                       |
| `bidi_pa`                   | Bidirectional screen load, pixmap first. | Backwards                                       |
| `bidi_pac`                  | Bidirectional pixmap load followed by attributes that start at the top and bottom and converge in the centre. | Backwards                                       |
| `bidi_pad`                  | Bidirectional pixmap load followed by attributes that start at the centre and diverge from top to bottom. | Backwards                                       |
| `bidi_pas2`                 | Bidirectional pixmap load followed by attributes loaded in blocks of 2x2 squares. | Forwards                                        |
| `bidi_pas4`                 | Bidirectional pixmap load followed by attributes loaded in blocks of 4x4 squares. | Forwards                                        |
| `bidi_pas8`                 | Bidirectional pixmap load followed by attributes loaded in blocks of 8x8 squares. | Forwards                                        |

## Linear screen loads ##

Define `LOADER_DYNAMIC_FORWARDS_ONLY` and `LOADER_DYNAMIC_FIXED_LENGTH` when using these effects.  `LOADER_DYNAMIC_FIXED_LENGTH` must be defined with the value output by 47loader-tzx.

| **`-fancyscreen` argument** | **Effect** | **Loader direction after the load is complete** | **`LOADER_DYNAMIC_FIXED_LENGTH` value** |
|:----------------------------|:-----------|:------------------------------------------------|:----------------------------------------|
| `linear_btt`                | The screen loads in a non-interlaced fashion from bottom to top. | Forwards                                        | 32                                      |
| `linear_c`                  | The screen loads in a non-interlaced fashion from the top and bottom, meeting in the middle. | Forwards                                        | 32                                      |
| `linear_d`                  | The screen loads in a non-interlaced fashion from the middle, spreading to the top and bottom. | Forwards                                        | 32                                      |
| `linear_s4`                 | The screen loads in a non-interlaced fashion from the top and bottom, in 4x4 square blocks. | Forwards                                        | 4                                       |
| `linear_s8`                 | The screen loads in a non-interlaced fashion from the top and bottom, in 8x8 square blocks. | Forwards                                        | 8                                       |
| `linear_ttb`                | The screen loads in a non-interlaced fashion from top to bottom. | Forwards                                        | 32                                      |

## Other effects ##

| **`-fancyscreen` argument** | **Effect** | **Loader direction after the load is complete** |
|:----------------------------|:-----------|:------------------------------------------------|
| `btt_fa`                    | Loads the pixmap bottom-to-top instead of in thirds, then the attributes forwards. | Forwards                                        |
| `btt_ra`                    | Loads the pixmap bottom-to-top instead of in thirds, then the attributes backwards. | Backwards                                       |
| `fa_btt`                    | Loads the attributes forwards, then the pixmap bottom-to-top instead of in thirds. | Backwards                                       |
| `fa_rp`                     | Loads the attributes forwards, then the pixmap backwards. | Backwards                                       |
| `fa_ttb`                    | Loads the attributes forwards, then the pixmap top-to-bottom instead of in thirds. | Forwards                                        |
| `fp_abidi`                  | Loads the pixmap forwards, then the attributes bidirectionally. | Forwards                                        |
| `fp_as2`                    | Loads the pixmap forwards, then the attributes in blocks of 2x2 squares. | Forwards                                        |
| `fp_as4`                    | Loads the pixmap forwards, then the attributes in blocks of 4x4 squares. | Forwards                                        |
| `fp_as8`                    | Loads the pixmap forwards, then the attributes in blocks of 8x8 squares. | Forwards                                        |
| `fp_ra`                     | Loads the pixmap forwards, then the attributes backwards. | Backwards                                       |
| `ra_btt`                    | Loads the attributes backwards, then the pixmap bottom-to-top instead of in thirds. | Backwards                                       |
| `ra_fp`                     | Loads the attributes backwards, then the pixmap forwards. | Forwards                                        |
| `ra_ttb`                    | Loads the attributes backwards, then the pixmap top-to-bottom instead of in thirds. | Forwards                                        |
| `rp_fa`                     | Loads the pixmap backwards, then the attributes forwards. | Forwards                                        |
| `ttb_fa`                    | Loads the pixmap top-to-bottom instead of in thirds, then the attributes forwards. | Forwards                                        |
| `ttb_ra`                    | Loads the pixmap top-to-bottom instead of in thirds, then the attributes backwards. | Backwards                                       |

For example, this sequence of commands creates a tape file containing:
  * a screen that loads the pixmap forwards but the attributes backwards;
  * a block containing a game, immediately afterwards with no gap, for loading using "[resume](Resume.md)":

```
47loader-tzx -fancyscreen fp_ra -pause 0 -output mygame.tzx mygame.scr
47loader-tzx -pilot resume -output mygame.tzx mygame.bin
```

When you use `-fancyscreen`, 47loader-tzx reports the length of the table containing the addresses, lengths and directions of each block comprising the screen.  The above example produces the following output:
```
Dynamic table length: 9 bytes
```

# Writing the loader #

Fancy screens are loaded using a module called "dynamic" that exports an entry point, `loader_dynamic`.

You will need to define some symbols:

  * `LOADER_DYNAMIC_TABLE_ADDR`: as described above, `-fancyscreen` instructs 47loader-tzx to output a block containing a table describing the blocks that comprise the screen.  The value of this symbol is the memory address at which the table will be loaded.  There must be enough free memory at this address to store the number of bytes reported by 47loader-tzx.
  * `LOADER_RESUME`: "dynamic" requires "resume", so you must define this symbol.
  * `LOADER_DYNAMIC_FORWARDS_ONLY`: define this symbol if 47loader-tzx tells you to.  It disables the direction-change code in the "dynamic" module, saving a few bytes if the fancy screen effect does not load any blocks backwards.  If this symbol is not defined, direction changes are needed so you must define `LOADER_CHANGE_DIRECTION` to bring the required routine into the loader.
  * `LOADER_DYNAMIC_ONE_BYTE_LENGTHS`: define this symbol if 47loader-tzx tells you to.  If defined, the block lengths in the dynamic table are one byte wide.  47loader-tzx writes dynamic tables like this if no dynamic block is longer than 127 bytes.
  * `LOADER_DYNAMIC_FIXED_LENGTH`: define this symbol if 47loader-tzx tells you to.  If defined, all dynamic blocks are the same length, so the lengths are omitted from the table; the length is specified at assemble time.  The symbol's value must be the block length that 47loader-tzx specifies in its output.

Because fancy screens generally involve one or more changes of load direction, it is possible that the loader will be left pointing backwards after the screen has loaded.  The tables above indicates which fancy effects this applies to.  Following a fancy screen load, you probably want to load the game forwards, so you must include a call to `loader_change_direction` before loading the game.

Here is an example of a loader that will be embedded in a BASIC REM statement.  It uses the "dynamic" module to load the fancy screen, then loads the game using "resume".

```
;;; relocate the loader to this address
LOADER_ABSOLUTE_ADDR:       equ     32768
        include "47loader_simple_basic_embed_top.asm"

;;;  this is the border theme to use.  A full list of the
;;;  available themes is on the Wiki:
;;;  https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:   equ 1

;;; this is the address at which the dynamic table will be
;;; loaded.  It happens to be the same address at which the
;;; game is to be loaded; once the screen in place, the table
;;; is not required any more
LOADER_DYNAMIC_TABLE_ADDR:equ 49152

;;; loader_dynamic requires the following options
LOADER_RESUME:equ 1
LOADER_CHANGE_DIRECTION:equ 1

        ;; loading the screen is a simple as just calling
        ;; loader_dynamic
        call    loader_dynamic
        ;; as with all 47loader entry points, the carry flag
        ;; is set if the load succeeded
        jr      nc,.err

        ;; the -fancyscreen option passed to 47loader-tzx in the
        ;; previous section specified the fp_ra effect.  This
        ;; effect leaves the loader pointing backwards, so we
        ;; need to change direction before loading the game
        call    loader_change_direction

        ;; load 8144 bytes at address 49152 using "resume"
        ld      ix,49152
        ld      de,8144
        call    loader_resume

        ;; if the load succeeded, jump to the game, otherwise
        ;; fall through to the error handler
        jp      c,49152

.err:
        include "47loader_error_handler.asm"
        include "47loader.asm"
        include "47loader_dynamic.asm"

        include "47loader_simple_basic_embed_bottom.asm"
```

Here is a second, similar example.  In this example, we are using the `linear_ttb` effect.

```
47loader-tzx -fancyscreen linear_ttb -pause 0 -output mygame.tzx mygame.scr
47loader-tzx -pilot resume -output mygame.tzx mygame.bin
```

The output from the first 47loader-tzx command is:
```
Dynamic table length: 649 bytes
Define LOADER_DYNAMIC_ONE_BYTE_LENGTHS
Define LOADER_DYNAMIC_FORWARDS_ONLY
```

The load routine is as follows:

```
;;; relocate the loader to this address
LOADER_ABSOLUTE_ADDR:       equ     32768
        include "47loader_simple_basic_embed_top.asm"

;;;  this is the border theme to use.  A full list of the
;;;  available themes is on the Wiki:
;;;  https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:   equ 1

;;; this is the address at which the dynamic table will be
;;; loaded.  It happens to be the same address at which the
;;; game is to be loaded; once the screen in place, the table
;;; is not required any more
LOADER_DYNAMIC_TABLE_ADDR:equ 49152

;;; loader_dynamic requires "resume"
LOADER_RESUME:equ 1

;;; 47loader-tzx told us to define these symbols:
;;; linear screen loads consist of many 32-byte blocks, so 47loader-tzx
;;; writes a more compact table using only one byte for each block length
LOADER_DYNAMIC_ONE_BYTE_LENGTHS:equ 1
;;; linear screen loads never change direction
LOADER_DYNAMIC_FORWARDS_ONLY:equ 1

        ;; loading the screen is a simple as just calling
        ;; loader_dynamic
        call    loader_dynamic
        ;; as with all 47loader entry points, the carry flag
        ;; is set if the load succeeded
        jr      nc,.err

        ;; load 8144 bytes at address 49152 using "resume"
        ld      ix,49152
        ld      de,8144
        call    loader_resume

        ;; if the load succeeded, jump to the game, otherwise
        ;; fall through to the error handler
        jp      c,49152

.err:
        include "47loader_error_handler.asm"
        include "47loader.asm"
        include "47loader_dynamic.asm"

        include "47loader_simple_basic_embed_bottom.asm"
```

If your game loads more than one screen using the "dynamic" module (for example, an initial loading screen, then a playfield screen at the very end of the load), you should use the same `-fancyscreen` option with both of them.  This is because different effects may specify conflicting options to be set at assemble time (for example, one may tell you to define `LOADER_DYNAMIC_ONE_BYTE_LENGTHS` and the other not).  However, if you really do want to use different effects, `linear_ttb`, `linear_btt`, `linear_c` and `linear_d` do all specify the same options and are thus safe to mix and match.