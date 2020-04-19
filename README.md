# openshift-docker-image-promotion

## Environments
- OpenShift v3.11.x
- Jenkins w/ OpenShift plugin support v2.x
- Skopeo v0.1.x

## Using OpenShift CLI and Skopeo Jenkins Slave

### Non-production cluster
1. Apply ServiceAccount and RoleBinding for pulling a referenced image across projects (or clusters.)

```sh
oc process -f across-project-image-puller.yaml | oc apply -f -
```

2. Create the Secret which contains Kubernetes bearer token of ServiceAccount created from Step 1.

```sh
oc create secret generic non-production-cluster-token --dry-run=true -o yaml \
    --from-literal=token=$(oc serviceaccounts get-token across-project-image-puller) \
    > non-production-cluster-token.yaml
```

3. Then apply the Secret generated from Step 2 to your project in the non-production cluster.

```sh
oc apply -f non-production-cluster-token.yaml
```

4. Apply the Secret for triggering the production cluster pipeline, replace `secrettext` with any base64-encoded desired text.

```sh
oc apply -f production-pipeline-webhook-secret-text.yaml
```

### Production cluster
5. Apply the Secret for pipeline webhook, replace `WebHookSecretKey` with the same value defined in Step 4.

```sh
oc apply -f production-pipeline-webhook-secret-key.yaml
```
