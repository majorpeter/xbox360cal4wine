# Using an XBOX 360 Wireless Controller with Wine under Arch Linux

## Initial steps

Connect your _Xbox 360 Wireless Receiver for Windows_ to one of the USB ports. The kernel should find the joystick and create the necessary files under `/dev/input`.

On Arch Linux, you also want to install the following packages:
* `linuxconsole` (community)

   Contains console utilities to calibrate the _joystick_.

* `jstest-gtk-git` (AUR)

   Simple graphical tool to test and calibrate the _joystick_.

## Calibration

The easiest way to calibrate a joystick is via `jstest-gtk` graphically. First organize the axes the the right order (Mapping), and then configure the deadzones etc.

`jscal` can be used to export mapping calibration (see its manpage). The exported test is actually a `jscal` command which can be run in a shell script.

**IMPORTANT:** Always run mapping command BEFORE calibration command (`-u` before `-s`).

My calibration data is in `xboxjscal.sh`. Running this after enabling the controller (every single time) configures it correctly.

## Udev rule

Udev can be used to run commands when a device is connected.

Get the attributes:
```sh
$ udevadm info -a -p $(udevadm info -q path -n /dev/input/js0)
[...]
    ATTRS{idProduct}=="0719"
    ATTRS{idVendor}=="045e"
[...]
    ATTRS{product}=="Xbox 360 Wireless Receiver for Windows"
```

Create a rule for the given product from the given vendor under the input substystem:
```
SUBSYSTEM=="input", ATTRS{idVendor}=="045e", ATTRS{idProduct}=="0719", ACTION=="add", RUN+="/usr/bin/xboxjscal.sh"
```

Copy the `xboxjscal.sh` to `/usr/bin` and the `85-xboxjscal.rules` under `/etc/udev/rules`.

### Testing

The following command lists the actions taken when js0 is connected.
```
# udevadm test $(udevadm info -q path -n /dev/input/js0)
```

If the ID's are correct, you should see the following line in its output:
```
run: '/usr/bin/xboxjscal.sh'
```

## Wine configuration

Wine detects the device via the older `Joystick API` and the newer `evdev API`. The configuration above is for Joystick API. So it is a good idea to disable the `evdev` controller in Wine's Control Panel.

* To make sure Wine detects the controller, run explorer instead of the actual Control Panel applet:
```
wine explorer
```
* Push a button on the controller (this attaches the device to the current Wine session (?))

* Navigate to `My Computer`/`Control Panel`/`Game Controllers`
* Blacklist the `evdev` controller

## See also
* https://wiki.archlinux.org/index.php/Gamepad
* https://wiki.archlinux.org/index.php/udev#Writing_udev_rules
