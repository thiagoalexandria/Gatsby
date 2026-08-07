---
image: /assets/img/AWS.png
title: Estruturando Multi-Account do Zero
description: Estruturar um ambiente multi-account na AWS é um passo fundamental
  para garantir governança, segurança e escalabilidade desde o início da jornada
  em nuvem. Neste artigo, você vai aprender como planejar e organizar uma
  arquitetura multi-conta do zero, utilizando boas práticas, definição de
  unidades organizacionais, políticas de acesso e estratégias de segregação de
  ambientes para criar uma base sólida, segura e preparada para crescimento
  sustentável.
date: 2026-03-13
category: aws
background: "#FF9900"
tags:
  - AWS
  - MULTIACCOUNT
  - AWSORGANIZATIONS
  - CLOUDCOMPUTING
  - GOVERNANCAEMNUVEM
  - CLOUDSECURITY
  - ARQUITETURACLOUD
  - INFRAESTRUTURAEMNUVEM
  - LANDINGZONE
  - FINOPS
  - SEGURANCAEMNUVEM
  - ORGANIZACAOAWS
  - AMBIENTEDEMULTICONTAS
categories:
  - AWS
  - MULTIACCOUNT
  - AWSORGANIZATIONS
  - CLOUDCOMPUTING
  - GOVERNANCAEMNUVEM
  - CLOUDSECURITY
  - ARQUITETURACLOUD
  - INFRAESTRUTURAEMNUVEM
  - LANDINGZONE
  - FINOPS
  - SEGURANCAEMNUVEM
  - ORGANIZACAOAWS
  - AMBIENTEDEMULTICONTAS
---
Adotar uma estratégia multi-account na AWS não é apenas uma recomendação de boas práticas é uma decisão estratégica que impacta diretamente a segurança, a governança e o controle financeiro da organização. 

Com o apoio do AWS Organizations, é possível estruturar contas separadas por ambiente, equipe ou domínio de negócio, aplicando políticas centralizadas e garantindo isolamento adequado de recursos. Hoje, vamos explorar como estruturar esse modelo desde o zero, evitando erros comuns e preparando sua arquitetura para escalar com eficiência.

Neste artigo, você vai aprender:

* Por que multi-account é essencial para ambientes corporativos
* Como habilitar e configurar AWS Organizations
* Estrutura de Organization Units (OUs) e quando usar cada uma
* Quando faz sentido criar contas separadas
* Como configurar contas delegadas para serviços centralizados
* Implementação de rede centralizada com Transit Gateway
* Gerenciamento de acessos com AWS IAM Identity Center
* Service Control Policies (SCPs) para governança
* Automação com AWS Control Tower e Account Factory

No final, você terá uma arquitetura de referência completa e pronta para produção.

## Por que multi-account?

Muitas organizações começam com uma única conta AWS e, conforme crescem, enfrentam problemas de:

* **Isolamento de segurança**: Um incidente em desenvolvimento pode afetar produção
* **Controle de custos**: Difícil separar gastos por projeto ou equipe
* **Limites de serviço**: Service quotas compartilhados entre todos os ambientes
* **Compliance**: Requisitos regulatórios exigem segregação de dados
* **Blast radius**: Um erro pode impactar toda a organização

### Benefícios do multi-account

1. **Isolamento de segurança**
   * Cada conta tem seu próprio conjunto de credenciais
   * Comprometimento de uma conta não afeta outras
   * Políticas de segurança específicas por ambiente

2. **Controle financeiro**
   * Custos separados por conta/projeto/equipe
   * Budgets e alertas específicos
   * Chargeback simplificado

3. **Limites independentes**
   * Service quotas por conta
   * Mais recursos disponíveis no total
   * Menos contenção entre ambientes

4. **Governança centralizada**
   * Políticas aplicadas via AWS Organizations
   * Auditoria e compliance simplificados
   * Controle de acesso unificado

5. **Flexibilidade operacional**
   * Times autônomos dentro de guardrails
   * Experimentação segura em contas sandbox
   * Ambientes isolados para testes


## Arquitetura de referência

Vamos construir uma estrutura multi-account completa seguindo as melhores práticas da AWS.

### Estrutura de Organization Units (OUs)

```
Root
├── Security OU
│   ├── Security Tooling Account (logs, GuardDuty, Security Hub)
│   └── Audit Account (CloudTrail, Config, compliance)
├── Infrastructure OU
│   ├── Network Account (Transit Gateway, VPCs compartilhadas)
│   ├── Shared Services Account (DNS, Active Directory, ferramentas)
│   └── Backup Account (AWS Backup centralizado)
├── Workloads OU
│   ├── Production OU
│   │   ├── App-A Production
│   │   └── App-B Production
│   ├── Non-Production OU
│   │   ├── Development
│   │   ├── Staging
│   │   └── QA
│   └── Sandbox OU
│       ├── Sandbox-Team-A
│       └── Sandbox-Team-B
├── Suspended OU
│   └── (Contas desativadas temporariamente)
└── Management Account (root, não usar para workloads)
```

### Quando criar contas separadas

| Cenário | Criar conta separada? | Motivo |
|---------|----------------------|--------|
| Ambiente de produção | ✅ Sim | Isolamento crítico |
| Ambiente de desenvolvimento | ✅ Sim | Evitar impacto em produção |
| Cada aplicação | ⚠️ Depende | Se aplicações são independentes |
| Cada equipe | ⚠️ Depende | Se equipes são autônomas |
| Cada cliente (SaaS) | ✅ Sim | Isolamento de dados |
| Sandbox/experimentação | ✅ Sim | Liberdade sem riscos |
| Disaster Recovery | ✅ Sim | Isolamento geográfico |
| Logs e auditoria | ✅ Sim | Proteção contra adulteração |

**Regra geral**: Crie contas separadas quando precisar de isolamento de segurança, controle de custos independente ou limites de serviço dedicados.


## Hands-on: Implementando a estrutura

### Pré-requisitos

