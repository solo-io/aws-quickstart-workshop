---
title: "Lab 1 - Deploy Gloo Platform"
chapter: true
weight: 2
---

## Lab 1 - Deploy Gloo Platform

![Gloo Platform EKS Workshop Architecture](/images/gloo-platform-eks-workshop-lab1.png)

Gloo Platform provides a management plane to interact with the service mesh and gateways in your environment. The management plane exposes a unified API that is multi-tenant and multi-cluster aware. It is responsible for taking your supplied configuration and updating the mesh and gateways in your clusters. Included in the management plane is a UI for policy and traffic observability.

1. Set this env var to the Gloo license key.

    ```sh
    export GLOO_PLATFORM_LICENSE_KEY=<licence_key>
    ```

2. Install meshctl, the Gloo command line tool for bootstrapping Gloo Platform, registering clusters, describing configured resources, and more. Be sure to download version 2.4.4, which uses the latest Gloo Mesh installation values.

    ```sh
    curl -sL https://run.solo.io/meshctl/install | GLOO_MESH_VERSION=v2.4.4 sh -
    export PATH=$HOME/.gloo-mesh/bin:$PATH
    ```

3. Install Gloo Platform. This command uses profiles to install the control plane components, such as the management server and Prometheus server, and the data plane components, such as the agent, managed Istio service mesh, rate limit server, and external auth server, in your cluster.

    ```
    meshctl install --profiles gloo-mesh-single,ratelimit,extauth \
      --set common.cluster=cluster-1 \
      --set demo.manageAddonNamespace=true \
      --set licensing.glooMeshLicenseKey=$GLOO_PLATFORM_LICENSE_KEY
   ```

4. Wait 2-3 minutes for all components to install. Use **meshctl check** to check status. 

    ![](/images/meshctl_check.png)

5. Once everything is ready, view the Gloo Platform Dashboard:

    ```
    meshctl dashboard
    ```

    ![](/images/dashboard-1.png)

6. Wait for the Gloo Gateway Service to become ready and set it's IP address to a variable for us to use later:

    ```sh
    export GLOO_GATEWAY=$(kubectl -n gloo-mesh-gateways get svc istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].*}')
    printf "\n\nGloo Gateway available at https://$GLOO_GATEWAY\n"
    ```

Note: No application will respond to this address...yet!
