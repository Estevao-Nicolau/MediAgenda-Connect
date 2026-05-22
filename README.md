# MediAgenda Connect

> Plataforma inteligente de agendamento médico, redução de faltas e integração com operadoras de saúde.

**Case técnico de arquitetura cloud · C4 Model · BPMN · AWS**

---

## Sobre o projeto

Este repositório documenta a arquitetura de solução do **MediAgenda Connect** — uma plataforma desenhada para transformar a jornada de agendamento médico de um processo manual e fragmentado em um fluxo digital, integrado e rastreável.

A solução resolve cinco problemas reais enfrentados por clínicas no Brasil:

- Agendamento e confirmação ainda dependem de ligações e planilhas
- Validação de convênio acontece tarde — no dia da consulta, quando o atrito já está instaurado
- Pacientes faltam ou cancelam em cima da hora, deixando horários ociosos sem reaproveitamento
- Equipe de recepção gasta tempo em retrabalho que poderia ser automatizado
- Gestores sem visibilidade de ocupação, faltas e pendências de convênio em tempo real

**Resultado esperado:** menos retrabalho, menos horário ocioso e mais previsibilidade operacional para clínica e paciente.

---

## Diferencial arquitetural

O diferencial da solução não é apenas marcar consulta.

A plataforma **desacopla o core de agendamento da complexidade de integração com operadoras**, usando uma camada de adapters isolados por padrão de protocolo (REST/FHIR, TISS/XML, APIs proprietárias), filas assíncronas, auditoria e retentativas.

Se uma operadora usa TISS/XML e outra usa API REST, o fluxo interno da plataforma não muda — apenas o adapter de comunicação.

---

## Arquitetura em camadas

```
┌─────────────────────────────────────────────────────────────────┐
│                    Canais e Perfis de Acesso                    │
│  Paciente (Web/App) · Recepção · Médico · Gestor (Dashboard)    │
└──────────────────────────────┬──────────────────────────────────┘
                               │ HTTPS
┌──────────────────────────────▼──────────────────────────────────┐
│               Borda, Segurança e Entrada de APIs                │
│       CloudFront + WAF · Cognito/IAM · API Gateway · TLS        │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│         Núcleo do Sistema de Agendamento (core isolado)         │
│  Agendamento · Elegibilidade · Cobertura/Autorização            │
│  Fila de Espera · Indicadores · Step Functions (orquestração)   │
└──────┬────────────────────────────────────────────┬────────────┘
       │ eventos                                     │ comandos de integração
┌──────▼────────────────┐          ┌─────────────────▼────────────┐
│ Orquestração e Filas  │          │  Camada de Integração         │
│ Step Functions        │          │  Integration Facade           │
│ EventBridge           │          │  Adapter REST/FHIR            │
│ SQS Integração        │          │  Adapter TISS/XML             │
│ SQS Notificações      │          │  Adapter Proprietário         │
│ DLQ + Retry           │          │  SQS + Retry + DLQ            │
└──────┬────────────────┘          └─────────────────┬────────────┘
       │                                             │
┌──────▼─────────────────────────────────────────────▼────────────┐
│              Dados, Segurança, Auditoria e Observabilidade       │
│  RDS PostgreSQL · DynamoDB · S3 Data Lake · QuickSight          │
│  KMS · Secrets Manager · IAM · CloudTrail · CloudWatch · X-Ray  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Stack de tecnologias

| Camada | Tecnologias |
|---|---|
| Frontend / CDN | Amazon S3, Amazon CloudFront |
| Proteção de borda | AWS WAF |
| Autenticação | Amazon Cognito, AWS IAM |
| API | Amazon API Gateway |
| Orquestração | AWS Step Functions |
| Compute | AWS Lambda, Amazon ECS Fargate |
| Integração | Amazon SQS, Amazon EventBridge |
| Cache | Amazon ElastiCache (Redis) |
| Banco transacional | Amazon RDS PostgreSQL (Multi-AZ) |
| Notificações | Amazon SNS, Amazon Pinpoint |
| Analytics | Amazon S3 (Data Lake), AWS Glue, Amazon Athena, Amazon QuickSight |
| Segurança | AWS KMS, AWS Secrets Manager |
| Observabilidade | Amazon CloudWatch, AWS CloudTrail, AWS X-Ray |
| Integração externa | REST/FHIR R4, TISS/SOAP XML |

---

## Processos mapeados (BPMN)

### Processo 1 — Agendamento com Validação de Convênio

Fluxo principal: desde a solicitação do paciente até confirmação ou rejeição pelo convênio.

```
Paciente agenda → Verificação de disponibilidade (lock 10 min)
→ Validação de elegibilidade na operadora
→ Solicitação de autorização prévia
→ [Aprovado] Confirma agendamento + notificação
→ [Pendente] Recepção resolve em 48h
→ [Negado] Cancela + notifica paciente
→ [Slot ocupado] Oferece fila de espera FIFO
```

### Processo 2 — Cancelamento e Reaproveitamento de Slot

Gerenciamento de cancelamentos com ativação automática da fila de espera.

```
Paciente cancela → Libera slot → Publica evento SlotLiberado
→ Verifica fila de espera (FIFO)
→ [Há candidatos] Notifica 1º da fila (30 min para aceitar)
  → [Aceita] Confirma encaixe
  → [Recusa/Timeout] Avança para próximo candidato
