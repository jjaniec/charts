# generic-cj-scaler

Generic chart to scale deployments or statefulsets using cronjobs.

## Example manual deployment

```bash
ENV='dev'
VERSION="0.1.0"
NAMESPACE="tooling"

helm upgrade --install -n "${NAMESPACE}" generic-cj-scaler . --version "${VERSION}" -f "${NAMESPACE}.values.${ENV}.yaml"
```
