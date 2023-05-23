---
image: /assets/img/AWS.png
title: Configurando external-dns no EKS utilizando CloudFlare como provider DNS
description: Aprenda a configurar o external-dns no EKS usando CloudFlare como
  provedor DNS. Automatize a atualização dos registros DNS no CloudFlare à
  medida que implanta e remove serviços no EKS. Simplifique a gestão de domínios
  e mantenha tudo em sincronia com external-dns e CloudFlare.
date: 2023-05-22
category: devops
background: "#05A6F0"
tags:
  - eks
  - external-dns
  - cloudflare
  - dns
  - configuracao
  - automatizacao
  - gestao
  - sincronia
categories:
  - eks
  - external-dns
  - cloudflare
  - dns
  - configuracao
  - automatizacao
  - gestao
  - sincronia
---
Vamos supor que você tenha uma aplicação sendo executada em um Cluster Kubernetes, você precisa disponibilizar o acesso a essa aplicação através de uma load balancer, seja pública ou interna. 

Agora, e se você tiver centenas de projetos e vários registros DNS para serem criados? E se o endpoint da load balancer mudar? Como você acompanharia esse gerenciamento massivo de registros DNS?

É ai que entra o external-dns, ele permite que você controle registros DNS de forma dinâmica por meio de recursos do Kubernetes de uma maneira independente do provedor de DNS. O ExternalDNS sincroniza os Serviços e Ingresses do Kubernetes expostos com os provedores de DNS, gerenciando apenas as entradas criadas por ele e sem interferir nas entradas já existentes no ambiente.