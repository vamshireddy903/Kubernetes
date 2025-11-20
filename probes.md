# Probes
In Kubernetes, probes are health checks used by the kubelet to understand the status of your container.

Kubernetes has three types of probes:

**✅ 1. Liveness Probe**

- **Purpose:** Checks if your application is alive.  
- If this probe fails, Kubernetes restarts the container.  

**Example:** Your app is stuck or deadlocked → liveness probe detects it → pod restarts.

```
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

**✅ 2. Readiness Probe**

- **Purpose:** Checks if your app is ready to receive traffic.  
- If this probe fails, Kubernetes removes the pod from the Service endpoints.  
- The pod is running, but it won’t receive traffic until it becomes ready.

```
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**✅ 3. Startup Probe**

- **Purpose:** Checks if your app has successfully started.  
- Useful for apps that take a long time to boot.  
- While startup probe is still failing:  
- Liveness & readiness probes are ignored.  

Once startup succeeds → liveness & readiness start.
```
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
``
