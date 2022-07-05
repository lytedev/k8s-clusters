# Home Cluster

## TODO

- **Netboot**: https://www.sidero.dev/v0.5/getting-started/prereq-dhcp/
  - Can probably leverage `dnsmasq` on the router for this?

## Setup

### Networking

- Prepare networking
  - Internally:
    - Add a DNS entry for the cluster endpoint (router's `/etc/hosts` + `dnsmasq`) to point to the initial node
  - Externally:
    - Add a DNS entry for the cluster endpoint to point to the router
    - Setup the router to forward external requests to the initial node

### Setup Kubernetes Cluster

> **Source**: https://www.talos.dev/v1.1/introduction/getting-started/

- Boot the Talos image on the initial node
- If you are not using _this_ configuration:
  - `talosctl gen config "cluster-name" "cluster-endpoint"`
  - Edit files as needed
- Apply the control plane config to the initial node
  - `talosctl apply-config --insecure --nodes "$INITIAL_NODE_ADDR" --file controlplane.yaml`
  - You will need to wait a bit for the configuration to be applied, Talos to
    install itself, for the node to reboot, and for post-boot initialization
- Setup the client to communicate with the newly-configured node
  - `talosctl --talosconfig=./talosconfig config endpoint "$INITIAL_NODE_ADDR"`
    - Optionally also make this the default with `talosctl config merge ./talosconfig`
- Bootstrap the cluster
  - `talosctl bootstrap --nodes "$INITIAL_NODE_ADDR"`
  - You will need to wait a bit for Kubernetes to initialize
- Pull down the kubeconfig
  - `talosctl kubeconfig`

Once the cluster has finished initializing _and starting up_, you should be
able to `kubectl get nodes`.

#### Adding Nodes

> **Note**: UNTESTED

- Boot the Talos ISO on the target node
- Apply the appropriate configuration to the target node
  - `talosctl apply-config --insecure --nodes "$TARGET_NODE_ADDR" --file controlplane.yaml`
  - You will need to wait a bit for Kubernetes to initialize, start up, and
    then join the cluster
- Add the node to `talosconfig` as needed

#### Untaint Masters

Since we're "frugal" (cheap) and we want to use all the hardware for all the
things:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Apply Initialization Manifests

```bash
kubectl apply -k manifests/initialization
```

### Setting up GitOps

**TODO**

### Storage

**TODO**

## Load Balancing

I can _probably_ handle this with my router?

**TODO**
