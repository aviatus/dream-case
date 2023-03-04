### OVERVIEW

### STEP 3

9. What would you do if you were to have canary deployments? Which tool would you use? How would you manage it through the pipeline? Please explain.

There are a few options for that purpose. Native CRD's can be used if the system has a service mesh such as Linkerd, Istio, and so on.

Example for Istio:

```sh
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
name: helloworld
spec:
hosts: - helloworld
http:

- route:
  - destination:
    host: helloworld
    subset: v1
    weight: 90
  - destination:
    host: helloworld
    subset: v2
    weight: 10
```

If the system does not have a service mesh, the NGINX Ingress Controller can be used for that purpose. Such as `nginx.ingress.kubernetes.io/canary` and `nginx.ingress.kubernetes.io/canary-weight` annotations can be set in resources.

Both methods can be implemented with CI/CD tools for automation. A pipeline that specifies a different canary distribution at each stage with a manual confirmation check would be the appropriate approach.

For a Jenkins pipeline that uses `input`, confirmation can be added at each stage. In each phase, the pipeline should update the traffic distribution with annotations or CRD changes.
