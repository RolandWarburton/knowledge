# Fan speed and PWM config setup examples

Sample output for my new computer which i am diagnosing with low RPM cpu fan speed issues.

Make sure you have lm-sensors installed first and have run and configured sensors-detect, and have the correct kernel modules loaded into your system (i was missing nct6779 for my asus p877v pro motherboard).

It essentially looks for pwm (pulse width modulation - turning something on and off really fast to make it spin at a certain speed) by changing different pwm controllers (hwmon\[num]/pwm\[num]) to associate the pwm controller with a fan, we can then use the pwmconfig script to configure that fan temp thresholds by binding a temperature sensor (which pwmconfig also helps you discover) with the pwm controller.

```output
roland@debian:~$ sudo pwmconfig
# pwmconfig revision $Revision$ ($Date$)
This program will search your sensors for pulse width modulation (pwm)
controls, and test each one to see if it controls a fan on
your motherboard. Note that many motherboards do not have pwm
circuitry installed, even if your sensor chip supports pwm.

We will attempt to briefly stop each fan using the pwm controls.
The program will attempt to restore each fan to full speed
after testing. However, it is ** very important ** that you
physically verify that the fans have been to full speed
after the program has completed.

Found the following devices:
   hwmon0 is acpitz
   hwmon1 is coretemp
   hwmon2 is nct6779
   hwmon3 is asus

Found the following PWM controls:
   hwmon2/pwm1           current value: 153
hwmon2/pwm1 is currently setup for automatic speed control.
In general, automatic mode is preferred over manual mode, as
it is more efficient and it reacts faster. Are you sure that
you want to setup this output for manual control? (n) y
   hwmon2/pwm2           current value: 76
hwmon2/pwm2 is currently setup for automatic speed control.
In general, automatic mode is preferred over manual mode, as
it is more efficient and it reacts faster. Are you sure that
you want to setup this output for manual control? (n) y
   hwmon2/pwm3           current value: 153
hwmon2/pwm3 is currently setup for automatic speed control.
In general, automatic mode is preferred over manual mode, as
it is more efficient and it reacts faster. Are you sure that
you want to setup this output for manual control? (n) y
   hwmon2/pwm4           current value: 153
hwmon2/pwm4 is currently setup for automatic speed control.
In general, automatic mode is preferred over manual mode, as
it is more efficient and it reacts faster. Are you sure that
you want to setup this output for manual control? (n) y
   hwmon2/pwm5           current value: 153
hwmon2/pwm5 is currently setup for automatic speed control.
In general, automatic mode is preferred over manual mode, as
it is more efficient and it reacts faster. Are you sure that
you want to setup this output for manual control? (n) y

Giving the fans some time to reach full speed...
Found the following fan sensors:
   hwmon2/fan1_input     current speed: 0 ... skipping!
   hwmon2/fan2_input     current speed: 1668 RPM
   hwmon2/fan3_input     current speed: 0 ... skipping!
   hwmon2/fan4_input     current speed: 0 ... skipping!
   hwmon2/fan5_input     current speed: 2086 RPM
   hwmon3/fan1_input     current speed: 0 ... skipping!

Warning!!! This program will stop your fans, one at a time,
for approximately 5 seconds each!!!
This may cause your processor temperature to rise!!!
If you do not want to do this hit control-C now!!!
Hit return to continue:

Testing pwm control hwmon2/pwm1 ...
  hwmon2/fan2_input ... speed was 1668 now 1713
    no correlation
  hwmon2/fan5_input ... speed was 2086 now 2086
    no correlation

No correlations were detected.
There is either no fan connected to the output of hwmon2/pwm1,
or the connected fan has no rpm-signal connected to one of
the tested fan sensors. (Note: not all motherboards have
the pwm outputs connected to the fan connectors,
check out the hardware database on http://www.almico.com/forumindex.php)

Did you see/hear a fan stopping during the above test (n)? n

Testing pwm control hwmon2/pwm2 ...
  hwmon2/fan2_input ... speed was 1668 now 601
    It appears that fan hwmon2/fan2_input
    is controlled by pwm hwmon2/pwm2

Would you like to generate a detailed correlation (y)? y
    PWM 255 FAN 1708
    PWM 240 FAN 1638
    PWM 225 FAN 1542
    PWM 210 FAN 1486
    PWM 195 FAN 1381
    PWM 180 FAN 1301
    PWM 165 FAN 1220
    PWM 150 FAN 1135
    PWM 135 FAN 1039
    PWM 120 FAN 939
    PWM 105 FAN 863
    PWM 90 FAN 770
    PWM 75 FAN 663
    PWM 60 FAN 625
    PWM 45 FAN 601
    PWM 30 FAN 596
    PWM 28 FAN 595
    PWM 26 FAN 605
    PWM 24 FAN 593
    PWM 22 FAN 593
    PWM 20 FAN 606
    PWM 18 FAN 608
    PWM 16 FAN 594
    PWM 14 FAN 606
    PWM 12 FAN 604
    PWM 10 FAN 592
    PWM 8 FAN 601
    PWM 6 FAN 603
    PWM 4 FAN 591
    PWM 2 FAN 591
    PWM 0 FAN 601

hwmon2/fan5_input ... speed was 2086 now 2099
    no correlation

Testing pwm control hwmon2/pwm3 ...
  hwmon2/fan2_input ... speed was 1668 now 1713
    no correlation
  hwmon2/fan5_input ... speed was 2086 now 2073
    no correlation

No correlations were detected.
There is either no fan connected to the output of hwmon2/pwm3,
or the connected fan has no rpm-signal connected to one of
the tested fan sensors. (Note: not all motherboards have
the pwm outputs connected to the fan connectors,
check out the hardware database on http://www.almico.com/forumindex.php)

Did you see/hear a fan stopping during the above test (n)? n

Testing pwm control hwmon2/pwm4 ...
  hwmon2/fan2_input ... speed was 1668 now 1687
    no correlation
  hwmon2/fan5_input ... speed was 2086 now 2073
    no correlation

No correlations were detected.
There is either no fan connected to the output of hwmon2/pwm4,
or the connected fan has no rpm-signal connected to one of
the tested fan sensors. (Note: not all motherboards have
the pwm outputs connected to the fan connectors,
check out the hardware database on http://www.almico.com/forumindex.php)

Did you see/hear a fan stopping during the above test (n)? n

Testing pwm control hwmon2/pwm5 ...
  hwmon2/fan2_input ... speed was 1668 now 1693
    no correlation
  hwmon2/fan5_input ... speed was 2086 now 0
    It appears that fan hwmon2/fan5_input
    is controlled by pwm hwmon2/pwm5

Would you like to generate a detailed correlation (y)? y
    PWM 255 FAN 2080
    PWM 240 FAN 2070
    PWM 225 FAN 1976
    PWM 210 FAN 1882
    PWM 195 FAN 1785
    PWM 180 FAN 1693
    PWM 165 FAN 1567
    PWM 150 FAN 1467
    PWM 135 FAN 1348
    PWM 120 FAN 1220
    PWM 105 FAN 1102
    PWM 90 FAN 958
    PWM 75 FAN 0
    Fan Stopped at PWM = 75


Testing is complete.
Please verify that all fans have returned to their normal speed.

The fancontrol script can automatically respond to temperature changes
of your system by changing fanspeeds.
Do you want to set up its configuration file now (y)? y
What should be the path to your fancontrol config file (/etc/fancontrol)?

Select fan output to configure, or other action:
1) hwmon2/pwm5
2) hwmon2/pwm2
3) Change INTERVAL
4) Just quit
5) Save and quit
6) Show configuration
select (1-n): 6

Common Settings:
INTERVAL=10

Settings of hwmon2/pwm5:
  Depends on
  Controls
  MINTEMP=
  MAXTEMP=
  MINSTART=
  MINSTOP=

Settings of hwmon2/pwm2:
  Depends on
  Controls
  MINTEMP=
  MAXTEMP=
  MINSTART=
  MINSTOP=


Select fan output to configure, or other action:
1) hwmon2/pwm5
2) hwmon2/pwm2
3) Change INTERVAL
4) Just quit
5) Save and quit
6) Show configuration
select (1-n): 5

Saving configuration to /etc/fancontrol...
Configuration saved
roland@debian:~$
```

