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
- An S3 bucket for storing the terasort data.

## Launching the terasort application over burst platform in AWS EKS

1. Create the k8s cluster from YAML file:
```bash
cd ~
git clone https://github.com/Burst-Computing/openwhisk-deploy-kube-burst.git
cd openwhisk-deploy-kube-burst
eksctl create cluster -f eks/terasort-half.yaml
```

2. Get the VPC ID of the created EKS cluster:
```bash
vpc_id="$(aws eks describe-cluster \
  --name terasort-cluster \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text)"
echo "EKS VPC ID: $vpc_id"
```

3. Create a security group for the DragonFly server with the required port:
```bash
sg_id="$(aws ec2 create-security-group \
  --group-name dragonfly-server-sg \
  --description "Security group for DragonFly server" \
  --vpc-id "$vpc_id" \
  --query "GroupId" \
  --output text)"
vpc_cidr="$(aws ec2 describe-vpcs \
  --vpc-ids "$vpc_id" \
  --query "Vpcs[0].CidrBlock" \
  --output text)"
aws ec2 authorize-security-group-ingress \
  --group-id "$sg_id" \
  --protocol tcp \
  --port 6379 \
  --cidr "$vpc_cidr"
echo "DragonFly server security group ID: $sg_id"
```

4. Launch a DragonFly server on c7i.24xlarge VM (96 vCPUs, 192 GB), ensure that the server is running and reachable.
```bash
subnet_id="$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$vpc_id" \
  --query "Subnets[0].SubnetId" \
  --output text)"
ami_id="$(aws ec2 describe-images \
  --owners self \
  --filters "Name=name,Values=ubuntu-dragonfly" \
  --query "Images[0].ImageId" \
  --output text)"
instance_id="$(aws ec2 run-instances \
  --image-id "$ami_id" \
  --instance-type c7i.24xlarge \
  --subnet-id "$subnet_id" \
  --security-group-ids "$sg_id" \
  --instance-market-options "MarketType=spot" \
  --count 1 \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=dragonfly-server}]" \
  --query "Instances[0].InstanceId" \
  --output text)"
aws ec2 wait instance-running --instance-ids "$instance_id"
internal_ip="$(aws ec2 describe-instances \
  --instance-ids "$instance_id" \
  --query "Reservations[0].Instances[0].PrivateIpAddress" \
  --output text)"
echo "DragonFly server internal IP: $internal_ip"
```

5. Deploy OpenWhisk:
    - Use helm to deploy OpenWhisk with the burst configuration:
    ```bash
    helm install owdev ./helm/openwhisk \
    -n openwhisk \
    --create-namespace \
    -f eks/ow-burst.yaml \
    --set whisk.middleware.RedisList="redis://$internal_ip:6379" \
    --set whisk.containerPool.userMemory="192000m" \
    --set whisk.limits.actions.memory.max="192000m"
    ```
    - Wait for the resources to be created and the OpenWhisk deployment to be ready.
    - Get the EKS control plane IP address:
    ```bash
    eks_control_plane_ip="$(kubectl get svc \
    -n openwhisk owdev-nginx \
    -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
    echo "EKS control plane IP: $eks_control_plane_ip"
    ```

6. Clone burst validation repository:
```bash
cd ~
git clone https://github.com/Burst-Computing/burst-validation.git
cd burst-validation
```

7. Install the required dependencies:
```bash
pip install -r requirements.txt
```

8. Compile the terasort application:
```bash
cd terasort/ow-burst
zip -r - * | docker run --rm -i burstcomputing/runtime-rust-burst:latest -compile main > ../terasort-burst.zip
```

9. Launch the terasort application:
```bash
cd ~/burst-validation
PYTHONPATH=. python3 terasort/terasort_burst.py \
    --ts-endpoint https://s3.us-east-1.amazonaws.com \
    --partitions 96 \
    --bucket terasort-burst \
    --key terasort-50g \
    --backend redis-list \
    --ow-host "$eks_control_plane_ip" \
    --ow-port 443 \
    --granularity 48 \
    --join False \
    --chunk-size 1024 \
    --runtime-memory 192000
```

10. The stats will be saved in the `terasort-burst.csv` file in the current directory. The file will be needed for rendering the figure.

11. Stop the DragonFly server:
```bash
aws ec2 terminate-instances --instance-ids "$instance_id"
aws ec2 wait instance-terminated --instance-ids "$instance_id"
```

12. Delete the security group:
```bash
aws ec2 delete-security-group --group-id "$sg_id"
```

## Launching the terasort application a la MapReduce in AWS EKS

The steps are similar to the burst mode:

1. You can use the same EKS cluster created in the previous section, but you will need to delete the existing burst OpenWhisk deployment and deploy the classic OpenWhisk.
```bash
cd ~/openwhisk-deploy-kube-burst
helm uninstall owdev -n openwhisk
# wait for the resources to be deleted
helm install owdev ./helm/openwhisk \
  -n openwhisk \
  --create-namespace \
  -f eks/ow-classic.yaml \
  --set whisk.containerPool.userMemory="192000m" \
  --set whisk.limits.actions.memory.max="192000m"
eks_control_plane_ip="$(kubectl get svc \
  -n openwhisk owdev-nginx \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
echo "EKS control plane IP: $eks_control_plane_ip"
```

2. Compile the terasort application:
```bash
cd ~/burst-validation/terasort/ow-map
zip -r - * | docker run --rm -i burstcomputing/runtime-rust-map:latest -compile main > ../terasort-map.zip
cd ../ow-reduce
zip -r - * | docker run --rm -i burstcomputing/runtime-rust-reduce:latest -compile main > ../terasort-reduce.zip
```

3. Launch the terasort application:
```bash
cd ~/burst-validation
PYTHONPATH=. python3 terasort/terasort_mapreduce.py \
    --ts-endpoint https://s3.us-east-1.amazonaws.com \
    --partitions 96 \
    --bucket terasort-burst \
    --key terasort-50g \
    --ow-host "$eks_control_plane_ip" \
    --ow-port 443 \
    --runtime-memory 4000
```

4. The stats will be saved in the `terasort-classic-stats.csv` file in the current directory. The file will be needed for rendering the figure.

5. Stop the EKS cluster:
```bash
cd ~/openwhisk-deploy-kube-burst
eksctl delete cluster -f eks/terasort-half.yaml
```

## Rendering the figure

1. Clone the validation results repository:
```bash
cd ~
git clone ghttps://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results/terasort
```

2. Copy the stats files from the previous steps:
```bash
cp ~/burst-validation/terasort/terasort-burst.csv .
cp ~/burst-validation/terasort/terasort-classic-stats.csv .
```

3. Update the `burst-validation-results/terasort/plot.py` script to set the correct paths for the stats files (variables `INPUT_MR_FILE` for MapReduce and `INPUT_BURST_FILE` for burst).
```python
INPUT_MR_FILE = 'terasort/terasort-classic-stats.csv'
INPUT_BURST_FILE = 'terasort/terasort-burst.csv'
```

4. Execute the plot script:
```bash
cd ~/burst-validation-results
python3 terasort/plot.py
```

5. The figures will be saved as `terasort_mr.pdf` and `terasort_burst.pdf` in the current directory.

