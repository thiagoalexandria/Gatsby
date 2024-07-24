---
image: /assets/img/AWS.png
title: Criando IAM policies como um profissional - Parte 1
description: Neste artigo, exploraremos as melhores práticas e técnicas
  avançadas para a criação de políticas IAM. Aprenda a proteger recursos
  críticos e a gerenciar permissões com eficiência, garantindo a segurança e a
  conformidade da sua infraestrutura em nuvem como um verdadeiro profissional.
date: 2024-07-26
category: aws
background: "#FF9900"
tags:
  - AWS
  - IAM
  - POLICIES
  - SEGURANCA
  - ACESSO
  - PERMISSOES
  - NUVEM
  - INFRAESTRUTURA
  - GERENCIAMENTO
  - TECNICAS
  - PROFISSIONAL
categories:
  - AWS
  - IAM
  - POLICIES
  - SEGURANCA
  - ACESSO
  - PERMISSOES
  - NUVEM
  - INFRAESTRUTURA
  - GERENCIAMENTO
  - TECNICAS
  - PROFISSIONAL
---

Se aprofundar na criação e gerenciamento de políticas IAM é obrigatório para sua infraestrutura mais segura e colaborativa. Mas vamos ser sinceros, configurar políticas com várias instruções, condições e valores-chave pode ser um baita desafio, especialmente quando entram condições de negação na jogada. 

Neste artigo, vamos simplificar esses conceitos e mostrar casos de uso reais para que a criação de políticas não seja mais um bicho de sete cabeças independente da sua função na equipe.

## Introdução às Políticas IAM

Criar políticas IAM bem estruturadas é essencial para manter a segurança e a eficiência da sua infraestrutura em nuvem. Não existe uma receita mágica para isso, mas seguir boas práticas faz toda a diferença na hora de proteger seus recursos e gerenciar permissões. E olha só, entender como isso funciona não é só trabalho da galera de segurança, não! Esse conhecimento é super útil para todas as equipes que lidam com recursos na AWS, já que muitos desses recursos têm suas próprias resource policies.

### Boas Práticas na Criação de Políticas IAM
Criar políticas IAM de forma eficaz é fundamental para garantir a segurança e o gerenciamento adequado dos recursos na AWS. Aqui estão algumas boas práticas que podem ajudar:

1. Princípio do Menor Privilégio
Conceda apenas as permissões necessárias para que os usuários realizem suas tarefas. Evite permissões excessivas que possam abrir brechas de segurança.

2. Use Políticas Gerenciadas pela AWS
Sempre que possível, utilize as políticas gerenciadas pela AWS. Elas são mantidas e atualizadas pela AWS, garantindo que estejam alinhadas com as melhores práticas de segurança.

3. Agrupe Permissões Comuns
Em vez de conceder permissões individualmente, crie grupos de permissões. Isso facilita a administração e garante que os usuários recebam as permissões necessárias de maneira eficiente.

4. Documente Suas Políticas
Adicione descrições e comentários às suas políticas para que outras pessoas da equipe possam entender facilmente o propósito e o funcionamento de cada política.

5. Teste Suas Políticas
Use ferramentas como o AWS IAM Policy Simulator para testar suas políticas antes de aplicá-las em produção. Isso ajuda a garantir que as permissões funcionem conforme esperado e que não haja permissões excessivas ou faltantes.

6. Revisão Periódica
Revise regularmente suas políticas IAM, utilize o IAM Access Analyzer, uma ferramenta poderosa para auxiliar a identificar politicas com Over Permissions, além de informar se as roles estão em uso para que seja passível de limpeza.

7. Segregação de Funções
Separe as funções administrativas das funções operacionais. Por exemplo, não conceda permissões de administração de contas a usuários que apenas precisam acessar recursos específicos.

8. Monitore e Audite Atividades
Ative o CloudTrail para registrar e monitorar as atividades dos usuários. Isso ajuda a detectar e responder a atividades suspeitas rapidamente.

9. Evite Políticas Inline
Prefira políticas gerenciadas a políticas inline. Políticas gerenciadas são mais fáceis de gerenciar, atualizar e reutilizar entre diferentes usuários e recursos.

10. Limite o Uso de Wildcards
Evite o uso excessivo de caracteres curinga (*) Eles podem abrir permissões mais amplas do que o necessário, comprometendo a segurança dos recursos.

Antes de iniciarmos, é necessário que saibamos toda a estrutura de uma policy, seus componentes e os tipos de políticas que podem ser desenvolvida.

## 2. Estrutura de uma Policy 

### Componentes de uma política IAM

- **Statements:** Declarações que definem o que é permitido ou negado.
- **Actions:** As ações que estão sendo permitidas ou negadas (por exemplo, `s3:PutObject`).
- **Resources:** Os recursos aos quais a política se aplica (por exemplo, um bucket S3 específico, uma ec2, um RDS).
- **Conditions:** Condições adicionais que devem ser atendidas para que a ação seja permitida ou negada.

### Exemplo de política básica

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example_bucket"
    }
  ]
}

