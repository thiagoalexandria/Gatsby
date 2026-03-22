---
image: /assets/img/AWS.png
title: Descomplicando a Seguranca na Nuvem com Prowler
description: "Segurança em ambientes cloud não é mais opcional, é prioridade. E
  se você trabalha com AWS, provavelmente já ouviu falar em CIS, PCI-DSS, ISO
  27001, SOC2 e outras sopas de letrinhas que exigem atenção especial. "
date: 2025-07-25
category: sec
background: "#4E457FFF"
tags:
  - SEGURANÇA
  - NUVEM
  - SECURITY
  - CLOUD
  - PROWLER
  - CLOUDSECURITY
  - COMPLIANCE
  - AWS
categories:
  - SEGURANÇA
  - NUVEM
  - SECURITY
  - CLOUD
  - PROWLER
  - CLOUDSECURITY
  - COMPLIANCE
  - AWS
---
Hoje vou te apresentar o Prowler, uma ferramenta open source que pode virar sua aliada na detecção de riscos e no fortalecimento da postura de segurança na AWS.

### O que é o Prowler?

O [Prowler](https://github.com/prowler-cloud/prowler) é uma ferramenta de linha de comando que roda checagens de segurança, boas práticas e conformidade na sua conta AWS. Ele foi desenvolvido iniciado por Toni e atualmente é mantido pela comunidade, com suporte a diversos frameworks de segurança como:

* C﻿IS AWS Foundations Benchmark
* P﻿CI-DSS
* I﻿SO 27001
* S﻿OC2
* G﻿DPR
* E﻿ntre outros

Além disso, o Prowler também identifica configurações perigosas ou desnecessárias, como buckets públicos, roles com permissões exageradas, chaves expostas e muito mais.

### Como instalar o Prowler

O Prowler pode ser instalado com Docker ou rodando localmente, clonando o projeto ou instalando com python. Aqui vou mostrar as três formas, para ambas as instalações você precisará ter o awscli configurado com permissões suficientes para rodar os checks.

#### Instalando via Git

```
git clone https://github.com/prowler-cloud/prowler
cd prowler
```

#### Instalando via Python

```
pip3 install prowler 
```

#### Usando com docker

```
docker run -it --rm -v ~/.aws:/root/.aws prowler/prowler:latest
```

### Como executar

Executar o Prowler é bem simples. Podemos utilizar diferentes formas de sadía como HTML, Json e CSV.

#### Rodando benchmark CIS completo

```
prowler aws -M html,json,csv -S -f us-east-1
```

#### Rodando um check específico

```
prowler aws -c check11
```

#### Ver todos o checks disponíveis

```
prowler aws -l
```

D﻿essa forma teremos uma visão sobre todos os itens que são analisados dentro da AWS:

![prowler-1](/assets/img/prowler-1.png "prowler-1")

#### Filtrando apenas itens que falhem na checagem por severidade

```
prowler aws -f sa-east-1 us-east-1 --status FAIL --severity medium high critical -M csv
```

E﻿le iniciará uma checagem completa envolvendo as duas regiões

![prowler-2](/assets/img/prowler-2.png "prowler-2")

### Analisando os relatórios

Após a execução, os relatórios são salvos na pasta output/.

O que você vai encontrar:

* ﻿Nome do check
* Descrição
* S﻿tatus
* Recurso afetado
* Recomendação se aplicável

Use o HTML para uma visão geral mais amigável e o JSON para integrar com ferramentas como Splunk, ELK ou até Lambda para automação. Se quiser apenas visualizar e armazenar históricos, podemos salvar com CSV.

### Organizando um plano de ações

Rodou os checks e apareceu uma penca de alertas? Calma! Nem tudo que parece crítica é, de fato, um risco real. O Prowler pode ser bem literal nas análises, por exemplo, se encontrar uma política com `ec2:*`, ele pode marcar como vulnerável de imediato. Mas antes de entrar em pânico, revise se essa política tem condições específicas que restringem seu uso. Muitas vezes, esse tipo de controle já é suficiente para reclassificarmos o risco como aceitável.

#### 1. Classifique os findings 

* Críticos: Risco imediato ( exemplo: Acesso root sem MFA )
* Altos: Possíveis violações ( exemplo: Bucket s3 publico )
* Médios: Ajustes recomendados
* Baixos: Boas práticas

#### 2. Agrupe por serviços
* IAM, S3, EC2, CloudTrail, etc.

#### 3. Monte uma planilha ou um épico no Jira/Trello
* Categorize os itens e direcione para as equipes que vão trabalhar nessa força tarefa

#### 4. Automatize as correções
* Se os recursos ja foram criados com IaC, utilize do gestor de configurações e aplique mudanças em lote, correções como não permitir instance metadata v1, bucket sempre privados....
* Crie alertas no CloudWatch baseado em eventos do Cloudtrail para analisar quando um recurso foi criado fora do padrão.
* Crie regras no AWS Config para tratar os recursos que ficarem não compliance
* Aplique SCPs para conter cenários mais críticos

### Final
Se você ainda não testou o Prowler, recomendo muito! Comece rodando em uma conta de teste e veja o quanto ele pode te mostrar.

Curtiu o conteúdo? Confere outros artigos em thiagoalexandria.com.br e me segue no LinkedIn pra trocar ideia sobre segurança, automação e tudo que envolve cloud.