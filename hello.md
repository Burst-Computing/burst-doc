# Hello world in Burst Computing
This example wants to show how to use the Burst Computing framework to run a minimal example that involves all the components of the system. The example is a terasort application, which sorts a 1GB file. The example is implemented in Rust and uses the OpenWhisk framework to run the application in burst mode.

## How to run the Hello World
This minimal example uses the simplest infrastructure to run the terasort application in burst mode. It also uses prebuilt Docker images for straightforward deployment. It has been successfully reproduced on a single m7i.xlarge (4 vCPUs, 16 GB RAM) EC2 instance.

Setup:
- Ubuntu Desktop 24.04
- Docker

It doesn't intend to be a production-ready example: to show large-scale burst computing, you need to move to "Detailed Instructions" section.

1. **Create a Minikube cluster**.
   - Install Minikube (https://minikube.sigs.k8s.io/docs/start/ and kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
   - Start Minikube:
        ```bash
        minikube start --cpu max --memory max
        ```

2. **Launch a communication backend inside k8s**.

    For communicate the different components of the system, we need to deploy a communication backend. We propose to launch Dragonfly.
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/Burst-Computing/openwhisk-deploy-kube-burst/refs/heads/master/dragonfly-minikube.yaml
    ```

3. **Deploy Openwhisk**.
You need to deploy Openwhisk over the Kubernetes cluster.
    - Install Helm (https://helm.sh/docs/intro/install/).
    - Label the minikube node:
        ```bash
        kubectl label nodes minikube openwhisk-role=invoker
        ```
    - Clone the openwhisk-deploy-kube-burst repository:
        ```bash
        git clone https://github.com/Burst-Computing/openwhisk-deploy-kube-burst.git
        cd openwhisk-deploy-kube-burst
        ```
    - Deploy Openwhisk:
        ```bash
        helm install owdev ./helm/openwhisk -n openwhisk --create-namespace -f minikube.yaml
        ```

4. Configure the data (MinIO deployment) over minikube:
    - Deploy MinIO:
        ```bash
        kubectl apply -f https://raw.githubusercontent.com/Burst-Computing/openwhisk-deploy-kube-burst/refs/heads/master/minio.yaml
        ```
    - Modify `/etc/hosts` to add the MinIO service:
        ```bash
        echo "$(minikube ip) minio-service.default.svc.cluster.local" | sudo tee -a /etc/hosts
        ```
    - Install MinIO client: [instructions here](https://min.io/docs/minio/linux/reference/minio-mc.html#install-mc)
    - Create a bucket in MinIO:
        ```bash
        mc alias set minio http://minio-service.default.svc.cluster.local:30000 minioadmin minioadmin
        mc mb minio/burstcomputing
        ```
    - Transfer the 1GB file to the bucket:
        ```bash
        curl -L https://github.com/Burst-Computing/terasort-1gb/raw/refs/heads/main/terasort-1g | mc pipe minio/burstcomputing/terasort-1g
        ```
    - Verify the file is in the bucket:
        ```bash
        mc ls minio/burstcomputing
        ```

5. **Launch the terasort application in burst mode**.
    - Clone the burst-validation repository:
        ```bash
        git clone https://github.com/Burst-Computing/burst-validation.git
        cd burst-validation
        ```
    - Ensure you're using Python 3.10:
        ```bash
        python3 --version
        ```
    - Create a virtual environment:
        ```bash
        python3 -m venv venv
        source venv/bin/activate
        ```
    - Install the required dependencies:
        ```bash
        pip install -r requirements.txt
        ```
    - Execute the terasort application:
        ```bash
        PYTHONPATH=. python3 terasort/terasort_burst.py \
            --ts-endpoint http://minio-service.default.svc.cluster.local:30000 \
            --partitions $(nproc) \
            --bucket burstcomputing \
            --key terasort-1g \
            --backend redis-list \
            --ow-host 192.168.49.2 \
            --ow-port 31001 \
            --granularity $(($(nproc)/2)) \
            --chunk-size 1024 \
            --runtime-memory 10240  # 10GB
        ```
    - The application will download a 1GB file in the S3 bucket and sort it. Data shuffling will be done using the communication middleware. 



