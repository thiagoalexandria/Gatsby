---
image: /assets/img/AWS.png
title: AWS Security Specialty - SCS-C02
description: Passei na prova AWS Certified Security – Specialty! Decidi criar
  esse blog compartilhando minha experiência com insights, estratégias de estudo
  e as principais lições que aprendi ao longo do caminho. Se você está
  considerando fazer essa prova, este post é para você!
date: 2024-11-23
category: sec
background: "#4E457FFF"
tags:
  - AWS
  - SECURITY
  - SPECIALTY
  - CERTIFICATION
  - PROVA
  - SEGURANÇA
  - NUVEM
  - PREPARAÇÃO
  - ESTRATÉGIAS
  - ESTUDO
  - AWS-CERTIFIED
  - EXPERIÊNCIA
  - DICAS
  - EXAME
  - CARREIRA
  - PROFISSIONAL
  - CLOUD
  - SEGURANÇA-NA-NUVEM
  - CERTIFICAÇÃO
  - AWS-SECURITY
categories:
  - AWS
  - SECURITY
  - SPECIALTY
  - CERTIFICATION
  - PROVA
  - SEGURANÇA
  - NUVEM
  - PREPARAÇÃO
  - ESTRATÉGIAS
  - ESTUDO
  - AWS-CERTIFIED
  - EXPERIÊNCIA
  - DICAS
  - EXAME
  - CARREIRA
  - PROFISSIONAL
  - CLOUD
  - SEGURANÇA-NA-NUVEM
  - CERTIFICAÇÃO
  - AWS-SECURITY
---
## Porque eu decidi fazer a prova? 

Eu decidi fazer a prova **AWS Certified Security Specialty** porque acredito que a segurança na nuvem é uma responsabilidade compartilhada, e cabe a todos os profissionais que projetam, implementam ou mantêm ambientes na AWS garantir que essas soluções sejam seguras e robustas.

Apesar de ser especialista em infraestrutura e não atuar diretamente como analista de segurança, percebo que, em um cenário onde ataques e violações estão cada vez mais sofisticados, desenvolver ambientes seguros e aderentes às boas práticas de segurança deixou de ser uma escolha e passou a ser uma obrigação.

Seja você arquiteto, engenheiro, DevOps ou SRE, entender os fundamentos de segurança, como controle de acesso, proteção de dados e mitigação de riscos, é essencial para entregar soluções confiáveis. A certificação não só valida meu conhecimento técnico, mas também fortalece meu compromisso em construir ambientes resilientes e preparados para o futuro.

![](/assets/img/security-specialty.jpeg)

## Certificação vale a pena?

Certificação em TI sempre divide opiniões. Tem gente que defende, tem quem critique, e dá para entender os argumentos dos dois lados. Mas, como eu fiz a prova, já dá para imaginar que, na minha visão, vale a pena. Para mim, certificação é uma forma de estudar de maneira organizada um conjunto específico de conhecimentos. É perfeita? Longe disso. Ter um certificado garante competência? Também não. E dá para ser um excelente profissional sem certificação? Com certeza, conheço vários exemplos disso.

O que realmente importa na certificação, para mim, não é o pedaço de papel ou o badge digital. É o que você aprende durante a jornada de preparação. O valor da certificação está no que você absorveu e no quanto aquele estudo te fez crescer. Se você estudar só para passar, decorando informações e sem entender os porquês, o ganho será pequeno. Mas, se você se dedicar a entender os conceitos, os motivos e as boas práticas, o valor vai ser enorme e não estou falando de dinheiro, mas sim do impacto no seu desenvolvimento como profissional. Nesse cenário, passar na prova é só uma consequência, e não o objetivo principal.

## Tempo de estudo

Estudei durante aproximadamente três meses, tinha pouco tempo para estudar durante a semana e aproveitei minhas férias em Setembro para maratonar o material de estudo e iniciar no final de outubro os simulados. Lembrando que boa parte do material eu já conhecia por causa da prova AWS Solutions Architect Professional que fiz em Dezembro de 2023. Pode ser que você precise de mais tempo caso esteja vendo o conteúdo pela primeira vez. Além disso, cada pessoa tem um ritmo e outros compromissos que acabam demandando tempo, então isso vai variar.

## Materiais de estudo

