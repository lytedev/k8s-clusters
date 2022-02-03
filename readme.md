# Provision Machines

- My nodes are Arch Linux machines on bare metal
  - Nodes must be ready to be controlled via Ansible
    - Have `python3` installed for Ansible
    - Need to be `ssh`-able from a workstation
      - You can grab keys like so: `curl -L files.lyte.dev/key.pub >> ~/.ssh/authorized_keys`
    - TODO: script this? maybe custom ISO+PXEBoot?
- Setup Ansible on the controller (from `./ansible`)
  - `ansible-galaxy install -r requirements.yml --force`
- Verify Ansible can reach hosts (from `./ansible`)
  - `ansible all -i inventory/hosts.yml --list-hosts`
  - `ansible all -i inventory/hosts.yml -m ping`
- Use Ansible to prepare hosts for k3s installation (from `./ansible`)
  - `ansible-playbook -i inventory/hosts.yml ./provision.yml`
- Use Ansible to install k3s as configured on all nodes (from `./ansible`)
  - `ansible-playbook -i inventory/hosts.yml ./install-k3s.yml`
    - You have to run this multiple times for the worker nodes to successfully
      connect to the control plane nodes
- Setup your local kubectl to work with the new cluster
  - `ansible -i ansible/inventory/hosts.yml $REMOTE_HOST -m fetch -a "src=/etc/rancher/k3s/k3s.yaml dest=./k3s-cluster-config.yaml flat=yes"`
  - Copy the cluster information from the `./k3s-cluster-config.yaml` file into
    your existing `~/.kube/config` (or just copy it there if it doesn't exist)
    - You will need to edit the host from `localhost`/`127.0.0.1` to the correct host
