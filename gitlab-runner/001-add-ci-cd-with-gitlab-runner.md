# CI/CD with gitlab runner

## Gitlab Runner Setup 

### Download binary 
```bash 
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permission to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab Runner user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash


# Install and run as a service
# change user with user prefered and change working-directory also
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

### Register runner 
Tag for runner will create for ci cd pipeline also 

```bash
gitlab-runner register  --url https://gitlab.com  --token ${token}
```


### Veiry Runner Already run 

```bash
gitlab-runner run
```


## Setup rootles user 

Run this on root user 

```sh
visudo 
```

Not recomened but to allow all app, better specify only with app that need run without rootless password
add this to down of visudo config
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
All of new service should have same network configuration

### Client side Frontend App 
Dockerfile FE React App Example 

create Dockerfile 
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

Docker Compose App example 
create docker-compose.yml file 

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

Gitlab pipeline example 
create .gitlab-ci.yml
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

Dockerfile BE App Example 
create Dockerfile 

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

Docker Compose App example 
create docker-compose.yml file 
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

Create proxy host to all desired service 

- Proxy Host 
- Add domain 
- Add forward domain use service name at docker-compose file eg `backend`, `frontend`
- Add forward port that service exposed on dockerfile 
- Add SSL to specify domain 

