---
image: /assets/img/AWS.png
title: Como o IAM Policy Autopilot pode melhorar suas políticas IAM
description: Criar políticas IAM com least privilege sempre foi um desafio que
  consome tempo e gera frustração. O IAM Policy Autopilot, lançado no re:Invent
  2024 e agora disponível como Kiro Power, muda esse cenário ao analisar seu
  código e gerar políticas IAM automaticamente. Entenda como essa ferramenta
  open source acelera desenvolvimento, reduz erros de permissão, e se integra
  perfeitamente com assistentes de IA.
date: 2026-03-02
category: ai
background: "#FF6B6B"
tags:
  - IAM
  - SECURITY
  - AWSIAM
  - LEASTPRIVILEGE
  - IAMPOLICIES
  - DEVOPS
  - AUTOMATION
  - KIRO
  - AIASSISTANTS
  - OPENSOURCE
categories:
  - IAM
  - SECURITY
  - AWSIAM
  - LEASTPRIVILEGE
  - IAMPOLICIES
  - DEVOPS
  - AUTOMATION
  - KIRO
  - AIASSISTANTS
  - OPENSOURCE
---
Você já passou horas debugando um erro de `AccessDenied` em produção? Ou criou uma política IAM com `"Action": "*"` porque não tinha certeza de quais permissões eram necessárias? Você não está sozinho. Criar políticas IAM seguindo o princípio de least privilege é um dos maiores desafios no desenvolvimento AWS.

O problema é real: desenvolvedores querem focar em escrever código de aplicação, não em decifrar documentação IAM ou troubleshooting de permissões. Assistentes de IA como Kiro, Claude e Cursor ajudam a escrever código rapidamente, mas frequentemente geram políticas IAM incorretas ou excessivamente permissivas.

Em dezembro de 2025, a AWS lançou o IAM Policy Autopilot no re:Invent, uma ferramenta open source que resolve esse problema. E em fevereiro de 2026, ela se tornou ainda mais acessível como Kiro Power, permitindo instalação com um clique e integração perfeita com seu fluxo de desenvolvimento.

Neste artigo, você vai aprender:

* Os desafios reais de criar políticas least privilege
* Como o IAM Policy Autopilot funciona internamente
* Diferenças entre uso via CLI e MCP server
* Integração com Kiro Powers
* Casos de uso práticos
* Limitações e boas práticas

No final, você terá o conhecimento necessário para usar o IAM Policy Autopilot e acelerar significativamente seu desenvolvimento AWS.

## O desafio de criar políticas least privilege

### Problema 1: Documentação complexa e em constante mudança

A AWS tem mais de 300 serviços, cada um com dezenas (às vezes centenas) de ações IAM. Encontrar as permissões corretas exige:

```
1. Identificar qual serviço você está usando
2. Descobrir quais ações IAM correspondem às chamadas SDK
3. Entender dependências entre serviços
4. Verificar se há permissões adicionais necessárias
5. Consultar documentação (que pode estar desatualizada)
```

**Exemplo real**:

Você quer fazer upload de um arquivo para S3 com criptografia KMS:

```python
# Código simples
s3_client.put_object(
    Bucket='my-bucket',
    Key='file.txt',
    Body=data,
    ServerSideEncryption='aws:kms',
    SSEKMSKeyId='arn:aws:kms:...'
)
```

**Permissões necessárias** (não óbvias):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",           // Óbvio
        "kms:GenerateDataKey",    // Não óbvio!
        "kms:Decrypt"             // Não óbvio!
      ],
      "Resource": "*"
    }
  ]
}
```

Sem `kms:GenerateDataKey`, você recebe `AccessDenied` mesmo tendo `s3:PutObject`.


### Problema 2: Ciclo de desenvolvimento lento

O fluxo tradicional de desenvolvimento com IAM:

```
1. Escrever código
2. Criar política IAM (chute inicial)
3. Deploy
4. Testar
5. AccessDenied ❌
6. Investigar qual permissão falta
7. Atualizar política
8. Deploy novamente
9. Testar
10. Outro AccessDenied ❌
11. Repetir 6-10 várias vezes...
```

**Tempo desperdiçado**: 30-60 minutos por erro de permissão.

**Frustração**: Alta. Você só quer que a aplicação funcione.

### Problema 3: Políticas excessivamente permissivas

Sob pressão de prazos, desenvolvedores frequentemente criam políticas permissivas demais:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",           // ❌ Tudo permitido
      "Resource": "*"          // ❌ Em todos os recursos
    }
  ]
}
```

