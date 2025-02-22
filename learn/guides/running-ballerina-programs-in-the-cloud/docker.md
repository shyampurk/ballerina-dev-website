---
layout: ballerina-guides-left-nav-pages-swanlake
title: Docker
description: See how the Ballerina deployment in Docker works
keywords: ballerina, programming language, services, cloud, docker
permalink: /learn/user-guide/deployment/docker/
active: docker
intro: Docker helps to package applications and their dependencies in a binary image, which can run in various locations whether on-premise, in a public cloud, or in a private cloud. 
redirect_from:
  - /learn/deployment/docker
  - /learn/deploying-ballerina-programs-in-the-cloud
  - /learn/deploying-ballerina-programs-in-the-cloud/
  - /learn/deployment/
  - /swan-lake/learn/deployment/docker/
  - /swan-lake/learn/deployment/docker
  - /learn/deployment/docker/
  - /learn/deployment/docker
  - /learn/user-guide/deployment/docker
---

To create a Docker image, you have to create a Dockerfile by choosing a suitable base image, bundling all dependencies, copying the application binary, and setting the execution command with proper permissions. To create optimized images, you have to follow a set of [best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/). Otherwise, the image that is built will be large in size, less secure, and have many other shortcomings.

The Ballerina compiler is capable of creating optimized Docker images out of the application source code. This guide includes step-by-step instructions on different use cases and executing the corresponding sample source code. 

## Enabling Docker Support

Docker support is inbuilt and comes by default in any Ballerina distribution. You can enable Docker artifact generation by adding annotations to your Ballerina code. Follow the steps below to enable Docker support.

1. Import the Docker extension module in your Ballerina code (e.g., `import ballerina/docker;`).

2. Add the relevant annotations within the code (e.g., `@docker:Config {}`).

3. Build the Ballerina source file/project (e.g., `ballerina build source.bal`).

## Use Cases

### Running a Ballerina Service in a Docker Container

This use case shows how to run a Ballerina service in a Docker container. The sample below demonstrates running a simple Ballerina hello world service in a Docker container. 

#### Setting Up the Prerequisites

