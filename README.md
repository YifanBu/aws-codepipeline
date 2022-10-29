# DevOps with AWS CodePipeline, ECS, and CloudFormation
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

Creating an [appspec.yml](https://github.com/YifanBu/aws-codepipeline/blob/a875f98a89dba13c17a81f0aae07d93b62e5ac9a/my-angular-project/appspec.yml) File for Deployments to EC2 Instance

![Screenshot_7](https://user-images.githubusercontent.com/31915035/196175104-5d2e913c-ea2d-454e-a5f0-e78894bc025d.png)

Deployment lifecycle events for debugging

Deployment Logs on EC2:

`ls /opt/codedeploy-agent/deployment-root/deployment-logs`

Streaming Deployment Logs to CloudWatch Logs: 

![Screenshot_8](https://user-images.githubusercontent.com/31915035/196175140-5f1245fc-7860-4b7f-bb60-76018d2bee34.png)

### Creating a Deployment Group With Auto Scaling and Load Balancing

![Screenshot_9](https://user-images.githubusercontent.com/31915035/196175180-6a02a023-32f3-48da-9541-e4ec987027e1.png)

In-Place All-At-Once Deployment is not suitable for PROD env.

# 5. CloudFormation

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

# 6. ECS

### 1. Preparing AWS CodeBuild buildspec file to build Docker images and pushing them to Docker Hub

[my-angular-project-ecs/buildspec.yml](https://github.com/YifanBu/aws-codepipeline/blob/a875f98a89dba13c17a81f0aae07d93b62e5ac9a/my-angular-project-ecs/buildspec.yml)

### 2.Building a pipeline on AWS CodePipeline to build  and push your  Docker images  using  AWS CodeCommit  and  AWS CodeBuild

CodeCommit:
Create repository -> MyAngularDockerRepo

CodePipeline:
Pipeline name -> MyAngularDockerPipeline
Source -> CodeCommit, Repository name -> MyAngularDockerRepo, Branch name -> Master

CodeBuild:
Build provider -> CodeBuild
Project name -> MyDockerBuild
Build type -> Single build

### 3. Defining  environment  variables on CodePipeline and  CodeBuild  to store Docker  Hub credentials and other data
CodeBuild:
Environment variables -> DOCKERHUB_TOKEN, DOCKERHUB_USER

### 4. Using AWS Secrets Manager  with AWS CodeBuild  to store your  Docker Hub credentials as environment variables.

![Screenshot_12](https://user-images.githubusercontent.com/31915035/198856417-4043fdf8-b937-4b22-a2b5-579ac3f7c76b.png)


![Screenshot_13](https://user-images.githubusercontent.com/31915035/198856418-337fca3b-9517-4f72-b9a8-921fa60bcb59.png)


### 5. Using AWS System Parameter Store environment variables on AWS CodeBuild as a free alternative.
![Screenshot_14](https://user-images.githubusercontent.com/31915035/198856423-d4dcfb39-19b0-4b5b-8391-1aa106dc0adc.png)


![Screenshot_15](https://user-images.githubusercontent.com/31915035/198856426-73bc69ec-cd3a-4e12-bc0e-eaae8f87f5f2.png)



### 6. Pushing your Docker image built with AWS Pipeline and AWS CodeBuild to Amazon ECR
ECR:
Create repository -> Visibility settings: Private -> Repository name: my-angular-app

IAM: ecr-access-policy-for-codebuild.json

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

[codebuild environment variables](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html)

CodeBuild:
Environment variables -> AWS_ACCOUNT_ID

### 7. Create ECS task definition for the Docker images built by your pipeline

ECS:
Task definition configuration -> Task definition family: my-angular-task-definition
Container - 1 -> Container details: angular-app -> Image URI
Environment -> App environment: AWS Fargate, EC2 instances -> OS/Arc: Linux/x86_64

![Screenshot_17](https://user-images.githubusercontent.com/31915035/198856431-43a83f7a-a0a4-462c-86ed-8888ab13e69d.png)


### 8. Create an ECS cluster with Fargate and ASG capacity providers for hands-on examples

![Screenshot_16](https://user-images.githubusercontent.com/31915035/198856437-8f09d075-2c58-4693-b0f2-0b5c46ab26d5.png)

![Screenshot_18](https://user-images.githubusercontent.com/31915035/198856439-491c1c19-08f2-4382-b7d8-f6e18230b282.png)

### 9. Create ECS service on Fargate for rolling deployments

ECS:
Deployment options -> Deployment type: Rolling update -> Min running tasks: 100 -> Max running tasks: 200
Load balancing -> Application Load Balancer: my-rolling-service-alb -> Choose container to load balance: angular-app 80:80
Listener -> Create new listener -> Target group: Create new target group -> name: my-rolling-service-target-group -> Protocol: HTTP

![Screenshot_19](https://user-images.githubusercontent.com/31915035/198856442-03f2a89f-2651-4c4b-b9ed-4f4405bd960a.png)

`ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:{CODEBUILD_RESOLVED_SOURCE_VERSION:0:8}"`
### 12. Amazon ECR Public Gallery

Using Amazon ECR Public Gallery to pull official Docker images instead of Docker Hub while creating your Docker images with AWS CodeBuild.
### 13. Enable Automated Rollbacks on ECS Rolling Deployments
![Screenshot_20](https://user-images.githubusercontent.com/31915035/198856448-884fdbf1-921a-49be-a818-292395ef794b.png)



### 10. Add an ECS standard deploy action on AWS CodePipeline to perform rolling deployments on your ECS service.

CodePipeline:
Add stage: Deploy -> Action name: DeployToECS -> Action provider: Amazon ECS -> Input artifacts: BuildArtifact -> Cluster name: my-cluster -> Service name: my-rolling-service

### 11. Tag your Docker images built by your pipeline with Git commit IDs on AWS CodeCommit that created them.

`ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:{CODEBUILD_RESOLVED_SOURCE_VERSION:0:8}"`
### 12. Amazon ECR Public Gallery

Using Amazon ECR Public Gallery to pull official Docker images instead of Docker Hub while creating your Docker images with AWS CodeBuild.
### 13. Enable Automated Rollbacks on ECS Rolling Deployments

ECS -> Clusters -> Services:
Deployment configuration -> Deployment failure detection: Use the Amazon ECS deployment circuit breaker -> Rollback on failures

### 14. Create ECS service on ASG Capacity Providers for rolling deployments

ECS -> Clusters -> Services:
Compute configuration -> Capacity provider strategy -> Use custom -> Capacity provider: Auto Scaling Group
Deployment configuration -> Application type: Service
