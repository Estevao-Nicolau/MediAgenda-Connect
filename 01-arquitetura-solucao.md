# 01 — Arquitetura de Solução

<!-- Adicione o print da página 01 aqui -->

Este diagrama é a porta de entrada para entender o MediAgenda Connect como um todo. Ele não desce para detalhes técnicos ainda — o objetivo é mostrar quem usa a plataforma, por onde entra, o que ela faz internamente e com quem ela conversa do lado de fora.

A leitura acontece da esquerda para a direita. À esquerda estão os quatro perfis que usam a plataforma: o paciente, que agenda pelo portal ou app; a recepção, que gerencia a agenda e a fila de espera; o médico, que visualiza sua agenda e o status dos atendimentos; e o gestor, que acompanha indicadores de ocupação e produtividade.

Toda essa entrada passa por uma camada de borda antes de chegar em qualquer serviço interno. CloudFront serve o frontend com baixa latência, o WAF bloqueia tráfego malicioso, o Cognito cuida de autenticação e perfis de acesso, e o API Gateway é o ponto único de entrada para todas as chamadas de API. Nenhuma requisição chega ao core sem passar por essa camada.

No centro do diagrama está a plataforma em si. O Step Functions orquestra a jornada completa da consulta — do agendamento até o registro de comparecimento ou falta. Os serviços de agendamento, elegibilidade, cobertura e autorização trabalham juntos mas são independentes entre si. A fila de espera reage automaticamente quando um horário é liberado. As notificações são enviadas via SNS e Pinpoint, desacopladas do fluxo principal por filas SQS.

À direita estão os sistemas externos: as operadoras de saúde, que validam plano e autorizam procedimentos; o provedor de mensageria, que entrega SMS, e-mail e WhatsApp; e o prontuário eletrônico, tratado como integração futura.

No bloco de dados, a separação já aparece aqui: RDS PostgreSQL para o dado transacional que precisa de consistência, e S3 com QuickSight para o histórico analítico e os dashboards gerenciais.
