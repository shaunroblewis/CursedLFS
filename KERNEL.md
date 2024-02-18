# Kernel instructions

1. Install git and cmake as per BLFS instructions.

2. Reinstall elf-utils as per LFS book, but ignore the instructions to only build libelf. Execute "make install" in place of "make -C libelf install"
  
3. Install pahole. This is not part of the BLFS book. Latest version currently at time of writing is at https://git.kernel.org/pub/scm/devel/pahole/pahole.git/tag/?=v1.25
```
mkdir build &&
cd build &&
cmake -D__LIB=lib -DCMAKE_INSTALL_PREFIX=/usr .. &&
make
```
As root:
```
make install
```

4. Download and extract the latest WSL2 Kernel source release from https://github.com/microsoft/WSL2-Linux-Kernel. At time of writing, this is linux-msft-wsl-6.1.21.2.
   Build it with the preconfigured .config, set up for WSL, with:
   ```
   make KCONFIG_CONFIG=Microsoft/config-wsl
   sudo make modules_install headers_install
   ```

5. Copy the resulting kernel image to the windows filesystem.
```
cp arch/x86/boot/bzImage /mnt/c/Users/<user>/WSL2/vmlinux
```
6. Edit %USERPROFILE%\.wslconfig to point to the newly built kernel:
```
[wsl2]
# Set the path to the custom kernel
kernel=C:\\Users\\shaun\\WSL2\\vmlinux
```
