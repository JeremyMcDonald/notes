####################
Linux Device Drivers
####################


*Abbreviations* 
- Industrial input/output(IIO)
- General-purpose input/output (GPIO)
- Interrupt request management (IRQ)
- Inter-Integrated Circuit (I2C)
- Serial Peripheral Interface (SPI)

*********************************************
Chapter 1: Introduction to Kernel Development 
*********************************************

Setting up the development enviroment
=====================================
* Target - is the machine that the binaries are being built for.
* Host - This is the place where the build is happening
* Compilation - This is also called **native compilation** or a **native build**. This happens when the compilation and the build target are the same.
* Cross-Compilation This is where the target and the host are different.

To check the system you are currently running on you can use the command:
::
   lsb_release -a
   # Example result
   Distributor ID: Ubuntu
   Description: Ubuntu 18.04.5 LTS
   Release: 18.04
   Codename: Bionic


Setting up the host machine
===========================
The packages we need for this development are
- gawk
- wget
- git
- diffstat
- unzip
- texinfo
- gcc-multilib
- build-essentials 
- chrpath
- socat
- libsdl1.2-dev
- xterm
- curses-dev
- lzop
- libelf-dev
- make

This installs some development tools and some mandatory libraries so that we have a nice user interface while developing
After this well install a compiler and some more tools(linker, assembler and so on).
This set of tools is called **Binutils** and **Binutils + the compiler** is called a **toolchain**.

Understanding and installing the toolchains
===========================================
Toolchain - gcc (compiler) & binutils

When you cross-compile you need to identify the right tool chain, for this a naming convention has been created.

``arch [-vendor] [-os] -abi``

The breakdown of this is:

* arch - identifies the **architecture**; i.e arm, mips, x86, i686 ... And so on.
* vendor - is the toolchain **supplier (company)**; i.e Bootlin, Linaro, none(if there is no provider) or you can omit this field.
* os - this is the **target operating system**; i.e linux or none(bare-metal). (If this is omitted then bare-metal is assumed)
* abi - this stands for **application binary interface**, it refers to what the underlying binary is going to look like.

The abi options in more detail:

* eabi - means the code will be compiled to run on a bare metal ARM core
* gnueabi - means the code will be compiled for Linux
* gnueabihf - is the same as above but the hf means hardfloat and indicates that the compiler and libraries are using floating-point hardware instructions.

Some example toolchain names to illustrate the pattern:

* **arm-none-eabi** : This uses ARM architecture has no vendor and targets a bare-metal system
* **arm-non-linux-gnueabi** or **arm-linux-gnueabi** : This produces  objects for the ARM architecture to be run on Linux and configured with the ABI (default) configuration. 

Installing the 32-bit toolchain for ARM architecture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Required installs are:
- gcc-arm-linux-gnueabihf
- binutils-arm-linux-gnueabihf

Installing the 64-bit tool chain for ARM architecture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Required installs are called aarch64, and this is whats required:
- make
- gcc-aarch64-linux-gnu
- binutils-aarch64-linux-gnu

note:: *aarch64 only supports/provides hardware for float 64 toolchains. Thus there is no need to specify 'hf' at the end.*

Getting the sources 
===================
Back in the day the Linux Kernel used X.Y.Z semantic versionin then it switch to X.Y versioning, now it uses non-sequential X.Y versioning with 
one Long Term Support (LTS) and one stable release:

.. code-block::
      
      git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git --depth 1

**note:** *The --depth 1 means we only get the most recent commits and not ALL the commits*

