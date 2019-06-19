# Kiali boookinfo 구축 및 UI



https://istio.io/docs/examples/bookinfo/#determining-the-ingress-ip-and-port

~~~
$ kubectl label namespace default istio-injection=enabled
~~~

~~~
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
~~~

~~~
$ kubectl get services
~~~

~~~
$ kubectl get pods
~~~

~~~
$ kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
~~~

~~~
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
~~~

~~~
$ kubectl get gateway
~~~

~~~
$ kubectl get svc istio-ingressgateway -n istio-system
~~~

~~~
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
~~~

~~~
$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
~~~

~~~
$ curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"
~~~

~~~
$ kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
~~~

~~~
$ kubectl get destinationrules -o yaml
~~~

~~~
$ kubectl -n istio-system get svc kiali
~~~

~~~
$ curl http://$GATEWAY_URL/productpage
~~~

~~~
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
~~~

[http://localhost:20001/kiali/console](http://localhost:20001/kiali/console)

