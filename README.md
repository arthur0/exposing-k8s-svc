# Exposing Services using Ingress and Creating own load balancer

## Configuration:

|   Hostname   | hypothetical IP |   Virtual Macine OS    |
| kube-master  |     1.2.3.4     |  Ubuntu 16.04 server   |
| kube-minion1 |     4.3.2.1     |  Ubuntu 16.04 server   |
| kube-minion2 |     4.3.2.2     |  Ubuntu 16.04 server   |

* VMs is running in OpenStack Clouds. The master node is in different cloud of other minions. 

* Kubernetes was installed with kubeadm: https://kubernetes.io/docs/getting-started-guides/kubeadm/

* Flannel Pod network (step 2 of instalation)  with `--pod-network-cidr 10.244.0.0/16` option 


## Prerequisites

* Kubernetes 1.2 and later (TLS support for Ingress has been added in 1.2)


## Running the Example

## 1. Deploy the Cafe Application

1. Create the coffee and the tea deployments:

  ```
  $ kubectl create -f deployments/tea-deploy.yaml
  $ kubectl create -f deployments/coffee-deploy.yaml
  ```

To view your deployments: `$ kubectl get deployments` or  `$ kubectl get deploy`

  ```
  NAME            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  coffee-deploy   2         2         2            2           32m
  tea-deploy      3         3         3            3           41m
  ```
  

2. Create services 
  
* From `expose` command:

  ```
  $ kubectl expose deployment coffee --port=80 --target-port=80 --type=NodePort
  $ kubectl expose deployment tea--port=80 --target-port=80 --type=NodePort
  ```

OR

* From files:

  ```
  $ kubectl create -f services/tea-svc.yaml
  $ kubectl create -f services/coffee-svc.yaml
  ```

This will expose your service on high level port,  randomly, of all your nodes (default to 30000–32767).
To view your services: `$ kubectl get services` or  `$ kubectl get svc`

  ```
  NAME         CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
  coffee-svc   10.105.32.58   <nodes>       80:31519/TCP   8s
  kubernetes   10.96.0.1      <none>        443/TCP        1d
  tea-svc      10.97.215.15   <nodes>       80:31785/TCP   3s
  ```

Now, we can view the ports `31519` and `31785` for the coffee-svc and tea-svc services, respectively.
We can too find this ports describing the service and filtering the information by 'NodePort':

  ```
  $ kubectl describe svc coffee-svc | grep NodePort
  Type:			NodePort
  NodePort:		http	31519/TCP
  ```

NodePort is the random port that the service was mapped to. If you create your service from the file, it's important the spec `type: NodePort` as well the flag '--type=NodePort' if you use `expose` command.

