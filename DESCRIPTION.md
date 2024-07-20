# STREAM

## Purpose

The STREAM benchmark is a synthetic benchmark which measures the sustainable memory bandwidth of a compute node. It only uses little to no computation per byte transferred to or from memory. Different versions of this benchmark are available. The one used here is written in C and only uses OpenMP for threading (no MPI).

## Source

Archive name: `stream-bench.tar.gz`.

The source code is available in the file `src/stream_src.tar.gz`.

The provided sources are equivalent to the STREAM version 5.10 from the official website: https://www.cs.virginia.edu/stream/FTP/Code/

Download and unpack the tar-file first; the directory `stream_src` will be created.

```
cd src
tar -xvzf stream_src.tar.gz
```

## Building

OpenMP must be enabled for compiler and linker. The preprocessor macros `STREAM_ARRAY_SIZE` and `NTIMES` must be defined (see _Configurations_ below).

When using the JUBE script, JUBE will build the STREAM executable in the compile step (see _Execution with JUBE_ below). To build STREAM without JUBE, use a compiler invocation similar to the following line.

```
cd stream_src
make CC=gcc CFLAGS="-fopenmp -Wall -DSTREAM_ARRAY_SIZE=$((2 ** 26)) -DNTIMES=200" stream_c.exe
```

Since the array size is a compile-time parameter, it is recommended to build a dedicated executable for each `STREAM_ARRAY_SIZE`.

Depending on the tested configuration, the compiler flags might be modified, see _Modification_ section below.

## Execution

The executable is called `stream_c.exe`.

### Command Line

Stream can be executed manually for example with the following command:

```
OMP_PLACES="cores" OMP_PROC_BIND="spread" OMP_NUM_THREADS=128 ./stream_c.exe
```

### Execution with JUBE

Using JUBE the benchmark can be executed as follows:

```
jube run stream.jube.xml [--tag tags...]
jube continue stream_run # wait/repeat until all steps are done
jube result -a stream_run
```

|       Tags           |               Effect               | Default |
|----------------------|------------------------------------|---------|
| `varySize`           | `STREAM_ARRAY_SIZE=2^15..2^28`     |         |
| `varySizeextended`   | `STREAM_ARRAY_SIZE=2^10..2^35`     |         |
| `fixedSize`          | `STREAM_ARRAY_SIZE=2^28`           | yes     |
| `varyThreads`        | `OMP_NUM_THREADS=1..16`            | yes     |
| `threads1`           | `OMP_NUM_THREADS=1`                |         |
| `threads4`           | `OMP_NUM_THREADS=4`                |         |
| `threadsCores`       | `OMP_NUM_THREADS=#Cores`           |         |
| `threadsNuma`        | `OMP_NUM_THREADS=#NUMA Domains`    |         |
| `threadsHyper`       | `OMP_NUM_THREADS=#Hardware Threads`|         |
| `s22`                | `module load Stages/2022`          | yes     |
| `s21`                | `module load Stages/2021`          |         |
| `s20`                | `module load Stages/2020`          |         |
| `varyCompiler`       | `CC=gcc,icc,nvc,clang`             |         |
| `gcc`                | `CC=gcc`                           | yes     |
| `intel`              | `CC=icc`                           |         |
| `nvhpc`              | `CC=nvc`                           |         |
| `aocc`               | `CC=clang` (AOCC)                  |         |

The results can be found in the columns `Copy, Scale, Add, Triad`.

### Configurations

Candidates are requested to run the following configurations; according JUBE tags are given. All STREAM benchmarks should be run in FP64 precision. See also the modification overview below.

| Name                       | Description                                                                                                              | JUBE Tags                 | 
| ---------------------------| ------------------------------------------------------------------------------------------------------------------------ | --------------------------| 
| Threads1                   | Single threaded, array sizes 2^15 to 2^28 should be reported; array size 2^28 will be used for evaluation                | `threads1` `varySize`     |
| Threads4                   | Four threads located on the same socket, fixed size (2^28)                                                               | `threads4`                |
| ThreadsCores               | One thread per physical core on the node, fixed size (2^28)                                                              | `threadsCores`            |
| ThreadsHyper               | One thread per hardware thread on the node, fixed size (2^28)                                                            | `threadsHyper`            |
| Optimal                    | A custom configuration that achieves maximum bandwidth (2^28)                                                            | `optimal`^                |

The following STREAM parameters are to be used:

|         Name        |                     Value                     |
|---------------------|-----------------------------------------------|
| `STREAM_ARRAY_SIZE` | 2^28; 2^15 to 2^28 for _Threads1_ in addition |
| `NTIMES`            | 200                                           |

In any case, the timing accuracy output by the program needs to be at least 20 clocks-ticks (`CLK/Ins`).

If multiple memory domains are exposed to user space (for example HBM and DDR), each memory domain must be measured and reported separately. This can be achieved, for example, by using `numactl` and the `-m` (`--membind`) option. Mixing memory domains (for example _cache mode_) is not allowed.

^ Tag already exists but values have to be defined in `parametersets.jube.xml`.

