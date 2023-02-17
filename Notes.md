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
- `docker pull mongo`
- `docker pull mongo-express`

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