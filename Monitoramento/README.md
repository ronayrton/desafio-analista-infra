# Etapa 3 - Tarefa: Configurar o Prometheus para coletar métricas via Jolokia e do Node Exporter. (Kind + WSL)

## Objetivo

O objetivo desta etapa é configurar e implantar o **Prometheus** no cluster Kubernetes criado com Kind no **WSL (Windows Subsystem for Linux)**. O Prometheus será configurado para coletar métricas via **Jolokia** e **Node Exporter**.

## Passos

### 1. Criar o Cluster Kubernetes com Kind

```bash
kind create cluster --name monitoring-cluster
```

Para verificar se o cluster foi criado corretamente:

```bash
kubectl get nodes
```

Se o Kind estiver rodando corretamente, você verá uma lista de nós em execução.

---

### 2. Criar os Manifestos YAML para Implantar o Prometheus

Criar um arquivo `prometheus-config.yaml` com a configuração do Prometheus para coletar métricas:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'jolokia'
        static_configs:
          - targets: ['jolokia-endpoint:8778']
      - job_name: 'node-exporter'
        static_configs:
          - targets: ['node-exporter:9100']
```

Criar um `prometheus-deployment.yaml` para implantar o Prometheus:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus
        args:
          - "--config.file=/etc/prometheus/prometheus.yml"
        ports:
          - containerPort: 9090
        volumeMounts:
          - name: config-volume
            mountPath: /etc/prometheus/
      volumes:
      - name: config-volume
        configMap:
          name: prometheus-config
```

Criar um `prometheus-service.yaml` para expor o Prometheus:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
spec:
  type: NodePort
  selector:
    app: prometheus
  ports:
    - protocol: TCP
      port: 9090
      targetPort: 9090
      nodePort: 30090
```

---

### 3. Implantar o Prometheus no Cluster

Criar o namespace `monitoring`:

```bash
kubectl create namespace monitoring
```

Aplicar os arquivos:

```bash
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
```

Verificar se os pods estão rodando:

```bash
kubectl get pods -n monitoring
```

Verificar se o serviço está ativo:

```bash
kubectl get svc -n monitoring
```

---

### 4. Acessando o Prometheus

O Prometheus pode ser acessado de duas formas:

#### Opção 1: Usando `kubectl port-forward`

```bash
kubectl port-forward svc/prometheus-service 9090:9090 -n monitoring
```

Agora acesse no navegador:

👉 **http://localhost:9090**

#### Opção 2: Usando o IP do Cluster (WSL)

Obter o IP do cluster no Windows (caso necessário):

```bash
echo $(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
```

Agora, acesse o Prometheus pelo IP retornado:

👉 **http://<IP-DO-CLUSTER>:30090**

Se o acesso não funcionar, use o **port-forward**.

---

## Problemas Enfrentados e Soluções

1. **Erro `helm not found`**
   - **Solução**: Como não quer usar Helm, instalamos manualmente via manifestos YAML.

2. **Porta `30090` não acessível no Windows**
   - **Solução**: Utilizamos `kubectl port-forward` para redirecionar a porta.

3. **Erro ao criar CRDs com Helm**
   - **Solução**: Evitamos Helm e usamos arquivos YAML personalizados.

---

## Conclusão

Agora o Prometheus está implantado corretamente e pode coletar métricas do Jolokia e Node Exporter. Se precisar de ajustes ou adicionar um painel de visualização como o Grafana, é só avisar! 🚀

