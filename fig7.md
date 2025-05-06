# Figure 7: Simultaneity - FaaS vs Burst

For this experiment, we need to create a k8s cluster in AWS EKS. The k8s cluster will be composed by 1 t4i.xlarge EC2 (4 vCPUs and
16 GB RAM) for the controller, and 20 c7i.12xlarge EC2 (48 vCPUs and 96 GB RAM) for the invokers.

Running the bursts requires the same steps like in the previous experiment ([Launching bursts over AWS EKS](./fig6.md#launching-bursts-over-aws-eks)). 

So, you find here directly the steps to render the output figure.

## Rendering the figure
1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results/invocation
```

2. Execute the plot R script `main-invocation.R` via RStudio. The plot will be generated in `invocation-paper.pdf`.


