---
title: "CI/CD with GitLab Runner"
date: 2023-11-22
category: devops
tags: [gitlab, ci-cd, docker, deployment]
summary: "Setting up GitLab Runner with Docker for automated CI/CD pipelines"
---

# CI/CD with GitLab Runner

## GitLab Runner Setup

### Download binary
```bash
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permission to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab Runner user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as a service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

### Register runner

Tag for runner will create for CI/CD pipeline also.

```bash
gitlab-runner register --url https://gitlab.com --token ${token}
```

### Verify Runner Already run

```bash
gitlab-runner run
```

## Setup rootless user

Run this on root user

```sh
visudo
```

Not recommended but to allow all apps, better specify only with apps that need run without rootless password. Add this to bottom of visudo config:

```sh
gitlab-runner ALL=(ALL:ALL) NOPASSWD: ALL
```

## Docker install

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Create the docker group.

```bash
sudo groupadd docker
```

Add your user to the docker group.

```bash
sudo usermod -aG docker $USER
```

Log out and log back in so that your group membership is re-evaluated.

## Add Nginx Proxy Manager

Deploy nginx proxy manager

```yaml
version: '3.8'
services:
  nginx:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    extra_hosts:
      - "host.docker.internal:host-gateway"

networks:
  default:
    name: nginx-network
    external: true
```

## Add Service

All new services should have same network configuration.

### Client side Frontend App

Dockerfile FE React App Example

```Dockerfile
# Stage 1: Build the React app
FROM node:lts as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Create the production image
FROM nginx:latest
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Docker Compose App example:

```yaml
version: '3.8'
services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    extra_hosts:
      - "host.docker.internal:host-gateway"

networks:
  default:
    name: nginx-network
    external: true
```

Gitlab pipeline example:

```yaml
deploy:
  stage: deploy
  tags:
    - prod
  environment:
    name: production
  script:
    - docker compose up -d --build
  only:
    - main
```

### Backend App

Dockerfile BE App Example:

```Dockerfile
FROM node:18.17.0-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
COPY .env.example .env

EXPOSE 3000
CMD ["npm", "run", "start"]
```

Docker Compose App example:

```yaml
version: '3.8'
services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    extra_hosts:
      - "host.docker.internal:host-gateway"
networks:
  default:
    name: nginx-network
    external: true
```

Gitlab pipeline example:

```yaml
deploy:
  stage: deploy
  tags:
    - prod
  environment:
    name: production
  script:
    - docker compose up -d --build
  only:
    - master
    - main
```

## Expose service

Create proxy host to all desired services:

- Proxy Host
- Add domain
- Add forward domain use service name at docker-compose file eg `backend`, `frontend`
- Add forward port that service exposed on Dockerfile
- Add SSL to specify domain
