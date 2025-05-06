# Figure 8: Cooperative downloading

For this experiment, we create a pool of functions that will download a file from S3. The functions download chunks of the files of different sizes depending on the granularity declared.

If you prefer to skip running the cooperative downloading experiment, just jump to [render figure section](#rendering-the-figure) and plot the already generated results.

## Launching the cooperative downloading experiment 
1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results/download
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

5. Run the application:
```bash
python3 download/function-s3-2.py [granularity]
```
where `granularity` is the number of functions that download cooperatively the file.


## Rendering the figure
1. Clone the validation results repository:
```bash
git clone https://github.com/Burst-Computing/burst-validation-results.git
cd burst-validation-results/download
```

2. Execute the plot R script `plot.R` via RStudio. The plot will be generated in `download-paper.pdf`.