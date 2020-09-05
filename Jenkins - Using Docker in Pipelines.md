# Jenkins - Using Docker in Pipelines

## Topics
- Containers as build agents
- Customizing the build container
- Using the Docker Pipeline plugin

## Containers as build agents - Docker Container Working as an Agent 

**Agent declaration:** Specifying the image that should be used to execute the steps declared in the Jenkinsfile
```groovy
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/core/sdk:3.1.101'
        }
    }
 }
```

## Customizing the build container

Docker file that is the full build of a project

```groovy
pipeline {
    agent {
        dockerfile {
            dir 'm4'
            filename 'Dockerfile.sdk'
        }
    }
```

The docker file declared is to build an image that will be used as the build agent, and not to be used to build the application;

PS: Do not declare the docker file of an application as the agent of a pipeline. The agent is where your aplication will be build and not the application build itself.

**JUST A CUSTOMIZED SDK**

### The docker file in the `m4`dir - The Docker file for the build agent
```docker
FROM mcr.microsoft.com/dotnet/core/sdk:3.1.1.101 as builder

ENV PS_MODULE=m4

```

## Using the Docker Pipeline plugin
Using docker to ship and run your app.

```groovy
def image

pipeline {
    agent any
    stages {
        stage('Build') {                 
            steps {
                script {
                    // (name of the docker image produced), (where to find docker file and pull down any images that get used in that docker file)
                    image = docker.build("sixeyed/pi:psod-pipelines", "--pull -f m4/Dockerfile m4")
                }                
            } 
        }   
        stage('Smoke Test') {
            steps {
                script {
                    // running a container from the image built from the docker file 
                    container = image.run()
                    container.stop()
              }
            }
        }
        stage('Push') {                 
            steps { 
                script {
                    // When this job completes, the image is going to be pushed in the dockerhub
                    withDockerRegistry([credentialsId: "docker-hub", url: "" ]) {        
                        image.push()
                    }     
                }
            }
        }
    }
}
```

### Docker file that ships the application
```docker
FROM mcr.microsoft.com/dotnet/core/sdk:3.1.101 AS builder

  

WORKDIR /src

COPY src/Pi.Math/Pi.Math.csproj ./Pi.Math/

COPY src/Pi.Runtime/Pi.Runtime.csproj ./Pi.Runtime/

COPY src/Pi.Web/Pi.Web.csproj ./Pi.Web/

  

WORKDIR /src/Pi.Web

RUN dotnet restore

  

COPY src/Pi.Math/ /src/Pi.Math/

COPY src/Pi.Runtime /src/Pi.Runtime/

COPY src/Pi.Web /src/Pi.Web/

RUN dotnet publish -c Release -o /out Pi.Web.csproj

  

# app image

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1.1

  

EXPOSE 80

ENTRYPOINT ["dotnet", "Pi.Web.dll"]

CMD ["-m", "console", "-dp", "6"]

  

WORKDIR /app

COPY --from=builder /out/ .
```
