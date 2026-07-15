# Carregar dados fictícios no OCI Database with PostgreSQL

## Introdução

Neste laboratório, você preparará uma VM Linux na Oracle Cloud Infrastructure para apoiar a administração, a validação e a geração de dados fictícios no **OCI Database with PostgreSQL** criado nas etapas anteriores.

A VM será utilizada como ambiente auxiliar para instalar e configurar o **pgAdmin**, validar a conectividade com o banco PostgreSQL e criar o banco de trabalho que receberá os dados do laboratório. Em seguida, você instalará e executará o **pgEdge LoadGen**, ferramenta utilizada para gerar carga e dados de exemplo em PostgreSQL.

Ao final desta etapa, o ambiente estará pronto para popular o PostgreSQL com dados fictícios e validar a base transacional antes da ingestão no Oracle AI Data Platform.

### Objetivos

- criar uma VM Linux na OCI para executar ferramentas de apoio ao laboratório;
- instalar e configurar o pgAdmin para acessar o OCI Database with PostgreSQL;
- registrar o servidor PostgreSQL no pgAdmin e criar o banco que receberá a carga;
- instalar e compilar o `pgedge-loadgen` na VM Linux;
- executar a carga de dados fictícios no PostgreSQL;
- disponibilizar uma base transacional de exemplo para uso nas próximas etapas do pipeline.


## Tarefa 1: Criar uma VM Linux

Siga a sequência abaixo para criar a VM Linux que será utilizada para instalar as ferramentas de apoio ao laboratório.

1. Acesse **Compute > Instances** e clique em **Create instance**.

    ![Lista de instâncias de compute](images/001.png)

2. Informe o nome da instância, selecione o compartimento e escolha o domínio de disponibilidade.

    ![Informações básicas da instância](images/002.png)

3. Selecione a imagem do sistema operacional Linux que será utilizada na VM.

    ![Seleção da imagem da instância](images/003.png)

4. Confirme ou altere o shape da instância conforme a capacidade necessária para o laboratório.

    ![Configuração do shape da instância](images/004.png)

5. Revise as opções de segurança da instância.

    ![Configuração de segurança da instância](images/005.png)

6. Configure a rede da VM, selecionando VCN, sub-rede e as opções de conectividade.

    ![Configuração de rede da instância](images/006.png)

7. Configure a atribuição de endereço IP privado e, se necessário, habilite o endereço IPv4 público.

    ![Configuração de endereços IP](images/007.png)

8. Adicione ou gere as chaves SSH que serão utilizadas para acessar a VM.

    ![Configuração de chaves SSH](images/008.png)

9. Revise as opções de boot volume e block volumes antes de criar a instância.

    ![Configuração de volumes da instância](images/009.png)

10. Revise todas as configurações e clique em **Create** para iniciar o provisionamento.

    ![Revisão e criação da instância](images/010.png)

11. Aguarde a conclusão da work request de criação da instância.

    ![Work request da instância](images/011.png)

## Tarefa 2: Instalar e configurar o PGAdmin

Referência: [pgAdmin](https://www.pgadmin.org/)

Siga a sequência abaixo para instalar e configurar as ferramentas necessárias para gerar e validar os dados fictícios.

1. Acesse a VM Linux via SSH e execute os comandos iniciais de instalação dos pacotes necessários.

    ![Instalação de pacotes na VM Linux](images/012.png)

    ```
    sudo dnf -y install git make go
    git clone https://github.com/pgEdge/pgedge-loadgen.git
    cd pgedge-loadgen/
    make build
    ```

2. Execute os comandos de instalação e configuração do `pgedge-loadgen`.

    ![Configuração do pgedge-loadgen](images/013.png)

    ```
    sudo /usr/pgadmin4/bin/setup-web.sh
    ```

3. Libere as portas necessárias no firewall da VM e recarregue as regras.

    ![Liberação de portas no firewall](images/014.png)

    ```
    sudo su -
    firewall-cmd --permanent --add-port=80/tcp
    firewall-cmd --reload
    ```

4. Acesse a interface do pgAdmin para validar a conectividade com o PostgreSQL, use o e-mail e senha de quando rodou o script sudo /usr/pgadmin4/bin/setup-web.sh

    ![Tela de login do pgAdmin](images/015.png)

5. No pgAdmin, registre um novo servidor.

    ![Registro de servidor no pgAdmin](images/016.png)

6. Informe os dados gerais do servidor.

    ![Configuração geral do servidor](images/017.png)

7. Copie as informações de conexão do OCI Database with PostgreSQL.

    ![Informações de conexão do PostgreSQL](images/018.png)

8. Configure os parâmetros de conexão com host, porta, usuário e senha.

    ![Parâmetros de conexão do servidor](images/019.png)

9. Após conectar ao servidor, acesse a opção para criar um banco de dados.

    ![Criação de banco de dados no pgAdmin](images/020.png)

10. Informe o nome do banco que será utilizado para receber os dados fictícios.

    ![Definição do banco de dados](images/021.png)

## Tarefa 3: Instalar e configurar o pgedge-loadgen

Referência: [pgEdge/pgedge-loadgen](https://github.com/pgEdge/pgedge-loadgen)

Siga a sequência abaixo para instalar o `pgedge-loadgen` e gerar os dados fictícios no PostgreSQL.

1. Na VM Linux, instale as dependências necessárias para compilar e executar o `pgedge-loadgen`.

    ![Instalação de dependências do pgedge-loadgen](images/022.png)

    ```
    sudo dnf -y install git make go
    ```

2. Clone o repositório do `pgedge-loadgen` e execute o processo de build.

    ![Clone e build do pgedge-loadgen](images/023.png)
    ```
    git clone https://github.com/pgEdge/pgedge-loadgen.git
    cd pgedge-loadgen/
    make build
    ```

3. Crie ou revise o script de carga com os parâmetros de conexão do PostgreSQL.

    ![Script de criação dos dados fictícios](images/024.png)
    ```
    # Initialize database with 1GB of wholesale data
    pgedge-loadgen init \
        --app wholesale \
        --size 1GB \
        --connection "postgres://admin:Senha@primary.c.postgresql.us-phoenix-1.oci.oraclecloud.com:5432/handson"
    ```

4. Execute o script para carregar os dados fictícios e acompanhe a conclusão do processo.

    ![Execução da carga de dados fictícios](images/025.png)
    ```
    chmod +x create-data.sh 
    echo export PATH=\$PATH:/home/opc/pgedge-loadgen/bin >> ~/.bash_profile
    . ~/.bash_profile 
    ./create-data.sh
    ``` 

## Conclusão

Nesta etapa, você criou uma VM Linux na OCI, configurou o acesso administrativo ao OCI Database with PostgreSQL com o pgAdmin e preparou o ambiente para executar o `pgedge-loadgen`.

Com essa preparação, o laboratório passa a contar com uma base PostgreSQL populada com dados fictícios e pronta para validação. Esses dados serão usados nas próximas etapas como origem transacional para ingestão, transformação e publicação no Oracle AI Data Platform.

## Autoria

- **Autores** - Adriano Tanaka, Fábio Silva
- **Último Updated Por/Data** - Fábio Silva, Jul/2026