* Conta AWS (será a Management Account)
* Acesso root ou permissões administrativas
* E-mails únicos para cada conta (use aliases: email+dev@company.com)
* Terraform instalado (opcional, mas recomendado)

### 1. Habilitando AWS Organizations

**Via Console AWS**:

1. Acesse o serviço AWS Organizations
2. Clique em "Create organization"
3. Escolha "Enable all features" (não apenas consolidated billing)
4. Confirme via e-mail

**Via AWS CLI**:

```bash
# Criar organização
aws organizations create-organization --feature-set ALL

# Verificar status
aws organizations describe-organization
```

**Via Terraform**:

```terraform
# Arquivo: organizations.tf

provider "aws" {
  region = "us-east-1"
}

resource "aws_organizations_organization" "main" {
  feature_set = "ALL"

  aws_service_access_principals = [
    "cloudtrail.amazonaws.com",
    "config.amazonaws.com",
    "guardduty.amazonaws.com",
    "securityhub.amazonaws.com",
    "sso.amazonaws.com",
    "ram.amazonaws.com"
  ]

  enabled_policy_types = [
    "SERVICE_CONTROL_POLICY",
    "TAG_POLICY",
    "BACKUP_POLICY"
  ]
}
```


### 2. Criando Organization Units

**Via Terraform**:

```terraform
# OUs principais
resource "aws_organizations_organizational_unit" "security" {
  name      = "Security"
  parent_id = aws_organizations_organization.main.roots[0].id
}

resource "aws_organizations_organizational_unit" "infrastructure" {
  name      = "Infrastructure"
  parent_id = aws_organizations_organization.main.roots[0].id
}

resource "aws_organizations_organizational_unit" "workloads" {
  name      = "Workloads"
  parent_id = aws_organizations_organization.main.roots[0].id
}

resource "aws_organizations_organizational_unit" "suspended" {
  name      = "Suspended"
  parent_id = aws_organizations_organization.main.roots[0].id
}

# OUs aninhadas em Workloads
resource "aws_organizations_organizational_unit" "production" {
  name      = "Production"
  parent_id = aws_organizations_organizational_unit.workloads.id
}

resource "aws_organizations_organizational_unit" "non_production" {
  name      = "Non-Production"
  parent_id = aws_organizations_organizational_unit.workloads.id
}

resource "aws_organizations_organizational_unit" "sandbox" {
  name      = "Sandbox"
  parent_id = aws_organizations_organizational_unit.workloads.id
}
```

**Via AWS CLI**:

```bash
# Obter ID da root
ROOT_ID=$(aws organizations list-roots --query 'Roots[0].Id' --output text)

# Criar OUs
aws organizations create-organizational-unit \
  --parent-id $ROOT_ID \
  --name Security

aws organizations create-organizational-unit \
  --parent-id $ROOT_ID \
  --name Infrastructure

aws organizations create-organizational-unit \
  --parent-id $ROOT_ID \
  --name Workloads
```


### 3. Criando contas de forma escalável

**IMPORTANTE**: Não recomendo criar contas diretamente com `aws_organizations_account` do Terraform. Para ambientes corporativos, use o **Account Factory for Terraform (AFT)**, que oferece:

* Provisionamento padronizado e automatizado
* Customizações aplicadas automaticamente em novas contas
* Baseline de segurança consistente
* Versionamento e auditoria de mudanças
* Integração com pipelines de CI/CD

Para entender como implementar AFT corretamente, leia o artigo completo:
👉 [Account Factory for Terraform: A Chave para o Provisionamento Escalável de Contas AWS](https://thiagoalexandria.com.br/account-factory-for-terraform-a-chave-para-o-provisionamento-escalavel-de-contas-aws/)

### Criação manual (apenas para setup inicial)

Para as primeiras contas (Security, Audit, Network), você pode criar manualmente via console:

**Via Console AWS**:

1. Acesse AWS Organizations
2. Clique em "Add an AWS account" → "Create an AWS account"
3. Preencha:
   - Account name: Security-Tooling
   - Email: aws-security-tooling@company.com
   - IAM role name: OrganizationAccountAccessRole
4. Clique em "Create AWS account"
5. Mova a conta para a OU correta

**Via AWS CLI**:

```bash
# Criar conta de Security Tooling
aws organizations create-account \
  --email aws-security-tooling@company.com \
  --account-name "Security-Tooling" \
  --role-name OrganizationAccountAccessRole

# Obter ID da conta criada
ACCOUNT_ID=$(aws organizations list-accounts \
  --query 'Accounts[?Name==`Security-Tooling`].Id' \
  --output text)

# Mover para OU Security
SECURITY_OU_ID=$(aws organizations list-organizational-units-for-parent \
  --parent-id $ROOT_ID \
  --query 'OrganizationalUnits[?Name==`Security`].Id' \
  --output text)

aws organizations move-account \
  --account-id $ACCOUNT_ID \
  --source-parent-id $ROOT_ID \
  --destination-parent-id $SECURITY_OU_ID
```

### Contas essenciais para começar

Crie estas contas primeiro (manualmente ou via AFT):

1. **Security-Tooling**: GuardDuty, Security Hub, Macie
2. **Audit**: CloudTrail, AWS Config, logs centralizados
3. **Network**: Transit Gateway, VPCs compartilhadas
4. **Shared-Services**: DNS, Active Directory, ferramentas comuns

Depois de configurar AFT, use-o para criar todas as outras contas (Production, Development, Sandbox, etc.)
```


### 4. Service Control Policies (SCPs)

SCPs são políticas que definem o máximo de permissões que podem ser concedidas em uma conta ou OU.

**SCP: Deny root user**

```terraform
resource "aws_organizations_policy" "deny_root_user" {
  name        = "DenyRootUser"
  description = "Deny all actions by root user"
  type        = "SERVICE_CONTROL_POLICY"

  content = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Deny"
      Action = "*"
      Resource = "*"
      Condition = {
        StringLike = {
          "aws:PrincipalArn" = "arn:aws:iam::*:root"
        }
      }
    }]
  })
}