**Consequências**:

* Violação do princípio de least privilege
* Maior superfície de ataque
* Falhas em auditorias de segurança
* Dificuldade de refinar depois

### Problema 4: Assistentes de IA alucinam políticas

Assistentes de IA como ChatGPT, Claude, e Cursor são ótimos para código, mas frequentemente erram em IAM:

**Exemplo de alucinação**:

```
Você: "Crie uma política para Lambda acessar DynamoDB"

AI: "Aqui está a política:"

{
  "Effect": "Allow",
  "Action": [
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:Query",
    "dynamodb:Scan",
    "dynamodb:UpdateTable"    // ❌ Ação que não existe!
  ],
  "Resource": "*"
}
```

**Problemas**:

* Ações inexistentes ou incorretas
* Sintaxe inválida
* Permissões faltando
* Recursos mal especificados


## IAM Policy Autopilot: A solução

O IAM Policy Autopilot é uma ferramenta open source da AWS que resolve esses problemas através de análise estática de código.

### Como funciona

```
┌─────────────────────────────────────────────────┐
│         Seu código (Python/Go/TypeScript)       │
│                                                 │
│  s3_client.put_object(...)                      │
│  dynamodb.get_item(...)                         │
│  lambda_client.invoke(...)                      │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│        IAM Policy Autopilot                     │
│                                                 │
│  1. Analisa chamadas SDK                        │
│  2. Mapeia para ações IAM                       │
│  3. Identifica dependências cross-service       │
│  4. Gera política JSON válida                   │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│         Política IAM gerada                     │
│                                                 │
│  {                                              │
│    "Effect": "Allow",                           │
│    "Action": [                                  │
│      "s3:PutObject",                            │
│      "kms:GenerateDataKey",  // Detectado!      │
│      "dynamodb:GetItem",                        │
│      "lambda:InvokeFunction"                    │
│    ],                                           │
│    "Resource": "*"                              │
│  }                                              │
└─────────────────────────────────────────────────┘
```

### Características principais

**1. Análise determinística**:

Diferente de IAs que podem alucinar, o IAM Policy Autopilot usa análise estática de código:

```python
# Seu código
import boto3

s3 = boto3.client('s3')
s3.put_object(Bucket='my-bucket', Key='file.txt', Body=data)
```

**IAM Policy Autopilot detecta**:
* Chamada SDK: `s3.put_object()`
* Ação IAM: `s3:PutObject`
* Dependências: `kms:GenerateDataKey` (se bucket usa KMS)

**Resultado**: Política sempre válida e sintaxe correta.

**2. Entende dependências cross-service**:

O IAM Policy Autopilot conhece padrões comuns de uso da AWS:

```
s3:PutObject → Pode precisar de kms:GenerateDataKey
lambda:InvokeFunction → Pode precisar de logs:CreateLogGroup
dynamodb:PutItem → Pode precisar de kms:Decrypt
```

**3. Sempre atualizado**:

A ferramenta é mantida pela AWS e atualizada regularmente com:
* Novos serviços
* Novas ações IAM
* Novos padrões de integração

**4. Suporta 3 linguagens**:

* Python
* Go
* TypeScript


## Modos de uso: CLI vs MCP Server

O IAM Policy Autopilot funciona de duas formas:

### 1. CLI (Command Line Interface)

**Quando usar**: Pipelines CI/CD, scripts, uso direto no terminal.

**Instalação**:

