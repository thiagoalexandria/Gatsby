---
image: /assets/img/AWS.png
title: AWS Network Security na Pratica
description: Em ambientes na AWS, a exposição indevida de serviços acontece mais
  rápido do que muitos imaginam. Um Security Group mal configurado, uma porta
  aberta para 0.0.0.0/0 ou a ausência de monitoramento de tráfego pode ser
  suficiente para transformar uma aplicação comum em alvo constante de
  varreduras e tentativas de invasão. O problema raramente está na falta de
  recursos de segurança — mas sim na forma como eles são projetados e operados.
  Neste artigo, exploramos como estruturar uma arquitetura de rede realmente
  segura na prática, combinando camadas de proteção, visibilidade e controle
  para reduzir riscos antes que eles se tornem incidentes.
date: 2026-03-20
category: aws
background: "#FF9900"
tags:
  - AWS
  - AWSNETWORKSECURITY
  - AMAZONVPC
  - SECURITYGROUPS
  - NETWORKACLS
  - VPCFLOWLOGS
  - AWSNETWORKFIREWALL
  - DEFENSEINDEPTH
  - CLOUDSECURITY
  - SEGURANCAEMNUVEM
  - ARQUITETURADEREDES
  - INFRAESTRUTURAEMNUVEM
  - PROTECAODEINFRAESTRUTURA
  - CLOUDMONITORING
categories:
  - AWS
  - AWSNETWORKSECURITY
  - AMAZONVPC
  - SECURITYGROUPS
  - NETWORKACLS
  - VPCFLOWLOGS
  - AWSNETWORKFIREWALL
  - DEFENSEINDEPTH
  - CLOUDSECURITY
  - SEGURANCAEMNUVEM
  - ARQUITETURADEREDES
  - INFRAESTRUTURAEMNUVEM
  - PROTECAODEINFRAESTRUTURA
  - CLOUDMONITORING
---
Segurança de rede na AWS não começa no firewall, começa na arquitetura. Em muitos ambientes, a proteção é tratada como ajuste fino após o deploy, quando na verdade deveria ser parte estrutural do desenho da VPC. Um servidor exposto à internet pode receber milhares de tentativas de acesso em poucas horas, e na maioria dos casos o problema não é sofisticado: é configuração.

Neste artigo, você vai aprender:

* Como estruturar defesa em profundidade com Security Groups, NACLs, WAF e Network Firewall
* Quando usar cada camada de proteção e como elas se complementam
* Implementação prática com Terraform de uma arquitetura multi-camadas
* Como monitorar e validar a efetividade das suas regras de segurança
* Casos reais de uso e armadilhas comuns

No final, você terá uma arquitetura de referência que pode adaptar para seus próprios ambientes.

## O modelo de defesa em profundidade

Defesa em profundidade (Defense in Depth) é o princípio de criar múltiplas camadas de segurança, onde cada uma compensa as limitações da outra. Na AWS, isso significa combinar diferentes serviços de rede, cada um operando em um nível diferente do modelo OSI.

### As quatro camadas de proteção

1. **Security Groups (Camada 4 - Stateful)**
   * Firewall de instância, opera no nível de ENI
   * Stateful: retorno automático de conexões estabelecidas
   * Permite apenas regras de permissão (allow)
   * Ideal para controle granular por recurso

2. **Network ACLs (Camada 4 - Stateless)**
   * Firewall de subnet, primeira linha de defesa
   * Stateless: requer regras explícitas de entrada e saída
   * Suporta regras de negação (deny)
   * Ideal para bloqueios amplos e proteção de subnet

3. **AWS WAF (Camada 7 - Application)**
   * Proteção de aplicações web
   * Inspeção de conteúdo HTTP/HTTPS
   * Proteção contra OWASP Top 10
   * Ideal para APIs e aplicações web

4. **AWS Network Firewall (Camadas 3-7 - Stateful)**
   * Firewall gerenciado de VPC
   * Inspeção profunda de pacotes (DPI)
   * Filtragem por domínio e IPS/IDS
   * Ideal para tráfego leste-oeste e saída para internet


