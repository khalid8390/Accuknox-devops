## Dockerization
1. **Clone the repository** 
git clone https://github.com/nyrahul/wisecow

2. **Write Dockerfile**:
 ```Dockerfile
# Use a base image
FROM ubuntu:latest

# Install necessary packages
RUN apt-get update && apt-get install -y cowsay fortune netcat

# Copy the script into the container
COPY wisecow.sh /app/wisecow.sh

# Set the working directory
WORKDIR /app

# Make the script executable
RUN chmod +x wisecow.sh

# Expose the server port
EXPOSE 4499

# Set the startup command
CMD ["./wisecow.sh"]
```

3. **Build the Docker image**:
```bash
docker build -t wisecow-app .
```
4. **Test the Docker image locally**:
```bash
docker run -p 8080:80 wisecow-app
```

## Kubernetes Deployment

1. **Create Deployment YAML (`wisecow-deployment.yaml`)**:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: wisecow-deployment
      labels:
        app: wisecow
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: wisecow
      template:
        metadata:
          labels:
            app: wisecow
        spec:
          containers:
          - name: wisecow
            image: <khalid8390>/wisecow-app:latest
            ports:
            - containerPort: 4499
    ```
2. **Create Service YAML (`wisecow-service.yaml`)**:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: wisecow-service
    spec:
      selector:
        app: wisecow
      ports:
      - protocol: TCP
        port: 80
        targetPort: 4499
      type: LoadBalancer
    ```
3. **Apply the manifest files**:
    ```bash
    kubectl apply -f wisecow-deployment.yaml
    kubectl apply -f wisecow-service.yaml
    ```

## Continuous Integration and Deployment (CI/CD)

1. **GitHub Actions Workflow**:
   - Create `.github/workflows/ci-cd.yml`:
     ```yaml
     name: CI/CD Pipeline

     on:
       push:
         branches:
           - main

     jobs:
       build-and-deploy:
         runs-on: ubuntu-latest
         steps:
         - name: Checkout code
           uses: actions/checkout@v2
         - name: Login to Container Registry
           uses: docker/login-action@v1
           with:
             username: ${{ secrets.REGISTRY_USERNAME }}
             password: ${{ secrets.REGISTRY_PASSWORD }}
         - name: Build Docker image
           run: docker build -t <your-container-registry>/wisecow-app:latest .
         - name: Push Docker image
           run: docker push <your-container-registry>/wisecow-app:latest
         - name: Deploy to Kubernetes
           run: kubectl apply -f wisecow-deployment.yaml
           env:
             KUBECONFIG: ${{ secrets.KUBE_CONFIG_DATA }}
     ```
   - Configure Docker and Kubernetes secrets in the GitHub repository settings.

## TLS Implementation

1. **Generate TLS Certificates**:
   - Use tools like Let's Encrypt to generate certificates.
   -cluster-issuer.yaml
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: shaikhk968@gmail.com
    privateKeySecretRef:
      name: letsencrypt-private-key
    solvers:
    - http01:
        ingress:
          class: nginx
```

2. **Apply the ClusterIssuer**:
 ```bash
kubectl apply -f cluster-issuer.yaml
```

3. **Create Kubernetes Ingress Resource with TLS**:
```yaml
   # ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-deployment
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt"
spec:
  tls:
  - hosts:
    - your-domain.com
    secretName: wisecow-deployment-tls
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wisecow-deployment
            port:
              number: 80
```


