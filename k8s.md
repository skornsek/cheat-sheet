# Kubernetes = k8s
## Start
```
minikube start

minikube start --vm-driver virtualbox           // določimo hypervisor

minikube start --kubernetes-version v1.16.2     //verzija

minikube start --cpus 4 --memory 8g             //limiti

minikube start --memory 4G --vm-driver virtualbox --kubernetes-version v1.17.0
```

kubectl config

Managing all aspects of contexts is done via the kubectl config command. Some examples include:

- `kubectl config current-context` trenutni kontekst
- `kubectl config get-contexts` list kontekstov
- `kubectl config use-context <context-name> command` uporabi drug kontekst
- `kubectl config set-context <context name> --cluster=<cluster name> --user=<user> --namespace=<namespace>` ustvari kontekst





kubectl get namespaces //list namespacesov

kubectl create namespace dev //ustvari namespace

kubectl create -f manifests/mypod.yaml //naredi pod iz yamla (vse povozi)

kubectl apply -f manifests/mypod.yaml // inkrementna sprememba iz yamla

kubectl get pod mypod -o yaml //izpis poda v yaml formatu

kubectl describe pod mypod //opis poda

kubectl delete pod mypod //izbris poda

kubectl describe pod mypod //opis poda

kubectl logs mypod // logi poda

kubectl exec mypod -- cat /etc/os-release //z exec dostopamo do poda, comanda cat

kubectl exec -i -t mypod -- /bin/sh //zaženemo interactive shell

minikube dashboard //dostop do dashborda

kubectl proxy //dostop do servicov

http://127.0.0.1:8001/api/v1/namespaces/dev/pods/mypod/proxy/ //dostop do service poda

kubectl label pod pod-example app=nginx environment=dev //označi pod z app=nginx environment=dev

kubectl get pods --show-labels // pod z labeli

kubectl get pods --selector environment=prod //filtriramo na podlagi label

kubectl get pods -l 'app in (nginx), environment notin (prod)' //primer filtra

Pod example1:
********************************************************
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
********************************************************

Pod multicontainer:
********************************************************
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-example
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: content
    image: alpine:latest
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          echo $(date)"<br />" >> /html/index.html;
          sleep 5;
        done
  volumes:
  - name: html
    emptyDir: {}
********************************************************

Service example1:
********************************************************
  apiVersion: v1
kind: Service
metadata:
  name: clusterip
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
********************************************************
http://127.0.0.1:8001/api/v1/namespaces/dev/services/clusterip/proxy/ //dostop do servica

kubectl exec pod-example -- nslookup clusterip.dev.svc.cluster.local //iz example poda pogledamo ip od servica

Service example2:
********************************************************
apiVersion: v1
kind: Service
metadata:
  name: nodeport
spec:
  type: NodePort
  selector:
    app: nginx        //targeta nginx
    environment: prod //targeta prod
  ports:
  - nodePort: 32410
    protocol: TCP
    port: 80
    targetPort: 80
********************************************************

minikube service -n dev nodeport // command to open the newly exposed nodeport

LoadBalancer example1:
********************************************************
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
    environment: prod
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
