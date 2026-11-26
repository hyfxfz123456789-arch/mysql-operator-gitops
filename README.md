# MySQL Operator GitOps

This repository holds the desired state for the existing MySQL Operator installation.

- `operator/mysql-operator/bundle.yaml`: CRD, RBAC, Service and Deployment rendered from the Operator source's `config/default` at commit `9530c94`.
- `operator/mysql-operator/manager-patch.yaml`: image reference and the paused controller replica count (`0`). Jenkins updates only the image digest on `develop` after the Harbor scan passes.
- `operator/mysql-operator/kustomization.yaml`: combines the rendered bundle and deployment patch.
- `argocd/mysql-operator-application.yaml`: Argo CD Application pointing at `develop`; apply this file separately after the repository and credentials are ready.

The existing `mysql-lab/mysqlcluster-sample` custom resource and its PVCs are deliberately absent. Keep Argo CD pruning disabled. Do not enable two Applications that manage the same Operator resources from `develop` and `main` at once. Production promotion can switch the existing Application to an approved `main` revision, or use a separate cluster.

When the CRD, RBAC, or controller Deployment templates change in the source repository, regenerate `bundle.yaml` from the approved source commit and review the diff before merging. The CI pipeline currently changes only the image digest.
