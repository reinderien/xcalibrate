#!/usr/bin/env python3

import re
import numpy as np
from subprocess import run, PIPE


prop_name = 'libinput Calibration Matrix'


def xinput(*args):
    return run(
        args=('/usr/bin/xinput', *args),
        stdout=PIPE, check=True,
        universal_newlines=True,
    ).stdout


def get_devs():
    groups = re.findall(r'.(\w.+(\w|\(\d+\)))\s+id=(\d+)\D+slave *pointer', xinput('--list', '--short'))
    devs = {int(group[2]): group[0] for group in groups}
    if not devs:
        print('No suitable input devices found')
        exit(1)
    return devs


def print_devs(devs) -> None:
    print('Pointer devices:')
    print('%4s %35s' % ('ID', 'Name'))
    for i, name in sorted(devs.items()):
        print('%4d %35s' % (i, name))
    print()


def choose_preferred(devs):
    preferred = [i for (i, n) in devs.items() if 'touch' in n.lower()]
    if preferred:
        return preferred[0]
    return next(iter(devs.keys()))


def choose_dev(devs, preferred):
    while True:
        devstr = input('Device to calibrate [%d]: ' % preferred)
        if not devstr:
            return preferred
        try:
            dev = int(devstr)
        except ValueError:
            continue
        if dev in devs.keys():
            return dev


def read_cal(dev) -> tuple:
    stdout = xinput('--list-props', str(dev))
    line = re.search(prop_name + r'.*:\s+(\S.+)', stdout)
    if not line:
        print('Cal property not set; is this an xinput device?')
        exit(1)
    vals = np.matrix(line.group(1)).reshape(3, 3)

    print('Old calibration:')
    print(vals)
    print()
    return vals, np.linalg.inv(vals)


def ask(q: str) -> bool:
    do = input(q + ' [y]: ')
    return (do or 'y').lower() == 'y'


def choose_points():
    p_min, default = 3, 4

    while True:
        p_str = input('Point count (min %d) [%d]: ' %
                      (p_min, default))
        if not p_str:
            return default
        try:
            p = int(p_str)
        except ValueError:
            continue

        if p >= p_min:
            return p


def transform(x, y, cal) -> tuple:
    p = np.matrix([[x], [y], [1]])
    out = np.matmul(cal, p)
    return out.item(0), out.item(1)


