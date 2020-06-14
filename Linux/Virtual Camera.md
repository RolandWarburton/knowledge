# Virtual Camera

Setting up a virtual camera is a useful tool to have available when you need to stream your desktop over a video chat app such as zoom that doesn't support screen share.

### Install requirements

You will need the following tools.

1. **v4l-utils** to support webcam stream formats
2. **v4l2loopback-dkms** to create a video loopback
3. **mon2cam** for multi-head support (link [here](https://github.com/ShayBox/Mon2Cam))
4. **mpv** to test your webcam (optional)
5. **obs-v4l2sink** for virtual camera support on obs (optional)

### Instructions

Install all the above tools. Make sure the location of mon2cam is in your path (`echo $PATH`)

```none
sudo pacman -S v4l-utils v4l2loopback-dkms mpv obs-v4l2sink
curl https://raw.githubusercontent.com/ShayBox/Mon2Cam/master/Mon2Cam.sh > ~/bin/mon2cam.sh
sudo chmod 755 ~/bin/mon2cam.sh
```

Next load v4l2loopback using modprobe. If you have issues doing this (cant find v4l2loopback error) restarting should fix it.

```none
sudo modprobe v4l2loopback
```

If you have one monitor you should already see a new video device in `/dev/video*`. Check you have a video device.

```none
ls /dev/video*
```

After confirming you have a video device. Run mon2cam if you have multiple monitors.
Otherwise if you dont have multiple monitors you are more or less done. When you run any web conference application (excluding OBS, see ahead) you can use your new input source **dummy-camera** to share your screen instead of your face.

```none
mon2cam.sh

Monitors: 3
 0: +*HDMI-0 1080/598x1920/336+0+173  HDMI-0
 1: +HDMI-1 1920/598x1080/336+1080+495  HDMI-1
 2: +DP-1 1080/531x1920/298+3000+0  DP-1
```

### OBS

Getting virtual cameras working in OBS is super easy. Make sure you have **obs-v4l2sink** installed. Restart OBS and go to **tools -> v4l2sink** and select your virtual camera. Then go ahead and add a *video Capture Device* and select your video\[number\] device.

### Testing

Run mpv and point it at your virtual device to see it working in action. For web testing use a webcam testing site like [this one](https://webcamtests.com/) or even [this one](https://www.onlinemictest.com/webcam-test/).

MPV testing

```none
mpv av://v4l2:/dev/video[number]
```
