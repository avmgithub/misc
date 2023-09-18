## References
- https://docs.openshift.com/container-platform/4.10/logging/cluster-logging-deploying.html
- https://docs.openshift.com/container-platform/4.10/logging/cluster-logging-external.html


Namespace for logging file: logging-namespace.yaml
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

Elastic search operator file: eo-og.yaml
```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-operators-redhat
  namespace: openshift-operators-redhat
spec: {}
```

Elastic Search subscrption file: eo-sub.yaml  
```
kind: Subscription
metadata:
  name: "elasticsearch-operator"
  namespace: "openshift-operators-redhat"
spec:
  channel: "stable-5.1"
  installPlanApproval: "Automatic"
  source: "redhat-operators"
  sourceNamespace: "openshift-marketplace"
  name: "elasticsearch-operator"
```





Namespace for Logging Operator Instance file: olo-namespace.yaml
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

Logging operator group file : olo-og.yaml
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


Logging operator subscription file: olo-sub.yaml
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


Log forwarding instance file: logfwd-instance.yaml
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



