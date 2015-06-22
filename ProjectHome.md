An open-source tape loader for the ZX Spectrum implemented from scratch.

# Why should I use 47loader for my next Spectrum project? #

  1. Custom loaders are always more fun than using the ROM loader :-)
  1. By default, loads at ~160% of the ROM loader speed (543 T-states for a zero pulse, 1086T for a one pulse, versus the ROM loader's 855T/1710T).  [Speed can be configured](Timings.md) up to 185% of the ROM loader speed (475T/950T).
  1. In addition to fast blocks, the loader can also cope with blocks using ROM timings, so the same build of your software can be distributed as both TZX and TAP files.
  1. Robust error detection using [Fletcher's algorithm](https://en.wikipedia.org/wiki/Fletcher's_checksum).
  1. [Choice of border effects](BorderThemes.md); additional themes can be added with a few lines of assembly language.
  1. [of FancyScreen screen loading effects](Choice.md), or an ["instant" loading screen](Instascreen.md) a la Speedlock.
  1. Optional colour-changing progress indicator.
  1. [Free, open-source software, liberally licensed](https://code.google.com/p/47loader/source/browse/trunk/LICENSE).

# Why shouldn't I use 47loader? #

  1. It's written by a rank amateur with next to no Z80 coding experience.


# How do I use it? #

  * Start by looking at a [simple example](SimpleExample.md).
  * Here's [how to specify the speed of the loader](Timings.md).
  * You can [load blocks glued together with no gaps](Resume.md).
  * A [loading screen that pops up instantly](Instascreen.md) always looks slick.
  * Or perhaps you'd like a [bidirectional screen load](FancyScreen.md)?
  * The [error handling can be configured](ErrorHandling.md).
  * It's fairly simple to [embed the loader in a REM statement in a BASIC loader](EmbeddingInBasic.md).