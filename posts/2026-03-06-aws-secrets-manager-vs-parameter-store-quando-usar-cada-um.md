---
image: /assets/img/AWS.png
title: "AWS Secrets Manager vs Parameter Store : Quando Usar Cada Um"
description: No universo da AWS, escolher a ferramenta certa para gerenciar
  segredos e configurações pode impactar diretamente a segurança, a
  escalabilidade e o custo da sua aplicação. Neste artigo, exploramos as
  principais diferenças entre o AWS Secrets Manager e o AWS Systems Manager
  Parameter Store, destacando seus casos de uso ideais, vantagens, limitações e
  implicações de custo.
date: 2026-03-06
category: security
background: "#7B68EE"
tags:
  - AWS
  - SECRETSMANAGER
  - PARAMETERSTORE
  - AWSSYSTEMSMANAGER
  - CLOUDCOMPUTING
  - SEGURANCAEMNUVEM
  - DEVOPS
  - ARQUITETURADESOFTWARE
  - GESTAODESEGREDOS
  - CONFIGURACAODEAPLICACAO
  - AWSSECURITY
  - AWSARCHITECTURE
  - CLOUDSECURITY
  - INFRAESTRUTURAEMNUVEM
categories:
  - AWS
  - SECRETSMANAGER
  - PARAMETERSTORE
  - AWSSYSTEMSMANAGER
  - CLOUDCOMPUTING
  - SEGURANCAEMNUVEM
  - DEVOPS
  - ARQUITETURADESOFTWARE
  - GESTAODESEGREDOS
  - CONFIGURACAODEAPLICACAO
  - AWSSECURITY
  - AWSARCHITECTURE
  - CLOUDSECURITY
  - INFRAESTRUTURAEMNUVEM
---
Ao projetar aplicações na Cloud, é comum surgir a dúvida sobre qual serviço utilizar para armazenar informações sensíveis e parâmetros de configuração. Escolher corretamente entre essas duas soluções pode impactar diretamente a segurança, a governança e a eficiência operacional do seu ambiente em nuvem.

### O Cenário

"Onde vamos guardar as credenciais do banco de dados?" 

"Parameter Store!" - disse o desenvolvedor júnior.

"Secrets Manager!" - disse o arquiteto sênior.

E começou a discussão. 30 minutos depois, ninguém tinha certeza de qual usar.

Spoiler: ambos estavam certos. E ambos estavam errados.

Deixa eu te contar o que aprendi depois de alguns anos trabalhando com AWS e gerenciamento de segredos.

## Porque você precisa se preocupar?

Essa decisão não é trivial. Aqui está o que está em jogo:

**O Custo da Escolha Errada:**

Cenário Real 1 (Empresa que escolheu errado):
- 5.000 secrets no Secrets Manager
- Custo: $2.000/mês
- Problema: 80% eram configs simples (não secrets)
- Solução: Migrar para Parameter Store
- Economia: $1.600/mês = $19.200/ano

Cenário Real 2 (Empresa que escolheu certo):
- Credenciais de banco RDS no Secrets Manager
- Rotação automática habilitada
- Incidentes de segurança: 0 em 2 anos
- Compliance: Passou em todas as auditorias

**O Problema:**

Muitos times escolhem baseado em:
- "É mais barato" (sem considerar features)
- "Todo mundo usa" (sem entender o caso de uso)
- "Já conhecemos" (sem avaliar alternativas)

**A Realidade:**

Você precisa de AMBOS. A questão é: quando usar cada um?


## AWS Secrets Manager

### O Que É?

Secrets Manager é o serviço premium da AWS para gerenciar informações sensíveis. Pense nele como um cofre digital com superpoderes.

### Principais Features

**1. Rotação Automática de Secrets**

Isso é ENORME. Vou mostrar um exemplo real:

