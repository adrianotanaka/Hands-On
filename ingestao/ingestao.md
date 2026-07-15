# Instalar os notebooks de ingestão

## Introdução

Neste laboratório, você preparará o Oracle AI Data Platform Workbench para executar os notebooks de ingestão e transformação do fluxo de dados. Esta etapa conecta as configurações do PostgreSQL ao ambiente de processamento do AIDP e organiza os recursos necessários para gravar os dados nas camadas Bronze e Silver.

Você configurará as variáveis de ambiente no cluster compute, criará o catálogo e os schemas utilizados pelo laboratório e fará o upload dos notebooks no workspace. Ao final, o ambiente estará pronto para executar manualmente os notebooks e validar a ingestão dos dados transacionais no lakehouse.

### Objetivos

- configurar as variáveis de ambiente usadas pelos notebooks;
- iniciar o cluster compute que executará os workloads PySpark;
- criar o catálogo do laboratório no Master catalog;
- criar os schemas `bronze` e `silver`;
- fazer upload dos notebooks no workspace do AIDP;
- preparar o ambiente para execução manual dos notebooks de ingestão e transformação.


## Tarefa 1: Configurar as variáveis de ambiente e subir o cluster

1. No workspace do AIDP, acesse **Compute** para visualizar o cluster que será utilizado pelos notebooks.

    ![Tela de Compute do workspace](images/001.png)


2. Abra o menu de ações do cluster para acessar as opções disponíveis.

    ![Menu de ações do cluster](images/002.png)

3. Edite o cluster e configure as variáveis de ambiente necessárias para conexão com o PostgreSQL e gravação das camadas Bronze, Silver e Gold.

    ![Configuração das variáveis de ambiente do cluster](images/003.png)

    ```
    AIDP_CATALOG = handson
    BRONZE_SCHEMA = bronze
    SILVER_SCHEMA = silver
    BRONZE_WRITE_MODE = overwrite
    SILVER_WRITE_MODE = overwrite
    PG_HOST = ENDPOINT_DO_SEU_POSTGRESQL.oci.oraclecloud.com
    PG_PORT = 5432
    PG_DB = handson
    PG_SCHEMA = public
    PG_USER = pgadmin
    PG_SSLMODE = require
    PG_GOLD_HOST = ENDPOINT_DO_SEU_POSTGRESQL.oci.oraclecloud.com
    PG_GOLD_PORT = 5432
    PG_GOLD_DB = handson
    PG_GOLD_SCHEMA = gold
    PG_GOLD_USER = pgadmin
    PG_GOLD_SSLMODE = require
    PG_PASSWORD = SENHA_DO_SEU_POSTGRESQL
    ```

5. Após salvar as configurações, abra novamente o menu de ações do cluster e selecione a opção para iniciar o compute.

    ![Opção para iniciar o cluster](images/004.png)

6. Aguarde até que o cluster entre em estado de inicialização ou execução antes de iniciar os notebooks.

    ![Cluster em inicialização](images/005.png)

## Tarefa 2: Criar o catálogo e schemas

1. No menu lateral do Workbench, acesse **Master catalog**.

    ![Tela do Master catalog](images/006.png)

2. Clique em **Create catalog** e informe os dados do catálogo que será utilizado no laboratório.

    ![Criação do catálogo](images/007.png)

3. Confirme que o catálogo foi criado com sucesso.

    ![Catálogo criado com sucesso](images/008.png)

4. Acesse o catálogo criado e visualize a lista de schemas.

    ![Detalhes do catálogo criado](images/009.png)

5. Clique em **Create schema** e crie os schemas necessários para as camadas do laboratório.

    ![Criação de schema](images/010.png)

6. Confirme que os schemas `bronze` e `silver` foram criados e estão ativos.

    ![Schemas criados no catálogo](images/011.png)

## Tarefa 3: Fazer upload dos notebooks e executar manualmente

1. No menu lateral, acesse **Workspaces** e selecione o workspace do laboratório.

    ![Workspace do laboratório](images/012.png)

2. No conteúdo do workspace, clique em **Upload** para enviar os notebooks.

    ![Upload de arquivos no workspace](images/013.png)

3. Selecione os notebooks de ingestão e transformação que serão utilizados no laboratório.

    ![Seleção dos notebooks para upload](images/014.png)

4. Confirme que os notebooks foram enviados e aparecem na lista de conteúdo do workspace.

    ![Notebooks enviados para o workspace](images/015.png)

## Conclusão

Nesta etapa, você configurou o ambiente de execução dos notebooks no Oracle AI Data Platform Workbench. O cluster compute recebeu as variáveis necessárias para acessar o PostgreSQL e gravar os dados nas camadas do laboratório.

Você também criou o catálogo e os schemas que serão usados pelas tabelas Delta, além de disponibilizar os notebooks no workspace. Com isso, o ambiente está pronto para executar a ingestão dos dados transacionais, persistir a camada Bronze e iniciar as transformações para a camada Silver.

## Autoria

- **Autores** - Adriano Tanaka, Fábio Silva
- **Último Updated Por/Data** - Fábio Silva, Jul/2026