### Quando usar cada camada

| Cenário | Security Group | NACL | WAF | Network Firewall |
|---------|---------------|------|-----|------------------|
| Controlar acesso entre microsserviços | ✅ | ❌ | ❌ | ❌ |
| Bloquear IPs maliciosos conhecidos | ⚠️ | ✅ | ✅ | ✅ |
| Proteger contra SQL Injection | ❌ | ❌ | ✅ | ❌ |
| Filtrar tráfego de saída por domínio | ❌ | ❌ | ❌ | ✅ |
| Inspeção de tráfego criptografado (TLS) | ❌ | ❌ | ❌ | ✅ |
| Proteção DDoS camada 7 | ❌ | ❌ | ✅ | ❌ |
| Controle de portas por subnet | ⚠️ | ✅ | ❌ | ✅ |

## Arquitetura de referência

Vamos construir uma arquitetura completa com todas as camadas de segurança. O cenário:

* VPC com subnets públicas e privadas
* Application Load Balancer na subnet pública
* Aplicação web em EC2 na subnet privada
* Banco de dados RDS em subnet isolada
* NAT Gateway para saída controlada


### Diagrama da arquitetura

```
Internet
    |
    v
[AWS WAF] ← Camada 7: Proteção de aplicação
    |
    v
[ALB - Subnet Pública]
    |
    v
[Security Group ALB] ← Camada 4: Controle stateful
    |
    v
[NACL Subnet Privada] ← Camada 4: Controle stateless
    |
    v
[EC2 - Subnet Privada]
    |
    v
[Security Group EC2]
    |
    v
[Security Group RDS]
    |
    v
[RDS - Subnet Isolada]

Saída para Internet:
[EC2] → [NAT Gateway] → [Network Firewall] → [Internet Gateway]
```

## Hands-on: Implementando a arquitetura

### Pré-requisitos

* Conta AWS com permissões administrativas
* Terraform instalado (versão 1.0+)
* AWS CLI configurado


### 1. Estrutura base da VPC

**Arquivo `vpc.tf`**:

```terraform
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "secure-vpc"
    Environment = "production"
  }
}

# Subnets públicas (ALB)
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "public-subnet-${count.index + 1}"
    Tier = "public"
  }
}

# Subnets privadas (EC2)
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "private-subnet-${count.index + 1}"
    Tier = "private"
  }
}

# Subnets isoladas (RDS)
resource "aws_subnet" "isolated" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 20}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "isolated-subnet-${count.index + 1}"
    Tier = "isolated"
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}
```


### 2. Camada 1: Network ACLs

**Arquivo `nacls.tf`**:

```terraform
# NACL para subnet pública
resource "aws_network_acl" "public" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = aws_subnet.public[*].id

  # Permite HTTP de qualquer origem
  ingress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 80
    to_port    = 80
  }

  # Permite HTTPS de qualquer origem
  ingress {
    protocol   = "tcp"
    rule_no    = 110
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 443
    to_port    = 443
  }

  # Permite portas efêmeras (retorno de conexões)
  ingress {
    protocol   = "tcp"
    rule_no    = 120
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 1024
    to_port    = 65535
  }

  # Bloqueia IPs maliciosos conhecidos (exemplo)
  ingress {
    protocol   = -1
    rule_no    = 50
    action     = "deny"
    cidr_block = "192.0.2.0/24"  # IP de exemplo
    from_port  = 0
    to_port    = 0
  }

  # Permite todo tráfego de saída
  egress {
    protocol   = -1
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  tags = {
    Name = "public-nacl"
  }
}
```


