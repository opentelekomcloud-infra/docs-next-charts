# Step-by-Step Deployment

All the steps are mandatory and the order matters.

## Dependencies

First we need to install all the dependencies and third components that support the functionality:

### argocd

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes. It automates the deployment and lifecycle management of applications from Git repositories to specified target environments.

#### Install the Helm Chart

```shell
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm install \
    argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace
```

> [!IMPORTANT]
> - **argocd** is optional but it is strongly recommended to be used for installing the rest of components for consistency and automation.
> - It is **not** recommended to publish argocd externally via a load balancer or an ingress. Prefer to port forward it to your working host. 

### cert-manager

cert-manager is an open source project that provides X.509 certificate management for Kubernetes and OpenShift workloads. It supports TLS for Ingress, mTLS for pod-to-pod communication, and integrates with various Issuers and service mesh add-ons. 

#### Install the Helm Chart

```shell
helm repo add jetstack https://charts.jetstack.io 
helm repo update 

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.15.3 \
  --set crds.enabled=true
```

### cert-manager-webhook-opentelekomcloud

That's an ACME DNS01 solver webhook for Open Telekom Cloud DNS, and requires **cert-manager** to be installed first.

#### Get Access/Secret Keys

Go to My Credentials -> Access Keys and either pick up an existing pair or create a new one:

![Access Keys](<assets/images/Screenshot from 2024-09-07 11-33-33.png>)

```shell
export OS_ACCESS_KEY={value}
export OS_SECRET_KEY={value}
```

#### Install the Helm Chart

```shell
helm repo add cert-manager-webhook-opentelekomcloud https://akyriako.github.io/cert-manager-webhook-opentelekomcloud/
helm repo update

helm upgrade --install \
    acme-dns-opentelekomcloud cert-manager-webhook-opentelekomcloud/cert-manager-webhook-opentelekomcloud \
  --set opentelekomcloud.accessKey=$OS_ACCESS_KEY \
  --set opentelekomcloud.secretKey=$OS_SECRET_KEY \
  --namespace cert-manager
```

#### Deploy Cluster Issuers

You are going to need one `ClusterIssuer` for the *production* and one for the *staging* Let's Encrypt endpoint. 

> [!CAUTION]
> cert-manager has a known bug, that prevents custom webhooks to work with an `Issuer`. For that reason you need to install your issuer as `ClusterIssuer`.

- Staging:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: opentelekomcloud-letsencrypt-staging
  namespace: cert-manager
spec:
  acme:
    email: user@company.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: opentelekomcloud-letsencrypt-staging-tls-key
    solvers:
    - dns01:
        webhook:
          groupName: acme.opentelekomcloud.com
          solverName: opentelekomcloud
          config:
            region: "eu-de"
            accessKeySecretRef:
              name: cert-manager-webhook-opentelekomcloud-creds
              key: accessKey
            secretKeySecretRef:
              name: cert-manager-webhook-opentelekomcloud-creds
              key: secretKey
```

- Production:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: opentelekomcloud-letsencrypt
  namespace: cert-manager
spec:
  acme:
    email: user@company.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: opentelekomcloud-letsencrypt-tls-key
    solvers:
    - dns01:
        webhook:
          groupName: acme.opentelekomcloud.com
          solverName: opentelekomcloud
          config:
            region: "eu-de"
            accessKeySecretRef:
              name: cert-manager-webhook-opentelekomcloud-creds
              key: accessKey
            secretKeySecretRef:
              name: cert-manager-webhook-opentelekomcloud-creds
              key: secretKey
```

> [!IMPORTANT]
> Replace placeholder `email` value, `user@company.com`, with the email that will be used for requesting certificates from Let's Encrypt.

### zalando-postgres-operator

Zalando Postgres Operator creates and manages PostgreSQL clusters running in Kubernetes. The operator delivers an easy to run highly-available PostgreSQL clusters on Kubernetes powered by [Patroni](https://github.com/patroni/patroni). It is configured only through Postgres manifests (CRDs) to ease integration into automated CI/CD pipelines with no access to Kubernetes API directly, promoting infrastructure as code vs manual operations.

> [!IMPORTANT]
> The **zalando-postgres-operator** is optional, as long as you have another way to provision internally or externally PostgreSQL databases, which are required by **Umami**. 

#### Install the Helm Chart

```shell
helm repo add postgres-operator-charts https://opensource.zalando.com/postgres-operator/charts/postgres-operator
helm repo update

helm install postgres-operator postgres-operator-charts/postgres-operator
```

## Prerequisites

After deploying the dependencies we need to provision and configure the following components in Open Telekom Cloud:

### Load Balancers

- Add 3 Shared Load Balancers in the same VPC and Subnet with your CCE Cluster. 
- Let Open Telekom Cloud to assign them with an Elastic IP Address.
- Note down their IDs and their EIPs

![Elastic Load Balancers](<assets/images/Screenshot from 2024-09-07 11-49-47.png>)

### DNS Records

Pick up a domain name for each one of the 3 components e.g.:

- Docusaurus (docs-next): arc.open-telekom-cloud.com 
- Typesense (typesense-reverse-proxy): arc-search.open-telekom-cloud.com
- Umami (umami-web): arc-analytics.open-telekom-cloud.com

For every environment you can suffix the subdomain with the environment identifier e.g.:

Production: arc.open-telekom-cloud.com
Staging: arc-preview.open-telekom-cloud.com
Development arc-dev.open-telekom-cloud.com

> [!TIP]
> - You **don't need** an Umami instance per enviroment. A single instance suffice and you can then add your different Docusaurus environment for tracking as under Umami Websites.
> - Umami can be installed on a separate management cluster, as long as it is exposed in a manner that would be reachable from all your environments.

Next:

- Go to Domain Name Service -> Public Zones and pick the public zone of your domain.
- Bind the EIP address of a Load Balancer with a domain name by create an A-Record in the DNS zone.

![Create A Records](<assets/images/Screenshot from 2024-09-07 12-50-54.png>)

## Typesense

## Umami

## Docusaurus

