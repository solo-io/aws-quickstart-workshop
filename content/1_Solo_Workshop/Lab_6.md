---
title: "Lab 6 - Traffic policies"
chapter: true
weight: 7
---

## Lab 6 - Traffic policies
![Gloo Platform EKS Workshop Architecture Lab 6](/images/gloo-platform-eks-workshop-lab6.png)

Implement intelligent routing rules for the apps in your cluster and optimize the responses to incoming requests with Gloo traffic policies. Traffic policies let you apply internal security and compliance standards to individual routes, destinations, or an entire workload so that you can enforce your networking strategy throughout your microservices architecture.

Gloo Platform supports a variety of policies to ensure network resiliency, traffic control, security, and observability for the microservices in your cluster. They can be applied to BOTH gateway and workloads in the mesh. You can apply the policies by using Kubernetes labels and selectors that match **RouteTables**, **VirtualDestinations**, or workloads.

![](/images/policies.png)

1. Let's apply a simple fault injection policy and dynamically apply it to our frontend **RouteTable** by using it's label.

    ```yaml
    kubectl apply -f - <<EOF
    apiVersion: resilience.policy.gloo.solo.io/v2
    kind: FaultInjectionPolicy
    metadata:
      name: 3sec-fault-injection
      namespace: online-boutique
    spec:
      applyToRoutes:
      - route:
          labels:
            route: frontend
      config:
        delay:
          fixedDelay: 3s
          percentage: 100
    EOF
    ```

2. Visit the Online Boutique webpage (**echo http://$GLOO_GATEWAY**). You will experience a 3 second delay.
