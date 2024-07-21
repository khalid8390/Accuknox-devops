1.Clone the repository  https://github.com/nyrahul/wisecow

2.Write Dockerfile:
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
3. Build the Docker image:
docker build -t wisecow-app .
4. Test the Docker image locally:
docker run -p 8080:80 wisecow-app


Kubernetes Deployment
1.Create Deployment YAML (wisecow-deployment.yaml):
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
        image: <your-container-registry>/wisecow-app:latest
        ports:
        - containerPort: 80
2.Create Service YAML (wisecow-service.yaml):
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
    targetPort: 80
  type: LoadBalancer
3. Apply the manifest files:
kubectl apply -f wisecow-deployment.yaml
kubectl apply -f wisecow-service.yaml
