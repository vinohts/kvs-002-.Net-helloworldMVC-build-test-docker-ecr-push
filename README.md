🚀 Project 2 — HelloWorldMVC (.NET 8 Web App on AWS ECR + ECS)
💻 Local Environment
Tool	Version / Details
OS	Windows 11
PowerShell	7.0
.NET SDK	8.0
AWS CLI	Installed & configured (aws configure)
AWS Region	ap-south-1
Working Folder	D:\devops-handson\SimpleDotnetCICD2\HelloWorldMVC
🪜 STEP 1 — Create Project

In PowerShell (or VS Code terminal):

cd D:\
mkdir devops-handson
cd devops-handson
mkdir SimpleDotnetCICD2
cd SimpleDotnetCICD2

dotnet new mvc -n HelloWorldMVC
cd HelloWorldMVC


✅ A new folder and project HelloWorldMVC will be created at:
D:\devops-handson\SimpleDotnetCICD2\HelloWorldMVC

Test Locally:
dotnet run


Browse →
👉 http://localhost:5000

(Port may differ based on your machine)

✅ Ensure the app runs successfully.

🐳 STEP 2 — Create Dockerfile

Create a file named Dockerfile inside HelloWorldMVC folder:

# Step 1: Build Stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o out

# Step 2: Runtime Stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/out .
EXPOSE 8586
ENV ASPNETCORE_URLS=http://+:8586
ENTRYPOINT ["dotnet", "HelloWorldMVC.dll"]

⚙️ STEP 3 — Build and Run Docker Image Locally
Build Image:
docker build -t helloworldmvc .

Run Container:
docker run -p 3535:8586 helloworldmvc


Browse to →
👉 http://localhost:3535

✅ App works inside Docker.

🪣 STEP 4 — Create ECR Repository (in AWS)
aws ecr create-repository --repository-name helloworldmvc --region ap-south-1


ECR URI (example):
123456789012.dkr.ecr.ap-south-1.amazonaws.com/helloworldmvc

🔐 STEP 5 — Authenticate Docker with ECR
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com


✅ Output → Login Succeeded

📦 STEP 6 — Tag and Push Image to ECR
docker tag helloworldmvc:latest 123456789012.dkr.ecr.ap-south-1.amazonaws.com/helloworldmvc:latest
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/helloworldmvc:latest

Verify Push:
aws ecr list-images --repository-name helloworldmvc --region ap-south-1


✅ You’ll see "imageTag": "latest"

☁️ STEP 7 — Manually Create ECS Cluster and Deploy

You can reuse the same cluster or create a new one.

Option 1 — Use Existing Cluster

Use the existing cluster:
kvs-cluster (already created in Project 1)

Option 2 — Create New Cluster
aws ecs create-cluster --cluster-name kvs-cluster --region ap-south-1

Create Task Definition (Manual Steps)

ECS → Task Definitions → Create new → Fargate

Name → helloworldmvc-task

CPU → 0.25 vCPU

Memory → 0.5 GB

Add container:

Name: helloworldmvc-container

Image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/helloworldmvc:latest

Port: 8586

Click Create

Create ECS Service

ECS → Cluster → kvs-cluster

Click Create → Service

Launch Type → Fargate

Task Definition → helloworldmvc-task

Service Name → helloworldmvc-service

Desired Tasks → 1

Network:

Select default VPC

Select 2 public subnets

Enable Auto-assign Public IP

Click Create Service

✅ ECS deploys your container automatically.

Test Application

Open ECS → kvs-cluster → Tasks → Running Task

Scroll to Public IP

Open in browser:

http://<PublicIP>:8586


✅ You’ll see your HelloWorldMVC app hosted on ECS.

🧾 Summary Table
Component	Name / Example	Description
Project Folder	D:\devops-handson\SimpleDotnetCICD2\HelloWorldMVC	Local code
Docker Image	helloworldmvc:latest	Local image
ECR Repo	helloworldmvc	Stores built image
Cluster	kvs-cluster	ECS execution environment
Task Definition	helloworldmvc-task	Defines container settings
Service	helloworldmvc-service	Keeps container running
Public IP	From ECS Task	Used to access app

✅ Result:
Your .NET 8 MVC App is now containerized, pushed to ECR, and deployed on ECS — all done manually and cleanly.
