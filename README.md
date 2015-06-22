An open-source tape loader for the ZX Spectrum implemented from scratch.

Why should I use 47loader for my next Spectrum project?

* Custom loaders are always more fun than using the ROM loader :-)
* By default, loads at ~160% of the ROM loader speed (543 T-states for a zero pulse, 1086T for a one pulse, versus the ROM loader's 855T/1710T). Speed can be configured up to 185% of the ROM loader speed (475T/950T).
* In addition to fast blocks, the loader can also cope with blocks using ROM timings, so the same build of your software can be distributed as both TZX and TAP files.
* Robust error detection using Fletcher's algorithm.
* Choice of border effects; additional themes can be added with a few lines of assembly language.
* Choice of FancyScreen screen loading effects?, or an "instant" loading screen a la Speedlock.
* Optional colour-changing progress indicator.
* Free, open-source software, liberally licensed. 

Why shouldn't I use 47loader?

* It's written by a rank amateur with next to no Z80 coding experience. 

[Wiki documentation is available](https://github.com/stephenw32768/47loader/blob/wiki/ProjectHome.md).
