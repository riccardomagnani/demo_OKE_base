## HOL1: Istallazione dell'applicazione via *helm*

Prerequisiti

I seguenti tool devono essere installato sulla macchina client:

- `helm`
- `git`

> NB: OCI Cloud Shell ha già installato questo software.

<br/>

### Clone del repository

Si effettua il clone del repository del workshop:

**AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA**



### Istallazione dell'applicazione microservices sample

Per evitare i warning di sicurezza di helm si cambia i diritti di accesso del file config di kubernetes:

```bash
chmod 400 ~/.kube/config
```

<br/>

Volendo si può aprire una shell e controllare il comportamento del cluster durante la fase di installazione dell'app:

```bash
watch -n 1 kubectl get po -n core-services -o wide
```

<br/>

Si usa `helm` per istallare l'applicazione "microservices sample":

```bash
cd ~/demo_OKE_base/handson_1/

helm install sample-app k8s_sample-microservices-app/ --values k8s_sample-microservices-app/values.yaml --namespace sample-app --create-namespace
```

L'applicazione installa anche un servizio di tipo LoadBalancer accessibile da Internet visualizzabile con questo comando:

```bash
kubectl get svc -n main-app
```

Si può vedere anche dalla OCI Console la creazione del LoadBalancer collegato al servizio Kubernetes:

![image-20210522114356504](image/image-20210522114356504.png)

<br/>

Quando l'installazione della applicazione sarà completata (pochi secondi dopo il lancio del comando `helm`) si può prendere IP del servizio con il seguente comando:

```bash
export SAMPLE_APP_SERVICE_IP=$(kubectl get svc --namespace main-app frontend-ui-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo $SAMPLE_APP_SERVICE_IP
```

<br/>

L'applicazione può essere testata con il seguente comando o via Browser:

```bash
export URL_APP="http://$SAMPLE_APP_SERVICE_IP/LoadOKE/TestOKEService?servlist=email-service.core-services.svc.cluster.local:8080,pdf-generation-service.core-services.svc.cluster.local:8080,digitalsignchecker-service.core-services.svc.cluster.local:8080&threadnum=5,5,5&elabtime=100,100,100&errperc=10,5,10"

echo $URL_APP

curl $URL_APP
```

