helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace --controller.service.httpPort.port 8090



An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls






  kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80

  helm install camunda camundacloud/zeebe-cluster-helm -f clustervalues.yaml
  helm install operate camundacloud/zeebe-operate-helm -f operatevalues.yaml
  helm install tasklist camundacloud/zeebe-tasklist-helm -f tasklistvalues.yaml  


zeebe ingress
  kubectl create ingress zeebe --class=nginx --rule=zeebe.127.0.0.1/ nginx.ingress.kubernetes.io/backend-protocol: "GRPC" *=camunda-zeebe-gateway:26500

operate ingress
  kubectl create ingress operate --class=nginx --rule=operate.127.0.0.1/ *=operate-zeebe-operate-helm:8090

tasklist ingress
  kubectl create ingress operate --class=nginx --rule=operate.127.0.0.1/ *=tasklist-zeebe-tasklist-helm:8091
