---
title: "Lab 3 - Routing workloads"
chapter: true
weight: 4
---

## Lab 3 - Routing to other workloads

![Gloo Platform EKS Workshop Architecture Lab 3](/images/gloo-platform-eks-workshop-lab3.png)

1. Lets see how easy it is to expose another application. This time, we will match on URI **prefix: /products** and send to the productcatalogservice application.

    ```yaml
    kubectl apply -f - <<EOF
    apiVersion: networking.gloo.solo.io/v2
    kind: RouteTable
    metadata:
      name: productcatalog
      namespace: online-boutique
    spec:
      weight: 100
      workloadSelectors: []
      http:
        - matchers:
          - uri:
              exact: /products
          - uri:
              prefix: /products
          name: products
          labels:
            route: products
          forwardTo:
            destinations:
              - ref:
                  name: productcatalogservice
                  namespace: online-boutique
                port:
                  number: 3555
    EOF
    ```

2. Get products from the Product Catalog API

    ```sh
    curl $GLOO_GATEWAY/products
    ```
   ![Expected Output](/images/products_output.png)

3. Next, lets route to an endpoint (http://httpbin.org) that is external to the cluster. **ExternalService** resource defines a service that exists outside of the mesh. ExternalServices provide a mechanism to tell Gloo Platform about its existance and how it should be communicated with. Once an ExternalService is created, a **RouteTable** can be used to send traffic to it. In this example, we will send traffic on URI **prefix: /httpbin** to this external service.

    ```yaml
    kubectl apply -f - <<EOF
    apiVersion: networking.gloo.solo.io/v2
    kind: ExternalService
    metadata:
      name: httpbin
      namespace: online-boutique
    spec:
      hosts:
      - httpbin.org
      ports:
      - name: https
        number: 443
        protocol: HTTPS
        clientsideTls: {}   ### upgrade outbound call to HTTPS
    EOF
    ```

4. Create a new **RouteTable** that will match on requests containing the prefix **/httpbin** and route it to the httpbin **ExternalService**. You may have also noticed that we are rewriting the path using **pathRewrite: /** because httpbin.org is listening for **/get**.

    ```yaml
    kubectl apply -f - <<'EOF'
    apiVersion: networking.gloo.solo.io/v2
    kind: RouteTable
    metadata:
      name: httpbin
      namespace: online-boutique
    spec:
      weight: 150
      workloadSelectors: []
      http:
        - matchers:
          - uri:
              prefix: /httpbin
          name: httpbin-all
          labels:
            route: httpbin
          forwardTo:
            pathRewrite: /
            destinations:
            - ref:
                name: httpbin
              port: 
                number: 443
              kind: EXTERNAL_SERVICE
    EOF
    ```

5. Let's test it.

    ```sh
    curl -v $GLOO_GATEWAY/httpbin/get
    ```

   ![Expected Output](/images/httpbin_output.png)