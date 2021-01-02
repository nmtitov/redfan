# Installation instructions

1. Download `zip` archive of the `main` branch
```
wget -c https://github.com/nmtitov/amdgpufan/archive/main.zip
```

2. `sudo apt install unzip -y`

3. `unzip amdgpufan-main.zip`

4. `cd amdgpufan-main`

5. `cp amdgpufan /usr/bin/`

6. `cp amdgpufan.service /etc/systemd/system/`

7. `sudo systemctl daemon-reload`

8. `sudo systemctl start amdgpufan`

9. `sudo systemctl enable amdgpufan`

10. `sudo systemctl status amdgpufan`

# Description

I'm trying to lock RPM of my AMD Radeon videocard fans at the full speed:

```
echo 1 > /sys/class/hwmon/hwmon1/pwm1_enable
echo 255 > /sys/class/hwmon/hwmon1/pwm1
```

### What I have tried so far

Obviously, it doesn't work due to missing permissions (even with `sudo`/`root`) because it is `/sys`:

```
$ sudo su
$ echo 255 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1
bash: echo: write error: Invalid argument
```

I have also tried `sysfs` config to edit these params but it didn't work:
```
$ cat /etc/sysfs.conf
class/drm/card1/device/hwmon/hwmon1/pwm1 = 255
class/drm/card1/device/hwmon/hwmon1/pwm1_enable = 1
```

`echo 5 | sudo tee ...` also doesn't work.

Neither does `sudo sh -c`:

```
sudo sh -c 'echo 225 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1'
sh: 1: echo: echo: I/O error
```

Archilinux Wiki states it should be possible though https://wiki.archlinux.org/index.php/fan_speed_control#Configuration_of_manual_control They edit values directly with `echo` and looks like it works for them. 

Another guide also recommends configuring fans this way https://linuxconfig.org/overclock-your-radeon-gpu-with-amdgpu

Python `amdgpu-fan` package also doesn't work for me. 

`sudo fancontrol` doesn't work as well:

```
$ sudo fancontrol
Loading configuration from /etc/fancontrol ...

Common settings:
  INTERVAL=10

Settings for hwmon1/pwm1:
  Depends on hwmon1/temp1_input
  Controls 
  MINTEMP=10
  MAXTEMP=60
  MINSTART=50
  MINSTOP=0
  MINPWM=0
  MAXPWM=255
  AVERAGE=1

Enabling PWM on fans...
Starting automatic fan control...
/usr/sbin/fancontrol: line 649: echo: write error: Invalid argument
Error writing PWM value to /sys/class/hwmon/hwmon1/pwm1
Aborting, restoring fans...
Verify fans have returned to full speed
```

Daemon (service) also doesn't work:

```
fancontrol[1877]:   MAXPWM=255
fancontrol[1877]:   AVERAGE=1
fancontrol[1877]: Enabling PWM on fans...
fancontrol[1877]: Starting automatic fan control...
fancontrol[1877]: /usr/sbin/fancontrol: line 649: echo: write error: Invalid argument
fancontrol[1877]: Error writing PWM value to /sys/class/hwmon/hwmon1/pwm1
fancontrol[1877]: Aborting, restoring fans...
fancontrol[1877]: Verify fans have returned to full speed
systemd[1]: fancontrol.service: Main process exited, code=exited, status=1/FAILURE
systemd[1]: fancontrol.service: Failed with result 'exit-code'.
```

To sum up: seems that I can't edit /sys/ amdgpu-related entries at all

### Part 2

It seems that there has to be another way around, like some `amdgpu` config or something like that. Maybe override kernel-defined values during boot?

In Windows, it's possible to tune fans directly from the AMD Radeon driver GUI app.

I don't want fancy curves, I'm simply trying to force lock static RPM (full-on mode). I'm using `amdgpu-pro` drivers, Ubuntu 20.04. I'd like to avoid using scripts like `fancontrol`

### The question itself

I wonder if that's possible just to set `pwm1_enable` to `1` and `pwm1` to `255`? Looks like the suggested method should be working, but Ubuntu 20.04 security limitations are more restrictive than other distros' ones. 

**update**
This thing works! But only for 1-2 seconds, after that, fans go back to system-defined speed `https://github.com/DominiLux/amdgpu-pro-fans/blob/master/amdgpu-pro-fans.sh`

**update 2**

Disabling pwm works for about 1-2 seconds.
`echo 0 > /sys/class/hwmon/hwmon1/pwm1_enable`

But after that, some daemon reverts this value back to 2. How could I prevent it from changing by other users except me? E.g. prevent it from changing by the system?

https://unix.stackexchange.com/questions/627182/how-to-lock-fan-speed-for-amd-gpu-in-ubuntu-20-04

