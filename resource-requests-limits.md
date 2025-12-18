# 1. ResourceQuota

ResourceQuota limits TOTAL resource usage in a namespace

Controls:

- CPU  
- Memory  
- Number of pods  
- Services  
- PVCs, etc.  

<img width="689" height="543" alt="image" src="https://github.com/user-attachments/assets/07e8cda2-490f-4314-bbc4-b810b8975667" />

<img width="551" height="397" alt="image" src="https://github.com/user-attachments/assets/bec990f1-869c-445e-ae4c-9733560f040b" />

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: dev
spec:
  hard:
    requests.cpu: "1"
    requests.memory: "1Gi"
    limits.cpu: "2"
    limits.memory: "2Gi"
    pods: "1"
    services: "1"
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      role: frontend

  template:
    metadata:
      labels:
        role: frontend
    spec:
      containers:
      - name: my-container
        image: oplinux/stress
        resources:
          requests:
            cpu: "250m"
            memory: "1Gi"
          limits:
            cpu: "500m"
            memory: "2Gi"
        ports:
        - containerPort: 80
```

# 2. Resource request

- This is the minimum guaranteed resource a container will get.  
- If a container requests 100m CPU → K8s guarantees it will get that much CPU.

**Example:**

```
resources:
  requests:
    cpu: "100m"
    memory: "150Mi"
```
Meaning: 

- Pod needs 0.1 vCPU to run.  
- Pod needs 150Mi RAM minimum to run.

**2. Resource Limits**

This is the maximum amount the container is allowed to use.

Example:
```
resources:
  limits:
    cpu: "150m"
    memory: "200Mi"
```
Meaning:

- Container cannot use more than 0.15 vCPU.  
- If it uses more memory → OOMKilled.  
- If it uses more CPU → throttled (slowed down) but not killed.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          - name: nginx-container
            image: polinux/stress
            ports:
              - containerPort: 80

            resources:
              requests:
                cpu: "100m"
                memory: "150Mi"
              limits:
                cpu: "150m"
                memory: "200Mi"

            command: ["stress"]
            args: ["--vm", "1", "--vm-bytes", "210M", "--vm-hang", "1"]
```

<img width="1334" height="356" alt="image" src="https://github.com/user-attachments/assets/b2e0b4b1-2755-4358-a555-02bfbdf21099" />
