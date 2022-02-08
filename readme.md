# home-k8s-cluster

This repository contains the configuration, scripts, and other goodies for
building and managing my home cluster. I share the source with you so you can
make exactly the same mistakes as I do.

## Setup

Setup the pre-commit hooks before you change anything!

```
pip install pre-commit
pre-commit install --install-hooks
pre-commit autoupdate
```

## Provision Machines

Before we interact with the cluster, we have some manual work to do.

### Manual Preparation

- Currently, my nodes are Arch Linux machines on bare metal
  - Nodes must be ready to be controlled via Ansible
    - Have `python3` installed
    - Need to be `ssh`-able from a controller (my workstation)
      - `curl -L files.lyte.dev/key.pub >> ~/.ssh/authorized_keys`

**TODO**: script this? maybe custom ISO+PXEBoot? Talos+Sidero?

### Automated Provisioning

- Setup Ansible on the controller (from `./ansible`)
  - `ansible-galaxy install -r requirements.yml --force`
- Verify Ansible can reach hosts (from `./ansible`)
  - `ansible all -i inventory/hosts.yml --list-hosts`
  - `ansible all -i inventory/hosts.yml -m ping`
- Use Ansible to build the cluster as configured on all nodes (from `./ansible`)
  - `ansible-playbook -i inventory/hosts.yml ./build-k3s-cluster`

And the cluster is up! If you want to interact with it from your controller,
you can do this:

```bash
ansible -i ansible/inventory/hosts.yml $REMOTE_HOST -m fetch \
  -a "src=/etc/rancher/k3s/k3s.yaml dest=./k3s-cluster-config.kubeconfig.yaml flat=yes"
# TODO: this did not work for me
# env KUBECONFIG="~/.kube/config:./k3s-cluster-config.kubeconfig.yaml" \
# kubectl config view --flatten | sed "s/127.0.0.1/$REMOTE_HOST/" > ~/.kube/new-config
```

- Copy the cluster information from the `./k3s-cluster-config.kubeconfig.yaml` file into
  your existing `~/.kube/config` (or just copy it there if it doesn't exist)
  - You will need to edit the host from `localhost`/`127.0.0.1` to the correct host

### Automated Teardown

```bash
ansible-playbook -i inventory/hosts.yml ./nuke-k3s-cluster
```

## Setting up Flux

- Install the `flux` CLI on a machine that can `kubectl` into the shiny, new cluster
  - `paru -S flux-bin`
  - or `curl -s https://fluxcd.io/install.sh | sudo bash`
  - https://fluxcd.io/docs/installation/
- Run the pre-flight check (you must have `~/.kube/config` setup!)
  - `flux check --pre`
- Create the `flux-system` namespace
  - `kubectl create namespace flux-system --dry-run=client -o yaml | kubectl apply -f -`
- Add the `sops-age` encryption key to the namespace
  ```bash
  pass home-k8s-cluster | grep age-secret-key | aws '{printf $2}' | \
    kubectl --namespace flux-system create secret generic sops-age \
    --from-file=age.agekey=/dev/stdin
  ```
- Install Flux
  - `kubectl apply --kustomize=