```bash
# Criar secret com rotação automática para RDS
$ aws secretsmanager create-secret \
    --name prod/db/mysql \
    --description "MySQL master credentials" \
    --secret-string '{"username":"admin","password":"MySecureP@ss123"}'

{
    "ARN": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/db/mysql-AbCdEf",
    "Name": "prod/db/mysql",
    "VersionId": "EXAMPLE1-90ab-cdef-fedc-ba987EXAMPLE"
}

# Habilitar rotação automática (a cada 30 dias)
$ aws secretsmanager rotate-secret \
    --secret-id prod/db/mysql \
    --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRDSRotation \
    --rotation-rules AutomaticallyAfterDays=30

{
    "ARN": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/db/mysql-AbCdEf",
    "Name": "prod/db/mysql",
    "VersionId": "EXAMPLE2-90ab-cdef-fedc-ba987EXAMPLE"
}
```

**O que acontece nos bastidores:**

1. Lambda é invocada automaticamente a cada 30 dias
2. Cria nova senha no banco
3. Testa a nova senha
4. Atualiza o secret
5. Marca a versão antiga como deprecated
6. Suas aplicações pegam a nova senha automaticamente

**Resultado:** Zero downtime. Zero intervenção manual.


**2. Versionamento Automático**

Cada vez que você atualiza um secret, uma nova versão é criada:

```bash
# Atualizar secret
$ aws secretsmanager update-secret \
    --secret-id prod/db/mysql \
    --secret-string '{"username":"admin","password":"NewSecureP@ss456"}'

# Listar versões
$ aws secretsmanager list-secret-version-ids \
    --secret-id prod/db/mysql

{
    "Versions": [
        {
            "VersionId": "EXAMPLE2-90ab-cdef-fedc-ba987EXAMPLE",
            "VersionStages": ["AWSCURRENT"],
            "CreatedDate": 1609459200.0
        },
        {
            "VersionId": "EXAMPLE1-90ab-cdef-fedc-ba987EXAMPLE",
            "VersionStages": ["AWSPREVIOUS"],
            "CreatedDate": 1606780800.0
        }
    ]
}

# Recuperar versão anterior (rollback)
$ aws secretsmanager get-secret-value \
    --secret-id prod/db/mysql \
    --version-stage AWSPREVIOUS
```

**Por que isso importa:**

Imagine que você rotacionou a senha e algo quebrou. Com versionamento, você faz rollback em segundos.


**3. Integração Nativa com RDS, Redshift, DocumentDB**

Esse é a feature diferencial do serviço:

```bash
# Criar secret vinculado ao RDS
$ aws secretsmanager create-secret \
    --name prod/rds/postgres \
    --secret-string '{"username":"postgres","password":"InitialP@ss"}' \
    --tags Key=Database,Value=PostgreSQL

# Vincular ao RDS instance
$ aws rds modify-db-instance \
    --db-instance-identifier my-postgres-db \
    --master-user-password $(aws secretsmanager get-secret-value \
        --secret-id prod/rds/postgres \
        --query SecretString \
        --output text | jq -r .password)
```

Agora, quando você habilita rotação, o Secrets Manager:
- Conecta no RDS
- Cria novo usuário/senha
- Testa conexão
- Atualiza o secret
- Remove credenciais antigas

Tudo automaticamente. Sem Lambda customizada.

**4. Auditoria Completa com CloudTrail**

Toda operação é logada:

```bash
# Ver quem acessou o secret
$ aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=ResourceName,AttributeValue=prod/db/mysql \
    --max-results 10

{
    "Events": [
        {
            "EventTime": "2026-03-06T10:30:00Z",
            "Username": "api-service-role",
            "EventName": "GetSecretValue",
            "Resources": [{
                "ResourceName": "prod/db/mysql"
            }]
        }
    ]
}
```

Perfeito para compliance (SOC2, PCI-DSS, HIPAA).


### Quando Usar Secrets Manager?

**Use quando você tem:**

1. **Credenciais de banco de dados**
   - RDS, Aurora, Redshift, DocumentDB
   - Precisa de rotação automática
   - Compliance exige rotação periódica

