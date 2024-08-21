Метрики обычно поставляются [метрик-сервером](https://kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/#metrics-server). Дальше я расскажу о замене метрик-сервера Prometheus’ом (потому что Prometheus реализует в себе данные, которые отдаются метрик-сервером и мы избавляемся от одного лишнего звена) и о том, как на основе метрик из Prometheus масштабировать свои приложения в Kubernetes.


Также удалите метрик-сервер, если он у вас есть.-- типо нужно удалять метрик сервер??


Разобраться, что означают урлы metrics.k8s.io​ и custom.metrics.k8s.io​ вам может помочь [документация](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-metrics-apis)​.



➜ kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "custom.metrics.k8s.io/v1beta1",
  "resources": []
}

kubectl get --raw "/apis/metrics.k8s.io/v1beta1/" | jq
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "metrics.k8s.io/v1beta1",
  "resources": [
    {
      "name": "nodes",
      "singularName": "",
      "namespaced": false,
      "kind": "NodeMetrics",
      "verbs": [
        "get",
        "list"
      ]
    },
    {
      "name": "pods",
      "singularName": "",
      "namespaced": true,
      "kind": "PodMetrics",
      "verbs": [
        "get",
        "list"
      ]
    }
  ]
}


Certainly! Let's break down your questions:

### 1. What is Kubernetes Metrics Server?
The Kubernetes Metrics Server is a cluster-wide aggregator of resource usage data, such as CPU and memory, within a Kubernetes cluster. It collects metrics from the kubelet on each node and exposes them through the Kubernetes API server. These metrics are typically used for Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA), as well as for the `kubectl top` command, which shows resource usage for nodes and pods.

**However, the Kubernetes Metrics Server provides only basic CPU and memory metrics. It doesn’t support custom metrics, which is why additional tools like Prometheus and the Prometheus Adapter are needed for more advanced use cases.**

**Does Prometheus Communicate with Kubernetes Metrics Server?**
No, Prometheus does not directly communicate with the Kubernetes Metrics Server. Prometheus is an independent system that scrapes metrics from various endpoints, including applications and the Kubernetes components themselves, by accessing the `/metrics` endpoint provided by the applications or Kubernetes components.

Prometheus stores these metrics in its time-series database**. To use Prometheus metrics for scaling, the Prometheus Adapter comes into play, which allows the Kubernetes API to query Prometheus metrics and use them in HPA.**

### 2. Why Should You Remove the Kubernetes Metrics Server Before Using Prometheus Adapter?
You generally don't need to remove the Kubernetes Metrics Server if you want to use the Prometheus Adapter, but it's important to understand why you might consider it.

If you're switching entirely to custom metrics for autoscaling, the Prometheus Adapter can replace the need for the Kubernetes Metrics Server. The adapter allows HPA to use custom metrics provided by Prometheus, which can include a wide range of application-level metrics beyond just CPU and memory usage.

However, if both the Kubernetes Metrics Server and the Prometheus Adapter are installed, you need to carefully configure your HPA resources to ensure they use the correct source of metrics. The potential conflict or confusion arises if the HPA references default resource metrics (like CPU and memory), as both the Metrics Server and Prometheus Adapter could be capable of providing this data, depending on your configuration.

In summary:
- **Kubernetes Metrics Server**: Provides basic CPU and memory metrics.
- **Prometheus Adapter**: Provides custom metrics (including those from Prometheus) for HPA.

You don’t necessarily need to remove the Kubernetes Metrics Server unless you want to simplify your environment or avoid conflicts. Instead, you should ensure that your HPA configurations are correctly set to use the metrics source you intend, whether that’s the Kubernetes Metrics Server or Prometheus (via Prometheus Adapter).



Оказывается 10s латенси на графиках в ingress-nginx это потолок и много кто с этим  ограничением сталкнулся 
https://github.com/kubernetes/ingress-nginx/issues/2924  - дело в какой кардинальности? вроде я это слово помню по следующему принципу - каждая секунда как бы отдельный параметр, по которому мы можем искать. у нас были проблемы, когда мы разибрались с закончившимися инодами в локи из-за айпи и прочих приколов дополнительный параметров для поиска, которые группировались. 



### Configuring the adapter

