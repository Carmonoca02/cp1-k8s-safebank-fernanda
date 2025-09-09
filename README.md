## CP 1 k8s | Fernanda Carmona | RM:557064

### Objetivo
Simular a publicação de uma aplicação em Kubernetes ( Minikube ou K8s em EC2 ou EKS) em um ambiente de cloud real (AWS Academy), criando objetos básicos (Pods, Services, Deployments) e expondo a aplicação para acesso público.

### Requisitos
- Acessar a aplicação no navegador
- Criar os manifestos para deploy da aplicação
  - deployment.yaml
  - service.yaml
  - pod.yaml
- Explicação do porque escolheu aquele service type específico

### Ferramentas
- minikube/kind
- docker

### Passo a passo

É necessário realizar a instalação do gerenciador de cluster kuberentes local. Para isso, utilizar o gerenciador de pacotes do windows chamado `chocolatey`.

> Caso não tenho o chocolatey instalado, instale utilizando o script de instalação ou acesse a *[documentação](https://docs.chocolatey.org/en-us/choco/setup/)*.
```powersehll
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

#### Instalando o kind
```powershell
choco install kind
```

> É necessário que os comandos sejam executados no `cmd/powershell` como administrador

#### Criação do cluster

```powershell
kind create cluster --name $clustername
```

**Validação de criação**

```powershell
kind get clusters
```
**Evidência da criação**

![image](.\assests\create-cluster.png)

![image](.\assests\docker-cluster.png)

#### Acesso ao cluster

Quando realiza a instalação via choco, é feita a instalação do `Docker Desktop` e do `kubectl`, é necessário instalar, pois por baixo dos panos o cluster rodar dentro de containers Docker.
> É possível verificar os 2 "nodes"/containers (control plane e worker) rodando acessando o Docker desktop

```powershell
kubectl cluster-info --context kind-cluster-safebank
```

#### Criação de manifestos

pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    env: fiap
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: safebank-app
  namespace: safebank
spec:
  selector:
    app: safebank-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

> Utilizei NodePort pelos seguintes motivos:
>- Acesso externo no ambiente local → Permite acessar os serviços do cluster kind a partir da minha máquina host.
>- Simplicidade → É a forma mais rápida e nativa de expor a aplicação sem depender de balanceadores de carga ou ingress.
>- Controle explícito da porta → É possível definir uma porta fixa para testes (no caso: http://localhost:80).
>- Compatibilidade com o kind → O kind não cria LoadBalancers, logo o NodePort é o mecanismo adequado para expor o serviço.

deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: safebank-app
  namespace: safebank
  labels:
    app: safebank-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: safebank-app
  template:
    metadata:
      labels:
        app: safebank-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

> Não esquecer de alterar os parâmetros de nomenclatura: namespace, image, containetPort, appname 

#### Criação de Namespace

Namespace é um separador lógico para segregar as aplicações

```powershell
kubectl create ns safebank
```

**Validação de criação**

```powershell
kubectl get ns
```

**Evidência da criação**

![image](.\assests\create-namespace.png)


#### Deploy de manifestos

pod.yaml

```powershell
kubectl apply -f pod.yaml -n safebank
```

service.yaml

```powershell
kubectl apply -f service.yaml -n safebank
```

deployment.yaml

```powershell
kubectl apply -f deployment.yaml -n safebank
```

**Validação de criação**

pod

```powershell
kubectl get pods -n safebank
```

service

```powershell
kubectl get service -n safebank
```

deployment

```powershell
kubectl get deployment -n safebank
```

**Evidência da criação**

pod

![image](.\assests\create-pod.png)

service

![image](.\assests\create-service.png)

deployment

![image](.\assests\create-deployment.png)

#### Expor a aplicação  HTTP

> Estou utilizando o kubectl port-forward porque estou em um ambiente de testes local (kind) que não possui balanceador de carga ou ingress configurado. O port-forward permite criar um túnel temporário entre minha máquina e o pod, possibilitando o acesso direto à aplicação de forma rápida, segura e sem necessidade de expor portas do cluster para fora.

```powershell
kubectl port-forward svc/safebank-app 80:80 -n safebank
```
**Validação de exposição**

Acessar http://localhost:80 no navegador

**Evidência da exposição**

![image](.\assests\expondo-aplicacao.png)

![image](.\assests\aplicacao.png)


#### Teste de Scale

Teste de Scale de 2 para 5 pods

```powershell
kubectl scale deployment safebank-app --replicas=5 -n safebank
```

**Validação de Scale**

```powershell
kubectl get pods -n safebank
```
**Evidência do scale**

Antes (2 pods)

![image](.\assests\scale-2.png)

Depois (5 pods)

![image](.\assests\scale-5.png)



