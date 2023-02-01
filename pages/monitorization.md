# Monitorization tools and commands

Envoy administrative dashboard corresponding to a specific pod or deployment

`istioctl dashboard envoy deploy/productpage-v1.default`

The Istio standar metrics are listed [here](https://istio.io/latest/docs/reference/config/metrics/)

## Prometheus

Using Prometheus for metrics is avaible and its dashboard can be invoked by

`istioctl dashboard prometheus`

For further query examples in prometheus listed [here](https://prometheus.io/docs/prometheus/latest/querying/examples/)

## Grafana

Grafana is a popular open-source tool that makes it easy to construct custom monitoring dashboards from a backing metrics source. Grafana has built-in support for Prometheus.

Launch the Grafana UI with the following command:

`istioctl dashboard grafana`

## Kiali

Kiali is an open-source graphical console specifically designed for Istio and includes numerous features.

Through alerts and warnings, it can help validate that the service mesh configuration is correct and that it does not have any problems.

With Kiali, one can view Istio custom resources, services, workloads, or applications.

As an alternative to drafting and applying Istio custom resources by hand, Kiali exposes actions that allow the operator to define routing rules, perform traffic shifting, configure timeouts and inject faults.

Kiali relies on the metrics collected in Prometheus. In addition, Kiali has the ability to combine information from metrics, traces, and logs to provide deeper insight into the functioning of the mesh.

`istioctl dashboard kiali`