```bash
# Opção 1: Via uv (recomendado)
uvx iam-policy-autopilot

# Opção 2: Via pip
pip install iam-policy-autopilot

# Opção 3: Script direto (macOS/Linux)
curl -sSL https://github.com/awslabs/iam-policy-autopilot/raw/refs/heads/main/install.sh | sudo sh
```

**Uso básico**:

```bash
# Gerar política a partir de código
iam-policy-autopilot generate-policies \
  ./src/app.py \
  --region us-east-1 \
  --account 123456789012 \
  --pretty

# Resultado:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "kms:GenerateDataKey",
        "kms:Decrypt"
      ],
      "Resource": "*"
    }
  ]
}
```

**Uso avançado com service hints**:

```bash
# Limitar análise a serviços específicos (mais preciso)
iam-policy-autopilot generate-policies \
  ./src/app.py \
  --service-hints s3 dynamodb lambda \
  --pretty
```

**Por que usar service hints**:

Sem hints, métodos genéricos podem gerar permissões desnecessárias:

```python
# Método chamado listAccounts()
def listAccounts():
    # Código aqui
```

**Sem hints**: Gera permissões para `organizations:ListAccounts` E `chime:ListAccounts`

**Com hints** (`--service-hints organizations`): Gera apenas `organizations:ListAccounts`

**Corrigir AccessDenied**:

```bash
# Analisar erro e sugerir fix
iam-policy-autopilot fix-access-denied \
  "User: arn:aws:iam::123456789012:user/test is not authorized to perform: s3:GetObject on resource: arn:aws:s3:::my-bucket/file.txt"

# Aplicar fix automaticamente
iam-policy-autopilot fix-access-denied \
  "User: arn:aws:iam::123456789012:user/test is not authorized to perform: s3:GetObject on resource: arn:aws:s3:::my-bucket/file.txt" \
  --yes
```


### 2. MCP Server

**Quando usar**: Integração com assistentes de IA (Kiro, Claude, Cursor, Cline).

**O que é MCP**:

Model Context Protocol é um padrão aberto que permite assistentes de IA acessarem ferramentas externas de forma estruturada.

**Como funciona**:

```
Você: "Crie uma Lambda que faz upload para S3"
    ↓
Assistente de IA (Kiro):
    1. Escreve código da Lambda
    2. Detecta que precisa de política IAM
    3. Chama IAM Policy Autopilot via MCP
    4. Recebe política gerada
    5. Cria CloudFormation/CDK com política correta
    ↓
Resultado: Código + Infraestrutura + IAM correto
```

**Configuração para Kiro** (via MCP tradicional):

Adicione ao `.kiro/settings/mcp.json`:

```json
{
  "mcpServers": {
    "iam-policy-autopilot": {
      "command": "uvx",
      "args": ["iam-policy-autopilot", "mcp-server"],
      "env": {
        "AWS_PROFILE": "your-profile-name",
        "AWS_REGION": "us-east-1"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

**Configuração para Kiro IDE**:

Adicione ao `~/.kiro/settings/mcp.json` (macOS):

```json
{
  "mcpServers": {
    "iam-policy-autopilot": {
      "command": "uvx",
      "args": ["iam-policy-autopilot", "mcp-server"],
      "env": {
        "AWS_PROFILE": "your-profile-name",
        "AWS_REGION": "us-east-1"
      }
    }
  }
}
```

**Exemplo de uso com assistente de IA**:

```
Você: "Preciso criar uma Lambda que faz upload de arquivos para S3 
      com criptografia KMS. Pode me ajudar?"

Kiro: "Vou criar a Lambda e a infraestrutura necessária."

[Kiro escreve o código da Lambda]

Kiro: "Agora vou gerar as permissões IAM necessárias usando 
      IAM Policy Autopilot."

[Kiro chama IAM Policy Autopilot via MCP]

Kiro: "IAM Policy Autopilot gerou as permissões incluindo S3 PutObject, 
      KMS GenerateDataKey, e CloudWatch Logs. Vou criar o template 
      CloudFormation completo."

[Kiro cria template com política correta]

