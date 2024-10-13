# section débutant
Pour définir les Resource Requests et Limits de mon pod, j'ai commencé par créer le namespace 'Limits'
 
```
kubectl create namespace limit
```

j'ai crée le pod avec ce script

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ressource-checker
  namespace: limit
spec:
  containers:
  - name: my-container
    image: httpd:alpine
    resources:
      requests:
        memory: "30Mi"
        cpu: "30m"
      limits:
        memory: "30Mi"
        cpu: "300m"
EOF
```
# Section elementaire 

# Section intermediaire

# Section avancé 


