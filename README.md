# jellyfin
K8s jellyfin deployment

Sadly jellyfin's config/database cant use NFS due to it needing a posix file system(for file locking). Since I currently have it setup to use local storage Im just pinning it to a single worker node :(

I will probably enable/setup iscsi or maybe even an external database in the future to allow my jellyfin pod to move to other worker nodes if it needs to. Because why even put this in k8s if I can move the pod?
### Prereqs
- A K8s cluster
    - [Initial node prep](https://github.com/ChrisZ-IT/initial_k8s_node_prep)
    - [Post cluster setup /w istio ingress](https://github.com/ChrisZ-IT/base_k8s_deployment)
- [Cert manager](https://github.com/ChrisZ-IT/cert-manager)
    - I use lets encrypt for my isto SSL cert for this. But if you followed my cert manager install you can also just use a self signed cert here. Just update the `IsssuerRef > name` to `selfsigned` in `certificate.yml`

### Deployment
1. Update `values.yml`
   - Set hostname of the worker node you want to pin this pod to: `export WORKER_NODE=<worker node Hostname>`
2. Ensure the local storage path on selected worker node exists(this for config only. media can/will still be read from NFS)
3. Create the base jellyfin resources:
    - namespace: `kubectl apply -f namespace.yml`
    - storage: `kubectl apply -f storage.yml`
    - service and cert:
        ```
        export DOMAIN_NAME=<your domain name>
        cat certificate.yml | envsubst | k apply -f -
        cat service.yml | envsubst | k apply -f -
        ```
4. Deploy jellyfin:
    - I Installed via their [official helm chart](https://jellyfin.org/docs/general/installation/advanced/kubernetes/). just make sure you point to your values.yml
        - Ensure you set `WORKER_NODE` env from step 1 before installing the helm chart

### TODO
- Add/test health checks
- Move config/database to iscsi or an external database
