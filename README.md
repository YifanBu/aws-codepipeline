# AWS DevOps
### Codepipeline - S3 Static Site

![Screenshot_2](https://user-images.githubusercontent.com/31915035/196174889-490ef1b3-3b96-4340-8299-f1f5cd29b14b.png)

Create and update codepipeline

Source Stage: S3 as source

Deploy Stage: S3

### CodeCommit

Connecting to AWS CodeCommit repositories and pushing content to them.

Creating pipelines from CodeCommit repositories and tirggering them using git commands.

Editing files and creating commits manually on AWS CodeCommit Console when needed.

Source Stage: CodeCommit

Deploy Stage: S3

!!!!!!!!!!!!!All the service used MUST be in the same region

### CodeBuild

Providing our build commands to AWS CodeBuild with our source code

Checking build logs in case of build failures

Using CodeBuild for automated tests on our pipelines.

CodeCommit: MyAngularRepo

CodePipeline: MyAngularPipeline

S3: my-angular-website

![Screenshot_3](https://user-images.githubusercontent.com/31915035/196174971-64654a1f-40b1-4675-b327-4942e52fd71a.png)

CodeBuild: MyAngularBuild

![Screenshot_4](https://user-images.githubusercontent.com/31915035/196175002-edd85846-87d9-4402-861f-83e8172f5545.png)


Deploy Stage:

![Screenshot_5](https://user-images.githubusercontent.com/31915035/196175021-2a024948-2489-4c15-8526-9c45bcd996b5.png)

### CodeDeploy

Deploy to EC2 instance

Configure EC2 instance for CodeDeploy Deployments

Create an IAM service role for CodeDeploy: AWSCodeDeployRole → CodeDeployEC2ServiceRole

CodeDeploy Deployment group:

![Screenshot_6](https://user-images.githubusercontent.com/31915035/196175069-13e28ba0-afa5-4212-97a5-2a81697bc3ba.png)

Add CodeDeploy Deploy Action to Pipeline

Creating an Appspec File for Deployments to EC2 Instance

![Screenshot_7](https://user-images.githubusercontent.com/31915035/196175104-5d2e913c-ea2d-454e-a5f0-e78894bc025d.png)

Deployment lifecycle events for debugging

Deployment Logs on EC2:

`ls /opt/codedeploy-agent/deployment-root/deployment-logs`

Streaming Deployment Logs to CloudWatch Logs: 

![Screenshot_8](https://user-images.githubusercontent.com/31915035/196175140-5f1245fc-7860-4b7f-bb60-76018d2bee34.png)

### Creating a Deployment Group With Auto Scaling and Load Balancing

![Screenshot_9](https://user-images.githubusercontent.com/31915035/196175180-6a02a023-32f3-48da-9541-e4ec987027e1.png)

In-Place All-At-Once Deployment is not suitable for PROD env.

### CloudFormation

Defining manual approval actions on our pipelines

Using action variabes on pipelines

Superseded pipeline executions

notification rules on our pipelines.

IAM Service Role: CloudFormationServiceRoleForMyPipeline

- AmazonEC2FullAccess
- IAMFullAccess

Adding a CloudFormation Deploy Action to the pipeline

SNS Topic

Manual approval:

![Screenshot_11](https://user-images.githubusercontent.com/31915035/196175286-4ecdf8fb-9d48-492f-b548-ae243df20b91.png)

Action Variables:

### ECS

Preparing AWS CodeBuild buildspecs to build Docker images and pushing  them to Docker Hub

Defining  environment  variables on CodePipeline and  CodeBuild  to store Docker  Hub credentials and other data

Building a pipeline on AWS CodePipeline to build  and push your  Docker images  using  AWS CodeCommit  and  AWS CodeBuild

Using AWS Secrets Manager  with AWS CodeBuild  to store your  Docker Hub credentials as environment variables.

Using AWS System Parameter Store environment variables on AWS CodeBuild as a free alternative.

Pushing your Docker image built with AWS Pipeline  and AWS CodeBuild  to Amazon ECR

ECS task definition, ECS cluster with Fargate and ASG, ECS with rolling deployments

ECS standard deploy action

Tagging Docker images built by your pipeline with Git commit IDs on AWS CodeCommit that created them.

Using ECR Public Gallery to pull offical Docker images

### DockerHub Credential

`docker login -u ivanbuau`

dckr_pat_iMNPqEkEhIhMLdEuRrM9ZdP9OHE

### Secret Manager

![Screenshot_12](https://user-images.githubusercontent.com/31915035/196175333-5daae9b3-9e47-4ae7-a024-abaaa1a034b5.png)

![Screenshot_13](https://user-images.githubusercontent.com/31915035/196175351-7b3946c2-496d-430f-952d-ed56293c2a0a.png)

buildspec.yml

```yaml
env:
  secrets-manager:
    DOCKERHUB_TOKEN: dockerhub:token
    DOCKERHUB_USER: dockerhub:user
```

### Parameter Store

![Screenshot_14](https://user-images.githubusercontent.com/31915035/196175383-8f38c47f-14fd-491a-af40-c43acf48c7c1.png)

![Screenshot_15](https://user-images.githubusercontent.com/31915035/196175399-961fcf74-bc1f-488c-8d06-58037453e426.png)

```yaml
env:
  parameter-store:
    DOCKERHUB_TOKEN: /dockerhub/token
    DOCKERHUB_USER: /dockerhub/user
```

### ECR

ECRAccessPolicy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",  
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:GetAuthorizationToken",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
              ],
            "Resource": "*"
        }
    ]
}
```

buildspec.yml

```yaml
phases:
  pre_build:
    commands:
      # Docker Hub login
      - echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USER} --password-stdin 
      
      # ECR login
      - ECR_MAIN_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
      - aws ecr get-login-password --region ${AWS_REGION} | docker login -u AWS --password-stdin ${ECR_MAIN_URI}

      - ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:latest"
```

### ECS

task definition

![Screenshot_17](https://user-images.githubusercontent.com/31915035/196175459-f2040ecd-02f5-42d2-b3c0-adf43ca3e747.png)


Create ECS Cluster

![Screenshot_16](https://user-images.githubusercontent.com/31915035/196175503-0a45701b-0186-445a-adfa-ba2099f3153a.png)

![Screenshot_18](https://user-images.githubusercontent.com/31915035/196175517-e5990923-2c4b-45aa-b68e-af2a74f1a33d.png)

Create an ECS Service on Fargate for Rolling Deployments

![Screenshot_19](https://user-images.githubusercontent.com/31915035/196175570-c101ec94-4ab3-44cd-9b55-1fbc13b594c3.png)

Create a new Security Group for ALB

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/default-custom-security-groups.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/default-custom-security-groups.html)

![Screenshot_20](https://user-images.githubusercontent.com/31915035/196175589-f1f47103-dc5a-4573-aa09-836d474636fb.png)

### Pipeline

Adding an ECS Standard Deploy Action to your Pipeline
