# 11 — Arquitetura AWS: Deployment

<!-- Adicione o print da página 11 aqui -->

Este diagrama sai do nível lógico e mostra como a plataforma está de fato organizada na AWS — VPC, subnets, zonas de disponibilidade, regiões e as conexões com sistemas externos.

Todo o tráfego dos usuários chega via HTTPS e entra pela camada pública: CloudFront distribui o conteúdo estático do S3, WAF protege a borda, Route 53 resolve o DNS e o ACM gerencia os certificados TLS.

Dentro da VPC, a separação de responsabilidades se reflete na topologia de rede. Os serviços de aplicação — Step Functions, Lambda e ECS Fargate com o Integration Service — ficam em subnets privadas sem exposição direta à internet. A comunicação com as APIs das operadoras externas sai por um NAT Gateway. As filas SQS de integração e de notificações ficam em subnets privadas de mensageria, com DLQs configuradas para preservar mensagens com falha definitiva. SNS e Pinpoint cuidam do envio final das notificações.

O banco de dados fica em uma região dedicada com alta disponibilidade garantida por duas instâncias: a primária na AZ-a com PITR habilitado, e a standby na AZ-b com replicação síncrona. Failover é automático. O ElastiCache Redis fica na mesma região, próximo ao banco, para minimizar latência nas consultas de cache. O S3 Data Lake armazena os eventos históricos em Parquet com criptografia SSE-KMS. O QuickSight consome esses dados e entrega os dashboards.

A observabilidade está presente em todas as camadas: CloudWatch coleta logs e métricas de todos os serviços, CloudTrail audita ações administrativas na conta AWS, e o X-Ray traceia requisições distribuídas entre Lambda, ECS, SQS e chamadas externas. Alertas críticos chegam via PagerDuty com severidades P1, P2 e P3.

Na parte inferior do diagrama estão os sistemas externos: três operadoras com protocolos e SLAs diferentes (REST/FHIR R4 a 99,5%, TISS/SOAP a 99,0%, API legada a 98,0%), o gateway de notificações Twilio/SNS, e o SIEM/SOC para gestão centralizada de segurança e logs.
