---
image: /assets/img/AWS.png
title: Troubleshooting de ambiente Kubernetes com Kiro
description: Problemas em clusters Kubernetes raramente acontecem de forma
  isolada. Um pod em CrashLoopBackOff pode estar relacionado a falhas de
  configuração, limites de recursos, problemas de rede ou dependências externas.
  Em ambientes complexos, identificar a causa raiz exige correlação rápida entre
  múltiplas fontes de informação. Com o Kiro, é possível acelerar o diagnóstico
  utilizando context keys e integração via MCP para analisar logs, eventos e
  configurações de forma estruturada.
date: 2026-03-09
category: ai
background: "#FF6B6B"
tags:
  - KUBERNETES
  - TROUBLESHOOTINGKUBERNETES
  - KIRO
  - KUBERNETESDEBUGGING
  - DEVOPS
  - PLATFORMENGINEERING
  - CLOUDNATIVE
  - INCIDENTRESPONSE
  - KUBERNETESOPS
  - CONTAINEROPERATIONS
  - CLUSTERDEBUGGING
  - SRE
  - AUTOMACAODEDIAGNOSTICO
categories:
  - KUBERNETES
  - TROUBLESHOOTINGKUBERNETES
  - KIRO
  - KUBERNETESDEBUGGING
  - DEVOPS
  - PLATFORMENGINEERING
  - CLOUDNATIVE
  - INCIDENTRESPONSE
  - KUBERNETESOPS
  - CONTAINEROPERATIONS
  - CLUSTERDEBUGGING
  - SRE
  - AUTOMACAODEDIAGNOSTICO
---
Troubleshooting em Kubernetes exige visibilidade e contexto. Logs isolados raramente contam a história completa, e a análise manual de eventos, manifests e métricas pode consumir tempo precioso durante um incidente. 

Utilizando o Kiro em conjunto com clusters do Kubernetes, é possível estruturar a investigação com base em contexto, correlacionar sinais técnicos e reduzir drasticamente o tempo para identificar a causa raiz.

### Cenário

Eram 2h da manhã de uma sexta-feira quando o telefone do plantão tocou, novamente.

"Pods em CrashLoopBackOff no cluster de produção."

Antes, isso significava: abrir laptop, conectar VPN, rodar 15 comandos kubectl diferentes, analisar logs confusos, procurar no Stack Overflow, testar soluções, falhar, tentar de novo...

2 horas depois (se tivesse sorte), problema resolvido.

Hoje? De 10 a 20 minutos. E vou te mostrar exatamente como.

## Mas o que é Kiro CLI + EKS MCP?

Kiro CLI é uma ferramenta de linha de comando com IA que entende contexto completo do seu projeto. Quando você adiciona o EKS MCP (Model Context Protocol), ele ganha superpoderes:

- Interage diretamente com seu cluster Kubernetes
- Consulta logs via MCP
- Analisa métricas em tempo real
- Busca documentação AWS/Kubernetes automaticamente
- Sugere soluções baseadas em best practices atuais

É como ter um SRE disponível 24/7, que nunca esquece um comando kubectl.

## Então, por que você deveria se importar?

Se você trabalha com Kubernetes, conhece esses problemas:

**O Problema do Troubleshooting Manual:**
- 15+ comandos kubectl para diagnosticar um problema
- Logs espalhados em múltiplos pods
- Documentação desatualizada
- Context switching entre terminal, browser, Teams
- Pressão para resolver RÁPIDO (é produção!)

**O Custo Real:**
- MTTR (Mean Time To Repair): 1-3 horas em média
- Downtime: R$5.000-R$50.000 por hora (dependendo do negócio)
- Burnout: acordar às 3h da manhã todo plantão não é sustentável
- Conhecimento: depende de quem está on-call

Kiro CLI + EKS MCP resolve tudo isso. Automaticamente.

## Como você utiliza isso?

### Instalação

Primeiro, você precisa do EKS MCP server. Se tem Python/uv instalado:

```bash
$ uv tool install awslabs.eks-mcp-server
```

Ou com uvx (sem instalar):

```bash
$ uvx awslabs.eks-mcp-server@latest
```

Verifique a instalação:

```bash
$ uvx awslabs.eks-mcp-server --version
eks-mcp-server 0.2.1
```

