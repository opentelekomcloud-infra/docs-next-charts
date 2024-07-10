# Open Telekom Cloud Architecture Center Helm Charts

Open Telekom Cloud Architecture Center (docs-next) consists of the following 5 components:

1. **docs-next**: The documentation site, based on [Docusaurus](https://docusaurus.io/).
2. **typesense**: [Typesense](https://typesense.org/) is a modern, privacy-friendly, open-source search engine  
engineered for performance & ease-of-use.
3. **typesense-scraper**: The scraper that crawls the documentation site and feeds typesense.
4. **typesense-reverse-proxy**: The reverse-proxy component to securely publish typesense endpoints externally.
5. **typesense-dashboard**: An [unofficial Typesense dashboard](https://github.com/bfritscher/typesense-dashboard) to manage and browse collections.

The first one belongs to the chart **docs-next/docs-next** and 2,3 & 4 belong to the chart **docs-next/typesense**.

## Prerequisites

You are going to need:

1. Two(2) external-facing Elastic Load Balancers for the Ingress objects of **docs-next** and **typesense-reverse-proxy**. 
2. Two(2) FQDN names for the external URLs of those exposed services.

Next you need to:

3. Register an A-Record in Open Telekom Cloud DNS service binding the EIP of the load balancer with the FQDN.
4. Copy the ID of Elastic Load Balancer because you will need to provide it as a value to each chart.
5. Copy the FQDN because you will need to provide it the host value of each Ingress. 

Additionally, you are going to need two(2) API keys:

6. An **admin** API key for Typesense
7. A **search-scoped** API key for docs-next (You are going to create this using either **curl** or the provisioned **typesense-dashboard**)

> [!CRITICAL]
> Although you can use the **admin** key for both Typesense and documentation site it is highly discouraged to share 
> admin keys in a public facing site.

## Installation of the documentation site

```shell
helm repo add docs-next https://akyriako.github.io/docs-next-charts
helm repo update

helm upgrade --install docs-next docs-next/docs-next \
    --set docusaurus.ingress.elbid = <*docs-next elastic load balancer id*> \
    --set docusaurus.ingress.host = <*docs-next fqdn*> \
    --set docusaurus.docusaurus.env.typesenseHost = <*typesense-reverse-proxy fqdn*> \
    -n docs-next-preview \
    --create-namespace 
```

## Installation of the search engine

```shell
helm repo add docs-next https://akyriako.github.io/docs-next-charts
helm repo update

helm upgrade --install docs-next docs-next/typesense \
    --set typesenseReverseProxy.elbid = <*typesense-reverse-proxy elastic load balancer id*> \
    --set typesenseReverseProxy.host = <*typesense-reverse-proxy fqdn*> \
    --set docusaurus.host = <*docs-next fqdn*> \
    -n docs-next-preview \
    --create-namespace 
```