```terraform
# NACL para subnet privada
resource "aws_network_acl" "private" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = aws_subnet.private[*].id

  # Permite tráfego da subnet pública
  ingress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "10.0.0.0/20"  # Range das subnets públicas
    from_port  = 8080
    to_port    = 8080
  }

  # Permite portas efêmeras
  ingress {
    protocol   = "tcp"
    rule_no    = 110
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 1024
    to_port    = 65535
  }

  # Permite todo tráfego de saída
  egress {
    protocol   = -1
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  tags = {
    Name = "private-nacl"
  }
}

# NACL para subnet isolada (RDS)
resource "aws_network_acl" "isolated" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = aws_subnet.isolated[*].id

  # Permite apenas tráfego da subnet privada
  ingress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "10.0.10.0/23"  # Range das subnets privadas
    from_port  = 3306
    to_port    = 3306
  }

  # Permite portas efêmeras
  ingress {
    protocol   = "tcp"
    rule_no    = 110
    action     = "allow"
    cidr_block = "10.0.10.0/23"
    from_port  = 1024
    to_port    = 65535
  }

  # Permite saída apenas para subnet privada
  egress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "10.0.10.0/23"
    from_port  = 1024
    to_port    = 65535
  }

  tags = {
    Name = "isolated-nacl"
  }
}
```


### 3. Camada 2: Security Groups

**Arquivo `security_groups.tf`**:

```terraform
# Security Group para ALB
resource "aws_security_group" "alb" {
  name        = "alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = aws_vpc.main.id

  # Permite HTTP da internet
  ingress {
    description = "HTTP from internet"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Permite HTTPS da internet
  ingress {
    description = "HTTPS from internet"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Permite todo tráfego de saída
  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "alb-security-group"
  }
}

# Security Group para EC2
resource "aws_security_group" "ec2" {
  name        = "ec2-sg"
  description = "Security group for EC2 instances"
  vpc_id      = aws_vpc.main.id

  # Permite tráfego apenas do ALB
  ingress {
    description     = "HTTP from ALB"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  # Permite todo tráfego de saída
  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "ec2-security-group"
  }
}
```


```terraform
# Security Group para RDS
resource "aws_security_group" "rds" {
  name        = "rds-sg"
  description = "Security group for RDS database"
  vpc_id      = aws_vpc.main.id

  # Permite tráfego apenas das instâncias EC2
  ingress {
    description     = "MySQL from EC2"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.ec2.id]
  }

  # Não permite tráfego de saída (banco não precisa iniciar conexões)
  tags = {
    Name = "rds-security-group"
  }
}
```

### 4. Camada 3: AWS WAF

**Arquivo `waf.tf`**:

```terraform
# Web ACL do WAF
resource "aws_wafv2_web_acl" "main" {
  name  = "secure-web-acl"
  scope = "REGIONAL"

  default_action {
    allow {}
  }

  # Regra 1: Rate limiting (proteção DDoS camada 7)
  rule {
    name     = "rate-limit-rule"
    priority = 1

    action {
      block {}
    }

    statement {
      rate_based_statement {
        limit              = 2000
        aggregate_key_type = "IP"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimitRule"
      sampled_requests_enabled   = true
    }
  }

  # Regra 2: AWS Managed Rules - Core Rule Set
  rule {
    name     = "aws-managed-core-rules"
    priority = 2

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesCommonRuleSet"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedCoreRules"
      sampled_requests_enabled   = true
    }
  }
```


```terraform
  # Regra 3: SQL Injection Protection
  rule {
    name     = "sql-injection-protection"
    priority = 3

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesSQLiRuleSet"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "SQLInjectionProtection"
      sampled_requests_enabled   = true
    }
  }

  # Regra 4: Bloqueio de IPs maliciosos conhecidos
  rule {
    name     = "block-known-bad-ips"
    priority = 4

    action {
      block {}
    }

    statement {
      ip_set_reference_statement {
        arn = aws_wafv2_ip_set.blocked_ips.arn
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "BlockedIPs"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "SecureWebACL"
    sampled_requests_enabled   = true
  }

  tags = {
    Name = "secure-web-acl"
  }
}

# IP Set para IPs bloqueados
resource "aws_wafv2_ip_set" "blocked_ips" {
  name               = "blocked-ips"
  scope              = "REGIONAL"
  ip_address_version = "IPV4"

  addresses = [
    "192.0.2.0/24",    # Exemplo de range bloqueado
    "198.51.100.0/24", # Exemplo de range bloqueado
  ]

  tags = {
    Name = "blocked-ips"
  }
}

# Associa WAF ao ALB
resource "aws_wafv2_web_acl_association" "alb" {
  resource_arn = aws_lb.main.arn
  web_acl_arn  = aws_wafv2_web_acl.main.arn
}
```


