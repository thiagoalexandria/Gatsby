---
image: /assets/img/AWS.png
title: "Innovation Sandbox on AWS: Potencializando Experimentações na Era da IA"
description: Com o avanço acelerado da inteligência artificial, times precisam
  de ambientes seguros para experimentar sem medo de quebrar produção ou
  estourar orçamentos. O Innovation Sandbox on AWS resolve esse desafio ao
  automatizar a criação e reciclagem de contas sandbox com controles de
  segurança, governança e custos integrados. Entenda como essa solução pode
  transformar a cultura de experimentação da sua organização.
date: 2026-08-06
category: aws
background: "#FF9900"
tags:
  - aws
  - sandbox
  - poc
  - innovation
  - sre
  - devops
  - security
categories:
  - aws
  - sandbox
  - poc
  - innovation
  - sre
  - devops
  - security
---
Imagine a seguinte cena: um engenheiro do seu time quer testar o Amazon Bedrock com um novo modelo de IA generativa. Ele precisa de uma conta AWS isolada, com permissões adequadas, controle de gastos e que seja automaticamente limpa depois do experimento. Hoje, quanto tempo esse processo leva na sua organização?

Se a resposta envolve "abrir ticket", "esperar aprovação de 3 gestores", "provisionar conta manualmente" e "torcer para alguém lembrar de limpar depois" — você não está sozinho. E é exatamente esse gargalo que mata a inovação.

Em um cenário onde cada semana surge um novo serviço de IA, um novo modelo, uma nova arquitetura para testar, o tempo entre "tive uma ideia" e "consegui testar" precisa ser mínimo. E é aqui que o Innovation Sandbox on AWS entra.

## O problema real

### Experimentação engessada

A maioria das organizações enfrenta um dilema:

**Opção A: Controle total**
- Processo burocrático para criar contas
- Semanas para provisionar
- Time de plataforma sobrecarregado
- Resultado: ninguém experimenta

**Opção B: Liberdade total**
- Contas compartilhadas sem isolamento
- Sem controle de gastos
- Recursos esquecidos acumulando custo
- Resultado: conta de $50.000 no final do mês

Nenhuma das duas opções funciona. Você precisa de velocidade COM governança.

### O custo da não-experimentação

Com IA generativa avançando na velocidade que avança, não experimentar é mais caro do que experimentar:

- **Bedrock** lança novos modelos quase toda semana
- **SageMaker** evoluiu drasticamente com JumpStart e Canvas
- **Lambda** agora suporta containers maiores para inference
- **ECS/EKS** com GPUs para workloads de ML

Se seu time leva 2 semanas para conseguir um ambiente de teste, vocês estão sempre 2 semanas atrasados em relação ao mercado.

## Innovation Sandbox on AWS: A solução

O Innovation Sandbox on AWS é uma solução open source da AWS que automatiza completamente o ciclo de vida de contas sandbox — do provisionamento à reciclagem.

### Como funciona

O fluxo é elegante na sua simplicidade:

```
Usuário solicita sandbox → Aprovação (opcional) → Conta ativada
                                                        ↓
                                              Usa por X dias ou $Y
                                                        ↓
                                              Limite atingido
                                                        ↓
                                              Limpeza automática (AWS Nuke)
                                                        ↓
                                              Conta reciclada → Pool
```

A solução não cria nem fecha contas AWS. Ela gerencia um **pool de contas existentes**, movendo-as entre Organizational Units (OUs) conforme o ciclo de vida:

```
OU AccountPool
├── Entry       → Contas entrando no pool
├── Available   → Prontas para uso
├── Active      → Em uso (lease ativo)
├── Frozen      → Budget/tempo estourado
├── CleanUp     → Sendo limpas pelo AWS Nuke
├── Quarantine  → Limpeza falhou (intervenção manual)
└── Exit        → Saindo do pool
```

> 📸 **Print sugerida:** Diagrama da arquitetura da solução (tela da documentação AWS ou diagrama customizado mostrando o fluxo entre OUs)

