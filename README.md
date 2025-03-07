# Operationalizing-an-AWS-ML-Project

## Project Summary

Project comprises of following steps:  

1. Train and deploy a model on Sagemaker, using the most appropriate instances. Set up multi-instance training in your Sagemaker notebook
2. Adjust your Sagemaker notebooks to perform training and deployment on EC2
3. Set up a Lambda function for your deployed model. Set up auto-scaling for your deployed endpoint as well as concurrency for your Lambda function
4. Ensure that the security on your ML pipeline is set up properly.

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


## Step 2: EC2 Training

A GPU equipped machines is better suited to train a CNN, which helps reduce the time to train. For this reason **g6.xlarge** was choosen. These are "Amazon EC2 G6 instances are designed to accelerate graphics-intensive applications and machine learning inference." (https://aws.amazon.com/ec2/instance-types/)
They features
  - 3rd generation AMD EPYC processors (AMD EPYC 7R13)
  - Up to 8 NVIDIA L4 Tensor Core GPUs
  - Up to 100 Gbps of network bandwidth
  - Up to 7.52 TB of local NVMe local storage

As a result the training completed in under 10 minutes, which took over 20 mins for ml.g6.large
![EC2 instance with g6.xlarge](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/Instances%20EC2%20us-east-1.png)



## Step 3: Lambda Function
lambda was setup that invokes the endpoint for inferring image class.

![deployed lambda](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/Functions%20Lambda.png)

![lambda function](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/lambda%20function-code.png)

![Lambda Monitor](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/lambda%20monitor%20udacity-course5.png)

## Step 4: Security and testing

lambda was granted invoke access to the specific endpoint, so it can call the inference endpoint. Access to AWS resources is denied by default and only the ones with explicit 'allow' can access the resources. so, it's pretty safe from invocation standpoint.

![lambda test result](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/lambda%20result.png)

![IAM policies](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/IAM%20Lambda%20udacity-course5-role-huip3qfy%20IAM%20Global.png)


## Step 5: Concurrency and auto-scaling

**Concurrency**: 
Its an configuration setting that defines the count of parallel invocation of the lambda function. By default there is only one instance of the lambda function unless additional instances are configured (see below). If the lambda invocation exceeds the execution time of the lambda, a performance degradation will be obsereved, as requests are queued for instance availability. Provisioning additional instances improves performance but also incur cost. One other aspect to consider regarding performance is cold-start. Lambda function execution environment lifecycle involves dowloading the code, starting execution environment, execution of initialization code and finally code execution. The environment stays warm for a little while once execution ends, in anticipation of future invocation before dying out. Getting ready to execute the code is termed as cold-start. Below are the ways to address cold-start.   

Concurrency is of two types, reserved and provisioned.

_Reserved concurrency_: Number of instances available to respond to the request. when there are more invocation then the instances, the invocations are queued and processed as instances frees up from previous execution

_Provisioned concrrency_: Always-on, ready and warmed up instances to serve the request. There is a cost associated with it, since instances are always on.

**auto-scaling**:
auto-scaling applies to the end points and it is a means to increase the number of endpoints to provide improved performance as traffic increases. There is a cost associated with the endpoints, and depends on the EC2 instances used. The configuration allows to define the maximum number of endpoint instance count. Optionally, one can define the cool-down period for scale-in, scale-out to ensure that new instances are not created immediately when invocations are increased and are not reduce immediately as invocations are reduced. This helps as the setting up the endpoint and tearing down takes some amount of time.

## STANDOUT SUGGESTIONS

## Distributed training or distributed data parallel (DDP)
Attempted DPP with **spot instance** training with following configuration. All attempts failed due to Account limitation on using the spot instances of P3, P2, Trn Spot Instances.

_ResourceLimitExceeded: An error occurred (ResourceLimitExceeded) when calling the CreateTrainingJob operation: The 
account-level service limit 'ml.trn1.2xlarge for spot training job usage' is 0 Instances, with current utilization 
of 0 Instances and a request delta of 2 Instances. Please use AWS Service Quotas to request an increase for this 
quota. If AWS Service Quotas is not available, contact AWS support to request an increase for this quota._

_ClientError: An error occurred (AccessDeniedException) when calling the CreateTrainingJob operation: User: arn:aws:sts::xxxxxxxxxxxx:assumed-role/AmazonSageMaker-ExecutionRole-20241229T155658/SageMaker is not authorized to perform: sagemaker:CreateTrainingJob on resource: arn:aws:sagemaker:us-east-1:xxxxxxxxxxxx:training-job/dog-pytorch-DDP-2025-01-02-16-54-30-431 with an explicit deny in a service control policy_

There are multiple modes available for DDP

DDP training offers multiple options, I attempted
1. Training using SMDataParallel Distributed Training Framework _distribution={"smdistributed": {"dataparallel": {"enabled": True}}},_ this required P3 Instances for training
2. Training with Torch_distributed:  _distribution={"torch_distributed": {"enabled": True}    }_ This works on GPU enabled and I attempted it on TRN instances
 

DDP Training: https://sagemaker-examples.readthedocs.io/en/latest/training/distributed_training/pytorch/data_parallel/mnist/pytorch_smdataparallel_mnist_demo.html

spot training: https://github.com/aws-samples/amazon-sagemaker-managed-spot-training/blob/main/pytorch_managed_spot_training_checkpointing/pytorch_managed_spot_training_checkpointing.ipynb




## MULTI INSTANCE TRAINING
Model was successfully trained with two instances of **ml.g4dn.xlarge** and deployed.


## AWS API Gateway
REST API was setup to invoke the lambda function.

![AWS API Gateway](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/API%20Gateway%20-%20APIs.png)

![successful invocation via API](https://github.com/mdeevan/Operationalizing-an-AWS-ML-Project/blob/master/screenshots/API%20call%20from%20browser%20.png)
