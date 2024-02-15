# CursedLFS
Instructions for building LFS on top of WSL2

# 1. Introduction
The basic outline is to follow the LFS book, but build the inital LFS cross-chain and temp tools on a already existing WSL distro, but inside a seperate virtual disk (VHD). We will then create a tarball of the root of that VHD and import it as a custom WSL distro before installing the rest of the system (almost) as per the book. We won't need a bootloader, and the instructions for building the Kernel will be different. It will requires patches from MS, and will likley  be a diffrenet version from the LFS book. The kernel CAN be built from source using our LFS tools, but the resulting kernel is then shared across all WSL distros. 

# 2. Preparing the host system
## Host System Requirements
- Install WSL2 and Base distro (Tested with Ubuntu 22.04.3 LTS from MS Store)
- Install extra required packages:
```
sudo apt-get install build-essentials bison texinfo
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
wsl --mount --vhd C:\Users\shaun\WSL2\base_lfs.vhd
```
Since there is not a filesystem yet, WSL will complain "The disk was attached but failed to mount: Operation not permitted.". This is expected. Check where the disk has been attached with `lsblk`. In my case, it was /dev/sde. Back inside the WSL distro, proceed as per "Creating a File System on the Partition", onwards.
```
mkfs -v -t ext4 /dev/<xxx>
```
...etc
# 3. Packages and Patches.
Proceed as per the book. On Ubuntu, remember to prefix the commands with sudo as required. You can remove GRUB and the kernel package from the wget list as they are unnessacary.  

