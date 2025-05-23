# Terasort

The terasort application executes over two m7i.12xlarge invokers (48 vCPUs, 192 GB). It sorts 50GB of data, in 96 workers.

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

4. Launch a DragonFly server on c7i.24xlarge VM, ensure that the server is running and reachable.

5. Create the k8s cluster from YAML file:
```bash
git clone https://github.com/Burst-Computing/openwhisk-deploy-kube-burst.git
cd openwhisk-deploy-kube-burst
eksctl create cluster -f eks/terasort-half.yaml
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
    --bucket terasort-burst \
    --key terasort-50g \
    --backend redis-list \
    --ow-host [eks-control-plane-ip] \
    --ow-port 443 \
    --granularity [granularity] \
    --join False \
    --chunk-size 1024 \
    --runtime-memory [memory]  # control depending on granularity
```