Resultado: Lambda + IAM + Infraestrutura prontos para deploy
```


## Kiro Power: A evolução do MCP

Em 23 de fevereiro de 2026, o IAM Policy Autopilot se tornou disponível como Kiro Power, oferecendo uma experiência ainda melhor que o MCP tradicional.

### O que são Kiro Powers?

Kiro Powers são integrações refinadas que vão além do MCP tradicional:

* **Instalação com um clique**: Sem configuração manual de JSON
* **Carregamento seletivo**: Ferramentas carregadas apenas quando necessário
* **Guias de uso**: Tutoriais integrados sobre como usar as ferramentas
* **Validação de instalação**: Detecção automática de problemas
* **Menos tokens LLM**: Contexto mais limpo e focado

### Diferenças: MCP tradicional vs Kiro Power

| Aspecto | MCP Tradicional | Kiro Power |
|---------|----------------|------------|
| Instalação | Manual (editar JSON) | Um clique na UI |
| Configuração | Requer conhecimento técnico | Guiada e automática |
| Carregamento | Todas as ferramentas sempre | Seletivo e sob demanda |
| Documentação | Externa | Integrada no Power |
| Troubleshooting | Manual | Validação automática |
| Uso de tokens | Alto (contexto completo) | Baixo (carregamento seletivo) |
| Onboarding | Você descobre sozinho | Tutorial guiado |

### Instalando o Kiro Power

**Opção 1: Via GitHub URL**:

1. Abra Kiro
2. Vá em "Powers" no menu lateral
3. Clique em "Add Custom Power" → "Import power from Github"
4. Cole a URL:
```
https://github.com/awslabs/iam-policy-autopilot/tree/main/power-iam-policy-autopilot
```
5. Kiro instala automaticamente

**Opção 2: Via pasta local**:

1. Clone o repositório:
```bash
git clone https://github.com/awslabs/iam-policy-autopilot.git
cd iam-policy-autopilot
```

2. No Kiro:
   * Vá em "Powers" → "Add Custom Power" → "Import power from a folder"
   * Selecione a pasta `power-iam-policy-autopilot`

3. Kiro instala automaticamente

### Usando o Kiro Power

Após instalação, o Power aparece no menu "Powers":

```
Você: "Ative o IAM Policy Autopilot Power"

Kiro: "Power ativado! Aqui está um tutorial rápido:

1. generate_iam_policies: Analisa seu código e gera políticas
2. fix_access_denied: Corrige erros de AccessDenied
3. explain_policy: Explica o que cada permissão faz

Vou validar a instalação..."

[Kiro verifica se uv está instalado, AWS configurado, etc.]

Kiro: "✅ Tudo configurado corretamente! Pronto para usar."
```

**Exemplo de uso**:

```
Você: "Tenho este código Python que acessa S3 e DynamoDB. 
      Pode gerar a política IAM?"

[Você cola o código]

Kiro: "Vou usar o IAM Policy Autopilot Power para analisar."

[Kiro chama generate_iam_policies]

Kiro: "Política gerada! Detectei:
- s3:GetObject, s3:PutObject
- dynamodb:GetItem, dynamodb:PutItem
- kms:GenerateDataKey (para criptografia S3)
- logs:CreateLogGroup (para CloudWatch)

Quer que eu crie o template CloudFormation com essa política?"
```

### Por que usar Kiro Power ao invés de MCP tradicional?

**1. Experiência de onboarding superior**:

```
MCP tradicional:
- Editar JSON manualmente
- Descobrir comandos por tentativa e erro
- Troubleshooting manual se algo falhar

Kiro Power:
- Instalação com um clique
- Tutorial integrado
- Validação automática de instalação
```

**2. Melhor performance**:

```
MCP tradicional:
- Carrega todas as ferramentas sempre
- Consome mais tokens LLM
- Contexto pode ficar poluído

Kiro Power:
- Carrega ferramentas sob demanda
- Usa menos tokens
- Contexto mais limpo e focado
```

**3. Guias contextuais**:

O Power inclui steering files que orientam o assistente sobre:
* Quando usar cada ferramenta
* Boas práticas de IAM
* Como interpretar resultados
* Casos de uso comuns


## Casos de uso práticos

### Caso 1: Lambda com S3 e KMS

**Cenário**: Criar Lambda que processa arquivos do S3 com criptografia KMS.

**Código**:

```python
import boto3
import json

