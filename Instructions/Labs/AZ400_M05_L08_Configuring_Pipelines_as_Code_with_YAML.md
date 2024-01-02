---
lab:
  title: Configuring Pipelines as Code with YAML
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configuring Pipelines as Code with YAML

## Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure. Para obter detalhes, veja [Listar atribuições de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e atribuir funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

Muitas equipes preferem definir pipelines de build e lançamento usando YAML. Isso permite que elas acessem os mesmos recursos de pipeline que as pessoas que usam o designer visual, mas com um arquivo de marcação que pode ser gerenciado como qualquer outro arquivo de origem. Definições de build de YAML podem ser adicionadas a um projeto simplesmente adicionando os arquivos correspondentes à raiz do repositório. O Azure DevOps também fornece modelos padrão para tipos de projeto populares e um designer de YAML para simplificar o processo de definição de tarefas de build e lançamento.

## Objetivos

Após concluir este laboratório, você poderá:

- Configurar pipelines de CI/CD como código com YAML no Azure DevOps.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (ignorar se concluído) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto do Azure DevOps com base no **eShopOnWeb_MultiStageYAML** para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb_MultiStageYAML** e deixe os outros campos com padrões. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (ignorar se concluído) importar repositório Git do eShopOnWeb

Nesta tarefa você importará o repositório Git do eShopOnWeb que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra a organização do Azure DevOps e o projeto **eShopOnWeb_MultiStageYAML** criado anteriormente. Clique em **Repos>Arquivos**, **Importar um Repositório**. Selecione **Importar**. Na janela **Importar um repositório Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

2. O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários do laboratório.
    - O contêiner da pasta **.github** contém as definições de fluxo de trabalho do GitHub YAML.
    - A pasta **src** contém o site .NET 6 usado nos cenários do laboratório.

#### Tarefa 2: criar recursos do Azure

Nesta tarefa, você criará um aplicativo Web do Azure DevOps usando o portal do Azure.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função de proprietário na assinatura do Azure que você usará neste laboratório e tem a função de administrador global no locatário do Microsoft Entra associado a essa assinatura.
2. No portal do Azure, na barra de ferramentas, clique no ícone do **Cloud Shell** localizado ao lado da caixa de pesquisa.
3. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

    >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

    > **Observação**: para obter uma lista de regiões e os alias, execute o seguinte comando no Azure Cloud Shell – Bash:

    ```bash
    az account list-locations -o table
    ```

4. No prompt **Bash**, no painel do **Cloud Shell**, execute o seguinte comando para criar um grupo de recursos (substitua o espaço reservado `<region>` pelo nome da região do Azure mais próxima de você, como "centralus", "westeurope" ou outra região que escolher).

    ```bash
    LOCATION='<region>'
    ```

    ```bash
    RESOURCEGROUPNAME='az400m05l11-RG'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. Para criar um plano do serviço de aplicativo do Windows executando o seguinte comando:

    ```bash
    SERVICEPLANNAME='az400m05l11-sp1'
    az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
    ```

6. Crie um aplicativo Web com um nome exclusivo.

    ```bash
    WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **Observação**: registre o nome do aplicativo Web. Você precisará dela mais adiante neste laboratório.

7. Feche o Azure Cloud Shell, mas deixe o portal do Azure aberto no navegador.

### Exercício 1: configurar pipelines de CI/CD como código com YAML no Azure DevOps

Neste exercício, você configurará pipelines de CI/CD como código com YAML no Azure DevOps.

#### Tarefa 1: adicionar uma definição de build do YAML

Nesta tarefa, você adicionará uma definição de build do YAML ao projeto.

1. Volte para o painel **Pipelines** no hub **Pipelines**.
2. Na janela **Criar seu primeiro pipeline**, clique em **Criar pipeline**.

    > **Observação**: usaremos o assistente para criar uma nova definição de pipeline YAML com base no projeto.

3. Na página **Onde está seu código?**, clique na opção **Git do Azure Repos (YAML)**.
4. No painel **Selecionar um repositório**, clique em **eShopOnWeb_MultiStageYAML**.
5. No painel **Configurar o pipeline**, role para baixo e selecione **Arquivo YAML do Azure Pipelines existente**.
6. Na folha **Selecionar um Arquivo YAML existente**, especifique os seguintes parâmetros:
   - Branch: **principal**
   - Caminho: **.ado/eshoponWeb-ci.yml**
