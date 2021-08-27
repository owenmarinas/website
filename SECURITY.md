CKA Course Resources

CHAPTER 1.1
Course Introduction

Kubernetes Essentials
ACG Course Link: https://acloudguru.com/course/kubernetes-essentials
LA Course Link: https://linuxacademy.com/cp/modules/view/id/281?redirect_uri=https://app.linuxacademy.com/search?query=kubernetes%20essentials


CHAPTER 1.2
CKA Exam Updates

CKA Curriculum Overview
https://github.com/cncf/curriculum

CKA FAQ (k8s Version Info)
https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks#what-application-version-is-running-in-the-exam-environment


CHAPTER 2.1
K8s Basics - Quick Recap

What Is Kubernetes?
https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/


CHAPTER 2.2
K8s Architectural Overview

What Is Kubernetes?
https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

Kubernetes Components
https://kubernetes.io/docs/concepts/overview/components/

Kubernetes API
https://kubernetes.io/docs/concepts/overview/kubernetes-api/

Understanding Kubernetes Objects
https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/


CHAPTER 2.3
Building a Cluster with kubeadm

Relevant Documentation
Install Docker Engine on Ubuntu
Install Kubeadm
Creating a Single Control-Plane Cluster With Kubeadm

Lesson Reference
On all nodes, install the container runtime, Docker.

sudo apt-get update
sudo apt-get install -y \
apt-transport-https \
ca-certificates \
curl \
gnupg-agent \
software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
sudo apt-get update
sudo apt-get install -y docker-ce=5:19.03.12~3-0~ubuntu-bionic
sudo apt-mark hold docker-ce


Verify Docker is working.

sudo docker version


Disable swap.

sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


Install kubeadm, kubelet, and kubectl.

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet=1.19.1-00 kubeadm=1.19.1-00 kubectl=1.19.1-00
sudo apt-mark hold kubelet kubeadm kubectl


Create a Kubeadm configuration file.

vi config.yml


Add the basic contents of a file with no custom configuration.

apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration


On the control plane node only, initialize the cluster and set up kubectl access.

sudo kubeadm init --config config.yml
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


Verify the cluster is working.

kubectl get nodes
Install the Calico network add-on.
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml


Get the join command.

kubeadm token create --print-join-command


On all worker nodes, run the join command as root.

sudo kubeadm join ...


On the control plane node, verify all nodes in your cluster are ready.

kubectl get nodes



Install Docker Engine on Ubuntu
https://docs.docker.com/engine/install/ubuntu/

Install kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

Creating a Single Control-Plane Cluster with kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/



CHAPTER 2.5
Using Namespaces in K8s

Relevant Documentation
Namespaces (https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)


Lesson Reference

List namespaces in the cluster.

kubectl get namespaces
Specify a namespace when listing other objects such as pods.
kubectl get pods -n kube-system


Create a namespace.

kubectl create namespace my-namespace



Namespaces
https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/


CHAPTER 3.2
Introduction to High Availability in K8s

Options for Highly Available Topology
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/


CHAPTER 3.3
Introducing K8s Management Tools

Kustomize
https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/

K8s Tools
https://kubernetes.io/docs/reference/tools/



CHAPTER 3.4
Safely Draining a K8s Node

Relevant Documentation
Draining a Node
https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

Lesson Reference

Begin by creating some objects. We will examine how these objects are affected by the drain process.

First, create a pod.

vi pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  restartPolicy: OnFailure

kubectl apply -f pod.yml

Create a deployment with two replicas.

vi deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl apply -f deployment.yml

Get a list of pods. You should see the pods you just created (including the two replicas from the deployment). Take node of
which node these pods are running on.

kubectl get pods -o wide

Drain the node which the my-pod pod is running.

kubectl drain <node name> --ignore-daemonsets --force

Check your list of pods again. You should see the deployment replica pods being moved to the remaining node. The regular
pod will be deleted.

kubectl get pods -o wide

Uncordon the node to allow new pods to be scheduled there again.

kubectl uncordon <node name>

Clean Up

Delete the deployment created for this lesson.

kubectl delete deployment my-deployment



Draining a Node
https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/



CHAPTER 3.5
Upgrading K8s with kubeadm
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

Lesson Reference
First, upgrade the control plane node.

Upgrade kubeadm.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.19.2-00

kubeadm version

Drain the control plane node.

kubectl drain <control plane node name> --ignore-daemonsets

Plan the upgrade.

sudo kubeadm upgrade plan

Upgrade the control plane components.

sudo kubeadm upgrade apply v1.19.2

Uncordon the control plane node.

kubectl uncordon <control plane node name>

Upgrade kubelet and kubectl on the control plane node.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.19.2-00 kubectl=1.19.2-00

Restart kubelet.

sudo systemctl daemon-reload
sudo systemctl restart kubelet

Verify that the control plane is working.

kubectl get nodes

Upgrade the worker nodes.

Note: In a real-world scenario, you should not perform upgrades on all worker nodes at the same time. Make sure
enough nodes are available at any given time to provide uninterrupted service.

Upgrade kubeadm on the first worker node.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.19.2-00

kubeadm version

Run the following on the control plane node to drain worker node 1:

kubectl drain <worker 1 node name> --ignore-daemonsets

