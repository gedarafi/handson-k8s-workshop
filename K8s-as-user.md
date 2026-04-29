Workshop: ”Hands-on Kubernetes as a User”

The Presenter
Daniel Medeiros
daniel.medeiros@ri.se
Researcher / MIMER and ENCCS

The Helpers (on chat)
• Lodovico Giaretta: Researcher at RISE / MIMER
• Ashwin Mohanan: Researcher at RISE / MIMER

General Guidelines
•
•
•
•
•

Write your questions in the HackMD preferably (check link on
chat), if they are not answered then we cover them during/after the
break, or at the Q&A session at the end.
Please mute yourself as we are a quite big number of people.
You should have ideally installed k3d beforehand if you want to
follow the hands-on part!
This workshop will be recorded!
Please be patient as live demos might have issues ☺

https://hackmd.io/@mimer-ai/hands-on-k8s/edit

Clone the repo! (link on chat)

https://github.com/mimer-ai/handson-k8s-workshop

Tentative Schedule
29 Apr 2026

What?
09:00 – 09:40

Theory: Introduction to Kubernetes

09:40 – 10:50

Theory 2: Concepts

10:50 – 11:00

Break

11:00 – 12:00

Hands-on 1: Jupyter, Prometheus, Grafana

12:00 – 13:15

Lunch

13:15 – 14:45

Hands-on 2: HPC and AI

14:45 – 15:00+

Q&A

What we are covering and not covering
• YES: Using Kubernetes and its user interfaces
• YES: Understanding what are Kubernetes objects and how they
work
• YES: Deploy applications and do some troubleshooting
• NO: Administering or build a production cluster (HA or non-HA)
• NO: Using specific APIs, building Operators or Device Plugins.

7

We start now ☺

8

Comparing different cloud models
Cloud IaaS

Traditional HPC

Cloud CaaS/PaaS
Container Images

(X CPU, Y Mem)

(A CPU, B Mem)

(D CPU, E Mem)

Application

Application

Application

(X CPU, Y Mem)

(A CPU, B Mem)

Dependencies

Dependencies

Dependencies

Application

Application

Guest OS

Guest OS

Guest OS

Dependencies

Dependencies

Application Layer
Operating System

Hypervisor

Host OS / Kernel

Hardware

Hardware

Hardware

Baremetal

Hypervisor

Containers

Container
Engine

9

Cgroups

We leverage control groups on Linux, while on other OS (Windows,
MacOS) we often use a full VM (e.g. Moby Linux).
10

Container engines

Image sources: project logos.

11

Container Engines – Docker layers

Layering structure

Reusing a layer for multiple images

Image sources: docker.io, dev.to.

12

Container Images
Base image
FROM debian:12-slim
RUN apt-get update --fix-missing \
&& apt-get install -y git wget cmake zip unzip \
&& apt-get install -y build-essential gfortran libopenmpi-dev
\
&& apt-get install -y ssh time python3
RUN cd /home/ && wget ftp://ftp.gromacs.org/gromacs/gromacs2023.tar.gz \
&& tar xvf gromacs-2023.tar.gz && cd gromacs-2023 \
&& mkdir build && cd build \
&& cmake .. -DGMX_BUILD_OWN_FFTW=ON DREGRESSIONTEST_DOWNLOAD=ON -DGMX_MPI=ON && make && make install
WORKDIR /home
SHELL ["/bin/bash", "-l", "-c"]
RUN cat /usr/local/gromacs/bin/GMXRC.bash > /root/.bashrc
RUN mkdir -p /var/run/sshd;
CMD /usr/sbin/sshd;

Dependencies

Application building

Post-build settings

13

Two-Stage Build
FROM golang:1.22 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o myapp .

Image 1

FROM debian:bookworm-slim
WORKDIR /app
COPY --from=builder /app/myapp .
EXPOSE 8080
CMD ["./myapp"]

Image 2

Application building

Copy app from 1

Execution

14

Architecture and Repository

Yes

Continue

No

Download
it

Check if
image
exists

This can also allow heterogeneous runs ☺
15

Docker Execution Model
•
•
•
•

Designed to be ephemeral, including storage;
PID 1 in the container is the application layer that is supposed to be executed;
Container lives while PID 1 lives.
Images are pulled from a registry (e.g. dockerhub).

OCI-Compliant

runC

User request

Docker daemon

Linux kernel
16