### Arquitetura

A solução é composta por 4 stacks CloudFormation:

| Stack | Função |
|-------|--------|
| **AccountPool** | Cria OUs, SCPs, roles e define regiões permitidas |
| **IDC** | Configura Identity Center (grupos, permission sets, assignments) |
| **Data** | DynamoDB para estado, AppConfig para configurações globais |
| **Compute** | API Gateway, Lambdas, Step Functions, CloudFront (UI), WAF |

Todos os componentes são serverless, resultando em um custo de infraestrutura de apenas **~$65/mês** — independente de quantas contas você gerencia.

> 📸 **Print sugerida:** Console do CloudFormation mostrando os 4 stacks deployados com status CREATE_COMPLETE ou UPDATE_COMPLETE

### O que protege suas contas sandbox

**Service Control Policies (SCPs):**
- Bloqueiam serviços caros/sensíveis automaticamente
- Restringem regiões permitidas
- Impedem alterações em configurações de segurança

**Controles de Budget:**
- Limite máximo configurável por lease (ex: $5.000)
- Alertas antes de atingir o limite
- Freeze automático quando o budget estoura

**Controles de Tempo:**
- Duração máxima configurável (ex: 90 dias)
- Notificações antes da expiração
- Cleanup automático ao expirar

**WAF + IP Allowlist:**
- Acesso à UI restrito por CIDR (VPN corporativa)
- Proteção contra exploits e bots

> 📸 **Print sugerida:** Tela de configuração do AppConfig mostrando os parâmetros de leases (maxBudget, maxDurationHours, maxLeasesPerUser)

## A interface: Self-service para o time

A UI é uma SPA moderna servida via CloudFront. Dependendo do seu papel, você vê funcionalidades diferentes:

### Visão do Usuário

O usuário final tem uma experiência simples:

1. Acessa a URL da solução
2. Faz login via Identity Center (SSO)
3. Aceita os termos de uso
4. Solicita um lease a partir de um template
5. Recebe acesso à conta sandbox via SSO
6. Usa pelo tempo/budget definido
7. Conta é automaticamente limpa e reciclada

> 📸 **Print sugerida:** Tela de login via Identity Center (SSO)

> 📸 **Print sugerida:** Tela de termos de serviço (Terms of Service) que o usuário vê ao solicitar um lease

> 📸 **Print sugerida:** Tela de "Request Lease" mostrando os templates disponíveis e campos de budget/duração

### Visão do Manager

Managers podem:
- Aprovar/rejeitar solicitações de lease
- Visualizar uso de budget dos leases ativos
- Monitorar status das contas do pool

> 📸 **Print sugerida:** Dashboard do Manager mostrando leases ativos e status

### Visão do Admin

Admins têm controle total:
- Criar Lease Templates
- Registrar contas no pool
- Configurar limites globais
- Ativar/desativar modo de manutenção
- Visualizar contas em quarentena

> 📸 **Print sugerida:** Painel de Admin mostrando contas registradas e seus status (Available, Active, Frozen, etc.)

> 📸 **Print sugerida:** Tela de criação de Lease Template

## Por que isso potencializa experimentação com IA

### Cenário 1: Testando novos modelos no Bedrock

Seu time de dados quer avaliar Claude Sonnet 4 vs Llama para um caso de uso específico. Com o Innovation Sandbox:

1. Desenvolvedor solicita lease (30 segundos)
2. Manager aprova (1 minuto)
3. Conta sandbox disponível com acesso via SSO (imediato)
4. Testa modelos, cria protótipos, avalia custos reais
5. Apresenta resultados com dados concretos
6. Lease expira → conta limpa automaticamente

**Tempo total: minutos.** Sem tickets, sem esperas, sem contas esquecidas.

### Cenário 2: PoC de RAG com Knowledge Bases

