# Flux

## What is Flux?

[Flux](https://fluxcd.io/) in a open source tool for keeping Kubernetes clusters in sync with vcs like (Git). It lives within the kubernetes cluster and is able to automate updates to configuration when there is new code to deploy.

It's the Gitops tools for managing deployments like ArgoCD

## What is Gitops?

Git-ops is a workflow that relies on software automation to establish a desired state in the in cluster, which has been declared declaratively in a vcs.

It's a pull workflow as opposed to a push workflow.

## Why Gitops?

Pro:

1. Easier to manage the state of the cluster. Everything is declaratively defined and maintained in a VCS. The Flux operator then is the only entity that deploys the state. As such there is no configuration drift as there is no manual involvement. 
2. Easy to reason about the state of the cluster. Everything is defined in vcs and as stated in point 1, configuration drift is managed.
3. Access is more easily managed. The only actor here is the Gitops tool that lives within the cluster and has access to deploy/make changes. No other entity should have the requisite permissions to update the cluster director.

Cons:

Limited set of tools like Flux, ArgoCD, JenkinsX as opposed to the more prevalent <i>Push based</i> workflow.

## Structure

This project mirrors two landscape:

1. acpt - For acceptance
2. prod - For Production.


## Getting started

Prerequisites:

1. Minikube or similar should be installed onto the system. For the rest of this tutorial, I'll use minikube. The steps ill be similar for other similar tools.
2. We'll set up two clusters mini-acpt, mini-prod 
   `minikube start --profile mini-acpt`
   `minikube start --profile mini-prod`
3. Set up Flux 
 ```
   kubectx mini-acpt

    kubectl create ns flux

    export GHUSER="your user"
    fluxctl install \
    --git-user=${GHUSER} \
    --git-email=${GHUSER}@users.noreply.github.com \
    --git-branch=main \
    --git-path=acpt \
    --manifest-generation=true \
    --git-url=git@github.com:${GHUSER}/flux-multi-stage \
    --namespace=flux | kubectl apply -f -
```

Similarly set up flux for mini-prod

```
   kubectx mini-prod

    kubectl create ns flux

    export GHUSER="your user"
    fluxctl install \
    --git-user=${GHUSER} \
    --git-email=${GHUSER}@users.noreply.github.com \
    --git-branch=main \
    --git-path=acpt \
    --manifest-generation=true \
    --git-url=git@github.com:${GHUSER}/flux-multi-stage \
    --namespace=flux | kubectl apply -f -
```

PS: If something goes wrong roll back the flux deployment using
```
    export GHUSER="your user"
    fluxctl install \
    --git-user=${GHUSER} \
    --git-email=${GHUSER}@users.noreply.github.com \
    --git-branch=main \
    --git-path=acpt \
    --manifest-generation=true \
    --git-url=git@github.com:${GHUSER}/flux-multi-stage \
    --namespace=flux | kubectl delete -f -
```

Now on running `kubectl get ns` the name space flux should be visisble.

4. Flux requires admin access to the said repository which can be retrived via `fluxctl identity --k8s-fwd-ns flux`. The public key then needs to be added as a <i> deploy key </i> in GitHub. Similar must be mirrored for mini-prod and mini-acpt respectively.
5. By default flux will pull every 5 mins. After five minutes if everything has gone well. There should be a `demo` namespace in each cluster. The `mini-acpt` cluster will have one deplpyment of `ngnix` while the prod should have 2.

Viola any further changes to our deployment should be auto picked up and redeplpyed to the cluster every 5 minstues.

6. Shut everything down via:

```
    minikube stop --profile mini-prod
    minikube stop --profile mini-acpt
```
   

Note: For purposes of this demo, we use `kustomize` reconciliation which is a custom resource that specifies a set of kubenetes resources that Flux is supposed to reconcile in the cluster. Likewise Helm and bucket can also be used. 

## Debugging 

So something has gone wrong. Luckily it's easy to debug.

Check the logs for the flux pods in the flux namespace via : ` kubectl logs <pod-name> -n flux`. There can be messages because of rate limiting of docker hub ignore those and look for messages that show flux operator mirroring and pulling from the VCS. A common error is invalid/incorrect credentials.

