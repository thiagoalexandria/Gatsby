---
image: /assets/img/AWS.png
title: Construindo IAM Least Privilege
description: Em ambientes AWS, dar permissões de Admin para todos pode até ser
  rápido, mas é como entregar a chave-mestra de um prédio para qualquer
  visitante, cedo ou tarde, algo errado vai acontecer. A prática de Least
  Privilege garante que cada usuário ou aplicação tenha apenas as permissões
  necessárias para realizar suas funções, reduzindo riscos e aumentando a
  segurança.
date: 2025-10-28
category: security
background: "#7B68EE"
tags:
  - IAM
  - AWSPERMISSIONS
  - LEASTPRIVILEGE
  - AWSIAM
  - TAGBASEDPOLICY
  - SECURITYBESTPRACTICES
  - ACCESSCONTROL
  - DEVSECOPS
  - AWSCLI
  - EC2
  - AWSPOLICIES
categories:
  - IAM
  - AWSPERMISSIONS
  - LEASTPRIVILEGE
  - AWSIAM
  - TAGBASEDPOLICY
  - SECURITYBESTPRACTICES
  - ACCESSCONTROL
  - DEVSECOPS
  - AWSCLI
  - EC2
  - AWSPOLICIES
---
Antes de falar em tag-based policies, é importante reforçar um ponto que poucas empresas percebem no começo: **na AWS, a identidade é mais importante que a rede**. Tradicionalmente no data center, segurança se baseia em limites de rede (firewalls, VLANs, perímetros). Na AWS, quem controla tudo é o IAM.

Se uma identidade (user, role ou aplicação) tem permissão, ela acessa de qualquer lugar do mundo, independentemente de rede. Por isso IAM é o primeiro pilar de segurança na nuvem, e também o mais negligenciado. 

### **O problema das permissões amplas**

Quando um desenvolvedor ou aplicação recebe uma **policy `AdministratorAccess`**, está recebendo **todas as ações em todos os serviços AWS**.\
Isso traz riscos claros:

* **Erro humano**: um simples comando errado pode apagar recursos críticos.
* **Ataques internos**: um funcionário mal-intencionado pode explorar privilégios para acessar dados sensíveis.
* **Compliance**: permissões amplas violam normas como ISO 27001, SOC 2, PCI-DSS.

Na maioria das empresas, permissões amplas nascem por três motivos principais:

| Motivo                                      | Consequência                                        |
| ------------------------------------------- | --------------------------------------------------- |
| Pressa para “fazer funcionar”               | Começa com Admin temporário e ele nunca é removido  |
| Falta de governança                         | Times não sabem quem deveria controlar cada recurso |
| Dificuldade em escrever policies granulares | IAM parece complexo, então usam *Resource "**       |



O problema é que **o erro não aparece imediatamente**, e sim no pior momento possível, em produção, durante migração ou auditoria.

## **O que é Least Privilege**

O Least Privilege, ou Princípio do Menor Privilégio, é um conceito de segurança que pode ser explicado em termos bem simples:

> *Dê a cada pessoa ou sistema apenas as chaves que eles realmente precisam, nada além disso.*

Imagine que você trabalha em um prédio corporativo com 20 andares.

* Você trabalha no **andar 5**.
* Para realizar seu trabalho, só precisa acessar o **andar 5** e a **sala de reuniões no andar 3**.
* No entanto, se a empresa te dá **um crachá que abre todos os andares**, você pode, por acidente ou intenção, acessar lugares onde não deveria estar, como o cofre no subsolo ou uma sala restrita no 18º andar.

Na nuvem, é a mesma coisa**,** um usuário, função ou aplicação só deveria ter acesso ao “andar” (ou serviço, ou recurso) que precisa usar. 

Dar acesso de **Administrador** para todo mundo é como distribuir o crachá-mestre para qualquer visitante: é prático no curto prazo, mas perigoso e caro no longo prazo.

### **Por que isso é tão importante na AWS?**

Na AWS (e em qualquer nuvem), permissões são extremamente granulares. Essa flexibilidade é um dos grandes diferenciais da nuvem, mas também é um dos maiores riscos quando se fala em permissões.

