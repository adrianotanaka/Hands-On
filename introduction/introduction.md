# Laboratório AIDP

## Introdução

Neste laboratório, você implementará um fluxo completo de engenharia de dados utilizando o **Oracle AI Data Platform (AIDP)**. O cenário parte de uma base transacional PostgreSQL e percorre as etapas de ingestão, organização, transformação e disponibilização dos dados para consumo analítico.

A solução utiliza a arquitetura em camadas **Bronze, Silver e Gold**. Os dados brutos são extraídos do PostgreSQL e armazenados como tabelas Delta na camada Bronze. Em seguida, são tratados, padronizados e integrados com PySpark para formar dimensões e fatos na camada Silver. Por fim, são criadas agregações e métricas de negócio na camada Gold, publicadas novamente no PostgreSQL.

Durante a execução, você conhecerá os principais componentes de infraestrutura e dados necessários para conectar a origem ao AIDP e construir um pipeline reproduzível, organizado e preparado para análises.

## Objetivos

Ao concluir este laboratório, você será capaz de:

- criar um compartimento para organizar e isolar os recursos utilizados no laboratório;
- configurar uma Virtual Cloud Network (VCN) para suportar a comunicação entre os componentes da solução;
- compreender como o PostgreSQL atua como origem transacional e destino da camada Gold;
- conectar o AIDP ao PostgreSQL por meio de JDBC;
- ingerir tabelas da origem e armazená-las no formato Delta na camada Bronze;
- utilizar PySpark para limpar, padronizar, relacionar e enriquecer os dados;
- construir dimensões e fatos analíticos na camada Silver;
- gerar indicadores e visões de negócio na camada Gold;
- executar os notebooks na ordem correta e compreender as dependências entre as etapas do pipeline;
- identificar onde os dados são persistidos em cada camada da arquitetura.

## Arquitetura

A arquitetura do laboratório combina recursos de infraestrutura da Oracle Cloud, uma origem PostgreSQL e os recursos de processamento e armazenamento do Oracle AI Data Platform.

O fluxo lógico é composto pelas seguintes etapas:

1. O PostgreSQL disponibiliza as tabelas transacionais utilizadas como fonte de dados.
2. Um notebook PySpark realiza a ingestão via JDBC e grava os dados brutos em tabelas Delta na camada Bronze do AIDP.
3. Um segundo notebook transforma os dados Bronze em dimensões e fatos Delta na camada Silver.
4. Um terceiro notebook produz agregações e indicadores de negócio a partir da camada Silver.
5. As tabelas Gold são publicadas em um schema analítico no PostgreSQL para consumo por aplicações e ferramentas de análise.

## Componentes

- **Compartimento:** organiza e isola os recursos OCI utilizados durante o laboratório.
- **VCN:** fornece a base de conectividade de rede entre os componentes da solução.
- **PostgreSQL:** funciona como origem dos dados transacionais e como destino das tabelas analíticas da camada Gold.
- **Oracle AI Data Platform:** fornece o ambiente de Workbench, o processamento distribuído com Spark/PySpark e o catálogo de tabelas Delta das camadas Bronze e Silver.
- **Notebooks PySpark:** implementam, de forma sequencial, a ingestão, as transformações e a publicação dos dados.

## Fluxo Proposto

O laboratório implementa três notebooks, executados obrigatoriamente nesta ordem:

| Etapa | Processamento | Resultado |
|---|---|---|
| **Bronze** | Ingestão das tabelas PostgreSQL via JDBC e inclusão de metadados técnicos | Tabelas Delta com os dados preservados em sua estrutura de origem |
| **Silver** | Padronização, integração, enriquecimento e modelagem com PySpark | Dimensões e fatos Delta preparados para análise |
| **Gold** | Agregação dos dados Silver e cálculo de indicadores de negócio | Tabelas analíticas publicadas no PostgreSQL |

Ao final, você terá construído um pipeline de dados de ponta a ponta: da captura dos dados operacionais até a disponibilização de informações consolidadas sobre vendas, clientes, estoque, pedidos, produtos e indicadores financeiros.

Para consultar o detalhamento das tabelas, parâmetros, notebooks e regras de transformação, acesse o documento [Fluxo Implementado — Bronze, Silver e Gold](../fluxo.md).