s3 = boto3.client('s3')
kms = boto3.client('kms')

def lambda_handler(event, context):
    # Ler arquivo do S3
    response = s3.get_object(
        Bucket='my-bucket',
        Key='input.json'
    )
    
    data = json.loads(response['Body'].read())
    
    # Processar dados
    processed = process_data(data)
    
    # Salvar resultado com criptografia KMS
    s3.put_object(
        Bucket='my-bucket',
        Key='output.json',
        Body=json.dumps(processed),
        ServerSideEncryption='aws:kms',
        SSEKMSKeyId='arn:aws:kms:us-east-1:123456789012:key/abc-123'
    )
    
    return {'statusCode': 200}
```

**Gerar política**:

```bash
iam-policy-autopilot generate-policies \
  lambda_function.py \
  --region us-east-1 \
  --account 123456789012 \
  --pretty
```

**Política gerada**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "kms:Decrypt",           // Para ler arquivo criptografado
        "kms:GenerateDataKey",   // Para criptografar novo arquivo
        "logs:CreateLogGroup",   // Para CloudWatch Logs
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

**Tempo economizado**: 20-30 minutos de pesquisa e troubleshooting.

### Caso 2: API Gateway + Lambda + DynamoDB

**Cenário**: API REST que salva dados no DynamoDB.

**Código**:

```python
import boto3
import json
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('users')

def lambda_handler(event, context):
    body = json.loads(event['body'])
    
    # Salvar no DynamoDB
    table.put_item(
        Item={
            'userId': body['userId'],
            'name': body['name'],
            'email': body['email'],
            'createdAt': datetime.now().isoformat()
        }
    )
    
    # Buscar usuário
    response = table.get_item(
        Key={'userId': body['userId']}
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps(response['Item'])
    }
```

**Gerar política**:

```bash
iam-policy-autopilot generate-policies \
  api_handler.py \
  --service-hints dynamodb \
  --pretty
```

**Política gerada**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

### Caso 3: Corrigir AccessDenied em produção

**Cenário**: Lambda em produção retornando erro de permissão.

**Erro no CloudWatch**:

```
User: arn:aws:sts::123456789012:assumed-role/LambdaRole/my-function 
is not authorized to perform: dynamodb:Query 
on resource: arn:aws:dynamodb:us-east-1:123456789012:table/users
```

**Corrigir com IAM Policy Autopilot**:

```bash
iam-policy-autopilot fix-access-denied \
  "User: arn:aws:sts::123456789012:assumed-role/LambdaRole/my-function is not authorized to perform: dynamodb:Query on resource: arn:aws:dynamodb:us-east-1:123456789012:table/users"
```

**Análise**:

```
IAM Policy Autopilot detectou:
- Ação faltando: dynamodb:Query
- Recurso: arn:aws:dynamodb:us-east-1:123456789012:table/users
- Role: LambdaRole

Sugestão de fix:
Adicionar à política da role LambdaRole:

{
  "Effect": "Allow",
  "Action": "dynamodb:Query",
  "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/users"
}

Aplicar fix? (y/n)
```

**Aplicar automaticamente**:

```bash
iam-policy-autopilot fix-access-denied \
  "User: arn:aws:sts::123456789012:assumed-role/LambdaRole/my-function is not authorized to perform: dynamodb:Query on resource: arn:aws:dynamodb:us-east-1:123456789012:table/users" \
  --yes
```

**Tempo economizado**: 10-15 minutos de troubleshooting e deploy.


## Limitações e considerações

### 1. Políticas baseadas em identidade apenas

**O que gera**:
* Identity-based policies (IAM roles, users)

**O que NÃO gera**:
* Resource-based policies (S3 bucket policies, KMS key policies)
* Service Control Policies (SCPs)
* Permission boundaries
* Resource Control Policies (RCPs)

**Exemplo**:

```python
# Código que acessa S3
s3.get_object(Bucket='my-bucket', Key='file.txt')
```

**IAM Policy Autopilot gera**:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "*"
}
```