### 5. Camada 4: AWS Network Firewall

**Arquivo `network_firewall.tf`**:

```terraform
# Política do Network Firewall
resource "aws_networkfirewall_firewall_policy" "main" {
  name = "secure-firewall-policy"

  firewall_policy {
    # Stateless default actions
    stateless_default_actions          = ["aws:forward_to_sfe"]
    stateless_fragment_default_actions = ["aws:forward_to_sfe"]

    # Stateful rule group references
    stateful_rule_group_reference {
      resource_arn = aws_networkfirewall_rule_group.block_domains.arn
    }

    stateful_rule_group_reference {
      resource_arn = aws_networkfirewall_rule_group.allow_domains.arn
    }

    stateful_rule_group_reference {
      resource_arn = aws_networkfirewall_rule_group.intrusion_detection.arn
    }
  }

  tags = {
    Name = "secure-firewall-policy"
  }
}

# Rule Group: Bloqueio de domínios maliciosos
resource "aws_networkfirewall_rule_group" "block_domains" {
  capacity = 100
  name     = "block-malicious-domains"
  type     = "STATEFUL"

  rule_group {
    rules_source {
      rules_source_list {
        generated_rules_type = "DENYLIST"
        target_types         = ["HTTP_HOST", "TLS_SNI"]
        targets = [
          ".malware-domain.com",
          ".phishing-site.net",
          ".cryptominer.org"
        ]
      }
    }
  }

  tags = {
    Name = "block-malicious-domains"
  }
}
```


```terraform
# Rule Group: Whitelist de domínios permitidos
resource "aws_networkfirewall_rule_group" "allow_domains" {
  capacity = 100
  name     = "allow-trusted-domains"
  type     = "STATEFUL"

  rule_group {
    rules_source {
      rules_source_list {
        generated_rules_type = "ALLOWLIST"
        target_types         = ["HTTP_HOST", "TLS_SNI"]
        targets = [
          ".amazonaws.com",
          ".github.com",
          ".docker.io",
          "api.example.com"
        ]
      }
    }
  }

  tags = {
    Name = "allow-trusted-domains"
  }
}

# Rule Group: Detecção de intrusão (IDS)
resource "aws_networkfirewall_rule_group" "intrusion_detection" {
  capacity = 1000
  name     = "intrusion-detection"
  type     = "STATEFUL"

  rule_group {
    rules_source {
      stateful_rule {
        action = "ALERT"
        header {
          destination      = "ANY"
          destination_port = "ANY"
          direction        = "ANY"
          protocol         = "TCP"
          source           = "ANY"
          source_port      = "ANY"
        }
        rule_option {
          keyword  = "sid"
          settings = ["1"]
        }
      }
    }

    rule_variables {
      ip_sets {
        key = "HOME_NET"
        ip_set {
          definition = ["10.0.0.0/16"]
        }
      }
    }
  }

  tags = {
    Name = "intrusion-detection"
  }
}
```


```terraform
# Subnet para Network Firewall
resource "aws_subnet" "firewall" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 30}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "firewall-subnet-${count.index + 1}"
  }
}

# Network Firewall
resource "aws_networkfirewall_firewall" "main" {
  name                = "secure-network-firewall"
  firewall_policy_arn = aws_networkfirewall_firewall_policy.main.arn
  vpc_id              = aws_vpc.main.id

  dynamic "subnet_mapping" {
    for_each = aws_subnet.firewall
    content {
      subnet_id = subnet_mapping.value.id
    }
  }

  tags = {
    Name = "secure-network-firewall"
  }
}

# Logging para Network Firewall
resource "aws_cloudwatch_log_group" "firewall" {
  name              = "/aws/networkfirewall/secure-firewall"
  retention_in_days = 30

  tags = {
    Name = "firewall-logs"
  }
}

resource "aws_networkfirewall_logging_configuration" "main" {
  firewall_arn = aws_networkfirewall_firewall.main.arn

  logging_configuration {
    log_destination_config {
      log_destination = {
        logGroup = aws_cloudwatch_log_group.firewall.name
      }
      log_destination_type = "CloudWatchLogs"
      log_type             = "ALERT"
    }

    log_destination_config {
      log_destination = {
        logGroup = aws_cloudwatch_log_group.firewall.name
      }
      log_destination_type = "CloudWatchLogs"
      log_type             = "FLOW"
    }
  }
}
```


