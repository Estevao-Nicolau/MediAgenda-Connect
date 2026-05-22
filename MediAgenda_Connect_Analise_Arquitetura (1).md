# MediAgenda Connect — Análise de Arquitetura (Documentos Consolidados)

> **Objetivo:** Estudo de arquitetura para repositório  
> **Fontes analisadas:**  
> - `Arquitetura_MediAgend0_-_Arquitetura_TI_-_Visão_Geral.drawio.pdf`  
> - `MediAgenda_Connect_Apresentacao_Completa.pptx`  
> **Foco:** Decisões técnicas identificadas + Pontos de atenção / gaps  
> **Nota:** Conteúdo exclusivo do PPT está marcado com 🆕

---

## Sumário

- [Slide 1 — Capa e Posicionamento](#slide-1--capa-e-posicionamento)
- [Slide 2 — Origem do Problema 🆕](#slide-2--origem-do-problema-)
- [Slide 3 — Contexto e Problema de Negócio 🆕](#slide-3--contexto-e-problema-de-negócio-)
- [Slide 4 / Drawio p.13 — Objetivos da Solução](#slide-4--drawio-p13--objetivos-da-solução)
- [Slide 5 / Drawio p.4 — Stakeholders e Canais](#slide-5--drawio-p4--stakeholders-e-canais)
- [Slide 6 — Jornada Proposta 🆕](#slide-6--jornada-proposta-)
- [Slide 7 / Drawio p.53 — BPMN Processo 1: Agendamento com Validação de Convênio](#slide-7--drawio-p53--bpmn-processo-1-agendamento-com-validação-de-convênio)
- [Slide 8 — BPMN Processo 2: Cancelamento e Reaproveitamento de Slot 🆕](#slide-8--bpmn-processo-2-cancelamento-e-reaproveitamento-de-slot-)
- [Slide 9 / Drawio p.5–6 — Arquitetura de Solução](#slide-9--drawio-p56--arquitetura-de-solução)
- [Slide 10 / Drawio p.14 — Arquitetura de TI / Operadoras](#slide-10--drawio-p14--arquitetura-de-ti--operadoras)
- [Slide 11 / Drawio p.41–44 — C4 Nível 1: Contexto do Sistema](#slide-11--drawio-p4144--c4-nível-1-contexto-do-sistema)
- [Slide 12 / Drawio p.47–51 — C4 Nível 2: Containers](#slide-12--drawio-p4751--c4-nível-2-containers)
- [Slide 13 / Drawio p.59–65 — Arquitetura AWS](#slide-13--drawio-p5965--arquitetura-aws)
- [Slide 14 / Drawio p.31–34 — Integração com Operadoras](#slide-14--drawio-p3134--integração-com-operadoras)
- [Slide 15 / Drawio p.39 — Segurança, Auditoria e LGPD](#slide-15--drawio-p39--segurança-auditoria-e-lgpd)
- [Slide 16 — Indicadores de Sucesso / KPIs 🆕](#slide-16--indicadores-de-sucesso--kpis-)
- [Slide 17 — Roadmap de Implementação 🆕](#slide-17--roadmap-de-implementação-)
- [Drawio — Camada 1: Canais e Entrada Segura](#drawio--camada-1-canais-e-entrada-segura)
- [Drawio — Camada 2: Core de Agendamento](#drawio--camada-2-core-de-agendamento)
- [Drawio — Camada 4: Eventos, Filas e Resiliência](#drawio--camada-4-eventos-filas-e-resiliência)
- [Drawio — Camada 5: Dados, Indicadores e Analytics](#drawio--camada-5-dados-indicadores-e-analytics)
- [Drawio — Decisões Arquiteturais para Defender](#drawio--decisões-arquiteturais-para-defender)
- [Resumo Consolidado dos Gaps](#resumo-consolidado-dos-gaps)

---

## Slide 1 — Capa e Posicionamento

**Conteúdo:**
> *"Plataforma inteligente de agendamento médico, redução de faltas e integração com operadoras de saúde. Case técnico · C4 · BPMN · AWS. Objetivo: transformar uma jornada manual em um fluxo digital, integrado e rastreável."*

### Decisões técnicas identificadas
- O posicionamento técnico usa três eixos: **C4 Model** (arquitetura), **BPMN** (processos) e **AWS** (infraestrutura) — cobertura completa de visões arquiteturais.
- A frase "fluxo digital, integrado e rastreável" aponta diretamente para três capacidades técnicas: digitalização (frontend + API), integração (adapters + operadoras) e rastreabilidade (CloudTrail + Audit Log).

### Pontos de atenção / gaps
- Para o repositório, a capa do PPT pode virar o cabeçalho do `README.md` — curta, direta e com as tecnologias explícitas.

---

## Slide 2 — Origem do Problema 🆕

**Conteúdo:** Narrativa pessoal que motivou o projeto — experiência como pai de criança pequena frequentando pediatra, enfrentando atrito no agendamento com convênio.

| Elemento | Descrição |
|---|---|
| Contexto | Pai com filho pequeno, frequentes consultas pediátricas |
| Jornada real | Ligar para clínica → aguardar retorno sobre convênio → ligar de novo |
| Atrito | Secretária sem acesso em tempo real à operadora |
| Percepção | Falta de orquestração entre clínica, paciente e operadora |

**Conclusão do slide:** *"Esse problema existe em milhares de clínicas. E ele tem solução."*

### Decisões técnicas identificadas
- A narrativa pessoal valida que o problema é **real e recorrente**, não hipotético — fortalece a justificativa do projeto para portfólio e entrevistas.
- A percepção de "falta de orquestração" mapeia diretamente para a decisão de usar **AWS Step Functions** como orquestrador central da jornada.

### Pontos de atenção / gaps
- Para o repositório, esse slide pode virar uma seção `## Motivação` no `README.md` — humaniza o projeto e diferencia de um case puramente técnico.

---

## Slide 3 — Contexto e Problema de Negócio 🆕

**Conteúdo:** Cinco problemas estruturais da jornada de agendamento médico.

| Problema | Impacto |
|---|---|
| Processos manuais | Agendamento, confirmação e validação dependem de ligações e planilhas |
| Retrabalho operacional | Equipe aguarda retorno da operadora para dados que poderiam ser antecipados |
| Faltas e cancelamentos | Horários ociosos que a recepção não tem tempo de realocar |
| Convênio descoberto tarde | Plano inativo ou autorização negada aparecem apenas no dia da consulta |
| Impacto na clínica | Perda de receita, produtividade desperdiçada, qualidade comprometida |

**Pergunta-chave do slide:** *"Como eliminar o atrito, antecipar validações e aumentar a ocupação da agenda?"*

### Decisões técnicas identificadas
- Cada problema mapeia para um componente técnico específico:
  - Processos manuais → portal digital (S3 + CloudFront)
  - Retrabalho operacional → elegibilidade antecipada (Lambda + Integration Service)
  - Faltas → lembretes (SNS + Pinpoint + EventBridge)
  - Convênio tarde → autorização prévia no fluxo de agendamento (Step Functions)
  - Horários ociosos → fila de espera ativa (EventBridge + Lambda)

### Pontos de atenção / gaps
- Os problemas estão bem mapeados para soluções técnicas, mas não há **métricas de baseline** — ex.: taxa média de faltas em clínicas brasileiras, custo médio de horário ocioso. Para um portfólio robusto, dados de mercado reforçariam o case.

---

## Slide 4 / Drawio p.13 — Objetivos da Solução

**Conteúdo:** Cinco objetivos com resultado esperado.

| # | Objetivo | Mecanismo técnico |
|---|---|---|
| 1 | Digitalizar o agendamento | Portal web/mobile — S3 + CloudFront |
| 2 | Antecipar validações | Elegibilidade e cobertura antes da consulta — Lambda + Integration Service |
| 3 | Reduzir faltas | Lembretes automáticos + confirmação prévia — SNS + Pinpoint + EventBridge |
| 4 | Reaproveitar cancelamentos | Fila de espera ativa — EventBridge + Lambda |
| 5 | Gerar indicadores | KPIs operacionais — Lambda + QuickSight |

**Resultado esperado:** menos atrito, menos retrabalho e maior previsibilidade para clínica e paciente.

### Decisões técnicas identificadas
- Os objetivos são **rastreáveis para componentes** — sinal de design guiado por requisitos de negócio.
- A fila de espera ativa (objetivo 4) é diferencial de mercado: maioria das clínicas não tem mecanismo automático de reaproveitamento de slots.

### Pontos de atenção / gaps
- Falta definir o **critério de priorização da fila de espera**: FIFO puro? Por urgência? Por tempo de espera? Essa decisão de negócio afeta o modelo de dados e a lógica do Serviço de Fila de Espera.
- Objetivo 2 pressupõe resposta em tempo real das operadoras — operadoras com TISS/XML podem ter latências altas. O impacto na UX (estado "aguardando validação") precisa ser endereçado.

---

## Slide 5 / Drawio p.4 — Stakeholders e Canais

**Conteúdo:** Seis participantes da jornada.

| Stakeholder | Papel |
|---|---|
| Paciente | Agenda, confirma, cancela e recebe lembretes |
| Recepção | Gerencia agenda, pendências e fila de espera |
| Médico | Visualiza agenda e status dos atendimentos |
| Gestor da clínica | Acompanha produtividade, faltas e ocupação |
| Operadora de saúde | Valida elegibilidade, cobertura e autorização |
| Notificação | Envia SMS, e-mail, WhatsApp |

### Decisões técnicas identificadas
- **Notificação como stakeholder explícito:** correto — o canal de notificação é um sistema externo com SLA próprio, não apenas uma feature interna.
- A separação de **Gestor** como stakeholder distinto do Médico justifica perfis diferentes no Cognito com escopos de acesso distintos.

### Pontos de atenção / gaps
- Não há definição do que a **Recepção não pode fazer** — para aplicar menor privilégio, é tão importante quanto o que ela pode.
- Para multi-clínica (futuro), um stakeholder **Administrador da Plataforma** precisaria ser adicionado.
- O paciente se autentica com **login próprio ou link tokenizado**? Ambas as abordagens têm implicações de segurança distintas e precisam ser definidas.

---

## Slide 6 — Jornada Proposta 🆕

**Conteúdo:** Fluxo de 6 etapas do ponto de vista do usuário.

```
1. Paciente agenda → escolhe especialidade, médico e horário
2. Plano validado → elegibilidade e cobertura consultadas
3. Consulta confirmada → sistema registra e notifica paciente
4. Lembretes enviados → paciente confirma, cancela ou não responde
5. Fila de espera → cancelamento libera vaga para outro paciente
6. Indicadores → gestão acompanha faltas e ocupação
```

### Decisões técnicas identificadas
- A jornada é **linear e centrada no paciente** — boa forma de apresentar o fluxo para stakeholders não técnicos.
- O passo 4 ("confirma, cancela ou não responde") implica três estados distintos no sistema — cada um com consequência diferente no Step Functions.
- O passo 5 mostra que **cancelamento dispara automaticamente a fila de espera** — sem intervenção manual da recepção.

### Pontos de atenção / gaps
- O fluxo não mostra o que acontece quando o paciente **não responde** ao lembrete — é registrado como possível falta? É enviado novo lembrete? Essa lógica precisa estar no Step Functions.
- Falta a etapa de **remarcação** — mencionada nos objetivos mas ausente do fluxo visual.

---

## Slide 7 / Drawio p.53 — BPMN Processo 1: Agendamento com Validação de Convênio

**Conteúdo:** Diagrama BPMN completo do fluxo principal — desde a solicitação do paciente até confirmação ou rejeição pelo convênio.

### Participantes (swim lanes)
- Paciente
- Sistema MediAgenda
- Operadora de Saúde
- Recepção / Clínica
- Notificações

### Fluxo mapeado

```
Paciente acessa portal → Seleciona médico/especialidade/data → Informa carteirinha
→ Sistema verifica disponibilidade
  [slot livre?]
  → Não → Oferece fila de espera → [aceita?] → Inclui em FIFO / Desiste
  → Sim → Lock 10 min no slot
→ Valida dados do convênio
→ Consulta elegibilidade na operadora
  [elegível?]
  → Não → Notifica rejeição
  → Sim → Solicita autorização prévia
    [decisão?]
    → Aprovado → Confirma agendamento (CONFIRMADO) → Notificação
    → Pendente → Aguarda 48h → Recepção recebe alerta → [resolvido?]
        → Sim → Confirma manualmente
        → Não → Rejeita e cancela
    → Negado → Marca REJEITADO → Notifica paciente

Timeout: slot liberado após 10 min sem confirmação
```

### Decisões técnicas identificadas
- **Lock de 10 minutos no slot:** evita double-booking durante validação — implementado via ElastiCache Redis com TTL.
- **Fluxo de autorização com prazo de 48h:** SLA explícito com caminho de resolução manual — evita que consultas fiquem em limbo indefinidamente.
- **FIFO na fila de espera:** critério simples e justo para MVP.
- **Estados explícitos do agendamento:** CONFIRMADO, PENDENTE, REJEITADO, NA FILA — mapeiam diretamente para o modelo de dados do RDS.

### Pontos de atenção / gaps
- O timeout de 10 min pode ser curto para pacientes buscando dados do convênio. Considerar valor **configurável por clínica**.
- O fluxo de autorização pendente não define o que acontece se a **recepção não agir em 48h** — automaticamente rejeita? Reabre o prazo? Essa decisão precisa ser explícita.
- O BPMN não cobre o fluxo de **remarcação** (paciente que já tem consulta e quer mudar o horário).
- Não há representação do fluxo de **lembretes (D-1, H-2)** dentro do processo.

---

## Slide 8 — BPMN Processo 2: Cancelamento e Reaproveitamento de Slot 🆕

> ⭐ **Este processo preenche um gap identificado na análise do draw.io**, que não tinha o fluxo de cancelamento documentado.

**Conteúdo:** Diagrama BPMN do processo de cancelamento com ativação automática da fila de espera.

### Participantes (swim lanes)
- Paciente (cancela)
- Sistema MediAgenda
- Paciente de Fila de Espera
- Notificações

### Fluxo mapeado

```
Paciente solicita cancelamento (com ou sem motivo)
→ [Prazo?]
  → ≥ 24h → Cancela com aviso 24h de antecedência
  → < 24h → Cancela com menos de 24h (tardio)

Sistema MediAgenda:
→ Valida solicitação de cancelamento
→ Libera slot na agenda
→ Registra intencional
→ Publica evento SlotLiberado
→ Envia confirmação de cancelamento ao paciente

→ Verifica fila de espera (FIFO)
  [Há candidatos?]
  → Não → Slot fica livre para novos agendamentos (fim)
  → Sim → Bloqueia slot temporariamente
           → Notifica 1º candidato da fila
           → Envia convite de encaixe (30 min para aceitar)

Paciente de Fila de Espera:
→ Recebe notificação
  [Aceita em 30 min?]
  → Sim → Confirma o encaixe → Encaixado (fim)
  → Não/Expirou → Recusa ou não responde
                   → Avisa: tempo expirado, próximo na fila
                   → Sistema notifica próximo da fila
```

### Decisões técnicas identificadas
- **Diferenciação de cancelamento por prazo (≥24h vs <24h):** base para políticas de cobrança ou penalização configuráveis por clínica.
- **Evento SlotLiberado via EventBridge:** padrão correto de event-driven — o cancelamento publica um evento e o serviço de fila de espera o consome de forma assíncrona e desacoplada.
- **Timeout de 30 minutos para aceitar o encaixe:** SLA explícito que evita que o slot fique bloqueado indefinidamente aguardando resposta.
- **Iteração na fila (próximo candidato):** se o primeiro recusar, o sistema avança automaticamente — lógica FIFO com fallback.
- **Registro intencional do cancelamento:** permite distinguir cancelamento voluntário de não-comparecimento nos indicadores.

### Pontos de atenção / gaps
- Não está definido o **número máximo de tentativas na fila** antes de liberar o slot para novos agendamentos — se todos os candidatos recusarem, o processo precisa ter um fim definido.
- O cancelamento tardio (< 24h) não tem consequência diferente no fluxo além do registro — se houver política de cobrança, onde ela é aplicada?
- Não está modelado o que acontece se o **sistema de notificação falhar** durante o convite de encaixe — o candidato perde a vaga por falha técnica?
- O **motivo do cancelamento** é coletado? Isso seria útil para os indicadores de gestão (cancelamentos por problema de convênio vs. conveniência vs. urgência).
- Não há tratamento para **cancelamento pela clínica/médico** — apenas pelo paciente. Esse fluxo inverso precisa ser mapeado.

---

## Slide 9 / Drawio p.5–6 — Arquitetura de Solução

**Conteúdo:** Visão geral da solução em cinco blocos: Usuários e Canais, Borda/Entrada Segura, Plataforma MediAgenda Connect, Dados/Auditoria/Analytics e Ecossistema Externo.

### Componentes por bloco

**Borda / Entrada Segura:**
- Web/Mobile Frontend — Amazon S3 + CloudFront
- Proteção Web — AWS WAF
- Identidade e Acesso — Amazon Cognito / IAM
- Entrada das APIs — Amazon API Gateway

**Plataforma MediAgenda Connect:**
- Orquestração da Jornada — AWS Step Functions
- Serviço de Agendamento — AWS Lambda / ECS
- Fila de Espera — EventBridge + Lambda
- Elegibilidade e Cobertura — AWS Lambda / ECS
- Autorização Prévia — AWS Lambda / ECS
- Indicadores e Auditoria — Lambda + QuickSight
- Notificações — SNS / Pinpoint / SQS

**Dados, Auditoria e Analytics:**
- Banco Transacional — Amazon RDS PostgreSQL
- Filas de Eventos — Amazon SQS
- Segurança dos Dados — KMS + Secrets Manager + IAM
- Histórico e Analytics — Amazon S3 + QuickSight
- Monitoramento e Auditoria — CloudWatch + CloudTrail

**Ecossistema Externo:**
- Operadoras de Saúde — TISS/XML ou REST/FHIR
- Provedor de Mensageria — SMS, e-mail, WhatsApp, push
- Prontuário / Sistema Clínico — integração opcional

### Decisões técnicas identificadas
- **Dual runtime Lambda + ECS:** Lambda para workloads curtos e stateless; ECS para workloads com duração maior (ex.: aguardar resposta assíncrona de operadora).
- **SNS + Pinpoint + SQS:** separação correta — SNS para fan-out, Pinpoint para controle de canal e segmentação, SQS para desacoplamento e resiliência.
- **Prontuário como integração opcional:** decisão consciente de não bloquear o MVP por uma integração complexa.

### Pontos de atenção / gaps
- **Critério de escolha Lambda vs ECS** não está documentado — vale registrar a heurística (ex.: execução > 15 min ou estado persistente → ECS; workload curto e stateless → Lambda).
- A **topologia de rede** (VPC, subnets, security groups) não aparece nesta visão — está parcialmente na arquitetura AWS (Slide 13).
- Prontuário descrito como "integração opcional" mas o contrato esperado (HL7/FHIR R4?) não está definido.

---

## Slide 10 / Drawio p.14 — Arquitetura de TI / Operadoras

**Conteúdo:** Visão mais técnica unindo o core desacoplado com a camada de integração por adapters.

### Camadas identificadas

```
[Canais e Perfis de Acesso]
Paciente (Web/Mobile) | Recepção (Agenda e fila) | Médico (Agenda do dia) | Gestor (Indicadores)

[Borda, Segurança e Entrada de APIs]
CloudFront + WAF → Autenticação (Cognito/IAM) → API Gateway → Rate Limit + TLS

[Núcleo do Sistema de Agendamento — regras independentes das operadoras]
Serviço de Agendamento | Serviço de Elegibilidade | Serviço de Cobertura/Autorização
Serviço de Fila de Espera | Serviço de Indicadores

[Orquestração, Eventos e Integração com Operadoras]
Step Functions (orquestra jornada) | SQS Integração (assíncrono) | SQS Notificações (desacoplado)
EventBridge (lembretes e eventos)

[Diferencial: Camada de Integração Desacoplada]
Integration Service (contrato interno único)
→ Adapter REST/FHIR → Operadora A
→ Adapter TISS/XML → Operadora B
→ Adapter específico → Operadora C
→ Retry + DLQ (falhas e retentativas)

[Dados, Segurança, Auditoria e Observabilidade]
RDS PostgreSQL | S3 Data Lake | QuickSight (dashboards)
CloudWatch | CloudTrail | KMS + Secrets Manager
SNS/Pinpoint (mensageria de saída)
```

### Decisões técnicas identificadas
- **Core isolado da camada de integração:** o núcleo de agendamento não conhece detalhes técnicos de nenhuma operadora — chama apenas comandos internos como "validar elegibilidade".
- **Integration Service com contrato único:** a Facade garante que adicionar nova operadora não afeta o core — apenas um novo adapter é implementado.
- **Retry + DLQ por adapter:** resiliência isolada por operadora — falha na Operadora B não afeta processamento da Operadora A.

### Pontos de atenção / gaps
- O **contrato interno da Integration Facade** não está definido — para o repositório, um documento de interface (ex.: `IIntegrationAdapter`) seria essencial.
- O **roteamento por operadora** (como a plataforma sabe qual adapter usar para um determinado paciente) não está documentado.
- Não há menção a **circuit breaker por operadora** — se uma operadora ficar indisponível, o sistema deveria falhar rápido para esse adapter.

---

## Slide 11 / Drawio p.41–44 — C4 Nível 1: Contexto do Sistema

**Conteúdo:** Diagrama C4 de contexto (System Context) mostrando o sistema e seus atores.

### Pessoas (usuários)
| Pessoa | Interação | Protocolo |
|---|---|---|
| Paciente | Agenda, remarca, cancela consultas | HTTPS |
| Recepcionista | Gerencia agenda e fila de espera | HTTPS |
| Médico | Visualiza agenda e histórico | HTTPS |
| Gestor | Analisa KPIs e configurações | HTTPS |

### Sistemas Externos
| Sistema | Papel | Protocolo |
|---|---|---|
| Operadora de Saúde | Valida elegibilidade e autoriza | REST/FHIR, TISS/XML |
| Porta de Notificação | Envia SMS, e-mail e WhatsApp | SNS/API |
| Prontuário Eletrônico | Histórico clínico do paciente | HL7/FHIR (futuro) |
| Autenticação (IdP) | Autentica usuários e valida tokens | OAuth2/JWT |

### Decisões técnicas identificadas
- **IdP tratado como sistema externo:** correto no C4 — o Cognito é externo ao domínio de negócio da plataforma.
- **Prontuário com linha tracejada:** indica dependência opcional/futura — boa prática no C4 para distinguir integrações obrigatórias de planejadas.
- **OAuth2/JWT:** protocolo moderno compatível com SPAs e apps mobile.

### Pontos de atenção / gaps
- Múltiplas operadoras estão agrupadas em um único "Operadora de Saúde" — para maior clareza, poderia ter uma nota indicando que são N instâncias com protocolos diferentes.
- A integração com o Prontuário deveria ter o protocolo marcado como "(futuro)" no diagrama, não apenas como linha tracejada.

---

## Slide 12 / Drawio p.47–51 — C4 Nível 2: Containers

**Conteúdo:** Diagrama de containers detalhando os componentes internos do sistema.

### Containers identificados

| Container | Tecnologia | Responsabilidade |
|---|---|---|
| CloudFront | AWS CloudFront | CDN + HTTPS termination |
| WAF | AWS WAF | OWASP Top 10, rate limiting |
| API Gateway | AWS API Gateway | Roteamento + throttling |
| Cognito | AWS Cognito | User Pools, JWT, MFA |
| Orchestrator | AWS Step Functions | Orquestra jornada de agendamento |
| API de Agendamento | Lambda / ECS Fargate | Agendamento, elegibilidade, fila |
| Integration Service | Lambda + SQS | Adapters por operadora |
| Notification Service | SNS + Pinpoint + SQS | SMS, e-mail, WhatsApp |
| Cache | ElastiCache Redis | Cache de elegibilidade (10-15 min) |
| Banco de dados | RDS PostgreSQL Multi-AZ | Dados transacionais |
| Analytics | S3 + QuickSight | KPIs e dashboards |
| Data Lake | Amazon S3 | Histórico, auditoria, eventos |

### Decisões técnicas identificadas
- **ElastiCache Redis para cache de elegibilidade:** evita chamadas repetidas à mesma operadora para o mesmo paciente em curto período — reduz latência e custo.
- **RDS PostgreSQL Multi-AZ:** alta disponibilidade com failover automático para o banco transacional.
- **ECS Fargate para Integration Service:** adequado para workloads de longa duração aguardando resposta de operadoras.
- **Analytics separado do transacional:** S3 + QuickSight para análise sem impactar o RDS operacional.

### Pontos de atenção / gaps
- O **C3 (Componentes)** não está presente — para completar o C4, seria necessário detalhar ao menos o Integration Service e a API de Agendamento.
- Não está claro se o **Step Functions se comunica com a API de Agendamento** via invocação direta (SDK) ou via API Gateway interno.
- **CloudFront e WAF** são representados como containers de aplicação, mas no C4 formal seriam parte da infraestrutura de borda.

---

## Slide 13 / Drawio p.59–65 — Arquitetura AWS

**Conteúdo:** Diagrama de deployment com topologia AWS completa — VPC, regiões, AZs e integrações externas.

### Topologia identificada

```
Usuários → HTTPS → [VPC]

Public Subnet:
CloudFront + WAF → Route 53 → ACM → IAM → S3 (Frontend)

Security Group:
User Pool/MFA → API Gateway → IAM (Least Privilege) → Secrets Manager → KMS

Região Primária:
[App Tier]
Step Functions (Orquestrador) | Lambda | ECS Fargate (Integration Svc) | EventBridge

[Messaging - Private Subnet]
SQS Integração (DLQ) | SQS Notificações (DLQ) | SNS | Pinpoint/SES

[Observabilidade]
CloudWatch (Logs/Métricas) | CloudTrail (Auditoria) | X-Ray (Tracing) | PagerDuty/SNS (On-call P1/P2/P3)

Região Secundária:
RDS PostgreSQL Primary (AZ-a, PITR) ←sync→ RDS PostgreSQL Standby (AZ-b)
ElastiCache Redis | S3 Data Lake (Parquet/SSE-KMS) | QuickSight

Sistemas Externos:
Operadora A (REST/FHIR R4, SLA 99.5%)
Operadora B (TISS/SOAP XML, SLA 99.0%)
Operadora C (API legada, SLA 98.0%)
Twilio/SNS (SMS, e-mail, WhatsApp)
Prontuário (HL7/FHIR, futuro)
SIEM/SOC (Segurança/Logs)
```

### Decisões técnicas identificadas
- **RDS Multi-AZ com replicação síncrona:** failover automático sem perda de dados — decisão correta para sistema de saúde.
- **PITR no RDS:** Point-in-Time Recovery — proteção contra corrupção ou exclusão acidental de dados.
- **S3 com SSE-KMS:** criptografia server-side com chaves gerenciadas — atende LGPD para dados históricos.
- **PagerDuty com P1/P2/P3:** estrutura de on-call com severidades definidas desde o design.
- **SLAs das operadoras documentados:** 99,5% / 99,0% / 98,0% — base para definir timeouts e circuit breakers por adapter.

### Pontos de atenção / gaps
- **NAT Gateway** não está explícito — necessário para que Lambda/ECS em subnets privadas acessem as APIs das operadoras na internet.
- **Regras de Security Group** não detalhadas — quais portas abertas entre quais componentes?
- **Subnet do RDS** não está claramente isolada em subnet privada sem rota de saída para internet.
- **Estratégia Multi-Region** não definida — o diagrama sugere duas regiões para o RDS, mas não está claro se é DR (disaster recovery) ou active-active.
- O **SIEM/SOC** aparece como sistema externo sem detalhamento de integração — quais logs são enviados, em qual formato?

---

## Slide 14 / Drawio p.31–34 — Integração com Operadoras

**Conteúdo:** Diferencial técnico da plataforma — camada de adapters desacoplada.

### O que é validado por operadora
- Elegibilidade do plano
- Cobertura da consulta
- Autorização prévia
- Status: aprovado, negado ou pendente

### Como integrar
- Adapter TISS/XML — operadoras com padrão de troca XML (SOAP)
- Adapter REST/FHIR — operadoras com APIs modernas
- Fila de retentativa — SQS com backoff exponencial
- Logs de auditoria — rastreabilidade de cada interação

### Fluxo da integração

```
Core → Integration Facade (contrato único)
     → Modelo Canônico (Paciente, Plano, Procedimento, Guia, Status)
     → Roteador por Operadora
         → Adapter REST/FHIR → Operadora A (REST/FHIR R4)
         → Adapter TISS/XML  → Operadora B (TISS/SOAP)
         → Adapter Proprietário → Operadora C (legada)
     → SQS + Retry + Backoff
     → DLQ (falhas finais)
     → Audit Log
```

**Frase do documento:**
> *"Se uma operadora usa TISS/XML e outra usa API REST, o fluxo interno da plataforma não muda, apenas o adapter de comunicação."*

### Decisões técnicas identificadas
- **Modelo canônico próprio:** a plataforma define suas entidades internas independentemente do formato de cada operadora — evita que formatos externos "vazem" pelo sistema.
- **Credenciais por operadora no Secrets Manager:** isolamento — uma operadora comprometida não expõe as demais.
- **Auditoria por interação:** cada chamada à operadora é registrada com request, response e status — essencial para rastreabilidade LGPD e debugging.

### Pontos de atenção / gaps
- **Contrato de interface dos adapters** não está definido — para o repositório, um documento de interface (`IIntegrationAdapter`) com métodos `checkEligibility`, `checkCoverage`, `requestAuthorization` seria essencial.
- **Versão do FHIR** não especificada — FHIR R4 é o mais comum no Brasil, mas precisa ser explicitado.
- **mTLS ou VPN** para comunicação com operadoras não mencionado — frequentemente exigido em integrações com operadoras de saúde.
- **Circuit breaker por operadora** não mencionado — se Operadora B ficar indisponível, o sistema deve falhar rápido para esse adapter sem afetar os demais.

---

## Slide 15 / Drawio p.39 — Segurança, Auditoria e LGPD

**Conteúdo:** Controles de segurança transversais aplicados em todas as camadas.

| Controle | Tecnologia | Função |
|---|---|---|
| Criptografia em trânsito | TLS | Protege comunicação entre todos os componentes |
| Criptografia em repouso | AWS KMS | Dados sensíveis no RDS, S3 e backups |
| Acesso mínimo | IAM + perfis Cognito | Menor privilégio por função: paciente, recepção, médico, gestor |
| Segredos | Secrets Manager | Credenciais de operadoras e chaves de integração |
| Auditoria técnica | CloudTrail | Trilha de ações administrativas na AWS |
| Auditoria de negócio | Audit Log customizado | Quem validou, qual resposta da operadora, qual status |
| Observabilidade | CloudWatch | Logs, métricas e alarmes operacionais |
| Proteção web | CloudFront + WAF | OWASP Top 10, rate limiting na borda |

**Princípio central documentado:**
> *"Dados sensíveis precisam ser protegidos, rastreados e acessados apenas por quem realmente precisa."*

### Decisões técnicas identificadas
- **Dois tipos de audit log (técnico + negócio):** separação correta — CloudTrail captura ações AWS; o Audit Log de negócio captura eventos do domínio (quem fez o agendamento, qual operadora respondeu).
- **LGPD tratada como capacidade transversal:** segurança e privacidade desde o design, não como adendo — decisão madura.
- **KMS com criptografia em repouso:** atende requisitos de dados de saúde sensíveis.

### Pontos de atenção / gaps
- **Base legal do consentimento LGPD** não definida — para dados de saúde (dados sensíveis pela LGPD art. 11), o consentimento explícito ou a execução do contrato precisa ser a base legal declarada.
- **Período de retenção dos dados** não definido — logs de auditoria em saúde têm exigências regulatórias específicas no Brasil (CFM recomenda mínimo 20 anos para prontuários).
- **Rotação automática de segredos** no Secrets Manager não mencionada — frequência de rotação deve ser documentada.
- **Processo de resposta a incidentes** não descrito — GuardDuty detecta ameaças mas quem recebe o alerta e qual o fluxo de resposta?

---

## Slide 16 — Indicadores de Sucesso / KPIs 🆕

**Conteúdo:** Métricas de impacto operacional, financeiro e de experiência do paciente.

### KPIs principais

| KPI | Direção esperada |
|---|---|
| Taxa de faltas | ↓ Reduzir |
| Ocupação da agenda | ↑ Aumentar |
| Tempo de confirmação | ↓ Reduzir |
| Horários reaproveitados | ↑ Aumentar |
| Pendências de convênio | ↓ Reduzir |

### KPIs de gestão detalhados
- Faltas por especialidade
- Cancelamentos por período
- Autorizações pendentes
- Produtividade da recepção
- **SLA de integração com operadoras** ← novo, não estava no draw.io

**Objetivo:** transformar a agenda médica em uma operação mensurável e previsível.

### Decisões técnicas identificadas
- **SLA de integração com operadoras como KPI:** decisão importante — coloca a qualidade da integração técnica como métrica de negócio visível para o gestor.
- Os KPIs mapeiam diretamente para o QuickSight: cada KPI precisa de uma query no Athena/S3 Data Lake ou uma view no RDS.
- **Faltas por especialidade** implica que a tabela de consultas tem relacionamento com especialidade — confirma que o modelo de dados do RDS precisa incluir essa dimensão.

### Pontos de atenção / gaps
- Não há **metas numéricas** para os KPIs — ex.: "reduzir taxa de faltas de X% para Y%". Para o repositório, isso deixaria o case mais robusto.
- O **SLA de integração com operadoras** precisa de definição técnica: é tempo médio de resposta? Disponibilidade? Taxa de erros? Cada operadora tem SLA diferente (99,5%, 99,0%, 98,0%).
- Não há KPI de **experiência do paciente** (ex.: NPS, tempo médio de agendamento, taxa de conclusão do fluxo) — vale adicionar.

---

## Slide 17 — Roadmap de Implementação 🆕

**Conteúdo:** Três fases de evolução da plataforma.

### Fase 1 · MVP
- Agenda digital
- Confirmação e lembretes
- Fila de espera
- Dashboard básico

### Fase 2 · Integração
- Elegibilidade com operadoras
- Cobertura
- Autorização prévia
- Retentativas e auditoria

### Fase 3 · Otimização
- Predição de faltas (IA)
- Relatórios avançados
- Integração com prontuário
- Melhoria contínua

### Decisões técnicas identificadas
- **MVP sem integração com operadoras:** decisão correta para validar o core de agendamento antes de adicionar a complexidade das integrações externas.
- **Predição de faltas na Fase 3:** uso futuro do S3 Data Lake como base para ML (SageMaker) — a arquitetura já foi projetada para suportar isso desde o início.
- **Prontuário na Fase 3:** integração complexa postergada — priorização correta para não bloquear o MVP.

### Pontos de atenção / gaps
- **Critérios de transição entre fases** não definidos — o que valida que o MVP está pronto para Fase 2? KPI mínimo? Número de clínicas? Estabilidade técnica?
- **Estimativas de prazo** não incluídas — para o repositório, mesmo uma estimativa em ordem de grandeza (ex.: Fase 1 ~3 meses) ajuda a contextualizar o roadmap.
- A Fase 1 inclui fila de espera mas não elegibilidade — isso significa que no MVP o convênio **não é validado antes da consulta**. Isso precisa ser comunicado claramente para o usuário (ex.: "validação de convênio disponível em breve").

---

## Drawio — Camada 1: Canais e Entrada Segura

**Fonte:** Drawio páginas 21–24

**Componentes:**
```
Canais (Web/App por perfil) → CloudFront (CDN + baixa latência)
→ WAF (OWASP Top 10 + rate limiting)
→ Cognito (login, MFA, perfis de acesso)
→ API Gateway (entrada única, rate limit + authorizer)
→ BFF/Backend APIs (normaliza chamadas e encaminha para serviços internos)
→ Logs de Acesso (WAF logs + API logs → CloudWatch / S3)
```

### Decisões técnicas identificadas
- **Token com perfil embutido (claims):** perfil do usuário no JWT evita consultas adicionais ao banco para resolver permissões.
- **CloudFront como CDN:** reduz latência do frontend estático e centraliza proteção WAF.
- **Rate limiting no API Gateway:** protege serviços internos e cotas das APIs das operadoras.

### Pontos de atenção / gaps
- **Tempo de expiração dos tokens JWT** não definido — crítico para UX do paciente que pode demorar para completar o agendamento.
- **BFF por perfil ou único?** Para perfis com necessidades muito diferentes (paciente vs. gestor), BFFs separados são recomendados.
- **CORS policies** não mencionadas — necessário quando frontend e API estão em domínios diferentes.
- **MFA obrigatório para quais perfis?** Apenas gestor/recepção? Ou todos?

---

## Drawio — Camada 2: Core de Agendamento

**Fonte:** Drawio páginas 27–29

### Serviços do Core

| Serviço | Responsabilidade |
|---|---|
| Agendamento | Cria, altera, cancela e consulta horários |
| Disponibilidade | Calcula horários livres por médico, especialidade e unidade |
| Confirmação | Registra resposta do paciente e status da consulta |
| Fila de Espera | Seleciona pacientes para horários cancelados |
| Elegibilidade | Solicita validação do plano |
| Cobertura/Autorização | Consulta cobertura e pedido de autorização prévia |

**Orquestração:** AWS Step Functions  
**Eventos:** Amazon EventBridge  
**Motor de Regras:** políticas de agenda, prazos e cancelamento (tecnologia não definida)

**Decisão documentada:**
> *"O core de agendamento não conhece detalhes técnicos de cada operadora."*

### Decisões técnicas identificadas
- **Elegibilidade e Cobertura/Autorização separados:** têm temporalidade diferente — elegibilidade na marcação; autorização pode ter prazo de validade.
- **Motor de Regras separado:** políticas configuráveis por clínica, não hardcoded.
- **EventBridge para lembretes:** scheduled rules (D-1, H-2) desacopla timing dos lembretes do fluxo principal.

### Pontos de atenção / gaps
- **Serviço de Disponibilidade** é potencialmente o mais complexo — precisa considerar agenda do médico, feriados, unidade, especialidade e lock de slots. Essa complexidade não está detalhada.
- **Lock de slot via Redis** está implícito mas não explicitamente ligado ao Serviço de Disponibilidade no draw.io.
- **Step Functions Standard vs Express Workflows** não definido — para fluxos com autorização pendente por 48h, Standard é obrigatório.
- **Motor de Regras** sem tecnologia associada — Lambda com lógica customizada? Rules engine?
- **Comportamento em falha do Step Functions** no meio da jornada não está descrito — compensação automática ou alerta manual?

---

## Drawio — Camada 4: Eventos, Filas e Resiliência

**Fonte:** Drawio páginas 35–36

### Eventos de negócio mapeados

| Evento | Consequência |
|---|---|
| Consulta criada | Dispara validação do plano |
| Consulta confirmada | Agenda lembretes |
| Consulta cancelada | Aciona fila de espera |
| Autorização pendente | Acompanha status |

### Componentes de resiliência

| Componente | Papel |
|---|---|
| EventBridge | Roteamento de eventos e lembretes agendados |
| SQS Integração | Desacopla tempo de resposta das operadoras |
| SQS Notificações | Desacopla envio de mensagens |
| Workers Lambda | Processam mensagens e atualizam status |
| Idempotência | Evita duplicar autorizações, notificações e agendamentos |
| Retry com backoff | Reprocessa falhas transitórias |
| DLQ | Preserva mensagens com falha final para análise |

### Decisões técnicas identificadas
- **Duas filas SQS separadas:** isolamento — falha em massa nas notificações não afeta o processamento de elegibilidade.
- **Idempotência explícita:** reconhece que retries podem causar efeitos colaterais — essencial para sistemas de saúde.
- **EventBridge como bus central:** adicionar novos consumidores de eventos sem modificar o produtor.

### Pontos de atenção / gaps
- **Mecanismo de idempotência** não detalhado tecnicamente — DynamoDB com TTL? UUID de correlação no header?
- **Visibility timeout das filas** não definido — deve ser maior que o tempo máximo de processamento de um worker.
- **Processo de triagem das DLQs** não descrito — quem monitora? Com que frequência? Alerta automático ou manual?

---

## Drawio — Camada 5: Dados, Indicadores e Analytics

**Fonte:** Drawio páginas 37–38

### Arquitetura de dados

**Transacional:**
- RDS PostgreSQL Multi-AZ — agenda, paciente, médico, convênio, consulta, autorização
- DynamoDB — status rápido da jornada e controle de idempotência
- S3 — anexos, documentos e trilhas exportadas

**Analítico:**
- Event Store — histórico de cada ação da jornada
- S3 Data Lake — histórico para análises futuras
- Glue + Athena — catalogação e consulta analítica
- QuickSight — dashboards executivos e operacionais

**Evolução futura:** SageMaker para predição de no-show

### Decisões técnicas identificadas
- **RDS para transacional, S3 para analítico:** separação correta que evita queries analíticas pesadas impactando o agendamento em produção.
- **DynamoDB para idempotência:** acesso por chave com latência de milissegundos, sem necessidade de joins.
- **Event Store separado:** permite reconstruir o histórico completo de uma jornada para auditoria.

### Pontos de atenção / gaps
- **Modelo de dados do RDS** não documentado — diagrama ER básico seria essencial para o repositório.
- **Estratégia de retenção** no RDS e S3 não definida — LGPD exige definição de período de retenção para dados de saúde.
- **Latência de atualização dos KPIs** no QuickSight não especificada — near-real-time (SPICE refresh) ou batch diário?
- **Jobs Glue/ETL** para normalizar eventos do EventBridge em datasets analíticos não detalhados.

---

## Drawio — Decisões Arquiteturais para Defender

**Fonte:** Drawio página 40

| # | Decisão | Justificativa |
|---|---|---|
| 1 | Arquitetura em camadas | Separação de canais, core, integração, dados e segurança |
| 2 | Core desacoplado das operadoras | Agendamento não depende do formato técnico de cada operadora |
| 3 | Integration Facade + Adapters | Suporta REST, FHIR, TISS/XML e APIs proprietárias |
| 4 | Processamento assíncrono | SQS e EventBridge evitam travar o fluxo quando há falhas externas |
| 5 | Retentativas e DLQ | Falhas transitórias reprocessadas; falhas finais rastreáveis |
| 6 | RDS para transacional | Consistência e relacionamentos claros |
| 7 | Data Lake para indicadores | Histórico sem impactar o banco operacional |
| 8 | Segurança by design | KMS, Secrets Manager, IAM, CloudTrail, WAF |
| 9 | Observabilidade | CloudWatch e X-Ray para latência, falhas e integrações |
| 10 | Evolução com IA | Base histórica para predição de no-show (SageMaker) |

**Frase de síntese:**
> *"Proteger o núcleo do negócio e isolar a complexidade externa. O core cuida da jornada da consulta; a camada de integração cuida das diferenças entre operadoras; as filas cuidam da resiliência; e segurança/observabilidade sustentam a operação."*

### Pontos de atenção / gaps
- Para o repositório, cada uma dessas 10 decisões merece um **ADR formal** com: contexto, opções consideradas, decisão tomada e consequências.
- A decisão de usar **Serverless (Lambda)** como padrão inicial deveria estar na lista — é uma decisão arquitetural relevante para o MVP.

---

## Resumo Consolidado dos Gaps

> Gaps marcados com 🆕 foram identificados ou complementados pela análise do PPT.

| Tema | Gap | Prioridade | Status |
|---|---|---|---|
| ADRs | Nenhuma decisão formalizada como ADR | Alta | Aberto |
| Modelo de dados | Sem diagrama ER do RDS PostgreSQL | Alta | Aberto |
| Contrato dos adapters | Interface `IIntegrationAdapter` não definida | Alta | Aberto |
| Roteamento de operadoras | Critério de discovery (por carteirinha, por clínica?) | Alta | Aberto |
| LGPD | Base legal do consentimento e período de retenção | Alta | Aberto |
| BPMN Cancelamento 🆕 | **Documentado no Processo 2 do PPT** | — | ✅ Coberto |
| BPMN Remarcação 🆕 | Fluxo de remarcação não está em nenhum BPMN | Alta | Aberto |
| BPMN Lembretes 🆕 | Fluxo de lembrete (D-1, H-2) sem BPMN dedicado | Média | Aberto |
| BPMN Cancelamento pela clínica 🆕 | Não modelado — apenas cancelamento pelo paciente | Média | Aberto |
| C4 Nível 3 | Componentes internos não documentados | Média | Aberto |
| Topologia de rede | Security Groups, NAT Gateway, isolamento de subnets | Média | Aberto |
| Idempotência | Mecanismo técnico não detalhado | Média | Aberto |
| Step Functions | Standard vs Express Workflows não definido | Média | Aberto |
| Motor de Regras | Tecnologia não definida | Baixa | Aberto |
| Roadmap 🆕 | **Documentado no PPT (3 fases)** | — | ✅ Coberto |
| KPIs 🆕 | **Expandidos no PPT com SLA de operadoras** | — | ✅ Coberto |
| Multi-tenancy | Suporte a múltiplas clínicas não endereçado | Média | Aberto |
| IaC | Sem menção a Terraform, CDK ou CloudFormation | Baixa | Aberto |
| Circuit breaker | Por operadora, não mencionado | Média | Aberto |
| mTLS/VPN | Comunicação segura com operadoras não detalhada | Alta | Aberto |
| Fases do roadmap 🆕 | Critérios de transição entre fases não definidos | Baixa | Aberto |

---

*Documento gerado com base na análise de dois arquivos:*  
*1. `Arquitetura_MediAgend0_-_Arquitetura_TI_-_Visão_Geral.drawio.pdf`*  
*2. `MediAgenda_Connect_Apresentacao_Completa.pptx`*