**Você ainda precisa criar manualmente** (se necessário):

```json
// Bucket policy
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::123456789012:role/LambdaRole"},
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

### 2. Recursos determinados em runtime

**Limitação**: Se o nome do recurso é determinado em runtime, o IAM Policy Autopilot não consegue prever.

**Exemplo**:

```python
# Bucket name vem de variável de ambiente
bucket_name = os.environ['BUCKET_NAME']
s3.get_object(Bucket=bucket_name, Key='file.txt')
```

**Política gerada**:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "*"  // Não sabe qual bucket específico
}
```

**Solução**: Refinar manualmente depois:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-specific-bucket/*"
}
```

### 3. Bibliotecas de terceiros que encapsulam AWS SDK

**Limitação**: Se você usa bibliotecas que encapsulam o AWS SDK, a análise pode não detectar todas as chamadas.

**Exemplo**:

```python
# Biblioteca customizada que encapsula boto3
from my_custom_lib import S3Helper

helper = S3Helper()
helper.upload_file('file.txt')  // IAM Policy Autopilot pode não detectar
```

**Solução**: 
* Use `--explain` para entender o que foi detectado
* Complemente com análise manual
* Ou use AWS SDK diretamente quando possível

### 4. Políticas funcionais, não mínimas

**Importante**: IAM Policy Autopilot prioriza funcionalidade sobre permissões mínimas.

**Exemplo**:

Para `s3.put_object()`, gera:

```json
{
  "Action": [
    "s3:PutObject",
    "kms:GenerateDataKey",  // Incluído preventivamente
    "kms:Decrypt"           // Incluído preventivamente
  ]
}
```

Mesmo que seu bucket não use KMS, as permissões KMS são incluídas "por precaução".

**Por quê**: Garantir que a aplicação funcione no primeiro deploy, independente da configuração.

**Boa prática**: 

1. Use a política gerada para deploy inicial
2. Monitore com IAM Access Analyzer
3. Remova permissões não utilizadas após validação


## Boas práticas

### 1. Use como ponto de partida, não como solução final

```
Fluxo recomendado:

1. Gerar política com IAM Policy Autopilot
2. Deploy inicial
3. Validar funcionalidade
4. Monitorar com IAM Access Analyzer
5. Refinar para least privilege
6. Especificar recursos (trocar * por ARNs específicos)
```

**Exemplo de refinamento**:

```json
// Política inicial (gerada)
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "*"
}

// Política refinada (após validação)
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": [
    "arn:aws:s3:::my-specific-bucket/*",
    "arn:aws:s3:::my-backup-bucket/*"
  ]
}
```

### 2. Use service hints para precisão

```bash
# Menos preciso (pode incluir permissões desnecessárias)
iam-policy-autopilot generate-policies app.py

# Mais preciso (apenas serviços que você usa)
iam-policy-autopilot generate-policies app.py \
  --service-hints s3 dynamodb lambda sqs
```

### 3. Use --explain para entender

```bash
# Ver quais operações geraram cada ação
iam-policy-autopilot generate-policies app.py \
  --explain 's3:*' \
  --pretty
```

**Resultado**:

```
s3:GetObject
  Detectado em: linha 15, s3.get_object(...)
  
s3:PutObject
  Detectado em: linha 23, s3.put_object(...)
  
kms:GenerateDataKey
  Dependência de: s3:PutObject (criptografia)
```

### 4. Integre com IAM Access Analyzer

```
1. Deploy com política gerada pelo IAM Policy Autopilot
2. Rode aplicação por 7-30 dias
3. Use IAM Access Analyzer para identificar permissões não usadas
4. Remova permissões não utilizadas
5. Resultado: Least privilege real
```

**Comando para analisar**:

```bash
# Criar analyzer
aws accessanalyzer create-analyzer \
  --analyzer-name my-analyzer \
  --type ACCOUNT

# Gerar recomendações após 90 dias
aws accessanalyzer get-generated-policy \
  --job-id <job-id>
```

### 5. Versionamento de políticas

```bash
# Salvar política gerada com timestamp
iam-policy-autopilot generate-policies app.py \
  --pretty > policies/policy-$(date +%Y%m%d-%H%M%S).json

# Commitar no Git
git add policies/
git commit -m "feat: add IAM policy for Lambda function"
```

### 6. Automação em CI/CD

```yaml
# .github/workflows/generate-iam-policies.yml
name: Generate IAM Policies

on:
  pull_request:
    paths:
      - 'src/**/*.py'
      - 'src/**/*.go'
      - 'src/**/*.ts'