## Monitoramento e visibilidade

Segurança sem visibilidade é segurança cega. Vamos configurar VPC Flow Logs e CloudWatch para monitorar todo o tráfego.

### VPC Flow Logs

**Arquivo `flow_logs.tf`**:

```terraform
# S3 Bucket para Flow Logs
resource "aws_s3_bucket" "flow_logs" {
  bucket = "vpc-flow-logs-${data.aws_caller_identity.current.account_id}"

  tags = {
    Name = "vpc-flow-logs"
  }
}

resource "aws_s3_bucket_lifecycle_configuration" "flow_logs" {
  bucket = aws_s3_bucket.flow_logs.id

  rule {
    id     = "delete-old-logs"
    status = "Enabled"

    expiration {
      days = 90
    }
  }
}

# IAM Role para Flow Logs
resource "aws_iam_role" "flow_logs" {
  name = "vpc-flow-logs-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "vpc-flow-logs.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy" "flow_logs" {
  name = "vpc-flow-logs-policy"
  role = aws_iam_role.flow_logs.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ]
      Resource = "*"
    }]
  })
}
```


```terraform
# CloudWatch Log Group para Flow Logs
resource "aws_cloudwatch_log_group" "flow_logs" {
  name              = "/aws/vpc/flow-logs"
  retention_in_days = 30

  tags = {
    Name = "vpc-flow-logs"
  }
}

# VPC Flow Logs
resource "aws_flow_log" "main" {
  vpc_id          = aws_vpc.main.id
  traffic_type    = "ALL"
  iam_role_arn    = aws_iam_role.flow_logs.arn
  log_destination = aws_cloudwatch_log_group.flow_logs.arn

  tags = {
    Name = "vpc-flow-logs"
  }
}

data "aws_caller_identity" "current" {}
```

### Queries úteis para análise de logs

**CloudWatch Insights - Top 10 IPs rejeitados**:

```sql
fields @timestamp, srcAddr, dstAddr, dstPort, action
| filter action = "REJECT"
| stats count() as rejectedCount by srcAddr
| sort rejectedCount desc
| limit 10
```

**CloudWatch Insights - Tráfego por porta**:

```sql
fields @timestamp, srcAddr, dstAddr, dstPort, protocol
| stats count() as connectionCount by dstPort, protocol
| sort connectionCount desc
| limit 20
```

**CloudWatch Insights - Conexões suspeitas (múltiplas portas)**:

```sql
fields @timestamp, srcAddr, dstPort
| stats count_distinct(dstPort) as uniquePorts by srcAddr
| filter uniquePorts > 10
| sort uniquePorts desc
```


## Alertas e automação

Configure alertas para eventos críticos de segurança.

**Arquivo `alarms.tf`**:

