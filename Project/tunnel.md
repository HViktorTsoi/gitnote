mapping时间同步部署

1. 将jsync, point_grey_camera_driver_sync, yesense这几个改过时间同步的驱动复制到目标机器上
2. 将目标机器的usb fs buffer size设置成1000MB, 按照https://www.flir.com/support-center/iis/machine-vision/application-note/understanding-usbfs-on-linux/的方法
3. echo 1000 > /sys/module/usbcore/parameters/usbfs_memory_mb