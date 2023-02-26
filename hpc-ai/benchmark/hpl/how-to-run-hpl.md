# HPL benchmark setup how-to

The HPL (High Performance Linpack) benchmark is a tool to evaluate the floating point performance of a computer.
The details of HPL introduction, you can turn to Wikipedia or its homepage at https://netlib.org/benchmark/hpl/hpl/.
By today, HPL kept updated its latest version v2.3 (https://netlib.org/benchmark/hpl/hpl-2.3.tar.gz). For this benchmark
 setup, we will use this version with its soruce code.

To run HPL, we need 2 foundamental libraries of OpenBLAS and OpenMPI, so prior to the benchmark, you need to get them ready.
OpenBLAS is standard BLAS library which can run on all X86 platforms, for the better performance, you can run "Intel oneMKL"
on a system with an Intel CPU.
Other than OpenBLAS and OpenMI, some other system utils or libraries are needed. For example, in my Ubuntu (22.04) host.

## Prepare the environment
The following packages are required to install:
```
sudo apt install build-essential hwloc libhwloc-dev libevent-dev gfortran
```

## Compile HPL binary from source code

In my test, HPL v2.3 is used by building from it source codes.
Note: for some reason, the benchmark needs to be compiled in the user directory "~/hpl", hence the command mv hpl-2.3 ~/hpl.

```
wget https://netlib.org/benchmark/hpl/hpl-2.3.tar.gz
tar zxvf hpl-2.3.tar.gz
mv ./hpl-2.3 ~/hpl
cd ~/hpl
cd ./setup
sh make_generic
cp Make.UNKNOWN ../Make.linux
cd ..
vi Make.linux
```

In Makefile Make.linux, the following lines need to be modified to tell the compiler where our libraries are located.
The name of the current architecture (same as in the filename: Make.linux)

```
ARCH         = linux
```

The location of the OpenMPI library, given OpenMPI was installed in "$(HOME)/opt/openMPI":
```
MPdir        = $(HOME)/opt/OpenMPI
MPinc        = -I$(MPdir)/include
MPlib        = $(MPdir)/lib/libmpi.so
```
The location of the OpenBLAS was installed in "$(HOME)/opt/openBLAS":
```
LAdir        = $(HOME)/opt/OpenBLAS
LAinc        =
LAlib        = $(LAdir)/lib/libopenblas.a
```

Now to compile, we run the following command:
```
make arch=linux
```
If things go successfully, the resulting binary will be located at: bin/linux/xhpl.

## Run HPL benchmark

### Configuration

First, we need to go to the directory containing the executable of the benchmark.
```
cd bin/linux
vi HPL.dat
```

Here is an example HPL.dat file for a computer with a single CPU:
```
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any) 
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
29232        Ns
1            # of NBs
232          NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
1            Ps
1            Qs
16.0         threshold
1            # of panel fact
2            PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
4            NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
1            # of recursive panel fact.
1            RFACTs (0=left, 1=Crout, 2=Right)
1            # of broadcast
1            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
1            DEPTHs (>=0)
2            SWAP (0=bin-exch,1=long,2=mix)
64           swapping threshold
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form
1            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
##### This line (no. 32) is ignored (it serves as a separator). ######
0                               Number of additional problem sizes for PTRANS
1200 10000 30000                values of N
0                               number of additional blocking sizes for PTRANS
40 9 8 13 13 20 16 32 64        values of NB
```

### Running on a single CPU system

On a single CPU system, in the HPL.dat file, set the number of problems to: P=1 and Q=1.
```
./xhpl
```

Note that if the system has a single CPU socket but the CPU includes multiple NUMA nodes (like a SPR MCC), to get the best
results, the computation should be distributed on the NUMA nodes (see the next section). "Running on a dual CPU system" - TBD.

