# Cluster Ingress

install nginx with helm
```helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace```

forward the ingress port
`kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80`

# Install Camunda

## Install elastic search single node
had to disable elastic search in camunda cluster helm chart as it would not allow single node
running from a local chart

 ```helm install elastic ../elasticsearch/helm-charts/elasticsearch/ -f elasticvalues.yaml```
 TODO move elastic chart into this repo

## Install Camunda Charts
  helm install camunda camundacloud/zeebe-cluster-helm -f clustervalues.yaml
  helm install operate camundacloud/zeebe-operate-helm -f operatevalues.yaml
  helm install tasklist camundacloud/zeebe-tasklist-helm -f tasklistvalues.yaml  

## Create ingress for camunda
`kubectl create -f ui-ingress.yaml`
zeebe ingress not yet working
kubectl create -f zeebe-ingress.yaml


install the cloudevents router
kn service create zeebe-cloudevents --image pcrofts/zeebe-cloudevents:latest --port 8080





# Install Knative

kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.2.0/operator.yaml

kubectl create -f knative.yaml


Wait for above to finish before doing dns
check status
`kubectl get KnativeServing knative-serving -n knative-serving`

Set the context
kubectl config set-context --current --namespace=default

configure knative dns
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.2.0/serving-default-domain.yaml



create a default broker

`kn broker create default`

`kn service create cloudevents-player \
--image ruromero/cloudevents-player:latest \
--env BROKER_URL=http://broker-ingress.knative-eventing.svc.cluster.local/default/default`



# Distributed Tracing
Installing Jaeger

Step 1 install cert manager and open telemetry operator

```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml &&
kubectl -n cert-manager wait --for=condition=Ready pods --all &&
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/download/v0.40.0/opentelemetry-operator.yaml
```

install yaeger operator
```
kubectl create namespace observability &&
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.28.0/deploy/crds/jaegertracing.io_jaegers_crd.yaml &&
kubectl create -n observability \
    -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.28.0/deploy/service_account.yaml \
    -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.28.0/deploy/role.yaml \
    -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.28.0/deploy/role_binding.yaml \
    -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.28.0/deploy/operator.yaml
```

create the jaeger instance
```
kubectl apply -n observability -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simplest
EOF
```

port forward jaeger
`kubectl -n observability port-forward service/simplest-query 16686`


create the collector
```
kubectl apply -f - <<EOF
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel
  namespace: observability
spec:
  config: |
    receivers:
      zipkin:
    exporters:
      logging:
      jaeger:
        endpoint: "simplest-collector.observability:14250"
        tls:
          insecure: true
    service:
      pipelines:
        traces:
          receivers: [zipkin]
          processors: []
          exporters: [logging, jaeger]
EOF
```

configure knative for tracing
```
for ns in knative-eventing knative-serving; do
  kubectl patch --namespace "$ns" configmap/config-tracing \
   --type merge \
   --patch '{"data":{"backend":"zipkin","zipkin-endpoint":"http://otel-collector.observability:9411/api/v2/spans", "debug": "true"}}'
done
```

# Kubernetes Dashboard
kubernetes dashboard
 kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml

Add ingress and user
kubectl create -f k8sui.yaml

Get token
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
