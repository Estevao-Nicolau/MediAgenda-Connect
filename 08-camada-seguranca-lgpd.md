# 08 — Camada 6: Segurança, LGPD e Observabilidade

<img width="1611" height="966" alt="Arquitetura_MediAgenda--16- AWS-09 - Camada 6 - Segurança, LGPD e observabilidade drawio" src="https://github.com/user-attachments/assets/8f108094-f444-4b48-9ab5-5a2c4f0dd0cb" />



Segurança aqui não é uma camada adicionada depois é uma decisão que atravessa todas as outras. A plataforma lida com dados pessoais sensíveis no sentido da LGPD: dados de saúde, dados de convênio, histórico de consultas. Isso exige controles desde o primeiro diagrama, não como um checklist no final.

**Criptografia** acontece em dois momentos. Em trânsito, TLS protege toda comunicação entre o usuário e o frontend, entre o frontend e a API, entre os serviços internos, entre os adapters e as operadoras. Em repouso, o KMS gerencia as chaves de criptografia para o RDS, o S3 e os backups. Dados de saúde não ficam em claro em nenhum momento.

**Controle de acesso** segue o princípio do menor privilégio em dois níveis. No nível do usuário, cada perfil no Cognito tem escopos diferentes um paciente não acessa endpoints de gestão, um médico não acessa dados de outros médicos. No nível dos serviços, cada Lambda e cada ECS task tem uma role IAM com apenas as permissões que precisa para funcionar. Um serviço que só lê do RDS não tem permissão de escrita, e não tem acesso ao Secrets Manager de outro serviço.

**Credenciais de operadoras** ficam no Secrets Manager, isoladas por operadora. A rotação é automática e o acesso é auditado. Se uma credencial for comprometida, o raio de impacto é limitado àquele adapter específico.

**Auditoria** acontece em duas camadas complementares. O CloudTrail registra toda ação administrativa na AWS quem criou um recurso, quem alterou uma policy, quem acessou um segredo. Já o Audit Log de negócio, implementado na aplicação, registra os eventos do domínio: quem agendou, qual operadora validou, qual foi a resposta, qual foi o status da autorização. Esses dois logs juntos permitem reconstruir qualquer evento técnico ou de negócio que tenha acontecido na plataforma.

**Observabilidade** é o que permite operar o sistema com confiança. CloudWatch coleta logs e métricas de todos os serviços e dispara alarmes quando algo sai dos limites esperados aumento no tamanho da DLQ, latência acima do threshold, taxa de erros 5xx crescendo. O X-Ray traceia requisições distribuídas, conectando o que aconteceu no Lambda com o que aconteceu no ECS e na chamada para a operadora, facilitando o diagnóstico de problemas de performance. Incidentes críticos chegam ao time via PagerDuty com severidades P1, P2 e P3 bem definidas.
