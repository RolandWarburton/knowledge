# Networked Webcams

This note is to serve as a companion for the camscripts repo on my github [here](https://github.com/RolandWarburton/camScripts). Specifically converting a usb webcam to an ip camera which is [here](https://github.com/RolandWarburton/camScripts/tree/master/usbCam2ip).

## Goals

The goals of the cam 2 ip project was to simulate what you would expect to see out of an IP security camera, that being RTSP access to the camera from anywhere within the local network.

## Gathering Inforation

I have two cameras i am testing on.

1. Logitech c127 (budget range 720p) (will be /dev/video2)
2. Rando cheap ebay ($10 super budget 720p) (will be /dev/video0)

### Camera Details

I need to determine what formats these cameras work with. As i know that security cameras work with h264 mostly it would be ideal to get that straight from the camera, unfortunately as expected these cameras are not good enough for that as seen below.

#### Extract Supported Containers

To extraact the camera details i used the following command. Replace `/dev/video0` with your device.

```none
ffmpeg -f v4l2 -list_formats all -i /dev/video0
```

```output
# the logitech camera
ffmpeg -f v4l2 -list_formats all -i /dev/video2
[video4linux2,v4l2 @ 0x55eb36e96280] Raw       :     yuyv422 :           YUYV 4:2:2 : 640x480 352x288 320x240 176x144 160x120 544x288 432x240 320x176 640x360
[video4linux2,v4l2 @ 0x55eb36e96280] Compressed:       mjpeg :          Motion-JPEG : 640x480 352x288 320x240 176x144 160x120 544x288 432x240 320x176 640x360 800x480 1024x768
```

```output
# the cheap ebay camera
ffmpeg -f v4l2 -list_formats all -i /dev/video0
...
[video4linux2,v4l2 @ 0x561f816a2280] Raw       :     yuyv422 :           YUYV 4:2:2 : 1280x720 640x480
[video4linux2,v4l2 @ 0x561f816a2280] Compressed:       mjpeg :          Motion-JPEG : 1280x720 640x480
```

As seen, we can support yuyv422 and mjpeg.

To get the framerates you need to install the v4l2 util package.

```none
sudo apt-get install v4l-utils
```

#### Extract Supported Framerates

Then get the framerates for the cameras.

Use this command:

```none
v4l2-ctl -d /dev/video[number] --list-formats-ext
```

```none
# cheap ebay camera

ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'YUYV' (YUYV 4:2:2)
                Size: Discrete 1280x720
                        Interval: Discrete 0.100s (10.000 fps)
                Size: Discrete 640x480
                        Interval: Discrete 0.040s (25.000 fps)
        [1]: 'MJPG' (Motion-JPEG, compressed)
                Size: Discrete 1280x720
                        Interval: Discrete 0.040s (25.000 fps)
                Size: Discrete 640x480
                        Interval: Discrete 0.040s (25.000 fps)
```

```none
# logitech camera

ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'YUYV' (YUYV 4:2:2)
                Size: Discrete 640x480
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 352x288
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 320x240
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 176x144
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 160x120
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 544x288
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 432x240
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 320x176
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 640x360
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
        [1]: 'MJPG' (Motion-JPEG, compressed)
                Size: Discrete 640x480
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 352x288
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 320x240
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 176x144
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 160x120
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 544x288
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 432x240
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 320x176
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 640x360
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 800x480
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 1024x768
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
```

Now we know all the supported dimensions and frame rates. Lets construct some chosen settings for our ffmpeg commands later. I will just work from the logitech camera from this point out because you can interpret these commands between webcam models.

```output
# logitech camera
-framerate 30 \
-video_size 1024x768 \
```

## Setting up mjpeg Streaming to the Browser

Based on the wizadry [here](https://www.rickmakes.com/streaming-an-ip-camera-to-a-web-browser-using-ffmpeg/) in this blog post. I have modified a setup in the [usbCam2ip](https://github.com/RolandWarburton/camScripts/tree/master/usbCam2ip) repo that uses ffmpeg to stream to a browser using a couple bits of software.

The parts used are ffmpeg, hls.js the JS library, nginx, and thats it.

The job of ffmpeg is to read chunks from the `/dev/video[n]` device and write those in a format that is HLS (http live streaming) friendly to be read. As discussed [here](https://en.wikipedia.org/wiki/HTTP_Live_Streaming#Architecture) on the HLS wiki (under architecture) ffmpeg provides the **server** architecture.

1. FFMPEG reads, and transcodes the webcam mjpeg stream into a H.264 encoded file
2. FFMPEG produces a .m3u8 file which describes the current state of the HLS stream (think of it as a dynamic playlist of .ts files)
3. FFMPEG provides multiple .ts files which contain the H.264 encapsulated video data to send to the client

Next we need the **distributor** which is played by nginx.

Lastly we need the **client** which is played by the hls.js client which knows how to interact with the HLS data ([source](https://github.com/video-dev/hls.js/#supported-m3u8-tags)) provided by FFMPEG through nginx.

### Hardware Encoding

Without hardware encoding my i7 3770k saw quite high (30%) usage just transcoding one low res stream so hardware encoding support is essential.

For hardware support you will need packages for Nvidia/AMD, and may need packages for intel. Check the [debian wiki](https://wiki.debian.org/HardwareVideoAcceleration) for more information.

For **intel** use this ffmpeg command to enable VAAPI (video acceleration api).

```none
ffmpeg \
-f alsa \
-i hw:0 \
-vaapi_device /dev/dri/renderD128 \
-f v4l2 \
-input_format mjpeg \
-i /dev/video2 \
-vf 'format=nv12,hwupload' \
-c:v h264_vaapi \
-hls_time 5 \
-hls_list_size 2 \
-start_number 1 \
-hls_flags delete_segments \
-segment_wrap 10 \
output.m3u8
```

1. `-f alsa -i hw:0` the **f**ormat for alsa: sending audio data from **i**interface hw:0 (not working for me)
2. `-vaapi_device /dev/dri/renderD128` tell ffmpeg where the hardware driver is for vaapi
3. `-f v4l2` **f**ormat for v4l2: set the **i**nput to `/dev/device[n]`
4. `-vf 'format=nv12,hwupload'` is a ffmpeg filter that transforms the normal frame into a vaapi one. See [here](https://stackoverflow.com/questions/45476554/ffmpeg-hwaccel-error-with-hwupload).
5. `-c:v h264_vaapi` set the codec to h264_vaapi so ffmpeg will convert to h264_vaapi
6. `-hls_time 5` various HLS options
7. `-hls_list_size 2` various HLS options
8. `-start_number 1` various HLS options
9. `-hls_flags delete_segments` various HLS options
10. `-segment_wrap 10` various HLS options

For Nvidia and AMD support i needed to install `mesa-va-drivers`, `vdpau-va-driver` and `vainfo` (libva-utils on some older versions of debian <11).

I got an error with `sudo vainfo` which had to be solved. At the current stage i do not have a solution but i have a suspicion that its caused by something else other than the VAAPI.

```none
error: XDG_RUNTIME_DIR not set in the environment.
libva info: VA-API version 1.10.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/nvidia_drv_video.so
libva info: va_openDriver() returns -1
vaInitialize failed with error code -1 (unknown libva error),exit
```

For the current moment this is as far as i got with networked webcams, the take away being that VAAPI is hard to set up and for now i am happy with my learning about ffmpeg and webcams, but i hope to expand on it in future.
