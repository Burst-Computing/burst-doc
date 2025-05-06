# Figure 1: Faas startup times

This action list explain how to run N parallel functions over AWS Lambda using Lithops. If you want to skip running the functions in AWS, just jump to [render figure section](#rendering-the-figure) and plot the already generated results.

## Launching N parallel functions over AWS Lambda
1. Clone Lithops and install it
```bash
git clone https://github.com/Burst-Computing/lithops-burst.git
```

2. Install the required dependencies
```bash
pip install .[aws]
```
3. Configure AWS account to use S3 and Lambda 
Visit [AWS Lambda Lithops backend](https://lithops-cloud.github.io/docs/source/compute_config/aws_lambda.html#configuration) and follow this instructions.

4. Configure the `~/.lithops/config` file for using S3 and Lambda:
```yaml
lithops:
  backend: aws_lambda
  storage: aws_s3
aws:
    aws_access_key_id: <your_aws_access_key_id>
    aws_secret_access_key: <your_aws_secret_access_key>
    aws_session_token: <your_aws_session_token>
    region: <your_aws_region>
aws_lambda:
    execution_role: <execution_role_arn>
```

5. Visit `examples/map_burst.py` and change the `BURST_ENABLED` variable to `False`. Adjust the `NUM_ACTIVATIONS` variable to the number of parallel functions you want to run.

6. Run the example
```bash
python3 examples/map_burst.py
```

7. Notice that the execution stats are stored in `./results.csv` file. 

## Rendering the figure
1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results
```

2. Execute the plot script:
```bash
python3 startup/plot.py
```

3. The figure will be saved in `startup/faas-startup-cdf.pdf`. 


