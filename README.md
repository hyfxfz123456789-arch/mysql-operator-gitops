# MySQL Operator GitOps

This repository holds the desired state for the existing MySQL Operator installation.

- `operator/mysql-operator/bundle.yaml`: CRD, RBAC, Service and Deployment rendered from the Operator source's `config/default` at commit `9530c94`.
- `operator/mysql-operator/manager-patch.yaml`: image reference and controller replica count. Jenkins updates the image digest on `develop` after the Harbor scan completes; the current scan reports vulnerabilities but does not block promotion. Replica changes remain manual.
- `operator/mysql-operator/kustomization.yaml`: combines the rendered bundle and deployment patch.
- `argocd/mysql-operator-application.yaml`: Argo CD Application pointing at `develop`; apply this file separately after the repository and credentials are ready.
- `instances/mysql-lab/mysqlcluster.yaml`: desired state of the existing `mysql-lab/mysqlcluster-sample` CR, based on the live spec (4 replicas and daily backup).
- `argocd/mysql-cluster-application.yaml`: a separate Argo CD Application for the CR. It follows `develop` and does not prune resources.

The existing `mysql-lab/mysqlcluster-sample` CR is managed by `mysql-cluster-lab`; its PVCs and backup resources remain managed by the Operator and are not included here. Keep Argo CD pruning disabled. Do not enable another Application that manages the same Operator or CR from `main`. In this lab, merging a promotion MR into `main` changes Git files only.

The Operator writes runtime status and annotations such as `apps.egonlin.com/current-master` to the CR. Do not store those in Git. Before changing the CR spec, review storage, backup and replica changes carefully because they affect persistent data and running MySQL Pods. Do not delete the CR while its PVCs must be retained.

On each new `develop` build input, Jenkins renders `config/default` from the checked-out source into `bundle.yaml`, updates the image digest in `manager-patch.yaml`, and writes an immutable candidate record under `releases/by-fingerprint/`. Repeated builds with the same source input reuse that record and skip image build and scan.

The source repository's `main` pipeline looks up that candidate by content fingerprint. It opens an MR against this repository's `main` branch containing the matching bundle and image digest. Merging that MR updates Git files only: the single Argo CD Application continues to follow `develop`, and no Application follows this repository's `main` branch. The `main` branch's controller replica count is managed manually and is not changed by the promotion pipeline.