Upgrade the kubelet configuration on the worker node.

sudo kubeadm upgrade node

Upgrade kubelet and kubectl on the worker node.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.19.2-00 kubectl=1.19.2-00

Restart kubelet.

sudo systemctl daemon-reload
sudo systemctl restart kubelet

From the control plane node, uncordon worker node 1.

kubectl uncordon <worker 1 node name>

Repeat the upgrade process for worker node 2.

Upgrade kubeadm.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.18.6-00

kubeadm version

From the control plane node, drain worker node 2.

kubectl drain <worker 2 node name> --ignore-daemonsets

Perform the upgrade on worker node 2.

sudo kubeadm upgrade node
sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.18.6-00 kubectl=1.18.6-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet

From the control plane node, uncordon worker node 2.

kubectl uncordon <worker 2 node name>

Verify that the cluster is upgraded and working.

kubectl get nodes



Upgrading kubeadm Clusters
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


CHAPTER 3.7
Backing Up and Restoring etcd Cluster Data

Relevant Documentation

Backing Up an etcd Cluster

Lesson Reference

Look up the value for the key cluster.name in the etcd cluster. The value should be 
beebox .

ETCDCTL_API=3 etcdctl get cluster.name \
--endpoints=https://10.0.1.101:2379 \
--cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key

Back up etcd using etcdctl and the provided etcd certificates.

ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
--endpoints=https://10.0.1.101:2379 \
--cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key

Reset etcd by removing all existing etcd data.

sudo systemctl stop etcd
sudo rm -rf /var/lib/etcd

Restore the etcd data from the backup. This command spins up a temporary etcd cluster, saving the data from the backup file
to a new data directory (in the same location where the previous data directory was).

sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
--initial-cluster etcd-restore=https://10.0.1.101:2380 \
--initial-advertise-peer-urls https://10.0.1.101:2380 \
--name etcd-restore \
--data-dir /var/lib/etcd

Set ownership on the new data directory.

sudo chown -R etcd:etcd /var/lib/etcd

Start etcd.

sudo systemctl start etcd

Verify that the restored data is present by looking up the value for the key cluster.name again. The value should be beebox .

ETCDCTL_API=3 etcdctl get cluster.name \
--endpoints=https://10.0.1.101:2379 \
--cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key



Backing Up an etcd Cluster
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster



CHAPTER 4.1
Working with kubectl

Relevant Documentation
kubectl

Lesson Reference

Create a pod.

vi pod.yml

apiVersion: v1
kind: Pod
metadata:
name: my-pod
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ['sh', '-c', 'while true; do sleep 3600; done']

kubectl apply -f pod.yml

Get a list of pods.

kubectl get pods

Experiment with various output formats.

kubectl get pods -o wide
kubectl get pods -o json
kubectl get pods -o yaml

Sort results.

kubectl get pods -o wide --sort-by .spec.nodeName

Filter results by a label.

kubectl get pods -n kube-system --selector k8s-app=calico-node

Describe a pod.

kubectl describe pod my-pod

Test create/apply. Note that create will only work if the object does not already exist.

kubectl create -f pod.yml
kubectl apply -f pod.yml

Execute a command inside a pod.

kubectl exec my-pod -c busybox -- echo "Hello, world!"

Delete a pod.

kubectl delete pod my-pod



kubectl
https://kubernetes.io/docs/reference/kubectl/overview/




CHAPTER 4.3
kubectl Tips

Relevant Documentation