```terraform
# SNS Topic para alertas
resource "aws_sns_topic" "security_alerts" {
  name = "security-alerts"

  tags = {
    Name = "security-alerts"
  }
}

resource "aws_sns_topic_subscription" "security_email" {
  topic_arn = aws_sns_topic.security_alerts.arn
  protocol  = "email"
  endpoint  = "security-team@example.com"
}

# Métrica customizada: Conexões rejeitadas
resource "aws_cloudwatch_log_metric_filter" "rejected_connections" {
  name           = "RejectedConnections"
  log_group_name = aws_cloudwatch_log_group.flow_logs.name
  pattern        = "[version, account, eni, source, destination, srcport, destport, protocol, packets, bytes, windowstart, windowend, action=REJECT, flowlogstatus]"

  metric_transformation {
    name      = "RejectedConnectionCount"
    namespace = "VPC/Security"
    value     = "1"
  }
}

# Alarme: Muitas conexões rejeitadas
resource "aws_cloudwatch_metric_alarm" "high_rejected_connections" {
  alarm_name          = "high-rejected-connections"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "1"
  metric_name         = "RejectedConnectionCount"
  namespace           = "VPC/Security"
  period              = "300"
  statistic           = "Sum"
  threshold           = "100"
  alarm_description   = "Alerta quando há muitas conexões rejeitadas"
  alarm_actions       = [aws_sns_topic.security_alerts.arn]

  tags = {
    Name = "high-rejected-connections"
  }
}
```


```terraform
# Métrica customizada: Bloqueios do WAF
resource "aws_cloudwatch_metric_alarm" "waf_blocked_requests" {
  alarm_name          = "waf-high-blocked-requests"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "BlockedRequests"
  namespace           = "AWS/WAFV2"
  period              = "300"
  statistic           = "Sum"
  threshold           = "1000"
  alarm_description   = "Alerta quando WAF bloqueia muitas requisições"
  alarm_actions       = [aws_sns_topic.security_alerts.arn]

  dimensions = {
    WebACL = aws_wafv2_web_acl.main.name
    Region = "us-east-1"
    Rule   = "ALL"
  }

  tags = {
    Name = "waf-blocked-requests"
  }
}

# EventBridge Rule: Mudanças em Security Groups
resource "aws_cloudwatch_event_rule" "security_group_changes" {
  name        = "security-group-changes"
  description = "Detecta mudanças em Security Groups"

  event_pattern = jsonencode({
    source      = ["aws.ec2"]
    detail-type = ["AWS API Call via CloudTrail"]
    detail = {
      eventName = [
        "AuthorizeSecurityGroupIngress",
        "AuthorizeSecurityGroupEgress",
        "RevokeSecurityGroupIngress",
        "RevokeSecurityGroupEgress",
        "CreateSecurityGroup",
        "DeleteSecurityGroup"
      ]
    }
  })
}

resource "aws_cloudwatch_event_target" "security_group_changes" {
  rule      = aws_cloudwatch_event_rule.security_group_changes.name
  target_id = "SendToSNS"
  arn       = aws_sns_topic.security_alerts.arn
}
```


## Testes de validação

Depois de implementar, valide se tudo está funcionando corretamente.

### 1. Teste de Security Groups

```bash
# De dentro de uma instância EC2, tente acessar o RDS diretamente
# Deve funcionar apenas se a instância estiver no security group correto

mysql -h <rds-endpoint> -u admin -p

# Tente acessar de uma instância não autorizada
# Deve falhar com timeout
```

### 2. Teste de NACL

```bash
# Tente acessar uma porta bloqueada pela NACL
nc -zv <ip-privado> 22

# Deve retornar "Connection refused" ou timeout
```

### 3. Teste de WAF

```bash
# Teste SQL Injection (deve ser bloqueado)
curl -X POST https://seu-alb.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "' OR '1'='1"}'

# Teste rate limiting (envie muitas requisições)
for i in {1..3000}; do
  curl https://seu-alb.com/ &
done

# Após 2000 requisições, deve começar a retornar 403
```

### 4. Teste de Network Firewall

```bash
# De dentro de uma instância, tente acessar um domínio bloqueado
curl https://malware-domain.com

# Deve falhar ou retornar erro de DNS/conexão

# Tente acessar um domínio permitido
curl https://api.github.com

# Deve funcionar normalmente
```


## Casos de uso reais

### Caso 1: Bloqueio de ataque de força bruta

**Problema**: Aplicação recebendo milhares de tentativas de login de IPs diferentes.