But this is only for
one node so far...

17

What should I use Kubernetes for?
Stockholm
Used: 20
Available: 16
Stockholm
Used: 14
Available: 4

Linköping
Used: 4
Available: 100
Has GPU

Linköping
Used: 12
Available: 14
Has GPU

Luleå
Used: 12
Available: 14

• Where to allocate?
• Deploy my application in all of
them
• Scale my application as needed

18

(Some) Kubernetes Flavors

Also several clients: Python, Go, Java, C, others…

Another orchestrators:
19

Architecture of a Kubernetes Cluster
Always need:
• api-server
• scheduler
• controller manager
• Etcd object storage
• Kubelet
• Container runtime
Addons:
• DNS
• Networking interface

20

Structure of a Kubernetes object
apiVersion:
kind:
metadata:
spec:

21

Structure of a Kubernetes object
apiVersion: v1
kind: Pod
metadata:
name: nginx
spec:
containers:
- name: nginx
image: nginx:1.14.2
ports:
- containerPort: 80

• Pods are an abstraction of resources, made
for long-running applications.
• Default pattern is to restart if fail.

22

Structure of a Kubernetes object
apiVersion: v1
• Pods are an abstraction of resources, made
kind: Pod
for long-running applications.
metadata:
• Default pattern is to restart if fail.
name: nginx
spec:
• Other relevant field is imagePullPolicy.
containers:
- name: nginx
image: nginx:1.14.2
ports:
- containerPort: 80
resources:
requests:
memory: “128Mi”
cpu: “250m”
Can also be used for custom resources,
limits:
GPUs, FPGAs, etc…
memory: “120Mi”
cpu: ”500m”

23

How the scheduler works?
Two Phases:
1. Scheduling, where the scheduler filters (based on resources, taints
and others) and scoring (uses a scoring function to determine the
best node)
2. Binding: Putting the pod to the node with the highest score

Source: kubernetes.io

24

Requests vs Limits

Source: shipit.dev

25

26: Let’s start our cluster and use our first kubectl

Create cluster in k3d:

```bash
sudo k3d cluster create my-cluster
```

## How to run kubectl as a non-root user?

```bash
cd .kube
```

```bash
sudo cp -R /root/.kube .
```

```bash
sudo chown -R $USER:$USER .kube
```

```bash
kubectl get pod --all-namespaces
```

See active nodes and pods:

```bash
sudo kubectl get nodes
sudo kubectl get pods -A
sudo kubectl get pod <pod-name> -n <namespace>
sudo kubectl describe node <node-name>
```

Create, examine, and enter into a pod:
sudo kubectl create –f <pod_A.yaml> -f <pod_B.yaml>
sudo kubectl describe pod <pod_name>
sudo kubectl exec –it <pod_name> -- /bin/bash
Delete a pod:
sudo kubectl delete –f <pod_A.yaml> -f <pod_B.yaml>
OR
sudo kubectl delete pod <pod-name>

```bash
cd 

```

26

Other interesting commands:
•
•

kubectl explain: see help
kubectl edit: edit existing object
Kubectl create

Kubectl apply

Kubectl replace

Resource doesn’t
exist

Creates it

Creates it

Fails

Resource already
exists

Fails

Update it

Replaces it

Partial update

No

Only changed
fields

Full replacement

Deletes and
recreates

No

No

Yes

Idempotent
No
(run several times,
get same result)

Yes

Yes
27

28

Attaching volumes in a pod
apiVersion: v1
kind: PersistentVolume
metadata:
name: busybox-pv
spec:
capacity:
Persistent Volume
storage: 1Gi
accessModes:
ReadWriteMany, ReadOnlyMany
- ReadWriteOnce
persistentVolumeReclaimPolicy: Retain
Delete, Recycle
storageClassName: local-path
Standard, longhorn, nfs-client, etc
hostPath:
path: /your/path/here

Persistent Volume

Attach to object

Since we are in k3d, we need to create the cluster with –volume flag
sudo k3d cluster create my-cluster --volume /home/daniel/handson-k8s-workshop/volumes:/pv-data@all 29

Attaching volumes in a pod
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: busybox-pvc
spec:
accessModes:
- ReadWriteOnce
storageClassName: local-path
resources:
requests:
storage: 1Gi

Persistent Volume

Persistent Volume

Attach to object

30

