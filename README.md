---

# Elasticsearch on Cloud Kuberenetes (ECK)
Built on the Kubernetes Operator pattern, [ECK] extends the basic Kubernetes orchestration capabilities to support the setup and management and security
* Elasticsearch
* Kibana
* [others]

This template only focus on `kind:elasticsearch` and `kind:kibana` resources 

NOTE: Operators/CRDs should be installed first
```
kubectl apply -f https://download.elastic.co/downloads/eck/0.9.0/all-in-one.yaml
``` 
### ES-Cluster
Ideally, the es-cluster should somewhat be separated either with `dedicated nodes` or a separate cluster. With 3 manster nodes and data-nodes (coordinator and ingest aslo supported)

##### Usage (es-master)
From [values.yaml](ansible/roles/kubernetes/addons/efk/values.yaml), you can choose to setup master with `ha` and/or `dedicated`. Null values will result to bolean false and will default for es cluster with role `master` and `data` in one pod.
```
master:
  ha: no # Set 'ha' (high avialability) to true: for 3 master nodes per zone
         # Set to bolean false: to merge master and data in one pod
  dedicated: {} #if master has dedicated nodes, set to true and add 'nodeSelector' % 'tolerations' values
```
##### Usage (es-data)
Same setup as master, you can setup `data`-node with its dedicated nodes (recommended)
```
data:
  dedicated: no 
  name: data-node
  count: 1
```
##### Secrets
By default, the operator will create elasticsearch *users* with roles and store them in kubernetes secret, to retrieve the `elastic` user, run
``` 
PASSWORD=$(kubectl -n monitoring get secret efk-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
```
##### Ingress: Exposing services
The operator will not create elasticsearch by default which is recommended, so you need to create an ingress separatly. You can expose (https by default) the cluster in your local by running
``` 
kubectl -n monitoring port-forward svc/efk-es-http 9200:9200  
```
### Kibana
Kibana resource is a bit straight forward, you can check advance configuration here 
https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-kibana.html

### Forwarder: fluentd: VS fluentbit: VS logstash:
There are alot of materials to which log forwarder to choose from, this template uses fluentd as it seems beneficial and more scalable/flexible for cloud projects. We can opt to disable it and use existing or different forwarder
```
fluentd:
  enabled: false
  namespace: monitoring
```

* https://www.youtube.com/watch?v=1ye0-sityBw
* https://www.youtube.com/watch?v=3BU-4CLRKhI 

### Improvements
Open to suggestions: Add necessary changes or manifest if needed and update the template.
* secure elasticsearch config (use secret or vault)
* update fluent-elasticsearch plugin v3.5 (needs rebuilding)
* enrich the data usng `filter` plugin if necessary

### Helm Chart
The official helm chart for ECK was not use, this is a custom helm chart as it's a new tech and codes are easier to break down since you only put what you need at the moment. Checkout the offcial helm installation
* https://www.elastic.co/guide/en/cloud-on-k8s/1.8/k8s-install-helm.html

```
helm install/upgrade -n monitoring --values efd/{env}.yaml efk
```

[ECK]: https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html
[others]: https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-overview.html
