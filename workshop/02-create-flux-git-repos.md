## Introduction

In this part you will perform the following tasks:

- Create a Flux Source Repository for dev and production environments.
- Create Flux Kustomization Custom Resources to orchestrate the deployment of the stateful application in dev and production environments.
- Check the difference between the dev and production settings in the Kubernetes cluster
- Deploy new version of the app into production

## Create Flux Source Git Repositories

The Flux Source API defines a set of Kubernetes objects that cluster admins and various automated operators can interact with to offload the Git and Helm repositories operations to a dedicated controller.

You will create Flux Git Repositories to expose the latest synchronized state from Git as an artefact. These repositories will be monitored by Flux and the source controller will reconcile the application state in Kubernetes upon changes in the Git repository, in accordance to the Kustomization definition.

You are going to create 2 Flux Source Repositories: one for `dev` environment and one for `prod` environment.

Let's start with forking application repository `https://github.com/maty0609/chuck-norris-app-clus2022.git`. This repository represents our app `chuck-norris-app`.

```yaml
cd /root
git clone https://$GITHUB_USER:$GITHUB_TOKEN@github.com/$GITHUB_USER/chuck-norris-app-clus2022.git
```

`tree /root/chuck-norris-app-clus2022`

```yaml
├── README.md
├── chuck-norris-app-src
│   ├── Dockerfile
│   ├── app.yaml
│   ├── main.py
│   ├── requirements.txt
│   ├── static
│   │   └── img
│   │       └── up.png
│   └── templates
│       ├── chuck.html
│       └── norris.html
└── env
    ├── dev
    │   ├── deployment.yaml
    │   └── service.yaml
    └── prod
        ├── deployment.yaml
        └── service.yaml
```

Run command `git branch -r` in the repository `chuck-norris-app-clus2022` to see that we have two branches - `main` and `dev`. This will be important to know for our next steps.

After you will fork the repository run the following command to create configuration for our `dev` environment:

```
mkdir /root/natilik-fleet/clusters/clus2022/apps
cd /root/natilik-fleet/clusters/clus2022
flux create source git dev \
--url https://github.com/$GITHUB_USER/chuck-norris-app-clus2022.git \
--branch dev \
--interval 30s \
--export \
| tee apps/dev.yaml

```

You should see this:

```
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: dev
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: dev
  url: https://github.com/$GITHUB_USER/chuck-norris-app-clus2022.git
```

Let's also create the second one for `prod` environment. Run the following command:

```
flux create source git prod \
--url https://github.com/$GITHUB_USER/chuck-norris-app-clus2022.git \
--branch main \
--interval 30s \
--export \
| tee apps/prod.yaml

```

You should see this:

```
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: prod
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: main
  url: https://github.com/$GITHUB_USER/chuck-norris-app-clus2022.git
```

We have now defined source for our applications. Flux will monitor git repository `https://github.com/$GITHUB_USER/chuck-norris-app.git` and its two branches. Branch `dev` will represent `dev` environment and branch `main` will represent `prod` environment. Now let’s link our new git repositories with our Kubernetes environment.