# Aplicar a todas as contas exceto Management
resource "aws_organizations_policy_attachment" "deny_root_workloads" {
  policy_id = aws_organizations_policy.deny_root_user.id
  target_id = aws_organizations_organizational_unit.workloads.id
}
```

**SCP: Deny region restriction**

```terraform
resource "aws_organizations_policy" "region_restriction" {
  name        = "RegionRestriction"
  description = "Restrict operations to specific regions"
  type        = "SERVICE_CONTROL_POLICY"

  content = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Deny"
      Action = "*"
      Resource = "*"
      Condition = {
        StringNotEquals = {
          "aws:RequestedRegion" = [
            "us-east-1",
            "us-west-2",
            "sa-east-1"
          ]
        }
      }
    }]
  })
}

resource "aws_organizations_policy_attachment" "region_restriction_workloads" {
  policy_id = aws_organizations_policy.region_restriction.id
  target_id = aws_organizations_organizational_unit.workloads.id
}
```


**SCP: Require encryption**

```terraform
resource "aws_organizations_policy" "require_encryption" {
  name        = "RequireEncryption"
  description = "Require encryption for EBS volumes and RDS instances"
  type        = "SERVICE_CONTROL_POLICY"

  content = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "DenyUnencryptedEBS"
        Effect = "Deny"
        Action = [
          "ec2:RunInstances"
        ]
        Resource = "arn:aws:ec2:*:*:volume/*"
        Condition = {
          Bool = {
            "ec2:Encrypted" = "false"
          }
        }
      },
      {
        Sid    = "DenyUnencryptedRDS"
        Effect = "Deny"
        Action = [
          "rds:CreateDBInstance",
          "rds:CreateDBCluster"
        ]
        Resource = "*"
        Condition = {
          Bool = {
            "rds:StorageEncrypted" = "false"
          }
        }
      }
    ]
  })
}

resource "aws_organizations_policy_attachment" "require_encryption_production" {
  policy_id = aws_organizations_policy.require_encryption.id
  target_id = aws_organizations_organizational_unit.production.id
}
```

**SCP: Sandbox restrictions**

```terraform
resource "aws_organizations_policy" "sandbox_restrictions" {
  name        = "SandboxRestrictions"
  description = "Limit expensive services in sandbox"
  type        = "SERVICE_CONTROL_POLICY"

  content = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Deny"
      Action = [
        "ec2:RunInstances"
      ]
      Resource = "arn:aws:ec2:*:*:instance/*"
      Condition = {
        StringNotLike = {
          "ec2:InstanceType" = [
            "t2.*",
            "t3.*",
            "t3a.*"
          ]
        }
      }
    }]
  })
}

resource "aws_organizations_policy_attachment" "sandbox_restrictions" {
  policy_id = aws_organizations_policy.sandbox_restrictions.id
  target_id = aws_organizations_organizational_unit.sandbox.id
}
```


## Contas delegadas (Delegated Administrator)

Algumas funcionalidades de segurança e governança devem ser gerenciadas de contas específicas, não da Management Account.

### Habilitando contas delegadas

**Via Terraform**:

```terraform
# Delegar GuardDuty para Security Tooling
resource "aws_guardduty_organization_admin_account" "main" {
  admin_account_id = aws_organizations_account.security_tooling.id
}

# Delegar Security Hub para Security Tooling
resource "aws_securityhub_organization_admin_account" "main" {
  admin_account_id = aws_organizations_account.security_tooling.id
}

# Delegar CloudTrail para Audit
resource "aws_cloudtrail_organization_delegated_admin_account" "main" {
  account_id = aws_organizations_account.audit.id
}

# Delegar AWS Config para Audit
resource "aws_config_organization_managed_rule" "main" {
  depends_on = [aws_organizations_account.audit]
  
  name     = "required-tags"
  rule_identifier = "REQUIRED_TAGS"
}

# Delegar Firewall Manager para Security Tooling
resource "aws_fms_admin_account" "main" {
  account_id = aws_organizations_account.security_tooling.id
}
```

**Via AWS CLI**:

```bash
# Delegar GuardDuty
aws guardduty enable-organization-admin-account \
  --admin-account-id 123456789012

# Delegar Security Hub
aws securityhub enable-organization-admin-account \
  --admin-account-id 123456789012

# Delegar Macie
aws macie2 enable-organization-admin-account \
  --admin-account-id 123456789012
```

### Serviços que suportam delegação

| Serviço | Conta recomendada | Propósito |
|---------|------------------|-----------|
| GuardDuty | Security Tooling | Detecção de ameaças |
| Security Hub | Security Tooling | Agregação de findings |
| Macie | Security Tooling | Proteção de dados sensíveis |
| CloudTrail | Audit | Logs de auditoria |
| AWS Config | Audit | Compliance e configuração |
| Firewall Manager | Security Tooling | Políticas de firewall |
| Access Analyzer | Security Tooling | Análise de permissões |


## Rede centralizada com Transit Gateway

Uma arquitetura de rede centralizada permite conectividade entre contas de forma segura e escalável.

### Arquitetura de rede

```
┌─────────────────────────────────────────────────────────┐
│              Network Account (Hub)                       │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         AWS Transit Gateway                       │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐ │  │
│  │  │ Route Table│  │ Route Table│  │ Route Table│ │  │
│  │  │  Prod      │  │  Non-Prod  │  │  Shared    │ │  │
│  │  └────────────┘  └────────────┘  └────────────┘ │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         Inspection VPC (Firewall)                 │  │
│  │  ┌────────────┐  ┌────────────┐                  │  │
│  │  │ Network    │  │ Internet   │                  │  │
│  │  │ Firewall   │  │ Gateway    │                  │  │
│  │  └────────────┘  └────────────┘                  │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼──────┐  ┌───────▼──────┐  ┌──────▼───────┐
│ Production   │  │ Development  │  │ Shared       │
│ Account      │  │ Account      │  │ Services     │
│              │  │              │  │ Account      │
│ VPC          │  │ VPC          │  │ VPC          │
│ 10.0.0.0/16  │  │ 10.1.0.0/16  │  │ 10.2.0.0/16  │
└──────────────┘  └──────────────┘  └──────────────┘
```

### Implementação do Transit Gateway

**Arquivo `transit_gateway.tf` (na conta Network)**:

```terraform
# Transit Gateway
resource "aws_ec2_transit_gateway" "main" {
  description                     = "Main Transit Gateway"
  default_route_table_association = "disable"
  default_route_table_propagation = "disable"
  dns_support                     = "enable"
  vpn_ecmp_support               = "enable"

  tags = {
    Name = "main-tgw"
  }
}

