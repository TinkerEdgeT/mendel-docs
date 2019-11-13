# Mendel release notes

This is a non-exhaustive summary of changes included in each version of Mendel.

To check your Mendel version, run `cat /etc/mendel_version`.

For system image downloads, see [coral.ai/software](https://coral.ai/software/).


## 4.0 (day)

+   First release based on [Debian 10 (buster)](
    https://www.debian.org/releases/buster/arm64/release-notes/ch-whats-new.en.html)
    (Note that our distribution does not include everything from Debian;
     for example, Mendel does not include desktop environments and applications,
     because our system is designed for embedded systems. Also, the
     new SecureBoot and AppArmor changes do not apply to Mendel)
+   Support for Python 3.7
+   Support for OpenCV and OpenCL
+   Support for device tree overlays
+   Upgraded GStreamer pipelines
+   Upgraded Linux Kernel (4.14)
+   Upgraded U-Boot bootloader (2017.03.3)


## 3.0 (chef)

+   First open source release
+   First release with the Mendel Development Tool (MDT)
+   Improved security for SSH authentication


## 2.0 (beaker)

+   Resolved Hynix/Micron memory timing stability
+   Support for USB-C connections in U-Boot
+   Support for Mac in flash.sh script
+   Released edgetpu_demo for out-of-box Edge TPU demo


## 1.0 (alpha)

Our first Mendel release!