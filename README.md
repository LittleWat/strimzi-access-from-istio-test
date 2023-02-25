# strimzi-access-from-istio-test

This is a sample repo to access kafka deployed by strimzi-operator from the pod that has istio-proxy sidecar.
This does not work as expected and now I need help to resolve the issue.

## setup

### start minikube

```shell
minikube start --memory=6144
```

### start istio

```shell
istioctl install --set profile=demo -y

kubectl apply -f ./k8s/istio-addons
```

### start cert-manager

```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml

kubectl wait --for=condition=Available --timeout=300s -n cert-manager deployment/cert-manager
kubectl wait --for=condition=Available --timeout=300s -n cert-manager deployment/cert-manager-cainjector
kubectl wait --for=condition=Available --timeout=300s -n cert-manager deployment/cert-manager-webhook

# create CA
kubectl apply -f ./k8s/ca-key-pair.yaml
kubectl apply -f ./k8s/ca-issuer.yaml

# create certificates for strimzi
kubectl create namespace kafka
kubectl apply -f ./k8s/broker-listner-cert.yaml
kubectl apply -f ./k8s/strimzi-ca.yaml

```

### start strimzi(kafka)

ref: [Quickstarts](https://strimzi.io/quickstarts/)

```shell
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka

kubectl apply -f ./k8s/kafka-my-cluster.yaml

kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka 
```

### deploy the kafka client app

```shell
# deploy mTLS resources
kubectl apply -f ./k8s/istio-resources-for-app.yaml

# deploy kafka client
kubectl apply -f ./k8s/kafka-cli-deployment.yaml
```

After the deployment, log in the cli:
```shell
kubectl exec --stdin --tty -n istio-kafka-client-sample pods/kafka-cli-<ID> -- /bin/bash
```

Plain text mode works 
```shell
# produce
kafka-console-producer --bootstrap-server my-cluster-kafka-bootstrap.kafka.svc:9092 --topic first_topic

# consume
kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap.kafka.svc:9092 --topic first_topic --from-beginning

```

TLS mode does not work... Why...
```shell
# tls
kafka-console-producer --bootstrap-server my-cluster-kafka-bootstrap.kafka.svc:9093 --topic first_topic

>[2023-02-23 09:27:49,234] WARN [Producer clientId=console-producer] Bootstrap broker my-cluster-kafka-bootstrap.kafka.svc:9093 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2023-02-23 09:27:49,366] WARN [Producer clientId=console-producer] Bootstrap broker my-cluster-kafka-bootstrap.kafka.svc:9093 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
[2023-02-23 09:27:49,527] WARN [Producer clientId=console-producer] Bootstrap broker my-cluster-kafka-bootstrap.kafka.svc:9093 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
```

```shell
$ kubectl logs -n kafka my-cluster-kafka-0
...
2023-02-23 12:13:34,442 INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /10.244.0.23 (channelId=10.244.0.17:9093-10.244.0.23:38710-39) (SSL handshake failed) (org.apache.kafka.common.network.Selector) [data-plane-kafka-network-thread-0-ListenerName(TLS-9093)-SSL-7]
2023-02-23 12:13:35,406 INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /10.244.0.23 (channelId=10.244.0.17:9093-10.244.0.23:38722-39) (SSL handshake failed) (org.apache.kafka.common.network.Selector) [data-plane-kafka-network-thread-0-ListenerName(TLS-9093)-SSL-8]
...
```
