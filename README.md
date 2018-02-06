*This is a BETA version! It works, but there is still A LOT to do.*

---


# Linux Driver for Xbox One S Wireless Gamepad

This is a driver for the Xbox One S Wireless Gamepad which I created for a student research project at fortiss GmbH.

At the moment of development there was no driver available which supports force feedback (Rumble) - at least not for the wireless version (Bluetooth). There still is none at January 2018), but until this driver is in a bit more presentable condition I won't submit it to the linux kernel.

The buildsystem consists not only of the driver itself, it also fixes a bug I found in L2CAP (which initially forced us to disable ertm completely), futhermore it adds the new driver to the hid-core (this way it automatically loads the new driver whenever the controller is registered). As an alternative, it offers a Udev-rule to load the driver, this is a workaround which is useful whenever you are not able to recompile hid-core (e.g. if it is not a module - e.g. on Raspbian - and you don't want to recompile the whole kernel).

## Build

You have to build the driver yourself if there is no suitable version available in out/.
To make life a bit easier, we offer you a Taskfile at the moment (you will need go-task https://github.com/go-task/task to execute that).

- build driver(s) for your local system
  ```
  task
  ```
  Alternatively, you can also run either `task default` or `task local`

- build driver(s) for raspberry pi 3 (linux-raspberrypi-kernel_1.20171029-1)
  ```
  task raspi3
  ```

## Installation

To install the driver after building, simply copy the hid-xpadneo module to extramodules and replace the other two modules in your system:

### Remove the original versions

  ```
  sudo mv $(modinfo -n bluetooth) $(modinfo -n bluetooth)_bck
  sudo mv $(modinfo -n hid) $(modinfo -n hid)_bck
  ```

### Install the new ones

```
cd out/<YOUR_ARCH>/<YOUR_KERNEL>
sudo cp ./hid-xpadneo.ko /lib/modules/<YOUR_KERNEL>/extramodules/hid-xpadneo.ko
sudo cp ./hid.ko /lib/modules/<YOUR_KERNEL>/kernel/drivers/hid/hid.ko
sudo cp ./bluetooth.ko /lib/modules/<YOUR_KERNEL>/kernel/net/bluetooth/bluetooth.ko
```

Register the new drivers by running `sudo depmod` afterwards.


#### Problems

- If bluetooth isn't a module, you can alternatively run `sudo /bin/bash -c "echo 1 > /sys/module/bluetooth/parameters/disable_ertm"` before connecting the controller to the computer.

- If hid isn't a module, you can alternatively copy `99-xpadneo.rules` to `/etc/udev/rules.d/` (run `udevadmn control --reload` afterwards)