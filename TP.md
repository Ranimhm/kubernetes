# section débutant
Pour définir les Resource Requests et Limits de mon pod, j'ai commencé par créer le namespace 'Limits'
 
```
kubectl create namespace limit
```
# pod
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
# service
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: europe
spec:
  selector:
    app: europe
  ports:
    - protocol: TCP
      port: 30291
      targetPort: 80
  type: LoadBalancer
EOF
```
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: asia
spec:
  selector:
    app: asia
  ports:
    - protocol: TCP
      port: 30292
      targetPort: 80
  type: LoadBalancer
EOF
```
```
kubectl get services
```

# déploiement blue/green
```
mkdir -p apps/kubefiles/
vi apps/kubefiles/wonderful-v1.yml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wonderful
  name: wonderful-v1
spec:
  replicas: 4
  selector:
    matchLabels:
      app: wonderful
      version: v1
  template:
    metadata:
      labels:
        app: wonderful
        version: v1
    spec:
      containers:
      - image: httpd:alpine
        name: httpd
```
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wonderful
  name: wonderful
spec:
  ports:
  - port: 30290
    protocol: TCP
    targetPort: 80
  selector:
    app: wonderful
    version: v1
  type: LoadBalancer
```
```
kubectl apply -f apps/kubefiles/wonderful-v1.yml
```
```
watch kubectl get pods
```
```
IP=$(minikube ip)
PORT=$(kubectl get service/wonderful -o jsonpath="{.spec.ports[*].nodePort}")
```
```
curl $IP:$PORT
```
```
while true
do curl $IP:$PORT
sleep .3
done
```
```
vi apps/kubefiles/wonderful-v2.yml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wonderful
  name: wonderful-v2
spec:
  replicas: 4
  selector:
    matchLabels:
      app: wonderful
      version: v2
  template:
    metadata:
      labels:
        app: wonderful
        version: v2
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
```
```
kubectl apply -f apps/kubefiles/wonderful-v2.yml
```
```
kubectl get pods -l app=wonderful -l version=v2
```
```
kubectl patch svc/wonderful -p '{"spec":{"selector":{"version":"v2"}}}'
```
```
kubectl scale deployment/wonderful-v1 --replicas=0
```
```
curl $IP:$PORT
```
```
while true
do curl $IP:$PORT
sleep .3
done
```







# Section elementaire 

# Section intermediaire

# Section avancé 


