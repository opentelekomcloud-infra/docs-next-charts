# Open Telekom Cloud Architecture Center Helm Charts

Open Telekom Cloud Architecture Center (docs-next) consists of the following 4 components:

1. **docs-next**: The documentation site, based on [Docusaurus](https://docusaurus.io/).
2. **typesense**: [Typesense](https://typesense.org/) is a modern, privacy-friendly, open-source search engine  
engineered for performance & ease-of-use.
3. **typesense-scraper**: The scraper that crawls the documentation site and feeds typesense.
4. **typesense-reverse-proxy**: The reverse-proxy component to securely publish typesense endpoints externally.

The first one belongs to the chart **docs-next/docs-next** and 2,3 & 4 belong to the chart docs-next/typesense.

## Installation of documentation site

```shell
helm repo add docs-next https://akyriako.github.io/docs-next-charts
helm repo update

helm upgrade --install docs-next docs-next/docs-next \
    -n docs-next-preview \
    --create-namespace 
```

## Installation of seach engine

```shell
helm repo add docs-next https://akyriako.github.io/docs-next-charts
helm repo update

helm upgrade --install docs-next docs-next/typesense \
    -n docs-next-preview \
    --create-namespace 
```
