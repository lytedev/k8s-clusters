# Provision Machines

- My nodes are Arch Linux machines on bare metal
  - Have `python3` installed for Ansible
  - Need to be `ssh`-able from a workstation
- Using Ansible
  - Setup [Ansible inventory][ainv] under the key `homek8s`
    ```
    homek8s:
      children:
        controllers:
        hybrid:
          hosts:
            root@kubeworker1:
        workers:
    ```
  - Test with `ansible homek8s -m ping` or groups like this `ansible homek8s:hybrid -m ping`

[ainv]: https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html#selecting-machines-from-inventory
