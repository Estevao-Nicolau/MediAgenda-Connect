# 13 — BPMN: Cancelamento e Reaproveitamento de Slot

<!-- Adicione o print da página 13 aqui -->

Este diagrama mapeia o que acontece quando um paciente cancela uma consulta — e mostra como a plataforma tenta aproveitar esse horário automaticamente, sem nenhuma intervenção manual da recepção.

O processo começa com o pedido de cancelamento do paciente, com ou sem motivo informado. A primeira verificação é sobre o prazo: se o cancelamento acontece com 24 horas ou mais de antecedência, é um cancelamento padrão. Se acontece com menos de 24 horas, é registrado como tardio — uma distinção útil para indicadores e para eventuais políticas de cobrança que a clínica queira aplicar.

O sistema então executa três ações em sequência: valida a solicitação, libera o slot na agenda e registra o cancelamento como intencional — diferenciando de uma falta, onde o paciente simplesmente não apareceu. Em seguida, publica o evento `SlotLiberado` no EventBridge e envia a confirmação de cancelamento ao paciente.

O evento `SlotLiberado` é consumido pelo Serviço de Fila de Espera de forma assíncrona. Ele verifica se há algum paciente aguardando por aquele tipo de horário — mesma especialidade, mesmo médico, mesma unidade. Se não houver candidatos, o slot simplesmente fica disponível para novos agendamentos. Se houver, o sistema bloqueia o slot temporariamente e envia um convite de encaixe para o primeiro da fila, com 30 minutos para aceitar.

Se o paciente da fila aceitar dentro do prazo, o encaixe é confirmado e o processo termina. Se recusar ou não responder dentro dos 30 minutos, o sistema avisa que o tempo expirou e avança para o próximo candidato na fila, repetindo o processo até que alguém aceite ou a fila se esgote. Se nenhum candidato aceitar, o slot é liberado para novos agendamentos normais.

Toda essa lógica acontece sem que a recepção precise fazer nada. O horário que seria perdido tem uma segunda chance de ser aproveitado automaticamente, e o paciente da fila recebe a oportunidade de ser atendido mais cedo do que o previsto.