7. Clique em **Continuar** para salvar essas configurações.
8. Na tela **Revisar seu Pipeline YAML**, clique em **Executar** para iniciar o processo de pipeline de build.
9. Aguarde o pipeline de build ser concluído com sucesso. Ignore os avisos sobre o código-fonte, pois eles não são relevantes para este exercício de laboratório.

    > **Observação**: cada tarefa do arquivo YAML está disponível para revisão, incluindo avisos e erros.

#### Tarefa 2: adicionar entrega contínua à definição YAML

Nesta tarefa, você adicionará entrega contínua à definição baseada em YAML do pipeline criado na tarefa anterior.

> **Observação**: agora que os processos de build e teste foram bem-sucedidos, podemos adicionar a entrega à definição YAML.

1. No painel de execução de pipeline, clique no símbolo de reticências no canto superior direito e, no menu suspenso, clique em **Editar pipeline**.
2. No painel que exibe o conteúdo do arquivo **eShopOnWeb_MultiStageYAML/.ado/eshoponWeb-ci.yml**, navegue até o final do arquivo (linha 56) e pressione **Enter/Retornar** para adicionar uma nova linha vazia.
3. Na linha **57**, adicione o seguinte conteúdo para definir o estágio de **Versão** no pipeline YAML.

    > **Observação**: você pode definir os estágios necessários para organizar e acompanhar melhor o progresso do pipeline.

    ```yaml
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
    ```

4. Defina o cursor em uma nova linha no final da definição YAML.

    > **Observação**: este será o local onde novas tarefas serão adicionadas.

5. Na lista de tarefas no lado direito do painel de código, procure e selecione a tarefa **Implantar do Azure App Service**.
6. No painel de **implantação do Azure App Service**, especifique as seguintes configurações e clique em **Adicionar**:

    - Na lista suspensa **Assinatura do Azure**, selecione a assinatura do Azure na qual você implantou os recursos do Azure anteriormente no laboratório, clique em **Autorizar** e, quando solicitado, autentique-se usando a mesma conta de usuário usada durante a implantação de recursos do Azure.
    - Na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web implantado anteriormente no laboratório.
    - Na caixa de texto **Pacote ou pasta**, **atualize** o valor padrão para `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
7. Confirme as configurações no painel do Assistente clicando no botão **Adicionar**.

    > **Observação**: essa ação adicionará automaticamente a tarefa de implantação à definição de pipeline YAML.

8. O snippet de código adicionado ao editor deve ser semelhante ao abaixo, refletindo o nome para os parâmetros azureSubscription e WebappName:

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```

9. Validar a tarefa é listado como um filho da tarefa de **etapas**. Caso contrário, selecione todas as linhas da tarefa adicionada, dê um **Tab** duas vezes para recuá-la quatro espaços, de modo que ela seja listada como filha da tarefa de **etapas**.

    > **Observação**: o parâmetro **packageForLinux** é enganoso no contexto deste laboratório, mas é válido para Windows ou Linux.

    > **Observação**: por padrão, esses dois estágios executados de forma independente. Como resultado, a saída da build do primeiro estágio pode não estar disponível para o segundo sem alterações adicionais. Para implementar essas alterações, adicionaremos uma nova tarefa para baixar o artefato de implantação no início do estágio de implantação.

10. Coloque o cursor na primeira linha sob o nó **etapas** do estágio de **implantação** e pressione Enter/Retornar para adicionar uma nova linha vazia (Linha 64).
11. No painel **Tarefas**, procure e selecione a tarefa **Baixar artefatos de build**.
12. Especifique os seguintes parâmetros para esta tarefa:
    - Baixar artefatos produzidos por: **build atual**
    - Tipo de download: **artefato específico**
    - Nome do artefato: **selecione "Site" na lista** (ou **digite "Site"** diretamente se ele não aparecer automaticamente na lista)
    - Diretório de destino: **$(Build.ArtifactStagingDirectory)**
13. Clique em **Adicionar**.
14. O snippet de código adicionado deve ser semelhante ao abaixo:

    ```yaml
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
    ```

15. Se o recuo do YAML estiver desativado, com a tarefa adicionada ainda selecionada no editor, pressione a tecla **Tab** duas vezes para recuá-la quatro espaços.

    > **Observação**: aqui também você pode adicionar uma linha vazia antes e depois para facilitar a leitura.

16. Clique em **Salvar**, no painel **Salvar**, clique em **Salvar** novamente para confirmar a alteração diretamente no branch principal.

    > **Observação**: como nosso CI-YAML original não foi configurado para acionar automaticamente uma nova build, temos que iniciar esta manualmente.