# Route Table para Production
resource "aws_ec2_transit_gateway_route_table" "production" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id

  tags = {
    Name = "production-rt"
  }
}

# Route Table para Non-Production
resource "aws_ec2_transit_gateway_route_table" "non_production" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id

  tags = {
    Name = "non-production-rt"
  }
}

# Route Table para Shared Services
resource "aws_ec2_transit_gateway_route_table" "shared" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id

  tags = {
    Name = "shared-rt"
  }
}
```


```terraform
# Compartilhar Transit Gateway com outras contas via RAM
resource "aws_ram_resource_share" "tgw" {
  name                      = "transit-gateway-share"
  allow_external_principals = false

  tags = {
    Name = "tgw-share"
  }
}

resource "aws_ram_resource_association" "tgw" {
  resource_arn       = aws_ec2_transit_gateway.main.arn
  resource_share_arn = aws_ram_resource_share.tgw.arn
}

# Compartilhar com Organization
resource "aws_ram_principal_association" "organization" {
  principal          = data.aws_organizations_organization.main.arn
  resource_share_arn = aws_ram_resource_share.tgw.arn
}

data "aws_organizations_organization" "main" {}
```

### Attachment de VPCs ao Transit Gateway

**Arquivo `tgw_attachments.tf` (em cada conta de workload)**:

```terraform
# Production Account
resource "aws_ec2_transit_gateway_vpc_attachment" "production" {
  subnet_ids         = aws_subnet.private[*].id
  transit_gateway_id = data.aws_ec2_transit_gateway.shared.id
  vpc_id             = aws_vpc.main.id

  transit_gateway_default_route_table_association = false
  transit_gateway_default_route_table_propagation = false

  tags = {
    Name = "production-attachment"
  }
}

# Associar ao route table correto
resource "aws_ec2_transit_gateway_route_table_association" "production" {
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.production.id
  transit_gateway_route_table_id = data.aws_ec2_transit_gateway_route_table.production.id
}

# Propagar rotas
resource "aws_ec2_transit_gateway_route_table_propagation" "production_to_shared" {
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.production.id
  transit_gateway_route_table_id = data.aws_ec2_transit_gateway_route_table.shared.id
}

data "aws_ec2_transit_gateway" "shared" {
  filter {
    name   = "tag:Name"
    values = ["main-tgw"]
  }
}

data "aws_ec2_transit_gateway_route_table" "production" {
  filter {
    name   = "tag:Name"
    values = ["production-rt"]
  }
}

data "aws_ec2_transit_gateway_route_table" "shared" {
  filter {
    name   = "tag:Name"
    values = ["shared-rt"]
  }
}
```


### Regras de roteamento

**Isolamento entre ambientes**:

```terraform
# Production pode acessar Shared Services, mas não Development
resource "aws_ec2_transit_gateway_route" "prod_to_shared" {
  destination_cidr_block         = "10.2.0.0/16"  # Shared Services CIDR
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.shared.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.production.id
}

# Development pode acessar Shared Services, mas não Production
resource "aws_ec2_transit_gateway_route" "dev_to_shared" {
  destination_cidr_block         = "10.2.0.0/16"  # Shared Services CIDR
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.shared.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.non_production.id
}

# Shared Services pode acessar todos
resource "aws_ec2_transit_gateway_route" "shared_to_prod" {
  destination_cidr_block         = "10.0.0.0/16"  # Production CIDR
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.production.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.shared.id
}

resource "aws_ec2_transit_gateway_route" "shared_to_dev" {
  destination_cidr_block         = "10.1.0.0/16"  # Development CIDR
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.development.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.shared.id
}
```

### Inspection VPC (Firewall centralizado)

```terraform
# VPC para inspeção de tráfego
resource "aws_vpc" "inspection" {
  cidr_block           = "10.255.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "inspection-vpc"
  }
}

# Network Firewall para inspeção
resource "aws_networkfirewall_firewall" "inspection" {
  name                = "inspection-firewall"
  firewall_policy_arn = aws_networkfirewall_firewall_policy.main.arn
  vpc_id              = aws_vpc.inspection.id

  subnet_mapping {
    subnet_id = aws_subnet.firewall_az1.id
  }

  subnet_mapping {
    subnet_id = aws_subnet.firewall_az2.id
  }

  tags = {
    Name = "inspection-firewall"
  }
}

# Rota padrão via Inspection VPC
resource "aws_ec2_transit_gateway_route" "default_via_inspection" {
  destination_cidr_block         = "0.0.0.0/0"
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.inspection.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.production.id
}
```


## AWS IAM Identity Center (SSO)

Gerenciar acessos em múltiplas contas sem Identity Center é impraticável. Vamos configurar SSO centralizado.

### Habilitando Identity Center

**Via Console**:
1. Acesse AWS IAM Identity Center
2. Clique em "Enable"
3. Escolha a região (us-east-1 recomendado)
4. Configure o identity source (AWS Directory ou external IdP)

**Via Terraform**:

```terraform
# Habilitar Identity Center
resource "aws_ssoadmin_instances" "main" {}

# Permission Set para Administradores
resource "aws_ssoadmin_permission_set" "admin" {
  name             = "AdministratorAccess"
  description      = "Full administrator access"
  instance_arn     = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  session_duration = "PT8H"

  tags = {
    Name = "AdministratorAccess"
  }
}

