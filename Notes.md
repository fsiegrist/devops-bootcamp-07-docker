## Notes on the videos
<br />

<details>
<summary>Video: 1 - What is a container</summary>
<br />

[Docker vs containerd vs cri-o](https://phoenixnap.com/kb/docker-vs-containerd-vs-cri-o) \
[The differences between Docker, containerd, CRI-O and runc](https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/)

For buildah, see links in notes on video 4 - Docker Architecture.

</details>

*****

<details>
<summary>Video: 3 - Docker vs Virtual Machine</summary>
<br />

Virtual machines consist of the Kernel layer and the application layer of the OS, whereas Docker containers just consist of the applications layer and use the Kernel layer of the host's OS.

</details>

*****

<details>
<summary>Video: 4 - Docker Architectue</summary>
<br />

[What is Buildah](https://www.redhat.com/en/topics/containers/what-is-buildah)
[buildah](https://buildah.io/)
[buildah - tutorials](https://github.com/containers/buildah/tree/main/docs/tutorials)

</details>

*****

<details>
<summary>Video: 5 - Main Docker Command</summary>
<br />

- `docker help <command>` = show help for a specific command
- `docker pull <image>` = download an image from a docker registry
- `docker images` or `docker image ls` = list the images available on the local machine
- `docker run <image> [command]` = start a container based on an image [and execute the command in the container]; pulls the image if it is not yet available on the local machine
  - `-d` = start the container in detached mode
  - `-p <host-port>:<container-port>` = bind a host port to a container port
  - `-n <name>` = give the container a name
- `docker ps` = shows information about all running containers; with option `-a` the stopped containers are listed too
- `docker stop <container-id|container-name>` = stops the container with the given id|name 
- `docker start <container-id|container-name>` = (re-)starts the container with the given id|name 

</details>

*****

<details>
<summary>Video: 6 - Debug Commands</summary>
<br />

- `docker logs <container-id|container-name>` = shows the log output of the specified container
  - `-f` = follow the log output (like `tail -f`)
- `docker exec -it <container-id|container-name> <command>` = execute the given command in the given continer, e.g. `docker exec -it <container-name> /bin/bash`

</details>

*****

<details>
<summary>Video: 8 - Developing with Docker</summary>
<br />

Download the required [mongo](https://hub.docker.com/_/mongo) and [mongo express](https://hub.docker.com/_/mongo-express) images:
```sh
docker pull mongo
docker pull mongo-express
```

Docker Network:\
Containers that are running in the same Docker network can talk to each other using just the container name (no ip address or port number is needed).

List available networks:
- `docker network ls`

Create our own network:
- `docker network create mongo-network`

Start the mongo database container attached to this network:
```sh
docker run -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --name mongodb \
  --network mongo-network \
  mongo
```

Start the mongo express container attached to the same network:
```sh
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  -e ME_CONFIG_MONGODB_SERVER=mongodb \
  --network mongo-network \
  --name mongo-express \
  mongo-express
```

Access the mongo-express page in your browser under `http://localhost:8081` and create a new database called `user-account`. Within this database create a collection called `users`.

Start the node application executing the following commands:
```sh
cd demo-projects/developing-with-docker/app
npm install
npm start
```

Access the application in your browser under `http://localhost:3000` and edit a user profile. You should see the data in the collection `users` of the mongo database `user-account`.

</details>

*****

<details>
<summary>Video: 9 - Docker Compose - Run multiple Docker containers</summary>
<br />

Docker Compose simplifies managing and running multiple Docker containers. The containers are specified in just one `docker-compose.yaml` file. To start the same containers as in video 8, the file looks like this:
```sh
version: '3.9'
services:              # the services section lists all containers
  mongodb:             # this is the name of the first container
    image: mongo       # this is the image the container is based on
    networks:
      - mongo-network  # attach to this network (could be omitted)
    ports:             # port mapping
      - 27017:27017
    environment:       # env variables
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:       # the name of the second container
    image: mongo-express
    networks:
      - mongo-network  # attach to this network (could be omitted)
    restart: always    # mongo-express depends on mongodb and has to restart
                       # until it can successfully connect to mongodb
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb

networks:
  mongo-network:       # define a custom network (could be omitted)
```
Docker Compose takes care of creating a common network for the containers (services) specified in the `docker-compose.yaml` file, so there's no need to define a custom network.

To start the containers, just execute `docker-compose up` or `docker-compose -f <file-name> up` if the name of the docker compose file is not `docker-compose.yaml` (which is the default). This will pull the images of the containers and start the containers as specified. If you want to start the containers in detached mode, add the `-d` option at the end of the `docker-compose` command.

To stop the containers (and the automatically created network), call `docker-compose down` or `docker-compose -f <file-name> down`.

</details>

*****

<details>
<summary>Video: 10 - Dockerfile - Build your own Docker Image</summary>
<br />

A Dockerfile is a blueprint for creating images.

Dockerfile Syntax:\
```sh
FROM <image>                          # specifies the base image for our new image
ENV <name>=<value>                    # define environment variables that will be set in the running container
RUN <Linux command>                   # execute any Linux command
COPY <host-source> <container-target> # copies the directory/file on the host into the image
CMD["cmd", "argument"]                # entry point (command that will be executed when a container is started based on this image)
```

To create an image based on a Dockerfile (the file must be called `Dockerfile`) in the current directory, execute the following command:
- `docker build -t <image-name>:<version> .`

If you start the application in a Docker container and need to access mongo db running in another Docker container, you cannot use `localhost` as the host name of the mongodb container. On Mac and Windows running Docker Desktop you can use `host.docker.internal` to access the Docker host, so `host.docker.internal:27017` is the URL to access the mongodb application running in its own container. On Linux you have to provide the following run flag when you start the container:
- `--add-host=host.docker.internal:host-gateway`

In `docker-compose.yaml` files add
```sh
extra_hosts:
  - "host.docker.internal:host-gateway"
```
to the specification of the container that wants to access the Docker host. Like this the `docker-compose.yaml` is portable for all plattforms.

</details>

*****

<details>
<summary>Video: 11 - Private Docker Repository</summary>
<br />

Prerequisites:\
- [Create an AWS account](https://aws.amazon.com/de/premiumsupport/knowledge-center/create-and-activate-aws-account/)
- [Install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
  - 1. Download the installer file using the following curl command. The -o option specifies the file name that the downloaded package is written to. In this example, the file is written to AWSCLIV2.pkg in the current folder:\
  `curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"`
  - 2. Run the standard macOS `installer` program, specifying the downloaded `.pkg` file as the source. Use the `-pkg` parameter to specify the name of the package to install, and the `-target /` parameter for which drive to install the package to. The files are installed to `/usr/local/aws-cli`, and a symlink is automatically created in `/usr/local/bin`. You must include sudo on the command to grant write permissions to those folders:\
  `sudo installer -pkg ./AWSCLIV2.pkg -target /` (requires Rosetta 2 to be installed)\
  After installation is complete, debug logs are written to `/var/log/install.log`.
  - 3. To verify that the shell can find and run the `aws` command in your `$PATH`, use the following commands:\
  `which aws`\
  `aws --version`
- Or install the AWS CLI using Homebrew (does not require Rosetta 2):\
`brew install awscli`
- [Configure AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)
  - Go to Services > IAM > Users and create a new user called `admin`, provide access to the AWS management console, select 'I want to create an IAM user' and click on 'Next', select 'Attach existing policies directly', select 'AdministratorAccess' and click on 'Next', check the data on the summary page and click 'Create User'. Download the csv-file containing the console login URL and the console password.
  - Logout as root user and open the console login URL (in the csv-file) and enter the username (`admin`) and password (also in the csv-file) to login as the new admin user. Change the password.
  - Go to Services > IAM > Users, select the admin user, open the tab 'Security login information' (Sicherheitsanmeldeinformationen), go to 'Access keys' and click on 'Create access keys', select 'Command Line Interface CLI' and click on 'Create access keys'. Download the csv-file containing the access key ID and the secret access key.
  - Execute `aws configure` and enter the access key ID, the secret access key, the region (e.g. `eu-central-1`) and the output format (e.g. `json`, or `yaml`).

Create an ECR Service:\
ECR (Elastic Container Registry) is the service on AWS you can use to create a Docker regristry.
- Login to the AWS account and open the [Management Console](https://eu-central-1.console.aws.amazon.com/console/home?region=eu-central-1#)
- Go to Services > Container > Elastic Container Registry
- Click on "Create a repository / Get Started"
- Select "Private" repository and give it a name (e.g. user-profile)\
Note that on AWS you use one ECR per Docker image (all versions of an image are stored in one ECR)
- Leave all other form fields unchanged an click on "Create repository"

Authenticate the Docker client for the private registry (`docker login`):\
`aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 369076538622.dkr.ecr.eu-central-1.amazonaws.com`

If the Docker image to be pushed to the registry is not yet created, go to the directory containing the Dockerfile and execute `docker build -t user-profile:1.0.0 .`.

Because we want to push this image to a private registry (not Docker Hub, where the default repository name `docker.io/library` is implicitly added, e.g. `docker pull nginx:latest` is interpreted as `docker pull docker.io/library/nginx:latest`), we have to tag (mark) it with the fully qualified image name (containing the repository name):\
`docker tag user-profile:1.0.0 369076538622.dkr.ecr.eu-central-1.amazonaws.com/user-profile:1.0.0`

Now we can push the image to the private registry:\
`docker push 369076538622.dkr.ecr.eu-central-1.amazonaws.com/user-profile:1.0.0`

</details>

*****

<details>
<summary>Video: 12 - Deploy Docker application on a server</summary>
<br />

To start the application using docker compose, we have to add a container with the application to the `docker-compose.yaml` file created in video 9:
```sh
version: '3.9'
services:
  user-profile:
    image: <private-repo-url>/user-profile:1.0.0
    networks: 
      - mongo-net
    ports:
      - 3000:3000
  mongodb:
    image: mongo
    networks: 
      - mongo-net
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    networks:
      - mongo-net
    restart: always # mongo-express depends on mongodb and has to restart
                    # until it can successfully connect to mongodb
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb

networks:
  mongo-net:
```

Now that all three containers are in the same Docker network, the application container can access the mongo-db using the service name. So instead of using `mongodb://admin:password@localhost:27017` (application running directly on the docker host, outside of a container) or `mongodb://admin:password@host.docker.internal:27017` (application running in a separate container not started by docker compose => different network), we can now use `mongodb://admin:password@mongodb`.

Add a file called `docker-compose.yaml` with the above content to the server where you want to start the application, go to the same directory and execute `docker-compose up -d`.

</details>

*****

<details>
<summary>Video: 13 - Docker Volumes - Persisting Data</summary>
<br />

There are three types of volumes:

- Host volumes\
`docker run -v /mounted/host/directory:/container/directory ...`\
You decide which folder on the host file system you mount into the container.
- Anonymous volumes\
`docker run -v /container/directory ...`\
For each container Docker automatically generates a folder that gets mounted.
- Named volumes\
`docker run -v name:/container/directory ...`\
For each container Docker automatically generates a folder that gets mounted. But you can reference that folder by the name you chose.

If you want to share data between containers, you can mount the same volume into more than one container.

In `docker-compose.yaml` files, named volumes are specified on the same level as services and referenced on container level:
```sh
version '3'

services:
  mongodb:
    image: mongo
    ports: ...
    ...
    volumes:
      - db-data:/data/db

volumes:
  db-data:
```

</details>

*****