# Figure 10: BCM - Latencies

For this experiment, we use one, two and four c7i.12xlarge VMs (48 vCPUs, 96 GB) for bursts of sizes 48, 96 and 192, respectively. The backend server runs on one c7i.48xlarge (192 vCPUs, 384 GB).

For simplicity, the experiment is run directly on the AWS EC2 instances, without the OpenWhisk controller, as we are only concerned with the throughput of the BCM.

If you want to skip running the experiment, just jump to [render figure section](#rendering-the-figure) and plot the already generated results.

## Launching the experiments

The steps to run the experiments are similar to the ones in the previous experiment ([Launching chunk size and maximum throughput experiments](./fig9.md#launching-chunk-size-and-maximum-throughput-experiments)).

1. Clone the validation repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation.git
cd burst-validation/microbenchmarks
```

2. Ensure you meet the prerequisites stated in the [AWS section](https://github.com/Burst-Computing/burst-validation/tree/main/microbenchmarks#execute-in-aws).

3. Follow the instructions in the [group collective benchmarks](https://github.com/Burst-Computing/burst-validation/tree/main/microbenchmarks#group-collevtive-benchmarks) to run the experiments for subfigure 10a and 10b.


## Rendering the figure

1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results
```

2. Execute the plot script:
```bash
python3 collectives/performance_plots_2in1.py
```