2. **API Keys de terceiros**
   - Stripe, Twilio, SendGrid
   - Precisa rotacionar periodicamente
   - Quer auditoria de acesso

3. **Tokens OAuth/JWT**
   - Refresh tokens
   - Service account tokens
   - Precisa invalidar versões antigas

4. **Requisitos de Compliance**
   - SOC2, PCI-DSS, HIPAA
   - Auditoria obrigatória
   - Rotação documentada

### Custos do Secrets Manager

Aqui está a realidade dos custos:

```
Preço (us-east-1):
- $0.40 por secret por mês
- $0.05 por 10.000 API calls

Exemplo Real:
- 100 secrets
- 1 milhão de API calls/mês
- Custo: (100 × $0.40) + (1.000.000 / 10.000 × $0.05)
- Total: $40 + $5 = $45/mês
```

**Parece caro?** Vamos comparar com o custo de um incidente de segurança:

- Vazamento de credenciais: R$20.000 - R$500.000
- Downtime por rotação manual: R$5.000 - R$50.000/hora
- Multa de compliance: R$10.000 - R$1.000.000

$45/mês parece barato agora, né?


## AWS Systems Manager Parameter Store

### O Que É?

Parameter Store é o canivete suíço da AWS para configurações. Ele guarda secrets, mas também configs, feature flags, e muito mais.

### Principais Features

**1. Três Tipos de Parâmetros**

```bash
# Standard (grátis, até 10.000 parâmetros)
$ aws ssm put-parameter \
    --name /app/config/api-url \
    --value "https://api.example.com" \
    --type String

# SecureString (criptografado com KMS)
$ aws ssm put-parameter \
    --name /app/secrets/api-key \
    --value "sk_live_abc123xyz" \
    --type SecureString \
    --key-id alias/aws/ssm

# Advanced (features extras, pago)
$ aws ssm put-parameter \
    --name /app/config/feature-flags \
    --value '{"new_ui":true,"beta_features":false}' \
    --type String \
    --tier Advanced \
    --policies '[{"Type":"Expiration","Version":"1.0","Attributes":{"Timestamp":"2026-12-31T23:59:59Z"}}]'
```

**2. Hierarquia de Parâmetros**

Organize por ambiente, aplicação, componente:

```bash
# Estrutura hierárquica
/production/
  /api/
    /database/
      host
      port
      username
      password
    /redis/
      host
      port
  /frontend/
    /config/
      api-url
      cdn-url

# Buscar todos os parâmetros de um path
$ aws ssm get-parameters-by-path \
    --path /production/api/database \
    --recursive \
    --with-decryption

{
    "Parameters": [
        {
            "Name": "/production/api/database/host",
            "Value": "prod-db.cluster-abc.us-east-1.rds.amazonaws.com",
            "Type": "String"
        },
        {
            "Name": "/production/api/database/password",
            "Value": "MySecurePassword123",
            "Type": "SecureString"
        }
    ]
}
```


**3. Versionamento Manual**

```bash
# Criar parâmetro
$ aws ssm put-parameter \
    --name /app/config/max-connections \
    --value "100" \
    --type String

# Atualizar (cria nova versão)
$ aws ssm put-parameter \
    --name /app/config/max-connections \
    --value "200" \
    --type String \
    --overwrite

# Ver histórico
$ aws ssm get-parameter-history \
    --name /app/config/max-connections

{
    "Parameters": [
        {
            "Name": "/app/config/max-connections",
            "Value": "200",
            "Version": 2,
            "LastModifiedDate": "2026-03-06T15:30:00Z"
        },
        {
            "Name": "/app/config/max-connections",
            "Value": "100",
            "Version": 1,
            "LastModifiedDate": "2026-03-01T10:00:00Z"
        }
    ]
}

# Recuperar versão específica
$ aws ssm get-parameter \
    --name /app/config/max-connections:1
```

