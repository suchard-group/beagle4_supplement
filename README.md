# BEAGLE 4.1: A high-performance library for evaluating gradients on phylogenetic trees across heterogeneous computing architectures

Please see instructions to compile BEAGLE and install BEAST for reproducing the benchmarks.

## Results

Log files, BEAST XML files, and associated scripts to reproduce the results and figures have been deposited on [Zenodo](https://doi.org/10.5281/zenodo.20403169).

## Installation 

BEAGLE

```
git clone -b tensor-cores https://github.com/beagle-dev/beagle-lib.git
cd beagle-lib/
mkdir build
cd build/
cmake -DBEAGLE_BENCHMARK_ENERGY=OFF -DBEAGLE_TENSOR_CORES=ON -DCMAKE_INSTALL_PREFIX=$HOME ..
make
make install
```

You can set the install location using `CMAKE_INSTALL_PREFIX`. 

BEAST

You can use the JAR file provided in `BEAST_JAR/beast.jar`.

## File structure

Log files, BEAST XML files, and associated scripts to reproduce the results and figures have been deposited on [Zenodo](https://doi.org/10.5281/zenodo.20403169).

```
├── results
│   ├── gpu_comparison_benchmarks
│   ├── gpu_multiple_instances_benchmark
│   ├── neon_benchmark
│   ├── random_effects_subst_model
│   └── taxa_patterns_benchmark
├── scripts
│   ├── plot_gpu_timings.R
│   ├── plot_multiple_instance_timings.R
│   ├── plot_timings.R
│   ├── plot_tips_benchmarks.R
│   ├── plot_tips_benchmarks_nuc.R
│   └── remove_seq_data.py
└── xml
    ├── gpu_comparison_benchmarks
    ├── gpu_multiple_instances_benchmark
    ├── neon_benchmark
    ├── random_effects_subst_model
    └── taxa_patterns_benchmark
```

XMLs (in `xml`) for each benchmark are organized in the following folders,

| Folder | Description |
|:--- |:--- |
| taxa_patterns_benchmark | Benchmark BEAGLE on CPU, GPU, and GPU tensor cores for varying patterns and taxa |
| gpu_comparison_benchmarks | Benchmark BEAGLE on AMD and NVIDA GPUs |
| gpu_multiple_instances_benchmark | Benchmark BEAGLE on multiple GPUs |
| neon_benchmark | Benchmark BEAGLE NEON instructions |
| random_effects_subst_model | Benchmark random effects substitution model |


Log files for each benchmark are organized in the same manner in `./results/`. The time taken by BEAGLE for calculating the likelihood and evaluating the gradient is reported in milliseconds in each log file as follows,

```
TIMING:
likelihood              5.700e+01

TIMING:
jointGradient.branchRates.ratesnull           3.535e+03
```

Scripts to parse the total beagle timing from the log files to reproduce figures are in `./scripts/`


## Running benchmarks

To reproduce the results for any XML on multi-core CPUs, GPU, or GPU tensor core hardware implementation, you can modify the script below.

```
#!/bin/bash

xml=$1
n=$(basename $xml)
n=${n/.xml/_profile}

gpu=1
nrep=1

# Set path to BEAGLE lib
export LD_LIBRARY_PATH=$HOME/lib

# Set path to BEAST jar
BEAST_JAR=BEAST_JAR/beast.jar

cmd=(
  -Xms2G
  -Xmx4G
  -Djava.library.path="$LD_LIBRARY_PATH"
  -jar "$BEAST_JAR"
  -seed 112358
  -overwrite
)

run_cmd() {
  local prefix="$1"
  shift

  java "$@"
}

#CPU
for instances in 1 2 4 6 8 16; do
    prefix=${xml/.xml/}_cpu_instances_${instances}_rep_${nrep}
    run_cmd "$prefix" "${cmd[@]}" -beagle_CPU -beagle_SSE -beagle_instances $instances $xml > ${xml/.xml/}_cpu_sse_instances_${instances}_rep_${nrep}.txt 2>&1
    sleep 2
done

# GPU
prefix=${xml/.xml/}_gpu_rep_${nrep}
run_cmd "$prefix" "${cmd[@]}" -beagle_GPU -beagle_order 1 $xml > ${prefix}.txt 2>&1

sleep 2

#GPU tensor cores
prefix=${xml/.xml/}_gpu_tensor_core_${nrep}
run_cmd "$prefix" "${cmd[@]}" -beagle_GPU -beagle_tensor_core -beagle_order 1 $xml > ${prefix}.txt 2>&1
sleep 2
```