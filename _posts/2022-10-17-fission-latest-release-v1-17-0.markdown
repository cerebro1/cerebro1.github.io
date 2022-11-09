<p align="center">
  <img src="https://fission.io/images/logo-gh.svg" width="300" />
  <br>
  <h1 align="center">Fission: Serverless Functions for Kubernetes</h1>
</p>

<p align="center">
  <a href="https://github.com/fission/fission/blob/main/LICENSE">
    <img alt="Fission Licence" src="https://img.shields.io/github/license/fission/fission">
  </a>
  <a href="https://github.com/fission/fission/releases">
    <img alt="Fission Releases" src="https://img.shields.io/github/release-pre/fission/fission.svg">
  </a>
  <a href="https://pkg.go.dev/github.com/fission/fission">
    <img alt="go.dev reference" src="https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white">
  </a>
  <a href="https://goreportcard.com/report/github.com/fission/fission">
    <img src="https://goreportcard.com/badge/github.com/fission/fission" alt="Go Report Card" />
  </a>
  <a href="https://github.com/fission/fission/graphs/contributors">
    <img alt="Fission contributors" src="https://img.shields.io/github/contributors/fission/fission">
  </a>
  <a href="https://github.com/fission/fission/commits/main">
    <img alt="Commit Activity" src="https://img.shields.io/github/commit-activity/m/fission/fission">
  </a>
  <br>
  <a href="https://fission.io/">
    <img alt="Fission website" src="https://img.shields.io/badge/website-fission.io-blue">
  </a>
  <a href="https://fission.io/slack">
    <img alt="Fission slack" src="https://badgen.net/badge/slack/Fission?icon=slack">
  </a>
  <a href="https://twitter.com/fissionio">
    <img alt="Fission twitter" src="https://img.shields.io/twitter/follow/fissionio?style=social">
  </a>
  <a href="https://github.com/fission/fission">
    <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/fission/fission?style=social">
  </a>
</p>

---
## Latest Version : v1.17.0
## Feature Release : Security Context Setting for Fission Installation

Fission is a fast serverless framework for Kubernetes with a focus on
developer productivity and high performance.

Fission operates on _just the code_: Docker and Kubernetes are
abstracted away under normal operation, though you can use both to
extend Fission if you want to.

Fission is extensible to any language; the core is written in Go, and
language-specific parts are isolated in something called
_environments_ (more below).  Fission currently supports NodeJS, Python, Ruby, Go, 
PHP, Bash, and any Linux executable, with more languages coming soon.

## Concept of Fission architecture

- Functions
- Environments
- Triggers

So as a developer, you only have to worry about writing the `function` based on
event it is supposed to be invoked by `trigger` and modify/create the
`environment` that has all softwares needed to build and run your Fission
`function`.

Let's talk about the latest feature released by Fission which allows to change
security context.


## Why ? [ why to introduce this feature ? ]

Container were running as root user by default. This led to function run as
root user. In need to operate `fission` without root, user manually updated
through environment configuration by marking the pod as `runAsNonRoot`,
`runAsUser` and `runAsGroup`. Since `fission` is all about increasing developer
productivity, Fission team has added this feature in the default helm chart as
good security practice.


## What ? [ what happened before and after release ? ]

**Before release :**

![before-implementation-con](./screenshots/before-con-set.png)

`whoami` returns that container is launched with root user as default in pod
`buildermgr-5988846597-5jbzz`. In Linux environment, `#` denotes system administrator
which is root login.


**After release :**

![after-implementation-con](./screenshots/after-con-set.png)

`whoami` returns `uid 10001` which is user uid for container running as builder
manager component. After the upgrade, old pod is replaced with new pod. Since
now the container in pod is launched as non root user, the functions can be run
as a non-root user. In Linux environment, `$` denotes non-root or normal user
login.


## How ? [ How to upgrade to use this feature ? ]

Prerequisites :

- Kubernetes CLI [ Eg : kubectl ]
- Kubernetes Cluster [ Eg : minikube ]
- Helm
- Fission