**Solução em camadas**:
1. **WAF**: Rate limiting por IP (2000 req/5min)
2. **Security Group**: Limitar acesso apenas do ALB
3. **NACL**: Bloquear ranges de IPs conhecidos por ataques
4. **Network Firewall**: Detectar padrões de ataque distribuído

**Resultado**: 99% das tentativas bloqueadas antes de chegar à aplicação.

### Caso 2: Prevenção de exfiltração de dados

**Problema**: Instância comprometida tentando enviar dados para servidor externo.

**Solução em camadas**:
1. **Network Firewall**: Whitelist de domínios permitidos
2. **Security Group**: Egress restrito apenas para serviços necessários
3. **VPC Flow Logs**: Detecção de conexões anômalas
4. **CloudWatch Alarms**: Alerta em volume alto de saída

**Resultado**: Conexão bloqueada automaticamente, alerta enviado em tempo real.

### Caso 3: Proteção contra SQL Injection

**Problema**: Aplicação web vulnerável a SQL Injection.

**Solução em camadas**:
1. **WAF**: Regras AWS Managed SQLi RuleSet
2. **Security Group RDS**: Acesso apenas de instâncias específicas
3. **NACL**: Subnet isolada para banco de dados
4. **CloudWatch**: Monitoramento de queries suspeitas

**Resultado**: Tentativas de SQL Injection bloqueadas no WAF, banco nunca exposto.


## Erros comuns e como evitar

### 1. Security Groups muito permissivos

**Erro**:
```terraform
ingress {
  from_port   = 0
  to_port     = 65535
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]
}
```

**Correto**:
```terraform
ingress {
  from_port       = 443
  to_port         = 443
  protocol        = "tcp"
  security_groups = [aws_security_group.alb.id]
}
```

### 2. Esquecer portas efêmeras em NACLs

**Erro**: Bloquear portas efêmeras (1024-65535) na saída.

**Correto**: Sempre permitir portas efêmeras para retorno de conexões:
```terraform
egress {
  protocol   = "tcp"
  rule_no    = 100
  action     = "allow"
  cidr_block = "0.0.0.0/0"
  from_port  = 1024
  to_port    = 65535
}
```

### 3. WAF sem rate limiting

**Erro**: Confiar apenas em regras de conteúdo.

**Correto**: Sempre incluir rate limiting:
```terraform
rule {
  name     = "rate-limit"
  priority = 1
  action {
    block {}
  }
  statement {
    rate_based_statement {
      limit              = 2000
      aggregate_key_type = "IP"
    }
  }
}
```


### 4. Não monitorar logs

**Erro**: Implementar segurança mas não analisar logs.

**Correto**: Configure CloudWatch Insights e crie dashboards:
```terraform
resource "aws_cloudwatch_dashboard" "security" {
  dashboard_name = "security-overview"

  dashboard_body = jsonencode({
    widgets = [
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/WAFV2", "BlockedRequests"],
            ["VPC/Security", "RejectedConnectionCount"]
          ]
          period = 300
          stat   = "Sum"
          region = "us-east-1"
          title  = "Security Events"
        }
      }
    ]
  })
}
```

### 5. Network Firewall sem logging

**Erro**: Não configurar logs do Network Firewall.

**Correto**: Sempre habilite ALERT e FLOW logs:
```terraform
logging_configuration {
  log_destination_config {
    log_destination = {
      logGroup = aws_cloudwatch_log_group.firewall.name
    }
    log_destination_type = "CloudWatchLogs"
    log_type             = "ALERT"
  }
  log_destination_config {
    log_destination = {
      logGroup = aws_cloudwatch_log_group.firewall.name
    }
    log_destination_type = "CloudWatchLogs"
    log_type             = "FLOW"
  }
}
```


## Custos estimados

Entenda o impacto financeiro de cada camada:

| Serviço | Custo mensal estimado | Observações |
|---------|----------------------|-------------|
| Security Groups | Gratuito | Sem custo adicional |
| Network ACLs | Gratuito | Sem custo adicional |
| VPC Flow Logs | $10-50 | Depende do volume de tráfego |
| AWS WAF | $5 + $1/milhão req | Base + regras + requisições |
| Network Firewall | $395 + $0.065/GB | $0.395/hora + processamento |
| CloudWatch Logs | $0.50/GB | Ingestão e armazenamento |
| CloudWatch Alarms | $0.10/alarme | Primeiros 10 gratuitos |

