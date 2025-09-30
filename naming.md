# Naming Things is Easy

## Don't stutter

Don't put the type of thing in the thing's name.

If you're naming an EKS cluster, don't call it "eks-pandora-dev", just call it "pandora-dev". If you're creating a policy, don't name it "guest-policy", just call it "guest".

## Say the name

Name things as you would speak them. If you have a cluster named project-wombat-us-east-1-dev, but you always refer to it as "wombat dev" when you talk about it to your colleagues, then name it "wombat-dev".

## Meaningless hostnames

Hostnames should be GUIDs or random strings like "burning-yak". They should not contain any metadata.

In the olden times, we used to have complex naming schemes where we tried to cram all this data into hostnames, because tagging hadn't been invented yet. Something like this:

| Characters | Metadata | Example |
| ---------- | -------- | ------- |
| 3          | location | ord, lax, lhr |
| 1          | type     | v (virtual), p (physical) |
| 1          | OS       | l (linux), w (windows) |
| 1          | make     | d (dell), i (ibm), h (HP) |
| 3          | env      | d (dev), s (staging), p (prod) |
| 3          | purpose  | vmw (VMWare), bld (build), web |
| 2          | number   | 01, 02, 03 |


Leading to names like, "ordplddweb01". This was suboptimal for a number of scenarios:

- We often used scripts to parse the hostname to do specific things based on this baked-in metadata, which meant that changing the schema of the metadata was very, very difficult.
- What if the host's purpose changes over time? Don't take down that "bld" machine, because it actually has our prod artifact repo running on it, which our customers use to download packages.
- What if we start buying another make that begins with a "d"? These new DEC servers have make code "e", because "d" is already Dell.
- What if we move a data center to a new city? We have to rename all of the machines? Ugh!

## Personal Access Tokens or API keys

A token or key's name should point to the git repo or script that uses that token. For example, if we had a pipeline in this repo that required a token, we would name the token "https://gitlab.com/devopscoop/organization/".

## Environment names

Don't make up new environment names. They've all been made up already. Read this: https://en.wikipedia.org/wiki/Deployment_environment

## Prod is prod

It's not "production" or "prd" - it's "prod".

## AWS Accounts

Based on:

- https://docs.aws.amazon.com/IAM/latest/UserGuide/console-account-alias.html
- [SEC01-BP01 Separate workloads using accounts](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/sec_securely_operate_multi_accounts.html)

Rules:

- The account alias must contain only digits, lowercase letters, and hyphens.
- The account alias must be unique across all Amazon Web Services products within a given network partition (ie. aws, aws-cn, aws-us-gov).
- Desired outcome: An account structure that isolates cloud operations, unrelated workloads, and environments into separate accounts, increasing security across the cloud infrastructure. 

We should use org name as a prefix to ensure that we have uniqueness. And we should have separate accounts per project and per environment per SEC01-BP01 Separate workloads using accounts.

Recommended naming scheme for AWS accounts:

```
${org_name}-${project}-${env}
```

For example:

```
devopscoop-project1-dev
```

## Scripts have suffixes

Shell scripts end in `.sh`. Python scripts end in `.py`. Add the correct shebang, and make them executable too.

## Use IATA codes for locations

Don't make up location names. Use the [IATA 3-letter airport code](https://www.iata.org/en/publications/directories/code-search/) for the closest city.

## Data centers

All of the cloud service providers (CSP) have their own, special region names.

AWS:

```
aws account list-regions | jq .Regions[].RegionName | head
"af-south-1"
"ap-east-1"
"ap-east-2"
"ap-northeast-1"
"ap-northeast-2"
"ap-northeast-3"
"ap-south-1"
"ap-south-2"
"ap-southeast-1"
"ap-southeast-2"
```

Azure:

```
az account list-locations | jq .[].name | head
"eastus"
"westus2"
"australiaeast"
"southeastasia"
"northeurope"
"swedencentral"
"uksouth"
"westeurope"
"centralus"
"southafricanorth"
```

To have a naming scheme that works for any org on any CSP(s), we should use use the location (see above) with an integer. So, for example, AWS us-east-2 would be "cmh1". If we happen to deploy more services to a different CSP in Columbus, Ohio (CMH), then we would name that second data center "cmh2". Why an integer instead of a code for the CSP? Because maintaining a list of codes for CSPs sounds miserable, and what if we use a non-cloud, colocation facility? Do we have to have codes for every possible CSP and colo? No thanks.

TODO: Find the code that shows where all of AWS's edges are located so we can map us-east-2 to cmh...

## Kubernetes clusters

Name them project-env. For example, if your org has a project called "tracker", then name the dev cluster "tracker-dev" and the prod cluster "tracker-prod".

If you multiple data centers for a single environment for the highest availability/performance, then put that at then end. For example: tracker-dev-cmh1

## DNS subdomains

Every Kubernetes cluster should have it's own epynomous subdomain, e.g. tracker-dev.devops.coop.
