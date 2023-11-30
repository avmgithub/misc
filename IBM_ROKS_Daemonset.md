## Creating a daemonset for IBM Cloud OpenShift without machine config

Reference: [link](https://cloud.ibm.com/docs/openshift?topic=openshift-rhcos-performance)

### Deploying the Node Feature Discovery Operator
Before you can enable NUMA, CPU pinning, and huge pages on your worker nodes, you must deploy the Node Feature Discovery Operator. For more information, see The [Node Feature Discovery Operator](https://docs.openshift.com/container-platform/4.8/scalability_and_performance/psap-node-feature-discovery-operator.html).

### Enabling non-uniform memory access (NUMA), CPU pinning, and huge pages on your worker nodes

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ibm-user-custom-configurator-privileged
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:openshift:scc:privileged
subjects:
  - kind: ServiceAccount
    name: ibm-user-custom-configurator
    namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ibm-user-custom-configurator
  namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: ibm-user-custom-configurator
  namespace: kube-system
data:
  89-hugepages.conf: |
    vm.nr_hugepages=<<NUMBER_OF_HUGEPAGES>>
  configure.sh: |
    #!/usr/bin/env bash
    set -x
    cp -f /scripts/ibm-user-custom-configuration.sh /host-usr-local-bin/ibm-user-custom-configuration.sh
    chmod 0755 /host-usr-local-bin/ibm-user-custom-configuration.sh
    cp -f /scripts/ibm-user-custom-configuration.service /host-etc-systemd-dir/ibm-user-custom-configuration.service
    chmod 0644 /host-etc-systemd-dir/ibm-user-custom-configuration.service
    if [[ -f /scripts/89-hugepages.conf ]]; then
      cp -f /scripts/89-hugepages.conf /host-etc-systctld-dir/89-hugepages.conf
    fi
    nsenter -t 1 -m -u -i -n -p -- systemctl daemon-reload
    nsenter -t 1 -m -u -i -n -p -- systemctl enable ibm-user-custom-configuration.service
    nsenter -t 1 -m -u -i -n -p -- systemctl start ibm-user-custom-configuration.service
  ibm-user-custom-configuration.sh: |
    #!/usr/bin/env bash
    set -x
    GIGABYTES_RESERVED_MEMORY=$(echo $SYSTEM_RESERVED_MEMORY | awk -F 'Gi' '{print $1}')
    GIGABYTES_RESERVED_MEMORY_ROUNDED_UP=$(echo $GIGABYTES_RESERVED_MEMORY | awk '{print int($1+0.999)}')
    sed -i "s/SYSTEM_RESERVED_MEMORY=.*/SYSTEM_RESERVED_MEMORY=${GIGABYTES_RESERVED_MEMORY_ROUNDED_UP}Gi/g" /etc/node-sizing.env
    TOTAL_NUMA_MEMORY_TO_ALLOCATE=$(echo "$GIGABYTES_RESERVED_MEMORY_ROUNDED_UP" "1024" | awk '{print $1 * $2 + 100}')
    cat >/tmp/ibm-user-config.conf <<EOF
    #START USER CONFIG
    topologyManagerPolicy: <<TOPOLOGY_MANAGER_POLICY_VALUE>>
    memoryManagerPolicy: Static
    cpuManagerPolicy: static
    reservedMemory:
      - numaNode: 0
        limits:
          memory: ${TOTAL_NUMA_MEMORY_TO_ALLOCATE}Mi
    #END USER CONFIG
    EOF
    sed -i '/#START USER CONFIG/,/#END USER CONFIG/d' /etc/kubernetes/kubelet.conf
    cat /tmp/ibm-user-config.conf >>/etc/kubernetes/kubelet.conf
  ibm-user-custom-configuration.service: |
    [Unit]
    Description=Add custom user config to kubelet
    Before=kubelet.service
    After=kubelet-auto-node-size.service
    
    [Service]
    Type=oneshot
    RemainAfterExit=yes
    EnvironmentFile=/etc/node-sizing.env
    ExecStart=/usr/local/bin/ibm-user-custom-configuration.sh
    
    [Install]
    WantedBy=multi-user.target
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: ibm-user-custom-configurator
  name: ibm-user-custom-configurator
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: ibm-user-custom-configurator
  template:
    metadata:
      labels:
        app: ibm-user-custom-configurator
    spec:
      nodeSelector:
        feature.node.kubernetes.io/memory-numa: "true"
        ibm-cloud.kubernetes.io/os: RHCOS
      tolerations:
        - operator: "Exists"
      hostPID: true
      serviceAccount: ibm-user-custom-configurator
      initContainers:
        - name: configure
          image: "registry.access.redhat.com/ubi8/ubi:8.6"
          command: ['/bin/bash', '-c', 'mkdir /cache && cp /scripts/configure.sh /cache && chmod +x /cache/configure.sh && /bin/bash /cache/configure.sh']
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /scripts
              name: script-config
            - mountPath: /host-etc-systemd-dir
              name: etc-systemd-dir
            - mountPath: /host-usr-local-bin
              name: usr-local-bin
            - mountPath: /host-etc-systctld-dir
              name: etc-systctld-dir
      containers:
        - name: pause
          image: registry.ng.bluemix.net/armada-master/pause:3.2
      volumes:
        - name: etc-systemd-dir
          hostPath:
            path: /etc/systemd/system
        - name: etc-systctld-dir
          hostPath:
            path: /etc/sysctl.d
        - name: usr-local-bin
          hostPath:
            path: /usr/local/bin
        - name: script-config
          configMap:
            name: ibm-user-custom-configurator
```

### CPU pinning

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ibm-user-custom-configurator-privileged
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:openshift:scc:privileged
subjects:
  - kind: ServiceAccount
    name: ibm-user-custom-configurator
    namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ibm-user-custom-configurator
  namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: ibm-user-custom-configurator
  namespace: kube-system
data:
  89-hugepages.conf: |
    vm.nr_hugepages=<<NUMBER_OF_HUGEPAGES>>
  configure.sh: |
    #!/usr/bin/env bash
    set -x
    cp -f /scripts/ibm-user-custom-configuration.sh /host-usr-local-bin/ibm-user-custom-configuration.sh
    chmod 0755 /host-usr-local-bin/ibm-user-custom-configuration.sh
    cp -f /scripts/ibm-user-custom-configuration.service /host-etc-systemd-dir/ibm-user-custom-configuration.service
    chmod 0644 /host-etc-systemd-dir/ibm-user-custom-configuration.service
    if [[ -f /scripts/89-hugepages.conf ]]; then
      cp -f /scripts/89-hugepages.conf /host-etc-systctld-dir/89-hugepages.conf
    fi
    nsenter -t 1 -m -u -i -n -p -- systemctl daemon-reload
    nsenter -t 1 -m -u -i -n -p -- systemctl enable ibm-user-custom-configuration.service
    nsenter -t 1 -m -u -i -n -p -- systemctl start ibm-user-custom-configuration.service
  ibm-user-custom-configuration.sh: |
    #!/usr/bin/env bash
    set -x
    cat >/tmp/ibm-user-config.conf <<EOF
    #START USER CONFIG
    cpuManagerPolicy: static
    #END USER CONFIG
    EOF
    sed -i '/#START USER CONFIG/,/#END USER CONFIG/d' /etc/kubernetes/kubelet.conf
    cat /tmp/ibm-user-config.conf >>/etc/kubernetes/kubelet.conf
  ibm-user-custom-configuration.service: |
    [Unit]
    Description=Add custom user config to kubelet
    Before=kubelet.service
    After=kubelet-auto-node-size.service
    
    [Service]
    Type=oneshot
    RemainAfterExit=yes
    EnvironmentFile=/etc/node-sizing.env
    ExecStart=/usr/local/bin/ibm-user-custom-configuration.sh    
    [Install]
    WantedBy=multi-user.target
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: ibm-user-custom-configurator
  name: ibm-user-custom-configurator
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: ibm-user-custom-configurator
  template:
    metadata:
      labels:
        app: ibm-user-custom-configurator
    spec:
      nodeSelector:
        ibm-cloud.kubernetes.io/os: RHCOS
      tolerations:
        - operator: "Exists"
      hostPID: true
      serviceAccount: ibm-user-custom-configurator
      initContainers:
        - name: configure
          image: "registry.access.redhat.com/ubi8/ubi:8.6"
          command: ['/bin/bash', '-c', 'mkdir /cache && cp /scripts/configure.sh /cache && chmod +x /cache/configure.sh && /bin/bash /cache/configure.sh']
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /scripts
              name: script-config
            - mountPath: /host-etc-systemd-dir
              name: etc-systemd-dir
            - mountPath: /host-usr-local-bin
              name: usr-local-bin
            - mountPath: /host-etc-systctld-dir
              name: etc-systctld-dir
      containers:
        - name: pause
          image: registry.ng.bluemix.net/armada-master/pause:3.2
      volumes:
        - name: etc-systemd-dir
          hostPath:
            path: /etc/systemd/system
        - name: etc-systctld-dir
          hostPath:
            path: /etc/sysctl.d
        - name: usr-local-bin
          hostPath:
            path: /usr/local/bin
        - name: script-config
          configMap:
            name: ibm-user-custom-configurator
```