Um arquiteto quer montar uma PoC de Retrieval Augmented Generation:
- Precisa de S3, Bedrock Knowledge Bases, OpenSearch Serverless
- Budget estimado: $200 em 2 semanas
- Quer testar diferentes chunking strategies

Com a sandbox, ele tem liberdade total dentro dos guardrails. Se algo der errado? A conta é limpa e reciclada. Zero risco para produção.

### Cenário 3: Treinamento e capacitação

Está treinando o time em serviços de IA? Crie um Lease Template específico:
- Budget: $50
- Duração: 5 dias
- Um lease por pessoa

Cada membro do time ganha sua própria conta isolada para aprender, errar e experimentar.

> 📸 **Print sugerida:** Lease Template configurado para treinamento com valores de budget e duração menores

### Cenário 4: Hackathons internos

Innovation Sandbox é perfeito para hackathons:
- Provisione 10 contas sandbox em minutos
- Cada time tem isolamento total
- Ninguém interfere no trabalho do outro
- Budget controlado (sem sustos na fatura)
- Após o hackathon: tudo limpo automaticamente

## Deploy e configuração

### Pré-requisitos

Antes de deployar, você precisa garantir:

- AWS Organization com SCPs habilitadas
- Identity Center configurado (pode ser delegated admin)
- StackSets trusted access habilitado
- RAM sharing habilitado
- Cost Explorer ativo (leva 24h para ativar)
- **Lambda concurrent quota ≥ 1000** (solicitar com antecedência!)
- Contas AWS já criadas para o pool

> 📸 **Print sugerida:** Console do AWS Organizations mostrando a estrutura de OUs criada pela solução (Entry, Available, Active, Frozen, CleanUp, Quarantine, Exit)

### Deploy dos stacks

O deploy segue uma ordem obrigatória:

```bash
# 1. AccountPool (Management Account)
aws cloudformation create-stack \
  --stack-name LabPoolAccountPool \
  --template-url https://solutions-reference.s3.amazonaws.com/innovation-sandbox-on-aws/latest/InnovationSandbox-AccountPool.template \
  --parameters ParameterKey=Namespace,ParameterValue=labpool \
               ParameterKey=HubAccountId,ParameterValue=<ACCOUNT_ID> \
               ParameterKey=ParentOuId,ParameterValue=<OU_ID> \
               ParameterKey=IsbManagedRegions,ParameterValue=us-east-1 \
  --capabilities CAPABILITY_NAMED_IAM

# 2. IDC
# 3. Data
# 4. Compute (upload template pro S3 antes - é >250KB)
```

> 📸 **Print sugerida:** Stack do CloudFormation durante criação mostrando os recursos sendo provisionados

### Pós-deploy: Configuração do Identity Center

A parte mais importante do pós-deploy é a configuração SAML:

1. Criar aplicação SAML 2.0 no Identity Center
2. Configurar ACS URL apontando para o CloudFront
3. Mapear atributo Subject → email
4. Atribuir grupos (Admins, Managers, Users) à aplicação
5. Upload do certificado SAML no Secrets Manager

> 📸 **Print sugerida:** Configuração da aplicação SAML no Identity Center mostrando o ACS URL e audience

> 📸 **Print sugerida:** Tela de attribute mapping no Identity Center (Subject → ${user:email})

### Adicionando contas ao pool

Após o deploy, você precisa registrar contas:

1. Mova as contas para a sub-OU **Entry** (não a OU pai!)
2. Na UI, vá em Admin → as contas aparecem como "Unregistered"
3. Registre pela interface

> 📸 **Print sugerida:** UI mostrando contas "Unregistered" prontas para serem registradas

> 📸 **Print sugerida:** UI mostrando contas já registradas no status "Available"

## Customização

### Personalização da UI

A UI é uma SPA estática no S3. Você pode personalizar:
- Cores (substituir palette da AWS pela da sua empresa)
- Logo no header
- Favicon
- Título e meta tags
- Termos de serviço (texto em português!)

