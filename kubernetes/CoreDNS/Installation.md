# CoreDNS Installation

This guide will show how to implement coreDNS on K8s Cluster. This implementation apply for RBAC Cluster configuration.

## Steps to execution

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **1.** Create Service Account.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **2.** Create Role and Role Binding.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **3.** Create ConfigMap

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **4.** Create Deployment

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **5.** Create Service

For this case, we have created a yaml file, from the node where kubectl is installed (this is an external node of the cluster). The file was named ```deploy_coredns.yaml```, and its specification is found below

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: coredns
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:coredns
rules:
- apiGroups:
  - ""
  resources:
  - endpoints
  - services
  - pods
  - namespaces
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:coredns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:coredns
subjects:
- kind: ServiceAccount
  name: coredns
  namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: coredns
  namespace: kube-system
  labels:
    k8s-app: corednsF
    kubernetes.io/name: "CoreDNS"
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      k8s-app: coredns
  template:
    metadata:
      labels:
        k8s-app: coredns
    spec:
      serviceAccountName: coredns
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
        - key: "CriticalAddonsOnly"
          operator: "Exists"
      containers:
      - name: coredns
        image: coredns/coredns:1.2.2
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
        args: [ "-conf", "/etc/coredns/Corefile" ]
        volumeMounts:
        - name: config-volume
          mountPath: /etc/coredns
          readOnly: true
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9153
          name: metrics
          protocol: TCP
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - all
          readOnlyRootFilesystem: true
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
      dnsPolicy: Default
      volumes:
        - name: config-volume
          configMap:
            name: coredns
            items:
            - key: Corefile
              path: Corefile
---
apiVersion: v1
kind: Service
metadata:
  name: coredns
  namespace: kube-system
  annotations:
    prometheus.io/port: "9153"
    prometheus.io/scrape: "true"
  labels:
    k8s-app: coredns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "CoreDNS"
spec:
  selector:
    k8s-app: coredns
  clusterIP: 10.252.0.10
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
    protocol: TCP
```

Finally, we have to apply the changes with the command 
```command
kubectl apply -f deploy_coredns.yaml
```


## Testing Application

For testing this Add-on, is necessary deploy a new service in the cluster, with the purpose of do called between them. See the [guide](../my_first_app_on_k8s)

After that, we have to connect into container for consume other service.
```
kubectl exec -it {POD_NAME} bash
```

or 

```
kubectl exec -it {POD_NAME} sh
```

Then, we have to execute 

```
ping coredns.kube-system
```

The shell will show the IP address that correspond to service

![Image of K8S_Cluster](../images/K8S_coredns.png)

Yo can use ```curl```, ```telnet``` , ```ncat``` or ```nc``` for consume other service in the cluster.