```

## 3. Tipos de Políticas na AWS

### Políticas IAM

Políticas anexadas a usuários, grupos ou roles IAM. Elas definem permissões que são aplicadas em uma base global para os recursos da AWS.

### Resource Policies

Políticas anexadas diretamente a recursos da AWS, como buckets S3, KMS, SQS. Elas definem quem pode acessar esses recursos e de que maneira. Por exemplo

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::example_bucket/sensitive_data/*"
    }
  ]
}
```

Esta declaração nega a qualquer pessoa a permissão de excluir objetos no diretório `sensitive_data` do bucket `example_bucket`

### Service Control Policies (SCP)

Políticas aplicadas a contas da AWS organizadas no AWS Organizations. Elas restringem as permissões máximas que as contas podem conceder.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "s3:DeleteBucket",
      "Resource": "*"
    }
  ]
}
```

Esta declaração nega a permissão de excluir qualquer bucket S3 em todas as contas dentro da organização.

### Permissions Boundaries

Permissions Boundaries são uma forma de definir limites de permissões para roles ou usuários no IAM da AWS. Elas funcionam como um "limite máximo" que uma política de permissões pode conceder. Enquanto as políticas IAM comuns definem o que um usuário ou role pode fazer, os boundaries definem o que esses usuários ou roles podem conceder a si mesmos ou a outros.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": "*"
    }
  ]
}
```

Esta politica permite listar qualquer bucket S3 e obter objetos de qualquer bucket S3, desde que a role ou o usuário tenha essa política anexada como uma boundary. No entanto, mesmo que a role tenha permissões adicionais na política principal, essas permissões não podem exceder as definidas aqui.


## 4. Ordem de Permissão

### Como a AWS avalia permissões

A AWS avalia permissões seguindo uma ordem específica:
1. **Negação explícita:** Se uma política nega explicitamente uma ação, a ação será negada.
2. **Permissão explícita:** Se uma política permite explicitamente uma ação, a ação será permitida, a menos que exista uma negação explícita.
3. **Negação padrão:** Por padrão, todas as ações são negadas, a menos que sejam explicitamente permitidas.

### Entendendo Permissões Cumulativas

Na AWS, as permissões são cumulativas. Isso significa que, quando um usuário, grupo ou role tem múltiplas políticas associadas, as permissões resultantes são a soma das permissões concedidas por todas essas políticas. Se uma política permite uma ação e outra política também permite a mesma ação, o usuário terá permissão para realizar essa ação.

### Exemplos de Permissões Cumulativas

Imagine que você tem um usuário com duas políticas anexadas:

1. **Política A**:
    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:ListBucket",
          "Resource": "arn:aws:s3:::example_bucket"
        }
      ]
    }
    ```

2. **Política B**:
    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::example_bucket/*"
        }
      ]
    }
    ```

Nesse caso, o usuário terá permissão tanto para listar o bucket `example_bucket` quanto para obter objetos dentro desse bucket.

### Lidando com Conflitos

Enquanto as permissões são cumulativas, os conflitos de permissões são resolvidos de forma que as negações explícitas prevaleçam sobre permissões explícitas. Se qualquer política anexada a um usuário ou grupo nega explicitamente uma ação, essa negação terá prioridade sobre qualquer permissão que permita essa ação.

### Exemplo de Conflito

Considere um usuário com as seguintes políticas anexadas:

1. **Política C** (Permitir):
    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:DeleteObject",
          "Resource": "arn:aws:s3:::example_bucket/*"
        }
      ]
    }
    ```

2. **Política D** (Resource Policy Negar) :
    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Deny",
          "Action": "s3:DeleteObject",
          "Resource": "arn:aws:s3:::example_bucket/sensitive_data/*"
        }
      ]
    }
    ```

Aqui, a Política C permite que o usuário exclua objetos em qualquer local do bucket `example_bucket`, mas o bucket possui uma `Resource Policy`, a Política D, que nega especificamente a exclusão de objetos no diretório `sensitive_data` dentro do bucket. A negação explícita na Política D prevalece, então o usuário não poderá excluir objetos em `example_bucket/sensitive_data`, mesmo que a Política C permita essa ação para outras partes do bucket.

### Princípios Importantes

1. **Negação Explícita Tem Prioridade**: Se uma política negar explicitamente uma ação, essa negação prevalecerá, mesmo que outras políticas permitam a ação.
2. **Permissões Implícitas São Negadas**: Qualquer ação não explicitamente permitida é implicitamente negada. Portanto, você deve ser explícito sobre todas as ações que deseja permitir.
3. **Avaliando Todas as Políticas**: A AWS avalia todas as políticas anexadas a um usuário ou grupo ao decidir permitir ou negar uma ação. Isso inclui políticas IAM, resource policies, SCPs e permissions boundaries.

## Fechamento

Nesta primeira parte, abordamos boas práticas na criação de políticas IAM, a estrutura de uma policy, os diferentes tipos de políticas na AWS, e como a ordem de permissão funciona. 

No próximo post vamos entender como criar políticas mais sofisticadas utilizando conditions e deixando-as mais seguras e com escopos bem definidos.
