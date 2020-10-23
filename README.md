### RISC-V Emulator supplementals

Code and binaries one needs to play around with the [RISC-V Emulator](https://github.com/AXKuhta/RISC-V_Emulation).

I'll just store the Linux binaries here for now. More to come.

### Linux Kernel

`vmlinux` is the kernel image. It was compiled from mainline source, no code changes whatsoever, version 5.8.15.

You can obtain the source code like that:

`git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git --branch=linux-5.8.y --depth=1`

It was compiled with `riscv64-linux-gnu-gcc` version 10.2.

Compilation procedure is as follows:

```
cd linux
export ARCH=riscv
export CROSS_COMPILE=riscv64-linux-gnu-
make nommu_virt_defconfig # Use the defconfig intended for QEMU
make menuconfig # go and disable compressed instructions and also alter the kernel cmdline parameters
make -j12
```

The file you are interested in afterwards, `vmlinux`, is the un-objcopy'd image. It should be sitting right in the current directory.

Drag and drop it onto the emulator to run it. You will likely need to alter the printk buffer address in the emulator if you are switching to a new image. You can determine it by setting a breakpoint onto the address of `<printk>:` (find it using `riscv64-linux-gnu-objdump -D vmlinux | grep "<printk>:"`) and observing where the memory writes go as it runs.

### Device tree

`riscvemu.dts` is the source code of the device tree. It needs to be compiled into a `.dtc` file and placed next to the emulator executable. A device tree is absolutely essential on RISC-V platforms. Without it the kernel would have no way of knowing how much RAM is it even allowed to use, what ranges it must ignore, and so on...

Compilation is as follows:

```
cd linux
scripts/dtc/dtc riscvemu.dts > riscvemu.dtc
```

### Glibc without compressed instructions

In standard configuration, userspace binaries produced by `riscv64-linux-gnu-gcc` will have compressed instructions in them because the libc it links against was compiled for rv64imafdc target. This will not do -- our emulator doesn't and _will not_ support the C extension. Thankfully, there is a way to rebuild glibc in-place:

```bash
# Download the same version that was bundled with your GCC!
# If not sure, check with `apt search libc riscv`
wget http://ftp.gnu.org/gnu/libc/glibc-2.31.tar.xz
tar -xf glibc-2.31.tar.xz
```

```bash
# Let's build it
cd glibc-2.31
mkdir build
cd build/
export "CROSS_COMPILE=riscv64-unknown-elf-"
export "CFLAGS=-march=rv64imafd -O2"
export "CXXFLAGS=-march=rv64imafd -O2"
export "ASFLAGS=-march=rv64imafd"
../configure riscv64-linux-gnu --target=riscv64-linux-gnu --build=i686-pc-linux-gnu --prefix=/usr/riscv64-linux-gnu/ --enable-add-ons
make -j12
```

```bash
# And install it -- this could potentially cripple your GCC
sudo make install install_root=/
```

Mine `riscv64-linux-gnu-gcc` wasn't crippled afterwards and was now producing binaries completely free of any compressed instructions! Sadly, it later turned out that all of this was moot because glibc is almost certainly a no-go on NOMMU. 
