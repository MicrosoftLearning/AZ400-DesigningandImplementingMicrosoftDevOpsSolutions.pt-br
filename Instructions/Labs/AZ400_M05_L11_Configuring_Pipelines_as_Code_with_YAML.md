---
lab:
  title: Configure pipelines como código com YAML
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configure pipelines como código com YAML

# Manual de laboratório do aluno

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador compatível com o Azure DevOps.](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou do Azure AD com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Azure AD associado à assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) e [Designar e atribuir funções de administrador no Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Visão geral do laboratório

Muitas equipes preferem definir pipelines de compilação e lançamento usando YAML. Isso permite que elas acessem os mesmos recursos de pipeline que as pessoas que usam o designer visual, mas com um arquivo de marcação que pode ser gerenciado como qualquer outro arquivo de origem. Definições de compilação de YAML podem ser adicionadas a um projeto simplesmente adicionando os arquivos correspondentes à raiz do repositório. O Azure DevOps também fornece modelos padrão para tipos de projeto populares e um designer de YAML para simplificar o processo de definição de tarefas de compilação e lançamento.

## Objetivos

Após concluir este laboratório, você poderá:

- Configurar pipelines de CI/CD como código com YAML no Azure DevOps.

## Tempo estimado: 60 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório, que consistem em um novo projeto do Azure DevOps com um repositório baseado no [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarefa 1: (ignorar se concluído) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto do Azure DevOps no **eShopOnWeb_MultiStageYAML** para ser usado por vários laboratórios.

1.  No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao seu projeto o nome **eShopOnWeb_MultiStageYAML** e deixe os outros campos com padrões. Clique em **Criar**.

    ![Criar Projeto](images/create-project.png)

#### Tarefa 2: (ignorar se concluído) importar repositório Git do eShopOnWeb

Nesta tarefa, você importará o repositório Git do eShopOnWeb que será usado por vários laboratórios.

1.  No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto criado anteriormente do **eShopOnWeb_MultiStageYAML**. Clique em **Repos>Files**, **Importar um repositório**. Selecione **Importar**. Na janela, **Importar um repositório Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

    ![Importar repositório](images/import-repo.png)

1.  O repositório está organizado da seguinte forma:
    - A pasta **.ado** contém os pipelines YAML do Azure DevOps
    - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces)
    - A pasta **.azure** contém a infraestrutura Bicep&ARM como modelos de código usados em alguns cenários do laboratório.
    - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
    - A pasta **src** contém o site .NET 6 usado nos cenários do laboratório.

#### Tarefa 2: criar recursos do Azure

Nesta tarefa, você criará um aplicativo Web do Azure DevOps usando o portal do Azure.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Azure AD associado a essa assinatura.
1. No portal do Azure, na barra de ferramentas, selecione o ícone **Cloud Shell** localizado à direita da caixa de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

    >**Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

    > **Observação:** para obter uma lista de regiões e seus alias, execute o seguinte comando no Azure Cloud Shell - Bash:

    ```bash
    az account list-locations -o table
    ```

1. No prompt **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar um grupo de recursos (substitua o espaço reservado `<region>` pelo nome da região do Azure mais próxima de você, como “centralus”, “westeurope” ou outra região de escolha).

    ```bash
    LOCATION='<region>'
    ```

    ```bash
    RESOURCEGROUPNAME='az400m05l11-RG'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

1. Execute o comando a seguir para criar um plano do serviço de aplicativo do Windows:

    ```bash
    SERVICEPLANNAME='az400m05l11-sp1'
    az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
    ```

1. Criar um aplicativo Web com um nome exclusivo.

    ```bash
    WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **Observação**: registre o nome do aplicativo Web. Você precisará dela mais adiante neste laboratório.

1. Feche o Azure Cloud Shell, mas deixe o Portal do Azure aberto no navegador.

### Exercício 1: configurar pipelines de CI/CD como código com YAML no Azure DevOps

Neste exercício, você configurará pipelines de CI/CD como código com YAML no Azure DevOps.

#### Tarefa 1: adicionar uma definição de compilação de YAML

Nesta tarefa, você adicionará uma definição de compilação YAML ao projeto existente.

