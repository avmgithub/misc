## References
- https://docs.openshift.com/container-platform/4.12/logging/cluster-logging-deploying.html
- https://docs.openshift.com/container-platform/4.12/logging/cluster-logging-external.html


### Create namespace for elastic search operator.

```
oc create -f eo-namespace.yaml
```

File: eo-namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-operators-redhat
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
```



### Create namespace for Logging Operator.

```
oc create -f olo-namespace.yaml
```

File: olo-namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
```

### Create operator group for elastic search operator

```
oc create -f eo-og.yaml
```

File: eo-og.yaml
```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-operators-redhat
  namespace: openshift-operators-redhat
spec: {}
```


### Create subscription for elastic search operator

```
oc create -f eo-sub.yaml
```

File: eo-sub.yaml  
```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: "elasticsearch-operator"
  namespace: "openshift-operators-redhat" 
spec:
  channel: "stable-5.5" 
  installPlanApproval: "Automatic" 
  source: "redhat-operators" 
  sourceNamespace: "openshift-marketplace"
  name: "elasticsearch-operator"
```

### Verify operator installation
```
oc get csv --all-namespaces
```

### Create logging operator group
```
oc create -f olo-og.yaml
```

File : olo-og.yaml
```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  targetNamespaces:
  - openshift-logging
```


### Create subscription for logging operator

```
oc create -f olo-sub.yaml
```

File: olo-sub.yaml
```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  channel: "stable"
  name: cluster-logging
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```


### Verify operator installation
```
oc get csv -n openshift-logging
```

### Create instance of logging (NOTE: This may not be needed as we will not use the OpenShift console)
```
oc create -f olo-instance.yaml
```


Logging operator instance file: olo-instance.yaml 
```
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogging"
metadata:
  name: "instance"
  namespace: "openshift-logging"
spec:
  managementState: "Managed"
  logStore:
    type: "elasticsearch"
    retentionPolicy:
      application:
        maxAge: 1d
      infra:
        maxAge: 7d
      audit:
        maxAge: 7d
    elasticsearch:
      nodeCount: 3
      storage:
        storageClassName: "<storage-class-name>"
        size: 200G
      resources:
        limits:
          memory: "16Gi"
        requests:
          memory: "16Gi"
      proxy:
        resources:
          limits:
            memory: 256Mi
          requests:
             memory: 256Mi
      redundancyPolicy: "SingleRedundancy"
  visualization:
    type: "kibana"
    kibana:
      replicas: 1
  collection:
    logs:
      type: "fluentd"
      fluentd: {}
```



### Create instance of logging forwarding 
```
oc create -f logfwd-instance.yaml
```

File: logfwd-instance.yaml
```
apiVersion: "logging.openshift.io/v1"
kind: ClusterLogForwarder
metadata:
  name: instance
  namespace: openshift-logging
spec:
  outputs:
   - name: elasticsearch-secure
     type: "elasticsearch"
     url: https://elasticsearch.secure.com:9200
     secret:
        name: elasticsearch
   - name: elasticsearch-insecure
     type: "elasticsearch"
     url: http://elasticsearch.insecure.com:9200
   - name: kafka-app
     type: "kafka"
     url: tls://kafka.secure.com:9093/app-topic
  inputs:
   - name: my-app-logs
     application:
        namespaces:
        - my-project
  pipelines:
   - name: audit-logs
     inputRefs:
      - audit
     outputRefs:
      - elasticsearch-secure
      - default
     parse: json
     labels:
       secure: "true"
       datacenter: "east"
   - name: infrastructure-logs
     inputRefs:
      - infrastructure
     outputRefs:
      - elasticsearch-insecure
     labels:
       datacenter: "west"
   - name: my-app
     inputRefs:
      - my-app-logs
     outputRefs:
      - default
   - inputRefs:
      - application
     outputRefs:
      - kafka-app
     labels:
       datacenter: "south"
```