```bash
# Exemplo: substituindo cores no CSS
sed -i 's/#006ce0/#SUA_COR_PRIMARIA/g' assets/index-*.css
sed -i 's/#004a9e/#SUA_COR_SECUNDARIA/g' assets/index-*.css

# Upload e invalidação de cache
aws s3 sync . s3://<bucket-frontend>/
aws cloudfront create-invalidation --distribution-id <ID> --paths "/*"
```

> 📸 **Print sugerida:** UI customizada com a identidade visual da sua empresa (antes e depois)

### Termos de serviço em português

No AppConfig, você configura termos de uso que aparecem ao solicitar um lease:

```yaml
termsOfService: |
  Os usuários que utilizam uma conta sandbox NÃO devem:
  * Tentar acessar dados para os quais não possuem autorização
  * Utilizar o ambiente para casos de uso não aprovados
  * Armazenar dados corporativos não aprovados
  * Utilizar senhas estáticas ou de ambientes produtivos
  * Alterar cotas fora do processo estabelecido
  
  Importante: Todos os recursos serão permanentemente excluídos
  quando o limite de budget ou tempo for atingido.
```

> 📸 **Print sugerida:** Tela do AppConfig com a Global Config mostrando o YAML dos termos de serviço

## Operação e manutenção

### Atualizações

A solução recebe updates frequentes. O processo de atualização é:

1. Ativar `maintenanceMode: true` no AppConfig
2. Atualizar stacks na ordem: AccountPool → IDC → Data → Compute
3. Desativar maintenance mode
4. Reaplicar customizações da UI (se houver)

```bash
# Exemplo: update do stack AccountPool para nova versão
aws cloudformation update-stack \
  --stack-name LabPoolAccountPool \
  --template-url https://solutions-reference.s3.amazonaws.com/innovation-sandbox-on-aws/latest/InnovationSandbox-AccountPool.template \
  --parameters ParameterKey=Namespace,UsePreviousValue=true \
               ParameterKey=HubAccountId,UsePreviousValue=true \
               ParameterKey=ParentOuId,UsePreviousValue=true \
               ParameterKey=IsbManagedRegions,UsePreviousValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

**Dica:** Salve um script de customização da UI para reaplicar após cada upgrade.

### Monitoramento

Pontos importantes para monitorar:
- Contas em **Quarantine** (limpeza falhou — requer intervenção manual)
- Leases próximos de expirar
- Budget consumption por lease
- Erros no Step Functions de cleanup

> 📸 **Print sugerida:** Step Functions mostrando o fluxo de cleanup de uma conta (execução bem-sucedida)

## Custo da solução

| Componente | Custo estimado |
|-----------|---------------|
| Infraestrutura da solução | ~$65/mês |
| Contas sandbox (uso real) | Variável (controlado por budget) |

A infraestrutura é quase irrelevante em termos de custo. O valor real está no controle: ao invés de uma conta com $50.000 de recursos esquecidos, você tem leases controlados com budget máximo e limpeza automática.

## Conclusão

O Innovation Sandbox on AWS transforma a experimentação de um processo burocrático e arriscado em algo ágil, seguro e governado. Em um momento onde a IA generativa está redefinindo como construímos software, a capacidade de experimentar rapidamente não é luxo — é necessidade competitiva.

Com essa solução, seu time pode:
- Testar novos modelos de IA em minutos, não semanas
- Fazer PoCs com dados reais em ambientes isolados
- Capacitar pessoas sem risco para produção
- Manter governança sem sacrificar velocidade

E tudo isso com um custo de infraestrutura de $65/mês.

Se sua organização ainda depende de processos manuais para provisionar ambientes de teste, o Innovation Sandbox é o caminho mais curto entre uma ideia e um experimento rodando na nuvem.

*PS: Se você já perdeu uma noite limpando recursos que alguém esqueceu em uma conta compartilhada, meus sentimentos. Mas agora você sabe que existe uma solução melhor.*
