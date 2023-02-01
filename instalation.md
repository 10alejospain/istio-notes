# Instalation and setup

### There are different istio configuration profiles:
- **Default profile:** The default profile is meant for production deployments and deployments of primary clusters in multi-cluster scenarios. It deploys the control plane and ingress gateway.
- **Demo profile:** The demo profile is intended for demonstration deployments. It deploys the control plane and ingress and egress gateways and has a high level of tracing and access logging enabled.
- **Minimal profile:** The minimal profile is equivalent to the default profile but without the ingress gateway. It deploys the control plane.
- **External profile:** The external profile is used for configuring remote clusters in a multi-cluster scenario. It does not deploy any components.
- **Empty profile:** The empty profile is used as a base for custom configuration. It does not deploy any components.
- **Preview profile:** The preview profile contains experimental features. It deploys the control plane and ingress gateway



To set up a config file with the demo profile: create a file called _demo-profile.yaml_ with the following contents:

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: demo-installation
spec:
  profile: demo
```

Next run the command `istioctl install -f demo-profile.yaml`

### For enabling sidecar injection on a ns 

`kubectl label namespace default istio-injection=enabled`

To check that the namespace is labeled, run the command below:

`kubectl get namespace -L istio-injection`

To completely delete the istio instalations:

`istioctl x uninstall --purge`
