# Criar e agendar jobs no Oracle AI Data Platform (AIDP)

## Introdução

Neste laboratório, você criará um job de workflow no Oracle AI Data Platform (AIDP), incluirá tarefas de notebook e definirá a ordem de execução entre elas. Ao final, criará um agendamento e verificará o histórico de execuções.

O exemplo organiza o pipeline em três etapas: ingestão de dados para a camada **Bronze**, transformações para a camada **Silver** e publicação na camada **Gold**. A mesma abordagem pode ser usada para qualquer conjunto de notebooks do seu workspace.

**Tempo estimado:** 15 minutos

### Objetivos

Ao concluir este laboratório, você será capaz de:

* Criar um job de workflow no AIDP.
* Adicionar tarefas que executam notebooks do workspace.
* Definir dependências para garantir a ordem correta do pipeline.
* Configurar limites de concorrência, timeout e novas tentativas.
* Agendar o job e acompanhar suas execuções.

### Pré-requisitos

* Acesso ao Oracle AI Data Platform Workbench e a um workspace.
* Permissão para criar jobs e usar o cluster de compute desejado.
* Notebooks já salvos no workspace. Neste exemplo, serão usados notebooks de ingestão, transformação e publicação.

## Tarefa 1: criar o job

1. No menu lateral do seu workspace, clique em **Workflow**.
2. Na aba **Jobs**, clique em **Create job** (Ou no sinal de +).

    ![Página Workflow com o botão Create job](images/workflow1.png)

3. Preencha os campos do painel **Create Job**:

    * **Job Name**: informe um nome claro para o fluxo, por exemplo `Bronze_to_Silver`.
    * **Description**: descreva brevemente o objetivo do job (opcional, mas recomendado).
    * **Job location**: mantenha ou escolha o diretório no workspace onde a definição será salva.
    * **Max concurrent Runs**: defina quantas execuções do mesmo job podem ocorrer simultaneamente. Para pipelines que escrevem nos mesmos destinos, use `1` para evitar sobreposição.

4. Clique em **Create**.

    ![Formulário Create Job preenchido](images/workflow2.png)

> **Observação:** criar o job não o executa. Primeiro é necessário incluir e configurar as tarefas do workflow.

## Tarefa 2: adicionar e configurar as tarefas

1. Na aba **Tasks**, clique em **Add task**.
2. No painel **Task details**, informe um nome descritivo, como `bronze_ingestao_postgresql_delta`.
3. Em **Task type**, selecione **Notebook task**.
4. Em **Source**, mantenha **Workspace** e clique em **Browse** para selecionar o notebook correspondente, aqui você também pode escolher arquivos que estão em um GitFolder.
5. Escolha o **Cluster** que executará o notebook, cada tarefa (ou step) pode ser vinculada a um cluster diferente.
6. Se aplicável, configure **Execution timeout**, **Retries** e **Retry on timeout**, se for uma tarefa de ingestão continua você pode marcar a opção de Streaming.

    ![Configuração de uma tarefa do tipo Notebook task](images/workflow3.png)

7. Repita o procedimento para as demais etapas do pipeline. No exemplo, as tarefas são:

    1. `bronze_ingestao_postgresql_delta` — carrega os dados para Bronze.
    2. `silver_transformacoes_delta` — transforma os dados na camada Silver.
    3. `gold_publicacao_postgresql` — publica o resultado na camada Gold.

## Tarefa 3: definir as dependências do fluxo

As dependências impedem que uma etapa seja iniciada antes de sua predecessora terminar nas condições definidas.

1. Selecione a tarefa que deve executar depois de outra, por exemplo `bronze_ingestao_postgresql_delta`.
2. No campo **Depends on**, escolha a tarefa predecessora — no exemplo, `silver_transformacoes_delta`.
3. Em **Run if**, mantenha **All dependencies have executed and succeeded** para executar a tarefa apenas quando todas as dependências tiverem sucesso.
4. Configure as outras dependências de acordo com o desenho do pipeline.

    ![Dependência entre tarefas configurada no workflow](images/workflow4.png)

> **Importante:** confirme a direção das setas no canvas. A ordem precisa refletir a sequência lógica do processamento. No exemplo, a transformação deve depender da ingestão, e a publicação deve depender da transformação.

## Tarefa 4: configurar parâmetros e opções do job

1. Abra a aba **Details** do job.
2. Na seção **Run**, revise **Max concurrent Runs** e, se necessário, defina um **Execution timeout** para todo o job.
3. Em **Job parameters**, adicione parâmetros reutilizáveis que serão passados às tarefas, por exemplo ambiente, data de referência ou caminho de saída.
4. Na seção **Compute**, confirme que o cluster exibido está ativo e pode executar as tarefas selecionadas.

    ![Aba Details com opções de execução e parâmetros](images/workflow6.png)

## Tarefa 5: criar um agendamento

1. Ainda na aba **Details**, localize a seção **Schedule** e clique em **Add**.
2. No painel **Create Schedule**, configure:

    * **Schedule Status**: selecione **Active** para habilitar o agendamento assim que ele for criado.
    * **Time Zone**: selecione o fuso horário que deve reger a execução. Escolha o fuso do processo de negócio; não assuma UTC se os horários operacionais forem locais.
    * **Schedule Type**: selecione **Calendar**.
    * **Frequency**: escolha a frequência, como diária, semanal ou mensal.
    * **Weekday**: quando a frequência exigir, escolha o dia da semana.
    * **Start time**: informe o horário no formato de 24 horas, considerando o fuso selecionado.

3. Revise a frase de resumo mostrada abaixo dos campos: ela traduz a regra que será aplicada.
4. Clique em **Create**.

    ![Painel Create Schedule com frequência e horário](images/workflow7.png)

> **Dica:** para testar o pipeline antes de automatizá-lo, deixe o agendamento inativo ou crie-o para um horário futuro e use **Run now** para uma execução controlada.

## Tarefa 6: executar e monitorar o job

1. Para uma validação imediata, clique em **Run now** no canto superior direito do job.
2. Abra a aba **Runs** para acompanhar cada execução.
3. Use os filtros de período e **Run status** para localizar execuções específicas.
4. Verifique as colunas **Trigger**, **Duration**, **Start time**, **End time** e **Status**. O gatilho indica se a execução foi manual ou disparada pelo agendamento.
5. Clique em **View** na execução desejada para inspecionar os detalhes e diagnosticar falhas ou tarefas ignoradas.

    ![Histórico de execuções na aba Runs](images/workflow8.png)

## Conclusão

Você criou um job no Oracle AIDP, conectou tarefas de notebook em um workflow, definiu suas dependências e configurou uma agenda de execução. Antes de colocar o processo em produção, valide uma execução manual e confirme que o cluster, os notebooks, os parâmetros e o fuso horário estão corretos.

## Saiba mais

* [Oracle AI Data Platform](https://www.oracle.com/database/ai-data-platform/)
* [Oracle LiveLabs](https://livelabs.oracle.com/)