resource "aws_ssoadmin_managed_policy_attachment" "admin" {
  instance_arn       = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  permission_set_arn = aws_ssoadmin_permission_set.admin.arn
  managed_policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
}

# Permission Set para Desenvolvedores
resource "aws_ssoadmin_permission_set" "developer" {
  name             = "DeveloperAccess"
  description      = "Developer access with restrictions"
  instance_arn     = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  session_duration = "PT8H"

  tags = {
    Name = "DeveloperAccess"
  }
}

resource "aws_ssoadmin_permission_set_inline_policy" "developer" {
  instance_arn       = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  permission_set_arn = aws_ssoadmin_permission_set.developer.arn

  inline_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "ec2:*",
          "s3:*",
          "lambda:*",
          "dynamodb:*",
          "rds:*",
          "cloudwatch:*",
          "logs:*"
        ]
        Resource = "*"
      },
      {
        Effect = "Deny"
        Action = [
          "ec2:DeleteVpc",
          "ec2:DeleteSubnet",
          "iam:*",
          "organizations:*"
        ]
        Resource = "*"
      }
    ]
  })
}
```


```terraform
# Permission Set para Read-Only
resource "aws_ssoadmin_permission_set" "readonly" {
  name             = "ReadOnlyAccess"
  description      = "Read-only access to all services"
  instance_arn     = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  session_duration = "PT4H"

  tags = {
    Name = "ReadOnlyAccess"
  }
}

resource "aws_ssoadmin_managed_policy_attachment" "readonly" {
  instance_arn       = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  permission_set_arn = aws_ssoadmin_permission_set.readonly.arn
  managed_policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess"
}

data "aws_ssoadmin_instances" "main" {}
```

### Atribuindo permissões a usuários/grupos

```terraform
# Criar grupo de administradores
resource "aws_identitystore_group" "admins" {
  identity_store_id = tolist(data.aws_ssoadmin_instances.main.identity_store_ids)[0]
  display_name      = "Administrators"
  description       = "Administrator group"
}

# Criar grupo de desenvolvedores
resource "aws_identitystore_group" "developers" {
  identity_store_id = tolist(data.aws_ssoadmin_instances.main.identity_store_ids)[0]
  display_name      = "Developers"
  description       = "Developer group"
}

# Atribuir AdministratorAccess ao grupo Admins na conta Production
resource "aws_ssoadmin_account_assignment" "admin_production" {
  instance_arn       = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  permission_set_arn = aws_ssoadmin_permission_set.admin.arn

  principal_id   = aws_identitystore_group.admins.group_id
  principal_type = "GROUP"

  target_id   = aws_organizations_account.production.id
  target_type = "AWS_ACCOUNT"
}

# Atribuir DeveloperAccess ao grupo Developers na conta Development
resource "aws_ssoadmin_account_assignment" "developer_development" {
  instance_arn       = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  permission_set_arn = aws_ssoadmin_permission_set.developer.arn

  principal_id   = aws_identitystore_group.developers.group_id
  principal_type = "GROUP"

  target_id   = aws_organizations_account.development.id
  target_type = "AWS_ACCOUNT"
}

# Atribuir ReadOnlyAccess ao grupo Developers na conta Production
resource "aws_ssoadmin_account_assignment" "developer_production_readonly" {
  instance_arn       = tolist(data.aws_ssoadmin_instances.main.arns)[0]
  permission_set_arn = aws_ssoadmin_permission_set.readonly.arn

  principal_id   = aws_identitystore_group.developers.group_id
  principal_type = "GROUP"

  target_id   = aws_organizations_account.production.id
  target_type = "AWS_ACCOUNT"
}
```


### Integração com IdP externo (Okta, Azure AD, Google Workspace)

```terraform
# Configurar SAML 2.0 com IdP externo
resource "aws_ssoadmin_instance_access_control_attributes" "main" {
  instance_arn = tolist(data.aws_ssoadmin_instances.main.arns)[0]

  attribute {
    key = "email"
    value {
      source = ["$${path:userName}"]
    }
  }

  attribute {
    key = "department"
    value {
      source = ["$${path:enterprise.department}"]
    }
  }
}
```

**Configuração manual no Console**:
1. IAM Identity Center → Settings → Identity source
2. Escolha "External identity provider"
3. Faça upload do metadata XML do seu IdP
4. Configure attribute mappings
5. Teste a integração

### Acesso via CLI com SSO

Depois de configurar, usuários podem acessar via CLI:

```bash
# Configurar perfil SSO
aws configure sso

# Informações necessárias:
# SSO start URL: https://d-xxxxxxxxxx.awsapps.com/start
# SSO Region: us-east-1
# Account ID: 123456789012
# Role name: AdministratorAccess

# Login
aws sso login --profile production-admin

# Usar o perfil
aws s3 ls --profile production-admin
```

**Arquivo `~/.aws/config`**:

```ini
[profile production-admin]
sso_start_url = https://d-xxxxxxxxxx.awsapps.com/start
sso_region = us-east-1
sso_account_id = 123456789012
sso_role_name = AdministratorAccess
region = us-east-1
output = json

[profile development-dev]
sso_start_url = https://d-xxxxxxxxxx.awsapps.com/start
sso_region = us-east-1
sso_account_id = 987654321098
sso_role_name = DeveloperAccess
region = us-east-1
output = json
```


## Logs e auditoria centralizados

Configure CloudTrail e AWS Config para todas as contas.

### CloudTrail organizacional

**Arquivo `cloudtrail.tf` (na conta Audit)**:

```terraform
# S3 Bucket para CloudTrail logs
resource "aws_s3_bucket" "cloudtrail" {
  bucket = "organization-cloudtrail-${data.aws_caller_identity.current.account_id}"

  tags = {
    Name = "organization-cloudtrail"
  }
}

