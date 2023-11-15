---
title: "Lab 4 - Authentication / API Key"
chapter: true
weight: 5
---

## Lab 4 - Authentication / API Key 

![Gloo Platform EKS Workshop Architecture Lab 4](/images/gloo-platform-eks-workshop-lab4.png)

API key authentication is one of the easiest forms of authentication to implement. Simply create a Kubernetes secret that contains the key and reference it from the **ExtAuthPolicy**. It is recommended to label the secrets so that multiple can be selected and more can be added later. You can select any header to validate against.

1. Create two secrets that Gloo will validate against. One with the api-key **admin** and the other **developer**.

    ```yaml
    kubectl apply -f - <<EOF
    apiVersion: v1
    kind: Secret
    metadata:
      name: solo-admin
      namespace: gloo-mesh-gateways
      labels:
        api-keyset: httpbin-users
    type: extauth.solo.io/apikey
    data:
      api-key: $(echo -n "admin" | base64)
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: solo-developer
      namespace: gloo-mesh-gateways
      labels:
        api-keyset: httpbin-users
    type: extauth.solo.io/apikey
    data:
      api-key: $(echo -n "developer" | base64)
    EOF
    ```

2. Create the API key **ExtAuthPolicy** that will match header **x-api-key** values againt the secrets created above. The **ExtAuthServer** resource configures
where authorization checks will be performed:

    ```yaml
    kubectl apply -f - <<EOF
    apiVersion: admin.gloo.solo.io/v2
    kind: ExtAuthServer
    metadata:
      name: ext-auth-server
      namespace: gloo-mesh-gateways
    spec:
      destinationServer:
        ref:
          cluster: cluster-1
          name: ext-auth-service
          namespace: gloo-mesh-addons
        port:
          name: grpc
    ---
    apiVersion: security.policy.gloo.solo.io/v2
    kind: ExtAuthPolicy
    metadata:
      name: products-apikey
      namespace: gloo-mesh-gateways
    spec:
      applyToRoutes:
      - route:
          labels:
            route: products
      config:
        server:
          name: ext-auth-server
          namespace: gloo-mesh-gateways
          cluster: cluster-1
        glooAuth:
          configs:
          - apiKeyAuth:
              headerName: x-api-key
              labelSelector:
                api-keyset: httpbin-users
    EOF
    ```

3. Call httpbin without an api key and you will get a **401 unauthorized message**:

    ```sh
    curl -i http://$GLOO_GATEWAY/products
    ```

4. Call httpbin with the developer api key **x-api-key: developer**:

    ```sh
    curl -H "x-api-key: developer" http://$GLOO_GATEWAY/products
    ```

5. Call httpbin with the admin api key **x-api-key: admin**:

    ```sh
    curl -H "x-api-key: admin" http://$GLOO_GATEWAY/products
    ```
