# [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

There are two primary flavours of secrets, a `Secret` and a `SealedSecret`. 

## Secret

A Kubernetes secret object lets a developer store and manage a *small* amount of sensitive
data, such as passwords, api-tokens or similar. 

Normally this data would be put directly into a pod, but with a Secret a developer
gains more control over how its used and reduces risk of accidental exposure.

For a pod to be able to use a Secret, it needs to refer it. This happens in two ways:
* It's referred through a volume that holds the secret or
* The secret is referred by the kublet when the image is / are pulled.

Read more about using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)

#### Gotchas

##### Concourse

You can reference Kubernetes secrets in Concourse, by setting the secret's namespace
to be the same as where the `Concourse` worker exists in. Then referencing it by 
following appropriate dot-notation (`context.secret`). Such as `github.access-token`. 

##### Base64

Following on to the Concourse gotchas, you will need to ensure that your secret is 
base64 encoded before creating the secret. This happened to me once when it turned out a Concourse pipeline
pulled out the secret and base64 decoded it.

Meaning if you have a secret `foo=bar`, `bar` needs to be encoded to `YmFy`. 
Because when Concourse pulls the secret out, it will base64 decode it. So if 
you didn't encode it, you would have `m�`.  
 