**4. Políticas de Expiração e Notificação**

```bash
# Parâmetro com expiração (Apenas no Advanced)
$ aws ssm put-parameter \
    --name /app/temp/beta-token \
    --value "temp_token_xyz" \
    --type SecureString \
    --tier Advanced \
    --policies '[
        {
            "Type":"Expiration",
            "Version":"1.0",
            "Attributes":{
                "Timestamp":"2026-04-01T00:00:00Z"
            }
        },
        {
            "Type":"ExpirationNotification",
            "Version":"1.0",
            "Attributes":{
                "Before":"7",
                "Unit":"Days"
            }
        }
    ]'
```

Você recebe notificação 7 dias antes de expirar!


### Quando Usar Parameter Store?

**Use quando você tem:**

1. **Configurações de Aplicação**
   - URLs de APIs
   - Feature flags
   - Timeouts, limites
   - Configurações por ambiente

2. **Secrets Simples (sem rotação)**
   - API keys que você rotaciona manualmente
   - Tokens de integração
   - Chaves de criptografia

3. **Parâmetros de Infraestrutura**
   - AMI IDs
   - VPC IDs
   - Security Group IDs
   - Subnet IDs

4. **Valores Compartilhados**
   - Configurações usadas por múltiplos serviços
   - Constantes do sistema
   - Endpoints internos

5. **Orçamento Limitado**
   - Standard tier é GRÁTIS
   - Até 10.000 parâmetros
   - Throughput: 1.000 TPS

### Custos do Parameter Store

```
Standard Tier (GRÁTIS):
- Até 10.000 parâmetros
- Até 4KB por parâmetro
- 1.000 TPS (transactions per second)
- Sem políticas avançadas

Advanced Tier:
- $0.05 por parâmetro por mês
- Até 8KB por parâmetro
- 10.000 TPS
- Políticas de expiração/notificação

API Calls:
- Standard: GRÁTIS (primeiros 1 milhão/mês)
- Advanced: $0.05 por 10.000 API calls

Exemplo Real:
- 500 parâmetros standard (configs)
- 50 parâmetros advanced (secrets)
- 500.000 API calls/mês
- Custo: $0 + (50 × $0.05) + $0 = $2.50/mês
```

Muito mais barato que Secrets Manager!


## Comparação Lado a Lado

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| **Rotação Automática** | Sim (built-in) | Não (manual) |
| **Versionamento** | Automático | Manual |
| **Integração RDS** | Nativa | Via Lambda |
| **Custo** | $0.40/secret/mês | GRÁTIS (standard) |
| **Limite de tamanho** | 64KB | 4KB (standard), 8KB (advanced) |
| **Auditoria** | CloudTrail | CloudTrail |
| **Criptografia** | KMS (obrigatório) | KMS (opcional) |
| **Cross-region replication** | Sim | Não |
| **Políticas de expiração** | Não | Sim (advanced) |
| **Hierarquia** | Via naming | Nativa |
| **API throughput** | Ilimitado | 1.000 TPS (standard) |

## Cenários Reais de Uso

Vou mostrar 3 arquiteturas fictícias.

### Cenário 1: E-commerce com Microserviços

**Arquitetura:**

```
Application Stack:
├── API Gateway
├── 15 microserviços (ECS Fargate)
├── RDS PostgreSQL (master + 2 replicas)
├── ElastiCache Redis
├── S3 + CloudFront
└── Integrações: Stripe, SendGrid, Twilio
```

**Decisão:**

```bash
# Secrets Manager (rotação automática)
/prod/secrets/
  ├── rds/postgres-master          # $0.40/mês
  ├── stripe/api-key               # $0.40/mês
  ├── sendgrid/api-key             # $0.40/mês
  └── twilio/auth-token            # $0.40/mês

# Parameter Store (configs)
/prod/config/
  ├── api/
  │   ├── base-url                 # GRÁTIS
  │   ├── timeout                  # GRÁTIS
  │   └── max-connections          # GRÁTIS
  ├── redis/
  │   ├── host                     # GRÁTIS
  │   └── port                     # GRÁTIS
  └── feature-flags/
      ├── new-checkout             # GRÁTIS
      └── beta-features            # GRÁTIS
```

