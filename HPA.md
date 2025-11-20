# Horizontal Pod Autoscaler (HPA) — Simple Explanation

HPA automatically increases or decreases the number of pod replicas based on real-time metrics such as:

- CPU utilization  
- Memory utilization  
- Custom metrics (from Prometheus, apps, etc.)

So when the load increases → HPA adds more pods  
When load decreases → HPA removes pods  
This ensures performance + cost savings.  

**🔥 How HPA Works**

1. Metrics server collects real-time metrics from pods  
2. HPA checks metrics every 15 seconds (configurable)  
3. If metrics exceed threshold → HPA scales up  
4. If metrics drop below threshold → HPA scales down

Formula used:

    Desired replicas = Current replicas × (Current metric / Target metric)

Basic Architecture

Clients → LoadBalancer/Ingress → Service → Pods (scaled by HPA)

deployment-svc.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - port: 80         # service port
      targetPort: 80   # container port     
```

hpa.yaml
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 5
```

This means:

- Keep 60% CPU usage per pod  
- Minimum 2 pods  
- Maximum 5 pods

**Enable Metrics Server (Mandatory)**

Without metrics-server, HPA will NOT work:

       kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

if it is minikube

    minikube addons enable metrics-server

Then verify:
```
kubectl top pods
kubectl top nodes
```

Check HPA Status
```
kubectl get hpa
kubectl describe hpa my-app-hpa
```

<img width="946" height="579" alt="image" src="https://github.com/user-attachments/assets/4d2c7c5f-4307-4d6d-aa74-b3acc2558409" />

<img width="1050" height="490" alt="image" src="https://github.com/user-attachments/assets/9e6f31b7-83c6-41db-8188-5bce24ea1eb3" />