**Total estimado para ambiente médio**: $450-600/mês

**Dica de economia**: 
- Use VPC Flow Logs apenas em subnets críticas
- Configure retenção adequada nos logs (30 dias é suficiente)
- Network Firewall pode ser substituído por NAT Gateway + Security Groups em ambientes menores

## Checklist de implementação

Antes de ir para produção, valide:

- [ ] Security Groups seguem princípio do menor privilégio
- [ ] NACLs têm regras de deny para IPs maliciosos conhecidos
- [ ] WAF tem rate limiting configurado
- [ ] WAF tem regras AWS Managed ativas
- [ ] Network Firewall tem whitelist de domínios
- [ ] VPC Flow Logs estão habilitados
- [ ] CloudWatch Alarms estão configurados
- [ ] SNS Topic para alertas está funcionando
- [ ] Logs têm retenção configurada
- [ ] Dashboard de segurança está criado
- [ ] Testes de validação foram executados
- [ ] Documentação está atualizada
- [ ] Runbook de resposta a incidentes existe


## Próximos passos

Depois de implementar a arquitetura base, considere:

### 1. Automação de resposta a incidentes

Use Lambda para responder automaticamente a eventos:

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    
    # Detecta Security Group com 0.0.0.0/0
    if '0.0.0.0/0' in event['detail']['requestParameters']:
        sg_id = event['detail']['responseElements']['securityGroupId']
        
        # Remove a regra permissiva
        ec2.revoke_security_group_ingress(
            GroupId=sg_id,
            IpPermissions=[{
                'IpProtocol': '-1',
                'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
            }]
        )
        
        # Envia alerta
        sns = boto3.client('sns')
        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:123456789012:security-alerts',
            Subject='Security Group Auto-Remediation',
            Message=f'Regra permissiva removida do SG {sg_id}'
        )
```

### 2. Integração com SIEM

Envie logs para ferramentas de análise:
- Splunk
- Elastic Stack
- AWS Security Hub
- Sumo Logic

### 3. Testes de penetração automatizados

Use ferramentas como:
- **Prowler**: Auditoria de segurança AWS
- **ScoutSuite**: Avaliação de configuração
- **CloudMapper**: Visualização de rede

### 4. Compliance contínuo

Implemente AWS Config Rules para:
- Validar que Security Groups não têm 0.0.0.0/0
- Verificar que VPC Flow Logs estão habilitados
- Garantir que WAF está associado a todos os ALBs
- Confirmar que Network Firewall está ativo


## Conclusão

Segurança de rede na AWS não é um produto que você compra, é uma arquitetura que você constrói. A diferença entre ambientes seguros e ambientes vulneráveis está na forma como as camadas de proteção são combinadas e operadas.

Com a implementação que vimos neste artigo, você tem:

* **Defesa em profundidade**: Múltiplas camadas que se complementam
* **Visibilidade total**: Logs e métricas de todo o tráfego
* **Resposta automatizada**: Alertas e remediação em tempo real
* **Compliance contínuo**: Validação constante das regras de segurança

Os principais pontos para lembrar:

1. **Security Groups** são sua primeira linha de defesa para controle granular
2. **NACLs** protegem subnets inteiras e permitem regras de negação
3. **WAF** protege aplicações web contra ataques da camada 7
4. **Network Firewall** oferece inspeção profunda e controle de domínios
5. **Monitoramento** é tão importante quanto a implementação

Comece com Security Groups e NACLs bem configurados. Adicione WAF se você tem aplicações web expostas. Considere Network Firewall quando precisar de controle granular de saída ou inspeção profunda de tráfego.

O importante é não deixar a segurança para depois. Cada dia sem proteção adequada é um dia de risco desnecessário.

Segurança é um processo contínuo, não um projeto com data de término.
