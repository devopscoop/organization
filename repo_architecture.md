# Repository Architecture

## Overview

How should we structure our code repositories? A thought experiment.

Assumptions:

- We use git. There are a few other [distributed, open version control systems](https://en.wikipedia.org/wiki/List_of_version-control_software), but git is good - everyone knows git.
- Our organization builds software that people rely on, so we must follow decent security and compliance practices. [Separation of Duties](https://en.wikipedia.org/wiki/Separation_of_duties) suggests that only developers should approve changes to application code, and only operators should approve changes to the production environment.
- Applications are packaged in containers.
- Containers run in Kubernetes.
- For infrastructure, we're using declarative, not imperative, code.
- Everyone in the organization has read-only access to all code for educational purposes, to encourage collaboration and code reuse, and to reduce duplication of work.
- Everyone in the organization can create a pull request (PR) in any repo.
- We use CI/CD environments[[1]](https://docs.gitlab.com/ee/ci/environments/)[[2]](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment) to ensure that a PR can't leak secrets or change production infrastructure.
- We avoid putting variables and secrets in our git platform provider. It's not GitOps, and it's a pain to migrate all of that if you ever change providers. (The last three companies I've worked for have been in the middle of a migration from one git provider to another...)
- We use [trunk-based development](https://trunkbaseddevelopment.com/).
- We prefer to use community-supported Helm charts to deploy workloads in Kubernetes.
- We use Flux instead of Helmfile, because "pull" paradigms offer some security and consistency advantages. We don't use ArgoCD, because it doesn't install true Helm releases, and it doesn't work with SOPS.

## Types of code

### Application (app)

This is the source code for an application, API, or service. It's written in Python, Golang, Node.js, Rust, etc. It has a Dockerfile, a pipeline to build a container image, and optionally a job to update the deployment code with the new image tag.

Only developers can approve and merge changes to app code.

### Infrastructure (infra)

Infrastructure as Code (IaC) defines: cloud resources, networks, servers, Kubernetes clusters, DNS providers (e.g., Cloudflare), etc. This code uses one of the common IaC tools like OpenTofu/Terraform or Ansible. It should have a pipeline that shows the plan, so reviewers can see what will change before the code is merged to the default branch. A merge to the default branch will apply the code.

Only operators can approve and merge changes to infra code in production environments. Developers can approve and merge changes to infra code for non-prod environments.

### Deployment (deploy)

This code contains the configuration for a specific [deployment environment](https://en.wikipedia.org/wiki/Deployment_environment) (e.g., dev, test, stage, prod, etc.) It should have the image that's being deployed, number of replicas, environment variables, secrets (or pointers to the secrets), etc. It should also contain the configuration for any supporting services like PostgreSQL databases, Redis, Rabbit, etc. It's probably going to be a bunch of Helm values that are applied with Flux.

In the Kubernetes world, I would also include the ingress controller, cert-manager, external-dns, and similar services in this repo, just because they're also going to be deployed as Helm releases. One could argue that those should go in the infra code, but that would mean that we would have to have Flux watching multiple repos, and that feels like an anti-pattern. That could lead to a situation where the same thing is defined slightly differently in two repos and Flux will constantly vacillate between the two different configurations. I'm less concerned about giving Devs access to change these basic services in non-prod environments than I am about mixing different types of code. But in my recommendation at the bottom, I have the `infra` and `deploy` code in directories inside a single repo, so perhaps we could split it into `tf_infra`, `flux_infra`, and `deploy`?

Only operators can approve and merge changes to deploy code in production environments. Developers can approve and merge changes to deploy code for non-prod environments.

## Roles

### Development (Dev)

Developers come in a few flavors: frontend, backend, full-stack, etc. Their languages are imperative. They cannot be trusted - see half of the factors listed in the [OWASP Top Ten](https://owasp.org/www-project-top-ten/).

### Operations (Ops)

Operators also come in some flavors: network, datacenter, infrastructure, cloud, security, observability, SRE, etc. These are the people that manage the customer-facing, production environments. They ensure that everything is highly-available, secure, compliant, efficient, and backed up. They have the business continuity and disaster recovery plans. They keep the lights on. They cannot be trusted either - see the other half of the factors listed in the OWASP Top Ten. But with our powers combined under the DevOps philosophy, maybe we can build something that's not a dumpster fire!

## Teams

I'm defining a team as a small group of software engineers that work together on a single product or collection of similar products. An organization usually has several teams.

### Mono-functional Team

Some orgs have multiple Dev teams, and a single Ops team that supports all of them.

### Cross-functional Team

Other orgs have an [embedded](https://en.wikipedia.org/wiki/Site_reliability_engineering#Embedded) model where each team has their own operators, or the most senior full-stack developer acts as an operator (this usually doesn't go well once they start getting real traffic.)

## Architectures

Code Separation (repo, branch, or directory):

- When should we create a new git repository?
- When should we create a new branch of an existing repository?
- When should we create a new directory in our repository?

You can assign pretty granular permissions on repos regardless of which git platform you use.

You can sometimes assign permissions per branch with branch protections rules if you pay for a business or enterprise license.

It's hard to assign permissions to individual directories in a git repo, so they should really just be used as logical separators for

Example architectures below will be in this format:

```
git repo name:
  git branch name:
    directory name/
      filename
```

### One monorepo to rule them all

```
the_one_repo:
  main:
    team1_app1/
      src/
      Dockerfile
      .github/workflows/build-deploy.yaml
    team1_app2/
    team1_dev_infra/
      Makefile
      main.tf
      .github/workflows/opentofu.yaml
    team1_dev_deploy/
      apps/
        kustomization.yaml
        app1/
          kustomization.yaml
          namespace.yaml
          release.yaml
        app2/
          ...
      flux-system/
        gotk-components.yaml
        gotk-sync.yaml
        kustomization.yaml
      apps.yaml
    team1_prod_infra/
    team1_prod_deploy/
    team2_app1/
    team2_app2/
    team2_dev_infra/
    team2_dev_deploy/
    team2_prod_infra/
    team2_prod_deploy/
```

Obvious flaws:

- Managing permissions for a single repo is not really possible.
- Number of commits to this repo is going to be really high - will likely cause annoyance at constantly having to `git pull` before you can merge code.
- People will laugh at us.

I'm not going any further with this option, because of the flaws.

### Repo per Team

```
team1:
  main:
    app1/
    app2/
    dev_infra/
    dev_deploy/
    prod_infra/
    prod_deploy/
team2:
  main:
    app1/
    app2/
    dev_infra/
    dev_deploy/
    prod_infra/
    prod_deploy/
```

Obvious Flaws:

- Compliance normally requires separation of development from deployment. It would be difficult to ensure that only developers can merge app code, and only operators can merge prod code.

### Repo per Team per Role

```
team1_Dev:
  main:
    app1/
    app2/
team1_DevOps:
  main:
    dev_infra/
    dev_deploy/
team1_Ops:
  main:
    prod_infra/
    prod_deploy/
team2_Dev:
  main:
    app1/
    app2/
team2_DevOps:
  main:
    dev_infra/
    dev_deploy/
team2_Ops:
  main:
    prod_infra/
    prod_deploy/
```

You know, I don't hate this model...

This would allow us to give repo-level permissions to the correct groups easily. Developers can approve changes to team1_Dev, both developers and operators can approve changes in the team1_DevOps repo, and operators can approve changes to the team1_Ops repo.

I question whether the apps aren't important enough to get their own repos, but if it's the same group of people working on the same set of apps, I can see how it would make sense for them to be in the same repo...

As an operator, having to flip from `team1_DevOps/dev_infra` to `team1_Ops/prod_infra` feels a little weird.

### Repo per App or Deployment Environment

```
team1_app1:

team1_app2:

team1_dev:
  main:
    infra/
    deploy/

team1_prod:
  main:
    infra/
    deploy/

team2_app1:

team2_app2:

team2_dev:
  main:
    infra/
    deploy/

team2_prod:
  main:
    infra/
    deploy/
```

This feels the closest to a common sense structure to me, but I'm sure that I'm biased by my previous work experience. The only issue with this structure is that we are probably hardcoding the team's name in the repo names. Teams change. Apps move to other teams. I've been through it before. It's miserable.

### TL;DR: Recommended Architecture

I would actually suggest that we don't use the team name at all. We want our repos to have the shortest names and closely match what people actually, verbally call this code. If an org already has an "app1" anywhere in it, then no other team should call their app "app1". Perhaps if an org has multiple Ops teams, I'd just prefix the environment repo with the Ops team's name. I'd simplify this to:

```
app1:

app2:

ops1_dev:
  main:
    infra/
    deploy/

ops1_prod:
  main:
    infra/
    deploy/

app3:

app4:

ops2_dev:
  main:
    infra/
    deploy/

ops2_prod:
  main:
    infra/
    deploy/
```
