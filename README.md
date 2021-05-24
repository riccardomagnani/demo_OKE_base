# Demo OKE

## Indice

- [HOL1: Istallazione dell'applicazione via *helm*](HOL1.md) 

- [HOL2: Stress test e Kubeview](HOL2.md) 

- [HOL3: Rolling update / Rollback & protezione dagli errori (probes)](HOL3.md) 

<br/>

## Creazione di un cluster OKE con QuickStart

In questo esempio useremo il Quickstart per creare il cluster:

![image-20210521170013846](image/image-20210521170013846.png)

<br/>

Poi si impostano le caratteristiche del cluster e si imposta una chiave pubblica per accedere ai nodi:

![image-20210521170114819](image/image-20210521170114819.png)

<br/>

## Presentazione della struttura interna al cluster da console

Lista dei Node Pools:

![image-20210521182024481](image/image-20210521182024481.png)

Dettaglio del Node Pool:

![image-20210521182044983](image/image-20210521182044983.png)

<br/>

## Collegamento al cluster da OCI Cloud Shell e da external bash shell

Si preme il bottone Access Cluster disponibile all'interno del dettaglio del Cluster OKE:

### OCI Cloud Shell

Si può accedere alla cloud shell direttamente dal browser:

![image-20210522110436676](image/image-20210522110436676.png)

Si configura la cloud shell:

![image-20210522110622203](image/image-20210522110622203.png)

<br/>

### Extenal bash shell

Si può accedere alla console utilizzando una macchina Bastion (nel caso di cluster privato) o una macchina Linux via internet (nel caso di cluster pubblico).

#### Prerequisiti

Sulla macchine cliente deve essere installato:

- OCI CLI

- `kubectl`

> NB: OCI Cloud Shell ha già installato questo software.

<br/>

Si utilizza l'accesso local access:

![image-20210522110737252](image/image-20210522110737252.png)

Si configuro la macchina locale per accedere al cluster pubblico OKE:

![image-20210522111007053](image/image-20210522111007053.png)

<br/>

#### Scollegare la macchina locale dal cluster Kubernetes

Per scollegare la macchiana locale (o anche la cloud shell) dal cluster basta cancellare il file config che si crea quando si scarica la configurazione.

```bash
rm $HOME/.kube/config
```

<br/>

## k9s

### Prerequisiti

Il seguente software deve essere installato sulla macchina client

- `k9s`

<br/>

Si visualizza i componenti Kubernetes installati nel cluster con **k9s**:

```bash
~/k9s
```

Una volta dentro l'applicazione si usano i seguenti comandi per vedere i diversi componenti di k8s:

```bash
:ns

:pod

:deploy

:service

:sts

:pv

:pvc

...
```



## OCIR

### Prerequisiti

Il seguente software deve essere installato sulla macchina client

- Docker

> NB: OCI Cloud Shell ha già installato questo software.

<br/>

Si scarica una immagine pubblica, ad esempio:

```bash
docker pull rmagnani/kubeserve:v1
```

Si fa il login su OCIR:

```bash
docker login fra.ocir.io/emeaseitalysandbox -u 'emeaseitalysandbox/oracleidentitycloud/<email-address>'
```

Come password si usa il token generato da OCI nella sezione:

`Identity / Users / <utente> / Auth Tokens`

Si può "taggare" l'immagine appena scaricata:

```bash
docker tag rmagnani/kubeserve:v1 fra.ocir.io/emeaseitalysandbox/test-repository/kubeserve:v1
```

Poi si fa il push dell'immagine su OCIR nel repository con nome *test-repository*:

```bash
docker push fra.ocir.io/emeaseitalysandbox/test-repository/kubeserve:v1
```

Si controlla poi che l'immagine sia stata salvata nel repository (root compartment) ed eventualmente si sposta su un compartment diverso per motivi di sicurezza.

![image-20210522085850200](image/image-20210522085850200.png)

<br/>

[HOL1: Istallazione dell'applicazione via *helm*](HOL1.md) 

<br/>

# Repository base

https://bitbucket.org/riccardo_magnani/sample-microservices-app/src/master/