resource "aws_s3_bucket_policy" "cloudtrail" {
  bucket = aws_s3_bucket.cloudtrail.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "AWSCloudTrailAclCheck"
        Effect = "Allow"
        Principal = {
          Service = "cloudtrail.amazonaws.com"
        }
        Action   = "s3:GetBucketAcl"
        Resource = aws_s3_bucket.cloudtrail.arn
      },
      {
        Sid    = "AWSCloudTrailWrite"
        Effect = "Allow"
        Principal = {
          Service = "cloudtrail.amazonaws.com"
        }
        Action   = "s3:PutObject"
        Resource = "${aws_s3_bucket.cloudtrail.arn}/*"
        Condition = {
          StringEquals = {
            "s3:x-amz-acl" = "bucket-owner-full-control"
          }
        }
      }
    ]
  })
}

# CloudTrail organizacional
resource "aws_cloudtrail" "organization" {
  name                          = "organization-trail"
  s3_bucket_name                = aws_s3_bucket.cloudtrail.id
  include_global_service_events = true
  is_multi_region_trail         = true
  is_organization_trail         = true
  enable_log_file_validation    = true

  event_selector {
    read_write_type           = "All"
    include_management_events = true

    data_resource {
      type   = "AWS::S3::Object"
      values = ["arn:aws:s3:::*/"]
    }

    data_resource {
      type   = "AWS::Lambda::Function"
      values = ["arn:aws:lambda:*:*:function/*"]
    }
  }

  insight_selector {
    insight_type = "ApiCallRateInsight"
  }

  tags = {
    Name = "organization-trail"
  }
}

data "aws_caller_identity" "current" {}
```


### AWS Config organizacional

```terraform
# S3 Bucket para AWS Config
resource "aws_s3_bucket" "config" {
  bucket = "organization-config-${data.aws_caller_identity.current.account_id}"

  tags = {
    Name = "organization-config"
  }
}

# Aggregator para consolidar dados de todas as contas
resource "aws_config_configuration_aggregator" "organization" {
  name = "organization-aggregator"

  organization_aggregation_source {
    all_regions = true
    role_arn    = aws_iam_role.config_aggregator.arn
  }

  tags = {
    Name = "organization-aggregator"
  }
}

