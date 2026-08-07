---
image: /assets/img/AWS.png
title: Usando Kiro para Estruturar Operacoes de Infraestrutura
description: Guia pratico sobre como utilizar o Kiro para estruturar operacoes
  de infraestrutura com MCP Servers, Steering e Agents.
date: 2026-04-06
category: devops
background: "#05A6F0"
tags:
  - KIRO
  - OPERACOES
  - INFRAESTRUTURA
  - DEVOPS
  - AUTOMACAO
  - MCP
  - AGENTS
  - STEERING
  - AWS
  - TERRAFORM
categories:
  - KIRO
  - OPERACOES
  - INFRAESTRUTURA
  - DEVOPS
  - AUTOMACAO
  - MCP
  - AGENTS
  - STEERING
  - AWS
  - TERRAFORM
---
Times de infraestrutura lidam diariamente com incidentes, mudancas, provisionamento, troubleshooting e decisoes que precisam ser rapidas, mas tambem consistentes. O desafio nao e apenas executar tarefas, e sim manter coerencia operacional, padronizacao e rastreabilidade ao longo do tempo.

E nesse cenario que o Kiro se torna uma peca estrategica. Quando bem configurado, ele deixa de ser apenas uma ferramenta e passa a atuar como uma camada de inteligencia operacional, utilizando MCP Servers para interagir com AWS e Terraform, aplicando Steering para manter padroes e utilizando Agents para automatizar fluxos recorrentes.

## O que sao MCP Servers

MCP (Model Context Protocol) e um protocolo que permite ao Kiro se conectar com ferramentas externas de forma estruturada. Atraves de MCP Servers, o Kiro consegue executar comandos AWS, consultar documentacao, gerar diagramas, analisar custos e muito mais.

A configuracao dos MCP Servers fica em `.kiro/settings/mcp.json` no workspace ou em `~/.kiro/settings/mcp.json` para configuracao global.

## MCP Servers para Operacoes AWS

Vamos explorar os principais MCP Servers que transformam o Kiro em uma ferramenta poderosa para operacoes de infraestrutura.

### AWS API MCP Server

O `aws-api-mcp-server` permite executar comandos AWS CLI diretamente pelo Kiro. E o servidor mais fundamental para operacoes.

Configuracao:

