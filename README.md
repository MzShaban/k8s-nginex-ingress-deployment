######

# Kubernetes NGINX-INGRESS Deployment with Minikube

######

## With this toutorial am using this infra:
- VirtualBox installed in Windows 11 
- Ubuntu installed in the above mentioned VirtualBox

## Step 1: Dependencies

### 1.1 Minikube
### 1.2 kubectl
### 1.3 Start Minikube  (driver=docker)
Start Minikube with the Docker driver Because we are in VirtualBox:
```bash
minikube start --driver=docker
```
### 1.4 Enable Ingress Controller
Enable the Ingress controller:
```bash
minikube addons enable ingress
```

## Step 2: Create Kubernetes Manifests (YAML)
I created a new directory in Ubuntu Machine k8s-app in the root directory and inside this directory i created all the yaml files

### 2.1 Create ConfigMap
Create a `configmap.yaml` file to define the `index.html` file for the NGINX deployment:
```bash
nano configmap.yaml
```
yaml content:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  index.html: |
    <html>
    <head><title>Hello</title></head>
    <body><h1>Hello, Kubernetes!</h1></body>
    </html>
```
Apply the ConfigMap:
```bash
kubectl apply -f configmap.yaml
```

### 2.2 Create Deployment
Create a `deployment.yaml` file for the NGINX deployment:
```bash
nano deployment.yaml
```
yaml content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-default-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: nginx-config
```
Apply the Deployment:
```bash
kubectl apply -f deployment.yaml
```

### 2.3 Create Service
Create a `service.yaml` file for the NGINX service:
```bash
nano service.yaml
```
yaml content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```
Apply the Service:
```bash
kubectl apply -f service.yaml
```

### 2.4 Create Ingress
Create an `ingress.yaml` file to define Ingress rules:
```bash
nano ingress.yaml
```
yaml content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: hello.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```
Apply the Ingress:
```bash
kubectl apply -f ingress.yaml
```

## Step 3: Networking

### 3.1 Minikube Tunnel
Start the tunnel in one terminal:
```bash
minikube tunnel
```
In a new terminal, run:
```bash
kubectl get ingress
```
![alt text](https://github.com/MzShaban/Devops-projects/blob/main/Images/networktestingress.png?raw=true)



and because we are in local enviroment we can too edit `/etc/hosts` and add the following line:
```bash
<MINIKUBE-IP> hello.local
```
Access the service via:
```bash
http://hello.local
```
![alt text](https://github.com/MzShaban/Devops-projects/blob/main/Images/hellokub.png?raw=true)


## Step 4: Logs & Debugging

### 4.1 Check All Resources
```bash
kubectl get all
kubectl describe ingress nginx-ingress
```
![alt text](https://github.com/MzShaban/Devops-projects/blob/main/Images/resources.png?raw=true)


### 4.2 Check Logs
```bash
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

![alt text](https://github.com/MzShaban/Devops-projects/blob/main/Images/logsingress.png?raw=true)

### 4.3 Test Connectivity
```bash
curl -v http://hello.local
```
![alt text](https://github.com/MzShaban/Devops-projects/blob/main/Images/connectivity.png?raw=true)