jobs:
  generate-policies:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install IAM Policy Autopilot
        run: pip install iam-policy-autopilot
      
      - name: Generate policies
        run: |
          iam-policy-autopilot generate-policies \
            src/ \
            --region us-east-1 \
            --account ${{ secrets.AWS_ACCOUNT_ID }} \
            --pretty > generated-policy.json
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: iam-policy
          path: generated-policy.json
      
      - name: Comment on PR
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const policy = fs.readFileSync('generated-policy.json', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Generated IAM Policy\n\`\`\`json\n${policy}\n\`\`\``
            });
```


## Quando usar cada abordagem

### Use CLI quando:

* Trabalhando em pipelines CI/CD
* Gerando políticas em batch para múltiplos arquivos
* Automatizando geração de políticas
* Preferindo controle direto via terminal
* Integrando com scripts existentes

**Exemplo de pipeline**:

```bash
#!/bin/bash
# generate-policies.sh

for file in src/**/*.py; do
  echo "Generating policy for $file"
  iam-policy-autopilot generate-policies "$file" \
    --service-hints s3 dynamodb lambda \
    --pretty > "policies/$(basename $file .py)-policy.json"
done
```

### Use MCP Server quando:

* Trabalhando com assistentes de IA (Claude, Cursor, Cline)
* Desenvolvimento interativo
* Querendo contexto adicional do assistente
* Criando infraestrutura completa (código + IAM + IaC)

### Use Kiro Power quando:

* Usando Kiro como IDE principal
* Querendo experiência mais refinada que MCP
* Querendo validação automática de instalação
* Preferindo instalação com um clique

## Conclusão

Criar políticas IAM com least privilege sempre foi um dos maiores desafios no desenvolvimento AWS. Documentação complexa, dependências cross-service não óbvias, e ciclos lentos de troubleshooting consomem tempo e geram frustração.

O IAM Policy Autopilot resolve esse problema de forma elegante através de análise estática de código. Ao invés de adivinhar permissões ou copiar políticas da internet, você gera políticas válidas e funcionais em segundos, baseadas no código real da sua aplicação.

Mas lembre-se: o IAM Policy Autopilot é um ponto de partida, não uma solução final. Use-o para gerar políticas funcionais rapidamente, depois refine para least privilege usando IAM Access Analyzer e especificando recursos específicos.

O IAM Policy Autopilot não elimina a necessidade de entender IAM, mas remove a fricção do dia a dia. Você ainda precisa revisar políticas e entender segurança, mas agora pode focar nisso ao invés de lutar com sintaxe e troubleshooting.

Quanto antes você adotar, mais tempo economizará.

### Recursos adicionais

* [IAM Policy Autopilot - Blog AWS Security](https://aws.amazon.com/blogs/security/iam-policy-autopilot-an-open-source-tool-that-brings-iam-policy-expertise-to-builders-and-ai-coding-assistants/)
* [IAM Policy Autopilot - GitHub](https://github.com/awslabs/iam-policy-autopilot)
* [IAM Policy Autopilot - Kiro Power](https://github.com/awslabs/iam-policy-autopilot/tree/main/power-iam-policy-autopilot)
* [Anúncio Kiro Power](https://aws.amazon.com/about-aws/whats-new/2026/02/aws-iam-policy-autopilot-kiro-power/)
* [Kiro Powers Documentation](https://kiro.dev/powers)
* [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
