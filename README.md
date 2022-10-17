# Codepipeline - S3 Static Site

![Screenshot_2.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c0b7123-6dc4-4c00-8a5f-98b6920e3a15/Screenshot_2.png)

Create and update codepipeline

Source Stage: S3 as source

Deploy Stage: S3

# CodeCommit

Connecting to AWS CodeCommit repositories and pushing content to them.

Creating pipelines from CodeCommit repositories and tirggering them using git commands.

Editing files and creating commits manually on AWS CodeCommit Console when needed.

Source Stage: CodeCommit

Deploy Stage: S3

!!!!!!!!!!!!!All the service used MUST be in the same region

# CodeBuild

Providing our build commands to AWS CodeBuild with our source code

Checking build logs in case of build failures

Using CodeBuild for automated tests on our pipelines.

CodeCommit: MyAngularRepo

CodePipeline: MyAngularPipeline

S3: my-angular-website

![Screenshot_3.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7ef6ab04-c138-4da5-8034-c00162541e7b/Screenshot_3.png)

CodeBuild: MyAngularBuild

![Screenshot_4.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/82dafdb1-4b4c-44e7-b6fb-2fa8b20325ce/Screenshot_4.png)

Deploy Stage:

![Screenshot_5.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a63723d2-3ffb-4c34-8efd-c09957187e1f/Screenshot_5.png)

# CodeDeploy

Deploy to EC2 instance

Configure EC2 instance for CodeDeploy Deployments

Create an IAM service role for CodeDeploy: AWSCodeDeployRole → CodeDeployEC2ServiceRole

CodeDeploy Deployment group:

![Screenshot_6.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/35daff48-2ade-48a3-a793-277b6a312551/Screenshot_6.png)

Add CodeDeploy Deploy Action to Pipeline

Creating an Appspec File for Deployments to EC2 Instance

![Screenshot_7.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/971366ce-8d65-488c-8077-dce558c29f60/Screenshot_7.png)

Deployment lifecycle events for debugging

Deployment Logs on EC2:

`ls /opt/codedeploy-agent/deployment-root/deployment-logs`

Streaming Deployment Logs to CloudWatch Logs: 

![Screenshot_8.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/feeb65bf-66cb-4631-a2c6-d55696234452/Screenshot_8.png)

### Creating a Deployment Group With Auto Scaling and Load Balancing

![Screenshot_9.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c631c0a-3bbb-484c-bb7a-83f665b5b312/Screenshot_9.png)

In-Place All-At-Once Deployment is not suitable for PROD env.

# CloudFormation

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

![Screenshot_11.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b9406052-5770-4c52-990f-09507c96e000/Screenshot_11.png)

Action Variables:

# ECS

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

![Screenshot_12.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3dd811b1-9d1d-41ff-8a27-0ddd4ac827fd/Screenshot_12.png)

![Screenshot_13.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc5efc5b-5f2d-4ca0-87b4-e73e48fdcd7f/Screenshot_13.png)

buildspec.yml

```yaml
env:
  secrets-manager:
    DOCKERHUB_TOKEN: dockerhub:token
    DOCKERHUB_USER: dockerhub:user
```

### Parameter Store

![Screenshot_14.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d97545d5-2f8f-4f8d-8248-d19377a20f86/Screenshot_14.png)

![Screenshot_15.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/23d74e2b-e815-4eda-8dca-e261eb89201a/Screenshot_15.png)

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

 

![Screenshot_17.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ec5bb2e2-9364-4f57-b2be-d66d33d8e4f2/Screenshot_17.png)

Create ECS Cluster

![Screenshot_16.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e6a231b7-6ee0-4fe7-98fd-a34e64dfb70c/Screenshot_16.png)

![Screenshot_18.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c8b4357f-c286-428f-b9d6-f0e4dd472ffe/Screenshot_18.png)

Create an ECS Service on Fargate for Rolling Deployments

![Screenshot_19.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7cc1ed37-119d-4463-8a5c-690a3343e448/Screenshot_19.png)

Create a new Security Group for ALB

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/default-custom-security-groups.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/default-custom-security-groups.html)

![Screenshot_20.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9414ea1f-0443-473c-ab8f-c531acd28fdc/Screenshot_20.png)

### Pipeline

Adding an ECS Standard Deploy Action to your Pipeline