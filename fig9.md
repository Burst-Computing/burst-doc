# Figure 9: BCM - Throughput

For the experiment in the subfigure 9a, we use c7i.large EC2 (4 vCPUs and 8 GB RAM) for the workers, and c7i.16xlarge EC2 (64 vCPUs and 128 GB RAM) for the intermediate server. For the experiment in subfigure 9b, we use EC2 instances scaled to the burst size (from c7i.xlarge for 8 workers to c7i.48xlarge for 384).

For simplicity, the experiment is run directly on the AWS EC2 instances, without the OpenWhisk controller, as we are only concerned with the throughput of the BCM.

If you want to skip running the experiment, just jump to [render figure section](#rendering-the-figure) and plot the already generated results.

## Launching chunk size and maximum throughput experiments

1. Clone the validation repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation.git
cd burst-validation/microbenchmarks
```

2. Ensure you meet the prerequisites stated in the [AWS section](https://github.com/Burst-Computing/burst-validation/tree/main/microbenchmarks#execute-in-aws).

3. Follow the instructions in the [throughput microbenchmarks section](https://github.com/Burst-Computing/burst-validation/tree/main/microbenchmarks#message-chunk-size-and-maximum-throughput-benchmarks) to run the experiments for subfigure 9a and 9b.


## Rendering the figure

1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results
```

2. Execute the plot script:
```bash
python3 pairs2/pairs2.py # for subfigure 9a
python3 throughput/throughput.py # for subfigure 9b
```
