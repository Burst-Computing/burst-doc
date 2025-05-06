# Figure 6: Burst workers startup latencies

For this experiment, we need to create a k8s cluster in AWS EKS. The k8s cluster will be composed by 1 t4i.xlarge EC2 (4 vCPUs and
16 GB RAM) for the controller, and 20 c7i.12xlarge EC2 (48 vCPUs and 96 GB RAM) for the invokers.

If you want to skip running the large-scale burst in EKS, just jump to [render figure section](#rendering-the-figure) and plot the already generated results.

## Launching bursts over AWS EKS
0. Ensure that you're using Python 3.9.
```bash
python3 --version
```

1. Clone Lithops and install it
```bash
git clone https://github.com/Burst-Computing/lithops-burst.git
```

2. Install the required dependencies
```bash
pip install .[aws]
```

3. Clone openwhisk-deploy-kube-burst repository
```bash
git clone https://github.com/Burst-Computing/openwhisk-deploy-kube-burst.git
cd openwhisk-deploy-kube-burst
```

4. Create the k8s cluster from YAML file
```bash
eksctl create cluster -f eks/960spot.yaml
```

5. Configure the Lithops config file for using OpenWhisk deployed
```yaml
lithops:
  backend: openwhisk
  storage: aws_s3

aws:
  access_key_id: <your_aws_access_key_id>
  secret_access_key: <your_aws_secret_access_key>
  session_token: <your_aws_session_token>
  region: <your_aws_region>

openwhisk:
  endpoint: https://<eks-control-plane-ip>:443
  insecure: True
  runtime_memory: <runtime_memory>
  runtime: manriurv/lithops-plus-boto3:3.9
  namespace: guest
  api_key: 23bc46b1-71f6-4ed5-8c54-816aa4f8c502:123zO3xZCLrMN6v2BKK1dXYFpXlPkccOFqm12CdAsMgRU4VrNZ9lyGVCGuMDGIwP
```

6. Visit `examples/map_burst.py` and change the `BURST_ENABLE` variable to `True`. Adjust the `NUM_ACTIVATIONS` variable to 960 and `join=False`. Changing `granularity` to different values will launch different worker sizes.
```bash
python3 examples/map_burst.py
```

7. Notice that the execution stats are stored in `./results.csv` file.

## Rendering the figure
1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results/invocation
```

2. Execute the plot R script `main-boxplot-only-2.R` via RStudio. The plot will be generated in `boxplot-paper-2.pdf`.


