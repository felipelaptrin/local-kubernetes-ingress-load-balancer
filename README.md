# Local Kubernetes LoadBalancer with Ingress

This repository is a demo repository for my [personal blog post]().

In this demo repo, we are going to deploy a local Kubernetes cluster using `Kind`, create a load balancer using `MetalLB` and manage ingress using `NGINX Ingress`.

Two applications will be deployed in the cluster:
- `app-load-balancer`: Kubernetes deployment exposed by service of type `LoadBalancer`
- `app-ingress`: Kubernetes deployment exposed by a service of type `ClusterIp` that gets external traffic because of the Ingress.

## Running the demo
If using a MacOS make sure you have [docker-mac-net-connect](https://github.com/chipmk/docker-mac-net-connect) installed and running (this is explained in the blog post).

1) Create a Docker bridge network

The MetalLB IPs need to be in the range of the IPs of the cluster. I good strategy is to before the Kubernetes cluster creation to create the range of IPs we want our Kubernetes cluster to be in. This way the Kubernetes manifest for the MetalLB ip pool (`IPAddressPool`) won't need to be modified. This docker network will be used by our Kind cluster on the following step.

```sh
docker network create --subnet 172.100.0.0/16 custom-kind-network
```

2) Install Devbox dependencies of the demo

```sh
devbox shell
```

3) Create a Kind cluster and install MetalLB and Ingress NGINX

```sh
devbox run init
```

This installs and waits for the resource to be healthy so it might take a few minutes to finish this command execution.

4) Install `app-load-balancer` application

```sh
devbox run app-load-balancer
```

5) Install `app-ingress` application

```sh
devbox run app-ingress
```

6) Check access to the `app-load-balancer` application

```sh
APP_LB_IP=$(kubectl get svc app-load-balancer -n default -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Open your browser on the following URL: http://${APP_LB_IP}"
curl $APP_LB_IP
```

7) Check access to the `app-ingress` application

The `app-ingress` application is exposed by the NGINX Ingress Load Balancer and routed accordingly by the Ingress resource.

Get the NGINX Ingress IP

```sh
INGRESS_LB_IP=$(kubectl get svc ingress-nginx-controller -n ingress-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Add $INGRESS_LB_IP to /etc/hosts"
```

Now add this IP to the `/etc/hosts` file

```sh
echo "$INGRESS_LB_IP app-ingress.local" | sudo tee -a /etc/hosts
```

Now access `http://app-ingress.local`. Check if the application is accessible.

When done, remember to delete the `/etc/hosts` entry.

8) Destroy the demo