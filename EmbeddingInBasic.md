# Introduction #

It isn't difficult to embed 47loader in a BASIC program. This obviates the need for a separate "bytes" block containing the 47loader code, so your game starts loading more quickly.

The idea is that we construct a chunk of code that looks like a REM statement to BASIC.  The code can't assume that it will run from any particular location because the base address of the BASIC program is not guaranteed; it will probably be 23755, but not necessarily.  Also, the loader cannot run directly from within the REM statement because it will be located in contended memory.  Therefore, we have to write some position-independent code that copies the loader into uncontended memory and then jumps to it.

# Writing a relocatable loader #

## A simple example ##

Due to the use of the .PHASE and .DEPHASE directives, you _must_ use version 0.6.0 of Pasmo to assemble this example.

```
        ;; we don't know what address we will run from, so we will
        ;; assemble the code from zero and use arithmetic to
        ;; calculate the true address of the loaded code

        ;; the first five bytes of our BASIC line are the line
        ;; number, the length of the line, and a REM keyword.
        ;; Our "line number" will be a JR instruction that jumps
        ;; over the length and REM keyword and into the code to
        ;; execute.  To BASIC, the assembled instruction looks like
        ;; line 6147; the 47 there is a happy coincidence :-)
        jr      .start
        dw      .end - 4  ; length of the line; this is the length of
                          ; all the code, minus the line number and length
        db      0xea      ; BASIC REM keyword

        ;; this is the address to which we will copy the loader.
        ;; It can be any address in uncontended memory
        .loader_addr: equ 32768

.start
        ;; BASIC's USR instruction places the address at which
        ;; execution begins in BC, so we can use this to
        ;; calculate the location of the code that we have to
        ;; copy.  We assembled with origin 0, so .reloc_start
        ;; is the offset of the code to copy relative to our
        ;; entry point.  So by adding .reloc_start to the value
        ;; in BC, we have the absolute address of the code to
        ;; copy
        push    bc
        pop     hl      ; HL now contains the entry address
        ld      bc,.reloc_start
        add     hl,bc   ; HL now contains the address of the code to copy
        ld      de,.loader_addr
        push    de;     ; stack the target address for later
        ld      bc,.reloc_len
        ldir

        ;; the loader has now been copied to the address specified
        ;; in .loader_addr, and we also pushed a copy of it onto
        ;; the stack.  We can thus jump into the relocated loader
        ;; with a simple RET
        ret

.reloc_start: equ $
        ;; this is the beginning of the code we need to relocate.
        ;; Using the .PHASE directive, we can tell Pasmo to start
        ;; assembling using addresses relative to the address to
        ;; which we will copy it
        .phase    .loader_addr

        ;; all the code from here until .reloc_len is a copy of
        ;; the simple example.  You can replace this with your
        ;; own code

        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1

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

        ;; this is the end of the code we want to relocate from the
        ;; REM statement into uncontended memory.  We're still
        ;; assembling relative to .loader_addr, so subtracting the
        ;; current address from .loader_addr  gives us the number of bytes
        ;; to copy
.reloc_len: equ $ - .loader_addr

        ;; from this point, we want to stop assembling relative to
        ;; .loader_addr and revert to assembling relative to zero.
        ;; The .DEPHASE directive tells Pasmo to do this
        .dephase

        ;; this is the last byte of our line of BASIC.  It's a carriage
        ;; return; all BASIC lines end with one of these
        db      13
        ;; we're assembling relative to zero, so this symbol both marks
        ;; the end of the code and also tells us its total length
.end:   equ     $
```

You don't _have_ to jump straight into the loader after relocating it.  You could, for example, put only the 47loader code in the relocated part and keep your own code in the non-relocated part.

## An even simpler example ##

The simple relocation code from the above example is available in two source files, `47loader_simple_basic_embed_top.asm` and `47loader_simple_basic_embed_bottom.asm`.  You can "sandwich" your own code between them.  All you need to do is define the symbol `LOADER_ABSOLUTE_ADDR` to the address to which you want to relocate the loader.  Here's the previous example done this way:

```
LOADER_ABSOLUTE_ADDR:    equ     32768
        include "47loader_simple_basic_embed_top.asm"

        ;; this is the border theme to use.  A full list of the
        ;; available themes is on the Wiki:
        ;; https://code.google.com/p/47loader/wiki/BorderThemes
LOADER_THEME_RAINBOW:equ 1

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

        include "47loader_simple_basic_embed_bottom.asm"
```

# Producing a TZX file #

The [47loader-bootstrap tool](https://code.google.com/p/47loader/source/browse/#svn%2Ftrunk%2Ftools) can be used to embed the assembled loader into a BASIC program.  Here are some example command lines that together construct a TZX file containing the loader, screen and game for the above examples, assuming that the loader has been assembled into a file called "myloader.bin":

```
47loader-bootstrap -clear 32767 -ink 7 -paper 0 -border 0 -name MYGAME -output mygame.tzx myloader.bin
47loader-tzx -output mygame.tzx -pause 100 mygame.scr
47loader-tzx -output mygame.tzx -pilot short mygame.bin
```

The above invocation of 47loader-bootstrap generates a BASIC loader with filename "MYGAME" that sets black border and paper, white ink, CLEARs to 32767 and then executes the loader.

You can of course embed loaders using any of the extra features such as ["instascreen"](Instascreen.md).  (Because "instascreen" also sets the screen colours, you might omit the -border, -paper and -ink options if you are using that).