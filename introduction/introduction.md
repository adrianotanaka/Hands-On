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

![Arquitetura do laboratório AIDP](images/arquitetura-aidp.png)

O PostgreSQL disponibiliza as tabelas transacionais da origem. O AIDP executa os três notebooks de forma sequencial, mantém as camadas Bronze e Silver no formato Delta e publica os produtos analíticos da camada Gold em um schema PostgreSQL.

## Componentes

- **Compartimento:** organiza e isola os recursos OCI utilizados durante o laboratório.
- **VCN:** fornece a base de conectividade de rede entre os componentes da solução.
- **PostgreSQL:** funciona como origem dos dados transacionais e como destino das tabelas analíticas da camada Gold.
- **Oracle AI Data Platform:** fornece o ambiente de Workbench, o processamento distribuído com Spark/PySpark e o catálogo de tabelas Delta das camadas Bronze e Silver.
- **Notebooks PySpark:** implementam, de forma sequencial, a ingestão, as transformações e a publicação dos dados.

## Fluxo Proposto

O fluxo utiliza três notebooks PySpark. Cada etapa depende dos dados produzidos pela anterior e, por isso, deve ser executada na seguinte ordem:

| Ordem | Notebook | Função |
|---|---|---|
| 1 | `01_bronze_ingestao_postgresql_delta.ipynb` | Ingerir as tabelas PostgreSQL e gravá-las como tabelas Delta na camada Bronze |
| 2 | `02_silver_transformacoes_delta.ipynb` | Transformar os dados Bronze em dimensões e fatos Delta na camada Silver |
| 3 | `03_gold_publicacao_postgresql.ipynb` | Agregar os dados Silver e publicar as tabelas Gold no PostgreSQL |

### Etapa 1 — Ingestão Bronze

O primeiro notebook lê, via JDBC, nove tabelas transacionais do PostgreSQL. A estrutura original é preservada e cada tabela recebe um nome com o prefixo `bronze_` no catálogo do AIDP.

| Tabela PostgreSQL | Tabela Bronze Delta |
|---|---|
| `warehouse` | `bronze_warehouse` |
| `district` | `bronze_district` |
| `customer` | `bronze_customer` |
| `history` | `bronze_history` |
| `item` | `bronze_item` |
| `stock` | `bronze_stock` |
| `orders` | `bronze_orders` |
| `new_orders` | `bronze_new_orders` |
| `order_line` | `bronze_order_line` |

Além dos dados de origem, o notebook adiciona as colunas técnicas `ingestion_timestamp`, `source_system`, `source_table`, `batch_id` e `record_hash`. As tabelas são gravadas no namespace `<AIDP_CATALOG>.<BRONZE_SCHEMA>.<bronze_table>`.

### Etapa 2 — Transformações Silver

O segundo notebook lê as tabelas Bronze, padroniza nomes e tipos, aplica os relacionamentos entre as entidades e produz um modelo analítico composto por dimensões e fatos.

| Tabela Silver | Principais fontes | Conteúdo |
|---|---|---|
| `dim_location` | `bronze_warehouse`, `bronze_district` | Armazéns, distritos, localização, impostos e valores acumulados |
| `dim_customer` | `bronze_customer` | Cadastro, crédito, desconto, saldo e histórico dos clientes |
| `dim_item` | `bronze_item` | Catálogo, nome, preço e atributos dos produtos |
| `fact_inventory` | `bronze_stock`, `bronze_item` | Quantidades, movimentações e valor do inventário |
| `fact_order_header` | `bronze_orders`, `bronze_customer` | Cabeçalho dos pedidos e identificação do cliente |
| `fact_order_line` | `bronze_order_line`, `bronze_orders`, `bronze_item` | Itens, quantidades, valores e dados de entrega dos pedidos |
| `fact_payment` | `bronze_history`, `bronze_customer` | Pagamentos, clientes, datas e valores |
| `fact_pending_order` | `bronze_new_orders`, `bronze_orders` | Pedidos pendentes, idade do backlog e status |

As tabelas Silver são armazenadas como Delta no namespace `<AIDP_CATALOG>.<SILVER_SCHEMA>.<silver_table>`.

### Etapa 3 — Publicação Gold

O terceiro notebook utiliza as dimensões e fatos Silver para calcular indicadores de negócio e publicar seis tabelas analíticas no schema `<PG_GOLD_SCHEMA>` do PostgreSQL.

| Tabela Gold | Finalidade e principais indicadores |
|---|---|
| `gold_sales_summary` | Pedidos, quantidade vendida, receita e ticket médio por data e localidade |
| `gold_customer_360` | Visão consolidada de compras, pagamentos, saldo, limite e histórico do cliente |
| `gold_inventory_health` | Quantidade, valor do inventário e status `OUT_OF_STOCK`, `LOW_STOCK` ou `OK` |
| `gold_order_fulfillment` | Pedidos pendentes e entregues, atendimento local e prazo médio de entrega |
| `gold_product_performance` | Receita, quantidade vendida, preço médio e rankings por produto |
| `gold_financial_kpis` | Receita estimada, pagamentos, valores YTD e taxas por localidade |

### Persistência e parâmetros principais

| Camada | Formato | Destino | Configuração padrão |
|---|---|---|---|
| Bronze | Delta | AIDP Master Catalog | Catálogo `main`, schema `bronze`, modo `overwrite` |
| Silver | Delta | AIDP Master Catalog | Catálogo `main`, schema `silver`, modo `overwrite` |
| Gold | PostgreSQL | Schema configurado por `PG_GOLD_SCHEMA` | Schema `gold`, porta `5432`, SSL `require`, modo `overwrite` |

A conexão com o PostgreSQL utiliza variáveis de ambiente, sem credenciais fixas nos notebooks. O driver JDBC utilizado é o PostgreSQL `42.7.4`. Cada notebook retorna um payload JSON ao final da execução e não encerra a sessão Spark, permitindo o uso compartilhado do cluster no Workbench.

Ao final, você terá construído um pipeline de dados de ponta a ponta: da captura dos dados operacionais até a disponibilização de informações consolidadas sobre vendas, clientes, estoque, pedidos, produtos e indicadores financeiros.