Minha principal fonte de estudo foi o curso do **Stephane Maarek**, disponível na **Udemy**. Esse curso é muito bom para quem quer uma visão estruturada e didática dos tópicos cobrados na prova. Além disso, ele costuma entrar em promoção frequentemente, e dá para comprar por cerca de R$30.

Além do curso, comprei também o pacote de simulados do Stephane. Ele oferece apenas dois simulados, e, embora sejam bem feitos, senti que as questões eram mais fáceis e com menos variedade em relação ao que a prova realmente exige. Não considero suficiente se o objetivo é treinar em um nível mais próximo da realidade do exame.

Por isso, decidi investir nos simulados da **Whizlabs**, por US$29. O pacote inclui **cinco simulados com 65 questões cada**, além de vários simulados menores (de 5 a 15 questões) focados em tópicos específicos, como IAM, CloudTrail e KMS. Achei o nível das questões bastante alinhado com o que a prova cobra, tanto em dificuldade quanto em diversidade de serviços abordados. Esse material me ajudou a identificar lacunas no meu conhecimento e a praticar a resolução de questões dentro do tempo limite.

No geral, achei que a combinação do curso do Stephane com os simulados da Whizlabs foi uma estratégia eficiente para me preparar. Cada material complementou o outro, e juntos me deram a confiança necessária para enfrentar a prova.

## Principais assuntos

Durante a realização da prova, percebi que a certificação é bem focada em segurança prática, cobrindo tópicos que vão além do básico e exigindo um entendimento sólido das boas práticas da AWS. Alguns dos principais assuntos que encontrei foram:

1. **IAM e Controle de Acesso**

   * Gestão de usuários, grupos, funções e políticas.
   * Uso de políticas baseadas em identidade (IAM) e recursos.
   * Condições em políticas (como IPs, tags e tempo).
   * Limites de segurança, como o número máximo de chaves por usuário IAM.
2. **Gerenciamento de Logs e Auditoria**

   * Configuração do CloudTrail para monitoramento de atividades na conta.
   * Integração com o **CloudWatch Logs** para análise e monitoramento contínuo.
   * Métricas relacionadas a auditoria.
3. **Proteção de Dados**

   * Uso do **KMS** para gerenciar chaves de criptografia e nesse tópico em especifico tive perguntas sobre as diversas formas de como criar uma chave, importar, rotacionar, gerenciar.
   * Configuração de criptografia em serviços como S3, RDS, EBS, RedShift.
   * Implementação de políticas de ciclo de vida em buckets S3 para atender aos requisitos de conformidade.
4. **Resiliência e Recuperação**

   * Melhores práticas de backup e as diversas configurações através  do AWS Backup.
   * Estratégias para recuperação de desastres e replicação de dados entre regiões.
5. **Segurança em Ambientes Multi-Conta**

   * Uso de organizações da AWS e SCPs (Service Control Policies).
   * Gerenciamento de contas e boas práticas com o **Control Tower**.
6. **Respostas a Incidentes e Investigação**

   * Estratégias para identificar e mitigar falhas de segurança.
   * Uso de ferramentas como **GuardDuty**, **AWS Config** e **CloudTrail Insights**.
7. **Segurança de Aplicações e Redes**

   * Configuração de segurança em VPCs (ACLs, SGs e endpoints privados).
   * Proteção contra ataques DDoS com o **AWS Shield** e WAF.

A prova não só aborda esses tópicos individualmente, mas também testa a capacidade de integrá-los em cenários reais, exigindo decisões baseadas em melhores práticas. Ter uma base sólida nesses assuntos foi essencial para me sentir preparado.

## Conclusão

Por se tratar de uma prova de especialidade, focada em segurança, o escopo é mais limitado, o que resulta em menos conteúdo para estudar em comparação com a **AWS Solutions Architect Professional**. Se você já fez essa última prova, vai perceber que boa parte do que estudou pode ser reaproveitada, o que é uma grande vantagem.

Vale lembrar que o método de estudo que funcionou para mim pode não funcionar da mesma forma para você. Cada pessoa tem seu próprio ritmo e preferências de aprendizado. Espero que este plano de estudo seja útil e contribua, de alguma forma, para sua preparação. Bons estudos e boa sorte na sua jornada!