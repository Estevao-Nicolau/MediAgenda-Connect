# 05 — Camada 3: Integração Desacoplada com Operadoras

<img width="2242" height="1286" alt="06 — Camada 4_ Eventos, Filas e Resiliência drawio" src="https://github.com/user-attachments/assets/c5d28399-4ea2-457d-a787-1b573be63364" />


Este diagrama detalha o diferencial arquitetural da plataforma. A premissa é simples: operadoras de saúde são heterogêneas. Algumas têm APIs REST modernas seguindo o padrão FHIR. Outras ainda usam TISS com SOAP XML, um padrão legado mas amplamente adotado no mercado brasileiro. Outras têm APIs proprietárias sem padrão definido. Se o core de agendamento tivesse que lidar com essa heterogeneidade diretamente, qualquer mudança em uma operadora exigiria mexer no fluxo principal — um risco alto demais para um sistema de saúde em produção.

A solução é uma camada de integração com três componentes principais.

A **Integration Facade** é o único ponto de contato entre o core e o mundo externo. Ela expõe um contrato interno único o core chama métodos como `verificarElegibilidade` ou `solicitarAutorizacao` sem saber nada sobre quem vai responder ou como.

O **Modelo Canônico** é a representação interna das entidades da plataforma: Paciente, Plano, Procedimento, Guia e Status. Quando a resposta de uma operadora chega independentemente do formato ela é traduzida para esse modelo antes de seguir para o core. Isso evita que formatos externos contaminem a lógica de negócio.

Os **Adapters** são implementações específicas por operadora e por protocolo. O Adapter REST/FHIR sabe como montar requisições e interpretar respostas no padrão FHIR R4. O Adapter TISS/XML lida com o envelope SOAP e o XML do padrão TISS. O Adapter Proprietário encapsula qualquer particularidade de uma operadora específica. Adicionar suporte a uma nova operadora significa criar um novo adapter nada mais.

A resiliência é tratada via SQS com retry e backoff exponencial. As credenciais de cada operadora ficam isoladas no Secrets Manager uma comprometida não expõe as demais. Cada interação com uma operadora gera um registro de auditoria com request, response e timestamp, essencial para rastreabilidade e para resolver disputas sobre o resultado de uma autorização.
