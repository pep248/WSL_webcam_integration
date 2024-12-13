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

A new window will open with the kernel configuration. Using the "arrow keys", and "enter" to navigate, and the "space" to select and unselect submodules, configure the following options:
```txt
Device Drivers --->
        <*>  Multimedia Support --->
                [*] Filter Media drivers
                Media device types --->
                        [*] Camera and video grabbers
                Video4Linux options --->
                        [*] V4L2 sub-device userspace API
                Media Drivers --->
                        [*] Media USB Adapeters --->
                                <*> USB Video Class (UVC)
                                <*> GSPCA based webcams
                                
                        
                
```        

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
