### 6. Arquitetura-alvo em Nuvem (Azure)

Para operacionalizar a plataforma de experimentação adaptativa em produção, a arquitetura que desenvolvemos se propõe a utilizar uma abordagem *serverless* de microsserviços na **Azure** para garantir baixa latência na recomendação de ofertas e rastreabilidade total de MLOps.

* **Camada de Serving & API:** A camada de serving e a API demonstrável (construídas em `FastAPI`) são empacotadas em um container Docker e implantadas de forma escalável utilizando o **Azure Functions** (rodando no plano *Premium/Flex Consumption* com suporte a containers) ou através do **Azure Container Apps**, expondo o endpoint seguro por meio do **Azure API Management (APIM)**.
* **Armazenamento de Dados:** Os dados históricos dos clientes (vindos da base tratada) e as tabelas sintéticas (`offer_catalog`, `offer_events` e `delayed_rewards`) são armazenados e consultados em tempo real no **Azure Cosmos DB** (utilizando a API NoSQL), devido à sua performance garantida de milissegundos de dígito único e SLA global, essencial para ambientes bancários digitais.

O ciclo de vida de Machine Learning e o tracking de experimentos realizados localmente e em ambiente de retreino assíncrono utilizam o **Azure Machine Learning (Azure ML) Pipelines**, integrado a um servidor gerenciado do **MLflow** acoplado nativamente ao espaço de trabalho do Azure ML (com backend de artefatos apontando para uma conta do **Azure Blob Storage** e dados de métricas persistidos no **Azure SQL Database**).

* **Mensageria & Delays:** O reajuste dinâmico das fronteiras de exploração e o processamento em lotes das recompensas atrasadas (*delayed rewards*) ocorrem de forma assíncrona orientada a eventos através do **Azure Event Grid**, disparando execuções complementares no **Azure Functions**.
* **Observabilidade & Governança:** O monitoramento, auditoria de decisões e métricas de desvio (*drift*) utilizam nativamente o **Azure Monitor** (via **Log Analytics** e **Application Insights**), atendendo plenamente aos critérios de governança e observabilidade de ponta a ponta sem dependência de infraestruturas locais.

### 6. Arquitetura-alvo em Nuvem e FinOps (Azure)

Para operacionalizar a plataforma de experimentação adaptativa em produção, a arquitetura que desenvolvemos se propõe a utilizar uma abordagem *serverless* de microsserviços na **Azure** para garantir baixa latência na recomendação de ofertas e rastreabilidade total de MLOps.

* **Camada de Serving & API:** A camada de serving e a API demonstrável (construídas em `FastAPI`) são empacotadas em um container Docker e implantadas de forma escalável utilizando o **Azure Functions** (rodando no plano *Premium/Flex Consumption* com suporte a containers) ou através do **Azure Container Apps**, expondo o endpoint seguro por meio do **Azure API Management (APIM)**.
* **Armazenamento de Dados:** Os dados históricos dos clientes (vindos da base tratada) e as tabelas sintéticas (`offer_catalog`, `offer_events` e `delayed_rewards`) são armazenados e consultados em tempo real no **Azure Cosmos DB** (utilizando a API NoSQL), devido à sua performance garantida de milissegundos de dígito único e SLA global, essencial para ambientes bancários digitais.

O ciclo de vida de Machine Learning e o tracking de experimentos realizados localmente e em ambiente de retreino assíncrono utilizam o **Azure Machine Learning (Azure ML) Pipelines**, integrado a um servidor gerenciado do **MLflow** acoplado nativamente ao espaço de trabalho do Azure ML (com backend de artefatos apontando para uma conta do **Azure Blob Storage** e dados de métricas persistidos no **Azure SQL Database**).

* **Mensageria & Delays:** O reajuste dinâmico das fronteiras de exploração e o processamento em lotes das recompensas atrasadas (*delayed rewards*) ocorrem de forma assíncrona orientada a eventos através do **Azure Event Grid**, disparando execuções complementares no **Azure Functions**.
* **Observabilidade & Governança:** O monitoramento, auditoria de decisões e métricas de desvio (*drift*) utilizam nativamente o **Azure Monitor** (via **Log Analytics** e **Application Insights**), atendendo plenamente aos critérios de governança e observabilidade de ponta a ponta sem dependência de infraestruturas locais.

#### 📊 Estimativa de Gastos Mensais (Cenário de MVP/POC)

Para fins demonstrativos e validação de viabilidade da solução, a tabela abaixo apresenta a estimativa de custos projetada para um cenário inicial de volumetria controlada (aproveitando os benefícios do *Azure Free Account*):

| Camada da Solução | Serviço Azure | Uso Estimado Base (Mensal) | Custo Estimado (USD) | Justificativa / Configuração |
| :--- | :--- | :--- | :--- | :--- |
| **Serving & API** | Azure API Management | 500 mil requisições | $1.75 | Camada serverless do APIM (Plano Consumption) para otimização de custo. |
| **Serving & API** | Azure Functions | 500 mil execuções (256MB / 200ms) | $0.00 | 100% coberto pelo *Free Account* da Azure (limite de até 1M de execuções). |
| **Armazenamento** | Azure Cosmos DB | Modo Serverless (5GB armazenados + RU/s leves) | $2.50 | Banco NoSQL serverless de ultra-baixa latência para o vetor de contexto. |
| **Mensageria & Delays** | Azure Event Grid | 200 mil eventos de conversão tardia | $0.12 | Ingestão e roteamento de eventos assíncronos baseados em pub/sub. |
| **MLOps & Infraestrutura** | Azure Container Apps | 1 réplica ativa 24/7 (0.5 vCPU / 1GB RAM) | $15.00 | Hospedagem serverless simplificada de containers para o servidor MLflow. |
| **MLOps & Infraestrutura** | Azure SQL Database | Camada Serverless (General Purpose - Gen5) + 20GB | $12.00 | Banco relacional auto-escalável para metadados e métricas do MLflow. |
| **MLOps & Infraestrutura** | Azure Blob Storage | 10 GB de armazenamento padrão Hot (LRS) | $0.20 | Repositório de arquivos brutos, tabelas Parquet e artefatos binários. |
| **MLOps & Infraestrutura** | Azure Machine Learning | Execuções esporádicas de pipelines de retreino | $5.00 | Cobrança estrita por tempo de uso do cluster gerenciado durante retreinos. |
| **Governança** | Azure Monitor | Ingestão padrão de telemetria e logs (5GB) | $2.30 | Centralização de telemetria da API (App Insights), reason codes e drifts. |
| **Custo Total Projetado** | | | **~$38.87** | **Excelente eficiência de custo para implantação de MVP.** |

*Nota de FinOps: O maior peso financeiro reside na sustentação da governança e rastreabilidade (Container Apps + Azure SQL para a infraestrutura do MLflow). A camada de atendimento aos canais digitais (Serving) escala de forma linear, garantindo que o custo de computação caia a zero caso não haja tráfego de usuários nas plataformas, assegurando alta eficiência financeira.*