1. Navegue de volta para o painel **Pipelines** no hub **Pipelines**.
1. Na janela **Criar seu primeiro pipeline**, clique em **Criar pipeline**.

    > **Observação**: usaremos o assistente para criar uma nova definição de Pipeline YAML com base no nosso projeto.

1. Na página **Onde está seu código?**, selecione a opção **Git do Azure Repos (YAML)**.
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb_MultiStageYAML**.
1. Na tela **Configurar o pipeline**, role para baixo e selecione **Arquivo YAML existente do Azure Pipelines**.
1. Na folha **Selecionando um arquivo YAML existente**, especifique os seguintes parâmetros:
- Ramificação: **principal**
- Caminho: **.ado/eshoponWeb-ci.yml**
1. Para salvar as configurações, clique em **Continuar**.
1. Na tela **Revisar seu YAML de pipeline**, clique em **Executar** para iniciar o processo Compilar pipeline.
1. Aguarde a conclusão da compilação do pipeline. Ignore os avisos sobre o código-fonte, pois eles não são relevantes para este exercício de laboratório.

    > **Observação**: cada tarefa do arquivo YAML está disponível para revisão, incluindo quaisquer avisos e erros.

#### Tarefa 2: adicionar a entrega contínua à definição de YAML

Nesta tarefa, você adicionará a entrega contínua à definição baseada em YAML do pipeline criado na tarefa anterior.

> **Observação**: agora que os processos de compilação e teste foram bem-sucedidos, podemos adicionar a entrega à definição YAML.

1. No painel de execução de pipeline, clique no símbolo de reticências no canto superior direito e, no menu suspenso, clique em **Editar pipeline**.
1. No painel que exibe o conteúdo do arquivo **eShopOnWeb_MultiStageYAML/.ado/eshoponWeb-ci.yml**, navegue até o final do arquivo (linha 56) e pressione **Enter/Return** para adicionar uma nova linha vazia.
1. Estando na linha **57**, adicione o seguinte conteúdo para definir o estágio de **Lançamento** no pipeline de YAML.

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

1. Defina o cursor em uma nova linha no final da definição de YAML.

    > **Observação**: este será o local onde novas tarefas serão adicionadas.

1. Na lista de tarefas no lado direito do painel de código, procure e selecione a tarefa **Implantação do serviço de aplicativo do Azure**.
1. No painel **Implantação do serviço de aplicativo do Azure**, especifique as seguintes configurações e clique em **Adicionar**:

    - na lista suspensa **Assinatura do Azure**, selecione a assinatura do Azure na qual você implantou os recursos do Azure anteriormente no laboratório, clique em **Autorizar** e, quando solicitado, autentique-se usando a mesma conta de usuário usada durante a implantação de recursos do Azure.
    - na lista suspensa **Nome do serviço de aplicativo**, selecione o nome do aplicativo Web implantado anteriormente no laboratório.
    - na caixa de texto **Pacote ou pasta**, **atualize** o Valor padrão para `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
1. Confirme as configurações no painel Assistente clicando no botão **Adicionar**.

    > **Observação**: Isso adicionará automaticamente a tarefa de implantação à definição de pipeline de YAML.

1. O snippet de código adicionado ao editor deve ser semelhante ao abaixo, refletindo seu nome para os parâmetros azureSubscription e WebappName:

```yaml
    - task: AzureRmWebAppDeployment@4
      inputs:
        ConnectionType: 'AzureRM'
        azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
        appType: 'webApp'
        WebAppName: 'eshoponWebYAML369825031'
        packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
```

1. Validar a tarefa é listado como filho da tarefa **etapas**. Caso contrário, selecione todas as linhas da tarefa adicionada, pressione a tecla **Tab** duas vezes para recuá-la quatro espaços, de modo que ela seja listada como filha da tarefa de **etapas**.

    > **Observação**: o parâmetro **packageForLinux** é enganoso no contexto deste laboratório, mas é válido para Windows ou Linux.

    > **Observação**: por padrão, esses dois estágios são executados de forma independente. Como resultado, a saída de compilação do primeiro estágio pode não estar disponível para o segundo estágio sem alterações adicionais. Para implementar essas alterações, adicionaremos uma nova tarefa para baixar o artefato de implantação no início do estágio de implantação.

1. Coloque o cursor na primeira linha sob o nó **etapas ** do estágio de **implantação** e pressione Enter/Return para adicionar uma nova linha vazia (Linha 64).
1. No painel **Tarefas**, procure e selecione a tarefa **Baixar artefatos de compilação**.
1. Especifique os seguintes parâmetros para esta tarefa:
- Baixar artefatos produzidos por: **Compilação atual**
- Tipo de download: **Artefato específico**
- Nome do artefato: **selecione "Site" na lista**
- Diretório de destino: **$(Build.ArtifactStagingDirectory)**
1. Clique em **Adicionar**.
1. O snippet de código adicionado deve ser semelhante ao abaixo:

```yaml
    - task: DownloadBuildArtifacts@0
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: 'Website'
        downloadPath: '$(Build.ArtifactStagingDirectory)'
