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
