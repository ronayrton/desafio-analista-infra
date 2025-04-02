# Etapa 2 - Tarefa: Execute a atividade da Etapa 1 em um cluster kubernetes

## Objetivo
Nesta etapa, a aplicação Jenkins foi implantada em um cluster Kubernetes local utilizando o Kind (Kubernetes in Docker).

## Passos para Implantação

### 1. Criando o Cluster Kubernetes com Kind
Para criar um cluster Kubernetes local utilizando Kind, foi utilizado o seguinte comando:
```bash
kind create cluster --name jenkins-cluster
```
Este comando inicializa um cluster local chamado `jenkins-cluster`.

### 2. Criando os Manifestos Kubernetes
Foram criados os seguintes arquivos YAML:

#### **Deployment do Jenkins**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  labels:
    app: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          ports:
            - containerPort: 8080
```

#### **Service para Expor o Jenkins**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
spec:
  selector:
    app: jenkins
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30000
  type: NodePort
```

### 3. Aplicando os Arquivos ao Cluster
Após criar os arquivos YAML, eles foram aplicados ao cluster com os seguintes comandos:
```bash
kubectl apply -f jenkins-deployment.yaml
kubectl apply -f jenkins-service.yaml
```

### 4. Verificando se o Pod foi Criado
Para garantir que o Jenkins estava rodando corretamente:
```bash
kubectl get pods
```
Saída esperada:
```
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-5589865fd4-gsztb   1/1     Running   0          2m
```

## Problemas Encontrados e Soluções

### 🚨 **Problema: Jenkins não acessível via `http://localhost:30000`**

#### **Causa:**
- O Kind roda os containers dentro de uma rede interna do Docker, então os serviços expostos via `NodePort` não são acessíveis diretamente do host Windows/WSL.

#### **Solução:**
A solução foi utilizar o port forwarding:
```bash
kubectl port-forward svc/jenkins-service 8080:8080
```
Isso permitiu acessar o Jenkins via:
```
http://localhost:8080
```

### 🚨 **Problema: Status do Pod "ContainerCreating" por muito tempo**

#### **Causa:**
- O Jenkins pode demorar para iniciar devido ao tempo de download da imagem.

#### **Solução:**
- Executar:
```bash
kubectl describe pod jenkins
```
- Se for erro de imagem, baixar manualmente:
```bash
docker pull jenkins/jenkins:lts
```
- Excluir o pod para que o Kubernetes recrie:
```bash
kubectl delete pod jenkins-5589865fd4-gsztb
```

### 🚨 **Problema: Jenkins pedindo senha inicial**
Após acessar o Jenkins pela primeira vez, ele pede uma senha inicial.

#### **Solução:**
Executar:
```bash
kubectl logs $(kubectl get pods -l app=jenkins -o jsonpath="{.items[0].metadata.name}") | grep -A 5 "Please use the following password"
```
Isso retorna a senha, que pode ser usada no primeiro login.

## Conclusão
- O Jenkins foi implantado com sucesso no Kubernetes usando Kind.
- O serviço foi acessado utilizando `kubectl port-forward`.
- Problemas de rede e acesso foram solucionados com debugging e ajustes na rede do Kind.

## Para excluir completamente seu cluster Kubernetes criado com o Kind, siga estes passos:

Liste os clusters ativos:

kind get clusters

Delete o cluster (substitua <nome-do-cluster> pelo nome correto se não for o padrão kind):

kind delete cluster --name kind

## Referências
- [Documentação do Kind](https://kind.sigs.k8s.io/)
- [Documentação do Kubernetes](https://kubernetes.io/docs/)
- [Documentação do Jenkins](https://www.jenkins.io/doc/)

