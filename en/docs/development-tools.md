# Development Tools

## GNU Compiler Collection (GCC)

GNU Compiler Collection (GCC) is available on the ABCI System.

List of compile/link command of GCC:

| Parallelism | Programming Language | command |
|:--|:--|:--|
| Serial | Fortran | gfortran |
| | C | gcc |
| | C++ | g++ |
| MPI parallel | Fortran | mpifort |
| | C | mpicc |
| | C++ | mpic++ |

## Intel Parallel Studio XE

Intel Parallel Studio XE is available on the ABCI System.
To use Intel Parallel Studio XE, set up user environment by the `module` command.
If you set up with the `module` command in compute node, environment variables for compilation and execution are set automatically.

Setting command for Intel Parallel Studio XE is following:

```
[username@g0001 ~]$ module load intel/2020.4.304
```

List of compile/link commands of Intel Parallel Studio XE:

| Programing Language | command |
|:--|:--|
| Fortran | ifort |
| C | icc |
| C++ | icpc |

## PGI

PGI Compiler is available on the ABCI System.
To use PGI compiler, set up user environment by the `module` command.
If you set up with the `module` command in compute node, environment variables for compilation and execution are set automatically.

Setting command for PGI Compiler is following:

```
[username@g0001 ~]$ module load pgi/20.4
```

List of compile/link commands of PGI Compiler:

| Programing Language | command |
|:--|:--|
| Fortran | pgf90 |
| C | pgcc |
| C++ | pgc++ |

## NVIDIA HPC SDK

NVIDIA HPC SDK is available on the ABCI System.
To use NVIDIA HPC SDK, set up user environment by the `module` command.
If you set up with the `module` command in compute node, environment variables for compilation and execution are set automatically.

Setting command for NVIDIA HPC SDK is following:

```
[username@g0001 ~]$ module load nvhpc/21.2
```

List of compile/link commands of NVIDIA HPC SDK:

| Programing Language | command |
|:--|:--|
| Fortran | nvfortran |
| C | nvc |
| C++ | nvc++ |

## OpenMP

The compilers provided on the ABCI System support thread parallelization by OpenMP specifications.
To activate the OpenMP specifications, specify the compile option as follows:

| | Compile option |
|:--|:--|
| GCC | -fopenmp |
| Intel Parallel Studio | -qopenmp |
| PGI | -mp |
| NVIDIA HPC SDK | -mp |

## CUDA

CUDA is available on the ABCI System.
To use CUDA compiler, set up user environment by the `module` command.
If you set up with the `module` command in compute node, environment variables for compilation and execution are set automatically.

| Programing Language | command |
|:--|:--|
| C++ | nvcc |
