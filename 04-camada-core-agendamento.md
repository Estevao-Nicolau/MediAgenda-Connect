# 04 — Camada 2: Core de Agendamento

<img width="2151" height="896" alt="05 — Camada 3_ Integração Desacoplada com Operadoras drawio" src="https://github.com/user-attachments/assets/4e1e2e24-32b5-498d-b8e7-1d174c3eaba7" />


Este é o coração da plataforma. Aqui vivem as regras de negócio da clínica e a decisão mais importante deste diagrama é o que está fora dele: nenhum serviço do core conhece detalhes técnicos de nenhuma operadora de saúde.

O **Step Functions** orquestra a jornada completa da consulta. Cada etapa é um estado da máquina: verificar disponibilidade, validar elegibilidade, solicitar autorização, confirmar o agendamento, agendar lembretes. Se uma etapa falha, o Step Functions sabe exatamente em qual ponto parou, pode retentar ou pode acionar um caminho de compensação. Isso dá visibilidade total sobre onde cada consulta está no fluxo  algo impossível de ter com chamadas diretas entre serviços.

Os serviços do core são independentes entre si e têm responsabilidades bem delimitadas:

O **Serviço de Agendamento** é responsável por criar, alterar, cancelar e consultar horários. Ele aplica o lock de 10 minutos no slot via Redis enquanto a validação do convênio acontece, evitando que dois pacientes ocupem o mesmo horário simultaneamente.

O **Serviço de Disponibilidade** calcula os horários livres considerando a agenda do médico, a unidade, a especialidade e os slots já bloqueados. É provavelmente o serviço com a lógica mais complexa do core.

O **Serviço de Elegibilidade** emite o comando de verificação do plano sem saber como a operadora vai responder. Ele entrega isso para a camada de integração e aguarda o resultado.

O **Serviço de Cobertura e Autorização** funciona de forma similar, mas com temporalidade diferente: a elegibilidade é verificada no momento do agendamento, enquanto a autorização prévia pode ter prazo de validade e precisa ser acompanhada até a consulta acontecer.

O **Serviço de Fila de Espera** entra em ação quando um slot é cancelado. Ele identifica o próximo candidato na fila FIFO, bloqueia o slot temporariamente e dispara a notificação de convite de encaixe.

O **EventBridge** complementa o Step Functions publicando eventos de negócio que outros serviços consomem de forma desacoplada — incluindo os lembretes programados para D-1 e H-2 antes da consulta.
