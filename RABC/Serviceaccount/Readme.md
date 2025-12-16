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

Expected:  
no
