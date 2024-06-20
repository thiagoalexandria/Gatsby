---
image: /assets/img/AWS.png
title: Configurando external-dns no EKS utilizando R53 como provider DNS
description: Aprenda a configurar o external-dns no EKS usando R53 como provedor
  DNS. Automatize a atualização dos registros DNS no R53 à medida que implanta e
  remove serviços no EKS. Simplifique a gestão de domínios e mantenha tudo em
  sincronia com external-dns e R53.
date: 2024-04-22
category: devops
background: "#05A6F0"
tags:
  - eks
  - external-dns
  - r53
  - dns
  - configuracao
  - automatizacao
  - gestao
  - sincronia
categories:
  - eks
  - external-dns
  - r53
  - dns
  - configuracao
  - automatizacao
  - gestao
  - sincronia
---
Vamos supor que você tenha uma aplicação sendo executada em um Cluster Kubernetes, você precisa disponibilizar o acesso a essa aplicação através de uma load balancer, seja pública ou interna. 

Agora, e se você tiver centenas de projetos e vários registros DNS para serem criados? E se o endpoint da load balancer mudar? Como você acompanharia esse gerenciamento massivo de registros DNS?

É ai que entra o external-dns, ele permite que você controle registros DNS de forma dinâmica por meio de recursos do Kubernetes de uma maneira independente do provedor de DNS. O ExternalDNS sincroniza os Serviços e Ingresses do Kubernetes expostos com os provedores de DNS, gerenciando apenas as entradas criadas por ele e sem interferir nas entradas já existentes no ambiente.

### 1﻿. Prepare as permissões no IAM

Neste tutorial, esperamos que você já tenha configurado a zona no seu Route 53, por se tratar de uma interação direta com o R53, precisaremos criar uma política que contenha as permissões necessárias para gerenciar nossa zona DNS.

É importante ter em mente o princípio do Privilégio Mínimo, assim entregaremos apenas as permissões necessárias para gerenciar as entradas DNS.