[](https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/config-walkthrough.md#configuring-the-adapter)

The adapter considers metrics in the following ways:

1. First, it discovers the metrics available (_Discovery_)
    
2. Then, it figures out which Kubernetes resources each metric is associated with (_Association_)
    
3. Then, it figures out how it should expose them to the custom metrics API (_Naming_)
    
4. Finally, it figures out how it should query Prometheus to get the actual numbers (_Querying_)



А нужно делать по поду или по чему время запроса? 



Вот у нас запрос 
nginx_ingress_controller_request_duration_seconds_bucket{app_kubernetes_io_component="controller", 
app_kubernetes_io_instance="ingress-nginx", 
app_kubernetes_io_managed_by="Helm", app_kubernetes_io_name="ingress-nginx", app_kubernetes_io_part_of="ingress-nginx", app_kubernetes_io_version="1.10.0", argocd_argoproj_io_instance="ingress-nginx", controller_class="k8s.io/ingress-nginx", controller_namespace="ingress-nginx", controller_pod="ingress-nginx-controller-v1-64b99bcb5f-4hnm5", helm_sh_chart="ingress-nginx-4.10.0", host="api.staging.thecrossingguardcompany.com", ingress="staging-backend-ingress-v2", instance="10.110.0.144:10254", job="kubernetes-service-endpoints", le="+Inf", method="GET", namespace="staging", node="ip-10-110-1-180.ec2.internal", path="/", service="staging-backend-service-v2", status="200"}


```yaml
      - seriesQuery: 'nginx_ingress_controller_request_duration_seconds_bucket'
        resources:
          overrides:  
            namespace:
              resource: namespace
            service:
              resource: service
            pod:
              resource: pod
        name:
          matches: "^(.*)_bucket"
          as: "${1}_avg"
        metricsQuery: 'histogram_quantile(0.99,sum by (le)(rate(nginx_ingress_controller_request_duration_seconds_bucket{namespace="staging"}[1m])))' 
```

крч, когда я это применил, похоже на то, что мы реально должны пойти и посмотрел, как поля содержит метрика nginx_ingress_controller_request_duration_seconds_bucket

потому что вот какой результат я получил 
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "custom.metrics.k8s.io/v1beta1",
  "resources": [
    {
      "name": "services/nginx_ingress_controller_request_duration_seconds_avg",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "namespaces/nginx_ingress_controller_request_duration_seconds_avg",
      "singularName": "",
      "namespaced": false,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    }
  ]
}

выше нету инфы по подам и сам прометеус по метрике не предоставляет инфу о том, что делать. 

теперь как-то надо достучаться до метрики, 
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/staging/services/\*/nginx_ingress_controller_request_duration_seconds_avg" | jq
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {},
  "items": []
}


если напрямую вызывать, то вот такое получается 
```
on ☸ teleport.staging.thecrossingguardcompany.com-teleport.staging.thecrossingguardcompany.com (staging) ~ 
➜ kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/staging/services/staging-backend-service-v2/nginx_ingress_controller_request_duration_seconds_avg" | jq
Error from server (NotFound): the server could not find the metric nginx_ingress_controller_request_duration_seconds_avg for services staging-backend-service-v2

```


крч хочу запустить просто пример http_requests_total из доки 
{__name__="http_requests_total", app_kubernetes_io_component="dex-server", app_kubernetes_io_instance="argo-cd", app_kubernetes_io_managed_by="Helm", app_kubernetes_io_name="argocd-dex-server", app_kubernetes_io_part_of="argocd", app_kubernetes_io_version="v2.10.6", argocd_argoproj_io_instance="argo-cd", code="302", handler="/auth", helm_sh_chart="argo-cd-6.7.11", instance="10.110.2.133:5558", job="kubernetes-service-endpoints", method="GET", namespace="argo-cd", node="ip-10-110-1-180.ec2.internal", service="argo-cd-argocd-dex-server"}


опана 

```yaml
      - seriesQuery: 'http_requests_total{namespace!="",service!=""}'
        resources:
          overrides:
            namespace: {resource: "namespace"}
            service: {resource: "pod"}
        name:
          matches: "^(.*)_total"
          as: "${1}_per_second"
        metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[2m])) by (<<.GroupBy>>)'
```



```shell
➜ kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "custom.metrics.k8s.io/v1beta1",
  "resources": [
    {
      "name": "services/nginx_ingress_controller_request_duration_seconds_avg",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "namespaces/http_requests_per_second",
      "singularName": "",
      "namespaced": false,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "pods/http_requests_per_second",
      "singularName": "",
      "namespaced": true,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "namespaces/nginx_ingress_controller_request_duration_seconds_avg",
      "singularName": "",
      "namespaced": false,
      "kind": "MetricValueList",
      "verbs": [
        "get"
      ]
    }
  ]
}

```


Actually namespace prefixed metrics are special, we should access them with `kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/*/metrics/node_memory_PageTables_bytes`.



Опана, для второй метрики чота отработало с аргоцд)) 

```shell
on ☸ teleport.staging.thecrossingguardcompany.com-teleport.staging.thecrossingguardcompany.com (staging) ~ 
✗  kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/*/metrics/nginx_ingress_controller_request_duration_seconds_avg" | jq
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {},
  "items": []
}

on ☸ teleport.staging.thecrossingguardcompany.com-teleport.staging.thecrossingguardcompany.com (staging) ~ 
➜ kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/*/metrics/http_requests_per_second" | jq
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {},
  "items": [
    {
      "describedObject": {
        "kind": "Namespace",
        "name": "argo-cd",
        "apiVersion": "/v1"
      },
      "metricName": "http_requests_per_second",
      "timestamp": "2024-08-19T09:57:39Z",
      "value": "0",
      "selector": null
    }
  ]
}

```

