To build:
 (dependences) sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
 ./configure --prefix=/home/tim/riscv --with-arch=rv32i --with-abi=ilp32
 make -j 3
It's very important to configure --with-arch=rv32i because the newlib library is built under this arch. with-abi might be inferred by with-arch anyway but it doesn't hurt to specify explicitly.

To clean:
make clean

Before changing anything:
git checkout -b "feature-branch-name" under riscv-gcc and riscv-binutils. This helps matching their changes.
NOTE: riscv-gdb and riscv-binutils are two different subdirs sharing the SAME git repo! That means you should avoid changing both subdirs. It's better to always make changes in riscv-binutils, push changes to repo, and then pull the change from riscv-gdb.

If you ever ran ./configure in subdirs (like riscv-gcc) by mistake, you will need to run "make distclean" in the subdir. Otherwise the top-level make will fail.

If you change gcc code, incremental make usually works.
But if you change binutils / gdb code, for some reason "make" cannot detect the change and will do nothing. You have to make clean and re-run make from scratch.

2021/5/8 update: added an intrinsic "__builtin_riscv_hipaic_mulsi(x,y)" to gcc that calls "mul" directly, even when the -march=rv32i does not support mul. See riscv-gcc/gcc/testsuite/hipaic/README.txt for testing. All subdirs (riscv-gcc, riscv-binutils, riscv-gdb) must be latest in add-builtin branch.
