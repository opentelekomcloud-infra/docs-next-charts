# Open Telekom Cloud Architecture Center Helm Charts

Open Telekom Cloud Architecture Center (docs-next) consists of the following 4 components:

1. **docs-next**: The documentation site, based on Docusaurus 
2. **typesense**: The 
3. **typesense-scraper**:
4. **typesense-reverse-proxy**:

```shell
helm repo add docs-next https://akyriako.github.io/docs-next-charts
helm repo update

helm upgrade --install docs-next docs-next/docs-next -n docs-next-preview --create-namespace 
```