# IAM Role para aggregator
resource "aws_iam_role" "config_aggregator" {
  name = "config-aggregator-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "config.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "config_aggregator" {
  role       = aws_iam_role.config_aggregator.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSConfigRoleForOrganizations"
}

# Regras de conformidade organizacionais
resource "aws_config_organization_managed_rule" "required_tags" {
  name            = "required-tags"
  rule_identifier = "REQUIRED_TAGS"

  input_parameters = jsonencode({
    tag1Key = "Environment"
    tag2Key = "Owner"
    tag3Key = "CostCenter"
  })
}

resource "aws_config_organization_managed_rule" "encrypted_volumes" {
  name            = "encrypted-volumes"
  rule_identifier = "ENCRYPTED_VOLUMES"
}

resource "aws_config_organization_managed_rule" "s3_bucket_public_read_prohibited" {
  name            = "s3-bucket-public-read-prohibited"
  rule_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"
}
```


## Automação com AWS Control Tower

AWS Control Tower automatiza a criação e governança de multi-account.

### Habilitando Control Tower

**Via Console**:
1. Acesse AWS Control Tower
2. Clique em "Set up landing zone"
3. Configure:
   - Home region: us-east-1
   - Região adicional: us-west-2, sa-east-1
   - Log Archive account email
   - Audit account email
4. Aguarde 30-60 minutos para setup

### Account Factory for Terraform (AFT)

Para provisionamento escalável e automatizado de contas, use o **Account Factory for Terraform (AFT)** em vez do Account Factory padrão do Control Tower.

**Por que AFT é superior**:

* **Infrastructure as Code**: Contas versionadas no Git
* **Customizações avançadas**: Terraform modules aplicados automaticamente
* **Pipeline integrado**: CI/CD nativo para mudanças
* **Baseline consistente**: Mesmas configurações em todas as contas
* **Auditoria completa**: Histórico de mudanças rastreável

**Exemplo de estrutura AFT**:

```
aft-account-request/
├── terraform/
│   ├── production-account.tf
│   ├── development-account.tf
│   └── sandbox-account.tf
└── account-customizations/
    ├── security-baseline/
    │   └── terraform/
    │       ├── guardduty.tf
    │       ├── config.tf
    │       └── cloudtrail.tf
    └── network-baseline/
        └── terraform/
            ├── vpc.tf
            └── transit-gateway-attachment.tf
```

**Exemplo de request de conta**:

```terraform
module "production_account" {
  source = "./modules/aft-account-request"

  control_tower_parameters = {
    AccountEmail              = "aws-app-production@company.com"
    AccountName               = "App-Production"
    ManagedOrganizationalUnit = "Production"
    SSOUserEmail              = "admin@company.com"
    SSOUserFirstName          = "Admin"
    SSOUserLastName           = "User"
  }

  account_tags = {
    Environment = "production"
    Application = "app-name"
    CostCenter  = "engineering"
    Owner       = "team-platform"
  }

  account_customizations_name = "production-baseline"
}
```

Para implementação completa do AFT, incluindo setup, customizações e melhores práticas, leia:
👉 [Account Factory for Terraform: A Chave para o Provisionamento Escalável de Contas AWS](https://thiagoalexandria.com.br/account-factory-for-terraform-a-chave-para-o-provisionamento-escalavel-de-contas-aws/)
```

### Customizações com Control Tower

Use Customizations for Control Tower (CfCT) para aplicar configurações adicionais:

**Estrutura de diretórios**:

```
customizations/
├── manifest.yaml
├── parameters/
│   ├── security-baseline.json
│   └── network-config.json
└── templates/
    ├── security-baseline.yaml
    └── network-config.yaml
```

**Arquivo `manifest.yaml`**:

```yaml
version: 2021-03-15

resources:
  - name: SecurityBaseline
    resource_file: templates/security-baseline.yaml
    parameters:
      - parameter_file: parameters/security-baseline.json
    deploy_method: stack_set
    deployment_targets:
      organizational_units:
        - Workloads
    regions:
      - us-east-1
      - us-west-2

  - name: NetworkConfig
    resource_file: templates/network-config.yaml
    parameters:
      - parameter_file: parameters/network-config.json
    deploy_method: stack_set
    deployment_targets:
      accounts:
        - Network
    regions:
      - us-east-1
```


## Gestão de custos multi-account

### Consolidated Billing

Todos os custos são consolidados na Management Account, mas você pode ver por conta individual.

**Via AWS CLI**:

```bash
# Custos por conta no último mês
aws ce get-cost-and-usage \
  --time-period Start=2026-02-01,End=2026-03-01 \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=LINKED_ACCOUNT

# Custos por serviço em uma conta específica
aws ce get-cost-and-usage \
  --time-period Start=2026-02-01,End=2026-03-01 \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE \
  --filter file://filter.json
```

**Arquivo `filter.json`**:

```json
{
  "Dimensions": {
    "Key": "LINKED_ACCOUNT",
    "Values": ["123456789012"]
  }
}
```

### Budgets por conta

```terraform
# Budget para conta de Production
resource "aws_budgets_budget" "production" {
  name              = "production-monthly-budget"
  budget_type       = "COST"
  limit_amount      = "5000"
  limit_unit        = "USD"
  time_period_start = "2026-01-01_00:00"
  time_unit         = "MONTHLY"

  cost_filter {
    name = "LinkedAccount"
    values = [
      aws_organizations_account.production.id
    ]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = ["finance@company.com"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 100
    threshold_type             = "PERCENTAGE"
    notification_type          = "ACTUAL"
    subscriber_email_addresses = ["finance@company.com", "cto@company.com"]
  }
}
```

### Cost Allocation Tags

```terraform
# Ativar tags para cost allocation
resource "aws_ce_cost_allocation_tag" "environment" {
  tag_key = "Environment"
  status  = "Active"
}

resource "aws_ce_cost_allocation_tag" "cost_center" {
  tag_key = "CostCenter"
  status  = "Active"
}

resource "aws_ce_cost_allocation_tag" "project" {
  tag_key = "Project"
  status  = "Active"
}
```


## Segurança centralizada

### GuardDuty organizacional

```terraform
# Habilitar GuardDuty na conta Security Tooling
resource "aws_guardduty_detector" "main" {
  enable = true

  datasources {
    s3_logs {
      enable = true
    }
    kubernetes {
      audit_logs {
        enable = true
      }
    }
    malware_protection {
      scan_ec2_instance_with_findings {
        ebs_volumes {
          enable = true
        }
      }
    }
  }

  tags = {
    Name = "organization-guardduty"
  }
}

# Auto-enable para novas contas
resource "aws_guardduty_organization_configuration" "main" {
  auto_enable = true
  detector_id = aws_guardduty_detector.main.id

  datasources {
    s3_logs {
      auto_enable = true
    }
    kubernetes {
      audit_logs {
        enable = true
      }
    }
    malware_protection {
      scan_ec2_instance_with_findings {
        ebs_volumes {
          auto_enable = true
        }
      }
    }
  }
}
```

### Security Hub organizacional

```terraform
# Habilitar Security Hub
resource "aws_securityhub_account" "main" {}

# Habilitar padrões de compliance
resource "aws_securityhub_standards_subscription" "cis" {
  standards_arn = "arn:aws:securityhub:us-east-1::standards/cis-aws-foundations-benchmark/v/1.4.0"
  depends_on    = [aws_securityhub_account.main]
}

resource "aws_securityhub_standards_subscription" "pci_dss" {
  standards_arn = "arn:aws:securityhub:us-east-1::standards/pci-dss/v/3.2.1"
  depends_on    = [aws_securityhub_account.main]
}

resource "aws_securityhub_standards_subscription" "aws_foundational" {
  standards_arn = "arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0"
  depends_on    = [aws_securityhub_account.main]
}

# Auto-enable para novas contas
resource "aws_securityhub_organization_configuration" "main" {
  auto_enable = true
}
```


## Casos de uso e padrões

### Caso 1: SaaS multi-tenant

**Estrutura**:
```
Root
├── Core OU
│   ├── Management
│   ├── Security
│   └── Network
└── Tenants OU
    ├── Customer-A
    ├── Customer-B
    └── Customer-C
```

**Benefícios**:
- Isolamento total entre clientes
- Custos separados por cliente
- Compliance simplificado
- Blast radius limitado

### Caso 2: Múltiplas aplicações independentes

**Estrutura**:
```
Root
├── Infrastructure OU
│   ├── Network
│   └── Shared Services
└── Applications OU
    ├── App-A OU
    │   ├── App-A-Prod
    │   └── App-A-Dev
    └── App-B OU
        ├── App-B-Prod
        └── App-B-Dev
```

**Benefícios**:
- Times autônomos
- Deploys independentes
- Limites de serviço dedicados
- Custos por aplicação

### Caso 3: Ambientes por região

**Estrutura**:
```
Root
├── US-East OU
│   ├── US-East-Prod
│   └── US-East-Dev
└── SA-East OU
    ├── SA-East-Prod
    └── SA-East-Dev
```

**Benefícios**:
- Compliance regional
- Latência otimizada
- Disaster recovery
- Custos por região


## Erros comuns e como evitar

### 1. Usar Management Account para workloads

**Erro**: Rodar aplicações na Management Account.

**Correto**: Management Account deve ser usada apenas para gerenciamento do Organizations. Crie contas separadas para workloads.

### 2. Não planejar CIDR blocks

**Erro**: Usar CIDRs sobrepostos entre contas.

**Correto**: Planeje o IP addressing desde o início:
- Production: 10.0.0.0/16
- Development: 10.1.0.0/16
- Shared Services: 10.2.0.0/16
- Network: 10.255.0.0/16

### 3. SCPs muito permissivas

**Erro**: Não aplicar SCPs ou usar apenas allow.

**Correto**: Use SCPs para deny explícito:
```terraform
# Deny deixar região
# Deny deletar logs
# Deny desabilitar CloudTrail
# Deny usar root user
```

### 4. Não usar Identity Center

**Erro**: Criar usuários IAM em cada conta.

**Correto**: Use Identity Center para SSO centralizado.

### 5. Logs não centralizados

**Erro**: CloudTrail e Config por conta.

**Correto**: CloudTrail organizacional na conta Audit, Config aggregator.

### 6. Não automatizar criação de contas

**Erro**: Criar contas manualmente via console sem padronização.

**Correto**: Use Account Factory for Terraform (AFT) para:
* Padronização automática de configurações
* Baseline de segurança aplicado em todas as contas
* Versionamento e auditoria de mudanças
* Self-service para times com aprovação

Leia mais: [Account Factory for Terraform](https://thiagoalexandria.com.br/account-factory-for-terraform-a-chave-para-o-provisionamento-escalavel-de-contas-aws/)

### 7. Não testar SCPs antes de aplicar

**Erro**: Aplicar SCP diretamente em produção.

**Correto**: Teste em conta sandbox primeiro, depois aplique gradualmente.


## Checklist de implementação

Antes de ir para produção:

**Estrutura organizacional**:
- [ ] AWS Organizations habilitado com all features
- [ ] OUs criadas seguindo a arquitetura de referência
- [ ] Contas criadas nas OUs corretas
- [ ] Naming convention documentado

**Segurança**:
- [ ] SCPs aplicadas em todas as OUs
- [ ] Root user da Management Account com MFA
- [ ] Identity Center configurado
- [ ] Permission Sets criados
- [ ] Grupos e usuários configurados
- [ ] GuardDuty habilitado em todas as contas
- [ ] Security Hub habilitado em todas as contas

**Rede**:
- [ ] Transit Gateway criado na conta Network
- [ ] VPCs criadas com CIDRs não sobrepostos
- [ ] Attachments configurados
- [ ] Route tables configuradas
- [ ] Inspection VPC implementada (se necessário)

**Governança**:
- [ ] CloudTrail organizacional habilitado
- [ ] AWS Config aggregator configurado
- [ ] Config Rules aplicadas
- [ ] Budgets configurados por conta
- [ ] Cost Allocation Tags ativadas

**Automação**:
- [ ] Control Tower habilitado (opcional)
- [ ] Account Factory for Terraform (AFT) configurado
- [ ] Account request repository criado
- [ ] Account customizations definidas
- [ ] Terraform/IaC para todas as configurações
- [ ] CI/CD para mudanças de infraestrutura

**Documentação**:
- [ ] Arquitetura documentada
- [ ] Runbooks criados
- [ ] Processo de criação de contas documentado
- [ ] Processo de acesso documentado
- [ ] Disaster recovery plan


## Custos estimados

Entenda o impacto financeiro da estrutura multi-account:

| Serviço | Custo | Observações |
|---------|-------|-------------|
| AWS Organizations | Gratuito | Sem custo adicional |
| IAM Identity Center | Gratuito | Até 50 usuários |
| CloudTrail (org) | $2/100k eventos | Primeiro trail gratuito |
| AWS Config | $2/regra/região | Por conta ativa |
| GuardDuty | $4.50/milhão eventos | Por conta |
| Security Hub | $0.0010/finding | Por conta |
| Transit Gateway | $0.05/hora + $0.02/GB | Por attachment |
| VPC | Gratuito | Exceto NAT Gateway |
| Control Tower | Gratuito | Custos dos serviços subjacentes |

**Exemplo para 10 contas**:
- CloudTrail: ~$50/mês
- AWS Config: ~$200/mês (10 regras x 2 regiões x 10 contas)
- GuardDuty: ~$100/mês
- Security Hub: ~$50/mês
- Transit Gateway: ~$150/mês (3 attachments)

**Total estimado**: $550-700/mês para governança completa de 10 contas.

## Próximos passos

Depois de implementar a estrutura base:

### 1. Implementar FinOps

- Configure Cost Anomaly Detection
- Crie dashboards de custos por conta/OU
- Implemente tagging strategy
- Configure Savings Plans e Reserved Instances

### 2. Melhorar segurança

- Implemente AWS Systems Manager Session Manager
- Configure AWS Backup centralizado
- Implemente AWS Secrets Manager
- Configure Amazon Macie para proteção de dados

### 3. Automação avançada

- Implemente Infrastructure as Code para tudo
- Configure pipelines de CI/CD
- Automatize resposta a incidentes
- Implemente self-service para desenvolvedores

### 4. Compliance contínuo

- Configure AWS Audit Manager
- Implemente relatórios automatizados
- Configure alertas de compliance
- Documente processos de auditoria


## Conclusão

Estruturar um ambiente multi-account na AWS não é apenas uma boa prática, é uma necessidade para organizações que levam segurança, governança e escalabilidade a sério.

Com a implementação que vimos neste artigo, você tem:

* **Isolamento de segurança**: Cada conta é uma barreira de segurança independente
* **Governança centralizada**: Políticas aplicadas via Organizations e SCPs
* **Controle financeiro**: Custos separados e rastreáveis por conta
* **Rede escalável**: Transit Gateway conectando todas as contas
* **Acesso unificado**: Identity Center para SSO em todas as contas
* **Auditoria completa**: CloudTrail e Config centralizados
* **Automação**: Account Factory para criação padronizada

Os principais pontos para lembrar:

1. **Planeje antes de implementar**: Defina OUs, naming conventions e CIDR blocks
2. **Comece pequeno**: Implemente a estrutura base e expanda gradualmente
3. **Automatize tudo**: Use Terraform ou CloudFormation desde o início
4. **Segurança em camadas**: SCPs + Identity Center + GuardDuty + Security Hub
5. **Monitore constantemente**: CloudTrail, Config, Cost Explorer

A diferença entre organizações que escalam com sucesso e aquelas que enfrentam problemas está na fundação. Uma estrutura multi-account bem planejada é essa fundação.

Não espere ter problemas para implementar. Comece certo desde o início Multi-account não é complexidade desnecessária. É a forma correta de usar AWS em escala.