**Custo Total:**
- Secrets Manager: 4 secrets × $0.40 = $1.60/mês
- Parameter Store: GRÁTIS
- Total: $1.60/mês

**Benefício:**
- Rotação automática de credenciais críticas
- Configs flexíveis sem custo
- Compliance garantido


### Cenário 2: SaaS Multi-tenant

**Arquitetura:**

```
Multi-tenant SaaS:
├── 500 clientes
├── 1 banco por cliente (RDS)
├── Shared services (auth, billing, analytics)
└── Tenant-specific configs
```

**Decisão:**

```bash
# Secrets Manager (1 secret por tenant)
/tenants/
  ├── tenant-001/rds-credentials   # $0.40/mês
  ├── tenant-002/rds-credentials   # $0.40/mês
  └── ... (500 tenants)

# Parameter Store (configs por tenant)
/tenants/
  ├── tenant-001/
  │   ├── plan                     # GRÁTIS
  │   ├── features                 # GRÁTIS
  │   └── limits                   # GRÁTIS
  └── tenant-002/
      ├── plan                     # GRÁTIS
      └── ...
```

**Custo Total:**
- Secrets Manager: 500 × $0.40 = $200/mês
- Parameter Store: GRÁTIS (1.500 parâmetros)
- Total: $200/mês

**Alternativa (se custo for problema):**

```bash
# Usar Parameter Store SecureString para tudo
/tenants/
  ├── tenant-001/
  │   ├── rds-credentials (SecureString)  # GRÁTIS
  │   ├── plan                            # GRÁTIS
  │   └── features                        # GRÁTIS

# Implementar rotação manual via Lambda + EventBridge
# Custo: ~$5/mês (Lambda + EventBridge)
# Economia: $195/mês
```

**Trade-off:**
- Economia de $195/mês
- Mas: rotação manual, mais complexidade
- Decisão: depende do seu orçamento e requisitos de compliance


### Cenário 3: Startup com Budget Apertado

**Arquitetura:**

```
MVP Stack:
├── 1 aplicação monolítica (ECS)
├── RDS PostgreSQL
├── Redis
└── 3 integrações externas
```

**Decisão (Budget-conscious):**

```bash
# Parameter Store para TUDO (inicialmente)
/prod/
  ├── database/
  │   ├── host                     # GRÁTIS
  │   ├── port                     # GRÁTIS
  │   ├── username                 # GRÁTIS
  │   └── password (SecureString)  # GRÁTIS
  ├── redis/
  │   ├── host                     # GRÁTIS
  │   └── port                     # GRÁTIS
  └── integrations/
      ├── stripe-key (SecureString)    # GRÁTIS
      ├── sendgrid-key (SecureString)  # GRÁTIS
      └── analytics-key (SecureString) # GRÁTIS
```

**Custo Total:** $0/mês 🎉

**Plano de Migração (quando crescer):**

```bash
# Fase 1: Startup (0-10k usuários)
- Tudo no Parameter Store
- Rotação manual trimestral
- Custo: $0/mês

# Fase 2: Growth (10k-100k usuários)
- Migrar DB credentials para Secrets Manager
- Manter configs no Parameter Store
- Custo: $0.40/mês

# Fase 3: Scale (100k+ usuários)
- Secrets Manager para todos os secrets críticos
- Parameter Store para configs
- Rotação automática habilitada
- Custo: $5-10/mês
```

**Lição:** Comece simples, escale conforme necessário.


## Implementação Prática

Para interagir com os serviços da AWS, precisamos utilizar o SDK para interagir. Vamos ver uma implementação de código com Python utilizando o Boto3.

