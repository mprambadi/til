# Nginx Proxy Manager

Nginx proxy manager used to simplify expose service, it already include with nginx and ui dashboard inside it, to manage service that we want to expose

Nginx Proxy manager docker-compose file

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
 
networks:
  default:
    name: nginx-network
    external: true
```