Diferente de um servidor físico no seu data center, que pode estar isolado por uma rede interna, na AWS cada API e serviço pode ser acessado de qualquer lugar do mundo se houver credenciais válidas.

Isso significa que:

* Um **erro de configuração** ou **excesso de permissão** pode causar um impacto **global** e **imediato**.
* Recursos e dados críticos podem ser alterados ou expostos **em segundos**.
* Os custos podem disparar com a criação acidental ou mal-intencionada de recursos caros (GPU, storage, instâncias grandes).

Por isso, o **Least Privilege** é tão importante na AWS, ele reduz a superfície de ataque, limita o impacto de credenciais comprometidas e impede ações não autorizadas mesmo dentro da própria equipe.

### **Cenários reais onde o Least Privilege faz diferença**

#### **1. Exclusão acidental de recursos críticos**

Imagine que um desenvolvedor precise criar instâncias EC2 para testes.\
Se ele tiver a permissão `AdministratorAccess`, ele também pode **apagar instâncias de produção** sem querer basta errar um ID ou rodar um script no ambiente errado.

* **Como o Least Privilege resolve:** Permitir apenas `ec2:RunInstances` e `ec2:TerminateInstances` para recursos com uma tag específica (ex.: `environment=dev`). Assim, mesmo que o comando seja executado com o ID errado, a AWS negará a ação.

#### **2. Aumento inesperado de custos**

Um analista de dados recebe acesso ao serviço SageMaker para rodar experimentos.\
Se ele tiver permissão ampla (`sagemaker:*`), pode acidentalmente criar instâncias p3.16xlarge (com GPU de alto desempenho) que custam **mais de US$ 30/hora**.

* **Como o Least Privilege resolve:** Restringir as instâncias permitidas a tipos e tamanhos específicos, além de aplicar limite de regiões.

#### **3. Ambiente multi-squad com governança fraca**

Sem restrição, a squad “SRE” pode apagar recursos da squad “Cloud” e vice-versa, gerando conflitos internos e risco operacional.

* **Como o Least Privilege resolve:** Usar **tag-based policies** para garantir que cada squad só gerencie recursos que tenham a tag correspondente ao seu time (`squad=cloud`, `squad=sre`, etc.).

#### **4. Vazamento de dados sensíveis**

Um engenheiro precisa acessar logs de uma aplicação no CloudWatch, mas com permissões amplas (`logs:*`) também consegue ler logs de outros sistemas que contêm dados sensíveis de clientes.

* **Como o Least Privilege resolve:** Permitir apenas acesso a grupos de logs específicos e negar explicitamente logs que contenham dados de compliance (ex.: PCI, HIPAA).

#### **5. Criação de recursos em regiões não autorizadas**

Um time cria recursos apenas na região `us-east-1`, mas um desenvolvedor com permissão ampla pode criar acidentalmente recursos em `ap-southeast-1`.\
Isso não só aumenta o custo, mas também **dificulta auditoria e resposta a incidentes**.

* **Como o Least Privilege resolve:** Restringir criação (`RunInstances`, `CreateBucket`, etc.) apenas para regiões aprovadas via `Condition` com `aws:RequestedRegion`.

### **Evaluation Strategy da AWS**

A AWS decide se permite ou não uma ação seguindo esta ordem:

1. **Explicit Deny**: Se houver negação explícita, o acesso é bloqueado.
2. **Explicit Allow**: Se não houver deny e houver allow explícito, o acesso é concedido.
3. **Default Deny**: Se nenhuma policy permitir, o acesso é negado.

Nas **tag-based policies**, o `Condition` é avaliado antes de conceder o `Allow`. Se não for atendido, o acesso é negado, mesmo com a ação listada.

### I﻿AM Users, Roles e Policies

Para contextualizar a aplicação das tag-based policies, precisamos entender o que estamos realmente autorizando:

| Componente                  | Função                                                  |
| --------------------------- | ------------------------------------------------------- |
| **IAM Users**               | Identidades humanas (cada vez menos usados)             |
| **IAM Roles**               | Identidades assumidas (aplicações, serviços, automação) |
| **Policies**                | Documento JSON que define o que pode ou não             |
| **Resource-based policies** | Policies anexadas ao recurso (ex: S3 Bucket)            |
| **IAM vs SCP**              | IAM controla “quem pode”, SCP controla “até onde pode”  |



