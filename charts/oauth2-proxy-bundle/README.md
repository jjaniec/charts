# oauth2-proxy-bundle

The `oauth2-proxy-bundle` Helm chart secures access to internal Kubernetes services by enforcing OAuth2 authentication with a specific AD group. It deploys multiple `oauth2-proxy` instances to proxy traffic to various services, ensuring only authenticated users from the specified AD group can access them.

For example this chart can be used to give restricted access to developers to specific workloads in the cluster.

## Prerequisites

- Kubernetes cluster
- Helm 3.x
- OAuth2 provider (e.g., Dex, Keycloak) configured with your AD
- HelmRelease CRD with flux2 installed

## Installation

1. **Add the Helm repository**:
    ```sh
    helm repo add your-repo https://your-helm-repo-url
    helm repo update
    ```

2. **Customize values**:
    Create a `values.yaml` file with your configuration. For example:
    ```yaml
	globalSettings:
	namespaceOverride: tooling
	oidcIssuerUrl: https://sso.company.xyz/application/o/oauth2-proxy-generic/
	scope: "openid email groups"
	allowedGroups: "developers"
	oauthConfig:
		clientID: "xxx"
		clientSecret: "xxx"

	baseHost: kube-internals.company.xyz
	ingressClassName: "cilium"
	ingressAnnotations:
		cert-manager.io/cluster-issuer: cloudflare-prod

	## oauth2-proxy helm chart configuration
	# Helm chart version of oauth2-proxy to use
	helmChartVersion: '7.10.2'
	# Extra values to pass to the oauth2-proxy helm release
	extraValues:
		autoscaling:
		enabled: false
		minReplicas: 1
		metrics:
		enabled: true
		resources:
		limits:
			cpu: 200m
		requests:
			cpu: 50m
			memory: 100Mi

    oauth2proxies:
    - name: service1
      targetSvc: service1.default:80
    - name: service2
      targetSvc: service2.default:80
    - name: service3
      targetSvc: service3.namespace:12345
      # host, allowedGroups and oauthConfig for example can also be specified per proxy, overriding global settings
      host: service3.kube-internals.company.xyz
      allowedGroups: "kube-admins"
      oauthConfig:
        clientID: "service2-client-id"
        clientSecret: "service2-client-secret"
    ```

3. **Deploy the chart**:
    ```sh
    helm upgrade --install oauth2-proxy-bundle your-repo/oauth2-proxy-bundle -f values.yaml
    ```

## Configuration

### Global Settings

- `oidcIssuerUrl`: URL of the OIDC issuer.
- `scope`: Scope to request from the OIDC provider.
- `allowedGroups`: AD groups allowed to access the services.
- `oauthConfig.clientID`: OAuth2 client ID.
- `oauthConfig.clientSecret`: OAuth2 client secret.
- `baseHost`: Base host for generating service URLs.
- `ingressClassName`: Ingress class to use.
- `ingressAnnotations`: Annotations for ingress resources.
- `helmChartVersion`: Version of the `oauth2-proxy` Helm chart.
- `extraValues`: Extra values for the `oauth2-proxy` Helm release.

### OAuth2 Proxies

Each entry in the `oauth2proxies` list represents an `oauth2-proxy` instance to deploy. Configure the following fields:

- `name`: Name of the proxy.
- `targetSvc`: Target Kubernetes service to proxy traffic to.
- `host`: (Optional) Host for the proxy. If not defined, it will be generated as `<name>.<baseHost>`.
- `allowedGroups`: (Optional) AD groups allowed to access this proxy.
- `oauthConfig.clientID`: (Optional) OAuth2 client ID for this proxy.
- `oauthConfig.clientSecret`: (Optional) OAuth2 client secret for this proxy.

