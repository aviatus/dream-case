# DREAM CASE

### STEP 1

In that case, the given `sample-java-app` is used as a demo Java project. The image of the project has 2 stages: build and runtime. During the building phase, Maven is preferred. Another user from root was created for security during runtime. Then the init process (`tini`) is installed for zombie process handling. The system runs on that init process. Created image pushed to `aviatus/dream-app` on Dockerhub.

### STEP 2

1. Create a Production Ready Kubernetes Cluster via Kubeadm, Kubespray etc.

Kubeadm is used for the installation of Kubernetes on three nodes. The three nodes were created on AWS EC2. Ansible Playbooks are created to provide simplicity of the process. Containerd is picked for the default container runtime interface. Docker is installed on the third node for the Jenkins Docker build process. In order to create space for other deployments, the schedule taint is removed from the control plane. After the cluster is created, the second node is tainted with the custom role: Jenkins expression.

2. Deploy Jenkins on K8s.

Based on https://charts.jenkins.io/jenkins, a Helm chart was created that identifies tolerations and affinity rules for each component. Also, the `role: jenkins` label was applied to the third node. To run docker build commands, Jenkins agents mounted (docker.sock) on node.

3. Deploy Elasticsearch on K8s for Kubernetes & application logging purposes.

The cluster uses the ELK stack with https communication. CA certificate and elastic credentials are injected with logstash and kibana configurations as secrets. Elastic search is limited to a single node for system architecture performance requirements.

4. Deploy Prometheus & Grafana on K8s for Kubernetes and application monitoring purposes.

Helm chart was created which is dependent on `prometheus-community/prometheus` and `grafana/grafana` with modifications for app monitoring. Prometheus data source added to Grafana and created a Grafana dashboard.

![Grafana Screenshot] (assets/grafana-dashboard.png)

https://snapshots.raintank.io/dashboard/snapshot/DgBzZvFuHupSaPUDRetbu3nRU42JojHZ

### STEP 3

1. Create the required Kubernetes manifests (deployment, service, ingress, secret, HPA etc.)

2. Application should be running on both worker nodes with at least 4 pods and service will be balanced behind NGINX.

3. Make sure the application receives requests as soon as it is ready and restarts if there is a problem.

The dream app helm files were created with deployment, service, ingress, and HPA. Antipod affinity was added to the cluster for distributed topology. The Nginx Ingress controller service type was changed to `NodePort` for the system environment. The Helm chart is modified for the Nginx ingress controller. The packet is analyzed with `kubescore` and reasonable improvements are added (resource, limits, initDelay, etc.).

4. Create a build pipeline for the application via Jenkins file.

Jenkins Docker and GitHub plugins installed as prerequisites. The GitHub and Docker access tokens were added to Jenkins credentials for security reasons. The pipeline pulls the image, builds the project with docker, and push to the designated dockerhub repository. The mentioned file can be found in the app folder.

5. Create a deployment pipeline for the application via Jenkins file.

Jenkins agent used with docker ansible image which is `quay.io/ansible/ansible-runner:stable-2.12-latest`. One of the prerequisites of the pipeline is the `ssh-agent` plugin for ssh key security reasons. The private ssh key is stored and used with Jenkins credentials via `ssh-agent`. The pipeline created for deployment is modest. The deploy pipeline can be modified in many ways to provide flexibility, simplicity, and security. The ansible package can be updated with a more generic version for resources and namespaces. Another approach would be creating a certificate for the kube-api-server and storing it in Jenkins credentials instead of the ssh key. After that, the agent can run the rollout command.

6. What would you do if you were to have a canary deployment? Which tool would you use? How would you manage it through the pipeline? Please explain.

There are a few options for doing that. Native CRDs can be used if the system has a service mesh such as Linkerd, Istio, and so on.

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

7. Write a custom validation webhook so that if a deployment does not have resource requests specified, it should fail.

The `cert-manager` is used for validation webhook certification for kube-api server communication. The validation webhook checks the deployment containers and rejects deployments where limits and requests are not set. Webhook Docker image pushed to `aviatus/validating-webhook`.

Note: Some of the items on the list developed on the Kind cluster because of poor virtualbox experience on arm64. The rest are natively built on ec2 instances.
