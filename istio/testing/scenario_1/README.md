# Install Istio
## Installation steps
From https://istio.io/docs/setup/consul/quick-start/
1. Go to the [Istio release page](https://github.com/istio/istio/releases) to download the installation file corresponding to your OS. 
If you are using a macOS or Linux system, you can also run the following command to download and extract the latest release automatically:
`curl -L https://git.io/getLatestIstio | sh -`
2. Extract the installation file and change the directory to the file location. The installation directory contains:
    - Sample applications in `samples/`
    - The `istioctl` client binary in the `bin/` directory. `istioctl` is used for some debug and diagnostics tasks.
    - The `istio.VERSION` configuration file
3. Add the `istioctl client` to your PATH. For example, run the following command on a macOS or Linux system:
`export PATH=$PWD/bin:$PATH`
4. Install `kubectl` using these [instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

`kubectl` is used to create, read, modify, and delete Istio API resources.

1. For Linux users, configure the `DOCKER_GATEWAY` environment variable
2. Change directory to the root of the Istio installation directory.
3. Bring up the Istio control plane containers:
    `docker-compose -f install/consul/istio.yaml up -d`
4. Confirm that all docker containers are running:
    `docker ps -a`
    - If the Istio Pilot container terminates, ensure that you ran the `kubectl config` commands below and re-run the command from the previous step.
5. Configure kubectl to use mapped local port for the API server:
```
kubectl config set-context istio --cluster=istio
kubectl config set-cluster istio --server=http://localhost:8080
kubectl config use-context istio
```

## Deploy your application
You can now deploy your own application or one of the sample applications provided with the installation like [Bookinfo](https://istio.io/docs/examples/bookinfo/).

Since there is no concept of pods in a Docker setup, the Istio sidecar runs in the same container as the application. We will use Registrator to automatically register instances of services in the Consul service registry.

The application must use HTTP/1.1 or HTTP/2.0 protocol for all its HTTP traffic because HTTP/1.0 is not supported.

See https://istio.io/docs/examples/bookinfo/

Bring up the application containers:
- If you are using a cluster with automatic sidecar injection enabled, label the `default` namespace with `istio-injection=enabled`
  - `kubectl label namespace default istio-injection=enabled`
- Then simply deploy the services using `kubectl`
   - `kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml`

Either of the above commands launches all four microservices as illustrated in the above diagram. All 3 versions of the reviews service, v1, v2, and v3, are started.
  - In a realistic deployment, new versions of a microservice are deployed over time instead of deploying all versions simultaneously.

Confirm all services and pods are correctly defined and running:
  - `kubectl get services` and `kubectl get pods`
  
## Determining the ingress IP and port 
Now that the Bookinfo services are up and running, you need to make the application accessible from outside of your Kubernetes cluster, e.g., from a browser. An Istio Gateway is used for this purpose.

1. Define the ingress gateway for the application:
  - `kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml`

2. Confirm the gateway has been created:
  - `kubectl get gateway`
  
3. Follow these [instructions](https://istio.io/docs/tasks/traffic-management/ingress/#determining-the-ingress-ip-and-ports) to set the `INGRESS_HOST` and `INGRESS_PORT` variables for accessing the gateway. Return here, when they are set.


NOTE: RUNNING INTO THIS ISSUE 
```
~/dd/integrations-core/istio/istio-1.0.1(gzu/yubikey_testing*) » kubectl get svc istio-ingressgateway -n istio-system                                                                                             gregzussa@Gregs-MacBook-Pro
Error from server (NotFound): namespaces "istio-system" not found
```

Greg M advise to use https://docs.google.com/document/d/19Ov4Y-ElR1b2i0fPUZxYqgfbc70fvslK4ZVUzB2CXM4/edit#