Looking inside the directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **arch/** -  This contains the processor-specific code, it has sub-directories per architecture alpha/ arm/ mips/ arm64/ etc..
* **block/** - This contains the code for block storage devices
* **crypto/** -  This containts the cryptographic API and encryption stuff
* **certs/** - This contains certs to sign files for the kernel
* **documentation/** - This contains descriptions of the APIs that are used for different Kernel frameworks. If you have questions look here first.
* **drivers/** - contains the drivers.
* **fs/** - This directory contains different implementations of file systems. FAT32, NTFS, ETX{2,3,4} sysfs, procfs,NFS etc..
* **include/** - This directory contains the kernel header files.
* **init/** - This contains initialization and startup code.
* **ipc/** - This is the inter-process-communication mechanisms, like message queues, semaphores and shared memory.
* **kernel/** - This contains architecture-independent portions of the base kernel.
* **lib/** - Contains help[er functions like kernel object(kobject) and cyclic redundancy code(CRC).
* **mm/** - Memory managment code.
* **net/** - Networking protocol code.
* **samples/** - This icontains the device driver samples for various subsystems
* **scripts/** - This contains scripts and tools.
* **security/** - security framework code.
* **sound/** - Code for sound.
* **tools/** - Linux Kernel development tooals and tools for testing subsystems. GPIO, IIO, SPI, etc.
* **usr/** - Intramfs implementation
* **virt/** - this has the kernel virtual machine (KVM) module for hypervisor

Any arichtecture specific code should go in the **arch/** directory.

Configuring and building the Linux kernel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The kernel uses the compiler specified in the Makefile:

.. code-block::

      $(CROSS_COMPILE)gcc

CROSS_COMPILE is the prefix for cross-compiling tools (gccm, as, ld, objcopy, etc..) and must be specified when invoking **make**. 

Only gcc and its related Binutils executables will be prefixed with $CROSS_COMPILE).

We will us the ARCH enviroment to configure and build linux:

.. code-block::

      ARCH=<XXXX> CROSS_COMPILE=<YYYY> make menuconfig

**Or**
.. code-block::

      ARCH=<XXXX> CROSS_COMPILE=<YYYY> make <make_target>

If we are running commands often we can export the enviroment variable to the current shell like this:

.. code-block::
   
      export CROSS_COMPILE=aarch64-linux-gnu-
      export ARCH=aarch64

*Remember* If theses are **NOT** specified the native host machine is going to be the build target instead of the machine you want targeted.


Understanding the kernel configuration process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We can use an interface if we make the kernel with the command:
::
    make menuconfig
    or
    make xconfig

The options for configuration are as follows

* Boolean
   * (blank) this ommits the feature.
   * (*) Compiles statically to the kernel

* Tristate options (Like boolean but with 3 options)
   * (blank) as above
   * (*) as above
   * (m) Produces a loadable module and can be selected by pressing M

* String
   * Expect string values eg.. CONFIG_CMDLINE="noinitrd console=ttymxc0,115200"

* Hex
   * Requires a hexidecimal value eg.. CONFIG_PAGE_OFFSET=0x80000000

* Int
   * Requires integer options DEFAULT=1

These options will be stored in a .config file at the root of the source tree.

Often it is not nesseccary to start from scratch, you can find configurations for many platforms in:

.. code-block::
   
      ls arch/<your architecture>/configs/

Using 

.. code-block::

      make <foo_defconfig>

Will generate a new .config file and rename the old one to .config.old.

To customize the current .config we can use

.. code-block::

      make menuconfig

To create a standard configuration with a minimal format to use a default starting configuration we can use the command.

.. code-block::

   make savedefconfig

Good practice is to move the file after creation to the arch directory of the architecture you are using so other developers on your
team can user the same default config you are using

.. code-block::

      mv defconfig arch/<arch>/configs/myoown_defconfig

The following are examples of commands we can use:

* For native compilation a lot of options can be omitted
.. code-block::
  
      make x86_64_defconfig
      make menuconfig

* For a cross compiled 32-bit ARM i.MX6-based board would look like this
.. code-block::

     ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make imx_v6_v7_defconfig  
     ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make menuconfig

* An example of a 64-bit system configuration would look like this

     .. code-block::

      ARCH=aarch64 CROSS_COMPILE=aarch64-linux-gnu- make defconfig
      ARCH=aarch64 CROSS_COMPILE=aarch64-linux-gnu- make menuconfig

*note* if you run into an error using xconfig its probably due to Qt4, you can fix it by installing the missing packages:
   
.. code-block::
   
      sudo apt install qt4-dev-tools qt4-qmake

Configs can be available on the running system in /boot/config and can be copied using
.. code-block::

      cp /boot/config 'uname -r' .config

A better option is to enable **IKCONFIG** and **IKCONFIG_PROC** in the kernel config options this will allow access to .config through /proc/configs.gz
This is considered a standard method and works with embedded systems.

Some useful configuration files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* IKCONFIG and IKCONFIG_RPOC: These let you see the state of the kernel configuration at run time. eg:

.. code-block::

      # zcat /proc/config.gz | grep CONFIG_SOUND
      CONFIG_SOUND=y
      CONFIG_SOUND_OS_CORE=y
      CONFIG_SOUND_OSS_CORE_PRECLAIM=y

      # CONFIG_SOUNDWIRE is not set
      #

* CMDLINE_EXTEND and CMDLINE: The first option is a Boolean that allows you to extend the kernel command-line from within the configuration and the second option is the actual extended command-line extension value.

* CONFIG_KALLSYMS: This makes the kernel symbol table available in /proc/kallsyms. This is useful for tracers and other tools to map the kernel symbols to addresses. Without this oops messages will be printed in hexadecimal. 

* CONFIG_PRINTK_TIME: This prints timing while printing info from the kernel. 

* CONFIG_INPUT_EVBUG: This lets you debug input devices.

* CONFIG_MAGIC_SYSRQ: This lets you have some control over the system even after a crash.

* DEBUG_FS: This enables support for debug files where GPIO, CLOCK, DMA, REGMAP, IRQs and other subsystems to be debugged from. 

* FTRACE and DYNAMIC_FTRACE: These options enable *ftrace* tracer. Once it is enabled it has more options that can be enabled
   * FUNCTION_TRACER: Traces a non-inline function in the kernel
   * IRQSOFF_TRACER: This lets you track off periods of IRQs in the kernel
   * PREEEMPT_TRACER: Allows you to measure preemption off latency.
   * SCHED_TRACER: This allows you to schedule latency tracing.

Building the Linux Kernel
^^^^^^^^^^^^^^^^^^^^^^^^^

You should do this in the same shell that you did the cross-compile in if you set the ARCH and CROSS-COMPILE enviroment variables.


by defualt the make target is **all**. So for x86 architectures the target points to **vmlinux bzImage modules**.
For ARM or aarch64 it targets **vmlinux zImage modules** and **dtbs**.

* bzImage is a x86 specific target thaty produces a binary also named bzImage
* vmlinux produces an image called vmlinux
* zImage and dtb are both ARM aarch64 specific. zImage produces a zImage image and dtb produces a device tree sources for the target CPU variant.
* modules is a target that will produce all the selected modules (maked with m configuration)

We can speed up our build by usnig more cores on the host system. -j is a make option that lets us do this. e.g:

.. code-block::

         make -j16

Most people use a number that is 1.5x the number of cores the host system has. But you can use as high as 2.

Example build commands for a host system that has an 8 core CPU:
**Native compilation**

.. code-block::
         
         make -j16

**32-bit ARM cross compilation**

.. code-block::
      
      ARCH=arm CROSS_COMPILATION=arm-linux-gnueabi- make -j16

If you don't want to do all the commands all at once you can also invoke them individually:


.. code-block::
      
      ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make dtbs

**or**


.. code-block::
      
      ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make zImage -j16

Or if you have set the variables in the shell you can even shorten it to this:

.. code_block::

      make bzImage -j16

After the project is built we will get a few files as a result:

a compressed and a uncompressed image that can be booted from can be found in:

``arch/<arch>/boot/Image``

In arch/<arch>/boot/dts/*.dtb contains the device tree blobs and vmlinux which is a raw unstripped uncompress kernel image in ELF format that is useful for debugging but not much else.



