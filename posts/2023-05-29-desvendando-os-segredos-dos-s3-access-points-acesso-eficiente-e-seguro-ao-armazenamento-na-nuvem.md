---
image: /assets/img/AWS.png
title: Desvendando os Segredos dos S3 Access Points Acesso Eficiente e Seguro ao
  Armazenamento na Nuvem
description: "Nessa postagem exploraremos um dos recursos mais poderosos do
  Amazon S3: os S3 Access Points, ele pode revolucionar a maneira como você
  gerencia o acesso e a segurança aos seus dados armazenados na nuvem"
date: 2023-06-02
category: aws
background: "#FF9900"
tags:
  - s3-access-points
  - armazenamento
  - cloud
  - aws
  - seguranca
  - acesso
  - controle
  - politicas
  - simplificacao
  - governanga
  - urls
  - granularidade
  - monitoramento
  - metricas
  - dados
  - endpoint
  - recurso
  - gerenciamento
  - s3
categories:
  - s3-access-points
  - armazenamento
  - cloud
  - aws
  - seguranca
  - acesso
  - controle
  - politicas
  - simplificacao
  - governanga
  - urls
  - granularidade
  - monitoramento
  - metricas
  - dados
  - endpoint
  - recurso
  - gerenciamento
  - s3
---
## O que são S3 Access Points?

Os pontos de acesso S3 são endpoints personalizados que fornecem acesso simplificado e seguro ao armazenamento no Amazon S3. Eles permitem que você controle o acesso granular aos seus dados, definindo políticas de acesso específicas para diferentes usuários, grupos ou aplicativos. Essa abordagem simplifica a organização e a segmentação do acesso aos buckets do S3, fornecendo uma camada adicional de segurança.

![](/assets/img/s3-access-points.png)

## Benefícios dos S3 Access Points

* `A simplificação da gestão do acesso do S3`: Os pontos de acesso do S3 permitem que você controle de forma centralizada as políticas de acesso aos seus buckets do S3. Isso torna o controle de acesso mais fácil de gerenciar e também mais simples, especialmente em ambientes onde há várias equipes e aplicativos.
* `Acesso granular`: Os Pontos de Acesso S3 permitem que você crie regras de acesso exclusivas para cada endpoint. Isso reduz o risco de exposição indesejada, pois permite que apenas entidades autorizadas interajam con os dados.
* `Personalização da estrutura de URLs`: Você pode simplificar os URLs de acesso aos seus objetos usando pontos de acesso S3. Você pode usar um formato mais fácil de entender e lembrar, como `accesspoint.example.com/objectkey`, em vez de URLs com a estrutura de nome de bucket padrão.
* `Monitoramento`: Os S3 Access Points fornecem logs detalhados e métricas de acesso, permitindo que você acompanhe e audite facilmente as atividades realizadas em cada ponto de acesso. Isso contribui para uma postura de segurança mais robusta e ajuda na detecção de possíveis anomalias ou ameaças.

## Como começar a usar S3 Access Points

Aqui estão alguns passos para começar a utilizar os S3 Access Points:

### Crie um S3 Access Point

No Console de Gerenciamento da AWS ou por meio da API, você pode criar um novo Access Point para o seu bucket do S3. Iremos realizar a criação através do console de gerenciamento, para isso acesse ou crie o seu bucket s3 e no menu superior clique em `pontos de acesso` ou `access points`:

![](/assets/img/s3-access-points-1.png)

Aqui você notará que tem a opção de escolher o tipo de acesso que será Virtual Private Network se quiser restringir seu ponto de acesso a uma VPC ou Internet se quiser que usuários fora da sua VPC também tenham acesso, no nosso laboratório será utilizado internet pois eu mesmo irei acessar:

![](/assets/img/s3-acess-points-2.png)

Defina políticas de acesso, configure as políticas de acesso no Access Point para controlar quem pode acessar os dados e quais ações estão permitidas, o principal pode ser definido igual uma bucket policy, sendo um usuário do IAM, uma role, um serviço, segue um modelo abaixo:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAccessToAccessPoint",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<conta>:user/user"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:<região>:<conta>:accesspoint/<nome-do-access-point>/object/<prefixo>/*"
    }
  ]
}
```

![](/assets/img/s3-acess-points-3.png)

﻿Garanta que as mesmas permissões constem na bucket policy, feito isso, agora, você pode acessar os dados no seu bucket do S3 por meio do Access Point, usando o URL personalizado ou através do aws cli no seguinte formato:

```
aws s3 cp <caminho_do_arquivo> s3://<nome_do_access_point>/<caminho_no_bucket>

```

## Conclusão

Os S3 Access Points são uma ferramenta poderosa para simplificar o gerenciamento de acesso e aumentar a segurança ao utilizar o Amazon S3. Ao permitir um controle granular e centralizado do acesso aos dados, os S3 Access Points oferecem uma solução robusta e eficaz para o gerenciamento de acesso, sendo mais eficiente e simples de usar que as antigas Buckets ACLs.

E﻿spero que tenham gostado!