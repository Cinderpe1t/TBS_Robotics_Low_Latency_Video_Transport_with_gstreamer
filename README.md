# TBS Robotics Low Latency Video Transport with gstreamer
TBS Robotics Low Latency Video Transport with gstreamer
## Low latency for real-time robot driving
- This a topic needs more study and improvement
## [gstreamer](https://gstreamer.freedesktop.org/)
- Pipeline of video and image processing framework
## Client side (JETSON Nano) setup
- It captures images, re-size them, merge into single image, compress, and send through UDP
```
gst-launch-1.0 nvarguscamerasrc sensor-id=0 sensor-mode=4 \
        ! 'video/x-raw(memory:NVMM),width=1280,height=720,format=NV12,framerate=60/1' \
    ! nvvidconv flip-method=2 \
        ! 'video/x-raw(memory:NVMM),format=RGBA,width=640,height=360' \
    ! comp. nvarguscamerasrc sensor-id=1 sensor-mode=4 \
        ! 'video/x-raw(memory:NVMM),width=1280,height=720,format=NV12,framerate=60/1' \
    ! nvvidconv flip-method=2 \
        ! 'video/x-raw(memory:NVMM),format=RGBA,width=640,height=360' \
    ! nvcompositor name=comp sink_0::xpos=0 sink_0::ypos=0 sink_0::width=640 sink_0::height=360 sink_1::xpos=640 sink_1::ypos=0 sink_1::width=640 sink_1::height=360 \
    ! nvvidconv \
        ! 'video/x-raw(memory:NVMM),format=NV12,framerate=30/1' \
    ! nvjpegenc \
    ! rtpjpegpay \
    ! udpsink host=192.168.0.100 port=5010
```
## Server side (control station laptop) setup
- Receive video stream, decode, and display at window
```
gst-launch-1.0 udpsrc port=5010 ! application/x-rtp,encoding-name=JEPD ! rtpjpegdepay ! jpegdec ! autovideosink
```
## Example code
- Connect two cameras to JETSON
- Set up IP address for JETSON and client control station
- Assign execution permission to both server and client scripts with `chmod 755 FILENAME`
- Modify JETSON side code with control station IP address and port
```
 ! udpsink host=192.168.0.100 port=5010
```
- Control station needs port number only
- At controller terminal: `./gstream_udp_player_1.sh`
- At JETSON terminal: `./gst_stream_udp_0.sh`
## VLC for video display
- TCP setup did not work well (with RTP)
- UDP works but there is noticeable latency
- VLC buffer time should be set to 0 for minimum latency


[Next: RMRC 2022 server and client source code and set up](https://github.com/Cinderpe1t/TBS_Robotics_RMRC_2022_Source_Code_and_Setup) / [Top: Introduction](https://github.com/Cinderpe1t/TBS_Robotics_Introduction)
