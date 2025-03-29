# Practical Exercise - 8

Create directory:
```bash
mkdir app
cd app
```

A full-blown kubernetes cluster is the next step up from Docker Compose.
In addition to orchestrating multiple services to work together, it allows to scale deployments with ease. It also
provides for more fine-grained control of the software defined infrastructure in the cluster.

In this example we will deploy an application consisting of a very simple application consisting of a public accessible
frontend component and a backend component that is only available within the cluster, which provides data to the
frontend service.

Create `namespace.yaml` to define the namespace for our application. It consists of the following parts:
- `Namespace`: the namespace for our application.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: app
```

Create `backend.yaml` to deploy a simple service that provides pretty pointless information to the current date. It
consists of following parts:
- `Deployment`: this is the meat of our backend component. It serves a similar purpose as an entry in the `services`
   section of a `docker-compose.yaml` file.
  - number of replicas: `2`
- `Service`: defines how the deployment presents itself to the world (i.e. how to access its _services_).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: greenlemon/api-date:development
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: app
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

Next we define our frontend (`frontend.yaml`), which merely consists of an `nginx` server acting as an HTTP reverse
proxy to forward user requests to our backend service:
- `ConfigMap`: embeds nginx configuration file for our frontend (`nginx.conf`).
- `Deployment`: run nginx on port `80` and use the `nginx.conf` file defined in the `ConfigMap` section (mount it as
  `/etc/nginx/nginx.conf`).
    - number of replicas: `3`
- `Service`: make the application accessible to the outside world on port `80`.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config-map
  namespace: app
data:
  nginx.conf: |
    events { }
    http {
      server {
        listen 80;
        location / {
          proxy_pass http://backend-service.app.svc.cluster.local:80;
        }
      }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-config-map
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: app
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
  type: LoadBalancer
```

Deploy and run application:
```bash
kubectl apply -f namespace.yaml
kubectl apply -f backend.yaml
kubectl apply -f frontend.yaml
kubectl -n app get all
```

There should now be two pods for our backend and three pods for our frontend component in the `app`-namespace:
```plain
$ kubectl -n app get pods

NAME                                       READY   STATUS    RESTARTS   AGE
pod/backend-deployment-94bd5d6f6-62ff8     1/1     Running   0          11m
pod/backend-deployment-94bd5d6f6-7zhqv     1/1     Running   0          11m
pod/frontend-deployment-68c876c9d7-f9bl8   1/1     Running   0          10m
pod/frontend-deployment-68c876c9d7-k7dn8   1/1     Running   0          10m
pod/frontend-deployment-68c876c9d7-xjc5w   1/1     Running   0          10m
```

To apply updates in our configuration, just rerun the above commands. Kubernetes will automatically determine whether
there are changes to be applied.

Open URL in web-browser: http://localhost:8080/

...or via `curl`:
```plain
$ curl localhost:8080

Fakta hari ini adalah  March 28th is the day in 1913 that Guatemala becomes a signatory to the Buenos Aires copyright treaty.
```

Remove application from cluster (this will delete the namespace and everything in it):
```bash
# Drop the namespace defined in namespace.yaml
kubectl delete -f namespace.yaml

# Alternatively we can drop the namespace explicitly:
kubectl delete namespace app
```