Почему только namespace с argocd конечно - вопрос 

https://habr.com/ru/companies/tochka/articles/685636/

```shell
➜ kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/*/metrics/http_requests_per_second" | jq
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {},
  "items": [
    {
      "describedObject": {
        "kind": "Namespace",
        "name": "argo-cd",
        "apiVersion": "/v1"
      },
      "metricName": "http_requests_per_second",
      "timestamp": "2024-08-21T12:30:49Z",
      "value": "0",
      "selector": null
    }
  ]
}

превратилось в 
sum(rate(http_requests_total{namespace=~"argo-cd|cert-manager|ci|default|grafana|ingress-nginx|karpenter|kube-node-lease|kube-public|kube-system|metrics-server|mongodb|monitoring|ops|opsgenie|prometheus|staging|tekton-pipelines|teleport-cluster|utils|velero"}[2m])) by (namespace)


```


```
I0821 13:03:11.880435       1 handler.go:143] prometheus-metrics-adapter: GET "/apis/custom.metrics.k8s.io/v1beta1/namespaces/staging/services/staging-backend-service-v2/nginx_ingress_controller_request_duration_seconds_p50" satisfied by gorestful with webservice /apis/custom.metrics.k8s.io/v1beta1
E0821 13:03:11.880869       1 series_registry.go:164] unable to construct query for metric services/nginx_ingress_controller_request_duration_seconds_p50(namespaced): unable to convert resource namespaces into label: no generic resource label form specified for this metric

```
  ошибка выше - я пытаюсь обратиться к ресрусу сервис в неймспейсе, но я не указал, как конвертировать в секции overrides ресурс namespace в лейбл для запроса прометеуса. 

следующая ошика 
metricsQuery: 'histogram_quantile(0.50, sum(rate(<<.Series>>{<<.LabelMatchers>>}[<<.Range>>])) by (le))'

```
E0821 13:09:36.917877       1 series_registry.go:164] unable to construct query for metric services/nginx_ingress_controller_request_duration_seconds_p50(namespaced): template: metrics-query:1:68: executing "metrics-query" at <.Range>: can't evaluate field Range in type naming.queryTemplateArgs

```


потом 

```shel

I0821 13:12:57.207570       1 handler.go:143] prometheus-metrics-adapter: GET "/apis/custom.metrics.k8s.io/v1beta1/namespaces/staging/services/staging-backend-service-v2/nginx_ingress_controller_request_duration_seconds_p50" satisfied by gorestful with webservice /apis/custom.metrics.k8s.io/v1beta1
I0821 13:12:57.210816       1 api.go:88] GET http://prometheus-server.prometheus:80/api/v1/query?query=histogram_quantile%280.50%2C+sum%28rate%28nginx_ingress_controller_request_duration_seconds_bucket%7Bnamespace%3D%22staging%22%2Cservice%3D%22staging-backend-service-v2%22%7D%5B2m%5D%29%29+by+%28le%29%29&time=1724245977.207&timeout= 200 OK
	E0821 13:12:57.211146       1 provider.go:186] None of the results returned by when fetching metric services/nginx_ingress_controller_request_duration_seconds_p50(namespaced) for "staging/staging-backend-service-v2" matched the resource name


histogram_quantile(0.50, sum(rate(nginx_ingress_controller_request_duration_seconds_bucket{namespace="staging",service="staging-backend-service-v2"}[2m])) by (le))
```






```
    - seriesQuery: 'nginx_ingress_controller_request_duration_seconds_bucket'
      resources:
        overrides:
          service: {resource: "service"}
          ingress: {resource: "ingress"}
          namespace: {resource: "namespace"}
      name:
        matches: "^(.*)_bucket"
        as: "${1}_p50"
      metricsQuery: 'histogram_quantile(0.50, sum(rate(<<.Series>>{<<.LabelMatchers>>}[2m])) by (le))'
```
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {},
        "value": [
          1724246141.373,
          "1.75"
        ]
      }
    ]
  }
}



А вот для этого запроса 

```
    - seriesQuery: 'http_requests_total{namespace!="",service!=""}'
      resources:
        overrides:
          service: {resource: "service"}
          namespace: {resource: "namespace"}
      name:
        matches: "^(.*)_total"
        as: "${1}_per_second"
      metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[2m])) by (<<.GroupBy>>)'
```

{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "service": "argo-cd-argocd-dex-server"
        },
        "value": [
          1724253266.793,
          "0"
        ]
      }
    ]
  }
}


**Вероятно дело в <<.GroupBy>> **

добавляя это, мы видим вот такое 
![[Pasted image 20240821222030.png]]


```
E0821 16:02:13.880136       1 provider.go:198] unable to list matching resource names: ingresses.networking.k8s.io is forbidden: User "system:serviceaccount:prometheus:prometheus-adapter" cannot list resource "ingresses" in API group "networking.k8s.io" in the namespace "staging"

```