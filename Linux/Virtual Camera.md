# Virtual Camera

Setting up a virtual camera is a useful tool to have available when you need to stream your desktop over a video chat app such as zoom that doesn't support screen share.

## Install requirements

You will need the following tools.

1. **v4l2loopback-dkms** to create a video loopback
2. **v4l2loopback-utils** to support webcam stream formats
3. **ffmpeg** to convert the desktop video stream to /dev/video0

Optional requirements.

1. **mon2cam** for multi-head support (link [here](https://github.com/ShayBox/Mon2Cam))
2. **mpv** to test your webcam (optional)

Note: mon2cam is just one way to support multi-head. There is a better way to do this with just ffmpeg.

## FFmpeg method

This method is harder to set up but makes more sense and has less moving parts once you get it going.

### Determine window resolution

First, we need to determine the resolutions and offsents for each monitor. Using nvidia-settings this can be quite easy.

Under the **position** we can see the offset of this screen (+0+240), and the size of it (1080x1920).

![nvidia_settins](https://i.imgur.com/jEMgabd.png)

If you dont have an nvidia card, you can also use a trick with `xwininfo | grep geometry` which will give you the geometry of the window. Just full screen a window, run this command, and click on it to have the geometry printed in the terminal.

![xwininfo](https://i.imgur.com/2VXP2D7.png)

However, you can see that this is not exactly accurate as the offset is not correctly reported (nvidia tools method is accurately reporting the offset in my situation).

### Stream with FFmpeg

Now that we know the dimensions we can create a video stream with ffmpeg.

* `-f` for input/output. `-f x11grab` grabs the screen. Then `-f v4l2 /dev/video0` pipes it to video0 as a readable stream.
* `-r` for framerate.
* `-video_size` for the resolution. This changes between 1920x1080 and 1080x1920 depending on the screen i am recording.
* `-grab_x` and `grab_y` for the offset.
* `-i` for the screen to record. I have one big X screen thats 4080x2160 if you stick all the monitors together.
* `-vcodec` and `-pix_fmt` and `-threads` remain the same between all commands (no need to change them).

to grab the primary bottom screen.

```none
sudo ffmpeg -f x11grab -r 60 -video_size 1920x1080 -grab_x 1080 -grab_y 1080 -i :0.0 -vcodec rawvideo -pix_fmt yuv420p -threads 0 -f v4l2 /dev/video0
```

to grab the top screen.

```none
sudo ffmpeg -f x11grab -r 60 -video_size 1920x1080 -grab_x 1080 -grab_y 0 -i :0.0 -vcodec rawvideo -pix_fmt yuv420p -threads 0 -f v4l2 /dev/video0
```

to grab the left screen.

```none
sudo ffmpeg -f x11grab -r 60 -video_size 1080x1920 -grab_x 3000 -grab_y 240 -i :0.0 -vcodec rawvideo -pix_fmt yuv420p -threads 0 -f v4l2 /dev/video0
```

to grab the right screen
sudo ffmpeg -f x11grab -r 60 -video_size 1080x1920 -grab_x 0 -grab_y 240 -i :0.0 -vcodec rawvideo -pix_fmt yuv420p -threads 0 -f v4l2 /dev/video0

Then test with ffplay.

```none
ffplay /dev/video0
```

### Adding to OBS

Use a V4L2 source.

![obs_01](https://i.imgur.com/jj9Pj7X.png)
![obs_02](https://i.imgur.com/Efy7x6O.png)
![obs_03](https://i.imgur.com/92pWTcc.png)

## Mon2Cam method

This method i did a long time ago, i suggest using the above ffmpeg version.

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
