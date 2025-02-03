# generic-cj-scaler

Generic chart to scale deployments or statefulsets using cronjobs.

## Example manual deployment

```bash
ENV='dev'
VERSION="1.0.0"
NAMESPACE="default"

helm upgrade --install -n "${NAMESPACE}" generic-cj-scaler . --version "${VERSION}" -f "${NAMESPACE}.values.${ENV}.yaml"
```
