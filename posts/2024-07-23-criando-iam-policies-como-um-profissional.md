---
image: /assets/img/AWS.png
title: Criando IAM policies como um profissional - Parte 1
description: Neste artigo, exploraremos as melhores práticas e técnicas
  avançadas para a criação de políticas IAM. Aprenda a proteger recursos
  críticos e a gerenciar permissões com eficiência, garantindo a segurança e a
  conformidade da sua infraestrutura em nuvem como um verdadeiro profissional.
date: 2024-07-23
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
Revise regularmente suas políticas IAM para garantir que ainda atendam aos requisitos de segurança e de negócios. Revise e ajuste as permissões conforme as necessidades da equipe e da organização mudem.

7. Segregação de Funções
Separe as funções administrativas das funções operacionais. Por exemplo, não conceda permissões de administração de contas a usuários que apenas precisam acessar recursos específicos.

8. Monitore e Audite Atividades
Ative o CloudTrail para registrar e monitorar as atividades dos usuários. Isso ajuda a detectar e responder a atividades suspeitas rapidamente.

9. Evite Políticas Inline
Prefira políticas gerenciadas a políticas inline. Políticas gerenciadas são mais fáceis de gerenciar, atualizar e reutilizar entre diferentes usuários e recursos.

10. Limite o Uso de Wildcards
Evite o uso excessivo de caracteres curinga (*) Eles podem abrir permissões mais amplas do que o necessário, comprometendo a segurança dos recursos.

Implementando essas boas práticas, você pode criar políticas IAM mais seguras, eficientes e fáceis de gerenciar

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
               "Action": "s3",
               "Resource": "arn:aws:s3:::example_bucket"
             }
              ]
}
```

