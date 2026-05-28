# 09 — C4 Model: Nível 1 — Contexto do Sistema

<!-- Adicione o print da página 09 aqui -->

O diagrama de contexto é o nível mais alto do C4 Model. Ele não mostra tecnologia — mostra relacionamentos. Quem usa o sistema, com quem o sistema conversa, e qual o propósito de cada conexão.

No centro está o MediAgenda Connect como uma caixa única. Ao redor dele estão as pessoas e os sistemas com quem ele se relaciona.

Do lado das pessoas, quatro perfis interagem com a plataforma via HTTPS: o paciente, que agenda, remarca e cancela consultas; a recepcionista, que gerencia a agenda e a fila de espera operacionalmente; o médico, que visualiza sua agenda e o status de convênio dos atendimentos; e o gestor, que acompanha KPIs e configurações da clínica.

Do lado dos sistemas externos, quatro dependências estão mapeadas. A **Operadora de Saúde** — que na prática são múltiplas operadoras com protocolos diferentes — valida elegibilidade e autoriza procedimentos via REST/FHIR ou TISS/XML. A **Porta de Notificação** recebe os eventos da plataforma e entrega SMS, e-mail e WhatsApp para os pacientes. O **Prontuário Eletrônico** aparece com linha tracejada, indicando que é uma integração planejada para o futuro — a arquitetura já prevê o protocolo HL7/FHIR, mas não bloqueia o MVP por isso. A **Autenticação (IdP)** — implementada com Amazon Cognito — é tratada como sistema externo ao domínio de negócio, o que é correto do ponto de vista do C4: ela é um serviço de suporte, não parte do core.

Este diagrama é o ponto de partida para qualquer conversa sobre a plataforma com stakeholders que não são técnicos. Ele responde à pergunta mais básica — o que esse sistema faz e com quem — sem entrar em nenhum detalhe de implementação.
