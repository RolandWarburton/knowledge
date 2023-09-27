# Systemd Power

Configuring power states can be done through systemd.

## Power States

A power state is a set of features that can be turned on or off to achieve a low power state.

There are many power states, such as `poweroff`, `reboot`, `suspend`, and `hibernate`
which can be set with `systemctl POWERSTATE`.

The interesting ones that will be covered here are **suspend** and **hibernate**.

Suspend puts the system into a suspended state in which the system consumes less power,
but the system can recover quickly.

Hibernate puts the system into a hibernation state in which the system saves its memory to *swap*,
the system is then mostly powered off,
when it resumes it loads this system state from swap and restores it to RAM.

## Configuration

Fist check `/etc/systemd/sleep.conf`.

Here we can `AllowSuspend`, `AllowHibernation`, `AllowSuspendThenHibernate`, and `AlloyHybridSleep`.

Hybrid Sleep is a low power state where OS execution is paused and saved to disk
(or suspend-to-both as its known in kernel space) It works by putting the system state
in RAM and on disk (swap) allowing for fast recovery,
and reduced chance of data loss in the event of complete power loss.

Suspend then hibernate is a mode which first combines suspend, and then hibernates when
the battery is low to prevent data loss.

### Suspend Mode and Suspend State

Mode defines what string will be written to `/sys/power/disk`.

`SuspendMode` has two values.

* suspend
* hibernate

Its best to leave this on `suspend`, actions such as closing the lid will result in
the suspend action being called, if you would prefer to hibernate, then change this to `hibernate`.

You may also stack this to try suspend, and then hibernate, this is useful to provide resilience for
systems that may not support suspending (this is unlikely).

```none
SuspendMode=suspend hibernate
```

State defines what string will be written to `/sys/power/state`.

`SuspendState` has 3 values.

* mem - suspend to RAM and power down
* standby - suspend to RAM and enter low CPU power mode
* freeze - suspend to disk and power down

These should be stacked to try each in order they are listed from left to right.

```none
SuspendState=mem standby freeze
```

### Hibernate Mode and Hibernate State

Mode defines what string will be written to `/sys/power/disk`.

`HibernateMode` has two options.

* platform - Hibernate with platform specific mechanisms
* shutdown - hibernate by writing to disk and shutting down

This should be stacked as well.

```none
HibernateMode=platform shutdown
```

State defines what string will be written to `/sys/power/state`.

`HibernateState` has two options.

* platform - Hibernate using platform-specific mechanisms. \
This value allows the system to use any platform-specific mechanisms that are available for \
hibernation, such as ACPI S4 (also known as "suspend-to-disk"). \
* disk - Hibernate to disk and then power off

These should be stacked to try and use any platform specific hibernation features.

```none
HibernateState=platform disk
```

## Example Config

```none
AllowSuspend=yes
AllowHibernation=yes
AllowSuspendThenHibernate=no
AllowHybridSleep=no
SuspendMode=suspend
SuspendState=mem standby freeze
HibernateMode=platform shutdown
HibernateState=platform disk
HybridSleepMode=suspend platform shutdown
HybridSleepState=disk
HybridDelaySec=120
```

## Resources

* [freedesktop.org/systemd-sleep.conf](https://www.freedesktop.org/software/systemd/man/systemd-sleep.conf.html)
* [arch wiki pwoer management with systemd](https://wiki.archlinux.org/title/Power_management#Power_management_with_systemd)
