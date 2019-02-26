# Running EJBCA CE on Kubernetes

This is an example setup an EJBCA installation on <https://microk8s.io/> Kubernetes.

The scope of this example is to demonstrate:

* the stateless nature of EJBCA containers in this 3 level setup.
* that EJBCA's shared database model is sufficient for clustering the application.
* demonstrating how the application can by upgraded without service down time.

A highly available service in production would additionally use

* EJBCA EE with 24/7 support by PrimeKey
* redundant highly available hardened and optimized Kubernetes platform with pro-active health checking load balancers.
* a redundant highly available hardened and optimized database like MariaDB Galera cluster.


## Prerequisites

You will need a running microk8s v1.13.3 setup with enabled Ingress module.

Additionally this example assumes that `pki.primekey.example` to be resolvable from the Ingress.
If you don't have access to a DNS server used by both the VM and your client machine, you can modify `/etc/hosts` with an appropriate IP for this DNS name.

## Setup

### Kubernetes Dashboard import

In the Kubernetes Dashboard, ensure that "All namespaces" are selected.

Locate the `+Create` button and import [ejbca-ce-with-ingress-and-mariadb.yaml](ejbca-ce-with-ingress-and-mariadb.yaml).

Once the deployment is done, you should be able to access the application at <http://pki.primekey.example/ejbca>.
The HTTPS version is also available and uses a self-signed certificate issued by the platform <https://pki.primekey.example/ejbca/adminweb/>.



## Rolling application upgrade

Since all relevant information is stored in the shared SQL database, we simply need to remove the replace the running instances with new ones.

In the Kubernetes Dashboard, locate one of the two EJBCA Pods. Delete it and watch a new Pod being created.
If you click on the new Pod you can see the message
```
pulling image "primekey/ejbca-ce"
```
which means that the latest version of the application will be used when the container instance is re-created (thanks to `imagePullPolicy: Always`).


## Using the CLI

As an alternative to the UI import, you could also use the CLI:
```
microk8s.kubectl create -f ejbca-ce-with-ingress-and-mariadb.yaml
```

To clean up all the deployed items, run
```
microk8s.kubectl delete namespaces pki-demo
```