### Python (Boto3)

```python
import boto3
import json
from botocore.exceptions import ClientError

# Clients
secrets_client = boto3.client('secretsmanager')
ssm_client = boto3.client('ssm')

# 1. Buscar secret do Secrets Manager
def get_secret(secret_name):
    try:
        response = secrets_client.get_secret_value(SecretId=secret_name)
        return json.loads(response['SecretString'])
    except ClientError as e:
        print(f"Error: {e}")
        raise

# 2. Buscar parâmetro do Parameter Store
def get_parameter(parameter_name, decrypt=True):
    try:
        response = ssm_client.get_parameter(
            Name=parameter_name,
            WithDecryption=decrypt
        )
        return response['Parameter']['Value']
    except ClientError as e:
        print(f"Error: {e}")
        raise

# 3. Buscar múltiplos parâmetros por path
def get_parameters_by_path(path):
    try:
        response = ssm_client.get_parameters_by_path(
            Path=path,
            Recursive=True,
            WithDecryption=True
        )
        return {p['Name']: p['Value'] for p in response['Parameters']}
    except ClientError as e:
        print(f"Error: {e}")
        raise

# Uso
if __name__ == "__main__":
    # Database credentials (Secrets Manager)
    db_creds = get_secret('prod/db/postgres')
    print(f"DB Host: {db_creds['host']}")
    print(f"DB User: {db_creds['username']}")
    
    # API URL (Parameter Store)
    api_url = get_parameter('/prod/config/api-url')
    print(f"API URL: {api_url}")
    
    # Todas as configs de um ambiente
    configs = get_parameters_by_path('/prod/config')
    print(f"Configs: {configs}")
```


## Porque Nem Tudo Funciona de Primeira

### Problema 1: Esqueci de Dar Permissão IAM

Primeira vez que tentei usar Secrets Manager:

```python
db_creds = get_secret('prod/db/postgres')

# Error: AccessDeniedException
# User: arn:aws:iam::123456789012:user/developer is not authorized 
# to perform: secretsmanager:GetSecretValue on resource: prod/db/postgres
```

Esqueci de adicionar a policy IAM!

**Solução:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "secretsmanager:GetSecretValue",
      "secretsmanager:DescribeSecret"
    ],
    "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/*"
  }]
}
```

### Problema 2: KMS Key Não Configurada

Tentei criar SecureString no Parameter Store:

```bash
$ aws ssm put-parameter \
    --name /prod/secrets/api-key \
    --value "secret123" \
    --type SecureString

# Error: InvalidKeyId
# The request contains an invalid key identifier
```

**O que aconteceu:** Por padrão, usa `alias/aws/ssm`, mas eu não tinha permissão.

**Solução:**

```bash
# Criar KMS key própria
$ aws kms create-key --description "SSM Parameters"

# Ou usar a key padrão com permissão correta
$ aws ssm put-parameter \
    --name /prod/secrets/api-key \
    --value "secret123" \
    --type SecureString \
    --key-id alias/aws/ssm
```


### Problema 3: Rotação Automática Quebrou a Aplicação

Habilitei rotação automática no Secrets Manager:

```bash
$ aws secretsmanager rotate-secret \
    --secret-id prod/db/postgres \
    --rotation-rules AutomaticallyAfterDays=30
```

2 semanas depois: aplicação começou a falhar intermitentemente.

**O que aconteceu:**

A aplicação estava cacheando as credenciais por 24 horas. Quando a rotação aconteceu, metade dos pods tinha credenciais antigas (que foram revogadas).

**Solução:**

```python
import boto3
from datetime import datetime, timedelta

