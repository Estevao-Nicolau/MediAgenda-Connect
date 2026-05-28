# 12 — BPMN: Agendamento com Validação de Convênio

<img width="1707" height="971" alt="Arquitetura_MediAgenda--16- AWS-13 - BPMN - P1 drawio" src="https://github.com/user-attachments/assets/77de46a2-379e-40ac-8166-c32b1600f093" />


Este diagrama mapeia o fluxo principal do negócio em notação BPMN, com swim lanes separando as responsabilidades de cada participante: Paciente, Sistema MediAgenda, Operadora de Saúde, Recepção / Clínica e Notificações.

O fluxo começa quando o paciente acessa o portal, escolhe o médico, a especialidade e o horário, e informa os dados do cartão do convênio. A partir daí, o sistema assume.

O primeiro passo é verificar a disponibilidade do horário escolhido. Se o slot já foi ocupado, o sistema oferece a fila de espera o paciente pode aceitar e entrar na fila FIFO, ou desistir. Se o slot está livre, ele é bloqueado por 10 minutos via Redis enquanto a validação acontece. Esse lock evita que dois pacientes ocupem o mesmo horário enquanto o processo de verificação ainda está em andamento.

Com o slot reservado, o sistema consulta a elegibilidade do plano junto à operadora. Se o plano estiver inativo ou o paciente não for elegível, o agendamento é rejeitado imediatamente e o paciente é notificado. Se estiver elegível, o sistema avança para a solicitação de autorização prévia.

A resposta da operadora pode ser uma de três: aprovado, pendente ou negado. Se aprovado, o agendamento é confirmado e o paciente recebe a notificação. Se negado, o agendamento é cancelado e o paciente é avisado. Se pendente a operadora precisa de mais documentação ou está em análise o sistema aguarda até 48 horas. A recepção recebe um alerta sobre a consulta pendente e é responsável por contatar a operadora e resolver. Se resolver, confirma manualmente no sistema. Se não resolver dentro do prazo, o agendamento é cancelado.

Um detalhe importante: se os 10 minutos de lock expirarem sem que o processo seja concluído, o slot é liberado automaticamente para outros pacientes. Isso garante que horários não fiquem presos indefinidamente por processos incompletos.

Os estados possíveis de uma consulta ao longo deste fluxo são: `NA_FILA`, `CONFIRMADO`, `PENDENTE` e `REJEITADO`.
