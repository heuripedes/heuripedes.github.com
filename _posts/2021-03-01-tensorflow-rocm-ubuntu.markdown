---
layout: post
title: "Setting up ROCm 4.0 and tensorflow-rocm on Ubuntu 20.04"
lang: english
---

ROCm 4.0 and Ubuntu Desktop 20.04 don't play well together out of the box so [while ROCm 4.1 isn't available](https://github.com/RadeonOpenCompute/ROCm/issues/1307#issuecomment-787671933), we'll have to make do with what we have. I hope you enjoy this guide.

## 1. The right kernel
The first step in the process is to check your kernel version as ROCm doesn't yet support kernels newer than 5.4. Ubuntu Server installs are usually fine in this department as they ship with the GA (General Availability) stack, but Ubuntu Desktop from 20.04 onwards will ship with the [Hardware Enablement Stack](https://wiki.ubuntu.com/Kernel/LTSEnablementStack#Ubuntu_20.04_LTS_-_Focal_Fossa) unless you're using one of those HP/Dell/etc prebuilts, in which case Ubuntu will try to use the OEM flavour.

Check your kernel version with

    $ uname -a

If the version reported starts with 5.4 then you can jump to step #2. Otherwise, run the following commands to downgrade to the general availability kernel but first read this note from Ubuntu wiki:

> It is advised to keep Ubuntu Desktop 20.04 LTS with the kernel flavour picked during installation. It can be either HWE or OEM flavour. Changing to track GA [General Availability] kernel may result in regressions of performance, hardware support, and certified features.

    sudo apt install --install-recommends linux-generic

Reboot, interrupt grub, go into `Advanced options` and boot the 5.4 kernel. Check that everything works as expected.

If everything is fine, get rid of the HWE/OEM kernel:

    sudo apt remove --purge linux-generic-hwe-20.04 linux-oem-20.04 linux-hwe-* linux-oem-* linux-modules-5.1* linux-modules-5.8.0-* linux-modules-5.6.0-* 

## 2. ROCm Setup

ROCm is only available from AMD's ROCm repository so you'll have to add it's PGP keys and make the repository url visible to `apt`. 

First download and verify the key:

    wget -q -O /tmp/rocm.gpg.key https://repo.radeon.com/rocm/rocm.gpg.key
    echo 'e85a40d1a43453fe37d63aa6899bc96e08f2817a /tmp/rocm.gpg.key' | sha1sum -c

If the check failed, please consult the [official installation guide](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html) or comment on the post (no promises). If everything went right, proceed with these commands to configure `apt` and install the ROCm driver and libraries:

    sudo apt-key add /tmp/rocm.gpg.key
    echo 'deb [arch=amd64] https://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
    sudo apt update
    sudo apt install rocm-dkms rocm-libs

If the install succeeded, add the rocm executable paths to your `$PATH` and add yourself to the `video` and `render` groups:

    echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/rocprofiler/bin:/opt/rocm/opencl/bin' | sudo tee -a /etc/profile.d/rocm.sh
    sudo gpasswd -a $LOGNAME render
    sudo gpasswd -a $LOGNAME video

Reboot your system.

After the reboot, check if ROCm is working by runnning `rocm-smi` and `clinfo`. Your GPU should be listed in them. If you can't see your GPU, consult the [official installation guide](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html) for more information.

## 3. Tensorflow

Tensorflow is available under two packages, `tensorflow-rocm` and `tensorflow-rocm-enhanced`. From a programming standpoint they are identical as far as I can tell so pick whatever you want:

    # preferrably from a virtualenv
    pip install tensorflow-rocm-enhanced

The following command will show the devices recognized by tensorflow:

    python -c 'from tensorflow.python.client import device_lib; print(device_lib.list_local_devices())'