### Configuração

Aqui está minha config do agent k8s (em `~/.kiro/agents/k8s.json`):

```json
{
  "name": "k8s",
  "description": "Kubernetes and EKS troubleshooting specialist",
  "prompt": "You are an expert in Kubernetes troubleshooting, especially on AWS EKS. You help diagnose and resolve issues quickly using kubectl, AWS CLI, and best practices. Always explain your reasoning and provide actionable solutions.",
  "mcpServers": {
    "eks": {
      "command": "uvx",
      "args": ["awslabs.eks-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      }
    },
    "aws-docs": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      }
    }
  },
  "tools": ["*"],
  "resources": []
}
```

**Por que dois MCPs?**
- `eks`: Interage com cluster (logs, métricas, eventos)
- `aws-docs`: Consulta documentação AWS/EKS em tempo real

Quando inicio o agent:

```bash
$ kiro-cli --agent k8s

✓ eks loaded in 1.2 s
✓ aws-docs loaded in 0.8 s

     ⢀⣴⣶⣶⣦⡀⠀⠀⠀⢀⣴⣶⣦⣄⡀⠀⠀⢀⣴⣶⣶⣦⡀
    ⢰⣿⠋⠁⠈⠙⣿⡆⠀⢀⣾⡿⠁⠀⠈⢻⡆⢰⣿⠋⠁⠈⠙⣿⡆
    ⢸⣿⠀⠀⠀⠀⣿⣇⣴⡿⠋⠀⠀⠀⢀⣼⠇⢸⣿⠀⠀⠀⠀⣿⡇
    
Model: claude-sonnet-4 (/model to change)
```

Pronto! Agora o Kiro CLI tem acesso direto ao meu cluster EKS.

## Kiro CLI + EKS MCP em ação

Vou mostrar 3 cenários reais que enfrentei. Com outputs completos do Kiro CLI.

### Cenário 1: O Clássico CrashLoopBackOff

**O Problema:**

```bash
$ kubectl get pods -n production

NAME                        READY   STATUS             RESTARTS   AGE
api-deployment-7d9f8c-xyz   0/1     CrashLoopBackOff   5          3m
```

Antes do Kiro CLI, eu faria:

```bash
$ kubectl logs api-deployment-7d9f8c-xyz -n production
$ kubectl describe pod api-deployment-7d9f8c-xyz -n production
$ kubectl get events -n production
$ kubectl top pod api-deployment-7d9f8c-xyz -n production
# ... mais 10 comandos
```

**Com Kiro CLI:**

```
[você] > O Pod api-deployment-7d9f8c-xyz na namespace production está em CrashLoopBackOff, analise, verifique a causa raiz e resolva.
#File k8s/deployment.yaml
```

**Output do Kiro CLI:**

```
Analyzing pod status...

Reading pod logs via EKS MCP...
✓ Retrieved 150 log lines

Reading pod description via EKS MCP...
✓ Retrieved pod events and status

Analyzing deployment manifest...
✓ Read k8s/deployment.yaml

🔍 Root Cause Identified:

Pod is being OOMKilled (Out of Memory)

Evidence:
1. Last State: Terminated
   Reason: OOMKilled
   Exit Code: 137

2. Current memory configuration:
   - Memory Limit: 512Mi
   - Memory Request: 256Mi

3. Logs show Java heap space error:
   java.lang.OutOfMemoryError: Java heap space
   at com.example.api.service.DataProcessor.process

📊 Analysis:

Your Java application is consuming more than 512Mi of memory.
Common causes:
- Memory leak in DataProcessor
- Increased workload/traffic
- JVM heap not configured (defaults to 1/4 of container memory)

🔧 Immediate Solutions:

Option 1: Increase memory limits (Quick fix)
Option 2: Configure JVM heap properly
Option 3: Investigate memory leak (Long-term)

I'll apply Option 1 + 2 now. Here's the updated deployment:
```

Kiro CLI então gera o deployment corrigido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  namespace: production
spec:
  template:
    spec:
      containers:
      - name: api
        image: myapp:v1.2.3
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"      # ← Aumentado
            cpu: "500m"
        env:
        - name: JAVA_OPTS      # ← Adicionado
          value: "-Xmx768m -Xms512m"
```

E continua:

```
Commands to apply:

