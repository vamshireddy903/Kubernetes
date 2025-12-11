# Network policies

 <img width="984" height="471" alt="image" src="https://github.com/user-attachments/assets/8a2a0995-4ada-4418-b4bd-d1ff3ac49cc7" />

 <img width="878" height="475" alt="image" src="https://github.com/user-attachments/assets/ed6420c6-bc04-4261-bb9a-13066a152813" />


 By default, Kubernetes allows all pods to talk to each other freely. A Network Policy is a set of rules that restricts this traffic.
 
**What it does:**
- Controls Traffic: It specifies exactly who (which pods) can connect to whom.  
- Uses Labels: It doesn't use IP addresses. It uses Labels (e.g., app: database) to identify which pods to block or allow.  
- Layer 3/4: It works at the IP and Port level (OSI Model).  

**The 3 Key Parts of a Policy:**  
**Target:** Which Pods are we protecting? (e.g., The Redis DB)  
**Direction:** Are we filtering incoming traffic (Ingress) or outgoing traffic (Egress)?  
**Rule:** Who is allowed? (e.g., Only pods labeled app: frontend).  

**Simple Example**  
If you apply this policy, only the Frontend can talk to the Database. Everyone else is blocked.

# STEP 1 — Install the Calico operator (CRDs + controller)

You MUST run this first:

     kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.2/manifests/tigera-operator.yaml
     
Verify CRDs installed:

    kubectl get crds | grep tigera


# STEP 2 — Apply custom-resources (Calico configuration)

        kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.2/manifests/custom-resources.yaml

# 3. Start minikube with --cni=calico

        minikube start --cni=calico

# 4. Check calico pods status

       kubectl get pods -n kube-system

# EXAMPLE:

**Step 1 — Deploy Frontend (nginx)**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        role: frontend
    spec:
      containers:
      - name: frontend
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80


---
```
# 🏷️ Step 2 — Deploy Backend (nginx)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        role: backend
    spec:
      containers:
      - name: backend
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80


---
```

# 🏷️ Step 3 — Deploy MySQL (DB)
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: "root123"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
        role: db
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
        ports:
        - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
  type: ClusterIP


---
```

# 🚧 Step 4 — Apply Network Policies

**1️⃣ Deny all inbound traffic to MySQL**
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mysql-deny-all
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
```
👉 After this, no pod can access DB.


---

**2️⃣ Allow ONLY backend pods to access MySQL**
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-mysql
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - port: 3306
      protocol: TCP
```
**One requirement:** Your Kubernetes cluster must use a network plugin (CNI) that supports these policies, like Calico or Weave.
If you use the basic default networking, these policies are ignored.


# If they are in different namespace:

**1. Create namespace**
```
apiVersion: v1
kind: Namespace
metadata:
  name: backend-ns
  labels:
    name: backend-ns
---
apiVersion: v1
kind: Namespace
metadata:
  name: db-ns
  labels:
    name: db-ns
---
apiVersion: v1
kind: Namespace
metadata:
  name: frontend-ns
  labels:
    name: frontend-ns

```
**2. Frontend deployment**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: frontend-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      tier: frontend
  template:
    metadata:
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: frontend-nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: frontend-ns
spec:
  selector:
    app: nginx
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
```

**3. Deploy Backend (nginx) in backend-ns**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: backend-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        role: backend
    spec:
      containers:
      - name: backend
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: backend-ns
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80

```

**4. Deploy MySQL DB in db-ns**

```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: db-ns
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: "root123"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: db-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
        role: db
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
        ports:
        - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: db-ns
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
  type: ClusterIP

```

**5 — Deny ALL inbound traffic to DB (default deny)**
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mysql-deny-all
  namespace: db-ns
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress

```

**6 — Allow ONLY backend pods to access DB**

Because backend and DB are in different namespaces, you must use:  

- namespaceSelector  
- podSelector

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-mysql
  namespace: db-ns
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: backend-ns       # namespace label
      podSelector:
        matchLabels:
          role: backend          # backend pod label
    ports:
    - protocol: TCP
      port: 3306

```

# To test

Install mysql client in both frontend and backed pod when login

    apt update
    apt install -y default-mysql-client

Then

    mysql -h mysql.db-ns.svc.cluster.local -P 3306 -u root -p --ssl=0
    
--ssl=0 tells the MariaDB client to not use SSL.

provide password and you wont see any response when you did this from froent end conatiner and do this same in backed container and you will be able to connect mysql
