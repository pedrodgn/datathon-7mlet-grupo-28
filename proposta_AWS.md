### 6. Arquitetura-alvo em Nuvem (AWS)

Para operacionalizar a plataforma de experimentação adaptativa em produção, a arquitetura que desenvolvemos se propõe a utilizar uma abordagem *serverless* de microsserviços na **AWS** para garantir baixa latência na recomendação de ofertas e rastreabilidade total de MLOps. 

* **Camada de Serving & API:** A camada de serving e a API demonstrável (construídas em `FastAPI`) são empacotadas em um container Docker e implantadas de forma escalável utilizando o **AWS Lambda** (através do *AWS Lambda Web Adapter*) ou **AWS Fargate (ECS)**, expondo o endpoint seguro por meio do **Amazon API Gateway**.
* **Armazenamento de Dados:** Os dados históricos dos clientes (vindos da base tratada) e as tabelas sintéticas (`offer_catalog`, `offer_events` e `delayed_rewards`) são armazenados e consultados em tempo real no **Amazon DynamoDB** devido à sua performance de milissegundos de dígito único, essencial para ambientes bancários digitais.

O ciclo de vida de Machine Learning e o tracking de experimentos realizados localmente e em ambiente de retreino assíncrono utilizam o **Amazon SageMaker Pipelines**, integrado a um servidor gerenciado do **MLflow** hospedado no **AWS Fargate** (com backend de artefatos apontando para um bucket do **Amazon S3** e dados de métricas persistidos no **Amazon RDS**).

* **Mensageria & Delays:** O reajuste dinâmico das fronteiras de exploração e o processamento em lotes das recompensas atrasadas (*delayed rewards*) ocorrem de forma assíncrona orientada a eventos através do **Amazon EventBridge**, disparando funções complementares no **AWS Lambda**.
* **Observabilidade & Governança:** O monitoramento, auditoria de decisões e métricas de desvio (*drift*) utilizam nativamente o **Amazon CloudWatch**, atendendo plenamente aos critérios de governança e observabilidade de ponta a ponta sem dependência de infraestruturas locais.

### 6. Arquitetura-alvo em Nuvem e FinOps (AWS)

Para operacionalizar a plataforma de experimentação adaptativa em produção, a arquitetura que desenvolvemos se propõe a utilizar uma abordagem *serverless* de microsserviços na **AWS** para garantir baixa latência na recomendação de ofertas e rastreabilidade total de MLOps. 

* **Camada de Serving & API:** A camada de serving e a API demonstrável (construídas em `FastAPI`) são empacotadas em um container Docker e implantadas de forma escalável utilizando o **AWS Lambda** (através do *AWS Lambda Web Adapter*) ou **AWS Fargate (ECS)**, expondo o endpoint seguro por meio do **Amazon API Gateway**.
* **Armazenamento de Dados:** Os dados históricos dos clientes (vindos da base tratada) e as tabelas sintéticas (`offer_catalog`, `offer_events` e `delayed_rewards`) são armazenados e consultados em tempo real no **Amazon DynamoDB** devido à sua performance de milissegundos de dígito único, essencial para ambientes bancários digitais.

O ciclo de vida de Machine Learning e o tracking de experimentos realizados localmente e em ambiente de retreino assíncrono utilizam o **Amazon SageMaker Pipelines**, integrado a um servidor gerenciado do **MLflow** hospedado no **AWS Fargate** (com backend de artefatos apontando para um bucket do **Amazon S3** e dados de métricas persistidos no **Amazon RDS**).

* **Mensageria & Delays:** O reajuste dinâmico das fronteiras de exploração e o processamento em lotes das recompensas atrasadas (*delayed rewards*) ocorrem de forma assíncrona orientada a eventos através do **Amazon EventBridge**, disparando funções complementares no **AWS Lambda**.
* **Observabilidade & Governança:** O monitoramento, auditoria de decisões e métricas de desvio (*drift*) utilizam nativamente o **Amazon CloudWatch**, atendendo plenamente aos critérios de governança e observabilidade de ponta a ponta sem dependência de infraestruturas locais.

#### 📊 Estimativa de Gastos Mensais (Cenário de MVP/POC)

Para fins demonstrativos e validação de viabilidade da solução, a tabela abaixo apresenta a estimativa de custos projetada para um cenário inicial de volumetria controlada (aproveitando os benefícios do *AWS Free Tier*):

| Camada da Solução | Serviço AWS | Uso Estimado Base (Mensal) | Custo Estimado (USD) | Justificativa / Configuração |
| :--- | :--- | :--- | :--- | :--- |
| **Serving & API** | Amazon API Gateway | 500 mil requisições | $1.75 | Implementação via HTTP API para otimização de custo. |
| **Serving & API** | AWS Lambda | 500 mil execuções (256MB / 200ms) | $0.00 | 100% coberto pelo *Free Tier* da AWS (limite de até 1M). |
| **Armazenamento** | Amazon DynamoDB | Modo On-Demand (5GB armazenados + leitura/escrita leves) | $1.25 | Entrega de contexto em milissegundos de dígito único. |
| **Mensageria & Delays** | Amazon EventBridge | 200 mil eventos de conversão tardia | $0.20 | Roteamento nativo e assíncrono de eventos de recompensa. |
| **MLOps & Infraestrutura** | AWS Fargate (ECS) | 1 tarefa ativa 24/7 (0.5 vCPU / 1GB RAM) | $18.00 | Servidor do MLflow ativo para tracking contínuo. |
| **MLOps & Infraestrutura** | Amazon RDS | Instância pequena (`db.t4g.micro`) + 20GB Storage | $15.00 | Banco relacional para persistência de métricas do MLflow. |
| **MLOps & Infraestrutura** | Amazon S3 | 10 GB de armazenamento padrão | $0.23 | Repositório de artefatos de modelos e logs consolidados. |
| **MLOps & Infraestrutura** | Amazon SageMaker | Execuções esporádicas de pipelines de retreino | $5.00 | Cobrança estrita por tempo sob demanda de computação ML. |
| **Governança** | Amazon CloudWatch | Ingestão padrão de logs e alarmes básicos | $1.50 | Auditoria de logs das decisões, reason codes e drifts. |
| **Custo Total Projetado** | | | **~$42.93** | **Excelente eficiência de custo para implantação de MVP.** |

*Nota de FinOps: O maior peso financeiro reside na sustentação da governança e rastreabilidade (Fargate + RDS para a infraestrutura do MLflow). A camada de atendimento aos canais digitais (Serving) escala de forma linear, garantindo que o custo de computação caia a zero caso não haja tráfego de usuários nas plataformas, assegurando alta eficiência financeira.*