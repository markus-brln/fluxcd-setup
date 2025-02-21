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

Install flux CLI:

- Check that prerequisites are met: https://fluxcd.io/docs/installation/#prerequisites
  - When I did this at work I realized our current kubernetes version (1.28) wasn't supported by the latest flux
    version, so I needed to use an older flux version

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

- Verify installation `flux --version`
- Start k9s to see what's happening in the cluster on bootstrap

Bootstrap command (https://fluxcd.io/flux/installation/bootstrap/github/#github-personal-account):

```bash
flux bootstrap github \
  --token-auth \
  --owner=markus-brln \
  --repository=fluxcd-setup \
  --branch=main \
  --path=flux-manifests \
  --personal
```

Errors:

```yaml
 Warning  FailedCreatePodSandBox  3m20s                 kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "f72243dfc0904c1cd1d4ccf2481ecb17f324bfde4f3a1b01628f2d5bd8fda8c3": plugin type="calico" failed (
add): error getting ClusterInformation: connection is unauthorized: Unauthorized
```

- Asked copilot and the internet a bit, seemed to agree that I'd just need to restart the calico node pod
  - Matches my prio experience of microk8s in general having some issues when rebooting the host machine
  - That did the trick, flux workloads were scheduled

Output of flux bootstrap, I think I got a timeout because of the scheduling issues:

```bash
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/markus-brln/fluxcd-setup.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed component manifests to "main" ("4a50ea7982ba8e69beb535b58bdfcc72b4ffac4d")
► pushing component manifests to "https://github.com/markus-brln/fluxcd-setup.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("50ba499d1c853fc5d150ab96dbe33b8cc073c59a")
► pushing sync manifests to "https://github.com/markus-brln/fluxcd-setup.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✗ context deadline exceeded
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✗ client rate limiter Wait returned an error: context deadline exceeded
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
✗ bootstrap failed with 2 health check failure(s): [error while waiting for GitRepository to be ready: 'context deadline exceeded', error while waiting for Kustomization to be ready: 'client rate limiter Wait returned an error: context deadline exceeded']
```

I think I don't need to re-run the command or anything because the components are running and healthy and the pods
don't have any error logs - I'll just try to commit something to the repo and see if it gets picked up

Flux also successfully added its components to my repo: https://github.com/markus-brln/fluxcd-setup/tree/main/flux-manifests/flux-system



## Uninstall

```bash
flux uninstall
```


- don't over-engineer



