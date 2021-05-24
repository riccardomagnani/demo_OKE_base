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





## HOL3: Rolling update / Rollback & protezione dagli errori (probes)

### Rolling update / rollback

#### Prima istallazione

Si installa una applicazione`Generic-Microservice` composta da:

1. un deployment (che referenzia una immagine in versione 1)
2. il relativo service di tipo: LoadBalancer

Si usano questi comandi:

```bash
cd ~/demo_OKE_base/handson_3

kubectl get po,svc

kubectl apply -f generic-microservice-nochecks.yaml

kubectl get po,svc
```

Si testa il generic-microservice deployment mandandogli un carico costante:

```bash
export GENERIC_MICRO_SERVICE_IP=$(kubectl get svc --namespace default generic-microservice -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo $GENERIC_MICRO_SERVICE_IP

while true; do curl http://$GENERIC_MICRO_SERVICE_IP; sleep 0.5;done
```

Si mantiene il loop che mostra la versione dell'immagine in esecuzione nel pod. Si può verificare che il servizio bilancia sui vari pod perché nella risposta viene mostrato il hostname del pod che risponde alla chiamata.

<br/>

#### Scaling manuale

Teniamo monitorato il compontamento del cluster con il comando seguente mentre si fanno altre operazioni:

```bash
watch -n 1 kubectl get po,svc
```

Usando un'altra shell si può ad esempio scalare manualmente il numero dei pod sotto il deployment.

```bash
kubectl scale deployment --replicas=10 generic-microservice
```

#### Aggiornamento dell'applicazione

Adesso procediamo con l'aggiornamento della versione applicativa cambiando l'immagine dentro il deployment e usando rolling update strategy:

```bash
kubectl set image deploy generic-microservice app=rmagnani/kubeserve:v2 --record
```

Si verifica lo status del deployment con questo comando:

```bash
kubectl rollout history deployment generic-microservice
```

Il loop mostra che il rilascio applicativo non ha dato alcun disservizio.

<br/>

Adesso facciamo un aggiornamento ad una versione **errata**.

```bash
kubectl set image deploy generic-microservice app=rmagnani/kubeserve:error --record
```

Il loop mostra il problema applicativo e l'utente finale sta avendo disservizio.

Si fa il roll-back applicativo alla versione precedente al rilascio:

```bash
kubectl rollout undo deployment generic-microservice
```

<br/>

### Exploit probing

Mantenere la shell con il loop attivo.

Si usa una versione di deployment con delle probe`generic-microservice-checks.yaml`:

...

```yaml
...
    readinessProbe:
      httpGet:
        path: /
        port: 80
        scheme: HTTP
      initialDelaySeconds: 1
    livenessProbe:
      httpGet:
        path: /
        port: 80
        scheme: HTTP
...
```

Si applica:

```bash
cd ~/demo_OKE_base/handson_3

kubectl apply -f generic-microservice-checks.yaml
```

Sono state aggiunte le sond di Readiness e Liveness.

<br/>

Se adesso si imposta di nuovo l'immagine con l'errore si vede che i nuovi pod non diventano mai live perché non superano il probing readiness perché non tornano 200.

Si apre un shell e si usa questo comando per monitorare il rilascio:

```bash
watch -n 1 kubectl get po,svc
```

Poi si fa il rilascio:

```bash
kubectl set image deploy generic-microservice app=rmagnani/kubeserve:error --record
```

Si verifica che la versione rimane sempre la v2 (come si può vedere dalla shell con il `curl` loop).

Il describe dei pod in "not ready" da maggiori informazioni:

```bash
[centos@centtos7 handson_3]$ kubectl get po
NAME                                    READY   STATUS             RESTARTS   AGE
generic-microservice-5868c89b7f-grhsl   0/1     Running            5          6m
generic-microservice-5868c89b7f-txfsv   0/1     Running            5          6m
generic-microservice-5868c89b7f-vl66l   0/1     CrashLoopBackOff   5          6m
generic-microservice-5868c89b7f-vlr8l   0/1     CrashLoopBackOff   5          6m
generic-microservice-756444848f-66nz5   1/1     Running            0          9m23s
...
generic-microservice-756444848f-wrh5q   1/1     Running            0          9m18s

[centos@centtos7 handson_3]$ kubectl describe po generic-microservice-5868c89b7f-vl66l
Name:         generic-microservice-5868c89b7f-vl66l
Namespace:    default
Priority:     0
Node:         10.0.10.104/10.0.10.104
...
Events:
  Type     Reason     Age                     From               Message

----     ------     ----                    ----               -------

  Normal   Scheduled  6m23s                   default-scheduler  Successfully assigned default/generic-microservice-5868c89b7f-vl66l to 10.0.10.104
  Normal   Pulled     5m33s (x2 over 6m23s)   kubelet            Container image "rmagnani/kubeserve:error" already present on machine
  Normal   Created    5m33s (x2 over 6m23s)   kubelet            Created container app
  Normal   Started    5m32s (x2 over 6m23s)   kubelet            Started container app
  Normal   Killing    5m3s (x2 over 6m3s)     kubelet            Container app failed liveness probe, will be restarted
  Warning  Unhealthy  4m39s (x11 over 6m19s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 500
  Warning  Unhealthy  83s (x16 over 6m23s)    kubelet            Liveness probe failed: HTTP probe failed with statuscode: 500
```

<br/>

[HOL3: Rolling update / Rollback & protezione dagli errori (probes)](HOL3.md) 