Attaching volumes in a pod
containers: .......
volumeMounts:
- name: persistent-storage
mountPath: /data
volumes:
- name: persistent-storage
persistentVolumeClaim:
claimName: busybox-pvc

Persistent Volume

Persistent Volume

Attach to object

31

32

Taints and Tolerations

Source: Zesty.co
33

Let’s test the taints and tolerations!
Create a new node in k3d:
sudo k3d node create worker -c <cluster-name>
Label the node (optional):
sudo kubectl label node <node-name> node-role.kubernetes.io/worker=
Taint the node:
sudo kubectl taint node <node-name> key1=value1:NoSchedule
34

Add the toleration
containers: .......
tolerations:
- key: "key1"
operator: "Equal"
value: "value1"
effect: "NoSchedule"

Exist
NoExecute

35

Let’s test the taints and tolerations!
Delete the taint:
sudo kubectl taint node <node-name> key1=value1-

36

Node Affinity

Source: Apptio
Source: Linuxhandbook.com

37

NodeSelector
Add a label:
sudo kubectl label node k3d-worker-0 gpu=true

spec:
nodeSelector:
gpu: "true"
Only “Equal” operator
containers:
- name: alpine
image: alpine
command: ["sleep", "3600"]

Remove a label:
sudo kubectl label node k3d-worker-0 gpu38

NodeAffinity
Add a label:
sudo kubectl label node k3d-worker-0 gpu=true

containers: .......
affinity:
nodeAffinity:
requiredDuringSchedulingIgnoredDuringExecution:
nodeSelectorTerms:
- matchExpressions:
node-role.kubernetes.io/worker
- key: gpu
Exists
operator: In
values:
- “true”

Required vs preferred: hard vs soft rule

Remove a label:
sudo kubectl label node k3d-worker-0 gpu39

40

ReplicaSet
apiVersion: apps/v1
kind: ReplicaSet
metadata:
name: nginx-rs
spec:
replicas: 3
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx

• Keeps the number of pods all the time

Pretty much a pod

41

DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
name: nginx-ds
spec:
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx

• One pod per node! (that’s why no
number of replicas here)
• Scales with the cluster nodes (one
new node = one new replica)
• Allows you to do rolling updates!

Pretty much a pod

42

Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx-deploy
spec:
replicas: 3
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx

• Extends ReplicaSet!
• Allows you to do Rolling Updates,
and Rollback.
These are for stateless applications,
where each pod is equal!

43

Namespaces
• In multi-tenant clusters, you are often
restricted to your own namespace.
• Alternatively:
kubectl create namespace my-namespace

apiVersion: v1
kind: Namespace
metadata:
name: my-namespace

Execute pods with the –n flag if necessary:
Kubectl create –f <file.yaml> -n mynamespace
Leave your namespace as default:
kubectl config set-context --current -namespace=my-namespace

44

Jobs
• Designed for finite-running tasks
• Relevant attributes:
• backOffLimit: How many times to retry
after failure
• Completions: How many times it
should succeed
• Parallelism: How many pods run in
parallel
• activeDeadlineSeconds: Kill the job
after X seconds.

apiVersion: batch/v1
kind: Job
metadata:
name: hello-job
spec:
template:
spec:
restartPolicy: Never
containers:
- name: hello
image: alpine
command:
- sh
- -c
-|
echo "Job started at $(date)"
echo "Running some work..."
sleep 5
echo "Computing something..."
echo "1 + 1 = 2"
echo "Job finished at $(date)"
backoffLimit: 3
45

Let’s test the different apps!
Create more nodes in k3d:
sudo k3d node create worker -c <cluster-name>
Then just execute the yaml files to see the replicaset and daemonset!

Check logs with:
sudo kubectl logs <pod-name>

46

Let’s test rolling out and back for Deploy!
Create the deployment:
sudo kubectl create -f 10_Deployment_with_RollingUpdates.yaml
Change the yaml file with a new version of nginx (e.g. 1.26), and:
sudo kubectl apply –f 10_Deployment_with_RollingUpdates.yaml
Now roll back to previous version:
kubectl rollout history deployment/nginx-deploy
kubectl rollout undo deployment/nginx-deploy

47

48

Kubernetes Networking Model

Image sources: inovex.de.

49

Kubernetes Networking Model - CNI

Calico
• For production environment
• Advanced network policy control
• Observability, encryption

•
•
•
•

Simple setups
Limited scalability
No network policy support
No encryption, no observability