Neste artigo estamos atuando no nível **IAM Policy**, que é onde ocorre o controle de granularidade real.



### T﻿ag-based policies: Porque são tão poderosas?

Além de restringir ações, elas criam **governança de times** sem precisar criar dezenas de policies diferentes para cada squad.\
Ou seja, você substitui:

* 10 policies diferentes
* com 10 listas de ARNs diferentes
* por 1 única lógica central

Essa estratégia escala quando o número de equipes ou recursos cresce.

## **Hands-on:**

#### **Pré-requisitos**

* AWS CLI configurado
* Permissão para criar policies e roles
* Duas instâncias EC2 (uma com tag `squad=cloud`, outra com tag diferente)

### **1. Criando a policy permissiva**

Permite criar e deletar qualquer EC2:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:TerminateInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

### **2. Criando a policy mínima (tag-based)**

Permite criar apenas instâncias com `squad=cloud` e deletar apenas instâncias que tenham essa tag.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowRunInstancesWithSquadTag",
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestTag/squad": "cloud"
        }
      }
    },
    {
      "Sid": "AllowTerminateInstancesWithSquadTag",
      "Effect": "Allow",
      "Action": "ec2:TerminateInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/squad": "cloud"
        }
      }
    }
  ]
}
```

### **3. Testando**

1. Criar instância sem tag `squad=cloud` → **AccessDenied**.
2. Criar instância com tag correta → **Permitido**.
3. Tentar deletar instância sem a tag → **AccessDenied**.
4. Deletar instância com tag correta → **Permitido**.

### **Erros comuns e soluções**

* **Falta de tag na criação**: Use `aws:RequestTag` para forçar envio da tag.
* **Confundir `RequestTag` com `ResourceTag`**: o primeiro é na criação, o segundo para recursos já existentes.
* **`Resource` amplo sem `Condition`**: sempre restringir.
* **Achar que IAM substitui SCP**: SCP é para restrições organizacionais, IAM é granular no nível de conta.

Migrar do acesso Admin para políticas mínimas é fundamental para segurança na AWS.\
O uso de **tag-based policies** é uma forma prática de aplicar controle granular, alinhando segurança e agilidade.

### Restringindo também por região

Além da tag, é comum que empresas limitem onde os recursos podem ser criados.\
Principalmente por motivos de:

* custo
* latência
* compliance
* controle de superfície de ataque

A própria AWS recomenda usar `aws:RequestedRegion` para reforçar governança.

```
{
  "Sid": "AllowRunInstancesWithSquadTagAndRegion",
  "Effect": "Allow",
  "Action": "ec2:RunInstances",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "aws:RequestTag/squad": "cloud",
      "aws:RequestedRegion": "us-east-1"
    }
  }
}

```

Dessa forma, mesmo que o usuário tente criar a instância com a tag correta, se for fora de `us-east-1` → **AccessDenied**.

## Auditoria e FinOps

Muita gente acha que IAM = segurança apenas, mas num ambiente maduro, tag-based policies fortalecem também:

| Área            | Benefício                                   |
| --------------- | ------------------------------------------- |
| Segurança       | impede ações indevidas                      |
| FinOps          | ajuda a identificar dono de custo           |
| Observabilidade | tagging consistente → dashboards precisos   |
| Auditoria       | prova de controle de acesso baseado em time |
| Governança      | elimina “recursos órfãos”                   |



Sem tag obrigatória, mais cedo ou mais tarde você descobre “caixas pretas” custando centenas de dólares/mês sem dono declarado. Tag-based IAM transforma governança em segurança de verdade, ela reduz risco operacional, elimina conflitos entre times, facilita auditoria, reforça FinOps e cria um ambiente onde cada identidade só pode tocar o que lhe pertence. 

Em vez de políticas amplas e perigosas, você implementa **least privilege real**, executado nativamente pela AWS.

Nos próximos passos, vale expandir o mesmo padrão para S3, RDS, Lambda, EKS e automatizar tudo via IaC, complementando com IAM Access Analyzer para monitorar acessos excessivos continuamente.