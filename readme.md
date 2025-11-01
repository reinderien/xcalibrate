What this?
----------

This is an alternative to _xinput_calibrator_ (as seen on
[github](https://github.com/tias/xinput_calibrator),
[FreeDesktop](https://www.freedesktop.org/wiki/Software/xinput_calibrator/),
[Ubuntu](https://packages.ubuntu.com/disco/xinput-calibrator), etc.).

Similarities:
- For Unix-like OS
- Calibrates touchscreens

Differences:

|Thing           |xinput_calibrator                |xcalibrate                     |
|----------------|---------------------------------|-------------------------------|
| Language       |C++                              |Python3                        |
| LOC            |2,289                            |348                            |
| Interface      |Command-line args with some GUI  |Walkthrough-based with some GUI|
| GUI toolkit    |X                                |tk                             |
| Cal points     |4                                |Selectable                     |
| Math           |Manual                           |Numpy least-squares            |
| Outputs        |Xorg.conf, hal, xinput           |xinput                         |
| Popularity     |High                             |Zero so far                    |
| Tested         |Pretty well                      |Not very well                  |
| OS distribution|Available in most major repos    |[AUR](https://aur.archlinux.org/packages/xcalibrate-git) |
| Dependencies   |libstdc++, libx11, libxi         |python3, tkinter, numpy        |

_xinput_calibrator_ features precalibration, timeouts, misclick detection, fake driver testing, and
a resizable window.
_xcalibrate_ features none of those things, but offers some other features not present in
_xinput_calibrator_, including pre-save cal test, cal quality indicator, arbitrary cal point count,
and a slightly more informative GUI. Also, the source is quite simpler.

Why this?
---------

After I couldn't figure out why _xinput_calibrator_ wasn't doing anything on my system (an old
[CF-29](https://na.panasonic.com/us/computers-tablets/computers/laptops/toughbook-cf-29)),
I read and followed
[this excellent walkthrough](https://wiki.archlinux.org/index.php/Talk:Calibrating_Touchscreen#Libinput_breaks_xinput_calibrator)
on the Arch wiki. It was too hacky for my comfort, so I wrote this tool inspired by Zootboy's
procedure but with some key differences - this is a stand-alone tool that does not require running
_xinput_calibrator_; and it uses Numpy to do some real matrix math instead of hacking around with
element-by-element calculations and a fixed calibration dataset size.

How this?
---------

    ./xcalibrate

Example output
--------------

```none
./xcalibrate 
Pointer devices:
  ID                                Name
   4          Virtual core XTEST pointer
  10                   USB Optical Mouse
  11              Elo Serial TouchScreen

Device to calibrate [11]: 

Old calibration (Coordinate Transformation Matrix):
[[ 1.277742  0.       -0.120407]
 [ 0.       -1.417391  1.235114]
 [ 0.        0.        1.      ]]

Calibrate? [y]: 
Point count (min 3) [4]: 9
Disable rotation? [y]: n

New calibration:
[[ 1.29356314 -0.00499606 -0.12657437]
 [ 0.03126622 -1.42132634  1.22818551]
 [ 0.          0.          1.        ]]
Quality (should be at least 3): 2.9

Test? [y]: y
Point count (min 3) [4]: 9
Use calibration? [y]: y
The new calibration has been activated.
To persist, create e.g.
/usr/share/X11/xorg.conf.d/99-libinput-ts-calib.conf
and add the following:

Section "InputClass"
    Identifier      "calibration"
    MatchProduct    "Elo Serial TouchScreen"
    Option          "Coordinate Transformation Matrix"  "1.29356314337069 -0.00499606097246703 -0.126574373908785 0.0312662187568204 -1.42132633629107 1.22818550687634 0 0 1"
EndSection
```

To do
-----

- Review some of the forks and issues logged
- Be more interactive in device selection

Discuss
-------

Check out the [Discussions](https://github.com/reinderien/xcalibrate/discussions) page.
Github Discussions has superceded the
[Gitter instance](https://gitter.im/reinderien_xcal/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge).
