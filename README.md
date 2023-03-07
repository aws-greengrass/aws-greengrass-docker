
# Running AWS IoT Greengrass V2 in a Docker Container  
## Overview  
AWS IoT Greengrass can run in a Docker container. You can use the Dockerfile in this package to build a container image that runs on `x86_64` platforms.   
  
This guide will show you how to:  
 * Build a Docker image from the Dockerfile for Amazon Linux 2 `x86_64`.  
 * Run an Amazon Linux Docker image containing AWS IoT Greengrass V2.
 * Use `docker-compose` to build and run AWS IoT Greengrass V2 in the Docker container.  
 * The Docker image supports Mac OSX, and Linux as Docker host platforms to run the Greengrass core software.  
  
## Prerequisites  
* Mac OSX or Linux host computer running Docker and Docker Compose (optional).  
 * The Docker installation (version 18.09 or later) can be found [here](https://docs.docker.com/install/ ).
 * The Docker Compose installation (version 1.22 or later) can be found [here]( https://docs.docker.com/compose/install/).
   Docker for Mac OS and Docker Toolbox include Compose, so those platforms don't need a separate Compose installation. Note: You must have version 1.22 or later because `init` support is required.  
  
## Running AWS IoT Greengrass in a Docker Container  
The following steps show how to build the Docker image from the Dockerfile and configure AWS IoT Greengrass to run in a Docker container.  
  
Note: If you have `docker-compose` installed, you can simply run `docker-compose up --build` from the unzipped directory for a quick build and basic installation, without device provisioning.   
  
### Step 1. Build the AWS IoT Greengrass Docker Image  
#### On Linux or Mac OSX  
  
**1.1** Download and decompress the `aws-greengrass-docker-<GREENGRASS_VERSION>` package.
  
**1.2** In a terminal, run the following commands in the location where you decompressed the `aws-greengrass-docker-<GREENGRASS_VERSION>` package.
```  
cd ~/Downloads/aws-greengrass-docker-<GREENGRASS_VERSION>
sudo docker build -t "x86_64/aws-iot-greengrass:<GREENGRASS_VERSION>" ./
```

**1.2.1** If you want to cross-build for a different platform, aarch64 for example.
```
cd ~/Downloads/aws-greengrass-docker-<GREENGRASS_VERSION>
sudo docker build --platform <platform> -t "aws-iot-greengrasss:<GREENGRASS_VERSION>" ./
```

`<platform>` can be `linux/aarch64` for example. See [Docker docs](https://docs.docker.com/build/building/multi-platform/#building-multi-platform-images) for more information.

Docker CLI v20.10.13 or greater is required to support this.
  
**1.2.2** If you have `docker-compose` installed, you can run the following commands instead:  
```
cd ~/Downloads/aws-greengrass-docker-<GREENGRASS_VERSION>
docker-compose -f docker-compose.yml build        
 ```
     
 * **Note**: If you want to provision the device upon startup for cloud deployments, you will need to add the following lines to your docker-compose file to mount your AWS credentials into the container to be picked up at `/root/.aws/credentials` . Ensure that the `:ro` suffix is present at the end of the command to ensure read-only access. (This will build image use long-term credentials from an IAM user) 
  
	```
	environment:  
	 - PROVISION=true
	volumes:  
	 - /path/to/credential/directory/:/root/.aws/:ro
	 ```
  
* **WARNING**: We strongly recommend removing this credential file from your host after the initial setup, as further authorization will be handled by the Greengrass Token Exchange Service using X.509 certificates. For more information, [see how Greengrass interacts with AWS services](
https://docs.aws.amazon.com/greengrass/v2/developerguide/interact-with-aws-services.html ) .
  
* If you would like to override any of the default configuration options or use your own config file to start Greengrass, specify those environment variables in the `environment` section as well. If you wish to use your own init config file, you must mount it to the directory you specify in the `INIT_CONFIG` environment variable, as well as mounting any extra files (e.g. custom certificates) you refer to in the init config file.  
Please see [the installer documentation](https://docs.aws.amazon.com/greengrass/v2/developerguide/configure-installer.html ) for configuration options and behavior. 

  
**1.3**  Verify that the Greengrass Docker image was built.  
```  
docker images  
REPOSITORY                          TAG                 	IMAGE ID            CREATED             SIZE
x86-64/aws-iot-greengrass           <GREENGRASS_VERSION>	3f152d6707c8        17 seconds ago      695MB
```  
  
### Step 2. Run the Docker Container  
#### On Linux or Mac OSX  

**2.1** In the terminal, run the following command to run the container in interactive mode in the foreground:  
  
```
docker run --init -it --name aws-iot-greengrass \  
x86_64/aws-iot-greengrass:<GREENGRASS_VERSION>
```
* Replace `-it` with `-d`  to run this container in the background in [detached mode](https://docs.docker.com/engine/reference/run/#detached-vs-foreground).

* **Note**: If you would like to provision your device for cloud deployments one of the following to retrieve credentials and make them available to the AWS IoT Greengrass Core software installer

	**2.1.1** **Use long-term credentials from an IAM user:** 
	Use the following lines in the above command to mount your AWS credentials into the container to be picked up at `/root/.aws/credentials`. Ensure that the `:ro` suffix is present at the end of the command to ensure read-only access. 

	```  
	-e PROVISION=true \  
	-v /path/to/credential/directory/:/root/.aws/:ro \  
	``` 

	* **WARNING**: We strongly recommend removing this credential file from your host after the initial setup, as further authorization will be handled by the Greengrass Token Exchange Service using X.509 certificates. For more information, [see how Greengrass interacts with AWS services](
https://docs.aws.amazon.com/greengrass/v2/developerguide/interact-with-aws-services.html ) . 

	**2.1.2** **(Recommended) Use temporary security credentials from an IAM role:** 
	Use the following lines in the above command to provide the access key ID, secret access key, and session token from an IAM role that you assume for the container. For more information about how to retrieve these credentials, see [Requesting temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_request.html) in the IAM User Guide.

	```  
	-e PROVISION=true \  
	-e AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE \  
	-e AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \  
	-e AWS_SESSION_TOKEN=AQoDYXdzEJr1K...o5OytwEXAMPLE= \  
	``` 
  
* If you would like to override any of the default configuration options or use your own config file to start Greengrass, specify those environment variables in the `environment` section as well. If you wish to use your own init config file, you must mount it to the directory you specify in the `INIT_CONFIG` environment variable, as well as mounting any extra files (e.g. custom certificates) you refer to in the init config file.  
Please see [the installer documentation](https://docs.aws.amazon.com/greengrass/v2/developerguide/configure-installer.html ) for configuration options and behavior. 
  
  
**2.1.1**  If you have `docker-compose` installed, you can run the following commands instead:  
```  
cd ~/Downloads/aws-greengrass-docker-<GREENGRASS_VERSION>
docker-compose -f docker-compose.yml up  
```  
  
**2.2** The output should look like this example:  
```  
Running Greengrass with the following options: -Droot=/greengrass/v2 -Dlog.store=FILE -Dlog.level= -jar /opt/greengrassv2/lib/Greengrass.jar --provision true --deploy-dev-tools false --aws-region us-east-1 --start false  
Installing Greengrass for the first time...  
...  
...  
Launching Nucleus...  
Launched Nucleus successfully.  
```  
  
### Debugging the Docker Container  
To debug issues with the container, you can persist the runtime logs or attach an interactive shell.  
  
#### Persist Greengrass Runtime Logs outside the Greengrass Docker Container  
You can run the AWS IoT Greengrass Docker container after bind-mounting the `/greengrass/v2/logs` directory to persist logs even after the container has exited or is removed. Alternatively, you can omit the `--rm` flag and use `docker cp` to copy the logs back from the container after it exits.  
  
  
#### Attach an Interactive Shell to the Greengrass Docker Container  
You can attach an interactive shell to a running AWS IoT Greengrass Docker container. This can help you to investigate the state of the Greengrass Docker container.  
Either use `docker attach` or run the following command in the terminal.  
```  
docker exec -it container-name sh -c "YOUR_COMMAND > /proc/1/fd/1"  
```  
  * This ensures that all output is redirected to PID 1 and will show up in the docker logs.
  
### Stopping the Docker Container  
To stop the AWS IoT Greengrass Docker Container, press Ctrl+C in your terminal (interactive mode) or run `docker stop` or `docker-compose stop` (detached mode). 
  
This action will send SIGTERM to the Greengrass process to gracefully shut down down the Greengrass process and all subprocesses that were started. The Docker container is initialized with the docker-init executable as process PID 1, which helps in removing any leftover zombie processes. For more information, see the [Docker documentation for init](https://docs.docker.com/engine/reference/run/#specify-an-init-process).

You may use `docker start` or `docker-compose start` to restart a stopped container.

### Removing the Docker Container

If you have not specified the `--rm` flag in your `docker run` command, your container will remain in the STOPPED state on your host. Use `docker rm` or `docker-compose down` to remove your container.
