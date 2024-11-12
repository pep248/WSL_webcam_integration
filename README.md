# WSL - Webcam Integration

The following repository contains basic instructions on how to allow WSL to detect and use Webcams.

First of all we will need [usbipd-win](https://github.com/dorssel/usbipd-win)

It can be easilly installed by oppening a PowerShell terminal and typing the following command:

```
winget install usbipd
```

It's also recommended to have [wsl-usb-manager](https://github.com/nickbeth/wsl-usb-manager), so you can connect devices to the WSL without having to struggle with powershell commands.


Then we will list all the devices in our system:
```
usbipd wsl list
```
Look for the deveice you want to share from the list and check its port.

Then we will proceed to attach it to the WSL session, using:
```
usbipd wsl attach --busid <desired_bus_id> --distribution <name_of_your_WLS_distribution>
# An example of <desired_bus_id> could be 4-2
# An example of <name_of_your_WLS_distribution> could be Ubuntu-22.04
```

If, for reasons, at any given moment, we desire to detach said device, we can easily do so by using:
```
usbipd wsl detach --busid <desired_bus_id>
```

Reached this point, the camera should be atached to the WSL but there are high chances that it might not be working because Linux WSL, by default, doesn't include webcam drivers.

First step is to download Linux Kernel Drivers.
To do so we will first check what Kernel are we using by opening our WLS window, an typing the following command:
```
uname -a
```

You can now look for said kernel has an existing repo in the official Windows repository:
WSL2-Linux-Kernel[https://github.com/microsoft/WSL2-Linux-Kernel/releases]

And we will follow by assigning the version of the kernel to a varialbe
```
VERSION=<kernel_version_number>
# example
# VERSION=5.15.90.1
```


We will install some dependencies before updating the kernel:
```
sudo apt update && sudo apt upgrade -y && \
sudo apt install -y  \
        autoconf \
        bc \
        bison \
        build-essential \
        cmake \
        dwarves \
        flex \
        git \
        iputils-ping \
        libelf-dev \
        libgtk2.0-dev \
        libncurses-dev \
        libssl-dev \
        libtool \
        libudev-dev \
        net-tools \
        python3-pip \
        unzip \
        v4l-utils \
        zip
```

Now we create a direcotry to store all the files that we will download and install (if it doesn't exist already)
```
sudo mkdir /usr/src
cd /usr/src
```

We will now clone the repo and install it
```
sudo git clone -b linux-msft-wsl-${VERSION} https://github.com/microsoft/WSL2-Linux-Kernel.git ${VERSION}-microsoft-standard && cd ${VERSION}-microsoft-standard && \
sudo cp /proc/config.gz config.gz && \
sudo gunzip config.gz && \
sudo mv config .config && \
sudo make menuconfig
```

A new window will open with the kernel configuration:
1) Using the "up" and "down" arrows, go down to "Device Drivers" and go inside it by pressing "enter".
2) For our case, we will be configuring a WebCam, so we will now go down and check "Multimedia Support" (To check it, press the "spacebar" twice untill you see "<*>  Multimedia Support")
3) Now go inside "Multimedia Support" by using the "up" and "down" arrows, pressing "enter".
4) Check "Filter Media drivers" (To check it, press the "spacebar" twice untill you see "\[*] Filter Media drivers")
5) Now go inside "Media device types" by using the "up" and "down" arrows, pressing "enter".
6) Check "Camera and video grabbers" (To check it, press the "spacebar" twice untill you see "\[*] Camera and video grabbers")
7) Now go back by scrolling to "Exit" using the "right" and "left" arrows and hit "enter".
8) Now go inside "Video4Linux options" by using the "up" and "down" arrows, pressing "enter".
9) Check "V4L2 sub-device userspace API" (To check it, press the "spacebar" twice untill you see "\[*] V4L2 sub-device userspace API")
10) Now go back by scrolling to "Exit" using the "right" and "left" arrows and hit "enter".
11) Now go inside "Media Drivers" by using the "up" and "down" arrows, pressing "enter".
12) Check "Media USB Adapeters" (To check it, press the "spacebar" twice untill you see "\[*] Media USB Adapeters")
13) Now go inside "Media USB Adapeters" by using the "up" and "down" arrows, pressing "enter".
14) Check "USB Video Class (UVC)" (To check it, press the "spacebar" twice untill you see "<*> USB Video Class (UVC)")
15) Check "GSPCA based webcams" (To check it, press the "spacebar" twice untill you see "<*> GSPCA based webcams")
16) Finally, go back by scrolling to "Exit" using the "right" and "left" arrows and hit "enter" untill a rpompt appears asking us if we want to save. Select "yes".

And we proceed to build the new kernel and copy it to replace the old one
```
sudo make -j$(nproc) && \
sudo make modules_install -j$(nproc) && \
sudo make install -j$(nproc) && \
sudo mkdir /mnt/c/Sources/ && \
sudo cp -rf vmlinux /mnt/c/Sources/
```

Now we well reboot the WLS sesion by typing:
```
exit
```
or also by simply closing the window.
And we will go back to our PowerShell terminal and type:
```
wsl --shutdown
```

Now we will go to our user folder by pressing "Win+R" and typing
```
%userprofile%
```

We will create an empty text file and rename it to:
```
.wslconfig
```
in case the file didn't exist yet.
And we will paste the direction of the newly created kernel and save it:
```
[wsl2]
kernel=C:\\Sources\\vmlinux
```

Relaunch again the WSL terminal.

Go back to the PowerShell terminal and redo the steps to attach the device:
```
usbipd wsl list
usbipd wsl attach --busid <desired_bus_id> --distribution <name_of_your_WLS_distribution>
```

Go back to the WSL terminal and check if the device is propperly recognised by typing
```
lsusb
ls /dev/video*
```

And we are done and all set to go!