That’s the one used defautly by k3d

50

Services – Why?
So A wants to talk with B.

Without Service:
Pod B IP = 10.0.0.2 ──► Pod B crashes and restarts
Pod B IP = 10.0.0.8 ──► Pod A is now talking to the wrong IP
With Service:
Service IP = 10.96.0.45 ──► always the same, always routes to healthy pods
It also load balances, allows DNS as well.
A pod DNS: 10-0-0-2.default.pod.cluster.local
A service DNS: nginx-svc.default.svc.cluster.local

51

Services
Every LB has a NodePort, every NodePort has a
ClusterIP. K3d has a default LB but there are
many others.
Port Range:
• ClusterIP: Any
• NodePort: 30000 – 32767
• LB: Any

52

Services
apiVersion: v1
kind: Service
metadata:
name: nginx-clusterip
spec:
type: ClusterIP
selector:
app: nginx
ports:
- name: http
protocol: TCP
port: 80
targetPort: 80

apiVersion: v1
kind: Service
metadata:
name: nginx-nodeport
spec:
type: NodePort
selector:
app: nginx
ports:
- name: http
protocol: TCP
port: 80
targetPort: 80
nodePort: 30080

apiVersion: v1
kind: Service
metadata:
name: nginx-loadbalancer
spec:
type: LoadBalancer
selector:
app: nginx
ports:
- name: http
protocol: TCP
port: 80
targetPort: 80

53

Services

Source: StackOverflow
54

Services - DNS
The full DNS:
<pod-ipv4-address>.<service-name>.<my-namespace>.svc.<cluster-domain.example>
Get pod IP via:
sudo kubectl get pod –o wide
sudo kubectl create –f 12B_Service_NodePort.yaml

Login into a non-service pod:
sudo kubectl exec –it curl-pod – sh
Run, for testing:
• curl <pod-IP>
• curl <pod-IP>.default.pod.cluster.local
• curl nginx-nodeport.default.svc.cluster.local

55

Port forward

Expose the port from a kubernetes pod/service to you.
Creates a tunnel through the K8S Api server to the pod/svc

56

Port forward
From a service:
sudo kubectl port-forward svc/nginx-nodeport 8080:80
From the deployment (random pod):
sudo kubectl port-forward deployment/nginx 8080:80
From a pod:
sudo kubectl port-forward pod/<pod-name> 8080:80

57

StatefulSets
apiVersion: apps/v1
kind: StatefulSet
metadata:
name: nginx
spec:
serviceName: nginx-headless
replicas: 3
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx
volumeMounts:
- name: data
mountPath: /data
volumeClaimTemplates:
- metadata:
name: data
spec:
accessModes: ["ReadWriteOnce"]
resources:
requests:
storage: 1Gi

apiVersion: v1
kind: Service
metadata:
name: nginx-headless
spec:
clusterIP: None
selector:
app: nginx
ports:
- port: 80

•
•
•
•
•
•

for stateful applications
each pod is numbered
always keep the same name
created/removed in sequential order
each pod has its own PVC
Needs to be connected with a headless
service (so you can ping nginx-1, nginx-2,
etc, instead of Ips that will change
regardless)

• Also supports rollout/rollback ☺

58

Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: nginx-ingress
spec:
ingressClassName: traefik
rules:
- host: nginx.local
http:
paths:
- path: /
Patch to be matched (nginx.local/) and how to match it (prefix, Exact)
pathType: Prefix
backend:
service:
name: nginx-svc
port:
number: 80

Which ingress handles the rule
Request only matches this header
Protocol

Where to forward matching traffic

Other ingress for example is the nginx one:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml
59

Ingress
Create a new k3d cluster with support to Ingress points:
sudo k3d cluster create my-cluster --port "8080:80@loadbalancer"
(if you are using a managed cluster, chances are this is already done to you)

Run:
echo "127.0.0.1 nginx.local" | sudo tee -a /etc/hosts
Then you can: curl http://nginx.local:8080 outside Kubernetes!
Alternatively, curl http://localhost:8080 -H "Host: nginx.local"

60

61

ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
name: my-service-account
--apiVersion: v1
kind: Pod
metadata:
name: sa-pod
spec:
serviceAccountName: my-service-account
containers:
- name: alpine
image: alpine
command: ["sleep", "3600"]

