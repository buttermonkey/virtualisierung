# Practical Exercise - 5

Create directory:
```bash
mkdir e5-nginx
cd e5-nginx
```
Create `Dockerfile` for our `e5-nginx`-image in this directory:
```yaml
FROM nginx:latest
```

Build image:
```bash
docker build -t e5-nginx .
docker image ls
```

Run service:
```bash
docker run -it --rm -d -p 8080:80 --name e5-nginx e5-nginx
docker container ls
```

Open URL in web-browser: http://localhost:8080/

Stop container:
```bash
docker stop n5-nginx
```