17. No Azure DevOps, navegue até a guia **Pipelines** e selecione **Pipelines** novamente.
18. Abra o pipeline **EShopOnWeb_MultiStageYAML** e clique em **Executar Pipeline**.
19. Confirme a opção **Executar** no painel exibido.
20. Observe que os 2 estágios diferentes, **Criar principais soluções do .Net** e **Implantar no aplicativo Web do Azure** aparecem.
21. Aguarde até que o pipeline seja iniciado e até que ele conclua o estágio de build com êxito.
22. Quando o estágio de implantação quiser iniciar, você será solicitado com **Permissões Necessárias**, bem como uma barra laranja dizendo:

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

23. Clique em **Exibição**.
24. No painel **Aguardando revisão**, clique em **Permitir**.
25. Valide a mensagem na janela pop-up **Permitir** e confirme clicando em **Permitir**.
26. Isso inicia o estágio para implantar. Aguarde a conclusão.

     > **Observação**: se a implantação falhar, devido a um problema com a sintaxe do pipeline YAML, use isso como referência:

     ```yaml
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    # trigger:
    # - main
    
    resources:
      repositories:
        - repository: self
          trigger: none
    
    stages:
    - stage: Build
      displayName: Build .Net Core Solution
      jobs:
      - job: Build
        pool:
          vmImage: ubuntu-latest
        steps:
        - task: DotNetCoreCLI@2
          displayName: Restore
          inputs:
            command: 'restore'
            projects: '**/*.sln'
            feedsToUse: 'select'
    
        - task: DotNetCoreCLI@2
          displayName: Build
          inputs:
            command: 'build'
            projects: '**/*.sln'
        
        - task: DotNetCoreCLI@2
          displayName: Test
          inputs:
            command: 'test'
            projects: 'tests/UnitTests/*.csproj'
        
        - task: DotNetCoreCLI@2
          displayName: Publish
          inputs:
            command: 'publish'
            publishWebProjects: true
            arguments: '-o $(Build.ArtifactStagingDirectory)'
        
        - task: PublishBuildArtifacts@1
          displayName: Publish Artifacts ADO - Website
          inputs:
            pathToPublish: '$(Build.ArtifactStagingDirectory)'
            artifactName: Website
        
        - task: PublishBuildArtifacts@1
          displayName: Publish Artifacts ADO - Bicep
          inputs:
            PathtoPublish: '$(Build.SourcesDirectory)/.azure/bicep/webapp.bicep'
            ArtifactName: 'Bicep'
            publishLocation: 'Container'
    
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    
    ```

#### Tarefa 4: revisar o site implantado

1. Volte para a janela do navegador da Web que exibe o portal do Azure e navegue até a folha que exibe as propriedades do aplicativo Web do Azure.
2. Na folha aplicativo Web do Azure, clique em **Visão geral** e, na folha da visão geral, clique em **Procurar** para abrir o site em uma nova guia do navegador da Web.
3. Verifique se o site implantado carrega conforme o esperado na nova guia do navegador, mostrando o site de comércio eletrônico EShopOnWeb.

### Exercício 2: definir as configurações de ambiente para pipelines de CI/CD como código com o YAML no Azure DevOps

Neste exercício, você adicionará aprovações a um pipeline baseado em YAML no Azure DevOps.

#### Tarefa 1: configurar ambientes de pipeline

Os pipelines YAML como código não têm portões de qualidade/liberação como temos com os pipelines de lançamento clássico do Azure DevOps. No entanto, algumas semelhanças podem ser configuradas para pipelines YAML como código usando **ambientes**. Nesta tarefa, você usará esse mecanismo para configurar aprovações para o estágio de build.

1. No projeto **EShopOnWeb_MultiStageYAML** do Azure DevOps, navegue até **Pipelines**.
2. No menu Pipelines à esquerda, selecione **Ambientes**.
3. Clique em **Criar ambiente**.
4. No painel **Novo Ambiente**, adicione um nome para o ambiente chamado **Aprovações**.
5. Em **Recursos**, selecione **Nenhum**.
6. Confirme as configurações pressionando o botão **Criar**.
7. Assim que o ambiente for criado, clique nas "reticências" (...) ao lado do botão "Adicionar Recurso".
8. Selecione **Aprovações e verificações**.
9. Em **Adicionar sua primeira verificação**, selecione **Aprovações**.
10. Adicione o nome da sua conta de usuário do Azure DevOps ao campo **aprovadores**.

    > **Observação**: em um cenário real, isso refletiria o nome da sua equipe de DevOps trabalhando neste projeto.

