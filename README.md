# Node and Nginx Docker Lab

This lab contains a small Express app that runs in its own Docker image and three independent Nginx containers serving different HTML folders.

## Project layout

```text
docker/                 Node.js app and Dockerfile
compose/                Compose file and three Nginx site folders
	folder1/index.html
	folder2/index.html
	folder3/index.html
```

## Prerequisites

- Docker Desktop installed and running (includes Docker Compose).
- A Docker Hub account for the optional image-publishing steps.

Run the commands below from this project directory.

## 1. Build the Node.js image

```powershell
docker build -t node-docker-sample:1.0.0 ./docker
```

Check that the image was created:

```powershell
docker image ls node-docker-sample
```

## 2. Run and test the Node.js container

```powershell
docker run --name node-docker-sample -d -p 3000:3000 node-docker-sample:1.0.0
```

Open <http://localhost:3000> or test from PowerShell:

```powershell
Invoke-RestMethod http://localhost:3000
```

Expected response:

```json
{"message":"Hello from inside the Docker container!"}
```

View logs and stop/remove the container:

```powershell
docker logs node-docker-sample
docker stop node-docker-sample
docker rm node-docker-sample
```

## 3. Publish the Node.js image to Docker Hub

Replace `YOUR_DOCKERHUB_USERNAME` with your Docker Hub username:

```powershell
docker login
docker tag node-docker-sample:1.0.0 YOUR_DOCKERHUB_USERNAME/node-docker-sample:1.0.0
docker push YOUR_DOCKERHUB_USERNAME/node-docker-sample:1.0.0
```

To build with the Docker Hub name from the start instead, use:

```powershell
docker build -t YOUR_DOCKERHUB_USERNAME/node-docker-sample:1.0.0 ./docker
```

## 4. Start the three Nginx containers

The Compose file uses the official `nginx:alpine` image and mounts each local folder read-only into a separate container. These services do not have custom Dockerfiles, so Compose does not need to build them.

```powershell
docker compose -f compose/compose.yaml pull
docker compose -f compose/compose.yaml up -d
```

Confirm all three services are running:

```powershell
docker compose -f compose/compose.yaml ps
```

Open each page:

- <http://localhost:3001> serves `compose/folder1/index.html`.
- <http://localhost:3002> serves `compose/folder2/index.html`.
- <http://localhost:3003> serves `compose/folder3/index.html`.

The folders are bind-mounted, so edits to an `index.html` file appear on refresh without rebuilding an image.

View logs or stop and remove the Compose containers:

```powershell
docker compose -f compose/compose.yaml logs -f
docker compose -f compose/compose.yaml down
```

Press `Ctrl+C` to stop following logs. `docker compose -f compose/compose.yaml down` does not delete the HTML files.
