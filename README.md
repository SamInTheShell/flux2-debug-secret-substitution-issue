# Flux2 Secret Substitution Issue
If you create a `Secret` or `ConfigMap` to be used for substitution where any value starts with `*`, the Kustomization controller will fail to apply the config.

The resulting error message is as follows.
```plaintext
{"level":"error","ts":"2025-04-30T21:32:32.792Z","msg":"Reconciliation failed after 178.632044ms, next try in 10m0s","controller":"kustomization","controllerGroup":"kustomize.toolkit.fluxcd.io","controllerKind":"Kustomization","Kustomization":{"name":"gitops","namespace":"flux-debug"},"namespace":"flux-debug","name":"gitops","reconcileID":"a611ae29-05ff-4e3d-85e5-4116ac7002a4","revision":"main@sha1:05fcae64958b769c1f9c69092c3bf4f4d0e4bad3","error":"post build failed for 'ingress-nginx': envsubst error: YAMLToJSON: yaml: line 30: did not find expected alphabetic or numeric character"}
```

## Reproducing the Error
1. Fork this repo.

2. Generate a deployment key for your fork.

```shell
ssh-keygen -t rsa -b 4096 -C gitops -f ~/.ssh/gitops -N ''
```

3. Go to your fork's settings and add the deployment key stored at `~/.ssh/gitops.pub`.

4. Install flux2.

```shell
helm --namespace flux-debug upgrade flux2 flux2 --install --create-namespace --version '~2.14.0' --repo https://fluxcd-community.github.io/helm-charts --wait
```

5. Create the source pointing toward your forked copy of this repository. This will copy your generated SSH key into a cluster secret.

```shell
flux --namespace flux-debug create source git gitops --branch main --url ssh://git@github.com/samintheshell/flux2-debug-secret-substitution-issue.git --private-key-file ~/.ssh/gitops -s
```

6. Create the kustomization that deploys resources and secret to use as references in the repository. (It fails in this example to illustrate the error)

```shell
kubectl -n flux-debug apply -f ./gitops.yaml
```
