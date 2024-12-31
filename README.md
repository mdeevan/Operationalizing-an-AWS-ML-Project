## Step 1: Training and deployment on Sagemaker

### Sagemaker Notebook setup

The training is meant to be performed via a batch training job, for which one can choose a GPU instances. To run the job, choosing low cost available instances works just fine. In choose "T" or "M" types, I decided in favor of "M", which provides balance of compute and memory. 

Due to the un-availablilty of ml.m5.large for the notebook instance, I choose ml.m5d.large, which costs little more and provides NVMe storage instead of EBS-Only SSD

"_M5 instances offer a balance of compute, memory, and networking resources for a broad range of workloads. This includes web and application servers, small and mid-sized databases, cluster computing, gaming servers, caching fleets, and app development environments. Additionally, M5d, M5dn, and M5ad instances have local storage, offering up to 3.6TB of NVMe-based SSDs._" From amazon https://aws.amazon.com/ec2/instance-types/m5/

 ![sagemaker ai notebook instance](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/sagemaker%20ai%20notebook%20instance.png)


### S3 bucket

**sagemaker-course5** bucket created to hold the image data for training and testing the ML algorithm.

![s3 bucket](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/s3%20bucket.png)


### Training and Deployment

Two separate inference points (deployed model) were created
1. *Single instance Trained model
   - Name : pytorch-inference-2024-12-30-19-13-26-424
   - ARN  : arn:aws:sagemaker:us-east-1:987941310748:endpoint/pytorch-inference-2024-12-30-19-13-26-424
   - URL  : https://runtime.sagemaker.us-east-1.amazonaws.com/endpoints/pytorch-inference-2024-12-30-19-13-26-424/invocations
   - Model container logs : /aws/sagemaker/endpoints/pytorch-inference-2024-12-30-19-13-26-424
  
2. *Multi instsance trained model
   - Name : pytorch-inference-2024-12-30-20-04-25-418
   - URL  : https://runtime.sagemaker.us-east-1.amazonaws.com/endpoints/pytorch-inference-2024-12-30-20-04-25-418/invocations
   - ARN  : arn:aws:sagemaker:us-east-1:987941310748:endpoint/pytorch-inference-2024-12-30-20-04-25-418
   - Model container logs : /aws/sagemaker/endpoints/pytorch-inference-2024-12-30-20-04-25-418
  
_* Instance were since removed to save on recurring cost._
 
![Inference Endpoints](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/Endpoints%20Amazon%20SageMaker%20AI%20us-east-1.png)


## EC2 Training

A GPU equipped machines is better suited to train a CNN, which helps reduce the time to train. For this reason **g6.xlarge** was choosen. These are "Amazon EC2 G6 instances are designed to accelerate graphics-intensive applications and machine learning inference." (https://aws.amazon.com/ec2/instance-types/)
They features
  - 3rd generation AMD EPYC processors (AMD EPYC 7R13)
  - Up to 8 NVIDIA L4 Tensor Core GPUs
  - Up to 100 Gbps of network bandwidth
  - Up to 7.52 TB of local NVMe local storage

As a result the training completed in under 10 minutes, which took over 20 mins for ml.g6.large
![EC2 instance with g6.xlarge](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/Instances%20EC2%20us-east-1.png)

