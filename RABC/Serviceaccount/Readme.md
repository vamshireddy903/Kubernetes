# Verify access
```
kubectl auth can-i get pods \
  --as=system:serviceaccount:<name-space-name>:<service-account-name> \
  -n <namespace-name>
```

Example:
```
kubectl auth can-i get pods \
  --as=system:serviceaccount:dev:my-user \
  -n dev
```

Expected output:

yes

**Try something not allowed**
```
kubectl auth can-i delete pods \
  --as=system:serviceaccount:dev:my-user \
  -n dev
```
Expected o/p

no

```
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create"]
```
1️⃣ apiGroups — WHICH API the resource belongs to

Kubernetes resources are grouped into API groups.

<img width="758" height="637" alt="image" src="https://github.com/user-attachments/assets/80684518-2fc3-4bf7-8648-7b9decc8740e" />

<img width="640" height="478" alt="image" src="https://github.com/user-attachments/assets/536e9dd1-5579-434d-bf31-a0d9c7718b3c" />

<img width="749" height="674" alt="image" src="https://github.com/user-attachments/assets/f2c8cb80-6c59-4be9-ba75-bc90a68f8158" />

<img width="590" height="616" alt="image" src="https://github.com/user-attachments/assets/874a0339-1e57-42e2-afc7-7b31abcbf760" />

**2️⃣ resources — WHAT objects the user can access**

This defines which Kubernetes resources.

🔹 Single resource

    resources: ["pods"]

🔹 Multiple resources

    resources: ["pods", "services", "configmaps"]

<img width="730" height="571" alt="image" src="https://github.com/user-attachments/assets/96ccdd46-98a7-4cde-a170-e054747d48e9" />

<img width="345" height="371" alt="image" src="https://github.com/user-attachments/assets/ea71a34d-59b6-44e0-9444-451ce8131a31" />


