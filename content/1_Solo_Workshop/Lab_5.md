---
title: "Lab 5 - Zero Trust"
chapter: true
weight: 6
---

## Lab 5 - Zero Trust

![Gloo Platform EKS Workshop Architecture Lab 5](/images/gloo-platform-eks-workshop-lab5.png)

Lets enforce a **Zero Trust** networking approach where all inbound traffic to any applications is denied by default.

1. Add a default deny-all policy to the backend-apis-team workspace:

    ```yaml
    cat << EOF | kubectl apply -f -
    apiVersion: security.policy.gloo.solo.io/v2
    kind: AccessPolicy
    metadata:
      name: allow-nothing
      namespace: online-boutique
    spec:
      applyToWorkloads:
      - selector:
          namespace: online-boutique
      config:
        authn:
          tlsMode: STRICT
        authz: {}
    EOF
    ```

2. Refresh the Online Boutique webpage (**echo http://$GLOO_GATEWAY**). You should see an error with message "RBAC: access denied"

3. Add AccessPolicy to explicitly allow traffic between the gateway and the frontend application:

    ```yaml
    kubectl apply -f - <<EOF
    apiVersion: security.policy.gloo.solo.io/v2
    kind: AccessPolicy
    metadata:
      name: frontend-api-access
      namespace: online-boutique
    spec:
      applyToDestinations:
      - selector:
          labels: 
            app: frontend
      config:
        authz:
          allowedClients:
          - serviceAccountSelector:
              name: istio-ingressgateway-1-18-2-service-account
              namespace: gloo-mesh-gateways
    EOF
    ```

3. Add AccessPolicy to explicitly allow traffic between the microservices online-boutique workspace. As you can see, these policies can be very flexible.

    ```yaml
    kubectl apply -f - <<EOF
    apiVersion: security.policy.gloo.solo.io/v2
    kind: AccessPolicy
    metadata:
      name: in-namespace-access
      namespace: online-boutique
    spec:
      applyToDestinations:
      - selector:
          namespace: online-boutique
      config:
        authz:
          allowedClients:
          - serviceAccountSelector:
              namespace: online-boutique
    EOF
    ```
