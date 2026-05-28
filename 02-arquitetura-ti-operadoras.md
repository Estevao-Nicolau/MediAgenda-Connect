# 02 — Arquitetura de TI e Integração com Operadoras

<!-- Adicione o print da página 02 aqui -->

Este diagrama aprofunda a visão técnica e coloca em evidência o principal diferencial de design da plataforma: o core de agendamento nunca conversa diretamente com nenhuma operadora de saúde.

O diagrama está organizado em faixas horizontais que representam responsabilidades distintas. Na parte superior ficam os canais de acesso — cada perfil de usuário com seu portal próprio. Logo abaixo está a camada de borda com CloudFront, WAF, Cognito e API Gateway. No meio fica o núcleo do sistema, onde vivem os serviços de agendamento, elegibilidade, cobertura, fila de espera e indicadores. Abaixo do núcleo está a camada de orquestração e eventos, com Step Functions coordenando o fluxo, EventBridge publicando eventos de negócio, e as filas SQS desacoplando o processamento. Na base estão os dados, a segurança e a observabilidade.

O que chama atenção é o bloco à direita: a camada de integração desacoplada. Quando o núcleo precisa validar um plano, ele emite um comando interno — "verificar elegibilidade" — sem saber nada sobre o protocolo da operadora. A Integration Facade recebe esse comando, identifica qual operadora é responsável por aquele paciente, e roteia para o adapter correto. Se a operadora usa REST/FHIR, vai para o adapter REST. Se usa TISS/XML, vai para o adapter TISS. Se tem uma API proprietária, tem um adapter específico para ela.

Essa separação tem uma consequência prática importante: adicionar suporte a uma nova operadora significa criar um novo adapter. O core não muda, o fluxo não muda, os outros adapters não são afetados.

Falhas são tratadas com retry e backoff exponencial via SQS. Se a tentativa final também falhar, a mensagem vai para a DLQ, onde fica disponível para análise e reprocessamento manual.
