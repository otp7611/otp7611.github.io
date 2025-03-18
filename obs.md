obs

# 使用虚拟摄像头

```shell
sudo apt install v4l2loopback-utils
v4l2-ctl --list-devices
sudo modprobe v4l2loopback
sudo modprobe -r v4l2loopback
```

Clicking on "Start Virtual Camera" 会出现/dev/video0这个就是虚拟摄像头了。

