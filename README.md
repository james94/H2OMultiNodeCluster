# Install H2O Multi-Node Cluster using Kubernetes

## Setup Environment

EC2 instance Type

- Type: t2.xlarge
- number of virtual cpus: 4 cpus
- size of mem: 16GB RAM
- size of hard disk: 40 GB
- security group: ssh port 22, all traffic port 0 - 65535 both on ip 0.0.0.0/0
- ubuntu version: 18.04

~~~bash
curl -sfL https://get.k3s.io | sh -
sudo chmod 777 /etc/rancher/k3s/k3s.yaml
kubectl get nodes
~~~

## Deploy H2O Cluster in Kubernetes Cluster

Set up the h2o headless service:

~~~bash
kubectl apply -f h2o-headless-service.yaml
~~~

- The headless service enables H2O node discovery by returning the addresses of the pods and as the h2o nodes query the headless service for the addresses of the other h2o pods, h2o is able to cluster itself.

Set up the h2o statefulset:

~~~bash
kubectl apply -f h2o-statefulset.yaml
~~~

- When the statefulset of h2o nodes is created, 3 h2o pods are created each having 1 docker container installed with h2o open source with open port 54321, 1 cpu, 3GB RAM and environment variable H2O_KUBERNETES_SERVICE_DNS set, 
- This env variable makes h2o assume it's running inside a kubernetes cluster. So, h2o waits until the clustering is over before the main h2o program starts.

Let's check the Kubernetes pods:

~~~bash
kubectl get pods
~~~

It will take some time for the pods to be ready since the following is happening:

- Kubernetes downloads the docker images and then creates a docker container in each pod with h2o installed

After about 60 seconds to 2 minutes, check the h2o-stateful-set-0 pod to see that the container has been started:

~~~bash
kubectl describe pod h2o-stateful-set-0
~~~

Look at the logs of leader node of h2o cluster to see size of cluster, leader node internal IP and more:

~~~bash
kubectl logs h2o-stateful-set-0
~~~

Set up the h2o ingress to expose the h2o application service outside of kubernetes on http:

~~~bash
kubectl apply -f h2o-ingress.yaml
~~~



Look at the Kubernetes ingresses:

~~~bash
kubectl get ingresses
~~~

Kubernetes ingress controller will generate an IP address for us, which we can access in the browser to access our h2o UI.

~~~bash
<ec2-public-dns>
~~~

## References:

- [Running H2O Cluster on a Kubernetes cluster](https://www.h2o.ai/blog/running-h2o-cluster-on-a-kubernetes-cluster/)
- [h2oai/h2o-open-source-k8s](https://hub.docker.com/r/h2oai/h2o-open-source-k8s)
- [k3s: Lightweight Kubernetes](https://k3s.io/)
- [Running K3s with Multipass on Macbook](https://medium.com/@zhimin.wen/running-k3s-with-multipass-on-mac-fbd559966f7c)
- [Kubernetes: StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [Kubernetes Service: Headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)
- [Kubernetes: Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Kubernetes: App Management](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-app-management-strong-)
- [Download and install H2O3](http://h2o-release.s3.amazonaws.com/h2o/rel-zeno/2/index.html)
- [superuser: Logical vs. Physical CPU performance](https://superuser.com/questions/1105654/logical-vs-physical-cpu-performance)