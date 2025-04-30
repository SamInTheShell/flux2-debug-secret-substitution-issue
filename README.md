# Flux2 Secret Substitution Issue
If you create a `Secret` or `ConfigMap` to be used for substitution where any value starts with `*`, the Kustomization controller will fail to apply the config.

The resulting error message is as follows.
```plaintext
{"level":"error","ts":"2025-04-30T21:54:42.778Z","msg":"Reconciliation failed after 43.484292ms, next try in 10m0s","controller":"kustomization","controllerGroup":"kustomize.toolkit.fluxcd.io","controllerKind":"Kustomization","Kustomization":{"name":"gitops","namespace":"flux-system"},"namespace":"flux-system","name":"gitops","reconcileID":"793e0b4f-b0eb-4cdc-aa3a-09d724a93879","revision":"main@sha1:dc1ae29cc1cc490c6cce8b6402cf406b40b4e9fb","error":"post build failed for 'my-awesome-configmap': envsubst error: YAMLToJSON: yaml: unknown anchor '-this-is-a-secret' referenced"}
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
helm --namespace flux-system upgrade flux2 flux2 --install --create-namespace --version '~2.14.0' --repo https://fluxcd-community.github.io/helm-charts --wait
```

5. Create the source pointing toward your forked copy of this repository. This will copy your generated SSH key into a cluster secret.

```shell
flux --namespace flux-system create source git gitops --branch main --url ssh://git@github.com/samintheshell/flux2-debug-secret-substitution-issue.git --private-key-file ~/.ssh/gitops -s
```

6. Create the kustomization that deploys resources and secret to use as references in the repository. (It fails in this example to illustrate the error)

```shell
kubectl -n flux-system apply -f ./gitops.yaml
```

7. See the failure in action.
```shell
% kubectl -n flux-system get kustomizations
NAME     AGE     READY   STATUS
gitops   2m17s   False   post build failed for 'my-awesome-configmap': envsubst error: YAMLToJSON: yaml: unknown anchor '-this-is-a-secret' referenced
```

## Additional Notes
Another form of this error occurs if you do `*.some-string` instead of `*-some-string`.
```plaintext
{"level":"error","ts":"2025-04-30T21:32:32.792Z","msg":"Reconciliation failed after 178.632044ms, next try in 10m0s","controller":"kustomization","controllerGroup":"kustomize.toolkit.fluxcd.io","controllerKind":"Kustomization","Kustomization":{"name":"gitops","namespace":"flux-debug"},"namespace":"flux-debug","name":"gitops","reconcileID":"a611ae29-05ff-4e3d-85e5-4116ac7002a4","revision":"main@sha1:05fcae64958b769c1f9c69092c3bf4f4d0e4bad3","error":"post build failed for 'ingress-nginx': envsubst error: YAMLToJSON: yaml: line 30: did not find expected alphabetic or numeric character"}
```

## Update
Known issue. Is documented here: https://fluxcd.io/flux/components/kustomize/kustomizations/#post-build-substitution-of-numbers-and-booleans

TLDR: You just need to do `${quote}${MY_SECRET}${quote}` in your gitops manifests.