```
1. Se o recuo de YAML estiver desativado, com a tarefa adicionada ainda selecionada no editor, pressione a tecla **Tab** duas vezes para recuá-la quatro espaços.

    > **Observação**: aqui também você pode querer adicionar uma linha vazia antes e depois para facilitar a leitura.

1. Clique em **Salvar**, no painel **Salvar**, clique em **Salvar** novamente para confirmar as mudanças diretamente no branch principal.

    > **Observação**: como nosso CI-YAML original não foi configurado para acionar automaticamente uma nova compilação, temos que iniciar esta manualmente.

1. No menu à esquerda do Azure DevOps, navegue até a guia **Pipelines** e selecione **Pipelines** novamente. 
1. Abra o Pipeline **EShopOnWeb_MultiStageYAML** e clique em **Executar pipeline**.
1. Confirme a opção **Executar** no painel exibido.
1. Observe que os 2 estágios diferentes, **Compilar solução do .Net Core** e **Implantar no aplicativo Web do Azure** são exibidos.
1. Aguarde até que o pipeline inicie e até que ele conclua o estágio de compilação com êxito.
1. Quando o Estágio de implantação quiser iniciar, você será solicitado com as **Permissões necessárias**, bem como uma barra laranja dizendo 
```
This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
```
1. Clique na **Exibição**.
1. No painel **Aguardando revisão**, clique em **Permitir**.
1. Valide a mensagem na janela **Permitir pop-up** e confirme clicando em **Permitir**.
1. Isso inicia o Estágio de implantação. Aguarde a conclusão.

     > **Observação**: se a implantação falhar, devido a um problema com a sintaxe do pipeline do YAML, use isso como referência:

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
1. Na folha Aplicativo Web do Azure, clique em **Visão geral** e, na folha Visão geral, clique em **Procurar** para abrir o site em uma nova guia do navegador da Web.
1. Verifique se o site implantado é carregado conforme o esperado na nova guia do navegador, mostrando o site de comércio eletrônico EShopOnWeb.

### Exercício 2: definir as configurações de ambiente para pipelines de CI/CD como código com o YAML no Azure DevOps

Neste exercício, você adicionará aprovações a um pipeline baseado em YAML no Azure DevOps.

#### Tarefa 1: configurar ambientes de pipeline

Os pipelines de YAML como código não têm Quality Gates/Lançamentos como temos com os pipelines de lançamento clássico do Azure DevOps. No entanto, algumas semelhanças podem ser configuradas para os pipelines de YAML como código usando **ambientes**. Nesta tarefa, você usará esse mecanismo para configurar aprovações para o Estágio de compilação.

1. No projeto do Azure DevOps **EShopOnWeb_MultiStageYAML**, navegue até **Pipelines**.
1. No menu Pipelines à esquerda, selecione **Ambientes**.
1. Clique em **Criar ambiente**.
1. No painel **Novo Ambiente**, adicione um Nome para o Ambiente, chamado **aprovações**.
1. Em **Recursos**, selecione **Nenhum**.
1. Confirme as configurações pressionando o botão **Criar**.
1. Assim que o ambiente for criado, clique nas "reticências" (...) ao lado do botão "Adicionar recurso".
1. Selecione **Aprovações e verificações**.
1. Em **Adicionar sua primeira verificação**, selecione **Aprovações**.
1. Adicione o nome da sua conta de usuário do Azure DevOps ao campo **aprovadores**.

    > **Observação:** em um cenário da vida real, isso refletiria o nome da sua equipe de DevOps trabalhando neste projeto.

1. Confirme as configurações de aprovação definidas pressionando o botão **Criar**.
1. Por último, precisamos adicionar as configurações necessárias de "ambiente: aprovações" ao código do pipeline YAML para o estágio de implantação. Para fazer isso, navegue até **Repos**, navegue até a pasta **.ado** e selecione o arquivo de pipeline como código **eshoponWeb-ci.yml**.
1. Na visualização Conteúdo, clique no botão **Editar** para alternar para o modo de edição.
1. Navegue até o início do **Trabalho de implantação** (-trabalho: implantar na Linha 60)
1. Adicione uma nova linha vazia logo abaixo e adicione o seguinte snippet:

```yaml
  environment: approvals
