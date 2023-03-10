# Internal (Accumulator) Sampling Rate
This experiment provokes aliasing to determine the OCC's internal sampling rate.

This directory only contains results, the executed code is part of roco2 (see online or parent dir).

## Experiment Code
The actual experiment code is contained in roco2's "p9_highlow" configuration.

Note that you are on a P9 host, so you might have to recompile your dependencies!

> The versions here are the ones that were conveniently available.
> We expect the setup to work with other versions too, though we have not tested that.

Top-level dependencies are:

- score-p: https://www.vi-hps.org/projects/score-p used version 6.0
- score-p ibmpowernv plugin: https://github.com/score-p/scorep_plugin_ibmpowernv used commit 3fd626813a68c8d87f04ce12cc3373176fecb613
- roco2: https://github.com/tud-zih-energy/roco2/ used commit 189965c533abd94dbabe944de50f865da2ca53a9

the following pre-installed modules from the HPC cluster were used:
(note: first module is environment, can be ignored)

```
1) modenv/ml
2) libevent/2.1.8-GCCcore-8.2.0
3) GCCcore/8.3.0
4) SIONlib/1.7.6-GCCcore-8.3.0-tools
5) OTF2/2.2-GCCcore-8.3.0
6) ncurses/6.1-GCCcore-8.3.0
7) zlib/1.2.11-GCCcore-8.3.0
8) bzip2/1.0.8-GCCcore-8.3.0
9) cURL/7.66.0-GCCcore-8.3.0
10) CMake/3.15.3-GCCcore-8.3.0
11) binutils/2.32-GCCcore-8.3.0
12) GCC/8.3.0
13) OpenBLAS/0.3.7-GCC-8.3.0
14) numactl/2.0.12-GCCcore-8.3.0
15) XZ/5.2.4-GCCcore-8.3.0
16) libxml2/2.9.9-GCCcore-8.3.0
17) libpciaccess/0.14-GCCcore-8.3.0
18) hwloc/1.11.12-GCCcore-8.3.0
19) CUDA/10.1.243-GCC-8.3.0
20) gcccuda/2019b
21) OpenMPI/3.1.4-gcccuda-2019b
22) gompic/2019b
23) CubeLib/4.4.4-GCCcore-8.3.0
24) CubeWriter/4.4.3-GCCcore-8.3.0
25) libunwind/1.3.1-GCCcore-8.3.0
26) OPARI2/2.0.5-GCCcore-8.3.0
27) PAPI/6.0.0-GCCcore-8.3.0
28) PDT/3.25-GCCcore-8.3.0
29) Score-P/6.0-gompic-2019b
```

build roco2 from roco2 top-level dir with:

```
mkdir build && cd build
SCOREP_WRAPPER_INSTRUMENTER_FLAGS='--user --openmp --thread=omp --nocompiler' SCOREP_WRAPPER=off cmake .. -DCMAKE_C_COMPILER=scorep-gcc -DCMAKE_CXX_COMPILER=scorep-g++ -DUSE_SCOREP=ON -DBUILD_TESTING=OFF -DROCO2_HIGHLOW_INSTRUMENT_PHASES=OFF -DP9_HIGHLOW_FREQS=500+-10,1000+-10,2000+-10,2100+-10,4000+-10
make SCOREP_WRAPPER_INSTRUMENTER_FLAGS='--user --openmp --thread=omp --nocompiler'
```

to execute:

- put `libibmpowernv_plugin.so` into `LD_LIBRARY_PATH`
- adjust & submit the `run_slurm.sh` generated by roco2
  - maybe use the slurm option `--hint=multithread`

> To investigate more thoroughly apply roco2_duration.patch to roco2 to increase duration per configuration to 120s.
> This patch has **not** been used for the runs in the final submission.

## Files
- results
  - `raw/run04.tar.xz`: compressed trace of experiment execution, contains OCC readouts during different workload frequencies
  - `raw/checksums.txt`: checksum of `raw/run04.tar.xz`
  - `data/XXXhz_sensor.dat`: trace for workload alternating with `XXX` Hz of the **direct samples**, fields:
    - `time(s)`: timestamp in since beginning of workload at `XXX` Hz
    - `power(W)`: direct sample
  - `data/XXXhz_derived.dat`: trace for workload alternating with `XXX` Hz of the **power from energy**, fields:
    - `time(s)`: timestamp in since beginning of workload at `XXX` Hz
    - `power(W)`: power from energy
- analysis scripts
  - `extract_from_trace.py`: populate `data/` from trace in `raw/run04`; invoke with `make extract`
    - requires python otf2 module
- plots
  - `plot/sensor_vs_acc.py`: generate stand-alone gnuplot script comparing direct samples and power from energy spread, uses files in `data/`
- other files
  - `Makefile`: organize invocation, targets:
    - `make`, `make raw/run04`: extract raw trace
    - `make extract`: populate `data/` from raw trace (automatically unpacks trace archive)
  - `README.md`: this file
  - `roco2_duration.patch`: can be applied to roco2 to increase measurement duration for further analysis
