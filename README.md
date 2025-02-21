# fluxcd-setup

## Tasks

- kubernetes cluster
- FluxCD bootstrap on my cluster, 

## Install lightweight kubernetes distro

- Kubernetes distro choice:
  - already have experience with microk8s, so don't want to re-invent the wheel
  - looked at other distros, but didn't want to do a full analysis with POCs for this task

```bash
sudo snap install microk8s --classic --channel=1.31/stable
```

- How do I determine versions? At the company we take the second-to-last major version / take an LTS
  - new features, new bugs
  - still patch releases

## Bootstrap FluxCD


- Follow GitHub-specific guide: https://fluxcd.io/flux/installation/bootstrap/github/
- Make GitHub access token: https://github.com/settings/tokens
  - Scoped to just this project (apparently a new feature in GitHub)
  - A word on token expiry: In our company it became a problem that we needed so many different tokens for our gitops
    pipeline and gitlab wouldn't allow creating tokens with an expiry for longer than a year -> operational load, toil
  - Now you can make bot users with tokens that don't expire, which is also suboptimal. I wished we had enough time to
    automate this



- don't over-engineer