```

o snippet de código resultante deve ter esta aparência:

```yaml
 jobs:
  - job: Deploy
    environment: approvals
    pool:
      vmImage: 'windows-2019'
```
1. Como o ambiente é uma configuração específica de um estágio de implantação, ele não pode ser usado por "trabalhos". Portanto, temos que fazer algumas mudanças adicionais na definição de trabalho atual.
1. Na Linha **60**, renomeie "- trabalho: Implantar" para **- implantação: implantar**
1. Em seguida, na linha **63** (vmImage: Windows-2019), adicione uma nova linha vazia.
1. Cole o seguinte snippet do YAML:

```yaml
    strategy:
      runOnce:
        deploy:
```
1. Selecione o snippet restante (Linha **67** até o final) e use a tecla **Tab** para corrigir o recuo do YAML. 

o snippet de YAML resultante deve ter esta aparência agora, refletindo o **Estágio de implantação**:

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

1. Confirme as alterações no arquivo YAML de código clicando em **Confirmar** e clicando em **Confirmar** novamente no painel Confirmar que é exibido.
1. Navegue até o menu Projeto do Azure DevOps à esquerda, selecione **Pipelines**, selecione **Pipelines** e observe o Pipeline**EshopOnWeb_MultiStageYAML** usado anteriormente.
1. Abra o pipeline.
1. Clique em **Executar pipeline** para acionar uma nova execução de pipeline, confirme clicando em **Executar**.
1. Assim como antes, o estágio de compilação começa como esperado. Aguarde a conclusão com sucesso.
1. Em seguida, como temos *environment:approvals* configurado para o Estágio de implantação, ele solicitará uma confirmação de aprovação antes de começar.
1. Isso é visível na visualização Pipeline, onde diz **Aguardando (0/1 verificações aprovadas).** Uma mensagem de notificação também é exibida dizendo que a **aprovação precisa ser revisada antes que essa execução possa continuar a Implantar em um Aplicativo Web do Azure.**
1. Clique no botão **Exibir** ao lado da mensagem.
1. No painel de exibição **Verificações e validações manuais para Implantar no Aplicativo Web do Azure**, clique na mensagem **Aguardando aprovação**.
1. Clique em **Aprovar**.
1. Isso permite que o Estágio de implantação inicie e implante com êxito o código-fonte do Aplicativo Web do Azure.

    > **Observação:** embora este exemplo tenha usado apenas as aprovações, saiba que as outras verificações, como o Azure Monitor, a API REST, etc., podem ser usadas de maneira semelhante

### Exercício 3: Remover os recursos do laboratório do Azure

Neste exercício, você removerá os recursos do Azure provisionados neste laboratório para eliminar cobranças inesperadas.

>**Observação**: lembre-se de remover todos os recursos do Azure que acabam de ser criados e que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 1: Remover os recursos do laboratório do Azure

Nesta tarefa, você usará o Azure Cloud Shell para remover os recursos do Azure provisionados neste laboratório para eliminar cobranças desnecessárias.

1. No portal do Azure, abra a sessão de shell **Bash** no painel **Cloud Shell**.
1. Liste todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].name" --output tsv
    ```

1. Exclua todos os grupos de recursos criados em todos os laboratórios deste módulo executando o seguinte comando:

    ```sh
    az group list --query "[?starts_with(name,'az400m05l11-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Observação**: o comando é executado de modo assíncrono (conforme determinado pelo parâmetro --nowait), portanto, embora você possa executar outro comando da CLI do Azure imediatamente depois na mesma sessão Bash, levará alguns minutos antes de o grupo de recursos ser removido.

## Revisão

Neste laboratório, você configurou pipelines de CI/CD como código com YAML no Azure DevOps.