---

```none
sudo vim /etc/fancontrol
```

By default when we ignore option 1 and 2 (save without configuring) we get this result in `/etc/fancontrol`

```output
# Configuration file generated by pwmconfig, changes will be lost
INTERVAL=10
DEVPATH=
DEVNAME=
FCTEMPS=
FCFANS=
MINTEMP=40
MAXTEMP=50
MINSTART=150
MINSTOP=0
```

Using the [fancontrol manual](https://www.systutorials.com/docs/linux/man/8-fancontrol/) we can see the different config options. Also worth checking out is the [arch wiki page](https://wiki.archlinux.org/index.php/fan_speed_control) for fan speed control which covers pwmconfig as well.

Here are some [other resources](https://askubuntu.com/questions/1187812/cannot-configure-fan-speed-with-pwmconfig) that i got the below code block from. and [another real life example](https://askubuntu.com/questions/22108/how-to-control-fan-speed) of someone elses `/etc/fancontrol` config.

```output
# Checks the temperature every 10 seconds.
INTERVAL=10

# Maps a fan to a temp sensor, each separated by a space 
FCTEMPS=fanpath=temppath fanpath2=temppath2

# Maps a fan to the fan speed sensor
FCFANS=fanpath=fanspeedpath fanpath2=fanspeedpath2

# The temperature below which the fan gets switched to minimum speed.
MINTEMP=fanpath=degreesC fanpath2=degreesC2

# The temperature over which the fan gets switched to maximum speed.
MAXTEMP=fanpath=degreesC fanpath2=degreesC2

# Sets the minimum speed at which the fan begins spinning.
MINSTART=fanpath=minspeed fanpath2=minspeed2

#The minimum speed at which the fan still spins.
MINSTOP=fanpath=minspeed fanpath2=minspeed2
```

The hardest things to configure is the DEVPATH and DEVNAME which map temp sensors to fans, its best to just do this through the guided `pwmconfig` setup script, re run `sudo pwmconfig` and select option 1 or 2 (your hwmon\[num]/pwm\[num] configuration options).

In my situation option 2 is the CPU fan, and the fan i want to configure, so i select option 2.
To find out the fan you want to control, navigate to `/sys/class/hwmon` and investigate the files here to look for `coretemp` (ie. cpu core temp), amongst other hardware monitoring probes that i'm not too knowledgeable about.

```output
cd /sys/class/hwmon

cat /sys/class/hwmon/hwmon1/name
coretemp
```

## Pre-reqs - Diagnosing pwm and temp sensors not being detected correctly

Make sure you have lm-sensors (lm_sensors on arch linux) installed before running pwnconfig. pwmconfig is bundled with lm-sensors.

Run `sensors-detect` and answer yes or no when prompted.

now running the `sensors` command will SHOULD provide verbose output of different temp and fan speeds. To monitor these you can use these commands.

```bash
# monitor the core temps
sudo watch -n 1 "sensors | grep -i core"
```

```bash
# monitor the fan speeds
sudo watch -n 1 "sensors | grep -i fan"
```

```bash
# or both at the same time!
sudo watch -n 1 "sensors | grep -i -E 'fan|core'"
```

docs for the coretemp are [here](https://www.kernel.org/doc/html/latest/hwmon/coretemp.html) which is how i discovered that core temp was associated with cpu cores.

pwmconfig also gave us some other drivers to look into

```output
hwmon0 is acpitz
   hwmon1 is coretemp
   hwmon2 is nct6779
   hwmon3 is asus
```

Just to make sure we should load these modules, im not sure what `hwmon3 is asus` actually means, however i know that nct6779 is a module that can be loaded into the kernel using `modprobe nct6779` and then rebooting.

Here is the real world example for my configured CPU fan after running through the script again.

```none
# Configuration file generated by pwmconfig, changes will be lost
INTERVAL=10
DEVPATH=hwmon1=devices/platform/coretemp.0 hwmon2=devices/platform/nct6775.656
DEVNAME=hwmon1=coretemp hwmon2=nct6779
FCTEMPS=hwmon2/pwm2=hwmon1/temp1_input
FCFANS= hwmon2/pwm2=hwmon2/fan2_input
MINTEMP=hwmon2/pwm2=20
MAXTEMP=hwmon2/pwm2=50
MINSTART=hwmon2/pwm2=150
MINSTOP=hwmon2/pwm2=0
MAXPWM=hwmon2/pwm2=150
```

## Using the script to configure fanspeeds

After re-reunning `sudo pwmconfig` and then selecting **option 2** (my CPU fan)

```output
Select fan output to configure, or other action:
1) hwmon2/pwm5
2) hwmon2/pwm2
3) Change INTERVAL
4) Just quit
5) Save and quit
6) Show configuration
select (1-n): 2

Devices:
hwmon0 is acpitz
hwmon1 is coretemp
hwmon2 is nct6779
hwmon3 is asus

Current temperature readings are as follows:
hwmon0/temp1_input      27
hwmon0/temp2_input      29
hwmon1/temp1_input      37
hwmon1/temp2_input      37
hwmon1/temp3_input      31
hwmon1/temp4_input      37
hwmon1/temp5_input      27
hwmon2/temp1_input      -62
hwmon2/temp2_input      28
hwmon2/temp3_input      -62
hwmon2/temp4_input      -62
hwmon2/temp5_input      -62
hwmon2/temp6_input      -62
hwmon2/temp7_input      28
hwmon2/temp8_input      0
hwmon2/temp9_input      0

Select a temperature sensor as source for hwmon2/pwm2:
1) hwmon0/temp1_input                      5) hwmon1/temp3_input                     9) hwmon2/temp2_input                    13) hwmon2/temp6_input                    17) None (Do not affect this PWM output)
2) hwmon0/temp2_input                      6) hwmon1/temp4_input                    10) hwmon2/temp3_input                    14) hwmon2/temp7_input
3) hwmon1/temp1_input                      7) hwmon1/temp5_input                    11) hwmon2/temp4_input                    15) hwmon2/temp8_input
4) hwmon1/temp2_input                      8) hwmon2/temp1_input                    12) hwmon2/temp5_input                    16) hwmon2/temp9_input
select (1-n): 3

Enter the low temperature (degree C)
below which the fan should spin at minimum speed (20): 20

Enter the high temperature (degree C)
over which the fan should spin at maximum speed (60): 50

Enter the PWM value (0-255) to use when the temperature
is over the high temperature limit (255): 70

Select fan output to configure, or other action:
1) hwmon2/pwm5
2) hwmon2/pwm2
3) Change INTERVAL
4) Just quit
5) Save and quit
6) Show configuration
select (1-n): 6

Common Settings:
INTERVAL=10

Settings of hwmon2/pwm5:
  Depends on
  Controls
  MINTEMP=
  MAXTEMP=
  MINSTART=
  MINSTOP=

  Settings of hwmon2/pwm2:
  Depends on hwmon1/temp1_input
  Controls hwmon2/fan2_input
  MINTEMP=20
  MAXTEMP=50
  MINSTART=150
  MINSTOP=0
  MAXPWM=70


Select fan output to configure, or other action:
1) hwmon2/pwm5
2) hwmon2/pwm2
3) Change INTERVAL
4) Just quit
5) Save and quit
6) Show configuration
select (1-n): 5

Saving configuration to /etc/fancontrol...
Configuration saved
roland@debian:~$
```

Now run these commands to make sure fancontrol is started, running smoothly, and is enabled for next boot.

```none
sudo systemctl start fancontrol
sudo systemctl status fancontrol
sudo systemctl enable fancontrol
```
