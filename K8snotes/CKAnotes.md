CKAnotes

# Certified Kubernetes Administrator

## Cluster Architecture

All definition files for k8s are written in [[YAML]]. Use [[yamllint]] or pyyaml to help with checking your format. http://www.yamllint.com/


Internet -> Ingress
  Internet may be a Content Delivery Network (CDN)

### Ingress
  The static IP location which proxies the more dynamically allocated Services, which proxy the very dynamically allocated VMs, and containers.

  May be of several types, the standard Ingress and igress-nginx (which is more like a load balancer). 
  I'm not sure how istio, fabric or other service meshes fit in, but likely they exist not at this layer, which is fairly static, 
  or at the pod layer, which is more at the hardware layer, but at the service layer.

  ```
  Ingress
     node1
        service1
            container1
            container2
            container3
        service2
    node2
        service1
            container4
            container5
            container6
        service2
```

Need both an ingress controller, and an ingress resource. [[Resources]] are the hardware required to do the thing. An ingress resource will include a rewrite rule 
to pass traffic to a service. [[Controller]]s are the description of the desired state of the resource, 
similar to a puppet module or ansible playbook. The Controller ensures that the right number and location of resources are available.

Example ingress controller nginx (yaml):

```
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: test-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /testpath
            pathType: Prefix
            backend:
              serviceName: test
              servicePort: 80
```

Example ingress controller (GCE):

```
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: my-ingress
    spec:
      backend:
        serviceName: my-products
        servicePort: 60001
      rules:
      - http:
          paths:
          - path: /
            backend:
              serviceName: my-products
              servicePort: 60000
          - path: /discounted
            backend:
              serviceName: my-discounted-products
              servicePort: 80
```

In both of those controllers, traffic coming into the ingress IP gets routed to http server paths (/testpath, /, /discounted), 
which are served by different services (test, my-products, my-discounted-products), which could be running on containers or
virtual machines.

Ingress abstracts away from the public internet, the particular implementation of your services. This allows for more dramatic
and flexible implementations without change on the part of the clients. It also abstracts your services from the hardware 
required to implement it, as the IP routing happens at the service level, not at the switch (hardware ports and cables) or VLAN
(overylay networks) level.

#### Ingress commands

```
kubectl
```
