Useful references:

https://codefresh.io/blog/stop-using-branches-deploying-different-gitops-environments/

https://codefresh.io/blog/how-to-model-your-gitops-environments-and-promote-releases-between-them/

https://kubebyexample.com/learning-paths/argo-cd/argo-cd-working-kustomize

https://codefresh.io/blog/stop-using-branches-deploying-different-gitops-environments/

https://github.com/avmgithub/gitops-environment-promotion/tree/main

https://cloud.redhat.com/blog/your-guide-to-continuous-delivery-with-openshift-gitops-and-kustomize

https://demo.openshift.com/en/latest/argocd/

#### Creating pull secret for private registries - IBM cloud docs

https://cloud.ibm.com/docs/containers?topic=containers-registry#other_registry_accounts

#### For demo purposes only. We use bitnami kubeseal to encrypt passwords
## Sealed Secrets

The app gets deployed using a helm chart which is included in this repo.
The app depends on a secret called `mq-quarkus-app` that contains two key value pairs
called `USER` and `PASSWORD` which contain the info to authenticate the client app with the MQ server.
We are using Sealed Secrets to create the secret (https://github.com/bitnami-labs/sealed-secrets).
The way sealed secrets work is, you create the sealed secret resource in the target kube/openshift namespace
and the operator will generate the actual secret.

Create a kube secret file with the unencrypted values as follows:

```
oc create secret generic mq-quarkus-app --from-literal=USER=<user-name> --from-literal=PASSWORD=<password> --dry-run=true -o yaml > mq-quarkus-app.yaml
```

Then generate the encrypted values using the kubeseal cli as follows:

```
kubeseal --scope cluster-wide --controller-name=sealedsecretcontroller-sealed-secrets --controller-namespace=sealed-secrets -o yaml < mq-quarkus-app.yaml > mq-quarkus-app-enc.yaml
```
The file mq-quarkus-app-enc.yaml`  will contain the encrypted values to modify  USERand PASSWORD in  chart/base/values.yaml.

In this particular case, the sealed secret created has a cluster-wide scope.
To further lock down the setup and enhance security, you can create the sealed secret with a namespace scope.
See kubbeseal docs to better understand this.