# Update deployment
kubectl apply -f k8s/deployment.yaml

# Watch rollout
kubectl rollout status deployment/api-deployment -n production

# Verify pods
kubectl get pods -n production -l app=api

📝 I've also created a monitoring alert for you:

apiVersion: v1
kind: ConfigMap
metadata:
  name: memory-alerts
data:
  alert.yaml: |
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes{pod=~"api-.*"} > 800000000
      for: 5m
      annotations:
        summary: "Pod {{ $labels.pod }} using > 800Mi"

🎯 Next Steps:

1. Apply the fix (deployment updated)
2. Monitor for 24h
3. Investigate DataProcessor for memory leak
4. Consider implementing HPA for auto-scaling

Would you like me to:
- Create a runbook for this incident?
- Set up HPA configuration?
- Analyze DataProcessor code for leaks?
```

**Tempo total: 12 minutos** (vs 1-2 horas antes)

### Cenário 2: ImagePullBackOff

**O Problema:**

```bash
$ kubectl get pods -n staging

NAME                      READY   STATUS             RESTARTS
frontend-abc123-xyz       0/1     ImagePullBackOff   0
frontend-abc123-abc       0/1     ImagePullBackOff   0
frontend-abc123-def       0/1     ImagePullBackOff   0
# ... mais 10 pods com o mesmo problema
```

**Com Kiro CLI:**

```
[você] > Múltiplos pods não estão subindo. 
Status: ImagePullBackOff

#Terminal (kubectl describe pod frontend-abc123-xyz -n staging)
#Terminal (kubectl get events -n staging --sort-by='.lastTimestamp')
```

**Output do Kiro CLI:**

```
Analyzing ImagePullBackOff error...

Reading pod events via EKS MCP...
✓ Retrieved events from multiple pods

🔍 Root Cause Identified:

ECR Rate Limiting (QPS Exceeded)
Error: "toomanyrequests: Rate exceeded"

Image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/frontend:v2.1.0

📊 Analysis:

This is an ECR API rate limiting issue, not authentication!

Current situation:
- 15 pods trying to pull the same image simultaneously
- ECR default limits:
  * GetAuthorizationToken: 500 requests/second
  * BatchGetImage: 200 requests/second
  * GetDownloadUrlForLayer: 200 requests/second

What's happening:
1. Deployment scaled from 3 to 15 replicas
2. All 12 new pods trying to pull image at once
3. ECR throttling requests → ImagePullBackOff
4. Pods retry → more throttling → cascading failure

🔧 Solutions:

**Immediate (Stop the bleeding):**
1. Use image pull secrets with caching
2. Implement PodDisruptionBudget to control rollout speed
3. Configure imagePullPolicy: IfNotPresent

**Short-term (Reduce impact):**
1. Pre-pull images to nodes
2. Use ECR pull through cache
3. Implement progressive rollout

**Long-term (Prevent recurrence):**
1. Request ECR quota increase
2. Use multiple ECR repositories (sharding)
3. Implement image caching layer

Let me implement the immediate fixes:
```

Kiro CLI então gera as correções:

```yaml
# 1. Update Deployment with better image pull policy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: staging
spec:
  replicas: 15
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2          # ← Limit concurrent new pods
      maxUnavailable: 1    # ← Gradual rollout
  template:
    spec:
      containers:
      - name: frontend
        image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/frontend:v2.1.0
        imagePullPolicy: IfNotPresent  # ← Use cached image if available
        
---
# 2. Add PodDisruptionBudget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
  namespace: staging
spec:
  minAvailable: 10
  selector:
    matchLabels:
      app: frontend
```

E continua:

```
Commands to apply:

# 1. Update deployment with better rollout strategy
kubectl apply -f k8s/deployment.yaml

# 2. Apply PodDisruptionBudget
kubectl apply -f k8s/pdb.yaml

# 3. Watch rollout (should be gradual now)
kubectl rollout status deployment/frontend -n staging

# 4. Verify pods are coming up
kubectl get pods -n staging -w

