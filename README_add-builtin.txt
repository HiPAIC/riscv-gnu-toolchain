To build:
 (dependences) sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
 ./configure --prefix=/home/$USER/riscv-gnu --with-arch=rv32i --with-abi=ilp32
 make -j 3
It's very important to configure --with-arch=rv32i because the newlib library is built under this arch. with-abi might be inferred by with-arch anyway but it doesn't hurt to specify explicitly.

To clean:
make clean
make distclean

Before changing anything:
git checkout -b "feature-branch-name-1" under riscv-gcc and git checkout -b "feature-branch-name-1" under riscv-binutils. These two are different repos, and we use same feature branch name to indicate we're working on the same thing.
But then run git checkout -b "feature-branch-name-2" under riscv-gdb. Use a different feature branch name!
NOTE: riscv-gdb and riscv-binutils are two different subdirs sharing the SAME git repo! But they are using different branches! This is because "riscv-gdb" has some changes specific to RISCV simulation (e.g. sim/riscv subdir) that is not yet merged into upstream riscv-binutils-gdb repo! 

Recommended workflow:
riscv-gcc$ git checkout -b "feature1"
Make changes in gcc

riscv-binutils$ git checkout -b "feature1"
Make changes in binutils
Push binutils to remote repo

riscv-gdb$ git checkout -b "feature1-gdb"
riscv-gdb$ git fetch
riscv-gdb$ git cherrypick <commit id from binutils changes>
This makes sure the common changes between binutils and gdb can be done just once and shared.
Then make gdb-specific changes (like simulation). Then push to remote repo (with feature1-gdb as branch name)


If you ever ran ./configure in subdirs (like riscv-gcc) by mistake, you will need to run "make distclean" in the subdir. Otherwise the top-level make will fail.

If you change gcc code, incremental make usually works.
But if you change binutils / gdb code, for some reason "make" cannot detect the change and will do nothing. You have to make clean and re-run make from scratch.

2021/5/8 update: added an intrinsic "__builtin_riscv_hipaic_mulsi(x,y)" to gcc that calls "mul" directly, even when the -march=rv32i does not support mul. See riscv-gcc/gcc/testsuite/hipaic/README.txt for testing.

2021/5/9 update: added 3 secret multiplication instructions: hp.grnd, hp.lops, hp.mul (GenNewRand, LoadOpX, Multiply). Simulation works. New test cases works fine. See riscv-gcc/gcc/testsuite/hipaic/README.txt for testing.
Updated README_add-builtin.txt with branch issue: riscv-binutils and riscv-gdb are using the same repo but different branches! To test out the new secret instructions, you must do this:
riscv-gcc$ git fetch origin
riscv-gcc$ git checkout --track origin/add-builtin
riscv-binutils$ git fetch origin
riscv-binutils$ git checkout --track origin/add-builtin
riscv-gdb$ git fetch origin
riscv-gdb$ git checkout --track origin/add-builtin-fsf-gdb-10.1-with-sim
Then do the usual make clean and ./configure and make -j 3

If you saw errors like fatal: A branch named 'add-builtin' already exists. when doing git checkout, that's probably because you already had a local branch tracking the remote branch. In that case, just git checkout <local branch name> and run git pull. It will automatically fetch the remote branch and merge it to your local branch.

