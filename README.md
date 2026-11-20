# Lab 10 - Software Deployment

This repository contains two English-language ASP.NET Core sample applications: MVC and Razor Pages. Each can be built as a Linux Docker image. The examples follow the lab handout while correcting its command syntax and container port mapping: current ASP.NET Core images listen on port 8080.

## Requirements

- .NET 10 SDK for local builds
- Docker Desktop (Windows 10/11) or Docker Engine (Ubuntu)
- Optional: Docker Hub account, GitHub repository, Ubuntu server, Azure subscription, or AWS account

## Exercise 1 - Add Docker support to MVC and Razor Pages

The projects are `src/MvcApp` and `src/RazorPagesApp`. Their Dockerfiles use a multi-stage build, publish Release output, and run as a small ASP.NET runtime image.

```powershell
dotnet build Lab10Deployment.sln
docker build -t myaspnetcoreapp .
docker run -d -p 8080:8080 --name myaspnetcoreappcontainer myaspnetcoreapp
```

Open `http://localhost:8080`. For the Razor Pages version:

```powershell
docker build -f src/RazorPagesApp/Dockerfile -t myrazorpagesapp .
docker run -d -p 8081:8080 --name razorpagescontainer myrazorpagesapp
```

Open `http://localhost:8081`. The host port can be changed while the container port remains 8080.

## Exercise 2 - Docker Desktop on Windows

Install and start Docker Desktop, then verify with `docker --version` and `docker info`. Run the build/run commands above from this repository root. Manage the container with:

```powershell
docker ps
docker logs myaspnetcoreappcontainer
docker stop myaspnetcoreappcontainer
docker rm myaspnetcoreappcontainer
```

Push/pull via Docker Hub (replace the username placeholder):

```powershell
docker login
docker tag myaspnetcoreapp YOUR_DOCKERHUB_USERNAME/lab10-mvc:latest
docker push YOUR_DOCKERHUB_USERNAME/lab10-mvc:latest
docker pull YOUR_DOCKERHUB_USERNAME/lab10-mvc:latest
docker run -d -p 8080:8080 --name myaspnetcoreappcontainer YOUR_DOCKERHUB_USERNAME/lab10-mvc:latest
```

## Exercise 3 - Ubuntu deployment

On Ubuntu, install and start Docker (`sudo apt-get update`, `sudo apt-get install -y docker.io`, `sudo systemctl enable --now docker`). Transfer a locally-built image with:

```powershell
docker save -o myaspnetcoreapp.tar myaspnetcoreapp
scp myaspnetcoreapp.tar USERNAME@UBUNTU_SERVER_IP:/tmp/myaspnetcoreapp.tar
```

Then on Ubuntu:

```bash
sudo docker load -i /tmp/myaspnetcoreapp.tar
sudo docker run -d --restart unless-stopped -p 8080:8080 --name myaspnetcoreappcontainer myaspnetcoreapp
sudo docker ps
```

Allow inbound TCP 8080 in the server firewall/security group and visit `http://UBUNTU_SERVER_IP:8080`. `compose.yaml` defines both apps; run `docker compose up -d --build` from the repository root and `docker compose down` to stop them.

## Exercise 4 - CI/CD with GitHub Actions

The workflow `.github/workflows/docker-image.yml` builds both apps on pushes and pull requests to `main`; pushes publish images to Docker Hub. Add repository Actions secrets `DOCKER_USERNAME` and `DOCKER_PASSWORD` (use a Docker Hub access token as the password), then push to `main`. Review the run under the repository Actions tab. Pull-request runs build but do not publish. Deployment to a remote server is a separate step requiring server credentials and is not enabled by default.

## Exercise 5 - Microsoft Azure

Create an Azure Container Registry (ACR), then set values for `REGISTRY_NAME` and `RESOURCE_GROUP`:

```bash
az login
az acr login --name REGISTRY_NAME
docker tag myaspnetcoreapp REGISTRY_NAME.azurecr.io/lab10-mvc:latest
docker push REGISTRY_NAME.azurecr.io/lab10-mvc:latest
```

Create an Azure App Service for Containers using the ACR image `lab10-mvc:latest`. Configure the container's HTTP port to `8080`, grant the App Service identity pull access to ACR, and verify the resulting HTTPS URL. Exact resource names, region, and URL depend on the student's subscription.

## Exercise 6 - AWS ECR and Elastic Beanstalk

Create an ECR repository named `lab10-mvc`, then substitute the account ID and region:

```bash
aws configure
aws ecr get-login-password --region REGION | docker login --username AWS --password-stdin AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com
docker tag myaspnetcoreapp AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lab10-mvc:latest
docker push AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lab10-mvc:latest
```

Use `deploy/aws/Dockerrun.aws.json` as the single-container Elastic Beanstalk configuration after replacing its image URI. Configure the environment's instance role to pull from ECR, allow HTTP traffic, and verify the environment URL. AWS account resources and costs depend on the selected region and configuration.

## Special exercise - Production website

The MVC and Razor Pages applications are deployable sample websites. Choose one hosting path above, publish the matching container, configure its HTTP port (8080), and record the live URL. Capture genuine screenshots of the running site, pipeline, and hosting console for the report. Do not claim a production deployment until the URL is live and checked.

## Submission checklist

- Replace placeholders with your student ID/name and actual registry/account details.
- Include both projects and any required `.sql` file in `StudentID_StudentName.zip` (no SQL database is configured in this starter).
- Produce `StudentID_StudentName.pdf` with bookmarks, GitHub link appendix, screenshots indexed by step, deployment URL, challenges and resolutions, and conclusion.
- Capture screenshots during your own execution; the report template does not fabricate screenshots or cloud deployment results.
