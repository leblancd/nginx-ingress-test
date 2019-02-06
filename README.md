# nginx-ingress-test
The repository provides yaml files for installing an Kubernetes NGINX ingress controller that supports multiple TLS certs.

These manifests assume that you have a Kubernetes cluster that supports
Kubernetes LoadBalancer operation. If you need to spin up a cluster, you
can try instantiating a cluster with minikube and installing the metallb
loadbalancer.

#### Minikube References:
https://kubernetes.io/docs/tasks/tools/install-minikube/  
https://jason.cwik.org/2018/12/minikube-metallb.html   
https://jason.cwik.org/2019/01/metallb-on-loopback.html  

#### NGINX Ingress Controller References:
https://github.com/kubernetes/ingress-nginx  
https://kubernetes.github.io/ingress-nginx/  
https://kubernetes.github.io/ingress-nginx/deploy/  
https://kubernetes.github.io/ingress-nginx/deploy/baremetal/  
https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/multi-tls  


# Installing NGINX Ingress Controller and httpbin application

## Create Kubernetes Secrets
TLS termination in the NGINX ingress controller depends up on the creation of Kubernetes secrets that contain TLS certificates. For multiple-TLS termination, one secret is needed for every hostname that will be used for access via the ingress controller.

In this example, three hostnames will be supported (foobar.com, inside.foobar.com, and bizzbuzz.com), so three Kubernetes secrets need to be created: NOTE: This deployment includes the httpbin application (see https://github.com/istio/istio/tree/master/samples/httpbin) as a backend service to use while testing ingress controller operation.
```
# inside.foobar.com, and bizzbuzz.com
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=foobar.com"
kubectl create -n ingress-nginx secret tls foobar-secret --key /tmp/tls.key --cert /tmp/tls.crt

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=inside.foobar.com"
kubectl create -n ingress-nginx secret tls inside-foobar-secret --key /tmp/tls.key --cert /tmp/tls.crt

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=bizzbuzz.com"
kubectl create -n ingress-nginx secret tls bizzbuzz-secret --key /tmp/tls.key --cert /tmp/tls.crt


```

## Install NGINX ingress controller, ingress objects, and httpbin
Use the commands below to install:
* NGINX ingress controller and associated Kubernetes service
  (LoadBalancer type).
* The httpbin application and Kubernetes service.
* Kubectl ingress objects to allow ingress access to the httpbin application.
```
git clone https://github.com/leblancd/nginx-ingress-test.git
cd nginx-ingress-test
kubectl create -f manifests
```

TODO: Add confirmation/verification commands here.

# Add /etc/hosts Entries for Supported Hostnames

TODO: Add instrux to add this.

# Testing Multi-TLS Access
Do 'kubectl get ing' to get external IP address for the ingress controller.

Using the IP address obtained to do a curl:
```
curl https://<ingress-controller-address>/html -H 'Host:foobar.com' -kL -v
curl https://<ingress-controller-address>/delay/5 -H 'Host:foobar.com' -kL -v

curl https://<ingress-controller-address>/html -H 'Host:inside.foobar.com' -kL -v
curl https://<ingress-controller-address>/delay/5 -H 'Host:inside.foobar.com' -kL -v

curl https://<ingress-controller-address>/html -H 'Host:bizzbuzz.com' -kL -v
curl https://<ingress-controller-address>/delay/5 -H 'Host:bizzbuzz.bar.com' -kL -v
```
