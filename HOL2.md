# HOL2: Stress test e Kubeview

## Prerequisiti

Sul cluster deve essere installato:

- `siege` (è una applicazione linux)

- metric server

- kubeview

Maggiori info di seguito:

### metric server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.4/components.yaml
```

Per verificare:

```bash
k8s_sample-microservices-app]$ kubectl rollout status deploy metrics-server -n kube-system
deployment "metrics-server" successfully rolled out
```

Si può controllare al presenza del metric-server:

```bash
kubectl get po -n kube-system -l k8s-app=metrics-server
```

Dopo l'installazione del metric server si puo controllare l'impegno di risorse dei nodi e dei pod:

```bash
kubectl top no
kubectl top po -A
```

### Kubeview

```bash
git clone https://github.com/riccardomagnani/kubeview

helm install kubeview kubeview/deployments/helm/kubeview -f ./kubeview/deployments/helm/myvalues-sample.yaml --namespace kubeview --create-namespace
```

Verifica:

```bash
export SERVICE_IP=$(kubectl get svc --namespace kubeview kubeview -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo http://$SERVICE_IP
```

Si usa il browser per vedere Kubeview.

#### Cancellazione di Kubeview

Per cancellare Kubeview si usa il seguente comando helm:

```bash
helm uninstall kubeview --namespace kubeview
```

<br/>

-------------

## HOL2

Si apre una shell per monitorare il comportamento del cluster:

```bash
watch -n 1 "kubectl top no && kubectl get hpa,po -n core-services"

kubectl get po,svc -n main-app
watch -n 1 kubectl top po -n core-services
watch kubectl get hpa,po,svc -n core-services
```

Si esegue il seguente comando per iniettare del carico nell'applicazione sample.

```bash
export SERVICE_IP=$(kubectl get svc --namespace main-app frontend-ui-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo $SERVICE_IP

siege -c 10 -r 10000 "http://$SERVICE_IP/LoadOKE/TestOKEService?servlist=email-service.core-services.svc.cluster.local:8080,pdf-generation-service.core-services.svc.cluster.local:8080,digitalsignchecker-service.core-services.svc.cluster.local:8080&threadnum=5,5,5&elabtime=100,100,100&errperc=10,5,10"
```

Utilizzando *k9s* si possono vedere i dei pod.

Quando il deployment ha raggiunto le dimensioni massime il `siege` può essere fermato con Ctrl+C.

<br/>

## Cancellazione dell'applicazione via helm

Per cancellare l'applicazione si usa il seguente comando:

```bash
helm ls -n sample-app

helm uninstall sample-app -n sample-app
```

<br/>

---------------------

<br/>

[HOL3: Rolling update / Rollback & protezione dagli errori (probes)](HOL3.md) 

