# Figure 12: Terasort application

The terasort application executes over two m7i.24xlarge invokers (96 vCPUs, 384 GB). It sort 100GB of data, in 192 workers. 

If you want to skip running the large-scale execution in EKS, just jump to [render figure section](#rendering-the-figure) and plot the already generated results.

## Launching the pagerank application over AWS EKS
1. Clone burst validation repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation.git
cd burst-validation
```
2. Install the required dependencies:
```bash
pip install -r requirements.txt
```

3. Compile the pagerank application:
```bash
cd terasort/terasort-burst
zip -r - * | docker run --rm -i burstcomputing/runtime-rust-burst:latest -compile main > ../terasort-burst.zip
```

4. Launch a DragonFly server on c7i.48xlarge VM, ensure that the server is running and reachable.

5. Create the k8s cluster from YAML file:
```bash
git clone https://github.com/Burst-Computing/openwhisk-deploy-kube-burst.git
cd openwhisk-deploy-kube-burst
eksctl create cluster -f eks/terasort.yaml
```

6. Deploy OpenWhisk:
    - Modify the `eks/deploy.yaml` file to set BCM address (ip and port) the `whisk.middleware` values. 
    - Deploy OpenWhisk:
    ```bash
    helm install owdev ./helm/openwhisk -n openwhisk --create-namespace -f eks/deploy.yaml
    ```

7. Launch the pagerank application:
```bash
PYTHONPATH=. python3 terasort/terasort_burst.py \
    --ts-endpoint https://s3.us-east-1.amazonaws.com \
    --partitions 192 \
    --bucket burstcomputing \
    --key terasort-100g \
    --backend redis-list \
    --ow-host [eks-control-plane-ip] \
    --ow-port 443 \
    --granularity [granularity] \
    --join False \
    --chunk-size 32 \
    --runtime-memory [memory]  # control depending on granularity
```

## Rendering the figure
1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results/download
```

2. Execute the plot script:
```bash
python3 pagerank/plot.py
```