• An user account but for non-humans
• Have access to the Kubernetes API as well ☺
Give it permissions:
kubectl create rolebinding msa-readonly --clusterrole=view -serviceaccount=default:my-service-account -namespace=default
Login into the pod and try:
TOKEN=$(cat
/var/run/secrets/kubernetes.io/serviceaccount/token)
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.c
rt
curl -s --cacert $CACERT -H "Authorization: Bearer $TOKEN" \
https://kubernetes.default.svc/api/v1/namespaces/default/p
ods
You can even install kubectl and run there!
62

RBAC

Role-Based Access Control
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
name: pod-reader
namespace: default
rules:
- apiGroups: [""]
This is for the core group API (v1)
resources: ["pods"]
Pods, services, nodes, configmaps
verbs: ["get", "list", "watch"]
No access to: create, delete, update, patch
--apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
Matches the role to the subject:
name: pod-reader-binding
subjects:
• ServiceAccount
- kind: User
• Users
name: admin
roleRef:
• Group
kind: Role
name: pod-reader
apiGroup: rbac.authorization.k8s.io

See your user (if using kind: User):
kubectl config view --raw -o
jsonpath='{.users[0].user.clientcertificate-data}' | \ base64 -d | \ openssl
x509 -noout –subject
(Return on k3d will be CN=admin)
* You can also use a ServiceAccount, or
create new users/keys for limited
permissions as well.

63

ConfigMaps & Secrets
apiVersion: v1
kind: ConfigMap
metadata:
name: my-config
data:
APP_ENV: "production"
APP_PORT: "8080"
config.yaml: |
server:
port: 8080
debug: false

apiVersion: v1
kind: Secret
metadata:
name: my-secret
namespace: default
type: Opaque
stringData:
DB_PASSWORD: "supersecret"
API_KEY: "abc123"

