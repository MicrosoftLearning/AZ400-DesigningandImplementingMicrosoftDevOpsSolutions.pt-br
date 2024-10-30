---
lab:
  title: Configurar pipelines como código com YAML
  module: 'Module 03: Design and implement a release strategy'
---

# Configurar pipelines como código com YAML

## Requisitos do laboratório

- Este laboratório requer o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização Azure DevOps que possa usar para este laboratório, crie uma seguindo as instruções disponíveis em [Criar uma organização ou coleção de projetos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique uma assinatura existente do Azure ou crie uma.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função Proprietário na assinatura do Azure e a função Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Visão geral do laboratório

Muitas equipes preferem definir pipelines de compilação e lançamento usando YAML. Isso permite que elas acessem os mesmos recursos de pipeline que as pessoas que usam o designer visual, mas com um arquivo de marcação que pode ser gerenciado como qualquer outro arquivo de origem. Definições de compilação de YAML podem ser adicionadas a um projeto simplesmente adicionando os arquivos correspondentes à raiz do repositório. O Azure DevOps também fornece modelos padrão para tipos de projeto populares e um designer de YAML para simplificar o processo de definição de tarefas de compilação e lançamento.

## Objetivos

Após concluir este laboratório, você poderá:

- Configurar pipelines de CI/CD como código com YAML no Azure DevOps.

## Tempo estimado: 45 minutos

## Instruções

### Exercício 0: configurar os pré-requisitos do laboratório

Neste exercício, você configurará os pré-requisitos para o laboratório.

#### Tarefa 1: (pular se feita) criar e configurar o projeto de equipe

Nesta tarefa, você criará um projeto **eShopOnWeb_MultiStageYAML** do Azure DevOps para ser usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps. Clique em **Novo projeto**. Dê ao projeto o nome **eShopOnWeb_MultiStageYAML** e deixe os outros campos com padrões. Clique em **Criar**.

   ![Captura de tela do painel criar um novo projeto.](images/create-project.png)

#### Tarefa 2: (pular se feita) importar repositório do Git eShopOnWeb

Nesta tarefa, você importará o repositório eShopOnWeb do Git que será usado por vários laboratórios.

1. No computador do laboratório, em uma janela do navegador, abra sua organização do Azure DevOps e o projeto **eShopOnWeb_MultiStageYAML** criado anteriormente. Clique em **Repos > Arquivos**, **Importar um repositório**. Selecione **Importar**. Na janela **Importar um repositório do Git**, cole a seguinte URL https://github.com/MicrosoftLearning/eShopOnWeb.git e clique em **Importar**:

   ![Captura de tela do painel importar repositório.](images/import-repo.png)

1. O repositório está organizado da seguinte forma:
   - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
   - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
   - A pasta **infra** contém a infraestrutura Bicep e ARM como modelos de código usados em alguns cenários de laboratório.
   - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
   - A pasta **src** contém o site do .NET 8 usado em cenários de laboratório.

1. Vá para **Repos > Branches**.
1. Passe o mouse sobre o branch **main** e clique nas reticências à direita da coluna.
1. Clique em **Definir como branch padrão**.

#### Tarefa 3: Criar recursos do Azure

Nesta tarefa, você usará o portal do Azure para criar um aplicativo Web do Azure.

1. No computador do laboratório, inicie um navegador da Web, navegue até o [**Portal do Azure**](https://portal.azure.com) e entre com a conta de usuário que tem a função Proprietário na assinatura do Azure que você usará neste laboratório e tem a função de Administrador global no locatário do Microsoft Entra associado a essa assinatura.
1. No portal do Azure, na barra de ferramentas, clique no ícone do **Cloud Shell** localizado ao lado da caixa de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   > **Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** e você receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

   > **Observação**: para obter uma lista de regiões e seus alias, execute o seguinte comando no Azure Cloud Shell – Bash:

   ```bash
   az account list-locations -o table
   ```

1. No prompt **Bash**, no painel do **Cloud Shell**, execute o seguinte comando para criar um grupo de recursos (substitua o espaço reservado `<region>` pelo nome da região do Azure mais próxima de você, como "centralus", "westeurope" ou outra região de escolha).

   ```bash
   LOCATION='<region>'
   ```

   ```bash
   RESOURCEGROUPNAME='az400m03l07-RG'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. Execute o comando a seguir para criar um plano do Serviço de Aplicativo.

   ```bash
   SERVICEPLANNAME='az400m03l07-sp1'
   az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
   ```

1. Crie um aplicativo Web com um nome exclusivo.

   ```bash
   WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **Observação**: registre o nome do aplicativo Web. Você precisará dela posteriormente neste laboratório.

1. Feche o Azure Cloud Shell, mas deixe o Portal do Azure aberto no navegador.

### Exercício 1: configurar pipelines de CI/CD como código com YAML no Azure DevOps

Neste exercício, você vai configurar pipelines de CI/CD como código com YAML no Azure DevOps.

#### Tarefa 1: adicionar uma definição de compilação do YAML

Nesta tarefa, você adicionará uma definição de compilação do YAML ao projeto existente.

1. Navegue de volta ao painel **Pipelines** no hub **Pipelines**.
1. Na janela **Criar seu primeiro pipeline**, clique em **Criar pipeline**.

   > **Observação**: usaremos o assistente para criar uma nova definição de pipeline do YAML com base em nosso projeto.

1. No painel **Onde está seu código?**, clique na opção **Git do Azure Repos (YAML).**
1. No painel **Selecionar um repositório**, clique em **eShopOnWeb_MultiStageYAML**.
1. No painel **Configurar seu pipeline**, role para baixo e selecione **Arquivo YAML existente do Azure Pipelines**.
1. Na folha **Selecionar um arquivo YAML existente** , especifique os seguintes parâmetros:
   - Ramificação: **principal**
   - Caminho: **.ado/eshoponweb-ci.yml**
1. Clique em **Continuar** para salvar essas configurações.
1. Na tela **Revisar seu YAML de pipeline**, clique em **Executar** para iniciar o processo de Pipeline de build.
1. Aguarde a conclusão do pipeline de build. Ignore quaisquer avisos sobre o código-fonte em si, pois eles não são relevantes para este exercício de laboratório.

   > **Observação**: cada tarefa do arquivo YAML está disponível para revisão, incluindo quaisquer avisos e erros.

#### Tarefa 2: adicionar entrega contínua à definição do YAML

Nesta tarefa, você adicionará a entrega contínua à definição baseada em YAML do pipeline criado na tarefa anterior.

> **Observação**: agora que os processos de compilação e teste foram bem-sucedidos, podemos adicionar a entrega à definição do YAML.

1. No painel de execução do pipeline, clique no símbolo de reticências no canto superior direito e, no menu suspenso, clique em **Editar pipeline**.
1. No painel que exibe o conteúdo do arquivo **eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml**, navegue até o final do arquivo (linha 56) e pressione **Enter/Return** para adicionar uma nova linha vazia.
1. Estando na linha **57**, adicione o seguinte conteúdo para definir o estágio de **Lançamento** no pipeline YAML.

   > **Observação**: você pode definir os estágios necessários para organizar e acompanhar melhor o progresso do pipeline.

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - job: Deploy
         pool:
           vmImage: "windows-latest"
         steps:
   ```

1. Coloque o cursor em uma nova linha no final da definição do YAML.

   > **Observação**: este será o local onde novas tarefas serão adicionadas.

1. Na lista de tarefas no lado direito do painel de código, procure e selecione a tarefa ** Implantação do Serviço de Aplicativo do Azure**.
1. No painel **Implantação do Serviço de Aplicativo do Azure**, especifique as seguintes configurações e clique em **Adicionar**:

   - na lista suspensa **Assinatura do Azure**, selecione a assinatura do Azure na qual você implantou os recursos do Azure anteriormente no laboratório, clique em **Autorizar** e, quando solicitado, autentique-se usando a mesma conta de usuário usada durante a implantação de recursos do Azure.
   - na lista suspensa **Nome do Serviço de Aplicativo**, selecione o nome do aplicativo Web implantado anteriormente no laboratório.
   - na caixa de texto **Pacote ou pasta**, **atualize** o Valor Padrão para `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
   - Em **Definições de Aplicativo e Configuração**, adicione `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`

1. Confirme as configurações no painel Assistente clicando no botão **Adicionar** .

   > **Observação**: isso adicionará automaticamente a tarefa de implantação à definição de pipeline YAML.

1. O snippet de código adicionado ao editor deve ser semelhante ao abaixo, refletindo seu nome para os parâmetros azureSubscription e WebappName:

   ```yaml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
       appType: "webApp"
       WebAppName: "eshoponWebYAML369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Validar a tarefa é listado como filho da tarefa **etapas**. Caso contrário, selecione todas as linhas da tarefa adicionada, pressione a tecla **Tab** duas vezes para recuá-la quatro espaços, de modo que ela seja listada como filha da tarefa **etapas**.

   > **Observação**: o parâmetro **packageForLinux** é enganoso no contexto deste laboratório, mas é válido para Windows ou Linux.

   > **Observação**: por padrão, esses dois estágios são executados de forma independente. Como resultado, a saída da compilação do primeiro estágio pode não estar disponível para o segundo estágio sem alterações adicionais. Para implementar essas alterações, adicionaremos uma nova tarefa para baixar o artefato da implantação no início do estágio de implantação.

1. Coloque o cursor na primeira linha sob o nó **etapas** do estágio de **implantação** e pressione Enter/Return para adicionar uma nova linha vazia (Linha 64).
1. No painel **Tarefas**, procure e selecione a tarefa **Baixar artefatos de compilação**.
1. Especifique os seguintes valores para esta tarefa:
   - Baixar artefatos produzidos por: **compilação atual**
   - Tipo de download: **artefato específico**
   - Nome do artefato: **selecione "Site" na lista** (ou **digite "`Website`" diretamente** se ele não aparecer automaticamente na lista)
   - Diretório de destino: **$(Build.ArtifactStagingDirectory)**
1. Clique em **Adicionar**.
1. O trecho de código adicionado deve ser semelhante ao abaixo:

   ```yaml
   - task: DownloadBuildArtifacts@1
     inputs:
       buildType: "current"
       downloadType: "single"
       artifactName: "Website"
       downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. Se o recuo do YAML estiver desativado, com a tarefa adicionada ainda selecionada no editor, pressione a tecla **Tab** duas vezes para recuá-la quatro espaços.

   > **Observação**: aqui você também pode querer adicionar uma linha vazia antes e depois para facilitar a leitura.

1. Clique em **Salvar**. No painel **Salvar**, clique em **Salvar** novamente para confirmar a alteração diretamente no branch main.

   > **Observação**: como nosso CI-YAML original não foi configurado para acionar automaticamente uma nova compilação, temos que iniciar esta manualmente.

1. No menu Azure DevOps à esquerda, navegue até **Pipelines** e selecione **Pipelines** novamente.
1. Abra o pipeline **eShopOnWeb_MultiStageYAML** e clique em **Executar Pipeline**.
1. Confirme a opção **Executar** no painel exibido.
1. Duas fases diferentes são exibidas, **Compilar solução .Net Core** e **Implantar no aplicativo Web do Azure**.
1. Aguarde até que o pipeline seja iniciado e conclua a fase Compilar.
1. Quando a Fase Implantar quiser iniciar, será solicitado as **Permissões Necessárias**, e aparecerá uma barra laranja dizendo:

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. Clique em **Exibição**.
1. No painel **Aguardando revisão**, clique em **Permitir**.
1. Valide a mensagem na janela **pop-up Permitir** e confirme clicando em **Permitir**.
1. Isso inicia a Fase Implantar. Aguarde a conclusão.

   > **Observação**: se a implantação falhar devido a um problema com a sintaxe do pipeline YAML, use isso como referência:

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
          PathtoPublish: '$(Build.SourcesDirectory)/infra/webapp.bicep'
          ArtifactName: 'Bicep'
          publishLocation: 'Container'

   - stage: Deploy
    displayName: Deploy to an Azure Web App
    jobs:
    - job: Deploy
      pool:
        vmImage: 'windows-latest'
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
          AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'

   ```

#### Tarefa 3: Revisar o site implantado

1. Volte para a janela do navegador da Web que exibe o portal do Azure e navegue até a folha que exibe as propriedades do aplicativo Web do Azure.
1. Na folha Aplicativo Web do Azure, clique em **Visão geral** e, na folha Visão geral, clique em **Procurar** para abrir seu site em uma nova guia do navegador da Web.
1. Verifique se o site implantado é carregado conforme o esperado na nova guia do navegador, mostrando o site de comércio eletrônico eShopOnWeb.

### Exercício 2: definir as configurações de ambiente para pipelines de CI/CD como código com o YAML no Azure DevOps

Neste exercício, você adicionará aprovações a um pipeline baseado no YAML no Azure DevOps.

#### Tarefa 1: configurar ambientes de pipeline

Os Pipelines do YAML como Código não têm Portões de Versão/Qualidade como temos com os Pipelines de Versão Clássica do Azure DevOps. No entanto, algumas semelhanças podem ser configuradas para os Pipelines do YAML como código usando **Ambientes**. Nesta tarefa, você usará esse mecanismo para configurar aprovações para a Fase de build.

1. No projeto **eShopOnWeb_MultiStageYAML** do Azure DevOps, navegue até **Pipelines**.
1. No menu Pipelines à esquerda, selecione **Ambientes**.
1. Clique em **Criar ambiente**.
1. No painel **Novo Ambiente**, adicione um Nome para o Ambiente, chamado **`approvals`**.
1. Em **Recursos**, selecione **Nenhum**.
1. Confirme as configurações pressionando o botão **Criar**.
1. Quand o ambiente for criado, clique na guia **Aprovações e verificações** no novo ambiente **aprovações**.
1. Em **Adicionar sua primeira verificação**, selecione **Aprovações**.
1. Adicione seu Nome de Conta de Usuário do Azure DevOps ao campo de **aprovadores**.

   > **Observação:** em um cenário da vida real, isso refletiria o nome da sua equipe de DevOps trabalhando neste projeto.

1. Confirme as configurações de aprovação definidas, pressionando o botão **Criar**.
1. Por último, precisamos adicionar as configurações necessárias de "environment: approvals" ao código de pipeline do YAML para a Fase de implantação. Para fazer isso, navegue até **Repos**. Navegue até a pasta **.ado** e selecione o arquivo Pipeline como código **eshoponweb-ci.yml**.
1. Na exibição Conteúdo, clique no botão **Editar** para alternar para o modo de edição.
1. Navegue até o início do **Implantar trabalho** (-job: Implantar na Linha 60)
1. Adicione uma nova linha vazia logo abaixo e adicione o seguinte snippet:

   ```yaml
   environment: approvals
   ```

   O snippet de código resultante deve parecer com:

   ```yaml
   jobs:
     - job: Deploy
       environment: approvals
       pool:
         vmImage: "windows-latest"
   ```

1. Como o ambiente é uma configuração específica de uma fase de implantação, ele não pode ser usado por "trabalhos". Portanto, temos que fazer algumas mudanças adicionais na definição de trabalho atual.
1. Na Linha **60**, renomeie "- job: Deploy" para **- deployment: Deploy**
1. Em seguida, na Linha **63** (vmImage: windows-latest), adicione uma nova linha vazia.
1. Cole o seguinte snippet do YAML:

   ```yaml
   strategy:
     runOnce:
       deploy:
   ```

1. Selecione o snippet restante (Linha **67** até o final) e use a tecla **Tab** para corrigir o recuo do YAML.

   O snippet do YAML resultante agora ficará assim, refletindo ao **Fase de build**:

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - deployment: Deploy
         environment: approvals
         pool:
           vmImage: "windows-latest"
         strategy:
           runOnce:
             deploy:
               steps:
                 - task: DownloadBuildArtifacts@1
                   inputs:
                     buildType: "current"
                     downloadType: "single"
                     artifactName: "Website"
                     downloadPath: "$(Build.ArtifactStagingDirectory)"
                 - task: AzureRmWebAppDeployment@4
                   inputs:
                     ConnectionType: "AzureRM"
                     azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
                     appType: "webApp"
                     WebAppName: "eshoponWebYAML369825031"
                     packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
                     AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Confirme as alterações no arquivo YAML de código clicando em **Confirmar** e clicando em **Confirmar** novamente no painel Confirmar que aparece.
1. Navegue até o menu Projeto do Azure DevOps à esquerda, selecione **Pipelines**, selecione **Pipelines** e veja o pipeline **EshopOnWeb_MultiStageYAML** usado anteriormente.
1. Abra o pipeline.
1. Clique em **Executar Pipeline** para disparar uma nova execução de pipeline; confirme clicando em **Executar**.
1. Assim como antes, a Fase de build começa como esperado. Aguarde a conclusão.
1. Em seguida, como temos o _environment:approvals_ configurado para a Fase de build, ele solicitará uma confirmação de aprovação antes de começar.
1. Isso é visível na exibição Pipeline, onde diz **Aguardando (0/1 verificação aprovada).** Uma mensagem de notificação também é exibida dizendo que a **aprovação precisa ser revisada antes que essa execução possa continuar a Implantar em um Aplicativo Web do Azure**.
1. Clique no botão **Exibir** ao lado da mensagem.
1. No painel exibido **Verificações e validações manuais para Implantar no Aplicativo Web do Azure**, clique na mensagem **Aguardando Aprovação**.
1. Clique em **Aprovar**.
1. Isso permite que a Fase de build inicie e implante com êxito o código-fonte do Aplicativo Web do Azure.

   > **Observação:** embora este exemplo tenha usado apenas as aprovações, saiba que as outras verificações, como o Azure Monitor, a API REST, etc., podem ser usadas de maneira semelhante

   > [!IMPORTANT]
   > Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você configurou pipelines de CI/CD como código com o YAML no Azure DevOps.
