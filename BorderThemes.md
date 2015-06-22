# Introduction #

By defining a label at assemble time, you can choose the border effect that the loader will use.  The effects, referred to as "themes", are encapsulated in chunks of code separate from the main logic.  New ones can easily be written.

The label to define to specify a theme is `LOADER_THEME_*`, where `*` is one of the theme names detailed below, e.g. `LOADER_THEME_RAINBOW`.

# Available themes #

| **Name** | **Searching border** | **Pilot border (fast timings)** | **Pilot border (ROM timings)** | **Data border** | **Notes** |
|:---------|:---------------------|:--------------------------------|:-------------------------------|:----------------|:----------|
| ARGENTINA | Black/white          | Black/cyan                      | Black/cyan                     | Cyan/white      | The data colours are those of the Argentinian flag. Designed by [Alessandro Grussu](http://alessandrogrussu.it/) for the Spanish edition of his game _[Cronopios y Famas](http://www.alessandrogrussu.it/zx/CYF-AR.zip)_. |
| BLEEPLOAD | Red/yellow           | Red/yellow                      | Red/yellow                     | Blue/cyan       | The Firebird Bleepload colour scheme. |
| BRAZIL   | Blue/white           | Blue/white                      | Blue/white                     | Green/yellow    | Based on the Brazilian flag. Requested by Alessandro Grussu. |
| CANDY    | Black/magenta        | Black/yellow                    | Black/white                    | Black/magenta/yellow | Inspired by the purple/yellow candy canes in _Wreck-It Ralph_. |
| CHRISTMAS | Black/red            | Black/white                     | Black/white                    | Black/red/green | Red and green, the holly and the ivy. |
| CYCLE\_VERSA | Solid black with fine red lines | Solid black with fine green lines | Solid black with fine white lines | Solid colour, cycling every 256 bytes, with fine black or white lines | Inspired by an effect used by Peter Knight's [Versaload](https://github.com/going-digital/versaload). |
| ELIXIRVITAE | Black/magenta        | Black/cyan                      | Black/white                    | Black/magenta/cyan | Designed to complement _[Elixir Vitae](http://www.worldofspectrum.org/forums/showthread.php?t=45308)_'s loading screen. |
| FIRE     | Black/red            | Black/yellow                    | Black/white                    | Black/red/yellow | Data border alternates between red and yellow once per byte. |
| ICE      | Blue/cyan            | Blue/white                      | Blue/white                     | Blue/cyan/white | Data border alternates between cyan and white once per byte. |
| ITALY    | Black/red            | Black/green                     | Black/white                    | Red/white/green | The data colours are those of the Italian tricolour. |
| JUBILEE  | Black/red            | Black/blue                      | Black/white                    | Red/white/blue  | The data colours are those of the British Union Flag. |
| LDBYTES  | Red/cyan             | Red/cyan                        | Red/cyan                       | Blue/yellow     | Colours used by the ROM loader. |
| LDBYTESPLUS | Black/red            | Black/cyan                      | Black/white                    | Black/blue/yellow | ROM loader colours with a twist. |
| ORIGINAL | Black/blue           | Black/blue                      | Black/blue                     | Black/red       | Colours used by 47loader early in its development.  Resembles the loader used by some Imagine/Ocean multiload games. |
| RAINBOW  | Black/red            | Black/green                     | Black/white                    | Black/colour, derived from the bits currently being loaded | The "standard" 47loader theme. |
| RAINBOW\_RIPPLE | Black/red            | Black/green                     | Black/white                    | Black/colour, derived from number of bytes remaining to be loaded. | The once-per-byte colour change gives a rippling effect. |
| RAINBOW\_VERSA | Solid red with fine black lines | Solid green with fine black lines | Solid white with fine black lines | Solid black with fine rainbow lines | Inspired by an effect used by Peter Knight's [Versaload](https://github.com/going-digital/versaload). |
| SETYBDL  | Blue/yellow          | Blue/yellow                     | Blue/yellow                    | Red/cyan        | Inverse of the ROM loader's colour scheme. |
| SPAIN    | Black/red            | Black/yellow                    | Black/white                    | Red/yellow      | The data colours are those of the Spanish flag. |
| SPEEDLOCK | Black/red            | Black/red                       | Black/red                      | Black/blue      | Resembles the colour scheme used by many iterations of the famous Speedlock loader. |
| TRINIDAD | Black/red            | Black/white                     | Black/white                    | Black/red/white | The data colours are those of the flag of Trinidad and Tobago. |
| VERSA    | Solid cyan with fine blue lines | Solid white with fine blue lines | Solid white with fine blue lines | Solid blue with fine cyan and white lines | Inspired by an effect used by Peter Knight's [Versaload](https://github.com/going-digital/versaload). |

# Example theme source #

Each theme defines four macros: `border`, `set_searching_border`, `set_pilot_border`, `set_data_border`.

`border` does the actual work of placing the border colour into the accumulator.  On entry, the sign bit of the accumulator is set or clear on alternate edges.
`set_*_border` make any adjustments to the code to set things up for the searching, syncing/pilot or data borders.  On entry to set\_searching\_border, carry is set if fast timings are in use and clear if ROM timings are in use.

```
        ifdef LOADER_THEME_BRAZIL
        ;; based on the Brazilian flag, as requested by
        ;; Alessandro Grussu (http://alessandrogrussu.it/)
        ;; Searching: blue/white
        ;; Pilot/sync:blue/white
        ;; Data:      green/yellow

        macro set_searching_border
        ;; on alternate edges, we want to flick the border between
        ;; blue and white.  The colour numbers are 1 for blue and
        ;; 7 for white; or, in binary, 001 and 111.
        ;;
        ;; So if we can arrange to have the accumulator containing
        ;; all zeros or all ones on alternate edges, we simply need
        ;; to mask off bits 1 and 2, giving us 000 or 110, then
        ;; set bit 0, giving 001 or 111
        ld      a,6             ; mask for bits 110
        ld      (.colour_mask),a
        ld      a,0xc7          ; SET 0,A
        ld      (.colour_instr),a
        endm

        macro set_pilot_border
        ;; same as searching border
        endm

        macro set_data_border
        ;; on alternate edges, we want to flick the border between
        ;; green and yellow.  The colour numbers are 4 for green and
        ;; 6 for yellow; or, in binary, 100 and 110.
        ;; 
        ;; Switching between these colours is very similar to the
        ;; strategy for the pilot border, except that we mask off
        ;; only bit 1, giving us 000 or 010, then always set bit 2,
        ;; giving us 100 or 110.
        ld      a,2             ; mask for bits 010
        ld      (.colour_mask),a
        ld      a,0xd7          ; SET 2,A
        ld      (.colour_instr),a
        endm

        ;; the loader expects a theme to run in either 19T or 23T,
        ;; and we have to declare the time required
.theme_t_states:equ 23
        macro border
        ;; 23T
        ;; in order for the logic described in the above comments
        ;; to work, we need the accumulator to contain all zeros or
        ;; all ones on alternate edges.  At the point that this code
        ;; is executed, the accumulator's sign bit is 0 or 1 on
        ;; alternate edges.  It's simple to move this into the
        ;; carry flag using a rotate instruction, so carry will be
        ;; either set or clear on alternate edges.  Following that,
        ;; we can use use SBC A,A to clear the accumulator and then
        ;; subtract the carry flag from it; so if carry was clear, we
        ;; finish with the accumulator containing zero; if it was set
        ;; we finish with it containing -1, aka 255, aka all ones
        rla                     ; move EAR bit into carry flag (4T)
        sbc     a,a             ; A=0xFF on high edge, 0 on low edge (4T)
        ;; now, we apply the logic described in the above comments.
        ;; The example values here are the values for the data border,
        ;; but they will be changed by set_searching_border and
        ;; set_data_border to be the colours appropriate for the
        ;; phase that the loader is currently in.
.colour_mask:equ $+1
        and     2               ; mask off the red bit (7T)
.colour_instr:equ $+1
        set     2,a             ; set the green bit (8T)
        endm

        endif
```