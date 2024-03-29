# CursedLFS
Instructions for building LFS on top of WSL2

# 1. Introduction
The basic outline is to follow the LFS book, specifically the Systemd version, but build the initial LFS cross-chain and temporary tools on an already existing WSL distro, within a separate virtual disk (VHD). We will then create a tarball of the root of that VHD and import it as a custom WSL distro. Notably, we won’t need a bootloader, and the instructions for building the kernel will differ. The kernel will require patches from Microsoft, and it will likely be a different version from the LFS book. Although the kernel can be built from source using our LFS tools, the resulting kernel is shared across all WSL distros.

# 2. Preparing the host system
## Host System Requirements
- Install WSL2 and Base distro (Tested with Ubuntu 22.04.3 LTS from MS Store)
- Install extra required packages:
```
sudo apt-get install build-essential bison texinfo
```

- Reconfigure /bin/sh to point to bash instead of the default dash:
```
sudo ln -s bash /bin/sh.bash
sudo mv /bin/sh.bash /bin/sh
```
- version-check.sh should now report all is OK
##  Creating a New Partition
 **In Windows**, run ``diskpart`` and use it to create a new VHD with a single partition for the cross-compile tools. We will not need a swap partion.
 ```
 create vdisk file="C:\Users\<user>\WSL2\base_lfs.vhd" maximum=50000 type=expandable
 ```
**In Windows**, mount the new VHD into WSL.
```
wsl --mount --vhd C:\Users\<user>\WSL2\base_lfs.vhd
```
Since there is not a filesystem yet, WSL will complain "The disk was attached but failed to mount: Operation not permitted.". This is expected. Check where the disk has been attached with `lsblk`. In my case, it was /dev/sde. Back inside the WSL distro, proceed as per "Creating a File System on the Partition", onwards.
```
mkfs -v -t ext4 /dev/<xxx>
```
...etc
# 3. Packages and Patches.
Proceed as per the book. On Ubuntu, remember to prefix the commands with sudo as required. You can remove GRUB and the kernel package from the wget list as they are unnecassary.  

## 5. Compiling the Cross-Toolchain

Proceed following the LFS book until you reach the Linux API headers. At this point, we'll diverge slightly because we'll eventually be building and using a WSL2 kernel from Microsoft's WSL2-Linux-Kernel releases.

1. **Fetch API Headers**:
   - Install the API headers from the `linux-msft-wsl` tarball matching the latest version.
   - These headers are essential for compatibility with the WSL2 environment.

2. **Kernel Version Considerations**:
   - As of the time of writing, there are only a few specific kernel releases available, with the newest being **6.1.21.2** from May 2023.
   - While we might attempt to port the Microsoft changes to the LFS book kernel in the future, for now, focus on using the API headers from the tarball.


# Chapters 6,7,8
Proceed exactly as per the book (skipping instructions for GRUB), taking care to appened "sudo" as required where instructions are to run command as root, up until chrooted in to the new LFS install.

# Chapter 9.
We will not use systemd-networkd for network configuration, so use the "note" commands, and ignore the remainder of "General Network Configuration" until "configuring the system hostname". I prefer to deal with locale issues with localectl after bootup, so I ignore the remainder of the instructions except creating /etc/profile, /etc/inputrc
and /etc/shells

# Chapter 10.
Ignore the book instructions. We will initially boot our WSL distro using the built-in MS-provided kernel, before building our own version of that kernel from source. Exit the chroot environment.

1. If you haven't already, remember to set a root passwd inside the chroot environment with ```passwd```

2 .Create a file at /etc/wsl.conf with the following contents:
```
[boot]
systemd=true
```

3.  Create a tarball of the $LFS rootfs. 
```
sudo tar cpjf ~/rootfs.tar.gz -C /mnt/lfs .
```

4. Next, from WINDOWS, import the rootfs as a custom WSL distro
```
wsl --import LFS C:\Users\<user>\AppData\Local\WSL\LFS "E:\rootfs.tar.gz"
```

5. LFS can now be booted (using the default WSL kernel) by issuing the command
```
wsl -d LFS
```

# Next steps.
I follow the instructions from the BLFS book for creating the Bash start up scripts, create a new user, and install wget, make-ca, and sudo from the BLFS book. You can set the default user to your new user by appending the following to /etc/wsl.conf
```
[user]
default=username
```
At this point, everything running inside your LFS WSL2 distro has been compiled from source with the exception of the kernel image. Note that the kernel image is shared by all WSL2 distributions. If you want to compile your own kernel image to use across all installed WSL2 distros, the instructions can be found at KERNEL.md.


