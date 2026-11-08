# Implementing Microservices with Docker Hub, Amazon ECR and Amazon ECS

A screenshot-led capstone guide covering Docker image creation, Docker
Hub publishing, and deployment concepts on Amazon ECS.

## Project overview

This project demonstrates how to package an application into a Docker
image, publish the image to a registry, define its runtime configuration
in an ECS task definition, and run it through an ECS service.

### What the supplied implementation evidence shows

-   Ubuntu EC2 instance access and security-group configuration.
-   GitHub source-code clone.
-   Docker image build and local image inspection.
-   Docker image tagging and push to Docker Hub repository
    `siri3/node_repo`.
-   ECS cluster, task-definition and service screens.
-   Browser test showing an Express welcome page on port `3000`.



## Architecture

``` text
GitHub source
    |
    v
Ubuntu EC2 + Docker
    |
    v
Docker image
    |
    +----> Docker Hub (push shown in evidence)
    |
    +----> Amazon ECR (AWS-native registry extension)
                 |
                 v
          ECS task definition
                 |
                 v
            ECS service
                 |
                 v
          Running container
                 |
                 v
          Application endpoint
```

## Main technologies

-   Git and GitHub
-   Ubuntu on Amazon EC2
-   Docker Engine and Docker CLI
-   Docker Hub
-   Amazon ECR
-   Amazon ECS
-   AWS IAM, VPC and security groups
-   CloudWatch Logs/metrics (recommended for observability)

## Docker Hub commands

``` bash
# Clone the application repository
git clone https://github.com/<OWNER>/<REPOSITORY>.git
cd <REPOSITORY>

# Build an image from the Dockerfile in the current directory
docker build -t node-img:latest .

# Authenticate to Docker Hub
docker login --username siri3

# Tag the local image for the Docker Hub namespace/repository
docker tag node-img:latest siri3/node_repo:latest

# Push the image
docker push siri3/node_repo:latest

# Verify local image references
docker images
```

If your local image is named `react-img:latest`, use that as the source
image in the `docker tag` command.

## Amazon ECR extension

Use the same AWS Region as the ECS deployment. The EC2 host or build
environment must have the required IAM permissions.

``` bash
AWS_REGION=us-east-1
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
ECR_REPO=microservices/node-image

aws ecr create-repository \
  --repository-name "$ECR_REPO" \
  --region "$AWS_REGION" \
  --image-scanning-configuration scanOnPush=true

aws ecr get-login-password --region "$AWS_REGION" | \
  docker login --username AWS --password-stdin \
  "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"

ECR_URI="$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO"
docker tag node-img:latest "$ECR_URI:v1"
docker push "$ECR_URI:v1"
```

## ECS concepts

-   **Cluster:** logical grouping for ECS workloads.
-   **Task definition:** versioned blueprint containing the image and
    runtime configuration.
-   **Task:** running instance of a task definition.
-   **Service:** maintains the desired number of running tasks and
    manages deployments.
-   **Task execution role:** allows ECS infrastructure to pull images
    and send logs.
-   **Task role:** grants AWS permissions to application code inside the
    container.

## Security reminders

-   Do not allow all inbound traffic from `0.0.0.0/0`.
-   Restrict SSH to trusted IP addresses.
-   Expose only the application ports required.
-   Prefer private ECR repositories for internal deployments.
-   Use IAM roles instead of long-lived AWS access keys.
-   Never commit passwords, access tokens, `.pem` files or secret `.env`
    files.
-   Use HTTPS, health checks and a load balancer for production
    workloads.

## Documentation

See
[`docs/AWS_Microservices_DockerHub_ECR_ECS_Implementation_Guide.docx`](docs/AWS_Microservices_DockerHub_ECR_ECS_Implementation_Guide.docx)
for the full explanation, all original screenshots in source order,
command breakdowns, ECS deployment concepts, troubleshooting,
validation, and interview preparation.

## Important note

The screenshots contain account and network details and show an overly
broad inbound security-group rule. Review the document and repository
visibility before publishing publicly. The screenshots in the guide are
preserved unchanged as requested; use a private repository or create
approved redacted copies if needed.
