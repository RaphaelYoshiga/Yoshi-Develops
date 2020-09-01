
[https://github.com/saaguero/ckad-notes/](https://github.com/saaguero/ckad-notes/)  [https://www.katacoda.com/contino/courses/kubernetes](https://www.katacoda.com/contino/courses/kubernetes)  Cheatsheet -  [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

Boss - CKAD master -  [https://www.katacoda.com/courses/kubernetes/first-steps-to-ckad-certification](https://www.katacoda.com/courses/kubernetes/first-steps-to-ckad-certification)

VIM basics 'i' > to start inserting :q to exit :wq to save exit
:set paste (To be able to paste without auto formatting)


## 13% - Core Concepts 

• Understand Kubernetes API primitives

• Create and configure basic Pods

## 18% - Configuration

• Understand ConfigMaps  [https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

Config maps can be exported kubectl get configmap {name} -o yaml

Config maps can be created

    kubectl get pods kubectl delete pod {name} kubectl logs {pod_name}  
    kubectl exec mypod -it sh -- reverse shell
    
    kubectl apply -f fileName

• Understand SecurityContexts

Security context defines which permissions the pod/container has inside itself or to nearby volumes

• Define an application’s resource requirements

Request - How much the container is requesting from, it isn't going to be created if it doesn't satisfy it Limits - Maximum amount of resource to be alocated to this resource

resources.requests.memory resources.requests.cpu resources.limits.memory resources.limits.cpu

CPU: 1 unit = one VCore/core on the cloud or 1 hyper thread 0.1 = 100m

This unit is absolute, it's going to be the same regardless of the computer CPU size

Memory Measure in bytes

1Gi =1024 Mi 1024Mi = 1070Mb

• Create & consume Secrets  [https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)

    apiVersion: v1 kind: Pod metadata: name: secret-env-pod spec: containers: - name: mycontainer image: alpine:latest command: ["sleep", "9999"] envFrom: configMapReference: name: test-secret env: - name: SECRET_USERNAME valueFrom: secretKeyRef: name: test-secret key: username - name: SECRET_PASSWORD valueFrom: secretKeyRef: name: test-secret key: password restartPolicy: Never

You can get secrets as files mounted in the disk or from env variables apiVersion: v1 kind: Pod metadata: name: secret-vol-pod spec: volumes:

-   name: secret-volume secret: secretName: test-secret containers:
    -   name: test-container image: alpine:latest command: ["sleep", "9999"] volumeMounts: - name: secret-volume mountPath: /etc/secret-volume

Learn linux Octal notation for the permissions

.asKey to create hidden files.

• Understand ServiceAccounts

## 10% Multi-Container Pods

• Understand Multi-Container Pod design patterns (e .g. ambassador, adapter, sidecar)

Ambassador is a load balancer L7/API gateway. Can control rate limits Can be used to proxy inside the cluster as well

Side car is an add-in to the main container, for example to collect logs:

Adapter - Same as the side car, but useful to do a bit of ETL in the logs as well.

## 20% - Pod Design

• Understand Deployments and how to perform rolling updates

k edit deployment {name} K run nginx-deployment -n contino --image=nginx --port 80 k scale deployment nginx-deployment --replicas=10 -n contino K set image deploy/nginx-deployment nginx-deployment=nginx:unresolvabletag -n contino

• Understand Deployments and how to perform rollbacks

kube rollout undo (deployName)  
kubectl rollout status deployment examplehttpapp

• Understand Jobs and CronJobs

[https://kubernetes.io/examples/controllers/job.yaml|](https://kubernetes.io/examples/controllers/job.yaml%7C)

apiVersion: batch/v1 kind: Job metadata: name: pi spec: template: spec: containers: - name: pi image: perl command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"] restartPolicy: Never backoffLimit: 4

spec.backoffLimit - Default is 6 Max number of retries for the job

[https://www.katacoda.com/contino/courses/kubernetes/jobs-initcontainers-cronjobs](https://www.katacoda.com/contino/courses/kubernetes/jobs-initcontainers-cronjobs)

• Understand how to use Labels, Selectors, and Annotations#

k create namespace xxx k get namespace xxx k get pods -n xxx k delete namespace xxx

Labels are simple tagss

apiVersion: v1 kind: Pod metadata: name: happypanda namespace: dev-service1 labels: app: redis segment: backend company: mycompany  
spec: containers:

-   name: redis image: redis:4.0.10 ports:
    -   name: redisport containerPort: 6379 protocol: TCP

Selectors:

nodeSelector Affinity Anti-Affinnity preferredDuringSchedulingIgnoredDuringExecution - Prefers, but if it doesn't find it, it still creates the container requiredDuringSchedulingIgnoredDuringExecution - Must find those affinity

spec: affinity: nodeAffinity: preferredDuringSchedulingIgnoredDuringExecution: - weight: 1 preference: matchExpressions: - key: fruit perator: NotIn values: - apple

## 8% - State Persistence

 • Understand PersistentVolumeClaims for storage

Persistent volume claim, is a claim into the Persistent Volume. Persistent Volume is a NFS drive

apiVersion: v1 kind: PersistentVolume metadata: name: spec: capacity: storage: 1Gi accessModes: - ReadWriteOnce - ReadWriteMany persistentVolumeReclaimPolicy: Recycle nfs: server: path:

----------

kind: PersistentVolumeClaim apiVersion: v1 metadata: name: claim-mysql spec: accessModes: - ReadWriteOnce resources: requests: storage: 3Gi

## 18% - Observability

• Understand LivenessProbes and ReadinessProbes

livenessProbe: httpGet: path: / port: 80 initialDelaySeconds: 1 timeoutSeconds: 1 readinessProbe: httpGet: path: / port: 80 initialDelaySeconds: 1 timeoutSeconds: 1

Liveness = Tell kubernetes when the container is unresponsive and it can be safely restarted Readiness = Tell Kubernetes when the container is ready. While the container is getting ready is not in the load balancer, so no traffic gets sent to it.

Types of readiness/liveness probes

1.  running a command inside a container,
2.  making an HTTP request against a container, or
3.  opening a TCP socket against a container.

init containers

• Understand container logging

k logs {podName}

• Understand how to monitor applications in Kubernetes

cAdvisor expose metrics

expose proxy

kubectl proxy > /dev/null &

Metrics-server aggregates the metrics short-term For long term persistance you can install Prometheus

• Understand debugging in Kubernetes

    kubectl run -it --rm --restart=Never alpine --image=alpine  -- /bin/sh -c 

k top node k top pod

[https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)

## 13% - Services & Networking

• Understand Services

Service provides a facade, so other can call it. Services can also abstract things outside of the cluster, for example a database.

Target Port = Port which the application is listening on (AKA node port)

    k expose deployment examplehttpapp --port 80 k get services -l app=examplehttpapp -o go-template='{{(index .items 0).spec.clusterIP}}' pod=$(k get pods -l app=examplehttpapp -o=jsonpath='{.items[0].metadata.name}')

There are three types of services: Cluster IP - Consumers can target that specific IP internally. NodePort - A node will listen into that specific port externally. Load Balancer - Cloud load balancer as ELB or Azure Load Balancer.

• Demonstrate basic understanding of NetworkPolicies

Network policies are additive

Network policies selects pods and namespace via labels selectors

Deny all Ingress
    apiVersion: networking.k8s.io/v1 kind: NetworkPolicy metadata:  
    name: deny-all-ingress spec:  
    podSelector: {} policyTypes:  
    - Ingress

Allow all ingress apiVersion: networking.k8s.io/v1 kind: NetworkPolicy metadata:  

    name: allow-all-ingress spec:  
    podSelector: {} ingress:  
    - {}  
    policyTypes:  
    - Ingress

Exercises

    k run --image=nginx --restart=Never --port=80 k run --image=nginx --restart=Never --port=80 -o yaml --dry-run > file.yaml k run --image=busybox --restart=Never -- env k run --image=busybox --restart=Never --rm -it -- env // To delete after execution
    
    k create ns mynamespace -o yaml > mynamespace.yaml
    
    k exec nginx --it -- /bin/sh -c 'echo' k exec nginx --it -- /bin/sh --it important for getting an interactive shell back
    
    k get po --label-columns app k get pod --show-labels k annotate pod {podName} description='' k label po nginx app=1
    
    k rollout history deploy nginx --revision=4
    
    k create job quick --image=busybox -- /bin/sh -c 'echo hello; sleep 30' echo world' k create cronjob quick --image=busybox -- /bin/sh -c 'echo hello; sleep 30' echo world' k logs job/jobname k get jobs -w k get po --show-labels
    
    k create cm --from-literal=var1=val1 k create secret generic lala —from-literal=var1=val2
    
    k create sa myuser
    
    k events | grep -i
    
    // Split string by : and get first position cut -f 1 -d ':' cut -f 1 -d ':'
    
    k cp busybox7:/etc/passwd /temp/passwd
