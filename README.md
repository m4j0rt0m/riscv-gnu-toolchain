RISC-V GNU Compiler Toolchain
=============================

---

This is a GCC 9.4 snapshot supporting the draft [RVV 0.7.1](https://github.com/brucehoult/riscv-v-spec/tree/0.7.1) specification.

The support in this release is primarily in binutils, with gcc knowing how to pass the
correct flags through, thus allowing the `gcc` driver to assemble `.s` files using RVV 0.7.1,
or to use inline asm in C.

There is no support for C intrinsics or auto-vectorisation.

This draft version 0.7.1 of RVV has been implemented by THead in their C906 and C910/C920
cores and widely sold in SoCs including the AllWinner D1, Bouffalo BL808, Cvitek CV1800B,
Sophgo SG2002 and SG2042, THead TH1520.

Use this toolchain if you want to write RVV 0.7.1 assembly language or inline asm using
the original 0.7.1 mnemonics. If you wish, you can create an object file or library and link it with
C/C++ code compiled by a later version of GCC or Clang.

If you are writing asm by hand, there is no disadvantage to using this slightly old toolchain.

GCC 14 and later support the RVV 0.7.1 draft spec under the name _xtheadvector. Technically it is the
same, but you have to prefix instruction and CSR mnemonics with `th.`.  You can also generate
RVV 0.7.1 code from the same C instrinsics as for 1.0.

If you want to use C intrinsics for RVV 0.7.1 then use standard upstream GCC 14 or later, and add
`_xtheadvector` to your ISA specification instead of `v`.

Quickstart for a 64 bit newlib toolchain for building embedded RVV-enabled programs (or just `.o` files)
on a Debian/Ubuntu or similar system:

    sudo mkdir /opt/rvv071
    sudo chown `whoami` /opt/rvv071
    export PATH=/opt/rvv071/bin:$PATH
    sudo apt update
    sudo apt install -y autoconf automake autotools-dev bc bison \
      build-essential coreutils curl flex gawk gcc git gperf \
      libexpat-dev libgmp-dev libmpc-dev libmpfr-dev libtool \
      patchutils pkg-config python3 texinfo zlib1g-dev
    git clone https://github.com/brucehoult/riscv-gnu-toolchain
    cd riscv-gnu-toolchain
    git submodule update --init --recursive --depth 1 \
      riscv-newlib riscv-binutils riscv-gcc riscv-gdb
    ./configure --prefix=/opt/rvv071 --with-arch=rv64gcv
    make -j`nproc` newlib

Your riscv64-unknown-elf-gcc (and friends) is ready!

If you don't want/need sudo to run commands as root, or don't have sudo installed, you can
`alias sudo=''` before the above commands.

---

This is the RISC-V C and C++ cross-compiler. It supports two build modes:
a generic ELF/Newlib toolchain and a more sophisticated Linux-ELF/glibc
toolchain.

###  Getting the sources

This repository uses submodules. You need the --recursive option to fetch the submodules automatically

    $ git clone --recursive https://github.com/brucehoult/riscv-gnu-toolchain
    
Alternatively :

    $ git clone https://github.com/brucehoult/riscv-gnu-toolchain
    $ cd riscv-gnu-toolchain
    $ git submodule update --init --recursive
    
    

### Prerequisites

Several standard packages are needed to build the toolchain.  On Ubuntu,
executing the following command should suffice:

    $ sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev

On Fedora/CentOS/RHEL OS, executing the following command should suffice:

    $ sudo yum install autoconf automake python3 libmpc-devel mpfr-devel gmp-devel gawk  bison flex texinfo patchutils gcc gcc-c++ zlib-devel expat-devel

On OS X, you can use [Homebrew](http://brew.sh) to install the dependencies:

    $ brew install python3 gawk gnu-sed gmp mpfr libmpc isl zlib expat

To build the glibc (Linux) on OS X, you will need to build within a case-sensitive file
system.  The simplest approach is to create and mount a new disk image with
a case sensitive format.  Make sure that the mount point does not contain spaces. This is not necessary to build newlib or gcc itself on OS X.

This process will start by downloading about 200 MiB of upstream sources, then
will patch, build, and install the toolchain.  If a local cache of the
upstream sources exists in $(DISTDIR), it will be used; the default location
is /var/cache/distfiles.  Your computer will need about 8 GiB of disk space to
complete the process.

### Installation (Newlib)

To build the Newlib cross-compiler, pick an install path.  If you choose,
say, `/opt/riscv`, then add `/opt/riscv/bin` to your `PATH` now.  Then, simply
run the following command:

    ./configure --prefix=/opt/riscv
    make

You should now be able to use riscv64-unknown-elf-gcc and its cousins.

### Installation (Linux)

To build the Linux cross-compiler, pick an install path.  If you choose,
say, `/opt/riscv`, then add `/opt/riscv/bin` to your `PATH` now.  Then, simply
run the following command:

    ./configure --prefix=/opt/riscv
    make linux

The build defaults to targetting RV64GC (64-bit), even on a 32-bit build
environment.  To build the 32-bit RV32GC toolchain, use:

    ./configure --prefix=/opt/riscv --with-arch=rv32gc --with-abi=ilp32d
    make linux

Supported architectures are rv32i or rv64i plus standard extensions (a)tomics,
(m)ultiplication and division, (f)loat, (d)ouble, or (g)eneral for MAFD.

Supported ABIs are ilp32 (32-bit soft-float), ilp32d (32-bit hard-float),
ilp32f (32-bit with single-precision in registers and double in memory, niche
use only), lp64 lp64f lp64d (same but with 64-bit long and pointers).

### Installation (Linux multilib)

To build the Linux cross-compiler with support for both 32-bit and
64-bit, run the following commands:

    ./configure --prefix=/opt/riscv --enable-multilib
    make linux

The multilib compiler will have the prefix riscv64-unknown-linux-gnu-,
but will be able to target both 32-bit and 64-bit systems.

### Troubleshooting Build Problems

Builds work best if installing into an empty directory.  If you build a
hard-float toolchain and then try to build a soft-float toolchain with
the same --prefix directory, then the build scripts may get confused
and exit with a linker error complaining that hard float code can't be
linked with soft float code.  Removing the existing toolchain first, or
using a different prefix for the second build, avoids the problem.  It
is OK to build one newlib and one linux toolchain with the same prefix.
But you should avoid building two newlib or two linux toolchains with
the same prefix.

If building a linux toolchain on a MacOS system, or on a Windows system
using the Linux subsystem or cygwin, you must ensure that the filesystem
is case-sensitive.  A build on a case-insensitive filesystem will fail when
building glibc because \*.os and \*.oS files will clobber each other during
the build eventually resulting in confusing link errors.

Centos (and RHEL) provide old GNU tools versions that may be too old to build
a RISC-V toolchain.  There is an alternate toolset provided that includes
current versions of the GNU tools.  This is the devtoolset provided as part
of the Software Collection service.  For more info, see the
[devtoolset-7](https://www.softwarecollections.org/en/scls/rhscl/devtoolset-7/)
URL.  There are various versions of the devtoolset that are available, so you
can also try other versions of it, but we have at least one report that
devtoolset-7 works.

### Advanced Options

There are a number of additional options that may be passed to
configure.  See './configure --help' for more details.

### Test Suite

The DejaGnu test suite has been ported to RISC-V.  This can run with GDB
simulator for elf toolchain or Qemu for linux toolchain, and GDB simulator
doesn't support floating-point.
To test GCC, run the following commands:

    ./configure --prefix=$RISCV --disable-linux --with-arch=rv64ima # or --with-arch=rv32ima
    make newlib
    make report-newlib

    ./configure --prefix=$RISCV
    make linux
    make report-linux