class SecretCache:
    def __init__(self, secret_name, ttl_seconds=300):  # 5 min cache
        self.secret_name = secret_name
        self.ttl_seconds = ttl_seconds
        self.cached_secret = None
        self.cached_at = None
        self.client = boto3.client('secretsmanager')
    
    def get_secret(self):
        now = datetime.now()
        
        # Cache expirado ou não existe
        if (self.cached_secret is None or 
            self.cached_at is None or 
            (now - self.cached_at).seconds > self.ttl_seconds):
            
            response = self.client.get_secret_value(
                SecretId=self.secret_name
            )
            self.cached_secret = json.loads(response['SecretString'])
            self.cached_at = now
        
        return self.cached_secret

# Uso
cache = SecretCache('prod/db/postgres', ttl_seconds=300)
db_creds = cache.get_secret()  # Revalida a cada 5 minutos
```

**Lição:** Sempre use cache com TTL curto ( de poucos minutos, até 10 ) para secrets que rotacionam.


## Boas práticas

### 1. Naming Conventions

Use hierarquia consistente:

```bash
# Bom
/production/api/database/host
/production/api/database/password
/production/frontend/config/api-url

# Ruim
/prod-api-db-host
/api_database_password
/frontend-api-url
```

### 2. Separação por Ambiente

```bash
# Estrutura recomendada
/{environment}/{application}/{component}/{parameter}

# Exemplos
/production/api/database/host
/staging/api/database/host
/development/api/database/host
```

### 3. Tags para Organização

```bash
# Secrets Manager
$ aws secretsmanager tag-resource \
    --secret-id prod/db/postgres \
    --tags Key=Environment,Value=production \
           Key=Application,Value=api \
           Key=CostCenter,Value=engineering \
           Key=Compliance,Value=pci-dss

# Parameter Store
$ aws ssm add-tags-to-resource \
    --resource-type Parameter \
    --resource-id /prod/config/api-url \
    --tags Key=Environment,Value=production \
           Key=Application,Value=api
```

### 4. Least Privilege IAM

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/api/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters",
        "ssm:GetParametersByPath"
      ],
      "Resource": [
        "arn:aws:ssm:us-east-1:123456789012:parameter/prod/api/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt"
      ],
      "Resource": [
        "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"
      ]
    }
  ]
}
```

Nunca use `"Resource": "*"`!

## Como decidir qual usar?

Indico fortemente que utilizem este fluxo para decidir:

```
Você precisa armazenar algo?
│
├─ É uma credencial que precisa rotacionar automaticamente?
│  │
│  ├─ SIM → Secrets Manager
│  │  └─ Exemplos: RDS, Redshift, DocumentDB credentials
│  │
│  └─ NÃO → Continue...
│
├─ É um secret crítico (API key, token, certificado)?
│  │
│  ├─ SIM → Precisa de compliance/auditoria rigorosa?
│  │  │
│  │  ├─ SIM → Secrets Manager
│  │  │  └─ Exemplos: PCI-DSS, HIPAA, SOC2
│  │  │
│  │  └─ NÃO → Orçamento é limitado?
│  │     │
│  │     ├─ SIM → Parameter Store (SecureString)
│  │     │  └─ Rotação manual via Lambda
│  │     │
│  │     └─ NÃO → Secrets Manager
│  │        └─ Melhor UX, menos manutenção
│  │
│  └─ NÃO → Continue...
│
└─ É uma configuração (não secret)?
   │
   └─ SIM → Parameter Store (Standard)
      └─ Exemplos: URLs, feature flags, configs
```

## Conclusão

A escolha entre Secrets Manager e Parameter Store não é "um ou outro". É "quando usar cada um".

**Regra de Ouro:**

```
Secrets Manager = Secrets críticos com rotação automática
Parameter Store = Configs + Secrets simples sem rotação
```

**Minha Recomendação:**

1. **Comece com Parameter Store** (é grátis!)
2. **Migre secrets críticos** para Secrets Manager conforme necessário
3. **Habilite rotação automática** para DB credentials
4. **Monitore custos** e otimize

E lembre-se: segurança não é custo, é investimento. Se você ainda tem secrets hardcoded no código, pare tudo e migre agora. Sério. Não espere acontecer um incidente.