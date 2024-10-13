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
#liveness & readiness
```
mkdir -p apps/kubefiles/
vi apps/kubefiles/space-alien-welcome-message-generator.yml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: space-alien-welcome-message-generator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: space-alien-welcome-message-generator
  template:
    metadata:
      labels:
        app: space-alien-welcome-message-generator
    spec:
      containers:
      - name: httpd
        image: httpd:alpine
        ports:
        - containerPort: 80
        readinessProbe:
          exec:
            command:
            - stat
            - /tmp/ready
          initialDelaySeconds: 10
          periodSeconds: 5
```
```
kubectl apply -f apps/kubefiles/space-alien-welcome-message-generator.yml
```
```
watch kubectl get pods

```
```
NAME                                      READY   STATUS    RESTARTS   AGE
space-alien-welcome-message-generator-xxxx   0/1     Running   0          Xs
```
```
PODNAME=$(kubectl get pod -l app=space-alien-welcome-message-generator -o jsonpath="{.items[0].metadata.name}")
kubectl exec -it $PODNAME -- /bin/sh
```
```
touch /tmp/ready
```
```
exit
```
```
watch kubectl get pods
```
```
NAME                                      READY   STATUS    RESTARTS   AGE
space-alien-welcome-message-generator-xxxx   1/1     Running   0          Xs
```
```
kubectl delete deployment space-alien-welcome-message-generator
```

# Section intermediaire

# Section avancé 


