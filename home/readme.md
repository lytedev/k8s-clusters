# Home Cluster

## TODO

- Netboot: https://www.sidero.dev/v0.5/getting-started/prereq-dhcp/
  - Should be able to use `dnsmasq` for this?

## Setup

You will need the following:

- `argocd`
- `talosctl`
- `kubectl`

### Networking Preparations

- Internally:
  - Add a DNS entry for the cluster endpoint (router's `/etc/hosts` and
    `dnsmasq`) to point to the initial node
  - If using MetalLB, Ensure that ARP tables for receiving machines will have
    a route matching your desired CIDR
- Externally:
  - Add a DNS entry for the cluster endpoint to point to the router
  - Setup the router to forward external requests to the initial node

### Initialize Kubernetes Cluster

> Source: https://www.talos.dev/v1.1/introduction/getting-started/

- Boot the Talos ISO on the initial node
- Prepare your talos config
  - `cd talos`
  - Or if you are not using _this_ configuration:
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
- Optionally enable the control plane node(s) to run pods
  - `kubectl taint nodes --all node-role.kubernetes.io/master-`

### Adding Nodes

- Boot the Talos ISO on the target node
- Apply the appropriate configuration to the target node
  - `talosctl apply-config --insecure --nodes "$TARGET_NODE_ADDR" --file controlplane.yaml`
  - You will need to wait a bit for Kubernetes to initialize, start up, and
    then join the cluster
- Add the node to `talosconfig` as needed
  - For another control plane node: `talosctl --talosconfig=./talosconfig config endpoint "$TARGET_NODE_ADDR"`
  - Otherwise: `talosctl --talosconfig=./talosconfig config endpoint "$TARGET_NODE_ADDR"` (untested)
- Add the node to the internal DNS entry for the cluster endpoint
- Optionally enable the control plane node(s) to run work
  - `kubectl taint nodes --all node-role.kubernetes.io/master-`

Once the cluster has finished initializing _and starting up_, you should be
able to `kubectl get nodes`.

### Setup GitOps

Now we should avoid changing the cluster manually so that all changes are
captured in source control. We do this with flux:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Storage

- TODO?

## Load Balancing

I can _probably_ handle this with my router?

- TODO?
