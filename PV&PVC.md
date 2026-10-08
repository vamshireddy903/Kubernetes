<img width="771" height="617" alt="image" src="https://github.com/user-attachments/assets/7f050ef8-1bcb-49f8-846b-64831255bfcd" />

<img width="609" height="674" alt="image" src="https://github.com/user-attachments/assets/0d4910eb-fba3-42e6-a28b-d7e61a767b37" />

# Creating PV

# Create Persistent Volume


pv.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  hostPath:
    path: /data/myapp
```

# Apply it

    kubectl apply -f pv.yaml

# Check

      kubectl get pv

<img width="762" height="391" alt="image" src="https://github.com/user-attachments/assets/1612777a-a9b3-4a48-b6f6-304b092cd6c3" />


<img width="1096" height="467" alt="image" src="https://github.com/user-attachments/assets/12fb4dda-ef54-4d2e-a566-c020e931cde7" />


Other common modes are:

# ReadWriteOnce (RWO)
    ReadOnlyMany  (ROX)
    ReadWriteMany (RWX)

# For example:

    RWO → commonly AWS EBS
    RWX → commonly AWS EFS / NFS
    
<img width="887" height="371" alt="image" src="https://github.com/user-attachments/assets/52219393-4fba-4604-aaf8-3c447d18c1c2" />

<img width="932" height="541" alt="image" src="https://github.com/user-attachments/assets/ff27b2cf-ac95-4fac-adec-45403e83513d" />

<img width="851" height="396" alt="image" src="https://github.com/user-attachments/assets/763123f3-1c21-45ee-a079-dda8637c1ca3" />

<img width="868" height="637" alt="image" src="https://github.com/user-attachments/assets/9b193a08-92cd-4383-a4a8-9ed7318e2e48" />

<img width="658" height="781" alt="image" src="https://github.com/user-attachments/assets/87140269-5750-4c68-8bfb-658b9daaf427" />

# Creating PVC

pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi
```

# Apply it

     kubectl apply -f pvc.yaml

# Get PVC

     kubectl get pvc

<img width="565" height="372" alt="image" src="https://github.com/user-attachments/assets/15f2ebde-32d1-4d19-9f8a-e6cfe0d7d1e3" />


# 3. StatefulSet

statefulset.yaml

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-app
spec:
  serviceName: my-app
  replicas: 1

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
      - name: nginx
        image: nginx

        volumeMounts:
        - name: storage
          mountPath: /app/data

      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: my-pvc
```

# Create it:

    kubectl apply -f statefulset.yaml

# Check:

    kubectl get pods

# You should get:

    my-app-0


# Now understand the complete flow

So if your application inside my-app-0 writes:

    echo "hello" > /app/data/test.txt

the data ultimately goes to:

    /data/myapp/test.txt

on the node.
<img width="586" height="754" alt="image" src="https://github.com/user-attachments/assets/6b77e2fc-f470-4f44-9c57-119d176c1d9f" />