# 5. Check ECR API calls (optional)
aws cloudwatch get-metric-statistics \
  --namespace AWS/ECR \
  --metric-name RepositoryPullCount \
  --dimensions Name=RepositoryName,Value=frontend \
  --start-time $(date -u -d '10 minutes ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 60 \
  --statistics Sum

📊 Monitoring:

I've also created a CloudWatch alarm for you:

aws cloudwatch put-metric-alarm \
  --alarm-name ecr-rate-limit-frontend \
  --alarm-description "Alert when ECR pull rate is high" \
  --metric-name RepositoryPullCount \
  --namespace AWS/ECR \
  --statistic Sum \
  --period 60 \
  --threshold 150 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

🎯 Next Steps:

1. Apply the fixes (deployment updated with gradual rollout)
2. Monitor ECR metrics for 24h
3. Consider requesting ECR quota increase:
   - Current: 200 req/s for BatchGetImage
   - Recommended: 500 req/s for your scale
4. Implement image caching layer (Harbor, Artifactory)

💡 Pro Tips:

- Always use `imagePullPolicy: IfNotPresent` for stable tags
- Implement progressive rollouts (maxSurge: 2-3)
- Pre-pull critical images with DaemonSet
- Monitor ECR metrics in CloudWatch
- Consider ECR pull through cache for frequently pulled images

Would you like me to:
- Create a runbook for ECR rate limiting incidents?
- Set up automated image pre-pulling?
- Generate Terraform for ECR quota increase request?
```

**Tempo: 20 minutos** (vs 1-3 horas investigando e procurando docs)

**Lições aprendidas:**

1. **Rate limiting é diferente de auth error** - Mesma mensagem de erro, causa diferente
2. **Scale events são gatilhos comuns** - 3 → 15 replicas = 12 pulls simultâneos
3. **ECR tem limites por operação** - GetAuthorizationToken, BatchGetImage, GetDownloadUrlForLayer
4. **Solução não é só aumentar quota** - Otimizar pull strategy é mais efetivo
5. **Gradual rollout é essencial** - maxSurge e maxUnavailable controlam a velocidade

### Cenário 3: Service Não Responde

**O Problema:**

```bash
$ curl http://api.staging.svc.cluster.local
curl: (7) Failed to connect to api.staging.svc.cluster.local port 80: Connection refused
```

**Com Kiro CLI:**

```
[você] > Service api não está acessível. Connection refused.

#File k8s/service.yaml
#File k8s/deployment.yaml
#Terminal (kubectl get svc api -n staging)
#Terminal (kubectl get endpoints api -n staging)
```

**Output do Kiro CLI:**

```
Analyzing service connectivity...

Reading service configuration...
✓ Read k8s/service.yaml

Reading deployment configuration...
✓ Read k8s/deployment.yaml

Checking service status via EKS MCP...
✓ Retrieved service details

Checking endpoints via EKS MCP...
✓ Retrieved endpoints

🔍 Root Cause Identified:

Service has NO endpoints!

Evidence:
```

```yaml
# Service
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: staging
spec:
  selector:
    app: api          # ← Looking for this label
  ports:
  - port: 80
    targetPort: 8080

# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: staging
spec:
  template:
    metadata:
      labels:
        app: api-service  # ← MISMATCH! Has different label
```

```
📊 Analysis:

Label mismatch between Service selector and Deployment labels!

Service is looking for: app=api
Deployment has: app=api-service

This is why endpoints are empty - Service can't find any pods.

🔧 Solution:

Fix the label mismatch. I recommend updating the Deployment to match
the Service (less disruptive than changing Service).

Updated deployment.yaml:
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: staging
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api        # ← Fixed
  template:
    metadata:
      labels:
        app: api      # ← Fixed
    spec:
      containers:
      - name: api
        image: myapp:v1.0.0
        ports:
        - containerPort: 8080
```

```
Commands to apply:

# Apply fixed deployment
kubectl apply -f k8s/deployment.yaml

# Verify endpoints are created
kubectl get endpoints api -n staging

# Should now show:
# NAME   ENDPOINTS                                   AGE
# api    10.0.1.5:8080,10.0.1.6:8080,10.0.1.7:8080  1m

# Test connectivity
kubectl run test --rm -it --image=busybox -- \
  wget -O- http://api.staging.svc.cluster.local

✅ Service should be accessible now!

💡 Pro tip: Use consistent naming conventions to avoid this.
I can help you create a naming standard if you'd like.
```

**Tempo: 5 minutos** (vs 1 hora debugando)

## Erros de gravação (Porque Nem Tudo é Perfeito)

### Problema 1: MCP Não Conectou ao Cluster

Primeira vez que tentei:

```bash
$ kiro-cli --agent k8s

⚠ eks failed to load
Error: No kubeconfig found
```

Esqueci de configurar o kubeconfig!

**Solução:**

```bash
$ aws eks update-kubeconfig --name my-cluster --region us-east-1

Added new context arn:aws:eks:us-east-1:123456789012:cluster/my-cluster

$ kiro-cli --agent k8s
✓ eks loaded in 1.2 s
```

### Problema 2: Permissões IAM

Tentei usar o EKS MCP e:

```
The SSO session associated with this profile has expired or is otherwise invalid. To refresh this SSO session run aws sso login with the corresponding profile.
```

Minha sessão do SSO precisa ter um token válido para avançar.

**Solução:** Fiz login novamente na AWS utilizando o `aws sso login`


### Problema 3: Timeout em Cluster Grande

Em um cluster com 200+ pods:

```
[você] > Liste todos os pods com problemas

⏳ Analyzing... (this is taking longer than usual)
```

Demorou 30 segundos (ainda rápido, mas inesperado).

**Aprendi:** Para clusters grandes, seja mais específico:

```
[você] > Liste pods com problemas no namespace production

✓ Completed in 3s
```

## Casos de uso avançados

### 1. Análise de Performance

```
[você] > O cluster eks está lento e os microserviços não estão escalando da forma que deveria. Analise performance.

#Terminal (kubectl top nodes)
#Terminal (kubectl top pods --all-namespaces)
```

Kiro CLI analisa e sugere:
- Scale horizontal (HPA)
- Scale vertical (VPA)
- Karpenter para auto-scaling
- Com estimativa de custos!

### 2. Security Audit

```
[você] > Alguém deletou um deployment da minha namespace production. Faça auditoria analisando os logs do cluster no Cloudwatch para identificar o horário e quem causou a remoção.
```

Kiro CLI identifica:
- Events do kubernets para identificar a ultima vez que o deployment gerou evento.
- Analisa os logs do CloudWatch em busca de evidências.
- Sugere restrições no modelo de acesso do usuário responsável pela remoção.
- Caso tenha acesso aos deployments originais, corrige automaticamente fazendo um novo deploy ( com sua aprovação )


### 3. Cost Optimization

```
[você] > Analise meus pods, deployments, daemonsets em busca de otimizações voltada a custos.

#Terminal (kubectl get pods --all-namespaces -o json)
```

Kiro CLI encontra:
- Pods com CPU/Memory request muito alto
- Nodes underutilized
- Oportunidades para Spot instances

## Tips & Tricks

### 1. Use Context Keys Sempre

```
❌ Ruim:
"Analise o pod api-xyz"

✅ Bom:
"Analise o pod api-xyz
#Terminal (kubectl logs api-xyz)
#Terminal (kubectl describe pod api-xyz)
#File deployment.yaml"
```

Quanto mais contexto, melhor a análise.

### 2. Crie Aliases para Comandos Comuns

No meu `.zshrc`:

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kd='kubectl describe'
alias kl='kubectl logs'

# Kiro com contexto
alias kiro-debug='kiro-cli --agent k8s'
```

### 3. Configure Hooks para Validação

Em `~/.kiro/agents/k8s.json`:

```json
{
  "hooks": {
    "pre-apply": {
      "command": "kubectl diff -f"
    },
    "post-apply": {
      "command": "kubectl get pods -w"
    }
  }
}
```

### 4. Mantenha Runbooks Atualizados

Sempre que Kiro CLI resolve um problema:

```
[você] > Documente este incident em um runbook
```

Kiro CLI cria/atualiza automaticamente.


## Conclusão

Troubleshooting de Kubernetes não precisa ser doloroso.

Com Kiro CLI + EKS MCP:
- MTTR de horas para minutos
- Menos stress, mais sono
- Documentação sempre atualizada
- Time mais produtivo e feliz

E o melhor: você pode começar hoje. Literalmente.


*PS: Se você acordou às 3h da manhã hoje por causa de um CrashLoopBackOff, meus sentimentos. Mas agora você sabe como evitar isso na próxima vez.*
