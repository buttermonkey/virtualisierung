# Practical Exercise - 6

Create directory:
```bash
mkdir e6-nginx
cd e6-nginx
```
Create `docker-compose.yaml` to run our `e5-nginx`-image:
```yaml
version: "3"
services:
    client:
        image: e5-nginx
        ports:
            - 8080:80
        volumes:
            - ./src:/usr/share/nginx/html
```

Run service:
```bash
docker-compose up -d
docker-compose ls
docker container ls
```

Open URL in web-browser: http://localhost:8080/

Stop service:
```bash
docker-compose down
```
