# 03 — Camada 1: Canais e Entrada Segura

<!-- Adicione o print da página 03 aqui -->

Antes de qualquer requisição chegar a um serviço de negócio, ela passa por quatro componentes que formam a fronteira da plataforma. Esse diagrama detalha cada um deles e mostra como eles se encadeiam.

O **CloudFront** é o primeiro ponto de contato. Ele serve o frontend estático armazenado no S3 com baixa latência para qualquer região, e centraliza a proteção do WAF antes que o tráfego chegue mais fundo na infraestrutura. Não existe caminho para a aplicação que não passe por aqui.

O **WAF** fica logo atrás do CloudFront e aplica regras gerenciadas contra os ataques mais comuns — injeção de SQL, XSS, requisições malformadas, força bruta. Ele também aplica rate limiting por IP, o que protege tanto os serviços internos quanto as cotas das APIs das operadoras de saúde.

O **Cognito** cuida de tudo relacionado a identidade: criação de conta, login, MFA e emissão de tokens JWT. Cada token carrega as informações do perfil do usuário como claims — o perfil (paciente, recepção, médico, gestor) está dentro do token, então os serviços não precisam consultar o banco para saber quem está fazendo a requisição.

O **API Gateway** é o ponto único de entrada para todas as chamadas de API. Ele valida o token JWT antes de encaminhar qualquer requisição, aplica throttling por rota e por perfil, e roteia para o serviço correto. Tudo que passa por aqui gera logs que vão para o CloudWatch e para o S3, formando a trilha de auditoria de acesso.

Os quatro perfis de usuário chegam pelo mesmo caminho, mas com tokens diferentes, escopos diferentes e acessos diferentes. Um paciente autenticado não consegue acessar os endpoints de gestão de indicadores. Um médico não consegue alterar a agenda de outro médico. Essas restrições estão no token, validadas no Gateway, sem depender de lógica adicional nos serviços.
