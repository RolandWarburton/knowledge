# Pipewire and Wireplumber

## Introduction

The following chart describes how audio gets from a program to a speaker (Left to right),
and how audio comes from a microphone to a program (Right to Left).

There are also additional parts between the **server** and **sound card**, these components are
**libraries** and **kernel modules**.

The userspace layer communicates to (usually) alsa via LADSPA which is a standard format to talk
to alsa.

```none
                         userspace layer | kernel   | hardware
                                         | layer    | layer
 +---------+     +--------------+        |          |  +-------+
 | program |---->| pipewire     |---+--------+-------->| sound |
 | (cliet) |<----| (server)     |<--+--------+---------| card  |
 +---------+     +--------------+   |    |   |      |  +-------+
                                    |    |   |      |    
                             gstreamer,  | alsa/oss
                             libcanberra,
                             libpulse,   
                             libalsa     
```

## Alsa

The core of linux audio revolves around alsa which is able to send audio to your sound card.

Alsa is the driver - It talks directly to your hardware.
Alsa is delivered in two parts, a kernel driver, and a user space API that other applications
consume. You always require alsa in any reasonable install.

To control alsa, you need pulseaudio which speaks to alsa.
However instead of using pulse, pipewire replaces pulseaudio


Do I Need Alsa?

Yes you need alsa as it speaks to the hardware directly, it converts from digital to PCM and
the same in reverse. Pipewire does not do this.

Alsa can only send one signal to the hardware in the form of PCM to play it however,
this is implemented with pipewire which muxes audio.

Do I Need Pulse?

It depends, but usually yes. Most people will want to use chromium, chromium by default ships
`libpulse0` in debian as a dependency, `libpulse0` is "used by applications that access a PulseAudio
sound server via PulseAudio's native interface" - this is the core of pulse
(its api and client libraries).

How Do I Know If An Application Is Using Pipewire?

Pipewire provides the `pw-top` command. You may run this command and then play some audio,
then observe if it plays through pipewire by observing it here.

What about applications that are not pipewire aware?

Lets take two examples, chromium, and mpv. As of 2022/11/13 mpv received 
[pipewire support](https://github.com/mpv-player/mpv/releases/tag/v0.35.0). Chromium however does
not afford this feature.

There are several APIs for audio at the client, server, and device level.
At the server level it is most interesting. Since pipewire is replacing pulseaudio and jack, it
provides mechanisms for intercepting applications that will play only to these APIs, 
i.e chromium which wants to send audio to pulseaudio. To resolve this pipewire provides
a pulseaudio compatible daemon that mimics pulseaudio and reroutes audio from pulse received over
that interface to pipewire.

```none
+--------+     +--------------+
|chromium|---->|pipewire-pulse|----+
+--------+     +--------------+    |
                                   v
+---+                              +--------+      +-------+
|mpv|----------------------------->|pipewire|----->| sound |
+---+                              +--------+  |   | card  |
                                               |   +-------+
                                               alsa/oss
```

A note on libpulse0. This library is required by software such as chromium and provides
the "PulseAudio client libraries". For all intents this is pulseaudio. For pipewire-pulse to 
function it requires this package to exist.

## Wireplumber

So what is the purpose of WirePlumber then?

> WirePlumber is a powerful session and policy manager for PipeWire.

Pipewire provides a stream exchange framework which allows it to create and destroy streams from
applications and connect them with devices (sound cards).

However stream exchange also requires a way to define who can talk with whom (policy management) and
which application is going to be connected to which device, how, and when it should be connected.

Pipewire solves this by introducing a controller to manage pipewire sessions and policies.

```none
              +-------------------+
              |    (pipewire)     |
              |   +-----------+   |
              |   |wireplumber|   |
              |   +-----------+   |
              |        |          |
 +---------+  |  +--------------+ |   +-------+
 | program |---->| pipewire     |--+->| sound |
 | (cliet) |<----| (server)     |<-+--| card  |
 +---------+  |  +--------------+ ||  +-------+
              |                   |\-alsa
              +-------------------+
```

Lets address the most important job of WirePlumber for the end user, session management.

Wireplumber watches for streams from applications and makes sure they get linked
to the appropriate devices. The way in which this is done is defined by a rule set that 
WirePlumber reads from.

Wireplumber can also watch devices in addition to applications.
Wireplumber will manage any SPA device that implements the `spa_device` interface 
such as ALSA, V4L2, and bluez5 monitors 
([source](https://www.collabora.com/news-and-blog/blog/2020/05/07/wireplumber-the-pipewire-session-manager/))
and turn them on and off as required.

The next job of WirePlumber is client permissions (policy management). I won't cover this
as there is not much i really know about it.

### WirePlumber Endpoints/Nodes And Use Cases

Within PipeWire a graph component is called a "node" which are linked to each other to wire audio.

Another concept is a "use case", this can be thought of as an audio stream such
as the one on android phones (notification sounds, vs media, vs calls).
Conceptualize a use case as having a single output "laptop speakers",
and having all your applications, your music player, and notification daemon,
plugged into the laptop speakers.

```none
                      +------------------------------+
                      |         laptop speakers      |
+------------+    +------+  +--------+---+           |
|media player|--->|music |->|DSP node|   v           |
+------------+    +------+  +--------+ +-----------+ |
  |                   |                |alsa device| |
  \music use case +------+  +--------+ +-----------+ |
                  |notif |->|DSP node|   ^           |
                  +------+  +--------+---+           |
                  |   |                              |
                  |   +------------------------------+
                  |
                  \notification use case
```