• Contains data that can be read-only
mounted to a system
• Almost identical, except for encoding
(plaintext vs base64)
• Secret = sensitive, ConfigMap = not
sensitive
• Secret supports different ”types”
Run pod and check vars inside it (cmap):
echo $APP_ENV
echo $APP_PORT
Check mounted file (cmap):
cat /etc/config/config.yaml
Run pod and check vars inside it (skey):
echo $DB_PASSWORD
echo $API_KEY
Check mounted file (skey):
cat /etc/secrets/*
The base64 is in etcd! Restrict secrets with
64
RBAC!

PriorityClass
apiVersion:
scheduling.k8s.io/v1
kind: PriorityClass
metadata:
name: high-prioritynonpreempting
value: 1000000
preemptionPolicy: Never
globalDefault: false
description: "This priority
class will not cause other
pods to be preempted."

• Ensure which pods will be scheduled first ☺
• Two default priorities in k8s: system-cluster-critical and
system-node-critical

65

HorizontalPodAutoscaler
apiVersion: autoscaling/v2
apiVersion:
autoscaling/v2
kind: HorizontalPodAutoscaler
kind: HorizontalPodAutoscaler
metadata:
metadata: name: nginx-hpa
name: nginx-hpa
namespace: default
spec:
scaleTargetRef: spec:
apiVersion:scaleTargetRef:
apps/v1
apiVersion: apps/v1
kind: Deployment
name: nginx
kind: Deployment
minReplicas: name:
2
nginx
maxReplicas:
10
minReplicas:
2
metrics: maxReplicas: 10
- type: Resource
metrics:
resource:
- type: Resource
name: cpu
resource:
target:
name: cpu
type: Utilization
averageUtilization:
50
target:
type: Utilization
averageUtilization: 50

66

Testing the HPA!
Start the HPA + Pod script!
Create a load generator:
kubectl run load-generator --image=busybox \ --restart=Never \ -- sh -c "while true; do wget -q -O- http://nginx-svc; done“
Get HPA count:
kubectl get hpa nginx-hpa –w
Watch CPU usage:
watch kubectl top pods
Watch pods scaling up:
kubectl get pods -w

67

Briefly: Custom Resource Definition (CRD)
https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/

68

...Maybe?
69

70

Jupyterhub, Prometheus and Grafana

71

Helm
A package manager for Kubernetes ☺
To install:

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh

Or:

brew install helm

Or:

sudo apt-get install curl gpg apt-transport-https --yes
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee
/usr/share/keyrings/helm.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any
main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

72

Jupyterhub
helm repo add jupyterhub https://hub.jupyter.org/helm-chart/
helm repo update

helm upgrade --cleanup-on-fail \
--install <helm-release-name> jupyterhub/jupyterhub \
--namespace <k8s-namespace> \
--create-namespace \
--version=<chart-version> \
--values config.yaml
Port forward it:
kubectl port-forward svc/proxy-public 8888:80
(put an ingress if on an actual cluster)

See releases using helm list.
To remove, helm uninstall <release-name>
To see values: helm get values<release-name>

values.yaml:
hub:
config:
DummyAuthenticator:
password: "123456"
JupyterHub:
authenticator_class: dummy
More info:
https://z2jh.jupyter.org/en/stable/jupyterhub/customizi
ng/extending-jupyterhub.html
73

Prometheus Architecture

74

Prometheus Data format
Normal format:
# HELP <metric_name> <description>
# TYPE <metric_name>
<type> <metric_name>{<label_key>="<label_value>", ...} <value> [timestamp]
Examples:
# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET", status="200", service="api"} 1027
http_requests_total{method="POST", status="500", service="api"} 3
# HELP cpu_usage_percent CPU usage percentage
# TYPE cpu_usage_percent gauge
cpu_usage_percent{host="node-1"} 72.4
cpu_usage_percent{host="node-2"} 45.1

Type

Use cases

counter

Only goes up
(errors,
requests)

gauge

Goes up and
down (CPU,
memory)

histogram

Latency
distribution with
buckets

summary

Same as above
but with
quantities75

Let’s deploy Prometheus & Grafana
Install through helm:
helm install <release-name> oci://ghcr.io/prometheus-community/charts/kube-prometheus-Stack
Access Prometheus UI:
kubectl port-forward svc/<release-name>-kube-prom-prometheus 9090:9090
• Look for a metric like http_requests_total or container_cpu_usage_seconds_total ☺
• Use submetric like avg()
Access Grafana UI:
kubectl --namespace default get secrets <release-name>-grafana -o jsonpath="{.data.admin-password}" |
base64 -d ; echo
kubectl port-forward svc/<release-name>-grafana 3000:80
Kube-state-metrics is deployed as well: https://github.com/kubernetes/kube-state-metrics
...Alternatively you can scrape directly from a kubelet too via the Kubernetes API (e.g. Python, go, etc).

76

To add into Prometheus:
Push yourself:
Need to install pushgateway first!
helm install <release-name> oci://ghcr.io/prometheus-community/charts/prometheus-pushgateway
Then, every time you need to push:
kubectl port-forward svc/pushgateway-prometheus-pushgateway 9091:9091
echo "my_metric 42" | curl --data-binary @- http://localhost:9091/metrics/job/test_job

Type

Use cases

counter

Only goes up
(errors,
requests)

gauge

Goes up and
down (CPU,
memory)

histogram

Latency
distribution with
buckets

summary

Same as above
but with
quantities77

Now let Prometheus scrape it:
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
name: pushgateway-monitor
namespace: default
labels:
release: <your-release-name-for-prometheus>
spec:
selector:
matchLabels:
app.kubernetes.io/name: prometheus-pushgateway
endpoints:
- port: http
interval: 30s
path: /metrics

HPC and AI

78

Volcano and MPI (Distributed Applications)

ssh

Master:
mpiexec --allow-run-as-root -wdir
/gromacs/ --host MPI_HOST -np 4
gmx_mpi mdrun -s benchMEM.tpr
-ntomp 1 -cpi state.cpt
Workers:
/usr/sbin/sshd

79

Kubernetes YAML for jobs
apiVersion: batch/v1
kind: Job
metadata:
name: fe-job
spec:
template:
spec:
containers:
- name: fe-job
image: raijenki/mpik8s:minife
command: ["/bin/bash"]
args: ["-c", "cd
/root/miniFE/openmp/src && ./miniFE.x -nx 1000
-ny 1000 -nz 1000;"]
imagePullPolicy: Always
restartPolicy: OnFailure
(env var for OMP and resources are omitted)

80

Install Volcano!
Execute helm:
helm repo add volcano-sh https://volcano-sh.github.io/helm-charts
helm install <release-name> volcano-sh/volcano -n volcano-system --create-namespace
Run a distributed application!

81

vLLM demonstration in a real cluster ☺

82

Thank you for today!
We’ll later send an e-mail to you, feedback will be really appreciated!

83

