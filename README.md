# docs-next-charts

```shell
helm repo add docs-next https://akyriako.github.io/docs-next-charts
helm repo update

helm upgrade --install docs-next docs-next/docs-next -n docs-next-preview --create-namespace 
```