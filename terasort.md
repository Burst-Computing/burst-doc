# Terasort

The terasort application executes over two m7i.12xlarge invokers (48 vCPUs, 192 GB). It sorts 50GB of data, in 96 workers.

## Pre-requisites

Ensure you have the following prerequisites met in your environment:

- AWS CLI installed and configured with your AWS credentials.
- `eksctl` installed for managing the EKS cluster.
- Helm installed for deploying OpenWhisk.
- Docker installed for building the terasort application.
- Python 3.10 installed with the required dependencies.

The following resources are required in your AWS account:

- An AMI with the DragonFly server available in your AWS account.
- A VPC with at least two subnets in different availability zones for the EKS cluster.
- A security group for the DragonFly server allowing inbound traffic on the default redis port (6379).
- An S3 bucket for storing the terasort data.

## Launching the terasort application over AWS EKS

1. Clone burst validation repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation.git
cd burst-validation
```

2. Install the required dependencies:
```bash
pip install -r requirements.txt
```

3. Compile the terasort application:
```bash
cd terasort/terasort-burst
zip -r - * | docker run --rm -i burstcomputing/runtime-rust-burst:latest -compile main > ../terasort-burst.zip
```

4. Create the k8s cluster from YAML file:
```bash
git clone https://github.com/Burst-Computing/openwhisk-deploy-kube-burst.git
cd openwhisk-deploy-kube-burst
eksctl create cluster -f eks/terasort-half.yaml
```

5. Launch a DragonFly server on c7i.24xlarge VM (96 vCPUs, 192 GB), ensure that the server is running and reachable.
```bash
instance_id="$(aws ec2 run-instances \
  --image-id [ami-0123456789abcdef0] \  # Replace with your DragonFly server AMI ID
  --instance-type c7i.24xlarge \
  --subnet-id [subnet-0123456789abcdef0] \  # Replace with your subnet ID
  --security-group-ids [sg-0123456789abcdef0] \  # Replace with your security group ID
  --instance-market-options "MarketType=spot" \
  --count 1 \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=dragonfly-server}]" \
  --query "Instances[0].InstanceId" \
  --output text)"
aws ec2 wait instance-running --instance-ids "$instance_id"
internal_ip="$(aws ec2 describe-instances
  --instance-ids "$instance_id"
  --query "Reservations[0].Instances[0].PrivateIpAddress" 
  --output text)"
echo "DragonFly server internal IP: $internal_ip"
```

6. Deploy OpenWhisk:
    - Modify the `eks/deploy.yaml` file to set BCM address (ip $iternal_ip and port 6379) the `whisk.middleware` values. 
    - Deploy OpenWhisk:
    ```bash
    helm install owdev ./helm/openwhisk -n openwhisk --create-namespace -f eks/deploy.yaml
    ```

7. Launch the pagerank application:
```bash
PYTHONPATH=. python3 terasort/terasort_burst.py \
    --ts-endpoint https://s3.us-east-1.amazonaws.com \
    --partitions 96 \
    --bucket terasort-burst \
    --key terasort-50g \
    --backend redis-list \
    --ow-host [eks-control-plane-ip] \
    --ow-port 443 \
    --granularity 48 \
    --join False \
    --chunk-size 1024 \
    --runtime-memory [memory]  # control depending on granularity
```
