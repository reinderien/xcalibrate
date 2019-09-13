What this?
==========

This is an alternative to _xinput_calibrator_ (as seen on
[github](https://github.com/tias/xinput_calibrator),
[FreeDesktop](https://www.freedesktop.org/wiki/Software/xinput_calibrator/),
[Ubuntu](https://packages.ubuntu.com/disco/xinput-calibrator), etc.).

Similarities:
- For Unix-like OS
- Calibrates touchscreens

Differences:

|Thing           |xinput_calibrator                |xcal                           |
|----------------|---------------------------------|-------------------------------|
| Language       |C++                              |Python3                        |
| LOC            |2,289                            |284                            |
| Interface      |Command-line args with some GUI  |Walkthrough-based with some GUI|
| GUI toolkit    |X                                |tk                             |
| Cal points     |4                                |Selectable                     |
| Math           |Manual                           |Numpy least-squares            |
| Outputs        |Xorg.conf, hal, xinput           |xinput                         |
| Popularity     |High                             |Zero so far                    |
| Tested         |Pretty well                      |Not very well                  |
| OS distribution|Available in most major repos    |Zero so far                    |
| Dependencies   |libstdc++, libx11, libxi         |python3, tkinter, numpy        |

_xinput_calibrator_ features precalibration, timeouts, misclick detection, fake driver testing, and
a resizable window.
_xcal_ features none of those things, but offers some other features not present in
_xinput_calibrator_, including pre-save cal test, cal quality indicator, arbitrary cal point count,
and a slightly more informative GUI. Also, the source is quite simpler.

Why this?
=========

After I couldn't figure out why _xinput_calibrator_ wasn't doing anything on my system (an old CF-29),
I read and followed
[this excellent walkthrough](https://wiki.archlinux.org/index.php/Talk:Calibrating_Touchscreen#Libinput_breaks_xinput_calibrator)
on the Arch wiki. It was too hacky for my comfort, so I wrote this tool inspired by Zootboy's
procedure but with some key differences - this is a stand-alone tool that does not require running
_xinput_calibrator_; and it uses Numpy to do some real matrix math instead of hacking around with
element-by-element calculations and a fixed calibration dataset size.

How this?
=========

    ./xcal

To do
=====

- ?

Discuss
=======

[![Join the chat at https://gitter.im/reinderien_xcal/Lobby](https://badges.gitter.im/reinderien_xcal/Lobby.svg)](https://gitter.im/reinderien_xcal/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