```json
{
  "mcpServers": {
    "aws-api": {
      "command": "uvx",
      "args": ["awslabs.aws-api-mcp-server@latest"],
      "env": {
        "AWS_PROFILE": "default",
        "AWS_REGION": "us-east-1"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Com esse servidor, voce pode pedir ao Kiro coisas como:

- "Liste todas as instancias EC2 paradas ha mais de 30 dias"
- "Quais security groups nao estao sendo usados?"
- "Mostre os buckets S3 sem versionamento habilitado"

### AWS Documentation MCP Server

O `aws-documentation-mcp-server` da ao Kiro acesso a documentacao oficial da AWS. Isso e util quando voce precisa de informacoes atualizadas sobre servicos.

Configuracao:

```json
{
  "mcpServers": {
    "aws-docs": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Exemplos de uso:

- "Qual o limite de ENIs por tipo de instancia?"
- "Como funciona o lifecycle do S3 Intelligent-Tiering?"
- "Quais sao as quotas do Lambda?"

### AWS Diagram MCP Server

O `aws-diagram-mcp-server` gera diagramas de arquitetura usando a biblioteca Python Diagrams. Perfeito para documentacao e apresentacoes.

Configuracao:

```json
{
  "mcpServers": {
    "aws-diagram": {
      "command": "uvx",
      "args": ["awslabs.aws-diagram-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Voce pode pedir:

- "Gere um diagrama da arquitetura com ALB, ECS e RDS"
- "Crie um diagrama mostrando a VPC com subnets publicas e privadas"
- "Desenhe a arquitetura de um pipeline CI/CD com CodePipeline"

### Terraform MCP Server

O `terraform-mcp-server` permite interagir com codigo Terraform, executar comandos e consultar documentacao de providers.

Configuracao:

```json
{
  "mcpServers": {
    "terraform": {
      "command": "uvx",
      "args": ["awslabs.terraform-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Casos de uso:

- "Execute terraform plan nesse diretorio"
- "Qual a documentacao do resource aws_lambda_function?"
- "Valide o codigo Terraform com Checkov"

### AWS Pricing MCP Server

O `aws-pricing-mcp-server` consulta precos de servicos AWS. Essencial para estimativas de custo.

Configuracao:

```json
{
  "mcpServers": {
    "aws-pricing": {
      "command": "uvx",
      "args": ["awslabs.aws-pricing-mcp-server@latest"],
      "env": {
        "AWS_REGION": "us-east-1"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Exemplos:

- "Quanto custa uma instancia m6i.xlarge em us-east-1?"
- "Compare o preco de RDS MySQL vs Aurora"
- "Qual o custo de 1TB de S3 Standard vs S3 Glacier?"

## Configuracao Completa

Para ter todos os servidores disponiveis, sua configuracao ficaria assim:

```json
{
  "mcpServers": {
    "aws-api": {
      "command": "uvx",
      "args": ["awslabs.aws-api-mcp-server@latest"],
      "env": {
        "AWS_PROFILE": "default",
        "AWS_REGION": "us-east-1"
      },
      "disabled": false,
      "autoApprove": []
    },
    "aws-docs": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    },
    "aws-diagram": {
      "command": "uvx",
      "args": ["awslabs.aws-diagram-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    },
    "terraform": {
      "command": "uvx",
      "args": ["awslabs.terraform-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    },
    "aws-pricing": {
      "command": "uvx",
      "args": ["awslabs.aws-pricing-mcp-server@latest"],
      "env": {
        "AWS_REGION": "us-east-1"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

## Steering para Padronizacao

Steering files sao arquivos markdown que dao contexto e instrucoes ao Kiro. Eles ficam em `.kiro/steering/` e podem ser sempre incluidos ou condicionais.

Exemplo de steering para operacoes AWS (`.kiro/steering/aws-operations.md`):

```markdown
# AWS Operations Standards

## Naming Convention
- Resources must follow pattern: {env}-{project}-{resource}-{identifier}
- Examples: prod-api-ec2-web01, dev-data-rds-primary

## Tagging Requirements
All resources must have:
- Environment: dev/staging/prod
- Project: project name
- Owner: team email
- CostCenter: cost allocation code

## Security Standards
- No public S3 buckets unless explicitly approved
- All EC2 instances must use IMDSv2
- Security groups must not allow 0.0.0.0/0 on SSH/RDP

## Terraform Standards
- Use modules from internal registry when available
- State must be stored in S3 with DynamoDB locking
- Always run checkov before apply
```

Com esse steering, quando voce pedir ao Kiro para criar recursos, ele automaticamente aplicara esses padroes.

## Cenarios Praticos

### Cenario 1: Gerar AMI e Restaurar em Outra Regiao

Situacao: Voce precisa criar uma AMI de uma instancia em us-east-1 e disponibiliza-la em sa-east-1.

Prompt para o Kiro:

```
Preciso criar uma AMI da instancia i-0abc123def456 em us-east-1, 
copiar para sa-east-1 e lancar uma nova instancia la com as mesmas 
configuracoes de security group e subnet equivalentes.
```

O Kiro vai:

1. Criar a AMI na regiao de origem
2. Aguardar a AMI ficar disponivel
3. Copiar para a regiao de destino
4. Identificar security groups e subnets equivalentes
5. Lancar a nova instancia

Output esperado:

```
Criando AMI da instancia i-0abc123def456...
AMI criada: ami-0xyz789 (us-east-1)

Copiando AMI para sa-east-1...
AMI copiada: ami-0abc456 (sa-east-1)

Identificando configuracoes equivalentes em sa-east-1:
- VPC: vpc-sa-east-prod
- Subnet: subnet-private-1a
- Security Group: sg-web-servers

Lancando instancia...
Nova instancia: i-0new123instance (sa-east-1)
```

### Cenario 2: Analisar Recursos e Importar para Terraform

Situacao: Voce tem recursos criados manualmente e precisa importa-los para Terraform.

Prompt para o Kiro:

```
Analise os recursos do projeto "api-gateway" na conta e gere o codigo 
Terraform para importa-los. Inclua VPC, subnets, security groups, 
ALB e instancias EC2.
```

O Kiro vai:

1. Listar recursos com a tag Project=api-gateway
2. Gerar codigo Terraform para cada recurso
3. Gerar comandos de import
4. Validar o codigo gerado

Output esperado:

```hcl
# vpc.tf
resource "aws_vpc" "api_gateway" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "prod-api-gateway-vpc"
    Environment = "prod"
    Project     = "api-gateway"
  }
}

# Import command:
# terraform import aws_vpc.api_gateway vpc-0abc123
```

### Cenario 3: Analisar Terraform e Estimar Custos

Situacao: Voce tem um codigo Terraform e precisa estimar o custo mensal.

Prompt para o Kiro:

```
Analise o codigo Terraform no diretorio ./infra e estime o custo 
mensal considerando us-east-1. Detalhe por recurso.
```

O Kiro vai:

1. Ler os arquivos Terraform
2. Identificar recursos provisionados
3. Consultar precos via AWS Pricing API
4. Gerar relatorio detalhado

Output esperado:

```
Analise de Custos - ./infra

EC2 Instances:
- 2x m6i.xlarge (on-demand): $277.40/mes
- 1x t3.medium (on-demand): $30.37/mes

RDS:
- 1x db.r6g.large MySQL Multi-AZ: $438.00/mes
- Storage 100GB gp3: $11.50/mes

ALB:
- 1x Application Load Balancer: $16.20/mes
- LCU estimado (1000 req/s): $58.40/mes

S3:
- 500GB Standard: $11.50/mes
- 10M requests: $5.00/mes

NAT Gateway:
- 2x NAT Gateway: $64.80/mes
- Data processing 1TB: $45.00/mes

Total Estimado: $958.17/mes

Recomendacoes de otimizacao:
1. Considerar Reserved Instances para EC2 (-40%)
2. Avaliar Aurora Serverless para RDS se carga variavel
3. Usar S3 Intelligent-Tiering para dados com acesso variavel
```

### Cenario 4: Auditoria de Seguranca

Prompt para o Kiro:

```
Faca uma auditoria de seguranca nos recursos da conta focando em:
- Security groups com regras muito permissivas
- Buckets S3 publicos ou sem criptografia
- Instancias EC2 sem IMDSv2
- IAM users com access keys antigas
```

Output esperado:

```
Auditoria de Seguranca - 2026-04-06

CRITICO:
- sg-0abc123: Permite 0.0.0.0/0 na porta 22 (SSH)
- bucket-logs-prod: Sem criptografia server-side
- i-0xyz789: Usando IMDSv1 (vulneravel a SSRF)

ALTO:
- user-deploy: Access key com 180 dias sem rotacao
- sg-0def456: Permite 0.0.0.0/0 na porta 3306 (MySQL)

MEDIO:
- bucket-assets: Sem versionamento habilitado
- 3 instancias sem tags de Owner

Recomendacoes:
1. Restringir sg-0abc123 para IPs especificos ou usar SSM
2. Habilitar SSE-S3 no bucket-logs-prod
3. Migrar instancias para IMDSv2 com HttpTokens=required
4. Rotacionar access key do user-deploy
```

### Cenario 5: Documentacao de Arquitetura

Prompt para o Kiro:

```
Analise a infraestrutura do projeto "ecommerce" e gere:
1. Diagrama de arquitetura
2. Documentacao em markdown
3. Lista de dependencias entre servicos
```

O Kiro vai usar o aws-api para listar recursos, aws-diagram para gerar o diagrama e criar a documentacao.

## Agents para Automacao

Agents sao configuracoes que permitem ao Kiro executar tarefas especificas de forma mais direcionada. Voce pode criar agents customizados em `.kiro/agents/`.

### Agent de Auditoria EKS

Um caso de uso poderoso e criar um agent especializado em auditoria de acoes no EKS. Esse agent combina analise de CloudTrail (acoes na API AWS) com logs de audit do Kubernetes (acoes dentro do cluster) para responder a pergunta: quem fez o que e quando?

Exemplo de agent para auditoria EKS (`.kiro/agents/eks-audit.md`):

```markdown
# EKS Audit Agent

## Role
You are an EKS security auditor specialized in investigating actions 
performed on Kubernetes clusters. You correlate AWS CloudTrail events 
with Kubernetes audit logs to provide complete visibility of who did 
what and when.

## Data Sources

### CloudTrail (AWS API Actions)
- Cluster creation, deletion, updates
- Node group modifications
- IAM role assumptions
- Access entries changes
- Add-on installations

### CloudWatch Logs - Kubernetes Audit
- Log group: /aws/eks/{cluster-name}/cluster
- Log streams: kube-apiserver-audit-*
- Contains all API server requests

## Investigation Process

1. Identify the timeframe of the incident
2. Query CloudTrail for AWS-level actions on EKS resources
3. Query Kubernetes audit logs for cluster-level actions
4. Correlate user identities across both sources
5. Build timeline of events
6. Identify the actor (IAM user/role or K8s identity)

## Common Queries

### CloudTrail - EKS Actions
Filter by eventSource: eks.amazonaws.com

### Kubernetes Audit - Pod Deletions
fields @timestamp, user.username, objectRef.name, objectRef.namespace
| filter verb = "delete" and objectRef.resource = "pods"
| sort @timestamp desc

### Kubernetes Audit - ConfigMap Changes
fields @timestamp, user.username, objectRef.name, verb
| filter objectRef.resource = "configmaps"
| filter verb in ["create", "update", "patch", "delete"]
| sort @timestamp desc

### Kubernetes Audit - RBAC Changes
fields @timestamp, user.username, objectRef.name, verb
| filter objectRef.resource in ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]
| sort @timestamp desc

### Kubernetes Audit - Secret Access
fields @timestamp, user.username, objectRef.name, objectRef.namespace, verb
| filter objectRef.resource = "secrets"
| sort @timestamp desc

## Identity Correlation

### AWS to Kubernetes Identity Mapping
- IAM User: appears as user.username in audit logs
- IAM Role (via aws-auth): maps to K8s user/group
- Pod Identity: appears with service account name
- Access Entries: direct IAM to K8s principal mapping

## Output Format
Always provide:
1. Timeline of events with timestamps
2. Actor identification (IAM and K8s identity)
3. Actions performed
4. Affected resources
5. Recommendations if security issue detected

## Tools to use
- aws-api for CloudTrail queries
- aws-api for CloudWatch Logs Insights queries
- eks-mcp for cluster information if available
```

Com esse agent, voce pode fazer perguntas como:

```
Quem deletou o deployment nginx-app no namespace production 
ontem entre 14h e 16h?
```

```
Liste todas as alteracoes de RBAC no cluster prod-eks 
na ultima semana.
```

```
Investigue quem acessou secrets no namespace payments 
nos ultimos 3 dias.
```

O agent vai correlacionar automaticamente as identidades entre AWS e Kubernetes, construir uma timeline e identificar o responsavel pela acao.

### Agent de Troubleshooting

Outro agent util e o de troubleshooting geral (`.kiro/agents/troubleshoot.md`):

```markdown
# Troubleshooting Agent

## Role
You are an infrastructure troubleshooting specialist.

## Capabilities
- Analyze CloudWatch logs and metrics
- Check resource health and connectivity
- Identify common issues and suggest fixes

## Process
1. Gather information about the issue
2. Check relevant logs and metrics
3. Identify root cause
4. Suggest remediation steps
5. Verify fix if possible

## Tools to use
- aws-api for resource inspection
- aws-docs for service limits and known issues
```

## Documentacao de Projetos Terraform

Uma das tarefas mais negligenciadas em projetos de infraestrutura e a documentacao. O Kiro pode automatizar a geracao e manutencao de documentacao para seus projetos Terraform.

### Gerando Documentacao Automaticamente

Prompt para o Kiro:

```
Analise o codigo Terraform neste diretorio e gere uma documentacao 
completa em markdown incluindo:
- Visao geral da arquitetura
- Recursos provisionados
- Variaveis de entrada com descricoes
- Outputs disponiveis
- Dependencias entre modulos
- Exemplos de uso
```

O Kiro vai analisar os arquivos `.tf`, extrair informacoes de variables, outputs, resources e gerar um README.md estruturado.

Output esperado:

```markdown
# Infraestrutura API Gateway

## Visao Geral
Este modulo provisiona a infraestrutura completa para a API Gateway,
incluindo VPC, subnets, ALB, ECS Fargate e RDS PostgreSQL.

## Arquitetura
- VPC com 3 AZs (subnets publicas e privadas)
- Application Load Balancer (internet-facing)
- ECS Fargate com auto-scaling
- RDS PostgreSQL Multi-AZ
- Secrets Manager para credenciais

## Recursos Provisionados
| Recurso | Tipo | Descricao |
|---------|------|-----------|
| vpc | aws_vpc | VPC principal |
| alb | aws_lb | Load balancer publico |
| ecs_cluster | aws_ecs_cluster | Cluster ECS |
| rds | aws_db_instance | Database PostgreSQL |

## Variaveis

| Nome | Tipo | Default | Descricao |
|------|------|---------|-----------|
| environment | string | - | Ambiente (dev/staging/prod) |
| vpc_cidr | string | 10.0.0.0/16 | CIDR block da VPC |
| instance_count | number | 2 | Numero de tasks ECS |

## Outputs

| Nome | Descricao |
|------|-----------|
| alb_dns_name | DNS do load balancer |
| rds_endpoint | Endpoint do RDS |
| vpc_id | ID da VPC criada |

## Uso

\```hcl
module "api_gateway" {
  source = "./modules/api-gateway"

  environment    = "prod"
  vpc_cidr       = "10.0.0.0/16"
  instance_count = 3
}
\```
```

### Steering para Padronizacao de Documentacao

Crie um steering file para garantir que a documentacao siga seus padroes (`.kiro/steering/terraform-docs.md`):

```markdown
# Terraform Documentation Standards

## README Structure
Every Terraform module must have a README.md with:
1. Module description (what it does)
2. Architecture diagram or description
3. Requirements (provider versions, dependencies)
4. Variables table with types, defaults, and descriptions
5. Outputs table
6. Usage example
7. Notes and considerations

## Variable Documentation
All variables must have:
- description: Clear explanation of purpose
- type: Explicit type definition
- default: When applicable
- validation: When applicable

## Output Documentation
All outputs must have:
- description: What the output represents
- When to use it

## Code Comments
- Complex logic must have inline comments
- Locals should explain transformations
- Data sources should explain why they're needed
```

### Hook para Manter Documentacao Atualizada

Configure um hook para lembrar de atualizar a documentacao quando arquivos Terraform mudam:

```json
{
  "name": "Terraform Docs Reminder",
  "version": "1.0.0",
  "when": {
    "type": "fileEdited",
    "patterns": ["*.tf"]
  },
  "then": {
    "type": "askAgent",
    "prompt": "Arquivo Terraform modificado. Verifique se o README.md precisa ser atualizado com as mudancas."
  }
}
```

### Documentacao de Modulos Existentes

Para projetos legados sem documentacao, voce pode pedir ao Kiro:

```
Analise todos os modulos Terraform em ./modules e gere um 
catalogo de modulos com:
- Nome e proposito de cada modulo
- Inputs e outputs
- Exemplos de uso
- Dependencias entre modulos
```

O Kiro vai criar um documento consolidado que serve como catalogo interno de modulos reutilizaveis.

### Gerando Diagramas da Infraestrutura

Combine com o aws-diagram MCP Server:

```
Analise o codigo Terraform e gere um diagrama de arquitetura 
mostrando os recursos e suas conexoes.
```

O Kiro vai interpretar os resources, identificar relacionamentos (security groups, subnets, etc.) e gerar um diagrama visual.

## Hooks para Automacao

Hooks permitem executar acoes automaticamente em resposta a eventos. Exemplo de hook para validar Terraform antes de commits:

```json
{
  "name": "Terraform Validate",
  "version": "1.0.0",
  "when": {
    "type": "fileEdited",
    "patterns": ["*.tf"]
  },
  "then": {
    "type": "runCommand",
    "command": "terraform validate"
  }
}
```

## Boas Praticas

1. Organize MCP Servers por funcao: Nao habilite todos os servidores se nao precisar. Isso melhora performance e reduz ruido.

2. Use Steering para padroes: Documente seus padroes em steering files. O Kiro vai segui-los automaticamente.

3. Crie Agents especializados: Em vez de um agent generico, crie agents focados em tarefas especificas como troubleshooting, provisionamento ou auditoria.

4. Aproveite autoApprove com cuidado: Voce pode configurar comandos que nao precisam de aprovacao, mas use com cautela em ambientes de producao.

5. Combine MCP Servers: Os cenarios mais poderosos combinam multiplos servidores. Por exemplo, usar aws-api para listar recursos, terraform para gerar codigo e aws-pricing para estimar custos.

## Conclusao

O Kiro com MCP Servers transforma a forma como times de infraestrutura trabalham. Em vez de alternar entre terminal, console AWS, documentacao e planilhas de custo, voce tem tudo integrado em uma interface conversacional.

Os cenarios que mostramos sao apenas o comeco. A combinacao de MCP Servers, Steering e Agents permite criar fluxos de trabalho customizados para qualquer operacao de infraestrutura.

O proximo passo e experimentar. Configure os MCP Servers, crie seus steering files e comece a explorar o que o Kiro pode fazer pelo seu time.
