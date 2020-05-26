# Purpose
- Ansible role to deploy [ArangoDB Operator](https://github.com/arangodb/kube-arangodb) in Kubernetes.
- Multiple operators can be deployed in multiple namespaces. Default namespace is: `arangodb`
- Multiple ArangoDB clusters can be deployed by an operator.


# Features
- Downloads [manifests](https://github.com/arangodb/kube-arangodb/tree/master/manifests) from desired release
- Applies user default patches, overriding defaults in original manifests
- Loads resulting manifests in Kubernetes:
    - default: by connecting to the master K8S node via ssh
    - alternatively: provinding local kubernetes config, by setting `kubeconfig_file_path`
- Instruct operator to deploy cluster(s) and optionally local storages


# Dependencies
- [Kustomize](https://github.com/kubernetes-sigs/kustomize) - will be automatically downloaded in project `bin/` directory
- Pip3 packages: see [requirements.pip](requirements.pip) - automatically installed in preflight


# Usage
1. Set up your ansible inventory with appropriate hosts and SSH keys
2. Set your cluster and deployments options in the vars file (See [sample-vars.yml](sample-vars.yml))
3. Run the playbook:

    ```bash
    ANSIBLE_SSH_PIPELINING=true \
    ANSIBLE_CONFIG=sample-ansible.cfg \
    ansible playbook \
        --inventory sample-inventory \
        sample-playbook.yml \
        --extra-vars "@sample-vars.yml"`
    ```

    - To skip preflight and speed up a bit, add `--skip-tags preflight`


# Backup using the operator
- TODO


# Reference
- https://www.arangodb.com/docs/stable/tutorials-kubernetes.html
- https://www.arangodb.com/docs/stable/deployment-kubernetes-deployment-resource.html


# Variables and parameters

| Variable                                 | Default                                                       | Description                                                                                                                                                                                                         |
|------------------------------------------|---------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| kustomize_version                        | 3.5.5                                                         | Kustomize version to download an use. Kustomize is used locally, even though there is one in the system.                                                                                                            |
| arangodb_operator_version                | 1.0.2                                                         | ArangoDB Operator version (do not put 'v' in front)                                                                                                                                                                 |
| arangodb_operator_namespace              | arangodb                                                      | Operator namespace                                                                                                                                                                                                  |
| arangodb_operator_manifests              | [arango-crd.yaml, arango-deployment.yaml, arango-backup.yaml] | List of Kubernetes manifests to download from the operator repo. This should not change normally.                                                                                                                   |
| arangodb_operator_manifest_local_storage | arango-storage.yaml                                           | Optional operator local storage manifest. Set to empty to not download it and not use local storage. If empty, `arangodb_local_storages` will be ignored                                                            |
| arangodb_operator_manifest_replication   | arango-deployment-replication.yaml                            | Optional operator for DC2DC replciation. Set to empty to not download and not use replication                                                                                                                       |
| |
| kustomizations                           | []                                                            | List of objects to locate and patch default kubernetes objects in the download manifests. You must specify `apiVersion`, `kind` and `metadata.name`                                                                 |
| arangodb_local_storages                  | []                                                            | Optional list of ArangoLocalStorage. See https://www.arangodb.com/docs/stable/deployment-kubernetes-storage-resource.html                                                                                           |
| arangodb_clusters                        | []                                                            | List of ArangoDeployment objects - actual ArangoDB cluster definition. See https://www.arangodb.com/docs/stable/deployment-kubernetes-deployment-resource.html                                                               |
| |
| kubeconfig_file_path                     |                                                               | Optional path to Kubernetes admin config file (on the master node is located at /etc/kubernetes/admin.conf). Setting this will not use SSH connection to master, instead try to access the Kubernetes API directly. |
| kubernetes_operation_retries             | 20                                                            | How many times to retry uploading/waiting for a kubernetes manifest  before giving up. Some resources, like Deployments may take longer to be ready.                                                                |
| kubernetes_operation_delay               | 10                                                            | Interval in seconds to retry                                                                                                                                                                                        |
| kubernetes_ignore_errors                 | yes                                                           | Set to 'no' to fail fast at the first encountered error when trying to create objects in kubernetes                                                                                                                 |


# License
MIT