def show_tk(n_points, old_cal_inv, new_cal=None):
    from tkinter import Tk, Canvas
    from math import ceil, sqrt

    root = Tk()
    X, Y = None, None
    root.attributes('-fullscreen', True)
    # as the above line doesn't work on all screens
    # we force the geometry to be the fullsize with next two lines
    w,h = root.winfo_screenwidth(), root.winfo_screenheight()
    root.geometry("%dx%d" % (w,h))
    canvas = Canvas(root)

    def resize(event) -> None:
        nonlocal X, Y
        X, Y = event.width, event.height
        draw_legends()
        next_point()
    canvas.bind('<Configure>', resize)
    canvas.pack(expand=True, fill='both')

    legend_y = None

    def legend(text, colour='#000') -> None:
        nonlocal legend_y
        canvas.create_text(X/2, legend_y, text=text, fill=colour)
        legend_y += 12

    def draw_legends() -> None:
        nonlocal legend_y
        legend_y = Y * 0.3
        legend('Esc to cancel.')
        legend('Raw point in black')
        legend('Old cal point in blue', '#00F')
        legend('Target point in red', '#F00')
        if new_cal is not None:
            legend('New cal point in green', '#0F0')

    point, points = {}, []
    index = -1
    n_cols = int(ceil(sqrt(n_points)))
    n_rows = int(ceil(n_points / n_cols))
    sensitive = False

    def next_point() -> None:
        nonlocal point, index, sensitive
        index += 1
        if index >= n_points:
            sensitive = False
            root.after(1000, root.destroy)
        else:
            sensitive = True
            x = 0.1 + 0.8*(index % n_cols)/(n_cols - 1)
            y = 0.1 + 0.8*(index // n_cols)/(n_rows - 1)
            point = {'sx': x, 'sy': y}

            draw_target(point['sx'], point['sy'])

    def cross(px, py, colour) -> None:
        x, y = px*X, py*Y
        canvas.create_line(x-10, y, x+10, y, fill=colour)
        canvas.create_line(x, y-10, x, y+10, fill=colour)

    def draw_target(px, py) -> None:
        x, y = px*X, py*Y
        canvas.create_oval(x-10, y-10, x+10, y+10, outline='#F00', width=3)
        cross(px, py, '#F00')

    def cancel_cal(_) -> None:
        print('Calibration cancelled')
        points.clear()
        root.destroy()
    root.bind('<Escape>', cancel_cal)
    canvas.bind('<Escape>', cancel_cal)

    def indicator(sx, sy, px, py, colour):
        canvas.create_line(X*sx, Y*sy, X*px, Y*py, fill=colour)
        cross(px, py, colour)

    def click(event):
        nonlocal sensitive
        if not sensitive:
            return
        sensitive = False
    
        sx, sy = point['sx'], point['sy']

        ox, oy = event.x/X, event.y/Y  # old-calibrated
        indicator(sx, sy, ox, oy, '#00F')

        ux, uy = transform(ox, oy, old_cal_inv)  # uncalibrated
        indicator(sx, sy, ux, uy, '#000')

        if new_cal is not None:
            nx, ny = transform(ux, uy, new_cal)  # new-calibrated (test only)
            indicator(sx, sy, nx, ny, '#0F0')

        point.update({'mx': ux, 'my': uy})
        points.append(point)

        canvas.after(500, next_point)
    canvas.bind('<Button-1>', click)

    root.mainloop()

    return points


def fit(screen_pts, mouse_pts) -> tuple:
    from math import log10
    m_screen = np.matrix([[*p, 1] for p in screen_pts])
    m_mouse = np.matrix([[*p, 1] for p in mouse_pts])
    m_transform, residuals, rank, singular = np.linalg.lstsq(m_mouse, m_screen)
    quality = -log10(residuals.sum())
    return m_transform, quality


def calibrate(points, disable_rot) -> tuple:
    if disable_rot:
        '''
        [mx 1] [a 0]   [sx 1]
        [mx 1] [e 1] = [sx 1]
        [... ]         [... ]
        
        [my 1] [d 0]   [sy 1]
        [my 1] [f 1] = [sy 1]
        [... ]         [... ]
        '''
        tx, qual_x = fit(screen_pts=((p['sx'],) for p in points),
                         mouse_pts=((p['mx'],) for p in points))
        ty, qual_y = fit(screen_pts=((p['sy'],) for p in points),
                         mouse_pts=((p['my'],) for p in points))
        m_transform = np.matrix([
            [tx[0, 0], 0,        0],
            [0,        ty[0, 0], 0],
            [tx[1, 0], ty[1, 0], 1]])
        quality = min(qual_x, qual_y)
    else:
        '''
        m_mouse * m_transform = m_screen
        [mx my 1] [a b 0]   [sx sy 1]
        [mx my 1] [c d 0] = [sx sy 1]
        [...    ] [e f 1]   [...    ]
        '''
        m_transform, quality = fit(screen_pts=[(p['sx'], p['sy']) for p in points],
                                   mouse_pts=[(p['mx'], p['my']) for p in points])
        m_transform[:, 2] = ([0], [0], [1])

    m_transform = m_transform.getT()
    return m_transform, quality


def use_cal(dev, new_cal) -> None:
    cal_array = [str(x)+',' for x in new_cal.flatten().tolist()[0]]
    xinput('--set-prop', str(dev), prop_name, *cal_array)


def print_xorg_config(dev_name, new_cal) -> None:
    print("Create a file (for example 99-libinput-ts-calib.conf) in /usr/share/X11/xorg.conf.d/ and put in the following")
    print()
    xorg_config = (
    'Section "InputClass"\n'
    '    Identifier      "calibration"\n'
    '    MatchProduct    "{}"\n'
    '    Option          "CalibrationMatrix"  "{}"\n'
    'EndSection\n'
    ).format(
        str(dev_name),
        " ".join(map(str, new_cal.flatten().tolist()[0]))
    )
    print(xorg_config)


def main() -> None:
    devs = get_devs()
    print_devs(devs)
    preferred = choose_preferred(devs)
    dev = choose_dev(devs, preferred)
    print()

    old_cal, old_cal_inv = read_cal(dev)

    new_cal = None
    if ask('Calibrate?'):
        n_points = choose_points()
        disable_rot = ask('Disable rotation?')
        print()

        points = show_tk(n_points, old_cal_inv)
        if points:
            new_cal, quality = calibrate(points, disable_rot)

            print('New calibration:')
            print(new_cal)
            print('Quality (should be at least 3): %.1f' % quality)
            print()

    if ask('Test?'):
        n_points = choose_points()
        show_tk(n_points, old_cal_inv, new_cal)

    if new_cal is not None and ask('Use calibration?'):
        use_cal(dev, new_cal)
        print_xorg_config(devs[dev], new_cal)	


if __name__ == '__main__':
    main()