Please visit Fission [installation](https://fission.io/docs/installation/) guide to validate fission and
it's prerequisites are installed.


**Expected Changes : There are two ways to do this as shown below**

*Method 1.* If you want complete Fission architecture to work as non-root user all the
following component values have to be set `true`.

```
    executor.securityContext.enabled: true
    router.securityContext.enabled: true
    buildermgr.securityContext.enabled: true
    controller.securityContext.enabled: true
    kubewatcher.securityContext.enabled: true
    storagesvc.securityContext.enabled: true
```

Recommended security context for builder and function pods :

```
    runtimePodSpec.enabled: true
    builderPodSpec.enabled: true
```

**Example :** 

For each component, Fission maintains a pod. Let's check if all the pods of Fission are `Active`.

![before-pod-set](./screenshots/before-pod-set.png)

Let's connect to a pod and check user with which container is running :

```
sonalis@cere:~/pyshorturl/docs [main] $ kubectl exec -it controller-5d949b66-bbqbd -n fission -- /bin/sh
/ # whoami
root
```

To make the changes, if you have a pre-existing values.yml available, update the same
or download one from fission-charts using :

```
wget https://github.com/fission/fission/blob/main/charts/fission-all/values.yaml?raw=True -O values.yaml
```

Note : user supplied values.yaml file has priority over parent chart's
values.yaml. Here parent chart's is fission default values.yaml where security
context is set to `false`.

Edit values.yaml :

![values.yaml controller component](./screenshots/values-yaml-before.png)

Update the enabled field here as shown below

```
    enabled: true
```

Make the changes for all components : contorller, executor, router, buildermgr,
kubewatcher and storagesvc. Similarly for builder and function pod.

![values.yaml-builderpod](./screenshots/values-yaml-builderpod-after.png)

![values.yaml-functionpod](./screenshots/values-yaml-runtimepod-after.png)


Fission pods run in a namespace. `namespace` help in providing isolation to the
pod or set of pods. In order to apply the changes we will need name of the
namespace in which Fission pods are running. Let's check it's name.

```
$ kubectl get namespace
NAME               STATUS   AGE
default            Active   29h
fission            Active   81s
fission-builder    Active   42s
fission-function   Active   42s
kube-node-lease    Active   29h
kube-public        Active   29h
kube-system        Active   29h
```

The namespace for Fission is `fission`.

Let's use helm to re-launch Fission with latest changes. `helm` is a package
manager for Kubernetes.

```
helm upgrade --namespace fission fission fission-charts/fission-all -f values.yaml
```

If you are doing a fresh install of fission, you can make use of `install`
instead of `upgrade`.

Above command may take 5-10 seconds to return you success.

Let's check if all the pods of Fission are re-launced successfully :

![after-pod-values](./screenshots/after-values-pod.png)

Here we can see that the name of the pods have changed if we compare from our
previous snapshot. Let us login in one of it and check user logged in.

```
sonalis@cere:~/pyshorturl/docs [main] $ kubectl exec -it controller-5b4b9cfcdb-tjtb6 -n fission -- /bin/sh
/ $ whoami
whoami: unknown uid 10001
/ $ date
Sat Oct 15 16:04:30 UTC 2022
```

The values.yaml file defines the `uid` that will be used for login as non-root
user.
```
    runAsNonRoot: true
    fsGroup: 10001
    runAsUser: 10001
    runAsGroup: 10001
```

*Method 2.* If you are sure which component you want to run as root and which
not, you can also pass that as cli option using `--set` while upgrading fission
using `helm` as shown below. 

Please note : `--set` has priority over user supplied values.yaml file.

```
helm upgrade --namespace $FISSION_NAMESPACE fission fission-charts/fission-all --set buildermgr.securityContext.enabled=true
```

Have queries ?

Drop me a mail : srivastava.sonali1@gmail.com

Believe in improving documentations, start from [Stackoverflow](https://stackoverflow.com/users/5043833/sonali-srivastava)

Happy reading!
