---
image: /assets/img/AWS.png
title: AWS IAM Identity Center Pipeline
description: O AWS IAM Identity Center, sucessor do AWS Single Sign-On, ajuda
  você a criar ou conectar com segurança suas identidades de força de trabalho e
  gerenciar seu acesso centralmente em contas e aplicativos da AWS
date: 2025-05-12
category: aws
background: "#FF9900"
tags:
  - AWS
  - IAM
  - SECURITY
  - PIPELINE
  - TERRAFORM
  - SSO
  - DEVOPS
categories:
  - AWS
  - IAM
  - SECURITY
  - PIPELINE
  - TERRAFORM
  - SSO
  - DEVOPS
---
O IAM Identity Center é a abordagem recomendada para autenticação e autorização de força de trabalho na AWS para organizações de qualquer tamanho e tipo. As empresas podem trazer suas identidades do Microsoft Active Directory para atribuí-las a contas da AWS com permissões específicas. Com o AWS Identity Center, você obtém uma experiência de administração unificada para definir, personalizar e atribuir acesso refinado.

Este padrão ajuda você a gerenciar a permissão do AWS IAM Identity Center no seu ambiente de múltiplas contas como um código. Com este padrão, você conseguirá obter o seguinte definido como código:

* Create, delete e update permission sets
* Create, update ou delete assignments do seu permission sets com seu destino (contas da AWS ou unidades organizacionais da AWS) com seus usuários federados do seu repositório de identidades do AWS IAM Identity Center (por exemplo, Microsoft Active Directory)

Para gerenciar suas permissões a equipe da Amazon desenvolveu um projeto chamado **aws-iam-identity-center-pipeline**, esta solução implantará um pipeline com serviços da AWS (AWS CodeCommit, AWS CodeBuild e AWS CodePipeline). A pipeline será acionado sempre que alguém confirmar alterações em arquivos específicos no repositório ou sempre que uma conta da AWS for movida de sua Unidade Organizacional da AWS. Os gatilhos são implementados usando regras do AWS EventBridge com base em padrões e uma função do AWS Lambda.

Originalmente a pipeline realiza apenas a criação de permission sets em grupos ja existentes, então realizei um fork para incrementar fluxos que permitam a criação de um grupo antes de realizarmos a criação  das permission sets, tornando a pipeline 100% autônoma.