You need a machine with [Docker](https://docs.docker.com/get-docker/) installed.

#### Sample Source Code 

`hello_world_docker.bal`
```ballerina 

import ballerina/http;
import ballerina/docker;

@docker:Config {}
service http:Service /hello on new http:Listener(9090) {
    resource function get sayHello(http:Caller caller) {
        caller->respond("Hello World!");
    }
}

```

#### Steps to Run

1. Compile the `hello_world_docker.bal` file.

    ```bash
    > ballerina build hello_world_docker.bal 
    Compiling source
      hello_world_docker.bal

    Generating executables
      hello_world_docker.jar

    Generating docker artifacts...
      @docker 		 - complete 2/2 

      Run the following command to start a Docker container:
      docker run -d -p 9090:9090 hello_world_docker:latest
    ```

    The artifacts files below will be generated with the build process.

    ```bash
    > tree
    .
    ├── docker
    │   └── Dockerfile
    ├── hello_world_docker.bal
    └── hello_world_docker.jar

    1 directory, 3 files

    ```

    The build process automatically generates a Dockerfile with the following content:

    ```
    # Auto Generated Dockerfile
    FROM ballerina/jre11:v1

    LABEL maintainer="dev@ballerina.io"
   
    WORKDIR /home/ballerina
   
    COPY ballerina-lang.float-1.0.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.__internal-0.1.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.int-1.1.0.jar /home/ballerina/jars/ 
    COPY hello_world_docker.jar /home/ballerina/jars/ 
    ...

    RUN addgroup troupe \
    && adduser -S -s /bin/bash -g 'ballerina' -G troupe -D ballerina \
    && apk add --update --no-cache bash \
    && chown -R ballerina:troupe /usr/bin/java \
    && rm -rf /var/cache/apk/*

    COPY hello_world_docker.jar /home/ballerina

    USER ballerina

    CMD java -Xdiag -cp "hello_world_docker.jar:jars/*" '$_init'

    ```


2. Verify that the Docker image is created.

    ```bash
    > docker images
    REPOSITORY          TAG         IMAGE ID            CREATED             SIZE
    hello_world_docker  latest      e48123737a65        7 minutes ago       134MB

    ```

    > Since the annotation is not configured to have a custom Docker image name and tag, the build process will create a Docker image with the default values: the file name of the generated .jar file with the `latest` tag (e.g., `hello_world_docker:latest`).

3. Run the Docker image as a container (use the below command printed in step 1).

    ```bash
    > docker run -d -p 9090:9090 hello_world_docker:latest
    32461676d3c22848088390483a414e5b1d11a7a73c2296eccb18e6c9f27c41c0
    ```


4. Verify that the Docker container is running.

    ```bash
    > docker ps
    CONTAINER ID  IMAGE    		 COMMAND    	      CREATED              STATUS            PORTS     NAMES
    32461676d3c2  hello_world_docker:latest  "/bin/sh -c 'java -j…"   About a minute ago   Up About a minute 0.0.0.0:9090->9090/tcp    lucid_turing
    ```


5. Access the hello world service with the cURL command.

    ```bash
    > curl http://localhost:9090/hello/sayHello           
    Hello World!
    ```


6. Clean up the used artifacts.

    ```bash
    Stop / kill running docker container
    > docker kill 32461676d3c2
    32461676d3c2

    Remove docker container files
    > docker em 32461676d3c2

    Remove docker image
    > docker rmi e48123737a65
  ```

### Creating a Custom Ballerina Docker Image and Pushing it Automatically to the Docker Registry

This use case shows how to run a simple Ballerina hello world service in a Docker container with annotation configurations, which can be used to create a Docker image with a custom image-name and tag, and then automatically push the created image to the Docker registry.

#### Setting Up the Prerequisites

- A machine with [Docker](https://docs.docker.com/get-docker/) installed
- A [Docker Hub](https://hub.docker.com/) account

#### Sample Source Code

`custom_docker_name.bal`

```ballerina
import ballerina/http;
import ballerina/docker;
 
@docker:Config {
   push: true,
   registry: "index.docker.io/$env{DOCKER_USERNAME}",
   name: "helloworld",
   tag: "v1.0.0",
   username: "$env{DOCKER_USERNAME}",
   password: "$env{DOCKER_PASSWORD}"
}
service http:Service /hello on new http:Listener(9090) {
    resource function get sayHello(http:Caller caller) {
        caller->respond("Hello World!");
    }
}
```

In this sample, the following properties are set in the `@docker:Config` annotation.

- `push`		  : enable pushing the Docker image to the registry 
- `registry`	: Docker registry URL
- `name`		  : name of the Docker image
- `tag`		    : Docker image tag (version) 
- `username`	: username of the Docker registry
- `password`	: password of the Docker registry

> The `$env` variable is used to read the environment variable values from the system.

#### Steps to Run

1. Export the username and password of Docker (registry).

    ```bash
    export DOCKER_USERNAME=<username>
    export DOCKER_PASSWORD=<password>
    ```

2. Compile the `custom_docker_name.bal` file.

    ```ballerina

    > ballerina build custom_docker_name.bal
    Compiling source
      custom_docker_name.bal

    Generating executables
      custom_docker_name.jar

    Generating docker artifacts...
      @docker 		 - complete 3/3 

      Run the following command to start a Docker container:
      docker run -d -p 9090:9090 index.docker.io/lakwarus/helloworld:v1.0.0

    ```

3. Verify that the Docker image is created.

    ```bash
    REPOSITORY          TAG      IMAGE ID            CREATED             SIZE
    lakwarus/helloworld v1.0.0   7e76efdd33e4        20 minutes ago      134MB
    ```

4. Log into [Docker Hub](https://hub.docker.com/) and verify that the image is pushed.

    ![Docker Hub](/learn/images/docker-hub.png)

### Running a Ballerina HTTPS Service in a Docker Container

An HTTP endpoint can be configured to communicate through HTTPS as well. To secure an endpoint using HTTPS, it needs to be configured with a keystore, a certificate, and a private key for the endpoint. When running this HTTPS service as a Docker container, it is required to copy the keystore with the custom certificate that is used to configure the HTTPS endpoint. 

#### Setting Up the Prerequisites

You need a machine with [Docker](https://docs.docker.com/get-docker/) installed.

#### Sample Source Code

The sample below uses a separate listener endpoint and it is configured with a custom keystore. In addition to the `@docker:Config` annotation, the `@docker:Expose` annotation is used with the listener endpoint object, which helps to expose the correct service ports when creating the Docker container. 

`https_service_in_docker.bal`

```ballerina

import ballerina/http;
import ballerina/docker;
 
@docker:Expose {}
listener http:Listener helloWorldEP = new(9095, {
   secureSocket: {
       keyStore: {
           path: "./ballerinaKeystore.p12",
           password: "ballerina"
       }
   }
});
 
@docker:Config {
   name: "https-helloworld"
}
service http:Service /helloWorld on helloWorldEP {
    resource function get sayHello(http:Caller caller) {
        caller->respond("Hello World!");
    }
}
```

#### Steps to Run

1. Compile the `https_service_in_docker.bal` file.

    ```ballerina
    > ballerina build https_service_in_docker.bal 
    Compiling source
      https_service_in_docker.bal

    Generating executables
      https_service_in_docker.jar

    Generating docker artifacts...
      @docker 		 - complete 2/2 

      Run the following command to start a Docker container:
      docker run -d -p 9095:9095 https-helloworld:latest

    ```
    The artifact files below will be generated with the build process.

    ```bash
    > tree
    .
    ├── ballerinaKeystore.p12
    ├── docker
    │   ├── Dockerfile
    │   └── ballerinaKeystore.p12
    ├── https_service_in_docker.bal
    └── https_service_in_docker.jar

    1 directory, 5 files

    ```

    > **Note:** It has copied the correct keystore used in the source code into the Docker folder and generated the Dockerfile with the content below.

    ```
    # Auto Generated Dockerfile
    FROM ballerina/jre11:v1

    LABEL maintainer="dev@ballerina.io"

    WORKDIR /home/ballerina
   
    COPY ballerina-lang.float-1.0.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.__internal-0.1.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.int-1.1.0.jar /home/ballerina/jars/ 
    COPY https_service_in_docker.jar /home/ballerina/jars/ 
    ...
    COPY https_service_in_docker.jar /home/ballerina
    COPY ballerinaKeystore.p12 ./ballerinaKeystore.p12

    RUN addgroup troupe \
    && adduser -S -s /bin/bash -g 'ballerina' -G troupe -D ballerina \
    && apk add --update --no-cache bash \
    && chown -R ballerina:troupe /usr/bin/java \
    && rm -rf /var/cache/apk/*

    EXPOSE  9095
    USER ballerina
    
    CMD java -Xdiag -cp "https_service_in_docker.jar:jars/*" '$_init'

    ```

    > The Ballerina compiler automatically adds a line to the Dockerfile to copy the required keystore into the Docker image.

2. Verify that the Docker image is created.

    ```bash
    > docker images
    REPOSITORY          TAG        IMAGE ID            CREATED             SIZE
    https-helloworld    latest     ed9bff9fabd7        15 minutes ago      134MB
    ```

3. Run the Docker image as a container (use the command below printed in step 1).

    ```bash
    > docker run -d -p 9095:9095 https-helloworld:latest
    25dfb84f1c9f3459baf7a4791f9de3cae260d9963e580802253621919d0bd2fb
    ```

4. Verify that the Docker container is running.

    ```bash
    > docker ps
    CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                    NAMES
    25dfb84f1c9f        https-helloworld:latest   "/bin/sh -c 'java -j…"   45 seconds ago      Up 45 seconds       0.0.0.0:9095->9095/tcp   charming_visvesvaraya
    ```

5. Access the hello world service with the cURL command.

    ```bash
    curl -k https://localhost:9095/hello/sayHello
    Hello World!
    ```
    > **Note:** The cURL command is used with the -k option because self-signed certificates are used in the keystore.

6. Clean up the used artifacts.

    ```bash
    > docker kill 25dfb84f1c9f
    25dfb84f1c9f

    > docker rm 25dfb84f1c9f
    25dfb84f1c9f

    > docker rmi ed9bff9fabd7
    Untagged: https-helloworld:latest
    ```

### Copying Additional Files to the Ballerina Docker Image

In some use cases, you need to process files in your applications. When these applications are run in a Docker container, you might need to copy these additional files into the image or mount them as external volume. This use case shows how to copy additional files into the Docker image while compiling the Ballerina source code.

#### Setting Up the Prerequisites

You need a machine with [Docker](https://docs.docker.com/get-docker/) installed.

#### Sample Source Code

`copy_file.bal`

```ballerina
import ballerina/http;
import ballerina/io;
import ballerina/docker;
 
@docker:Expose {}
listener http:Listener helloEP = new(9090);
 
@docker:Config {}
@docker:CopyFiles {
   files: [
       { sourceFile: "./name.txt", target: "/home/ballerina/name.txt" }
   ]
}
service /hello on helloEP {
 
   resource function get greet(http:Caller caller) returns error? {
       http:Response response = new;
       string payload = readFile("./name.txt");
       response.setTextPayload("Hello " + <@untainted> payload + "\n");
       check caller->respond(response);
   }
}
 
function readFile(string filePath) returns  string {
   io:ReadableByteChannel bchannel = checkpanic io:openReadableFile(filePath);
   io:ReadableCharacterChannel cChannel = new io:ReadableCharacterChannel(bchannel, "UTF-8");
 
   var readOutput = cChannel.read(50);
   if (readOutput is string) {
       return <@untainted> readOutput;
   } else {
       return "Error: Unable to read file";
   }
}
```

`name.txt`

This sample sends a greeting to the caller by getting the name from a text file. When this is run in a container, you need to copy the `name.txt` file into the Docker image. The `@docker:CopyFiles` annotation is used for this and you can give multiple files by following the syntax below.

```ballerina
@docker:CopyFiles {
   files: [
       { sourceFile: "./name.txt", target: "/home/ballerina/name.txt" },
	{ sourceFile: "./abc.txt", target: "/home/ballerina/abc.txt" }
   ]
}
```

>**Note:** In addition to the above, if you want to copy driver files such as JDBC, you can create a Ballerina project, add the following entries to its `Ballerina.toml` file, and change the path to the JDBC driver appropriately.

`Ballerina.toml`

```
[project]
org-name= "sample"
version= "0.1.0"

[platform]
target = "java8"

    [[platform.libraries]]
    artafactId = "mysql-connector-java"
    version = "8.0.17"
    path = "/path/to/mysql-connector-java-8.0.17.jar"
```

#### Steps to Run

1. Create a `name.txt` file in the same directory in which the `copy_file.bal` file resides.

    ```bash
    > echo Ballerina > ./name.txt
    ```

2. Compile the `copy_file.bal` file.

    ```ballerina
    > ballerina build copy_file.bal 
    Compiling source
      copy_file.bal

    Generating executables
      copy_file.jar

    Generating docker artifacts...
      @docker 		 - complete 2/2 

      Run the following command to start a Docker container:
      docker run -d -p 9090:9090 copy_file:latest
    ```

    The artifacts files below will be generated with the build process.

    ```bash
    > tree
    .
    ├── copy_file.bal
    ├── copy_file.jar
    ├── docker
    │   ├── Dockerfile
    │   └── name.txt
    └── name.txt

    1 directory, 5 files
    ```

    > **Note:** The generated Dockerfile includes the COPY command to copy files that are defined by the `@docker:CopyFiles` annotation.

    ```
    # Auto Generated Dockerfile
    FROM ballerina/jre11:v1

    LABEL maintainer="dev@ballerina.io"

    WORKDIR /home/ballerina
   
    COPY ballerina-lang.float-1.0.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.__internal-0.1.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.int-1.1.0.jar /home/ballerina/jars/ 
    COPY copy_file.jar /home/ballerina/jars/ 
    ...
    COPY copy_file.jar /home/ballerina
    COPY name.txt /home/ballerina/name.txt

    RUN addgroup troupe \
    && adduser -S -s /bin/bash -g 'ballerina' -G troupe -D ballerina \
    && apk add --update --no-cache bash \
    && chown -R ballerina:troupe /usr/bin/java \
    && rm -rf /var/cache/apk/*

    EXPOSE  9090
    USER ballerina
   
    CMD java -Xdiag -cp "copy_file.jar:jars/*" '$_init'
    ```

3. Verify that the Docker image is created.

    ```bash
    > docker images
    REPOSITORY          TAG        IMAGE ID            CREATED             SIZE
    copy_file           latest     cc198c0af86e        6 minutes ago       134MB
    ```

4. Run the Docker image as a container (use the command below printed in step 1).

    ```bash
    > docker run -d -p 9090:9090 copy_file:latest
    d9f72d8ef6b2f27df099cc3e57676aeb3110c330004b5539e557ec49cf50878a
    ```

5. Verify that the Docker container is running.

    ```bash
    > docker ps 
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    d9f72d8ef6b2        copy_file:latest    "/bin/sh -c 'java -j…"   15 seconds ago      Up 15 seconds       0.0.0.0:9090->9090/tcp   inspiring_joliot
    ```

6. Access the hello world service with the cURL command below.

    ```bash
    > curl http://localhost:9090/hello/greet
    Hello Ballerina
    ```

7. Clean up the used artifacts.

    ```bash
    > docker kill d9f72d8ef6b2
    d9f72d8ef6b2

    > docker rm d9f72d8ef6b2
    d9f72d8ef6b2

    > docker rmi cc198c0af86e
    ```

### Using a Custom Base Image to Build Ballerina Docker Images

Ballerina ships a base image (e.g., `ballerina/jre11:v1`) with some security hardening. It is used to build Docker images with the user's application code. However, sometimes, you might need to use your own Docker base image depending on your company policies or any additional requirements. This use case shows how to use a custom Docker base image to build Ballerina Docker images with the application code. 

#### Setting Up the Prerequisites

You need a machine with [Docker](https://docs.docker.com/get-docker/) installed.

#### Sample Source Code

`base_image.bal`

```ballerina
import ballerina/http;
import ballerina/docker;
 
@docker:Config {
   name: "helloworld_custom_baseimage",
   baseImage: "openjdk:11-jre-slim"
}
service /hello on new http:Listener(9090){
 
  resource function get sayHello(http:Caller caller) returns error? {
      check caller->respond("Hello World!");
  }
}
```

> **Note:** This sample uses `openjdk:11-jre-slim` as the custom Docker image by using the `baseImage` property in the `@docker:Config` annotation.

#### Steps to Run

1.  Compile the `base_image.bal` file.

    ```ballerina
    > ballerina build base_image.bal 
    Compiling source
      base_image.bal

    Generating executables
      base_image.jar

    Generating docker artifacts...
      @docker 		 - complete 2/2 

      Run the following command to start a Docker container:
      docker run -d -p 9090:9090 helloworld_custom_baseimage:latest
    ```

    The artifact files below will be generated with the build process.

    ```bash
    tree
    .
    ├── base_image.bal
    ├── base_image.jar
    └── docker
        └── Dockerfile

    1 directory, 3 files
    ```

    The Dockerfile will be generated as follows.

    > **Note:**  The FROM section of the Dockerfile has the base image, which is defined by the `@docker:Config` annotation.

    ```
    # Auto Generated Dockerfile
    FROM openjdk:11-jre-slim

    LABEL maintainer="dev@ballerina.io"
     
    WORKDIR /home/ballerina
    
    COPY ballerina-lang.float-1.0.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.__internal-0.1.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.int-1.1.0.jar /home/ballerina/jars/ 
    COPY base_image.jar /home/ballerina/jars/ 
    ...
    COPY base_image.jar /home/ballerina

    EXPOSE  9090
    CMD java -Xdiag -cp "base_image.jar:jars/*" '$_init'
    ```

2. Verify that the Docker image is created.

    ```bash
    > docker images
    REPOSITORY     			TAG  		IMAGE ID            CREATED             SIZE
    helloworld_custom_baseimage      latest    	4468889e4ed0        11 minutes ago      109MB
    ```

3. Run the Docker image as a container (use the command below printed in step 1).

    ```bash
    > docker run -d -p 9090:9090 helloworld_custom_baseimage:latest
    0c31cfaf483493988ee9ace73f6bcf9188a80d33ac3640052265c316058ec55a

    ```

4. Verify that the Docker container is running.

    ```bash
    > docker ps
    CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS                    NAMES
    0c31cfaf4834        helloworld_custom_baseimage:latest   "/bin/sh -c 'java -j…"   17 seconds ago      Up 16 seconds       0.0.0.0:9090->9090/tcp   charming_hypatia

    ```

5. Access the hello world service with the cURL command below.

    ```bash
    > curl http://localhost:9090/hello/sayHello
    Hello World!
    ```

6. Clean up the used artifacts.

    ```bash
    > docker kill 0c31cfaf4834
    0c31cfaf4834

    > docker rm 0c31cfaf4834
    0c31cfaf4834

    > docker rmi 4468889e4ed0
    Untagged: helloworld_custom_baseimage:latest
    ```

### Overriding the CMD of the Generated Ballerina Dockerfile

The Ballerina Docker image builder uses the generic CMD command to execute the created JAR file. However, sometimes, you might need to override the default CMD command. This use case shows how to override the default CMD command.

### Setting Up the Prerequisites

You need a machine with [Docker](https://docs.docker.com/get-docker/) installed.

#### Sample Source Code

`docker_cmd.bal`

```ballerina

import ballerina/http;
import ballerina/docker;
 
@docker:Config {
   name: "custome_cmd",
   cmd:    cmd: "CMD java -Xdiag -cp \"${APP}:jars/*\" '$_init' --b7a.http.accesslog.console=true"
}
service /hello on new http:Listener(9090){
 
  resource function get sayHello(http:Caller caller) returns error? {
      check caller->respond("Hello World!");
  }
}
```

This sample enables HTTP trace logs by overriding the CMD value of the generated Dockerfile. The cmd field will be as `CMD java -jar ${APP} --b7a.http.accesslog.console=true` in the `@docker:Config{}` annotation.

### Steps to Run

1. Compile the `base_image.bal` file.

    ```ballerina
    > ballerina build docker_cmd.bal
    Compiling source
      docker_cmd.bal

    Generating executables
      docker_cmd.jar

    Generating docker artifacts...
      @docker 		 - complete 2/2 

      Run the following command to start a Docker container:
      docker run -d -p 9090:9090 custome_cmd:latest
    ```

    The artifact files below will be generated with the build process.

    > **Note:** The CMD line will be updated with the custom command defined in the annotation.

    ```bash
    > tree
    .
    ├── docker
    │   └── Dockerfile
    ├── docker_cmd.bal
    └── docker_cmd.jar

    1 directory, 3 files
    ```

    The Dockerfile will be generated as follows.

    ```
    # Auto Generated Dockerfile
    FROM ballerina/jre11:v1

    LABEL maintainer="dev@ballerina.io"

    WORKDIR /home/ballerina
   
    COPY ballerina-lang.float-1.0.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.__internal-0.1.0.jar /home/ballerina/jars/ 
    COPY ballerina-lang.int-1.1.0.jar /home/ballerina/jars/ 
    COPY docker_cmd.jar /home/ballerina/jars/ 
    ...
    COPY docker_cmd.jar /home/ballerina 
   
    RUN addgroup troupe \
    && adduser -S -s /bin/bash -g 'ballerina' -G troupe -D ballerina \
    && apk add --update --no-cache bash \
    && chown -R ballerina:troupe /usr/bin/java \
    && rm -rf /var/cache/apk/*

    COPY docker_cmd.jar /home/ballerina

    EXPOSE  9090
    USER ballerina
    
    CMD java -Xdiag -cp "docker_cmd.jar:jars/*" '$_init' --b7a.http.accesslog.console=true
    ```

2. Verify that the Docker image is created.

    ```bash
    > docker images
    REPOSITORY          TAG        IMAGE ID            CREATED             SIZE
    custome_cmd         latest     08611185ed10        10 minutes ago      134MB
    ```

3. Run the Docker image as a container (use the command below printed in step 1).

    ```bash
    > docker run -d -p 9090:9090 custome_cmd:latest
    0f66739200dc07d667229f49b66beaf8135e0f9bb000feea23aaa5f9a7cc1d18
    ```

4.  Verify that the Docker container is running.

    ```bash
    > docker ps
    CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                    NAMES
    0f66739200dc        custome_cmd:latest   "/bin/sh -c 'java -j…"   55 seconds ago      Up 55 seconds       0.0.0.0:9090->9090/tcp   pensive_jackson
    ```

5. Access the hello world service with the cURL command below.

    ```bash
    > curl http://localhost:9090/hello/sayHello
    Hello World!
    ```

6. View the HTTP access logs.

    ```bash
    > docker logs 0f66739200dc
    ballerina: HTTP access log enabled
    [ballerina/http] started HTTP/WS listener 0.0.0.0:9090
    172.17.0.1 - - [14/Jun/2020:04:38:22 +0000] "GET /hello/sayHello HTTP/1.1" 200 12 "-" "curl/7.64.1" 
    172.17.0.1 - - [14/Jun/2020:04:38:23 +0000] "GET /hello/sayHello HTTP/1.1" 200 12 "-" "curl/7.64.1"
    ```

7. Clean up the created artifacts.

    ```bash
    > docker kill 0f66739200dc
    0f66739200dc

    > docker rm 0f66739200dc
    0f66739200dc

    > docker rmi 08611185ed10
    Untagged: custome_cmd:latest
    ```

## Troubleshooting

Mini-kube users should configure the annotations below in every sample with valid values.

```ballerina
@docker:Config {
   		dockerHost: "tcp://192.168.99.100:2376",
   		dockerCertPath: "/Users/lakwarus/.minikube/certs"
}
```
