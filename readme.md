# k8s-clusters

This repository contains the configuration, scripts, and other goodies for
building and managing my kubernetes clusters (right now, that's just my home
cluster). I share the source with you so you can make exactly the same mistakes
as I do.

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
  pass home-k8s-cluster | grep age-secret-key | awk '{printf $2}' | \
    kubectl --namespace flux-system create secret generic sops-age \
    --from-file=age.agekey=/dev/stdin
  ```
- Install Flux
  ```
  flux bootstrap git --url=$SSH_REPO_URL --branch=master \
    --path=./cluster/home --private-key-file=$FLUX_PRIVATE_KEY_FILE
  ```

### Troubleshooting

If you screw something up here, here are some things you can do:

- `flux uninstall` will nuke flux from the cluster so you can retry from the
  beginning of this section
- If you get something like `sync path configuration ... would overwrite path ... of existing Kustomization`, you can edit the `path: ...` field in the
  `flux-system/gotk-sync.yaml` file in whatever you're passing as `--path`,
  commit, and try the bootstrap again
- You can pretty easily nuke the entire cluster and start from scratch as
  a last resort?

# To Do & Status

- How am I going to handle highly-available storage?
- `cert-manager` with CloudFlare?
- `external-dns` with CloudFlare?
- I still need to figure out my overall cluster structure
  - Since my goal is to have full redundancy, I believe I need at least
    2 control plane nodes, which since I need an odd number means 3 control
    plane nodes, and at least 2 worker nodes. This means 5 nodes total.
    I should be able to use some of my rpi4s in the cluster, probably as
    control plane nodes.
  - Where/how is storage attached?
- I need to figure out a migration plan from my current Netlify + Custom DDNS + Docker Compose setup
  - I should be able to do something like the following:
    - Setup all applications on the cluster using some dummy domain
    - Make sure everything works with the dummy domain
    - Change dummy domain to real domain
    - Change domain's nameserver to cloudflare
    - Should be all set!
- I want to look into Talos/Sidero + PXEBoot, since that could remove a lot of the ansible stuff?
- `k3s` has a decent amount of magic AFAICT, so I'd like to learn more about it
  and all its components so I better understand what my system is actually
  _doing_
