# echo-app
Simple echo app and helm chart repository template to use with Loft Labs vCluster.Pro demo examples.

## vCluster.Pro Integration Examples

### Argo CD

vCluster.Pro includes an Argo CD integration that will automatically add a vCluster instance, created with a virtual cluster template, to Argo CD as a target cluster. 

*Example `management.loft.sh/v1` `VirtualClusterTemplate` manifest (with unrelated configuration execluded - full version here) that enables the automatic syncing of the vCluster instance created with the template to Argo CD:*

```yaml
kind: VirtualClusterTemplate
apiVersion: management.loft.sh/v1
metadata:
  name: preview-template
spec:
  displayName: vCluster.Pro Preview Template
  template:
    metadata:
      labels:
        loft.sh/import-argocd: 'true'
...
```

The virtual cluster template integration requires that the vCluster.Pro project, where the vCluster instance is created from said template, to have the Argo CD integration for projects enabled. 

*Example `management.loft.sh/v1` `Project` manifest (with unrelated configuration execluded - full version here) that enables the syncing of vCluster instances to Argo CD:*

```yaml
kind: Project
apiVersion: management.loft.sh/v1
metadata:
  name: api-framework
spec:
  displayName: API Framework
...
  argoCD:
    enabled: true
    cluster: loft-cluster
    namespace: argocd
    project:
      enabled: true
```

#### Example: ApplicationSet Pull Request Generator

#### Example: ApplicationSet Cluster Generator
In addition to automatically adding/syncing vCluster instances to Argo CD, the vCluster.Pro integration also syncs `instanceTemplate` `labels` of a virtual cluster template to the Argo CD cluster `Secret` generated by the integration discussed above.

*Example `management.loft.sh/v1` `Project` manifest (with unrelated configuration execluded - full version here) that enables the syncing of vCluster instances to Argo CD:*

```yaml
apiVersion: management.loft.sh/v1
kind: VirtualClusterTemplate
metadata:
  name: vcluster-pro-template
  labels:
    app.kubernetes.io/instance: loft-configuration
spec:
  displayName: Virtual Cluster Pro Template
...
  template:
...
  versions:
    - template:
        metadata: {}
        instanceTemplate:
          metadata:
            labels:
              env: '{{ .Values.env }}'
              team: '{{ .Values.loft.project }}'
        pro:
          enabled: true
...
      parameters:
        - variable: env
          label: Deployment Environment
          description: Environment for deployments for this vCluster used as cluster label for Argo CD ApplicationSet Cluster Generator
          options:
            - dev
            - qa
            - prod
          defaultValue: dev
      version: 1.0.0
    - template:
        metadata: {}
        instanceTemplate:
          metadata: {}
      version: 0.0.0
...
```
In this example the values for the `instanceTemplate.metadata.labels` - `env` and `team` are populated with the `env` parameter, but they could also be hardcoded so that every vCluster instance created from this template had the same `env` label value.

