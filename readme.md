# k8s-clusters

[🖥️ Upstream][upstream] • [🐙 GitHub Mirror][github]

This repository contains the configuration, scripts, and other goodies for
building and managing my kubernetes clusters. I share the source with you so
you can make exactly the same mistakes as I do.

## Setup

Setup the pre-commit hooks before you change anything!

```
pip install pre-commit
pre-commit install --install-hooks
pre-commit autoupdate
```

## Usage

See each cluster's individual readme for more information

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

[upstream]: https://git.lyte.dev/lytedev/k8s-clusters
[github]: https://github.com/lytedev/k8s-clusters
