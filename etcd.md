# etcd

etcd is a distributed, consistent key-value store used by Kubernetes to store all cluster state and configuration data.

# 🔑 What etcd stores

 etcd is the single source of truth for the Kubernetes cluster. It stores:  
- Pods, Services, Deployments  
- ConfigMaps & Secrets  
- Node information  
- RBAC policies  
- Network policies  
- Cluster configuration & metadata  

👉 Anything you create using kubectl is stored in etcd.

🧠 Why Kubernetes uses etcd
  
- Strong consistency (uses Raft consensus)  
- High availability  
- Fast read/write operations  
- Reliable leader election  

# 🏗 Where etcd runs

- In self-managed clusters (kubeadm):
  
etcd runs as a static pod on control-plane nodes

- In managed services (EKS, AKS, GKE):

etcd is fully managed by the cloud provider

<img width="827" height="615" alt="image" src="https://github.com/user-attachments/assets/6a6d2166-8f00-4626-9104-908a4cfed4cc" />

# etcd backup

Simple:

      kubectl get -A -o yaml > backup.yaml
      
<img width="786" height="777" alt="image" src="https://github.com/user-attachments/assets/26356746-3505-4a68-b9ee-4bb48876425f" />

<img width="891" height="765" alt="image" src="https://github.com/user-attachments/assets/52c75702-703d-41b4-b633-d84dd16e5921" />

<img width="748" height="544" alt="image" src="https://github.com/user-attachments/assets/42e86e8c-b13a-4d45-a393-5123bea42064" />

# 🔹 How to confirm the path (recommended way)

1️⃣ Check etcd pod

     kubectl -n kube-system get pod | grep etcd

2️⃣ Inspect etcd manifest  
inside minikube:

    cat /etc/kubernetes/manifests/etcd.yaml


Look for:

    - --data-dir=/var/lib/etcd
That is the actual storage path.

in mikube:

<img width="986" height="295" alt="image" src="https://github.com/user-attachments/assets/74290a92-f1ff-4de4-8d61-8e1de502433a" />


<img width="783" height="707" alt="image" src="https://github.com/user-attachments/assets/5078cd36-6db4-4494-8059-97edada799ec" />

# Steps to take backup

Install inside minikube (minikube ssh)

**1. Install etcd client**

    sudo apt update
    sudo apt install etcd-client -y

  Other ways:
  
1️⃣ Set the version variable (IMPORTANT)

    export ETCD_VER=v3.5.13


Verify:

    echo $ETCD_VER

2️⃣ Re-download correctly

    curl -LO https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz

3️⃣ Verify file type

    file etcd-${ETCD_VER}-linux-amd64.tar.gz

Expected:

gzip compressed data

If it says HTML document, delete and re-download.

4️⃣ Extract archive

    tar -xzf etcd-${ETCD_VER}-linux-amd64.tar.gz

5️⃣ Move etcdctl binary

     sudo mv etcd-${ETCD_VER}-linux-amd64/etcdctl /usr/local/bin/

     sudo mv etcd-v3.5.13-linux-amd64/etcdutl /usr/local/bin/


6️⃣ Verify installation

     etcdctl -v

**2. Snapshot using etcdctl**
```
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>
```
example:

```
    sudo ETCDCTL_API=3 etcdctl snapshot save /opt/backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/var/lib/minikube/certs/etcd/ca.crt \
  --cert=/var/lib/minikube/certs/etcd/server.crt \
  --key=/var/lib/minikube/certs/etcd/server.key
```

<img width="1539" height="268" alt="image" src="https://github.com/user-attachments/assets/2fa30740-273a-45bd-a7e4-64acc7b05da5" />

**3 Verify the snapshot**

      etcdutl --write-out=table snapshot status <backup file>
      
example:

         sudo etcdutl --write-out=table snapshot status /opt/backup.db

  <img width="1233" height="189" alt="image" src="https://github.com/user-attachments/assets/1b52655e-f6ea-40a2-98e0-fb5c4dd74c24" />


For more info: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/


# Restore backup

  ```
 ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/var/lib/minikube/certs/etcd/ca.crt --cert=/var/l
ib/minikube/certs/etcd/server.crt --key=/var/lib/minikube/ce
rts/etcd/server.key \
snapshot restore /opt/backup.db --data-dir=/var/lib/minikube/etcd-restore-from-backup
``
