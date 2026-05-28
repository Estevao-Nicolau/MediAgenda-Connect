# 07 — Camada 5: Dados, Indicadores e Analytics

<img width="1561" height="1046" alt="Arquitetura_MediAgenda--16- AWS-08 - Camada 5 - Dados, indicadores e analytics drawio" src="https://github.com/user-attachments/assets/db711af6-48de-4309-b462-550e778b3450" />

A estratégia de dados da plataforma parte de uma separação clara: dado transacional fica no RDS, dado analítico fica no S3. Essa divisão não é estética ela tem um motivo prático direto. Rodar uma query de relatório pesada sobre o mesmo banco que processa agendamentos em tempo real é uma receita para degradar a performance do sistema no pior momento possível.

O **RDS PostgreSQL Multi-AZ** armazena tudo que precisa de consistência transacional: pacientes, médicos, especialidades, agendas, consultas, convênios, autorizações. São dados que precisam de relacionamentos claros, transações ACID e failover automático em caso de falha de uma zona de disponibilidade. A replicação é síncrona para a instância standby sem perda de dados em caso de failover.

O **DynamoDB** complementa o RDS para dois casos específicos: controle de status rápido da jornada, onde a latência precisa ser de milissegundos, e controle de idempotência, onde é preciso verificar rapidamente se uma mensagem já foi processada antes de executar novamente.

Do outro lado da separação, o **S3 Data Lake** recebe eventos históricos de tudo que acontece na plataforma: cada mudança de status de uma consulta, cada resposta de operadora, cada lembrete enviado, cada falta registrada. Esses eventos chegam no formato Parquet, particionados por data, prontos para consulta analítica.

O **Glue** cataloga esses dados e o **Athena** permite consultá-los sem precisar mover nada. O **QuickSight** consome os resultados dessas consultas e entrega os dashboards para o gestor da clínica — taxa de ocupação, taxa de faltas por especialidade, horários reaproveitados, autorizações pendentes, SLA de integração por operadora.

A base histórica no Data Lake serve também como fundação para a Fase 3 do roadmap: usar SageMaker para construir um modelo de predição de risco de falta, ajudando a clínica a agir preventivamente antes que o horário seja perdido.