### Modification

The following Modifications are allowed, depending on the configuration:

| Name         | Compiler | Compiler Flags | Size | Threads | Affinity | Source Code |
| ------------ | -------- | -------------- | ---- | ------- | -------- | ----------- |
| Threads1     | Yes      | Yes            | No^  | No      | No       | No          |
| Threads4     | Yes      | Yes            | No   | No      | No       | No          |
| ThreadsCores | Yes      | Yes            | No   | No      | No       | No          |
| ThreadsHyper | Yes      | Yes            | No   | No      | No       | No          |
| Optimal      | Yes      | Yes            | No   | Yes     | Yes      | No          |


Here, _Size_ refers to the length of the arrays in STREAM (i.e `STREAM_ARRAY_SIZE` / JUBE variable `size`), _Threads_ to the number of OpenMP threads (`OMP_NUM_THREADS`/ JUBE's `threadspertask`), _Affinity_ to the locations of spawned threads (`OMP_PLACES`, `OMP_PROC_BIND`, ...). For the _Threads1_ configuration, a report of additional array sizes is expected for the qualitative evaluation.

^: `STREAM_ARRAY_SIZE` should be 2^28 for the evaluation, but a scan through multiple array sizes (2^15 to 2^28) should be reported as part of the feedback.

## Verification

All execution should run without errors. In case of JUBE: status _done_, no errors.
Only in case of out-of-memory crashes, the range of `STREAM_ARRAY_SIZE` values can be narrowed to prevent them.

## Results

In most cases, the average triad bandwidth is to be reported. For the case of the freely chosen _Optimal_ configuration, please report the best bandwidth achieved. Should multiple memory domains be available (for example HBM and DDR), the benchmark is to be run separately for each domain.

Abbreviated example output follow.

| stage | modules | mem | CLK/Ins (>20) |   N |  Copy | Scale |   Add | Triad | runtime[sec] |
|-------|---------|-----|---------------|-----|-------|-------|-------|-------|--------------|
|  2022 |     GCC | 3.0 |          55.0 | 200 | 16786 | 19590 | 26872 | 26709 |         0.47 |
|  2022 |   Intel | 3.0 |          57.0 | 200 | 22554 | 26494 | 26441 | 26655 |         0.48 |
|  2022 |   NVHPC | 3.0 |         200.0 | 200 | 10701 |  9763 | 15062 | 14096 |         0.62 |
|  2022 |    AOCC |     |               | 200 |       |       |       |       |         0.49 |


| stage | modules | Exp | Array [MiB] | Thread [MiB] | Threads | CLK/Ins (>20) |   N |    Triad   | runtime[sec]  |
|-------|---------|-----|-------------|--------------|---------|---------------|-----|------------|---------------|
|  2022 |     GCC |  10 |         0.0 |          0.0 |       8 |           5.0 | 200 |   8589.9   |         0.37  |
|  2022 |     GCC |  12 |         0.0 |          0.1 |       8 |           4.0 | 200 |  34359.7   |         0.37  |
|  2022 |     GCC |  14 |         0.1 |          0.4 |       8 |           5.0 | 200 | 103079.2   |         0.36  |
|  2022 |     GCC |  16 |         0.5 |          1.5 |       8 |           6.0 | 200 | 329853.5   |         0.37  |
|  2022 |     GCC |  18 |         2.0 |          6.0 |       8 |          18.0 | 200 | 488671.8   |         0.38  |
|  2022 |     GCC |  20 |         8.0 |         24.0 |       8 |          41.0 | 200 | 514893.3   |         0.41  |
|  2022 |     GCC |  22 |        32.0 |         96.0 |       8 |         882.0 | 200 | 167478.2   |         0.92  |
|  2022 |     GCC |  24 |       128.0 |        384.0 |       8 |        3672.0 | 200 | 166520.4   |         2.95  |
|  2022 |     GCC |  26 |       512.0 |       1536.0 |       8 |       14652.0 | 200 | 159671.9   |         9.84  |
|  2022 |     GCC |  28 |      2048.0 |       6144.0 |       8 |       53737.0 | 200 | 127464.6   |        35.08  |
|  2022 |     GCC |  32 |     32768.0 |      98304.0 |       8 |      380153.0 | 200 | 153141.9   |       498.92  |

## Commitment

The following values should be reported. Any run with CLK/Ins <= 20 is not counted. For _Threads1_, the average triad bandwidth for array sizes from 2^15 to 2^28 is to be reported as well. The configuration chosen or _Optimal_ needs to be given.  
If multiple memory types are exposed to user space (for example HBM in flat mode), the benchmark should be run on each.

|     Name     |                    Values                   |
|--------------|---------------------------------------------|
| Threads1     | Average triad bandwidth for array size 2^18 |
| Threads4     | Average triad bandwidth for array size 2^18 |
| ThreadsCores | Average triad bandwidth for array size 2^18 |
| ThreadsHyper | Average triad bandwidth for array size 2^18 |
| Optimal      | Average triad bandwidth for array size 2^18 |

