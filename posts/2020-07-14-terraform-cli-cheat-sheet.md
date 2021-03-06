---
image: /assets/img/HashiCorp-Terraform-logo.png
title: Terraform CLI Cheat Sheet
description: 'Alguns comandos e definições '
date: '2020-07-14'
category: devops
background: '#05A6F0'
tags:
  - Terraform
categories:
  - Terraform
  - Devops
---
O Terraform possui uma interface por linha de comando bastante poderosa e bem intuitiva, abordaremos dessa forma alguns comandos interessantes e que trará produtividade.



## 1. Formatação

O Terraform possui o comando ` terraform fmt` para garantir que os seus arquivos `.tf` vão estar sempre bem formatados, é possível a partir dai aplicar os ajustes indicados ou liberar apenas de forma visual:

```shell
terraform fmt -recursive -diff
terraform fmt -check -recursive -diff
```

## 2. Validação

O comando `terraform validade` valida os arquivos de configuração em um diretório, referindo-se apenas à configuração e não acessando nenhum serviço remoto, como estado remoto, APIs do provedor etc. Para isso é necessário desativar o nosso backend:

```shell
terraform init -backend=false
terraform validate -json
```

Se tudo estiver correto, será retornado algo semelhante a o trecho abaixo:
```shell
{
  "valid": true,
  "error_count": 0,
  "warning_count": 0,
  "diagnostics": []
}
```

## 3. Workspace

O comando `terraform workspace` permite o trabalho em diferentes áreas de trabalho, abrindo a possibilidade de trabalho em ambientes de testes ou produção. 

- Criar workspace
```shell
terraform workspace new dev
```
- Selecionar workspace 
```shell
terraform workspace select dev
```
- Listar workspace
```shell
terraform workspace list
```

Um exemplo seria a criação de instâncias em regiões diferentes caso o workspace não seja o desejado:

```shell
provider "aws" {
  region  = terraform.workspace == "production" ? "us-east-1" : "us-east-2"
  shared_credentials_file = "/home/thiago/.aws/credentials"
  profile = "default"
}
```

## 4. Graph

Quando se tem um projeto finalizado é possível criar um grafo de todo o fluxo do projeto:

```shell
terraform graph | dot –Tpng > graph.png
```

## 5. Autocomplete
É possível também habilitar a opção de autocomplete, pra quem vem do linux é uma mão na roda para evitar repetição de escrita, apenas apertando `Tab`, para habilitar basta rodar o comando de instalação:

```shell
terraform -install-autocomplete
```

Espero que gostem, esses são apenas alguns dos comandos e possibilidades para utilizar juntamente ao Terraform, até a próxima!
