# 1. Resource request

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