11. Confirme as configurações de aprovação definidas pressionando o botão **Criar**.
12. Por último, precisamos adicionar as configurações necessárias de "ambiente: aprovações" ao código do pipeline YAML para o estágio de implantação. Para fazer isso, navegue até **Repos**, navegue até a pasta **.ado** e selecione o arquivo do pipeline como código **eshoponWeb-ci.yml**.
13. Na exibição Conteúdo, clique no botão **Editar** para alternar para o modo de edição.
14. Navegue até iniciar **Implantar trabalho** (-job: Deploy on Line 60)
15. Adicione uma nova linha vazia logo abaixo e adicione o seguinte snippet:

    ```yaml
      environment: approvals
    ```

    O snippet de código deve parecer com:

    ```yaml
     jobs:
      - job: Deploy
        environment: approvals
        pool:
          vmImage: 'windows-2019'
    ```

16. Como o ambiente é uma configuração específica de um estágio de implantação, ele não pode ser usado por "trabalhos". Portanto, temos que fazer algumas alterações adicionais na definição de trabalho atual.
17. Na Linha **60**, renomeie "-job: Deploy" para **-deployment: Deploy**
18. Em seguida, na linha **63** (vmImage: Windows-2019), adicione uma nova linha vazia.
19. Cole o seguinte snippet de YAML:

    ```yaml
        strategy:
          runOnce:
            deploy:
    ```

20. Selecione o snippet restante (Linha **67** até o final) e use a tecla **Tab** para corrigir o recuo do YAML.

    O snippet do YAML resultante deve ter esta aparência agora, refletindo o **estágio de implantação**:

    ```yaml
    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - deployment: Deploy
        environment: approvals
        pool:
          vmImage: 'windows-2019'
        strategy:
          runOnce:
            deploy:
              steps:
              - task: DownloadBuildArtifacts@0
                inputs:
                  buildType: 'current'
                  downloadType: 'single'
                  artifactName: 'Website'
                  downloadPath: '$(Build.ArtifactStagingDirectory)'
              - task: AzureRmWebAppDeployment@4
                inputs:
                  ConnectionType: 'AzureRM'
                  azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
                  appType: 'webApp'
                  WebAppName: 'eshoponWebYAML369825031'
                  packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```

21. Confirme as alterações no arquivo YAML de código clicando em **Confirmar** e clicando em **Confirmar** novamente no painel Confirmar exibido.
22. Navegue até o menu do Projeto do Azure DevOps à esquerda, selecione **Pipelines**, selecione **Pipelines** e observe o pipeline do **EshopOnWeb_MultiStageYAML** usado anteriormente.
23. Abra o pipeline.
24. Clique em **Executar Pipeline** para disparar uma nova execução de Pipeline, confirme clicando em **Executar**.
25. Assim como antes, o estágio de build inicia como esperado. Aguarde a conclusão.
26. Em seguida, como temos as *aprovações de ambientes* configuradas para o estágio de implantação, será solicitado uma confirmação de aprovação antes de começar.
27. Isso é visível na visualização do pipeline, onde lê-se **Aguardando (0/1 verificações aprovadas)**. Uma mensagem de notificação também é exibida dizendo que a **aprovação precisa ser revisada antes que essa execução possa continuar a implantar em um aplicativo Web do Azure**.
28. Clique no botão **Exibir** ao lado mensagem.
29. No painel de exibição **Verificações e validações manuais para implantar no aplicativo Web do Azure**, clique na mensagem **Aguardando aprovação**.
30. Clique em **Aprovar**.
31. Isso permite que o estágio de implantação inicie e implante com êxito o código-fonte do aplicativo Web do Azure.

    > **Observação:** embora este exemplo tenha usado apenas as aprovações, saiba que as outras verificações, como o Azure Monitor, a API REST etc., podem ser usadas de maneira semelhante.

### Exercício 3: Remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure recém-criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: remover os recursos do Azure Lab

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório com o objetivo de eliminar cobranças inesperadas.

1. No portal do Azure, abra a sessão do **Shell Bash** no painel do **Cloud Shell**.
2. Liste todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].name" --output tsv
    ```

3. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de os grupos de recursos serem removidos.

## Revisão

Neste laboratório, você configurou pipelines de CI/CD como código com YAML no Azure DevOps.
