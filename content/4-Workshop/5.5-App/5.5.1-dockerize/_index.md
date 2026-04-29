---
title: "Package with Docker"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.5.1 </b> "
---

Before moving to Cloud, we need to package the .NET Core application into a Docker Image

1.  **Create Dockerfile:**
    At the root directory of the Solution, create a file named **Dockerfile** (no extension)

![docker1](/images/5-Workshop/5.5-App/docker1.png)

    ```dockerfile
    # STAGE 1: BUILD
    FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
    WORKDIR /src
    COPY ["MiniMarket.sln", "./"]
    COPY ["WebShop/WebShop.csproj", "WebShop/"]
    # ... (Copy other projects if any)
    RUN dotnet restore "MiniMarket.sln"
    COPY . .
    WORKDIR "/src/WebShop"
    RUN dotnet build "WebShop.csproj" -c Release -o /app/build
    RUN dotnet publish "WebShop.csproj" -c Release -o /app/publish

    # STAGE 2: RUNTIME
    FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
    COPY --from=build /app/publish .
    WORKDIR /app
    EXPOSE 8080
    ENTRYPOINT ["dotnet", "WebShop.dll"]
    ENV ASPNETCORE_URLS=http://+:8080
    ENV ASPNETCORE_ENVIRONMENT=Development
    ```

2.  **Create buildspec.yml:**
    Create file **buildspec.yml** to instruct AWS CodeBuild how to package and push to ECR

![docker2](/images/5-Workshop/5.5-App/docker2.png)

    ```yaml
    version: 0.2

    phases:
      pre_build:
        commands:
          - echo Logging in to Amazon ECR...
          # --- INFORMATION CONFIGURATION ---
          - AWS_DEFAULT_REGION=ap-southeast-1
          # Replace your Account ID in the line below:
          - AWS_ACCOUNT_ID= YOUR ACCOUNT ID
          - IMAGE_REPO_NAME=market-app
          - IMAGE_TAG=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
          - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
          # ---------------------------
          - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com

      build:
        commands:
          - echo Build started on `date`
          - echo Building the Docker image...
          - docker build -t $REPOSITORY_URI:latest .
          - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG

      post_build:
        commands:
          - echo Build completed on `date`
          - echo Pushing the Docker image...
          - docker push $REPOSITORY_URI:latest
          - docker push $REPOSITORY_URI:$IMAGE_TAG
          - echo Writing image definitions file...
          # Automatically create Dockerrun.aws.json configuration file for Beanstalk
          # Map Port 80 (Host) to 8080 (Container .NET)
          - printf '{"AWSEBDockerrunVersion":"1","Image":{"Name":"%s","Update":"true"},"Ports":[{"ContainerPort":8080,"HostPort":80}]}' "$REPOSITORY_URI:$IMAGE_TAG" > Dockerrun.aws.json
          - cat Dockerrun.aws.json

    artifacts:
      files:
        - Dockerrun.aws.json
    ```