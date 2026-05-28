# 06 — Camada 4: Eventos, Filas e Resiliência

<img width="2011" height="1041" alt="07 — Camada 5_ Dados, Indicadores e Analytics drawio" src="https://github.com/user-attachments/assets/b9c1bf06-70bb-49c1-af30-7d8c59652468" />


Uma das decisões mais importantes da arquitetura foi garantir que uma falha externa uma operadora indisponível, um provedor de SMS sobrecarregado não trave a jornada do paciente. Este diagrama mostra como isso é feito.

O ponto central é a separação entre o que precisa acontecer agora e o que pode ser processado de forma assíncrona. Quando o Step Functions confirma um agendamento, ele não liga diretamente para a operadora esperando uma resposta em tempo real. Ele publica uma mensagem em uma fila SQS e segue em frente. Um worker Lambda lê essa mensagem, chama o adapter correto, e atualiza o status quando a resposta chega. Se a operadora demorar ou falhar, o fluxo principal já avançou.

Existem duas filas SQS intencionalmente separadas: uma para integrações com operadoras e outra para notificações. Esse isolamento garante que uma falha em massa nas notificações digamos, o provedor de SMS fora do ar não afeta o processamento de elegibilidade. E vice-versa.

O **EventBridge** funciona como o bus de eventos da plataforma. Quando um evento de negócio acontece consulta criada, consulta cancelada, autorização aprovada ele é publicado no EventBridge, e qualquer serviço interessado pode consumi-lo sem que o produtor precise saber quem são esses consumidores. Isso permite adicionar novos comportamentos sem alterar o código existente.

O tratamento de falhas segue uma lógica clara. Erros transitórios timeout, instabilidade temporária da operadora são reprocessados automaticamente com backoff exponencial. Erros permanentes resposta inválida, operadora retornou erro definitivo vão para a DLQ, onde ficam preservados para análise e reprocessamento manual quando necessário.

A idempotência é tratada como requisito, não como detalhe. Em um cenário com retries, a mesma mensagem pode ser processada mais de uma vez. O sistema garante que processar a mesma autorização duas vezes não resulta em duas autorizações enviadas, e que o mesmo lembrete não chegue duas vezes para o paciente.
