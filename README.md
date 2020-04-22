# openshift-docker-image-promotion

## Environments
- OpenShift v3.11.x
- Jenkins w/ OpenShift plugin support v2.x
- Skopeo v0.1.x

## Using OpenShift CLI and Skopeo Jenkins Slave

### Non-production cluster
1. Apply ServiceAccount and RoleBinding for pulling a referenced image across projects (or clusters.)

```sh
oc process -f non-production/across-project-image-puller.yaml | oc apply -f -
```

2. Create the Secret which contains Kubernetes bearer token of ServiceAccount created from Step 1.

```sh
oc create secret generic non-production-cluster-token --dry-run=true -o yaml \
    --from-literal=token=$(oc serviceaccounts get-token across-project-image-puller) \
    > non-production-cluster-token.yaml
```

3. Apply the Secret for triggering the production cluster pipeline, replace `secrettext` with any base64-encoded desired text.

```sh
oc apply -f non-production/production-pipeline-webhook-secret-text.yaml
```

4. Apply the non-production pipeline.

```
oc apply -f non-production/non-production-pipeline.yaml
```

### Production cluster
5. Apply the Secret generated from Step 2 to your project in the production cluster.

```sh
oc apply -f non-production/non-production-cluster-token.yaml
```

6. Apply the Secret for pipeline webhook, replace `WebHookSecretKey` with the same value defined in Step 4.

```sh
oc apply -f production/production-pipeline-webhook-secret-key.yaml
```

7. Apply and run Skopeo Jenkins Slave pipeline

```sh
oc apply -f skopeo/skopeo-slave-build-pipeline.yaml
```

8. Apply the production pipeline.

```
oc apply -f production/production-pipeline.yaml
```

Run the non-production pipeline, it should trigger production pipeline with correct `GIT_COMMIT`.