kubectl (https://kubernetes.io/docs/reference/kubectl/overview/)
Object Management (https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)
kubectl Cheat Sheet (https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

Lesson Reference

Run kubectl create to see a list of objects that can be created with imperative commands.
kubectl create

Create a deployment imperatively.

kubectl create deployment my-deployment --image=nginx

Do a dry run to get some sample yaml without creating the object.

kubectl create deployment my-deployment --image=nginx --dry-run -o yaml

Save the yaml to a file.

kubectl create deployment my-deployment --image=nginx --dry-run -o yaml > deployment.yml

Create the object using the file.

kubectl create -f deployment.yml

Scale a deployment and record the command.

kubectl scale deployment my-deployment replicas=5 --record
kubectl describe deployment my-deployment




CHAPTER 4.4
Managing K8s Role-Based Access Control (RBAC)

Relevant Documentation
RBAC (https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

Lesson Reference

Create a Role spec file.

vi role.yml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
namespace: default
name: pod-reader
rules:
- apiGroups: [""]
resources: ["pods", "pods/log"]
verbs: ["get", "watch", "list"]

Create the Role.

kubectl apply -f role.yml

Bind the role to the dev user.

vi rolebinding.yml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: pod-reader
namespace: default
subjects:
- kind: User
name: dev
apiGroup: rbac.authorization.k8s.io
roleRef:
kind: Role
name: pod-reader
apiGroup: rbac.authorization.k8s.io

Create the RoleBinding.

kubectl apply -f rolebinding.yml




CHAPTER 4.6
Creating ServiceAccounts

Relevant Documentation
Configure Service Accounts for Pods (https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

Using RBAC Authorization (https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

Lesson Reference

Create a basic ServiceAccount.

vi my-serviceaccount.yml

apiVersion: v1
kind: ServiceAccount
metadata:
name: my-serviceaccount

kubectl create -f my-serviceaccount.yml

Create a ServiceAccount with an imperative command.

kubectl create sa my-serviceaccount2 -n default

View your ServiceAccount.

kubectl get sa

Attach a Role to the ServiceAccount with a RoleBinding.

vi sa-pod-reader.yml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: sa-pod-reader
namespace: default
subjects:
- kind: ServiceAccount
name: my-serviceaccount
namespace: default
roleRef:
kind: Role
name: pod-reader
apiGroup: rbac.authorization.k8s.io

kubectl create -f sa-pod-reader.yml

Get additional information for the ServiceAccount.

kubectl describe sa my-serviceaccount



CHAPTER 4.7
Inspecting Pod Resource Usage

Relevant Documentation
Tools for Monitoring Resources (https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
kubectl cheat sheet (https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods)

Lesson Reference

Install Kubernetes Metrics Server.

Verify that the metrics server is responsive. Note that it may take a few minutes for the metrics server to become responsive to
requests.

kubectl get --raw /apis/metrics.k8s.io/

Create a pod to monitor.

vi pod.yml

apiVersion: v1
kind: Pod
metadata:
name: my-pod
labels:
app: metrics-test
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ['sh', '-c', 'while true; do sleep 3600; done']

kubectl apply -f pod.yml

Use kubectl top to view resource usage by pod.

kubectl top pod

Sort output with --sort-by .

kubectl top pod --sort-by cpu

Filter output by label with --selector .

kubectl top pod --selector app=metrics-test
kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-c




CHAPTER 5.2
Managing Application Configuration

Relevant Documentation
ConfigMaps (https://kubernetes.io/docs/concepts/configuration/configmap/)
Secrets (https://kubernetes.io/docs/concepts/configuration/secret/)

Lesson Reference

Create a ConfigMap.

vi my-configmap.yml

apiVersion: v1
kind: ConfigMap
metadata:
name: my-configmap
data:
key1: Hello, world!
key2: |
Test
multiple lines
more lines
kubectl create -f my-configmap.yml

View your ConfigMap data.

kubectl describe configmap my-configmap

Create a secret.

Get two base64-encoded values.

echo -n 'secret' | base64
echo -n 'anothersecret' | base64

vi my-secret.yml

Include your two base64-encoded values in the file.

apiVersion: v1
kind: Secret
metadata:
name: my-secret
type: Opaque
data:
secretkey1: <base64 String 1>
secretkey2: <base64 String 2>

kubectl create -f my-secret.yml

Create a pod and supply configuration data using environment variables.

vi env-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: env-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'echo "configmap: $CONFIGMAPVAR secret: $SECRETVAR"']
env:
- name: CONFIGMAPVAR
valueFrom:
configMapKeyRef:
name: my-configmap
key: key1
- name: SECRETVAR
valueFrom:
secretKeyRef:
name: my-secret
key: secretkey1

kubectl create -f env-pod.yml

Check the log for the pod to see your configuration values!

kubectl logs env-pod

Create a pod and supply configuration data using volumes.

vi volume-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: volume-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'while true; do sleep 3600; done']
volumeMounts:
- name: configmap-volume
mountPath: /etc/config/configmap
- name: secret-volume
mountPath: /etc/config/secret
volumes:
- name: configmap-volume
configMap:
name: my-configmap
- name: secret-volume
secret:
secretName: my-secret

kubectl create -f volume-pod.yml

Use kubectl exec to navigate inside the pod and see your mounted config data files.

kubectl exec volume-pod -- ls /etc/config/configmap
kubectl exec volume-pod -- cat /etc/config/configmap/key1
kubectl exec volume-pod -- cat /etc/config/configmap/key2
kubectl exec volume-pod -- ls /etc/config/secret
kubectl exec volume-pod -- cat /etc/config/secret/secretkey1
kubectl exec volume-pod -- cat /etc/config/secret/secretkey2




CHAPTER 5.4
Managing Container Resources

Relevant Documentation
Resource Requests and Limits (https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container)

Lesson Reference

Create a pod with resource requests that exceed aviable node resources.

vi big-request-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: big-request-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'while true; do sleep 3600; done']
resources:
requests:
cpu: "10000m"
memory: "128Mi"

kubectl create -f big-request-pod.yml

Check the pod status. It should never leave the Pending state since no worker nodes have enough resources to meet the
request.

kubectl get pod big-request-pod

Create a pod with resource requests and limits.

vi resource-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: resource-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'while true; do sleep 3600; done']
resources:
requests:
cpu: "250m"
memory: "128Mi"
limits:
cpu: "500m"
memory: "256Mi"

kubectl create -f resource-pod.yml

Check the pod status.

kubectl get pods




CHAPTER 5.5
Monitoring Container Health with Probes

Relevant Documentation
Pod Lifecycle (https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
Container Probes (https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)

Lesson Reference

Create a pod with a command-based liveness probe.

vi liveness-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: liveness-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'while true; do sleep 3600; done']
livenessProbe:
exec:
command: ["echo", "Hello, world!"]
initialDelaySeconds: 5
periodSeconds: 5

kubectl create -f liveness-pod.yml

Check the pod status.

kubectl get pod liveness-pod

Create a pod with an http-based liveness probe.

vi liveness-pod-http.yml

apiVersion: v1
kind: Pod
metadata:
name: liveness-pod-http
spec:
containers:
- name: nginx
image: nginx:1.19.1
livenessProbe:
httpGet:
path: /
port: 80
initialDelaySeconds: 5
periodSeconds: 5

kubectl create -f liveness-pod-http.yml

Check the pod status.

kubectl get pod liveness-pod-http

Create a pod with a startup probe.

vi startup-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: startup-pod
spec:
containers:
- name: nginx
image: nginx:1.19.1
startupProbe:
httpGet:
path: /
port: 80
failureThreshold: 30
periodSeconds: 10

kubectl create -f startup-pod.yml

Check the pod status.

kubectl get pod startup-pod

Create a pod with a readiness probe.

vi readiness-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: readiness-pod
spec:
containers:
- name: nginx
image: nginx:1.19.1
readinessProbe:
httpGet:
path: /
port: 80
initialDelaySeconds: 5
periodSeconds: 5

kubectl create -f readiness-pod.yml

Check the pod status.

kubectl get pod readiness-pod



CHAPTER 5.6
Building Self-Healing Pods with Restart Policies

Relevant Documentation
Restart Policy (https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)

Lesson Reference

Create a pod with the Always restart policy that completes after a few seconds.

vi always-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: always-pod
spec:
restartPolicy: Always
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10']

kubectl create -f always-pod.yml

Check the pod status. You should see it restarting after every 10-second completion.

kubectl get pod always-pod

Create a pod with the OnFailure restart policy.

vi onfailure-pod.yml
apiVersion: v1
kind: Pod
metadata:
name: onfailure-pod
spec:
restartPolicy: OnFailure
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10']

kubectl create -f onfailure-pod.yml

Check the pod status. Note that the pod does not restart because it completed successfully.

kubectl get pod onfailure-pod

Delete, modify, and recreate the pod so that it fails.

vi onfailure-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: onfailure-pod
spec:
restartPolicy: OnFailure
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']

kubectl create -f onfailure-pod.yml

Check the pod status. Note that the pod restarts because it exited with an error code.

kubectl get pod onfailure-pod

Create a pod with the Never restart policy that completes after a few seconds.

vi never-pod.yml
apiVersion: v1
kind: Pod
metadata:
name: never-pod
spec:
restartPolicy: Never
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']

kubectl create -f never-pod.yml

Check the pod status. You should see that it does not restart after completing.

kubectl get pod never-pod



CHAPTER 5.8
Creating Multi-Container Pods

Relevant Documentation
Using Pods (https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)
Shared Volumes (https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)
Patterns for Composite Containers (https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)

Lesson Reference

Create a simple multi-container Pod.

vi multi-container-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: multi-container-pod
spec:
containers:
- name: nginx
image: nginx
- name: redis
image: redis
- name: couchbase
image: couchbase

kubectl create -f multi-container-pod.yml

View your pod's status.

kubectl get pod multi-container-pod

Create a multi-container Pod that uses shared storage.

vi sidecar-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: sidecar-pod
spec:
containers:
- name: busybox1
image: busybox
command: ['sh', '-c', 'while true; do echo logs data > /output/output.log; sleep 5; done']
volumeMounts:
- name: sharedvol
mountPath: /output
- name: sidecar
image: busybox
command: ['sh', '-c', 'tail -f /input/output.log']
volumeMounts:
- name: sharedvol
mountPath: /input
volumes:
- name: sharedvol
emptyDir: {}

kubectl create -f sidecar-pod.yml

View the logs for the sidecar container.

kubectl logs sidecar-pod -c sidecar




CHAPTER 5.9
Introducing Init Containers

Relevant Documentation
Init Containers (https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

Lesson Reference

Create a pod with an init container that delays startup by 30 seconds.

vi init-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: init-pod
spec:
containers:
- name: nginx
image: nginx:1.19.1
initContainers:
- name: delay
image: busybox
command: ['sleep', '30']

kubectl create -f init-pod.yml

Check the pod status. You should see it remain in an Init state until the 30-second delay passes, then it should fully start up.

kubectl get pod init-pod



CHAPTER 6.1
Exploring K8s Scheduling

Relevant Documentation
Kubernetes Scheduler (https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)
Assigning Pods to Nodes (https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)


Lesson Reference

Log in to the Control Plane node.

List your nodes.

kubectl get nodes

Add a label to one of your worker nodes.

kubectl label nodes <node name> special=true

Create a pod that uses a nodeSelector to filter which nodes it can run on using labels.

vi nodeselector-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: nodeselector-pod
spec:
nodeSelector:
special: "true"
containers:
- name: nginx
image: nginx:1.19.1

kubectl create -f nodeselector-pod.yml

Check your Pod's status and verify that it has been scheduled on the correct node.

kubectl get pod nodeselector-pod -o wide

Create a pod that uses nodeName to bypass scheduling and run on a specific node.

vi nodename-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: nodename-pod
spec:
nodeName: <node name>
containers:
- name: nginx
image: nginx:1.19.1

kubectl create -f nodename-pod.yml

Check your Pod's status and verify that it is on the correct node.

kubectl get pod nodename-pod -o wide



CHAPTER 6.3
Using DaemonSets

Relevant Documentation
DaemonSet (https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

Lesson Reference

Log in to the Control Plane node.

Create a DaemonSet descriptor.

vi my-daemonset.yml

apiVersion: apps/v1
kind: DaemonSet
metadata:
name: my-daemonset
spec:
selector:
matchLabels:
app: my-daemonset
template:
metadata:
labels:
app: my-daemonset
spec:
containers:
- name: nginx
image: nginx:1.19.1

Create the DaemonSet in the cluster.

kubectl create -f my-daemonset.yml


Get a list of pods, and verify that a DaemonSet pod is running on each worker node.

kubectl get pods -o wide



CHAPTER 6.5
Using Static Pods

Relevant Documentation

Create static Pods (https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

Lesson Reference

Log in to one of your Worker Nodes.


Create a static pod manifest file.

sudo vi /etc/kubernetes/manifests/my-static-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: my-static-pod
spec:
containers:
- name: nginx
image: nginx:1.19.1

Restart kubelet to start the static pod.

sudo systemctl restart kubelet

Log in to the Control Plane Node.

Check the status of your static Pod.

kubectl get pods

If you wish, you can attempt to delete the static Pod using the k8s API. The Pod will be immediately re-created, since it is only a
mirror Pod created by the worker kubelet to represent the static Pod.

kubectl delete pod my-static-pod-<worker node name>



CHAPTER 7.1
K8s Deployments Overview

Relevant Documentation
Deployments (https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

Lesson Reference

Log in to your Control Plane server.

Create a deployment specification file.

vi my-deployment.yml

apiVersion: apps/v1

kind: Deployment
metadata:
name: my-deployment
spec:
replicas: 3
selector:
matchLabels:
app: my-deployment
template:
metadata:
labels:
app: my-deployment
spec:
containers:
- name: nginx
image: nginx:1.19.1
ports:
- containerPort: 80

Create the deployment.

kubectl create -f my-deployment.yml

Check the deployment status.

kubectl get deployments

Examine pods created by the deployment.

kubectl get pods




CHAPTER 7.2
Scaling Applications with Deployments

Relevant Documentation
Scaling a Deployment (https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)


Lesson Reference

Log in to your Control Plane server.

Edit a deployment spec, changing the number of replicas to scale the deployment up.

vi my-deployment.yml

...
spec:
replicas: 5

...

Update the deployment.

kubectl apply -f my-deployment.yml

Examine the deployment and it's pods to see it scale up.

kubectl get deployments
kubectl get pods

Scale the deployment back down to 3 replicas. This time, use the kubectl scale method.

kubectl scale deployment.v1.apps/my-deployment --replicas=3

Check the status of the deployment and its pods again.

kubectl get deployment
kubectl get pods



CHAPTER 7.3
Managing Rolling Updates with Deployments

Relevant Documentation

Updating a Deployment (https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

Rolling Back a Deployment (https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)

Lesson Reference

Log in to your Control Plane server.

Edit the deployment spec, changing the image version to 1.19.2 .
kubectl edit deployment my-deployment

...
spec:
containers:
- name: nginx
image: nginx:1.19.2
...

Check the rollout status, deployment status, and pods.

kubectl rollout status deployment.v1.apps/my-deployment
kubectl get deployment my-deployment
kubectl get pods

Perform another rollout, this time using the kubectl set image method. Intentionally use a 
bad image version.

kubectl set image deployment/my-deployment nginx=nginx:broken --record

Check the rollout status again. You will see the rollout unable to succeed due to a failed image pull.

kubectl rollout status deployment.v1.apps/my-deployment
kubectl get pods

Check the rollout history.

kubectl rollout history deployment.v1.apps/my-deployment

Roll back to an earlier working version with one of the following methods.

kubectl rollout undo deployment.v1.apps/my-deployment

Or:

kubectl rollout undo deployment.v1.apps/my-deployment --to-revision=<last working revision>



CHAPTER 8.1
K8s Networking Architectural Overview

Cluster Networking
https://kubernetes.io/docs/concepts/cluster-administration/networking/



CHAPTER 8.2
CNI Plugins Overview

Network Plugins
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/




CHAPTER 8.3
Understanding K8s DNS

Relevant Documentation
DNS for Services and Pods (https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)


Lesson Reference

Log in to your Control Plane server.

Create a descriptor that will set up some sample Pods which you can use to test DNS functionality.

vi dnstest-pods.yml

apiVersion: v1
kind: Pod
metadata:
name: busybox-dnstest
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ['sh', '-c', 'while true; do sleep 3600; done']
---
apiVersion: v1
kind: Pod
metadata:
name: nginx-dnstest
spec:
containers:
- name: nginx
image: nginx:1.19.2
ports:
- containerPort: 80

kubectl create -f dnstest-pods.yml

Get the IP address of the nginx-dnstest Pod.

kubectl get pods nginx-dnstest -o wide

Verify that you can reach the nginx-dnstest over the cluster network using its IP address.

kubectl exec busybox-dnstest -- curl <nginx-dnstest IP address>

Use the busybox-dnstest Pod to look up the DNS record for the nginx-dnstest Pod.

kubectl exec busybox-dnstest -- nslookup <nginx-dnstest-ip>.default.pod.cluster.local

Verify that you can reach the nginx-dnstest Pod using its domain name.

kubectl exec busybox -- curl <nginx-dnstest-ip>.default.pod.cluster.local




CHAPTER 8.5
Using NetworkPolicies

Relevant Documentation

Network Policies (https://kubernetes.io/docs/concepts/services-networking/network-policies/)

Lesson Reference

Create a new namespace.

kubectl create namespace np-test

Add a label to the Namespace.

kubectl label namespace np-test team=np-test

Create a web server Pod.

vi np-nginx.yml

apiVersion: v1
kind: Pod
metadata:
name: np-nginx
namespace: np-test
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx

kubectl create -f np-nginx.yml

Create a client Pod.

vi np-busybox.yml

apiVersion: v1
kind: Pod
metadata:
name: np-busybox
namespace: np-test
labels:
app: client
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ['sh', '-c', 'while true; do sleep 5; done']

kubectl create -f np-busybox.yml

Get the IP address of the nginx Pod and save it to an environment variable.

kubectl get pods -n np-test -o wide

NGINX_IP=<np-nginx Pod IP>

Attempt to access the nginx Pod from the client Pod. This should succeed since no 
NetworkPolicies select the client Pod.

kubectl exec -n np-test np-busybox -- curl $NGINX_IP

Create a NetworkPolicy that selects the Nginx Pod.

vi my-networkpolicy.yml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
name: my-networkpolicy
namespace: np-test
spec:
podSelector:
matchLabels:
app: nginx
policyTypes:
- Ingress
- Egress

kubectl create -f my-networkpolicy.yml

This NetworkPolicy will block all traffic to and from the Nginx Pod. Attempt to communicate with the Pod again. It should fail
this time.

kubectl exec -n np-test np-busybox -- curl $NGINX_IP

Modify the NetworkPolicy so that it allows incoming traffic on port 80 for all Pods in the 
np-test Namespace.

kubectl edit networkpolicy -n np-test my-networkpolicy

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
name: my-networkpolicy
namespace: np-test
spec:
podSelector:
matchLabels:
app: nginx
policyTypes:
- Ingress
- Egress
ingress:
- from:
- namespaceSelector:
matchLabels:
team: np-test
ports:
- port: 80
protocol: TCP

Attempt to communicate with the Pod again. This time, it should work!

kubectl exec -n np-test np-busybox -- curl $NGINX_IP





CHAPTER 9.1
K8s Services Overview

Service
https://kubernetes.io/docs/concepts/services-networking/service/



CHAPTER 9.2
Using K8s Services

Relevant Documentation
Service (https://kubernetes.io/docs/concepts/services-networking/service/)

Lesson Reference

Create a deployment.

vi deployment-svc-example.yml

apiVersion: apps/v1
kind: Deployment
metadata:
name: deployment-svc-example
spec:
replicas: 3
selector:
matchLabels:
app: svc-example
template:
metadata:
labels:
app: svc-example
spec:
containers:
- name: nginx
image: nginx:1.19.1
ports:
- containerPort: 80

kubectl create -f deployment-svc-example.yml

Create a ClusterIP Service to expose the deployment's Pods within the cluster network.

vi svc-clusterip.yml

apiVersion: v1
kind: Service
metadata:
name: svc-clusterip
spec:
type: ClusterIP
selector:
app: svc-example
ports:
- protocol: TCP
port: 80
targetPort: 80
kubectl create -f svc-clusterip.yml

Get a list of the Service's endpoints.

kubectl get endpoints svc-clusterip

Create a busybox Pod to test your service.

vi pod-svc-test.yml

apiVersion: v1
kind: Pod
metadata:
name: pod-svc-test
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ['sh', '-c', 'while true; do sleep 10; done']

kubectl create -f pod-svc-test.yml

Run a command within the busybox Pod to make a request to the service.

kubectl exec pod-svc-test -- curl svc-clusterip:80

You should see the Nginx welcome page, which is being served by one of the backend Pods created earlier using a
Deployment.

Create a NodePort Service to expose the Pods externally.

vi svc-nodeport.yml

apiVersion: v1
kind: Service
metadata:
name: svc-nodeport
spec:
type: NodePort
selector:
app: svc-example
ports:
- protocol: TCP
port: 80
targetPort: 80
nodePort: 30080

kubectl create -f svc-nodeport.yml

Test the service by making a request from your browser to http://<Control Plane Node Public IP>:30080 . You should see the Nginx welcome page.




CHAPTER 9.4
Discovering K8s Services with DNS


Relevant Documentation

DNS for Services and Pods (https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

Lesson Reference

Note: This lesson depends on objects that were created in the previous lesson. If you are following along, you will need to go
through the previous lesson first.

Get the IP address of the ClusterIP Service.

kubectl get service svc-clusterip

Perform a DNS lookup on the service from the busybox Pod.

kubectl exec pod-svc-test -- nslookup <Service IP Address>

Use the Service's IP address to make a request to the Service from the busybox pod.

kubectl exec pod-svc-test -- curl <Service IP Address>

Make a request using the service name.

kubectl exec pod-svc-test -- curl svc-clusterip

Make a request using the service fully-qualified domain name.

kubectl exec pod-svc-test -- curl svc-clusterip.default.svc.cluster.local

Create another busybox Pod in a new namespace.

kubectl create namespace new-namespace

vi pod-svc-test-new-namespace.yml

apiVersion: v1
kind: Pod
metadata:
name: pod-svc-test-new-namespace
namespace: new-namespace
spec:
containers:
- name: busybox
image: radial/busyboxplus:curl
command: ['sh', '-c', 'while true; do sleep 10; done']

kubectl create -f pod-svc-test-new-namespace.yml

Attempt to make a request to the Service from the busybox Pod that is in another 

Namespace.

kubectl exec -n new-namespace pod-svc-test-new-namespace -- curl svc-clusterip
kubectl exec -n new-namespace pod-svc-test-new-namespace -- curl svc-clusterip.default.svc.cluster.local

The request using just the Service name will fail, but it will succeed when using the fully-qualified domain name.





CHAPTER 9.6
Managing Access from Outside with K8s Ingress


Relevant Documentation
Ingress (https://kubernetes.io/docs/concepts/services-networking/ingress/)
Ingress Controllers (https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

Lesson Reference

Note: This lesson depends on objects that were created in the previous lesson. If you are following along, you will need to go
through the previous lesson first.

Create an ingress that maps to a Service.

vi my-ingress.yml

apiVersion: networking.k8s.io/v1

kind: Ingress
metadata:
name: my-ingress
spec:
rules:
- http:
paths:
- path: /somepath
pathType: Prefix
backend:
service:
name: svc-clusterip
port:
number: 80

kubectl create -f my-ingress.yml

Check the status of the ingress.

kubectl describe ingress my-ingress

Update the Service to provide a name for the Service port.

vi svc-clusterip.yml

apiVersion: v1
kind: Service
metadata:
name: svc-clusterip
spec:
type: ClusterIP
selector:
app: svc-example
ports:
- name: http
protocol: TCP
port: 80
targetPort: 80

kubectl apply -f svc-clusterip.yml

Edit the Ingress to use the port name.

vi my-ingress.yml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: my-ingress
spec:
rules:
- http:
paths:
- path: /somepath
pathType: Prefix
backend:
service:
name: svc-clusterip
port:
name: http

kubectl apply -f my-ingress.yml

Check the status of the ingress again.

kubectl describe ingress my-ingress



CHAPTER 10.1
K8s Storage Overview

Volumes
https://kubernetes.io/docs/concepts/storage/volumes/

Persistent Volumes
https://kubernetes.io/docs/concepts/storage/persistent-volumes/




CHAPTER 10.2
Using K8s Volumes

Relevant Documentation
Volumes (https://kubernetes.io/docs/concepts/storage/volumes/)

Lesson Reference

Create a Pod that uses a hostPath volume to store data on the host.

vi volume-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: volume-pod
spec:
restartPolicy: Never
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'echo Success! > /output/success.txt']
volumeMounts:
- name: my-volume
mountPath: /output
volumes:
- name: my-volume
hostPath:
path: /var/data

kubectl create -f volume-pod.yml

Check which worker node the pod is running on.

kubectl get pod volume-pod -o wide

Log in to that host and verify the contents of the output file.

cat /var/data/success.txt

Create a multi-container Pod with an emptyDir volume shared between containers.

vi shared-volume-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: shared-volume-pod
spec:
containers:
- name: busybox1
image: busybox

command: ['sh', '-c', 'while true; do echo Success! > /output/output.txt; sleep 5; done']
volumeMounts:
- name: my-volume
mountPath: /output
- name: busybox2
image: busybox
command: ['sh', '-c', 'while true; do cat /input/output.txt; sleep 5; done']
volumeMounts:
- name: my-volume
mountPath: /input
volumes:
- name: my-volume
emptyDir: {}

kubectl create -f shared-volume-pod.yml

Check the container log for busybox2. You should see the data that was generated by busybox1.

kubectl logs shared-volume-pod -c busybox2




CHAPTER 10.4
Exploring K8s Persistent Volumes

Persistent Volumes
https://kubernetes.io/docs/concepts/storage/persistent-volumes/


CHAPTER 10.5
Using K8s Persistent Volumes

Relevant Documentation
Persistent Volumes (https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

Lesson Reference

Create a StorageClass that supports volume expansion.

vi localdisk-sc.yml

apiVersion: storage.k8s.io/v1

kind: StorageClass
metadata:
name: localdisk
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true

kubectl create -f localdisk-sc.yml

Create a PersistentVolume.

vi my-pv.yml
kind: PersistentVolume
apiVersion: v1
metadata:
name: my-pv
spec:
storageClassName: localdisk
persistentVolumeReclaimPolicy: Recycle
capacity:
storage: 1Gi
accessModes:
- ReadWriteOnce
hostPath:
path: /var/output

kubectl create -f my-pv.yml

Check the status of the PersistentVolume.

kubectl get pv

Create a PersistentVolumeClaim that will bind to the PersistentVolume.

vi my-pvc.yml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: my-pvc
spec:
storageClassName: localdisk
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 100Mi

kubectl create -f my-pvc.yml

Check the status of the PersistentVolume and PersistentVolumeClaim to verify that they 
have been bound.

kubectl get pv
kubectl get pvc

Create a Pod that uses the PersistentVolumeClaim.

vi pv-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: pv-pod
spec:
restartPolicy: Never
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'echo Success! > /output/success.txt']
volumeMounts:
- name: pv-storage
mountPath: /output
volumes:
- name: pv-storage
persistentVolumeClaim:
claimName: my-pvc

kubectl create -f pv-pod.yml

Expand the PersistentVolumeClaim and record the process.

kubectl edit pvc my-pvc --record

...
spec:
...
resources:
requests:
storage: 200Mi

Delete the Pod and the PersistentVolumeClaim.

kubectl delete pod pv-pod
kubectl delete pvc my-pvc

Check the status of the PersistentVolume to verify that it has been successfully recycled and is available again.

kubectl get pv



CHAPTER 11.2
Troubleshooting Your K8s Cluster

Relevant Documentation
Troubleshooting (https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/)
Troubleshoot Clusters (https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

Lesson Reference

Log in to your K8s Control Plane node.

Get a list of nodes and view their statuses, and then examine a single node more closely:

kubectl get nodes
kubectl describe node <NODE_NAME>

Check the status of your services:

sudo systemctl status kubelet

Try logging in to a worker node and stopping and disabling the kubelet service:

sudo systemctl stop kubelet
sudo systemctl disable kubelet

Check the status of the service now that it is stopped:

sudo systemctl status kubelet

Return to the Control Plane node and run kubectl get nodes again. You may have to wait a few moments and run it a few times, but you should see the node status change.

Don't forget to return to the worker node and start and enable kubelet again!

sudo systemctl start kubelet
sudo systemctl enable kubelet

Return to the Control Plane node.

Check the status of your system Pods:

kubectl get pods -n kube-system




CHAPTER 11.3
Checking Cluster and Node Logs



Relevant Documentation

Looking at Logs (https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/#looking-at-logs)

Lesson Reference

Log in to your K8s Control Plane node.

Check the logs for your K8s services:

sudo journalctl -u kubelet

List your system pods:

kubectl get pods -n kube-system

Look for the kube-apiserver Pod and take note of its full Pod name.

Check the logs for the Kube API Server:

kubectl logs -n kube-system <KUBE-APISERVER_POD_NAME>




CHAPTER 11.5
Troubleshooting Your Applications

Relevant Documentation
Troubleshoot Applications ((https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)
Get a Shell to a Running Container (https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)
Determine the Reason for Pod Failure (https://kubernetes.io/docs/tasks/debug-application-cluster/determine-reason-pod-failure/)

Lesson Reference

Log in to your K8s Control Plane node.

Create a pod to work with:

vi troubleshooting-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: troubleshooting-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'while true; do sleep 5; done']

kubectl apply -f troubleshooting-pod.yml

List your pods:

kubectl get pods

Look more closely at a pod:

kubectl describe pod troubleshooting-pod

Run a command inside a pod's container:

kubectl exec troubleshooting-pod -c busybox -- ls

Get an interactive shell to a container:

kubectl exec troubleshooting-pod -c busybox --stdin --tty -- /bin/sh
ls
echo hello
exit



CHAPTER 11.6
Checking Container Logs


Relevant Documentation
Debug Running Pods (https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/)

Lesson Reference

Log in to your K8s Control Plane node.

Create a pod to work with:

vi logs-pod.yml

apiVersion: v1
kind: Pod
metadata:
name: logs-pod
spec:
containers:
- name: busybox
image: busybox
command: ['sh', '-c', 'while true; do echo Here is my output!; sleep 5; done']

kubectl apply -f logs-pod.yml

View the container logs:

kubectl logs logs-pod -c busybox




CHAPTER 11.8
Troubleshooting K8s Networking Issues


Checking Container Logs
Relevant Documentation
Debugging DNS Resolution (https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
Debug Services (https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
netshoot (https://github.com/nicolaka/netshoot)


Lesson Reference

Log in to your K8s Control Plane node.

Check the status of kube-proxy, and check the kube-proxy logs:

kubectl get pods -n kube-system
kubectl logs -n kube-system <kube-proxy_POD_NAME>

Check the status of the K8s DNS pods:

kubectl get pods -n kube-system

Look for pods that have names beginning with coredns .

Create a simple Nginx Pod to use for testing, as well as a service to expose it:

vi nginx-netshoot.yml

apiVersion: v1
kind: Pod
metadata:
name: nginx-netshoot
labels:
app: nginx-netshoot
spec:
containers:
- name: nginx
image: nginx:1.19.1
---

apiVersion: v1
kind: Service
metadata:
name: svc-netshoot
spec:
type: ClusterIP
selector:
app: nginx-netshoot
ports:
- protocol: TCP
port: 80
targetPort: 80

kubectl create -f nginx-netshoot.yml

Create a Pod running the netshoot image in a container:

vi netshoot.yml

apiVersion: v1
kind: Pod
metadata:
name: netshoot
spec:
containers:
- name: netshoot
image: nicolaka/netshoot
command: ['sh', '-c', 'while true; do sleep 5; done']

kubectl create -f netshoot.yml

Open an interactive shell to the netshoot container:

kubectl exec --stdin --tty netshoot -- /bin/sh

curl svc-netshoot
ping svc-netshoot
nslookup svc-netshoot


CHAPTER 13.1
About the Exam

Official Kubernetes Documentation
https://kubernetes.io/docs/home/


