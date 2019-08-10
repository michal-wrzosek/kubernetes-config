# My Kubernetes config

### Setup:
- mashcrisp.com domain
- Cloudflare
- 3 node kubernetes cluster in GKE
- Google Container Registry

### Services:
- express-test-app
- kubernetes-cloudflare-sync (https://github.com/calebdoxsey/kubernetes-cloudflare-sync)
- nginx

### Google Setup

- https://console.cloud.google.com
- project name: mw-infra
- zone: europe-west3-a
- cluster: super-cheap-cluster-1

**Cluster and node pool**
- 3-node pool using the cheapest instance type (f1-micro).
- boot disk size 10GB,
- preemptible nodes (they're cheaper)
- auto-upgrade and auto-repair
- disable HTTP load balancing
- disable all the StackDriver stuff
- disabling the kubernetes dashboard

**VPC Network > Firewall Rules > Add "http" rule:**
- Ingress
- Allow all instances in the network
- IP range of 0.0.0.0/0
- tcp:80,443,8080

**Local setup:**
- Install SDK https://cloud.google.com/sdk/docs
- `gcloud auth login`
- Install Docker
- `gcloud auth configure-docker`
- `gcloud components install kubectl`
- `gcloud config set project mw-infra`
- `gcloud config set compute/zone europe-west3-a`
- `gcloud container clusters get-credentials super-cheap-cluster-1`

### express-test-app service
To build image:
```
docker build -t gcr.io/mw-infra/express-test-app:latest .
```

To run locally:
```
docker run -p 8080:8080 gcr.io/mw-infra/express-test-app:latest
```

To push to GCR:
```
docker push gcr.io/mw-infra/express-test-app:latest
```

### nginx service
To find public ips run:
```
kubectl get node -o yaml
```
and look for:
```
 - address: ...
   type: ExternalIP
```

### kubernetes-cloudflare-sync
https://github.com/calebdoxsey/kubernetes-cloudflare-sync

To set secrets for Cloudflare:
`kubectl create secret generic cloudflare --from-literal=email='EMAIL' --from-literal=api-key='API_KEY'`

Additionaly before applying this service:
`kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user YOUR_EMAIL_ADDRESS_HERE`

### Useful commands

To apply this configuration run:
`kubectl apply -f .`

To test that it's running:
`kubectl get pod`

And we can also create a proxy API so that we can access it:
`kubectl proxy`

And then visit:
http://localhost:8001/api/v1/namespaces/default/services/express-test-app/proxy/

---

This project was built based on this awesome tutorial:

http://www.doxsey.net/blog/kubernetes--the-surprisingly-affordable-platform-for-personal-projects
