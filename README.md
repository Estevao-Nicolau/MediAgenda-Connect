# MediAgenda Connect

Plataforma de agendamento médico com integração desacoplada com operadoras de saúde, construída sobre AWS.

---

## O problema

Quem já tentou marcar uma consulta médica usando plano de saúde sabe como a experiência pode ser frustrante. Você liga para a clínica, a secretária precisa verificar o convênio, mas a operadora só responde depois. Você liga de novo. Às vezes descobre no dia da consulta que o plano está inativo ou que a procedimento não tem cobertura.

Esse atrito não é culpa da clínica, nem da secretária, nem da operadora — é falta de integração entre as partes. E esse problema se repete em milhares de clínicas no Brasil todos os dias.

Do lado da clínica, o impacto é direto: horários ociosos por falta ou cancelamento de última hora, equipe consumida por retrabalho manual e nenhuma visibilidade sobre o que está acontecendo na agenda.

---

## A solução

O MediAgenda Connect resolve isso em três frentes:

**Digitaliza a jornada**: paciente agenda, confirma, cancela ou remarca pelo portal ou app, sem depender de ligação.

**Antecipa as validações**: elegibilidade, cobertura e autorização são verificadas antes do dia da consulta, não no balcão.

**Reaproveit cancelamentos**: quando um horário é cancelado, a fila de espera é acionada automaticamente. O próximo paciente recebe um convite e tem 30 minutos para aceitar o encaixe.

---

## Arquitetura

A decisão central de design foi proteger o núcleo de agendamento da complexidade de integração com operadoras. O core cuida da jornada da consulta. Uma camada separada, com adapters isolados por protocolo, cuida de conversar com cada operadora no formato que ela entende — REST/FHIR, TISS/XML ou API proprietária — sem que isso afete nada do fluxo principal.

Os diagramas estão organizados na pasta [`/docs`](./docs), do mais geral para o mais específico:

| Arquivo | O que mostra |
|---|---|
| [01 — Arquitetura de Solução](./docs/01-arquitetura-solucao.md) | Visão completa da plataforma e seus blocos |
| [02 — Arquitetura de TI e Operadoras](./docs/02-arquitetura-ti-operadoras.md) | Camadas técnicas e integração desacoplada |
| [03 — Canais e Entrada Segura](./docs/03-camada-canais-entrada-segura.md) | Borda, autenticação e proteção web |
| [04 — Core de Agendamento](./docs/04-camada-core-agendamento.md) | Serviços de negócio e orquestração |
| [05 — Integração com Operadoras](./docs/05-camada-integracao-operadoras.md) | Adapters, modelo canônico e resiliência |
| [06 — Eventos, Filas e Resiliência](./docs/06-camada-eventos-filas-resiliencia.md) | Processamento assíncrono e tratamento de falhas |
| [07 — Dados, Indicadores e Analytics](./docs/07-camada-dados-analytics.md) | Persistência transacional e analítica |
| [08 — Segurança, LGPD e Observabilidade](./docs/08-camada-seguranca-lgpd.md) | Controles transversais de segurança |
| [09 — C4 Nível 1: Contexto](./docs/09-c4-contexto.md) | Sistema e atores externos |
| [10 — C4 Nível 2: Containers](./docs/10-c4-containers.md) | Componentes internos e protocolos |
| [11 — Arquitetura AWS](./docs/11-arquitetura-aws.md) | Topologia de infraestrutura e deployment |
| [12 — BPMN: Agendamento](./docs/12-bpmn-agendamento.md) | Fluxo de agendamento com validação de convênio |
| [13 — BPMN: Cancelamento](./docs/13-bpmn-cancelamento.md) | Cancelamento e reaproveitamento de slot |

---

## AWS

AWS Lambda · ECS Fargate · Step Functions · API Gateway · EventBridge · SQS · SNS · Pinpoint · RDS PostgreSQL · ElastiCache Redis · S3 · Glue · Athena · QuickSight · CloudFront · WAF · Cognito · KMS · Secrets Manager · CloudWatch · CloudTrail · X-Ray

Integração externa: REST/FHIR R4 · TISS/SOAP XML

---

## Roadmap

O projeto foi pensado para ser construído em fases, começando pelo que entrega valor mais rápido e avançando para o que exige mais maturidade técnica e operacional.

**Fase 1 — MVP:** agenda digital, confirmação automática, lembretes, fila de espera e dashboard básico de ocupação.

**Fase 2 — Integração:** validação de elegibilidade, cobertura e autorização com operadoras de saúde, com retentativas e auditoria.

**Fase 3 — Otimização:** predição de risco de falta com machine learning, relatórios avançados e integração com prontuário eletrônico.
