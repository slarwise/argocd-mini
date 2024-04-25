# argocd - Testing apps in any namespace

Admin sets up an app-of-apps which points to a team's app-of-apps. The team
project can deploy applications to namespace team. The team's app-of-apps
consists of two identical guestbook applications, which are deployed to
namespace team. They cannot change the guestbook applications to use e.g. the
default project, since only applications in the argocd namespace can use the
default project:

> If no .spec.sourceNamespaces is set to an AppProject, a default of the control
> plane's namespace (e.g. argocd) will be assumed, and the current behaviour
> would not be changed (only Application resources created in the argocd
> namespace would be allowed to associate with the AppProject).

From the
[proposal](https://github.com/argoproj/argo-cd/blob/master/docs/proposals/003-applications-outside-argocd-namespace.md#proposal).
Tenant self-service use case in the proposal
[here](https://github.com/argoproj/argo-cd/blob/master/docs/proposals/003-applications-outside-argocd-namespace.md#use-case-3-easy-onboarding-of-new-applications-and-tenants).

```
default
  App/app-of-apps
  Project/team
team
  App/team-app-of-apps
  App/guestbook
  App/guestbook2
  Deployment/guestbook
  Service/guestbook
  Deployment/guestbook2
  Service/guestbook2
```

## Running locally

Instructions taken from
[here](https://argo-cd.readthedocs.io/en/stable/operator-manual/app-any-namespace/).
Start a kubernetes cluster, e.g. `kind`:

```sh
kind create cluster
```

Install, configure and inspect argo:

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Enable applications in any namespaces
k edit configmaps argocd-cmd-params-cm
# data:
#   application.namespaces: team
kubectl rollout restart -n argocd deployment argocd-server
kubectl rollout restart -n argocd statefulset argocd-application-controller

k apply -k admin/

kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get default password for the admin
argocd admin initial-password -n argocd

# Go to localhost:8080 and login as admin
```