→ [Sem candidatos] Slot livre para novos agendamentos
```

---

## C4 Model

O projeto foi documentado seguindo o C4 Model em dois níveis:

**Nível 1 — Contexto do Sistema:** relação entre a plataforma e seus atores externos (pacientes, clínica, operadoras de saúde, provedor de notificações e IdP).

**Nível 2 — Containers:** todos os containers AWS e suas responsabilidades, com os protocolos de comunicação entre eles.

Os diagramas completos estão na pasta [`/docs/architecture`](./docs/architecture).

---

## Integração com operadoras de saúde

A plataforma suporta três padrões de integração:

| Operadora | Protocolo | SLA |
|---|---|---|
| Operadora A | REST / FHIR R4 | 99,5% |
| Operadora B | TISS / SOAP XML | 99,0% |
| Operadora C | API legada / proprietária | 98,0% |

Cada operadora tem um **adapter isolado**. O core de agendamento trabalha apenas com comandos internos (`validar elegibilidade`, `solicitar autorização`) — nunca com detalhes técnicos de protocolo.

Falhas são tratadas com **retry exponencial + DLQ**, garantindo que instabilidade em uma operadora não afeta o fluxo do paciente nem as demais operadoras.

---

## Indicadores (KPIs)

| KPI | Objetivo |
|---|---|
| Taxa de faltas | ↓ Reduzir |
| Taxa de ocupação da agenda | ↑ Aumentar |
| Tempo médio de confirmação | ↓ Reduzir |
| Horários reaproveitados (fila de espera) | ↑ Aumentar |
| Pendências de convênio no dia da consulta | ↓ Reduzir |
| SLA de integração com operadoras | Monitorar disponibilidade por adapter |

---

## Roadmap

| Fase | Escopo |
|---|---|
| **Fase 1 · MVP** | Agenda digital, confirmação e lembretes, fila de espera, dashboard básico |
| **Fase 2 · Integração** | Elegibilidade, cobertura e autorização com operadoras, retentativas e auditoria |
| **Fase 3 · Otimização** | Predição de faltas (ML), relatórios avançados, integração com prontuário |

---

## Segurança e LGPD

A arquitetura trata segurança e privacidade como capacidades transversais desde o design — não como adendo.

- **Criptografia em trânsito:** TLS em todas as comunicações
- **Criptografia em repouso:** AWS KMS para dados sensíveis (RDS, S3, backups)
- **Acesso mínimo:** IAM com menor privilégio por serviço e função
- **Segredos:** credenciais de operadoras isoladas por operadora no Secrets Manager
- **Auditoria técnica:** CloudTrail para ações administrativas na AWS
- **Auditoria de negócio:** Audit Log customizado para eventos do domínio (quem validou, qual resposta da operadora, qual status da autorização)
- **Observabilidade:** CloudWatch + X-Ray para logs, métricas, tracing e alarmes

---

## Estrutura do repositório

```
/
├── README.md
├── docs/
│   ├── architecture/
│   │   ├── c4-context.drawio
│   │   ├── c4-containers.drawio
│   │   └── aws-deployment.drawio
│   ├── bpmn/
│   │   ├── processo-1-agendamento.bpmn
│   │   └── processo-2-cancelamento.bpmn
│   ├── adr/
│   │   ├── ADR-001-core-desacoplado-das-operadoras.md
│   │   ├── ADR-002-integration-facade-adapters.md
│   │   ├── ADR-003-processamento-assincrono-sqs.md
│   │   └── ADR-004-rds-transacional-s3-analitico.md
│   └── analysis/
│       └── MediAgenda_Connect_Analise_Arquitetura.md
└── presentation/
    └── MediAgenda_Connect_Apresentacao_Completa.pptx
```

---

## Decisões arquiteturais

As principais decisões de arquitetura estão documentadas como ADRs (Architecture Decision Records) na pasta [`/docs/adr`](./docs/adr).

Resumo das decisões-chave:

1. **Core desacoplado das operadoras** — o agendamento não depende do formato técnico de cada operadora
2. **Integration Facade + Adapters** — suporte a REST/FHIR, TISS/XML e APIs proprietárias sem reescrever o core
3. **Processamento assíncrono** — SQS e EventBridge evitam travar o fluxo do paciente quando uma operadora falha
4. **Retentativas e DLQ** — falhas transitórias são reprocessadas; falhas finais ficam rastreáveis
5. **RDS para transacional, S3 para analítico** — separação que evita queries pesadas impactando o banco operacional
6. **Segurança by design** — KMS, Secrets Manager, IAM, CloudTrail e WAF desde o início

---

*Case técnico de arquitetura — MediAgenda Connect*
