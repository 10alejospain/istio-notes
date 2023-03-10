# SPIRE 

<div align="center">
<img src="https://github.com/10alejospain/istio-notes/blob/main/images/spire-and-cert-manager.svg" alt="image by tetrate.io" width="auto" height="300">
</div>

## Installation quickstart guide example

1. Once the istio is [configured](https://github.com/10alejospain/istio-notes/blob/main/pages/instalation.md) navigate the Istio folder for auto-injection the next step is to set the __SPIRE CA__ and the __SPIRE Kubernetes Workload Registrar__

```shell
cd istio-1.16.*/

kubectl apply -f samples/security/spire/spire-quickstart.yaml
```

After seting the spire server deployed in the previous step with the quickstart run the following command 

```shell
istioctl install --skip-confirmation -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
spec:
  profile: default
  meshConfig:
    trustDomain: example.org
  values:
    global:
    # This is used to customize the sidecar template
    sidecarInjectorWebhook:
      templates:
        spire: |
          spec:
            containers:
            - name: istio-proxy
              volumeMounts:
              - name: workload-socket
                mountPath: /run/secrets/workload-spiffe-uds
                readOnly: true
            volumes:
              - name: workload-socket
                csi:
                  driver: "csi.spiffe.io"
  components:
    ingressGateways:
      - name: istio-ingressgateway
        enabled: true
        label:
          istio: ingressgateway
        k8s:
          overlays:
            - apiVersion: apps/v1
              kind: Deployment
              name: istio-ingressgateway
              patches:
                - path: spec.template.spec.volumes.[name:workload-socket]
                  value:
                    name: workload-socket
                    csi:
                      driver: "csi.spiffe.io"
                - path: spec.template.spec.containers.[name:istio-proxy].volumeMounts.[name:workload-socket]
                  value:
                    name: workload-socket
                    mountPath: "/run/secrets/workload-spiffe-uds"
                    readOnly: true
                - path: spec.template.spec.initContainers
                  value:
                    - name: wait-for-spire-socket
                      image: busybox:1.28
                      volumeMounts:
                        - name: workload-socket
                          mountPath: /run/secrets/workload-spiffe-uds
                          readOnly: true
                      env:
                        - name: CHECK_FILE
                          value: /run/secrets/workload-spiffe-uds/socket
                      command:
                        - sh
                        - "-c"
                        - |-
                          echo `date -Iseconds` Waiting for: ${CHECK_FILE}
                          while [[ ! -e ${CHECK_FILE} ]] ; do
                            echo `date -Iseconds` File does not exist: ${CHECK_FILE}
                            sleep 15
                          done
                          ls -l ${CHECK_FILE}
EOF
```

To check if the istio injection is enabled in the default namespace and spire namespace is created run

```shell 
kubectl get pods -n istio-system

kubectl get namespace -L istio-injection
```

The output should look like:

```console
NAME              STATUS   AGE     ISTIO-INJECTION
default           Active   x       enabled
istio-system      Active   x   
kube-node-lease   Active   x     
kube-public       Active   x     
kube-system       Active   x 
spire             Active   x
```

2. To check that the SPIRE server is installed run:

```
SPIRE_SERVER_POD=$(kubectl get pod -l app=spire-server -n spire -o jsonpath="{.items[0].metadata.name}")

kubectl exec -i -t $SPIRE_SERVER_POD -n spire -c spire-server -- /bin/sh -c "bin/spire-server entry show -socketPath /run/spire/sockets/server.sock"
```

<details><summary>The output should give something like this</summary>

```console

Entry ID         : 64d7f17d-a9a7-46b3-b27d-ff48623d8d54
SPIFFE ID        : spiffe://example.org/k8s-workload-registrar/demo-cluster/node/docker-desktop
Parent ID        : spiffe://example.org/spire/server
Revision         : 0
TTL              : default
Selector         : k8s_psat:agent_node_uid:e4d5a8fd-079d-4f2a-826a-459eb81c3a1f
Selector         : k8s_psat:cluster:demo-cluster

Entry ID         : bc568e88-c161-4a80-bd77-d0640d8739f6
SPIFFE ID        : spiffe://example.org/ns/istio-system/sa/istio-egressgateway-service-account
Parent ID        : spiffe://example.org/k8s-workload-registrar/demo-cluster/node/docker-desktop
Revision         : 1
TTL              : default
Selector         : k8s:node-name:docker-desktop
Selector         : k8s:ns:istio-system
Selector         : k8s:pod-uid:163e1795-4489-4aeb-b24e-2bd1df76d986
DNS name         : istio-egressgateway-76f4cfc696-zk79x
DNS name         : istio-egressgateway.istio-system.svc

Entry ID         : 3857ac7b-eb02-4500-a916-5cfe2a180e1e
SPIFFE ID        : spiffe://example.org/ns/istio-system/sa/istio-ingressgateway-service-account
Parent ID        : spiffe://example.org/k8s-workload-registrar/demo-cluster/node/docker-desktop
Revision         : 1
TTL              : default
Selector         : k8s:node-name:docker-desktop
Selector         : k8s:ns:istio-system
Selector         : k8s:pod-uid:c3c2e282-cc13-4dd8-a687-7ddbd2743a27
DNS name         : istio-ingressgateway-69db67f844-hr28q
DNS name         : istio-ingressgateway.istio-system.svc

Entry ID         : 6981b604-3d55-43f3-9da1-bd7566b5b16c
SPIFFE ID        : spiffe://example.org/ns/istio-system/sa/istiod
Parent ID        : spiffe://example.org/k8s-workload-registrar/demo-cluster/node/docker-desktop
Revision         : 1
TTL              : default
Selector         : k8s:node-name:docker-desktop
Selector         : k8s:ns:istio-system
Selector         : k8s:pod-uid:a77657f5-74f4-4ab9-ad88-91f9a9425e5a
DNS name         : istiod-5766658d88-7rzkj
DNS name         : istiod.istio-system.svc

Entry ID         : 031f3701-b432-4788-bff7-85beda37be2f
SPIFFE ID        : spiffe://example.org/ns/spire/sa/spire-agent
Parent ID        : spiffe://example.org/k8s-workload-registrar/demo-cluster/node/docker-desktop
Revision         : 0
TTL              : default
Selector         : k8s:node-name:docker-desktop
Selector         : k8s:ns:spire
Selector         : k8s:pod-uid:83960f6a-42e7-452d-84f6-64e4f90706fc
DNS name         : spire-agent-x7nqf

Entry ID         : 4bd0e8e0-1565-4910-bc40-9125a6514640
SPIFFE ID        : spiffe://example.org/ns/spire/sa/spire-server
Parent ID        : spiffe://example.org/k8s-workload-registrar/demo-cluster/node/docker-desktop
Revision         : 1
TTL              : default
Selector         : k8s:node-name:docker-desktop
Selector         : k8s:ns:spire
Selector         : k8s:pod-uid:43660165-a85b-4479-8a93-4c8972d8e612
DNS name         : spire-server-0
DNS name         : spire-server.spire.svc

```
  
</details>

After registering an entry for the Ingress-gateway pod, Envoy receives the identity issued by SPIRE and uses it for all __TLS and mTLS__ communications.

3. Deploying an example 

For the first example we can use the preconfigured _sleep-spire_ file provided by Istio

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
spec:
  replicas: 1
  selector:
      matchLabels:
        app: sleep
  template:
      metadata:
        labels:
          app: sleep
        # Injects custom sidecar template
        annotations:
            inject.istio.io/templates: "sidecar,spire"
      spec:
        terminationGracePeriodSeconds: 0
        serviceAccountName: sleep
        containers:
        - name: sleep
          image: curlimages/curl
          command: ["/bin/sleep", "3650d"]
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: tmp
              mountPath: /tmp
          securityContext:
            runAsUser: 1000
        volumes:
          - name: tmp
            emptyDir: {}
          # CSI volume
          - name: workload-socket
            csi:
              driver: "csi.spiffe.io"
              readOnly: true
```
Then we run the commands to set de variables to get the pod's name and id

```shell
SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath="{.items[0].metadata.name}")
SLEEP_POD_UID=$(kubectl get pods $SLEEP_POD -o jsonpath='{.metadata.uid}')
```

And register the workload in the spire server where we asing to the pod the _spiffe://example.org<...>_ url that was created by the quickstart spire file

```shell
kubectl exec -n spire spire-server-0 -- \
/opt/spire/bin/spire-server entry create \
    -spiffeID spiffe://example.org/ns/default/sa/sleep \
    -parentID spiffe://example.org/ns/spire/sa/spire-agent \
    -selector k8s:ns:default \
    -selector k8s:pod-uid:$SLEEP_POD_UID \
    -dns $SLEEP_POD \
    -socketPath /run/spire/sockets/server.sock
```
We can check that the workload waas issued by SPIRE by running 

```shell
istioctl proxy-config secret $SLEEP_POD -o json | jq -r \
'.dynamicActiveSecrets[0].secret.tlsCertificate.certificateChain.inlineBytes' | base64 --decode |  echo -
```
