# 10 — C4 Model: Nível 2 — Containers

<img width="1951" height="1286" alt="Arquitetura_MediAgenda--16- AWS-12 - C4 - Containers drawio" src="https://github.com/user-attachments/assets/8e77c636-bc54-4108-b3d3-db769f45e7d8" />


O diagrama de containers abre a caixa do MediAgenda Connect e mostra os componentes que o formam cada um com sua tecnologia, sua responsabilidade e os protocolos de comunicação entre eles.

**Na borda**, CloudFront termina o HTTPS e serve o frontend. O WAF inspeciona o tráfego antes de qualquer requisição chegar à aplicação. O API Gateway roteia as chamadas autenticadas para os serviços internos. O Cognito valida tokens e resolve perfis de acesso.

**No centro da lógica de negócio**, o Orchestrator implementado com AWS Step Functions coordena a jornada de ponta a ponta. A API de Agendamento, rodando em Lambda ou ECS Fargate dependendo da característica do workload, executa as operações de criação, alteração e consulta de horários. O ElastiCache Redis serve como cache de elegibilidade por 15 minutos evitando chamadas repetidas à operadora para o mesmo paciente no mesmo período.

**Para integração com operadoras**, o Integration Service encapsula os adapters e a lógica de roteamento. Ele se comunica com as operadoras de forma assíncrona via SQS, o que significa que a latência da operadora não bloqueia o fluxo do paciente. O Notification Service cuida do envio de mensagens via SNS, Pinpoint e SQS, separado da integração com operadoras para que uma falha em um não afete o outro.

**Para dados**, o RDS PostgreSQL Multi-AZ armazena tudo que precisa de consistência transacional. O Data Lake em S3 recebe eventos históricos em formato Parquet. O Analytics — S3 com QuickSight — consome esses dados e entrega os dashboards gerenciais.

Os sistemas externos aparecem nas bordas do diagrama: Prontuário Eletrônico com linha tracejada (futuro), Operadoras de Saúde com seus protocolos distintos, e o Gateway de Notificações que entrega as mensagens ao paciente.
